---
title: "DNS查询"
date: 2018-11-11T00:10:14+08:00
---

作者：Windson Yang

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处（www.enginego.org）。

当你在浏览器输入 www.apple.com。按下回车之后，浏览器跳到苹果的官网，把 iPhone 的介绍和图片显示出来。浏览器是如何通过这个域名找到 iPhone 的内容并且正确显示呢？第一步就要经过 **DNS 查询**。

![域名browser](https://raw.githubusercontent.com/EngineGirl/enginegirl.github.io/markdown/images/browser.png)

#### DNS 查询

DNS 查询其实很好理解，生活上比较贴近的例子就是通讯录，我们通常能够记住自己的电话号码，不过要找其他人的电话，一般都需要通过通讯录。我们输入朋友的名字，通讯录就会找到对应的电话号码。在这里，通讯录担任了把难记的电话号码转成好记的人名的工作。DNS查询其实也一样，负责把难记的 IP 地址转成容易记忆的域名。

![dns_convert](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/dns/dns_convert.png)

互联网刚开始发展的时候，联网的电脑加起来才几千台（没错～）。每台电脑都需要保存一个文件（称为 **hosts 文件**）用作记录域名对应哪个IP地址（就像通讯录记录人名对应哪个电话号码）。

    www.enginego.org 104.24.120.11（每一行分别对应着域名和 IP 地址）
    www.apple.com 119.145.144.223
    www.ieee.org  23.38.177.118

所以早期的时候，当我们在浏览器输入域名的时候，浏览器会先查询 hosts 文件，找到该域名对应的 IP 地址。然后再通过 IP 地址获取到服务器里面的内容并且显示在浏览器中。不过问题随着互联网的发展慢慢显现了：

1. 每个小时都有新的域名被注册，新的主机加入互联网，现在已经超过 10亿 台设备接入互联网，如果每次查询的话都要从文件中找到对应的 IP 地址，那么会非常耗时，而且 hosts 文件也会变得非常大。
2. 大部分的网站用户根本不会浏览，每上线一个小网站就要求全球的电脑都更新 hosts 文件这样显然小题大做了。


为了解决这几个问题，现代的 DNS 查询会按顺序经过 3个 步骤，一旦查询到结果，就会跳过剩下的步骤：

1. 先查询浏览器有没有保留缓存

    如果你之前访问过这个网站，那么浏览器会保存网站对应的 IP 地址，这样下一次访问就能直接从缓存中取出地址，减少查询 IP 地址的时间。

![browser_cache](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/dns/browser_cache.png)

2. 查询本地的hosts文件

    如果hosts文件有对应域名包含的 IP 地址，例如

        www.apple.com 119.145.144.223

    那么计算机就会直接使用这个 IP 地址，你可以查看自己电脑的 hosts 文件，不同系统分别存储在不同的位置

        Windows
        c:\windows\system32\drivers\etc\hosts

        macOS
        /etc/hosts

![hosts_file](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/dns/root.png)

3. DNS 服务器查询

    当计算机在前两个方法都没有找到对应的IP地址，就会进行DNS服务器查询，**它会询问最近的子域名服务器该域名对应的 IP 地址**（默认是 ISP 运营商提供，在国内就是中国电信，中国联通，中国移动这几家），**如果最近的子域名服务器也不知道的话会一级级向上查询，直到查询到根域名服务器。**  （根域名服务器保存着所有域名与 IP 地址的对应关系，子域名服务器也会定时从上级服务器中获取最新的内容。）
    
经过这 3步，我们只需要记住某个网站的域名，就能获取到该网站的IP地址了。

#### 使用域名的优点

1. 容易记忆

    计算机比较擅长处理数字，但是人类就差多了，访问苹果官网要背那么多数字显然不现实。使用域名就像我们使用手机的通讯录一样，我们会为常用的电话号码添加联系人姓名，打电话的时候直接输入名字就可以拨打，不需要背电话号码。

2. 容易扩展

    早期的计算机和IP地址是一一对应的关系，但是一台计算机的性能并不能承受大量用户的请求。而一个域名可以背后可以对应多个 IP 地址。通过给不同地区的用户分配不同的 IP 地址，公司就能使用多台服务器来提供服务了。
      
3. 层级关系
    
    从域名上我们可以判断 apple.com 与 edu.apple.com 都是属于 apple 公司的，这种子域名的层级关系也能有效提高用户的安全性。
