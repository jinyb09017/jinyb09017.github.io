---
layout: post
title: ShadowSocks全局上网
date: 2017-12-02 14:32:24.000000000 +09:00
categories: network
tags: 翻墙-科学上网-终端上网
---

如果用过ShadowSocks的朋友，可能会慢慢意识到，ShadowSocks开启后，在浏览器上好使，但是我们在使用terminal终端的时候，发现一些wget、curl、git、brew等命令行工具都会变得很慢。。 这是为什么呢，可能大家像我一样困惑，那么下面我们来简单介绍下，出现这种问题的原因，以及该如何去做呢？

首先我们先了解一些ShadowSocks的基本工作原理。用一张图简单来表示下。
![ShadowSocks的基本工作原理](http://upload-images.jianshu.io/upload_images/2159256-bd93ae4bb4bdac61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


简单来说，ShadowSocks分为客户端和服务端，用户发出的请求基于Socks5协议与ShadowSocks客户端进行通信，一般情况下SS客户端都在本机，通过ShadowSocksX、GoAgentX等应用启动，所以这一步是不会经过GFW的，然后ShadowSocks提供了多种加密方式供客户端和服务端之间进行通信，并且在经过GFW时是普通的TCP协议数据包，没有明显的特征，而且GFW也无法解密分析，从而实现绕墙访问资源。

由于是基于socket5协议，我也就导致了使用ShadowSocks代理的局限性。即只有支持socket协议的软件才能正常翻墙使用。而我们在使用terminal终端进行网络连接的时候，都是基于http协议，我就导致了我们不能正常的代理上网。

可能大家会疑问，浏览器本身不也是使用http协议上网的，为什么就能通过ShadowSocks进行翻墙上网呢，这是因为chrome浏览器本身是支持从http转换为socket协议的。（并非所有的浏览器都支持socket协议的）

ps：为什么vpn就可以直接全局上网，这是因为vpn本身是经过一道加密的通道进行上网，所以开启vpn后就是全局的走vpn通道了。

所以为了使不支持socket协议的应用也能够使用ShadowSocks代理上网，所以我们需要使用工具进行一层转换。其中最经典的就是使用proxifier了。关于proxifier的工作原理，我们也用一个图做一个简单介绍

![proxifier代理转换](http://upload-images.jianshu.io/upload_images/2159256-124cd6c75eaea4df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Proxifier的客户端[下载地址](https://www.proxifier.com/download.htm)

操作步骤如下
### 一、添加socket代理

选择proxies-->add

![image.png](http://upload-images.jianshu.io/upload_images/2159256-ed81b4b10e481d66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 二、设置rulues

![image.png](http://upload-images.jianshu.io/upload_images/2159256-2732c6e6e1e66d05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在defalut对应的栏目中，设置action为我们设置的代理即可。这样，表示默认所有请求都走这个代理。

其它配置，请大家根据需要自己配置即可。










 
