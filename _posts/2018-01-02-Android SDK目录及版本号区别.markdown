---
layout: post
title: Android SDK目录及版本号区别
date: 2018-01-02 14:32:24.000000000 +09:00
categories: android
tags: sdk-android-工具介绍
---
# SDK目录

## add-ons
这里面保存着附加库，第三方公司为android 平台开发的附加功能系统。比如GoogleMaps，当然你如果安装了OphoneSDK，这里也会有一些类库在里面。

## docs
这里面是Android SDKAPI参考文档，所有的API都可以在这里查到。

## extras

该文件夹下存放了==Android support v4，v7，v13，v17包==；
还有google提供额USB驱动、Intel提供的硬件加速等附加工具包，
和market_licensing作为AndroidMarket版权保护组件，一般发布付费应用到电子市场可以用它来反盗版。

## platforms

是每个平台的SDK真正的文件，存放了不同版本的android系统。里面会根据APILevel划分的SDK版本，这里就以Android2.2来说，进入后有 一个android-8的文件夹，android-8进入后是Android2.2SDK的主要文件，其中ant为ant编译脚本，data保存着一些系 统资源，images是模拟器映像文件，skins则是Android模拟器的皮肤，templates是工程创建的默认模板，**android.jar则 是该版本的主要framework文件**

## samples
是Android SDK自带的默认示例工程，里面的apidemos强烈推荐初学者运行学 习，对于SQLite数据库操作可以查看NotePad这个例子，对于游戏开发Snake、LunarLander都是不错的例子，对于Android主 题开发Home则是androidm5时代的主题设计原理。


# 下面重点介绍这3个！！！！
## platform-tools
保存着一些Android平台相关通用工具，比如adb、和aapt、aidl、dx等文件，这里和platforms目录中tools文件夹有些重复，主要是从android2.3开始这些工具被划分为通用了。Fastboot 刷机工具。

## tools
作为SDK根目录下的tools文件夹，这里包含了android **开发和调试的工具**，比如ddms用于启动Android调试工具，比如logcat、屏幕截图和文件管理器，而draw9patch则是绘制android平台的可缩放png图片的工具，sqlite3可以在PC上操作SQLite数据库， 而monkeyrunner则是一个不错的压力测试应用，模拟用户随机按键，mksdcard则是模拟器SD映像的创建工具，emulator是 Android SDK模拟器主程序，不过从android 1.5开始，需要输入合适的参数才能启动模拟器，traceview作为android平台上重要的调试工具。

## build-tools

**保存着一些Android平台相关通用工具，比如adb、和aapt、aidl、dx等文件。**

**aapt**即Android Asset Packaging Tool , 在SDK的build-tools目录下. 该工具可以查看, 创建, 更新ZIP格式的文档附件(zip, jar, apk). 也可将资源文件编译成二进制文件.

**Adb** 即android debug bridge 管理模拟器和真机的万能工具，ddms 调试环境

**AIDL** 即 Android Interface definition language
它是一种android内部进程通信接口的描述语言，通过它我们可以定义进程间的通信接口
Emulator即android 的模拟器

**dx**：转化.class中间代码为dvlik中间代码,所有经过java编译的生成.class文件都需要此工具进行转换,最后打包进apk文件中.

**Dexdump** 即Android Emulator中可以找到一个名为dexdump的程序，通过dexdump可以查看出apk文件中的dex执行情况，粗略分析出原始java代码是什 么样的和Dot Net中的Reflector很像。

注意：这里会涉及到一个问题，就是build-tools后边会有不同的api版本号！
①buildeToolVersion是你构建工具的版本，这个版本号一般是API-LEVEL.0.0。 例如I/O2014大会上发布了API20对应的build-tool的版本就是20.0.0，在这之间可能有小版本，例如20.0.1等等。

②在ecplise的project.properties中可以设置sdk.buildtools=20.0.0。也可以不设置，不设置的话就是指定最新版本。而在android studio中是必须在build.gradle中设置。

③Android都是向下兼容的，你可以用高版本的**build-tool去构建一个低版本的sdk工程**，例如build-tool的版本为20，去构建一个sdk版本为18的工程！


### min、compile、target版本的区别
compileSdkVersion, minSdkVersion 和 targetSdkVersion 的作用：他们分别控制可以使用哪些 API ，要求的 API 级别是什么，以及应用的兼容模式。

### compileSdkVersion

compileSdkVersion 告诉 Gradle 用哪个 Android SDK 版本编译你的应用。使用任何新添加的 API 就需要使用对应等级的 Android SDK。

