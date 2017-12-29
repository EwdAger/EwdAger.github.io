---
title: python实现单工、半双工、全双工聊天室
date: 2017-12-29 08:56:16
tags: 
 - python
 - 计算机网络
 - Socket编程
 - 聊天室
categories: 计算机网络相关
---

## 聊天室单工实现：

单工版非常简单，只能客户端单方面向服务端发消息，服务端回复固定模板消息。

**Server:**

```python
# -*- coding: utf-8 -*-

import socket
import threading
import time

def tcplink(sock, addr):
    print 'Accept new connection from %s:%s...' % addr
    sock.send('Welcome!')
    while True:
        data = sock.recv(1024)
        time.sleep(1)
        if data == 'exit' or not data:
            break
        sock.send('Hello, %s!' % data)
    sock.close()
    print 'Connection from %s:%s closed.' % addr

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('127.0.0.1', 9999))
s.listen(5)
print 'Waiting for connection...'
while True:
    # 接受一个新连接:
    sock, addr = s.accept()
    # 创建新线程来处理TCP连接:
    t = threading.Thread(target=tcplink, args=(sock, addr))
    t.start()
```

<!-- more -->

**Client:**
```python
# -*- coding: utf-8 -*-

import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 建立连接:
s.connect(('127.0.0.1', 9999))
# 接收欢迎消息:
print s.recv(1024)
for data in ['Michael', 'Tracy', 'Sarah']:
    # 发送数据:
    s.send(data)
    print s.recv(1024)
s.send('exit')
s.close()
```

此为最基础的基于TCP协议的聊天程序，实现了Socket编程的主要流程。

> 服务器进程首先要绑定一个端口并监听来自其他客户端的连接。如果某个客户端连接过来了，服务器就与该客户端建立Socket连接，随后的通信就靠这个Socket连接了。
所以，服务器会打开固定端口（比如80）监听，每来一个客户端连接，就创建该Socket连接。由于服务器会有大量来自客户端的连接，所以，服务器要能够区分一个Socket连接是和哪个客户端绑定的。一个Socket依赖4项：服务器地址、服务器端口、客户端地址、客户端端口来唯一确定一个Socket。
但是服务器还需要同时响应多个客户端的请求，所以，每个连接都需要一个新的进程或者新的线程来处理，否则，服务器一次就只能服务一个客户端了。

## 聊天室半双工实现：

半双工实现是连接建立以后，服务器等待客户端发送消息，客户端发送消息后等待接收服务器，这样一来一回循环往复下去。直到出现quit，关闭连接。

**Server:**
```python
# -*- coding: utf-8 -*-

import socket
from time import ctime

HOST = 'localhost'
PORT = 3300
BUSIZ = 1024
ADDR = (HOST, PORT)

def closeTCnt():
    TCntSock.close()
    print "Session closing.."

TSerSock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
TSerSock.bind(ADDR)
TSerSock.listen(1)
try:
    while True:
        print 'Waitting for connection...'
        (TCntSock, cntAddr) = TSerSock.accept()
        print '...connection from:', cntAddr

        try:

            while True:
                rData = TCntSock.recv(BUSIZ)
                if not rData:
                    continue
                elif rData == 'quit':
                    break
                else:
                    print 'From [%s] %s \n  %s' % (cntAddr[0], ctime(), rData)

                while True:
                    sData = raw_input('>  ')
                    if not sData:
                        continue
                    else:
                        TCntSock.send('From [%s] %s \n  %s' %
                                      (cntAddr[0], ctime(), sData))
                        break

        except socket.error, detail:
            print detail
        closeTCnt()

finally:
    TSerSock.close()
```

