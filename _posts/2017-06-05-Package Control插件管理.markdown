---
layout: post
title: Package Control插件管理
date: 2017-06-05 09:32:24.000000000 +09:00
tags: sublime 工具
categories: tools
---

# 前言

曾几何时，可以自由在谷歌里自由玩耍，后来，gw让一切变得不那么容易。慢慢地，我们开始接受百度，可是，百度越来越不像当初的百度了，而谷歌却越来让你觉得惊奇。尤其做为一个程序员，没有谷歌的世界是寂寞的。
而在如何跨过这道墙，更好的看看外面的世界。你又做过哪些努力呢？
1. 找动态代理...
2. 修改hosts重定位ip(可能一段时间就不能用了)...
3. goAgent代理（貌似已经不如当初了）...
4. 免费vpn帐号（速度太慢，不够稳定）...
5. 花钱在vpn网站买服务（别忘记zf建gw的目的）...
6. 其他折腾

最后，你会发现，以上的种种尝试都不是一个长期的解决方案，或许在你迫切需要的时候，它已经over了，这时候有没有一种要抓狂的感觉。尤其像一些软件需要翻墙更新的时候，正所谓屋漏偏逢连夜雨。那作为一个程序员，是否有一个更好的解决方案。**答案是肯定的，就那是搭建自己的代理服务**

## 翻墙原理
在科学上网之前，总得搞清楚代理上网的基本原理，这样在搭建代理服务时候，才会如鱼得水般容易。
#### 防火墙工作原理
用一张可以简单理解如下

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2159256-435930deecaff5bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- **DNS缓存污染**：DNS通常使用 UDP，GFW对捕获的DNS查询报文也进行关键词过滤并返回伪DNS响应。被污染的域名往往被解析特定的几个ip。
- **限制特定Ip访问**：通过修改路由器静态路由把特定ip的数据包发送到“黑洞服务器”，这样用户就无法和这个ip真正的服务器有数据往来。
- **关键字过滤嗅探**：嗅探到敏感内容后，会向链接两端的服务器发送RST包，导致无法建立连接。
（内容比较专业，不关心的可以略过下面的原理分析）

#### 突破GFW的方法
- DNS缓存污染—->修改host
- 封锁ip—->使用代理
- 内容审查—->加密传输

## 那我们如何科学上网
1. **ShadowSocks翻墙**
2. **搭建vpn服务器**


## 1、**ShadowSocks翻墙**
**一、ShadowSocks翻墙原理**

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2159256-bd93ae4bb4bdac61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1. PC客户端（即你的电脑）发出请求基于Socks5协议跟SS-Local端进行通讯，由于这个SS-Local一般是本机或路由器等局域网的其他机器，不经过GFW，所以解决GFW通过特征分析进行干扰的问题。
2. SS-Local和SS-Server两端通过多种可选的加密方法进行通讯，经过GFW的时候因为是常规的TCP包，没有明显特征码GFW也无法对通讯数据进行解密，因此通讯放行。
3. SS-Server将收到的加密数据进行解密，还原初始请求，再发送到用户需要访问的服务网站，获取响应原路再返回SS-04，返回途中依然使用了加密，使得流量是普通TCP包，并成功穿过GFW防火墙

**二、如何搭建ShadowSocks服务端**
1. 购买[vps](http://baike.baidu.com/item/VPS)
2. 在VPS上搭建ShadowSocks
3. 在本机安装ShadowSocks客户端

## 2、**vpn翻墙**

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2159256-49d49397f0cc0854.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
VPN代理，是在VPN（Virtual Private Network）[虚拟专用网络](http://baike.baidu.com/view/480950.htm)的基础上衍生出来的提高网络访问速度和安全的技术。它利用VPN的特殊加密通讯协议在因特网位于不同地方的两个结点间临时建立一条穿过混乱公用网络的安全稳定的专用隧道。

**二、如何搭建vpn服务**
1. 购买[vps](http://baike.baidu.com/item/VPS)
2. 在VPS上搭建vpn服务
3. 在本机创建vpn连接

## 如何操作
上述两种方案，归根结底都需要购买[vps](http://baike.baidu.com/item/VPS)，所以也说明，天底下毕竟没有免费的午餐，有也不会好吃到哪里。
目前vps的选择比较多，但毕竟我们只是偶尔有需要，看看外面的世界，所以也不想承担太贵的服务。所以目前性价比比较高的vps可以选择搬瓦工，但貌似现在资费比较低的服务也已经卖完了。国外vps的购买付款都比较麻烦，目前搬瓦工支持支付宝付款，确实帮我们省了不少事情。做为一个程序员，买个服务器自己倒腾，也是有必要的。

[搬瓦工VPS](https://bandwagonhost.com/aff.php?aff=8759) 官方网站：https://bandwagonhost.com/aff.php?aff=8759。


### 一、购买vps服务
首先填写相关资料注册，如果只是作为翻墙需要，基本上可以先买最便宜的vps服务就行了，如果还想倒腾服务器空间，发布网站服务的话，可以根据需要购买。

选择vps Hosting栏目，出现如下
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2159256-80099c4b5d75a15d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
购买相应的vps服务后，付款即可。

### 二、搭建ShadowSocks服务端
**购买服务后，进入我的服务面板**
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2159256-250275af54cc9066.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2159256-417bef27e33b3556.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**点击右边的**KiwiVM Control Panel**面板，你可以看到VPS的所有信息如下。**

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2159256-7774f424ba65ad15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 **一键ShadowSocks服务端**
在**KiwiVM Control Panel**面板，在**KiwiVM Extras**下面，可以点击**ShadowSocks Server**一键创建好ShadowSocks Server服务器。然后根据面板里面的说明和相关帐号信息，在本地安装好ShadowSocks 客户端，即可科学上网，无所拘束了。
ps:如果有兴趣的话，可以自己搭建ShadowSocks Server.
### 三、搭建vpn服务
可以直接参考这篇文章[搬瓦工VPS架设VPN教程](http://blog.csdn.net/hjhjw1991/article/details/45848565)，再简单不过了。
ps:这样也可以创建**n多子帐号**贡献给其它朋友们使用，就像小编自己一样。


### 四、总结
防火墙帮我们屏蔽了一些不健康的内容，但是也“不小心”地档住了一片蓝天，希望大家可以通过翻墙可以健康安全地上网，最后衷心祝愿大家上网愉快。

如果大家在搭建过程中遇到什么问题，可以直接找我咨询，希望能够帮到大家
下面为个人联系微信帐号，如果有临时代理上网的需要，或者**需要vpn账号**的也可以找我帮帮忙 :）

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2159256-7e67fa864047647a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





 
