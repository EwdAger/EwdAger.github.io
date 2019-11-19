---
title: requests 代理问题
tags:
  - 爬虫
  - requests
categories: python学习心得&备忘
abbrlink: 7c092ea
date: 2018-07-13 13:15:10
---

在给一个本地的Flask项目测试post接口时遇到一个问题，无论用requests的get还是post请求localhost全部都会超时。

经过仔细分析后，发现是开了系统代理的锅(毕竟写代码少不了Google)，然而直接关闭系统代理仍然超时。

最后的解决方案如下：

```python
import os
import requests

os.environ['NO_PROXY'] = '127.0.0.1'
r = requests.get('http://127.0.0.1:5000')
print(r.content)
```

设置不走代理的url，而不是直接把请求的proxies设置为本地代理！


参考：

[stackoverflow: requests-how-to-disable-bypass-proxy](https://stackoverflow.com/questions/28521535/requests-how-to-disable-bypass-proxy)

[stackoverflow: python-requests-return-504-in-localhost](https://stackoverflow.com/questions/43254103/python-requests-return-504-in-localhost)

[github: Issues with HTTP proxy and accessing localhost - does requests ignore no_proxy?](https://github.com/requests/requests/issues/879)