**Client:**
```python
# -*- coding: utf-8 -*-

import socket

HOST = 'localhost'
PORT = 3300
BUFSIZ = 1024
ADDR = (HOST, PORT)
tryCon = 0

def TCnt():
    tcpCliSock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    while True:
        try:
            tcpCliSock.connect(ADDR)
        except:
            print u"正在尝试连接远程主机 "
            global tryCon
            tryCon += 1
            if tryCon == 3:
                print u"无法连接上远程主机，请稍后再试"
                exit()
        else:
            break
    print u'登陆成功（通讯结束请输入"quit"退出）\n'

    try:
        while True:
            data = raw_input('>  ')
            if not data:
                continue
            elif data == 'quit':
                tcpCliSock.send(data)
                break
            else:
                tcpCliSock.send(data)

            while True:
                data = tcpCliSock.recv(BUFSIZ)

                if not data:
                    continue
                else:
                    print data
                    break
    except socket.error, e:
        print "Session closing"
        print e
    tcpCliSock.close()


if __name__ == "__main__":
    TCnt()

```

这里就出现了一个有趣的问题，为什么这段代码只能一来一回的发送消息呢？讲道理应该发完消息不应该可以接着发消息吗？凭什么发了一条消息必须等待另一端发消息回来才能继续发？这就引出了全双工实现的原理。


## 聊天室全双工(P2P)实现：
因为TCP连接是一个流，所以Socket模块的recv()是**直到Scoket连接终断不会停止等待接受从另一端发送**的消息的。全双工实现比半双工工多了个线程处理，所以服务器与客户端必须开两个线程，一个收消息一个发消息，并且发消息的线程需要阻塞收消息的线程。

**Server:**
```python
# -*- coding: utf-8 -*-

from socket import *
from time import ctime
import threading
import re

HOST = ''
PORT = 9999
BUFSIZ = 1024
ADDR = (HOST, PORT)

tcpSerSock = socket(AF_INET, SOCK_STREAM)
tcpSerSock.bind(ADDR)
tcpSerSock.listen(5)

clients = {}  # username -> socket
chatwith = {}  # user1.socket -> user2.socket


# clients字典中记录了连接的客户端的用户名和套接字的对应关系
# chatwith字典中记录了通信双方的套接字的对应

# messageTransform()处理客户端确定用户名之后发送的文本
# 文本只有四种类型：
#   None
#   Quit
#   To:someone
#   其他文本
def messageTransform(sock, user):
    while True:
        data = sock.recv(BUFSIZ)
        if not data:
            if chatwith.has_key(sock):
                chatwith[sock].send(data)
                del chatwith[chatwith[sock]]
                del chatwith[sock]
            del clients[user]
            sock.close()
            break
        if data == 'Quit':
            sock.send(data)
            if chatwith.has_key(sock):
                data = '%s.' % data
                chatwith[sock].send(data)
                del chatwith[chatwith[sock]]
                del chatwith[sock]
            del clients[user]
            sock.close()
            break
        elif re.match('^To:.+', data) is not None:
            data = data[3:]
            if clients.has_key(data):
                if data == user:
                    sock.send('Please don\'t try to talk with yourself.')
                else:
                    chatwith[sock] = clients[data]
                    chatwith[clients[data]] = sock
            else:
                sock.send('the user %s is not exist' % data)
        else:
            if chatwith.has_key(sock):
                chatwith[sock].send('[%s] %s: (%s)' % (ctime(), user, data))
            else:
                sock.send('Nobody is chating with you. Maybe the one talked with you is talking with someone else')


# 每个客户端连接之后，都会启动一个新线程
# 连接成功后需要输入用户名
# 输入的用户名可能会：
#   已存在
#   (客户端直接输入ctrl+c退出)
#   合法用户名
def connectThread(sock, test):  # client's socket

    user = None
    while True:  # receive the username
        username = sock.recv(BUFSIZ)
        if not username:  # the client logout without input a name
            print('The client logout without input a name')
            break
        if clients.has_key(username):  # username existed
            sock.send('Reuse')
        else:  # correct username
            sock.send('OK')
            clients[username] = sock  # username -> socket
            user = username
            break
    if not user:
        sock.close()
        return
    print('The username is: %s' % user)
    # get the correct username

    messageTransform(sock, user)


if __name__ == '__main__':
    while True:
        print('...WAITING FOR CONNECTION')
        tcpCliSock, addr = tcpSerSock.accept()
        print('CONNECTED FROM: ', addr)
        chat = threading.Thread(target=connectThread, args=(tcpCliSock, None))
        chat.start()
```

