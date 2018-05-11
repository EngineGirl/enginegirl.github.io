---
title: "Ping命令"
date: 2018-04-25T00:10:14+08:00
---

作者：Windson Yang

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处 (www.enginego.org)。

- [基础版](#基础版)
- [进阶版](#进阶版)

### 基础版

#### ping命令
有时候当我们无法上网，会计算机的朋友会说，你ping一下网关，或者你ping一个网站看看。ping这个命令，其实是操作系统自带的命令之一，常用作在网络诊断中。你可以在[终端](../终端)中输入：

    ping www.enginego.org

你可以把"www.enginego.org"替换成其他任意的域名，终端会显示：

    PING www.enginego.org (104.24.120.11): 56 data bytes
    64 bytes from 104.24.121.11: icmp_seq=0 ttl=54 time=170.383 ms
    64 bytes from 104.24.121.11: icmp_seq=1 ttl=54 time=170.053 ms
    64 bytes from 104.24.121.11: icmp_seq=2 ttl=54 time=171.298 ms
    64 bytes from 104.24.121.11: icmp_seq=3 ttl=54 time=170.352 ms
    64 bytes from 104.24.121.11: icmp_seq=4 ttl=54 time=170.636 ms
    ...

这表示从我的计算机发送64个字节的数据到104.24.121.11 (这个是经过[DNS查询](../dns查询/)后的www.enginego.org的ip地址)，从发送到接受对方返回总共经过了170.xxx毫秒。代表www.enginego.org对应的那台服务器是开启并且响应ping指令的。

![ping_gif](https://raw.githubusercontent.com/EngineGirl/enginegirl.github.io/master/images/ping/ping.gif)

如果返回：

    PING www.enginego.org (104.24.121.11): 56 data bytes 
    Request timeout for icmp_seq 0
    Request timeout for icmp_seq 1
    Request timeout for icmp_seq 2
    Request timeout for icmp_seq 3

这就代表连接超时，访问失败。原因有可能是本地计算机网络问题，也有可能是对应的服务器关闭或者不响应ping指令，或者你根本只是输错域名了。

#### 网络诊断
当我们要连接互联网的时候，无论使用手机还是计算机，一般都要经过网关 (路由器或者交换机)，网关接受到数据包的时候经过NAT转换再连接到互联网。当无法上网的时候，可以通过以下两个方法排查故障。

1. 检查计算机到路由器的连接是否正常 
    
           ping 192.168.1.1 (路由器/交换机地址)

       超时则代表：
       
       - 网线没有连接好或者
       - IP地址冲突

2. 然后检查路由器到互联网的连接是否正常

           ping www.enginego.org (网站地址)

       超时则代表：
            
       - 路由器设置出错
       - DNS解析出错
       - 服务器关闭或者不响应ping指令

**我们可以从这两步判断网络到底哪里出现了问题。在本地网络没有问题的前提下，使用ping命令可以判断目标主机是否开启及其响应速度。**

### 进阶版

#### ping指令实际是如何实现的

ping指令使用的是**ICMP协议**，就像我们每天在用的HTTP协议，客户端（我们的ping程序）根据约定好的[协议](../协议/)发送二进制数据到服务器，然后服务器根据协议规则解析二进制数据中的设置与数据。 数据里面会指定使用第几个版本，发送请求的IP地址，接受请求的地址等等，如果大家对wireshark有了解，可以尝试打开wireshark之后再执行ping指令，然后查找ICMP协议对应的数据传输。

当我使用计算机运行（这里出于方便显示把传输的数据转换成十六进制）

    ping www.enginego.org
  
实际发送的数据是：

    0000   b0 7f b9 a3 68 36 98 e0 d9 9d a3 8f 08 00 45 00
    0010   00 54 69 21 00 00 40 01 6f 64 c0 a8 01 58 68 18
    0020   78 0b 08 00 47 00 62 90 00 03 5a a1 2a fa 00 06
    0030   dd c7 08 09 0a 0b 0c 0d 0e 0f 10 11 12 13 14 15
    0040   16 17 18 19 1a 1b 1c 1d 1e 1f 20 21 22 23 24 25
    0050   26 27 28 29 2a 2b 2c 2d 2e 2f 30 31 32 33 34 35
    0060   36 37

![ping_wireshark](https://raw.githubusercontent.com/EngineGirl/enginegirl.github.io/master/images/ping/ping_wireshark.png)

可以看到数据中包含了发送的类别，校验和，发送程序的进程编号（上图中的Identification），IP地址等等，例如**c0 a8 01 58对应着10进制的192 168 1 88，也就是我的计算机的IP地址。**注意ping指令是不需要指定端口的，它是根据协议头部信息纪录的进程编号来辨识返回的数据的。
