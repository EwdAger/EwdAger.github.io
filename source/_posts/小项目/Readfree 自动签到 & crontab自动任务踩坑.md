---
title: Readfree 自动签到 & crontab自动任务踩坑
tags:
  - 小项目
  - Linux
  - python
categories: 小项目
abbrlink: 2ba86c7b
date: 2018-08-07 10:06:00
---

# 自动签到 Python 脚本

这部分没什么难度，主要是这个网站的`cookies`的`Max-Age`有`31449600`秒，大概1年的寿命，所以直接将存好的`cookies`用`requests`发一个get请求到验证地址就行。稍微修改了[博主杨英明](http://www.yangyingming.com/article/381/)代码如下：

```python
import requests
import time

# 登录验证地址
check_url = 'http://readfree.me/accounts/checkin'

# 记录程序运行时的时间
fp = open('./auto_signon_readfree.log','a')
ISOTIMEFORMAT='%Y-%m-%d %X'
curtime = time.strftime(ISOTIMEFORMAT, time.localtime(time.time()))
print ('at %s'%curtime)
fp.write('at %s\n'%curtime)

# 准备cookie
print ('准备cookie中……')
fp.write('准备cookie中……\n')
cookie_str = 'cookies here'
cookie = {}
for line in cookie_str.split(';'):
    name,value=line.strip().split('=',1)
    cookie[name]=value
print (cookie)
fp.write('%s\n'%cookie)

# 使用cookie访问网站
print ('签到中……')
fp.write('签到中……\n')
res = requests.get(check_url,cookies=cookie)
print (res)
fp.write('%s\n\n'%res)
```


# crontab 自动任务

这一步确实是踩了不少坑，还是Linux知识太欠缺了。

## 几次失败

1. 直接在`crontab -e`中加入指令`0 1 * * * python3 ~/autoSign/autoSign_readfree/py`不执行
2. 更换`python3`绝对路径后依旧不执行
3. 查看`crontab`log发现文件不存在
4. `.py`文件头部添加`#!/usr/bin/env python3`报错`env: python\r: No such file or directory`

## 几次尝试

1. 1-2 几次修改后依旧无果，在尝试`2`的后依旧不执行，考虑用新自动任务[输出`hello`到log](https://www.aliyun.com/jiaocheng/121986.html)检测`crontab`是否出错，发现`crontab`能正常运行，随后考虑修复`3`问题

2. 经查询发现`crontab`是默认不开启log功能的，解决方法如下：
修改`rsyslog`服务，将 `/etc/rsyslog.d/50-default.conf`  文件中的 `#cron.*` 前的 `#`删掉,再使用`service rsyslog restart；`重启`rsyslog`服务

3. 查询log文件发现.py依旧不执行，只能剑走偏锋不直接在`crontab`中使用`python3`命令，遂使用`4`在python文件头部添加`#!/usr/bin/env python3`让`crontab`以类似运行`.sh`文件的形式打开`.py`。
在使用`chmod a+x autoSign_readfree.py`修改权限后报错`env: python\r: No such file or directory`
参考[stackoverflow](https://stackoverflow.com/questions/19425857/env-python-r-no-such-file-or-directory)发现是脚本包含CR字符。shell将这些CR字符解释为参数。使用以下脚本去除CR字符:
```python
with open('autoSign_readfree.py', 'rb+') as f:
    content = f.read()
    f.seek(0)
    f.write(content.replace(b'\r', b''))
    f.truncate()
```
## 成功！
最后在`crontab`中添加命令`0 1 * * * ./autoSign_readfree.py`大功告成！(据说每条命令必须换行才能执行)

# 补充几个`crontab`小知识

1. 添加`crontab`任务
```bash
crontab -e
```

2. 查看`crontab`任务
```bash
crontab -l
```

3. 使用实例

```bash
0 2 * * * /bin/sh backup.sh                                   //每天 02:00 执行任务
0 5,17 * * * /scripts/script.sh                               //每天 5:00和17:00执行任务
* * * * *  /scripts/script.sh                                 //每分钟执行一次任务
0 17 * * sun  /scripts/script.sh                              //每周日 17:00 执行任务
*/10 * * * * /scripts/monitor.sh                              //每 10min 执行一次任务
* * * jan,may,aug * /script/script.sh                         //在特定的某几个月执行任务
0 17 * * sun,fri /script/scripy.sh                            //在特定的某几天执行任务
0 2 * * sun  [ $(date +%d) -le 07 ] && /script/script.sh      //在某个月的第一个周日执行任务
* * * * *  sleep 30; /scripts/script.sh                       //每个30秒执行一次任务
* * * * * /scripts/script.sh; /scripts/scrit2.sh              //多个任务在一条命令中配置
@reboot /scripts/script.sh                                    //系统重启时执行
```