**Client:**
```python
# -*- coding: utf-8 -*-

from socket import *
import threading

HOST = '127.0.0.1'
PORT = 9999
BUFSIZ = 1024
ADDR = (HOST, PORT)

tcpCliSock = socket(AF_INET, SOCK_STREAM)
tcpCliSock.connect(ADDR)
'''
因为每个客户端接收消息和发送消息是相互独立的，
所以这里将两者分开，开启两个线程处理
'''

def Send(sock, test):
    while True:
        try:
            data = raw_input()
            sock.send(data)
            if data == 'Quit':
                break
        except KeyboardInterrupt:
            sock.send('Quit')
            break


def Recv(sock, test):
    while True:
        data = sock.recv(BUFSIZ)
        if data == 'Quit.':
            print('He/She logout')
            continue
        if data == 'Quit':
            break
        print('         %s' % data)

if __name__ == '__main__':
    print('Successful connection')
    while True:
        username = raw_input('Your name(press only Enter to quit): ')
        tcpCliSock.send(username)
        if not username:
            break
        # username is not None
        response = tcpCliSock.recv(BUFSIZ)
        if response == 'Reuse':
            print('The name is reuse, please set a new one')
            continue
        else:
            print('Welcome! %s' % username)
            break

    if not username:
        tcpCliSock.close()

    recvMessage = threading.Thread(target=Recv, args=(tcpCliSock, None))
    sendMessage = threading.Thread(target=Send, args=(tcpCliSock, None))
    sendMessage.start()
    recvMessage.start()
    sendMessage.join()
    recvMessage.join()

```

这里的实现逻辑是，每当一个客户端连接进服务端，客户端需要向服务端申请一个ID，服务端将这个ID与客户端连接的Socket对象存入字典，然后监听客户端向服务端发出的消息中有没有"To:"如果有就取出至三个字符后的字符，与字典中的key去比对，如果有存在的ID(不能是当前客户端自己的ID)，就将这两个Socket对象存入一个chatwith的字典中。实现A客户端发消息给服务端，服务端从chatwith字典中查询当前客户端对应的连接，再将A客户端的消息发送给连接的B客户端。

不过这里出现了一个问题，如果客户端A在和B聊天的过程中，进来了一个客户端C。客户端C也想加入AB的聊天，但是我们的服务端将C的Socket对象覆盖掉了B的，于是B发送什么消息A都接受不到了，而A发的消息只有C能收到了。

这里通过广播(P2M)的形式解决。

## 聊天室全双工(P2M)实现：

这里稍微修改了P2P实现的服务端逻辑，不在将Socket连接一一对应，而是将所有的Socket连接存入一个列表，每当一个客户端发送消息，服务端就将这段消息广播给所有的客户端。

**Server:**