==需要强调的是修改 compileSdkVersion 不会改变运行时的行为==。当你修改了 compileSdkVersion 的时候，可能会出现新的编译警告、编译错误，但新的 compileSdkVersion 不会被包含到 APK 中：**它纯粹只是在编译的时候使用**。（你真的应该修复这些警告，他们的出现一定是有原因的！）

因此我们强烈推荐你总是使用最新的 SDK 进行编译。在现有代码上使用新的编译检查可以获得很多好处，避免新弃用的 API ，并且为使用新的 API 做好准备。

**注意**，如果使用 Support Library ，那么使用最新发布的 Support Library 就需要使用最新的 SDK 编译。例如，要使用 23.1.1 版本的 Support Library，compileSdkVersion 就必需至少是 23 （大版本号要一致！）。通常，新版的 Support Library 随着新的系统版本而发布，它为系统新增加的 API 和新特性提供兼容性支持。

### minSdkVersion
如果 compileSdkVersion 设置为可用的最新 API，那么 minSdkVersion 则是应用可以运行的最低要求。minSdkVersion 是 **Google Play 商店用来判断用户设备是否可以安装某个应用的标志之一（还有应用和版本号）**。

在开发时 minSdkVersion 也起到一个重要角色：lint 默认会在项目中运行，它在你使用了高于 minSdkVersion 的 API 时会警告你，帮你避免调用不存在的 API 的运行时问题。如果只在较高版本的系统上才使用某些 API，通常使用“运行时检查系统版本”的方式解决。

**请记住**，你所使用的库，如 Support Library 或 Google Play services，可能有他们自己的 minSdkVersion 。你的应用设置的 minSdkVersion 必须大于等于这些库的 minSdkVersion 。例如有三个库，它们的 minSdkVersion 分别是 4, 7 和 9 ，那么你的 minSdkVersion 必需至少是 9 才能使用它们。在少数情况下，你仍然想用一个比你应用的 minSdkVersion 还高的库（处理所有的边缘情况，确保它只在较新的平台上使用），你可以使用 tools:overrideLibrary 标记，但请做彻底的测试！

### targetSdkVersion

三个版本号中最有趣的就是 targetSdkVersion 了。 targetSdkVersion 是 Android 提供向前兼容的主要依据，**在应用的 targetSdkVersion 没有更新之前系统不会应用最新的行为变化**。这允许你在适应新的行为变化之前就可以使用新的 API （因为你已经更新了 compileSdkVersion 不是吗？）。
targetSdkVersion 所暗示的许多行为变化都记录在 VERSION_CODES 文档中了，但是所有恐怖的细节也都列在每次发布的平台亮点中了，在这个 API Level 表中可以方便地找到相应的链接。

如果描述不太清楚，[查考文章](http://www.race604.com/android-targetsdkversion/)即可明白了。

### 综合来看

如果你按照上面示例那样配置，你会发现这三个值的关系是：

```
minSdkVersion <= targetSdkVersion <= compileSdkVersion
```
这种直觉是合理的，如果 compileSdkVersion 是你的最大值，minSdkVersion 是最小值，那么最大值必需至少和最小值一样大且 target 必需在二者之间。
理想上，在稳定状态下三者的关系应该更像这样：


```
minSdkVersion (lowest possible) <= targetSdkVersion == compileSdkVersion (latest SDK)
```
**用较低的 minSdkVersion 来覆盖最大的人群，用最新的 SDK 设置 target 和 compile 来获得最好的外观和行为。**


### buildtoolsVersion vs compileSdkVersion

compileSdkVersion is the API version of Android that you compile against.

buildToolsVersion is the version of the compilers (**aapt, dx, renderscript** compiler, etc...) that you want to use. For each API level (starting with 18), **there is a matching .0.0 version**.

简而言之，buildToolsVersion是负责把指定用哪一组编译程序来将你所编写的 Source Code 转成操作系统可以载入的形式，也就是产出 Apk。这并不会影响到应用程序的运行。不同的buildToolsVersion版本，原则上不会有太大的差异，大概就是**编译的效能、产出的文件大小等等的细节**。


但有时候还是可能会牵涉到 Apk 内容或结构的问题影响运行，仍然要多注意改版的资讯。最保险的做法还是让 **buildToolsVersion 的==主版本编号==与 compileSdkVersion 同步**。

当然可用使用高版本的buildToolsVersion了与compileSdkVersion使用，但是没啥必要。
