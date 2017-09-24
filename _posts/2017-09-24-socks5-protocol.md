---
title: 计算机网络(一)  走近socks5
description: socks是一种网络传输协议，主要用于客户端与外网服务器之间通讯的中间传递。根据OSI七层模型来划分，SOCKS属于会话层协议，位于表示层与传输层之间。
categories:
 - network
tags: socks5
---

<div  align="center">    
 <img src="http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/3306c8cab2d1cbe2faa5e01aafe1f73e.png" width = "868" height = "410" alt="图片名称" align=center />
</div>


最近项目中涉及到socket5协议，趁此机会补一下这一块的空缺。

## 1. 什么是socks5
或许你没听说过socks5，但你一定听说过ShadowSocks，ShadowSockS内部使用的正是socks5协议。

socks是"SocketS"的缩写，因此socks5也叫sockets5。

RFC地址：

- [rfc1928](https://www.ietf.org/rfc/rfc1928.txt)
- [rfc1929](https://www.ietf.org/rfc/rfc1929.txt)

socks是一种网络传输协议，主要用于客户端与外网服务器之间通讯的中间传递。根据OSI七层模型来划分，SOCKS属于会话层协议，位于表示层与传输层之间。

当防火墙后的客户端要访问外部的服务器时，就跟socks代理服务器连接。该协议设计之初是为了让有权限的用户可以穿过过防火墙的限制，使得高权限用户可以访问外部资源。经过10余年的时间，大量的网络应用程序都支持socks5代理。 

这个协议最初由David Koblas开发，而后由NEC的Ying-Da Lee将其扩展到版本4，最新协议是版本5，与前一版本相比，socks5做了以下增强：

- 增加对UDP协议的支持；
- 支持多种用户身份验证方式和通信加密方式；
- 修改了socks服务器进行域名解析的方法，使其更加优雅；

## 2. socks5使用场景
socks协议的设计初衷是在保证网络隔离的情况下，提高部分人员的网络访问权限，但是国内似乎很少有组织机构这样使用。一般情况下，大家都会使用更新的网络安全技术来达到相同的目的。

但是由于socksCap32和PSD这类软件，人们找到了socks协议新的用途：突破网络通信限制，这和该协议的设计初衷正好相反。

下面是两个典型的运用场景：

- 美国某网游的服务器仅允许本国的IP进行连接。非美国玩家为了突破这种限制，可以找一个该地区的socks5代理服务器，然后用PSD接管网游客户端，通过socks5代理服务器连接游戏服务器。这样游戏服务器就会认为该玩家的客户端位于本地区，从而允许该玩家进行游戏（在天朝也叫科学\*\*，属于正向代理）。

![image.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/7e520a634b13ecab4fc9aeaf99e42b66.png)
- 某服务器的防火墙仅允许部分端口（如http的80端口）通信，那么可以利用socks5协议和一个打开80端口监听的socks5服务器连接，从而可以连接公网上其他端口的服务器。利用一些额外的技术手段，甚至可以骗过内部的http代理服务器，这时在使用内网http代理上网的环境下也可以不受限制的使用网络服务，这称之为socks over HTTP（我们常说的穿墙）。


- 内网穿透：在大学里，学校给我们提供了很多服务器资源，我们可以在内网使用。但放寒假回家后，无法进入学校内网，也就无法连接上内网的服务器资源。解决办法：在公网的VPS上搭一个socks代理，并将内网的一台web服务器和该VPS的socks端口打通，通过这台web服务器便可以访问所有内网服务器资源（常见的花生壳nat穿透和这个类似）。
  ![image.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/29f285b07e53f661e4d2ed21727e7f49.png)

当然，使用代理服务器后，将不可避免的出现通信延迟，所以应该尽量选择同网络（通运营商）、距离近的服务器。

## 3. 与HTTP代理的对比

socks支持多种用户身份验证方式和通信加密方式。

socks工作在比HTTP代理更低的网络层：socks使用握手协议来通知代理软件其客户端试图进行的连接socks，然后尽可能透明地进行操作，而常规代理可能会解释和重写报头（例如，使用另一种底层协议，例如FTP；然而，HTTP代理只是将HTTP请求转发到所需的HTTP服务器）。

socks5代理支持转发UDP报文，而HTTP属于tcp协议，不支持UDP报文的转发。

虽然HTTP代理有不同的使用模式，CONNECT方法允许转发TCP连接；然而，socks代理还可以转发UDP流量和反向代理，而HTTP代理不能。HTTP代理更适合HTTP协议，执行更高层次的过滤；socks不管应用层是什么协议，只要是传输层是TCP/UDP协议就可以代理。

## 4. socks5协议详解

### 4.1 socks5认证协议

![image.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/49da7e7a09b5f92bbc954d63d5b929be.png)
在客户端、服务端协商好使用用户名密码认证后，客户端发出用户名密码，格式为：

![image.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/a2910a58561ec2d3b1c715c3858015a8.png)

- VER：鉴定协议版本
- ULEN：用户名长度
- UNAME：用户名
- PLEN：密码长度
- PASSWD：密码

服务器鉴定后发出如下回应：

![image.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/6fbc952d1edbea80f3620a92acdb3406.png)

- VER：鉴定协议版本
- STATUS：鉴定状态

其中鉴定状态 0x00 表示成功，0x01 表示失败。

### 4.2 socks5传输协议

创建与socks5服务器的TCP连接后，客户端需要先发送请求来协商版本及认证方式，格式为：
![image.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/a5e379e63dbf96c07f59ce2a237d8aed.png)

- VER：socks版本（在socks5中是0x05）；
- NMETHODS：在METHODS字段中出现的方法的数目；
- METHODS：客户端支持的认证方式列表，每个方法占1字节。

服务器从客户端提供的方法中选择一个最优的方法并通过以下消息通知客户端（贪心算法：双方都支持、安全性最高）：

![image.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/0a63fea1e27472867ac8db56c9cf9159.png)

- VER：socks版本（在socks5中是0x05）；
- METHOD：服务端选中的方法（若返回0xFF表示没有方法被选中，客户端需要关闭连接）；


METHOD字段的值可以取如下值：

-  X'00' NO AUTHENTICATION REQUIRED
-  X'01' GSSAPI
-  X'02' USERNAME/PASSWORD
-  X'03' to X'7F' IANA ASSIGNED
-  X'80' to X'FE' RESERVED FOR PRIVATE METHODS
-  X'FF' NO ACCEPTABLE METHODS

之后客户端和服务端根据选定的认证方式执行对应的认证。认证结束后客户端就可以发送请求信息（如果认证方法有特殊封装要求，请求必须按照方法所定义的方式进行封装）。

socks5请求格式：

![image.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/e8cf60f2adbf1ce8b3584359221552bf.png)

- VER：socks版本（在socks5中是0x05）
- CMD：SOCK的命令码：
  - CONNECT X'01'
  - BIND X'02'
  - UDP ASSOCIATE X'03'


- RSV：保留字段
- ATYP：地址类型：
  - IP V4地址: X'01'
  - 域名地址: X'03'
  - IP V6地址: X'04'
- DST.ADDR：目的地址

- DST.PORT：目的端口

服务器按以下格式回应客户端的请求：

![image.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/67e6d22304ce092659d4fb5d2f158004.png)

- VER：socks版本（在socks5中是0x05）
- REP：应答状态码：
  - X'00' succeeded
  - X'01' general socks server failure
  - X'02' connection not allowed by ruleset
  - X'03' Network unreachable
  - X'04' Host unreachable
  - X'05' Connection refused
  - X'06' TTL expired
  - X'07' Command not supported
  - X'08' Address type not supported
  - X'09' to X'FF' unassigned


- RSV：保留字段（需设置为X'00'）
- ATYP：地址类型：
  - IP V4 address: X'01'
  - DOMAINNAME: X'03'
  - IP V6 address: X'04'


- BND.ADDR：服务器绑定的地址
- BND.PORT：服务器绑定的端口

如果被选中的方法包括有认证信息的封装、完整性和/或机密性相关检查，则server端在发送响应包时也需要把这些响应消息封装进去。


## 5. ​SOCKS相关工具

### 5.1 SOCKS服务器

部分SOCKS服务器软件：

- Dante Socks Server，<http://www.inet.no/dante>
- Java Socks Server，[http://jsocks.sourceforge.net](http://jsocks.sourceforge.net/)
- Socks4 Server，<https://archive.is/20130502024508/>
- SS5 Socks Server，[http://ss5.sourceforge.net](http://ss5.sourceforge.net/)
- TcpToute2，<https://github.com/GameXG/TcpRoute2>

### 5.2 SOCKS客户端

一般情况下应用程序会内嵌对SOCKS协议的支持。

| 客户端               | 许可证                                      | 版本     | 发布日期    | 平台                                       | 支持协议   |
| ----------------- | ---------------------------------------- | ------ | ------- | ---------------------------------------- | ------ |
| Dante client      | [BSD/CMU](https://zh.wikipedia.org/wiki/BSD%E8%AE%B8%E5%8F%AF%E8%AF%81) | 1.1.18 | 09/2005 | Linux                                    | v4, v5 |
| FreeCap           | [GPL](https://zh.wikipedia.org/wiki/GPL) | 3.18   | 02/2005 | Windows                                  | -      |
| Hummingbird socks | -                                        | -      | -       | Windows                                  | -      |
| ProxyCap          | -                                        | 2.03   | -       | Windows                                  | -      |
| SocksCap          | Non-Comercial home use                   | -      | -       | -                                        | v5     |
| Super Socks5Cap   | -                                        | 1.5.3  | -       | Windows                                  | -      |
| tsocks            | GPL                                      | 1.8    | 10/2002 | -                                        | -      |
| nylon             | -                                        | -      | 06/2003 | [OpenBSD](https://zh.wikipedia.org/wiki/OpenBSD) | -      |


## 6. python实现socks5代理
为了方便，我们直接使用python中的`SocketServer`库，直接运行以下程序即可在本机建立了一个socks5的代理服务器：

```python
import socket, sys, select, SocketServer, struct, time


class ThreadingTCPServer(SocketServer.ThreadingMixIn, SocketServer.TCPServer): pass


def handle_tcp(sock, remote):
    fdset = [sock, remote]
    while True:
        r, w, e = select.select(fdset, [], [])
        if sock in r:
            if remote.send(sock.recv(4096)) <= 0: break
        if remote in r:
            if sock.send(remote.recv(4096)) <= 0: break


class Socks5Server(SocketServer.StreamRequestHandler):

    def handle(self):
        try:
            print('socks connection from ', self.client_address)
            sock = self.connection
            # 1. Version
            sock.recv(262)
            sock.send(b"\x05\x00");
            # 2. Request
            data = self.rfile.read(4)
            mode = ord(data[1])
            addrtype = ord(data[3])
            if addrtype == 1:  # IPv4
                addr = socket.inet_ntoa(self.rfile.read(4))
            elif addrtype == 3:  # Domain name
                addr = self.rfile.read(ord(sock.recv(1)[0]))
            port = struct.unpack('>H', self.rfile.read(2))
            reply = b"\x05\x00\x00\x01"
            try:
                if mode == 1:  # 1. Tcp connect
                    remote = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                    remote.connect((addr, port[0]))
                    print('Tcp connect to', addr, port[0])
                else:
                    reply = b"\x05\x07\x00\x01"  # Command not supported
                local = remote.getsockname()
                reply += socket.inet_aton(local[0]) + struct.pack(">H", local[1])
            except socket.error:
                # Connection refused
                reply = '\x05\x05\x00\x01\x00\x00\x00\x00\x00\x00'
            sock.send(reply)
            # 3. Transfering
            if reply[1] == '\x00':  # Success
                if mode == 1:  # 1. Tcp connect
                    handle_tcp(sock, remote)
        except socket.error:
            print('socket error')


def main():
    server = ThreadingTCPServer(('', 1080), Socks5Server)
    server.serve_forever()


if __name__ == '__main__':
    main()
```

客户端实现代码：

```python
import socket, socks, requests


def main():
    socks.set_default_proxy(socks.SOCKS5, "127.0.0.1", 1080)
    socket.socket = socks.socksocket
    print(requests.get('http://127.0.0.1:1080').text)


if __name__ == '__main__':
    main()
```
## 7. 参考文献

- [rfc1929](https://www.ietf.org/rfc/rfc1929.txt), [rfc1928](https://www.ietf.org/rfc/rfc1928.txt)
- [HTTP协议和SOCKS5协议](http://www.cnblogs.com/yinzhengjie/p/7357860.html)
- [socket4和socket5的区别](http://blog.csdn.net/cuiyifang/article/details/8784394)
- [socket5 协议学习与实现(一)](http://www.mojidong.com/network/2015/03/07/socket5-1/)
- [Socks代理和http代理的区别](https://wrfly.kfd.me/SOCKS%E4%BB%A3%E7%90%86%E5%92%8CHTTP%E4%BB%A3%E7%90%86%E7%9A%84%E5%8C%BA%E5%88%AB/)