```python
# -*- coding: utf-8 -*-

import socket, select

host = socket.gethostname()
port = 8080
addr = (host, port)

inputs = []
fd_name = {}

def who_in_room(w):
    name_list = []
    for k in w:
        name_list.append(w[k])

    return name_list

def conn():
    print '...WAITING FOR CONNECTION'
    ss = socket.socket()
    ss.bind(addr)
    ss.listen(5)

    return ss

def new_coming(ss):
    client, add = ss.accept()
    print 'welcome %s %s' % (client, add)
    wel = '''''Your name(press only Enter to quit): '''
    try:
        client.send(wel)
        name = client.recv(1024)
        inputs.append(client)
        fd_name[client] = name

        nameList = "Some people in talking room, these are %s" % (who_in_room(fd_name))
        client.send(nameList)

    except Exception, e:
        print e

def server_run():
    ss = conn()
    inputs.append(ss)

    while True:
        r, w, e = select.select(inputs, [], [])
        for temp in r:
            if temp is ss:
                new_coming(ss)
            else:
                disconnect = False
                try:
                    data = temp.recv(1024)
                    data = fd_name[temp] + ' say : ' + data
                except socket.error:
                    data = fd_name[temp] + ' leave the room'
                    disconnect = True

                if disconnect:
                    inputs.remove(temp)
                    print data
                    for other in inputs:
                        if other != ss and other != temp:
                            try:
                                other.send(data)
                            except Exception, e:
                                print e
                    del fd_name[temp]

                else:
                    print data

                    for other in inputs:
                        if other != ss and other != temp:
                            try:
                                other.send(data)
                            except Exception, e:
                                print e

if __name__ == '__main__':
    server_run()  
```

**Client:**

```python
# -*- coding: utf-8 -*-

import socket, select, threading

host = socket.gethostname()

addr = (host, 8080)

def conn():
    s = socket.socket()
    s.connect(addr)
    return s

def lis(s):
    my = [s]
    while True:
        r, w, e = select.select(my, [], [])
        if s in r:
            try:
                print s.recv(1024)
            except socket.error:
                print 'socket is error'
                exit()

def talk(s):
    while True:
        try:
            info = raw_input()
        except Exception, e:
            print 'can\'t input'
            exit()
        try:
            s.send(info)
        except Exception, e:
            print e
            exit()

def main():
    ss = conn()
    t = threading.Thread(target=lis, args=(ss,))
    t.start()
    t1 = threading.Thread(target=talk, args=(ss,))
    t1.start()

if __name__ == '__main__':
    main()  
```

## 聊天室全双工(P2M)WebSocket实现：

这里又有一个奇思妙想出现了，因为在学习Socket编程的时候接触到了一个叫WebSocket的好玩的东西，于是实现了一个以浏览器为客户端的聊天室程序。使用Nodejs编写聊天室不仅代码简洁优雅功能强大，并且逼格都高很多。

> 此处以node.js + nodejs-websocket实现，首先需要安装Node.js和这个第三方模块

**Server:**
```js
var ws = require("nodejs-websocket")

var clientCount = 0

// Scream server example: "hi" -> "HI!!!"
var server = ws.createServer(function (conn) {
	console.log("New connection")
	clientCount++
	conn.nickname = 'user' + clientCount
	broadcast(conn.nickname + ' comes in')
	conn.on("text", function (str) {
		console.log("Received "+str)
		broadcast(conn.nickname + ": " + str)
	})
	conn.on("close", function (code, reason) {
		console.log("Connection closed")
		broadcast(conn.nickname + ' left')
	})
	conn.on("error", function(err) {
		console.log("handle err")
		console.log(err)
	})
}).listen(8001)

console.log("connect is close");

function broadcast(str){
	server.connections.forEach(function(connection){
		connection.sendText(str)
	})
}
```

**Client:**

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8" />
	<title>WebSocket</title>
</head>
<body>
	<h1>Chat Room</h1>
	<input id="sendTxt" type="text" />
	<button id="sendBtn">发送</button>
	<script type="text/javascript">
		var websocket = new WebSocket("ws://localhost:8001/");
		function showMessage(str) {
			var div = document.createElement('div');
			div.innerHTML = str;
			document.body.appendChild(div);
		}
		websocket.onopen = function() {
			console.log("websocket open");
			document.getElementById("sendBtn").onclick = function(){
				var txt = document.getElementById("sendTxt").value;
				if(txt) {
					websocket.send(txt);
				}
			}
		}
		websocket.onclose = function() {
			console.log("websocket close");
		}
		websocket.onmessage = function(e) {
			console.log(e.data);
			showMessage(e.data);
		}
	</script>
</body>
</html>
```