---
layout: post
title: Package Control插件管理
date: 2017-06-05 09:32:24.000000000 +09:00
tags: sublime|工具
categories: tools
---

### Package Control插件管理

提到Sublime Text插件安装，就不得不提==Package Control== ，简而言之，它是用来管理插件的插件。所以，首次使用前也是需要安装的


### 一、安装package control

#### 1、脚本安装
使用Ctrl+`（Esc键下方）快捷键或者通过View->Show Console菜单打开命令行

输入以下代码，回车。
```
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
ps：此段脚本可以在官网找到。（https://packagecontrol.io/installation）。这是package control安装的一种形式。

如果安装成功，就可以在Preferences菜单下看到Package Settings和Package Control两个菜单。

#### 2、手动安装

1. 点击Preferences->Browse Packages...菜单

![image](http://images.cnblogs.com/cnblogs_com/hykun/607566/o_2-2.jpg)

2. 进入目录后，跳转到Installed Packages目录

![image](http://images.cnblogs.com/cnblogs_com/hykun/607566/o_2-5.jpg)

3. 点击下载[Package Control.sublime-package](https://packagecontrol.io/Package%20Control.sublime-package)并复制到如下图中的Installed Packages目录

4. https://packagecontrol.io/Package%20Control.sublime-package

二、通过package control安装其它插件

1、快捷键 Ctrl+Shift+P（菜单 – Tools – Command Paletter）

![image](https://res.jianhui.org/2013/08/430773095220130807.png?imageMogr2/format/webp)

2、输入 install 选中Install Package并回车，输入或选择你需要的插件回车就安装了

![image](https://res.jianhui.org/2013/08/400496516120130807.png?imageMogr2/format/webp)

ps：如果选中Install Package后回车，未出现预期界面，有可能是因为==网络问题==，无法访问到所有的安装列表，而导致此界面出现失败。


三、Some of the possible commands are:

**Disable Package**

Temporarily turn off a package (if you think it's causing problems, for instance)

**Discover Packages**

List all available packages in your web browser

**Enable Package**

If you disabled a package & now want to re-enable it

**Install Package**

List all available packages, allowing you to narrow down results by typing; clicking on the package will install it

**List Packages**

Show all installed packages, even those manually installed

**Remove Package**

**Upgrade Package**

Upgrade a specific package

**Upgrade/Overwrite All Packages**

Run this periodically to upgrade all your installed packages

Package Control Settings – Default

Package Control Settings – User