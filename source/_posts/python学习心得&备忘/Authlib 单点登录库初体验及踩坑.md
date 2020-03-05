---
title: Authlib 单点登录库初体验及踩坑
tags:
  - Flask
  - OAuth
  - 单点登录
  - authlib
categories: python学习心得&备忘
abbrlink: 51c3bc14
date: 2020-03-05 20:14:10
---

## 起因

项目突然要接入TX云，理所应当的要使用tx的单点登录了。于是乎，经过各方推荐，使用了大名鼎鼎的[Authlib](https://authlib.org/)库。

## 初体验

经过各方文档，整理了一下，在`Flask`中使用`Authlib`相当简单。如果是接入有名的`OAuth2`站点如`Github`、`Google`这种，直接使用官方已经封装好的类即可快速实现，但此处使用的是TX方为工业互联网平台新搭建的`OAuth2`服务，理所应当不能直接使用。但仍可以使用较为便捷的封装进`Flask`中的认证方法，具体步骤如下：


### 新建存储Token的表

根据存储的`access_token`校验后续接口用户登录情况。 其中`refresh_token`为`access_token`过期后，下次去`OAuth2`服务器请求新的Token的字段。如果在注册`Authlib`对象时写了`update`方法，即可自动更新token。


```python
from app.models.base import db, Base


class OAuth2Token(db.Model):
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(40))
    token_type = db.Column(db.String(40))
    access_token = db.Column(db.String(200))
    refresh_token = db.Column(db.String(200))
    expires_in = db.Column(db.Integer())
    user_id = db.Column(db.Integer(), db.ForeignKey("user.id"))
    user = db.relationship("User", backref=db.backref("auth_token", lazy="dynamic"))

    def to_token(self):
        return dict(
            access_token=self.access_token,
            token_type=self.token_type,
            refresh_token=self.refresh_token,
            expires_in=self.expires_in,
        )

    @staticmethod
    def save_token(response):
        with db.auto_commit():
            oauth = OAuth2Token()
            oauth.name = 'tencent'
            oauth.access_token = response['access_token']
            oauth.token_type = response['token_type']
            oauth.expires_in = response['expires_in'] if 'expires_in' in response else None
            oauth.refresh_token = response['refresh_token'] if 'refresh_token' in response else None
            oauth.user_id = response['user_id'] if 'user_id' in response else None
            db.session.add(oauth)
```

### 创建oauth对象

`authlib`会自动从`flask`的全局`config`即`app/config/setting.py`中读取你设置的`{YOUR_NAME}_CLIENT_ID`和`{YOUR_NAME}__CLIENT_SECRET`。

`authorize_url`中填写第一步跳转到单点登录接口的url，`access_token_url`填写从单点登录服务器拿到code和state后，换取token的地址。


```python
from authlib.integrations.flask_client import OAuth
from app.models.oauth import OAuth2Token
from flask import g
from app.models.base import db

from app.config.setting import TENCENT_CLIENT_ID, TENCENT_CLIENT_SECRET


def fetch_token(name):
    token = OAuth2Token.query.filter_by(name=name, user_id=g.user.uid).first()
    return token.to_token()


def update_token(name, token, refresh_token=None, access_token=None):
    if refresh_token:
        item = OAuth2Token.filter_by(name=name, refresh_token=refresh_token).first()
    elif access_token:
        item = OAuth2Token.filter_by(name=name, access_token=access_token).first()
    else:
        return
    if not item:
        return
    # update old token
    item.access_token = token['access_token']
    item.refresh_token = token.get('refresh_token')
    item.expires_in = token['expires_in']
    db.session.commit()


oauth = OAuth(fetch_token=fetch_token, update_token=update_token)

tencent = oauth.register(
    name='tencent',
    access_token_url=f'{CloudIndustryHost_internet}',
    access_token_params={
        'method': "GET",
        'grant_type': 'authorization_code'
    },
    authorize_url=f'{CloudIndustryHost_internet}',
    authorize_params=None,
    api_base_url=f'{CloudIndustryHost_internet}',
    client_kwargs={'scope': 'all'}
)
```

### 视图函数

```python
from flask import url_for, redirect, make_response

from app.models.oauth import OAuth2Token
from app.libs.oauthlib import tencent
from . import oauth as api
from .utils import *


@api.route('/login', methods=['GET', 'POST'])
def login():
    redirect_uri = url_for("v1.oauth+auth", _external=True)
    return tencent.authorize_redirect(redirect_uri)


@api.route('/redirect', methods=['GET', 'POST'])
def auth():
    token_info = tencent.authorize_access_token()

    OAuth2Token.save_token(token_info)
    response = make_response(redirect('/'))
    response.set_cookies('session_id', token_info['access_token'])
```


### 将oauth对象注册进flask app

```python
from app.libs.oauthlib import oauth


app = create_app()

oauth.init_app(app)
```

以上基本就能正常愉快的完成单点登录的全过程啦~


## 踩坑

好吧，实际上并不是这么一帆风顺的，在取得返回的code的视图函数中`authorize_access_token()`这一步一直碰到一个`JSONDecodeError: Extra data: line 1 column 5 - line 1 column 19 (char 4 - 18)`json无法序列化的bug。

经过进入`Authlib`的源码中深入调查，发现在`fetch_token()`这一步中，用`OAuth`服务器返回的code、states等参数向获取token的接口发请求时直接报404了。

反复查看文档发现地址并没有填错，最后发现，TX云那边使用的是GET方法拿token，而`OAuthlib`的`fetch_token()`方法默认使用的是`POST`方法！！！

而要额外设置`fetch_token()`的方法需要再编写`oath`对象时，在`access_token_params`中加入所需要的方法。

`authorize_access_token()`方法会默认将`access_token_params`作为`**kwarg`传入后续的内部函数中，即可设置`fetch_token()`的参数。
