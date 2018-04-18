---
layout: post
title: jekyll Github Pages 博客搭建 并实现评论 阅读量 以及分类功能
date: 2017-06-06 12:32:24.000000000 +09:00
categories: talk
tags: 网站搭建|个人博客
---
搭建一个属于自己的博客系统，不仅可以将平时记录的知识进行归档和存储，同时也可以进行分享，让自己积累的成果得到别人的认同。相比使用一些博客平台，没有规则的限制，烦恼于编辑器不够好用，以及担心哪天博客gg了，数据会丢失，自建的博客系统可以随心所欲地满足自己的需求，哪时不爽调整哪里，如果你是个文艺小青年，创建一个独一无二的博客系统，不是逼格满满的，想想都让人兴奋吧

当然，你会说，自建一个博客平台，需要太多技术的支持，例如数据库，前端页面，后台逻辑等，买域名，建服务器，岂不是得不偿失。当博客技术发展到今天，已经有不少简单和靠谱的方法，就能实现上面我们所说的需求。接下来，我会给大家讲解，如何实现一个逼格满满，并且功能齐全的博客站点。

[Abbott的博客](https://jinyb09017.github.io/)就是我个人花了一天时间实现的博客系统，接下来，我把整个博客搭建的过程记录下来，希望大家能够搭建一个属于你自己的博客网站。


![image](http://note.youdao.com/yws/public/resource/9c0aa2797b78886ad1cdf66d45169a4a/xmlnote/98E26220F077461F9199458815E2D855/7077)
### 一、在本地搭建一个jekyll环境

简单地说，Jekyll是一个用ruby写的解析引擎，用于从动态的组件中生成静态的网站。

#### 1、安装ruby环境

以下为windows的环境，其它环境大同小异，操作稍有不同。

[ruby下载地址](https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.1.tar.gz).

安装完成后，在cmd下面执行 

```
ruby -v

output:
H:\Users\***>ruby -v
ruby 2.3.3p222 (2016-11-21 revision 56859) [x64-mingw32]

```
如果看到了ruby版本,说明ruby已经安装成功。如果不成功，请查看ruby是否添加到环境变量。

#### 2、安装jekyll


```
gem install jekyll

```
如果是中国大陆用户那么默认的gem源进行安装会有一定困难。这里推荐使用taobao的ruby源。简单的使用以下命令就可以将淘宝repo作为默认repo。


```
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/
gem sources -l
```

#### 3、主题下载到本地Vno-Jekyll

下载主题 [onevcat](https://github.com/onevcat/vno-jekyll),更新到本地，进入主题根目录。

再次执行下面的命令（下载工程依赖，类似于npm install）

```
gem install jekyll 
bundle install
```

#### 4、启动server


```
bundle exec jekyll serve

output:
Configuration file: E:/blog/abbott.github.io/_config.yml
Configuration file: E:/blog/abbott.github.io/_config.yml
            Source: E:/blog/abbott.github.io
       Destination: E:/blog/abbott.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 1.04 seconds.
  Please add the following to your Gemfile to avoid polling for changes:
    gem 'wdm', '>= 0.1.0' if Gem.win_platform?
 Auto-regeneration: enabled for 'E:/blog/abbott.github.io'
Configuration file: E:/blog/abbott.github.io/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.

```
如果看到以上输出，表示jekyll启动成功。在浏览器访问http://127.0.0.1:4000/，即可看到博客的初步界面了。

ps:Ruby 环境 SCSS 编译中文出现 Syntax error: Invalid GBK character 错误解决方法

找到 Ruby 的安装目录，里面也有sass模块，如这个路径：

C:\Ruby21-x64\lib\ruby\gems\2.1.0\gems\sass-3.4.8\lib\sass 

在该路径文件里面 engine.rb，添加一行代码（放在所有的require XXXX 之后即可）：
```
require ...

require'sass/supports'

Encoding.default_external = Encoding.find('utf-8')
```
### 二、发布到Github Pages.
这个步骤比较简单，[Github Pages](https://pages.github.com/)官网首页就有图文说明。

#### 1、创建仓库
打开创建代码仓库界面，创建一个与<用户名>.github.io的创建，如下 
![image](http://note.youdao.com/yws/public/resource/9c0aa2797b78886ad1cdf66d45169a4a/xmlnote/6DB40DFC32044E7C8DC5696A84C3D828/7079)

ps：记住用户名一定要是自己的github的用户名。

#### 2、将仓库克隆到本地

```
git clone 仓库地址
```

#### 3、添加文件index.html,上传到仓库。

```
<!DOCTYPE html>
<html>
<body>
<h1>Hello World</h1>
<p>I'm hosted with GitHub Pages.</p>
</body>
</html>
```
#### 4、浏览器输入(你的用户名).github.io，就初步看到效果啦

![image](http://upload-images.jianshu.io/upload_images/1812927-42b11234a5c3c09c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 5、将步骤一中下载的代码上传到步骤二中建立的仓库，即可以看到通过(你的用户名).github.io域名访问的博客了。



---


到此为止，我们已经创建了一个简单的在线博客系统了。那么问题来了，我们如何管理自己的博客呢，以及如何为博客添加阅读数目统计，评论系统，以及分类呢，请看下面的介绍。

### 三、添加博客

在学习如何添加博客内容的时候，我们先大概了解下jekyll的基本结果

Jekyll的核心其实就是一个文本的转换引擎，用你最喜欢的标记语言写文档，可以是Markdown、Textile或者HTML等等，再通过layout将文档拼装起来，根据你设置的URL规则来展现，这些都是通过严格的配置文件来定义，最终的产出就是web页面。


```
|-- _config.yml
|-- _includes
|-- _layouts
|   |-- default.html
|    -- post.html
|-- _posts
|   |-- 2007-10-29-why-every-programmer-should-play-nethack.textile
|    -- 2009-04-26-barcamp-boston-4-roundup.textile
|-- _site
 -- index.html
```

config文件放全局配置信息

include文件放布局文件

layouts下面放通用模板布局

posts下面则是我们的博客内容

_site则是jekyll生成的静态网站目录

index则是访问首页

1、添加[《科学上网》](https://jinyb09017.github.io/2017/06/%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/)的博客

![image](http://note.youdao.com/yws/public/resource/9c0aa2797b78886ad1cdf66d45169a4a/xmlnote/AD82B45A9905499BA3257AF5D8C34A30/7134)

如上图所示，我们添加的文章是通过markdown的格式添加的，熟悉markdown格式的小伙伴会发现，基本没啥难点。


```
---
layout: post
title: 科学上网
date: 2017-06-05 09:32:24.000000000 +09:00
categories: network
tags: 翻墙-科学上网
---
```
头部的内容为[YAML Front Matter](http://jekyllrb.com/docs/frontmatter/),列举了博客的相关属性。

ps：博客的文件名尽量不要做修改，博客的id是通过文件名字映射索引的。


### 四、添加Disqus评论系统 

Disqus款国外第三方社会化评论系统,将当前不同网站的相对孤立、隔绝的评论系统，连接成具有社会化特性的大网.

#### 1、注册[Disqus](https://disqus.com/)

#### 2、选择setting图标，添加Add disqus to site

如果找不到，直接[点击](https://disqus.com/admin/create/)跳转，输入Website Name

![image](http://note.youdao.com/yws/public/resource/9c0aa2797b78886ad1cdf66d45169a4a/xmlnote/D5A7006B6E8242E7AF853AA20BEA224D/7174)

#### 3、设置根目录下_config.yml文件的disqus的URL

修改comment下面的disqus的值为自己填的Website Name.
```
# Comment 这是disqus的评论系统
comment:
    disqus: abbott-king
    # duoshuo: 
```

这样，我们的的评论系统就建立好了，可以登陆Disqus帐号后台，查看评论相关的数据。

### 五、添加阅读量

一般的博客系统，都会有帖子的阅读数量，可以查看自己文章有多少人阅读了。jekyll好像暂时没有相应的插件支持，但是我们可以通过cloudlean进行支持。

一般的博客系统，都会有帖子的阅读数量，可以查看自己文章有多少人阅读了。jekyll好像暂时没有相应的插件支持，但是我们可以通过LeanCloud进行支持。

LeanCloud 介绍：LeanCloud 是行业领先的一站式后端云服务提供商，专注于为移动开发者提供一流的工具、平台和服务。LeanCloud提供了很强大的功能，本文中用到的只是LeanCloud中的数据存储功能，而且没有涉及复杂的数据处理。

思路：在进入文章的时候，调用LeanCloud的api，实现具体文章的点击数目添加。并存储到LeanCloud云平台。

#### 1、注册LeanCloud

#### 2、创建应用，申请appid和appkey

需要注意：App ID和App Key要用在你的博客中，所以基本上等同于是公开的。所以需要做一些安全设置，以防止App ID和 App Key被滥用

千万不要泄露Master Key，拥有Master Key相当于拥有了root权限。

#### 3 创建Class

名字自定义，名字限制：只能包含字母、数字、下划线，必须以字母开头。 

我们可以创建了名为Counter的Class，采用默认的“限制写入”这种ACL权限设置

#### 4 修改代码

**4.1 在_config.yml文件中，添加如下代码：**


```
leancloud:
  enable: true
  app_id: your_id
  app_key: your_key
```
app_id和app_key为我们自己注册申请的。
  
**4.2 添加leancloud-analytics.html文件**


```
{% if site.leancloud.enable %}

<script src="https://code.jquery.com/jquery-3.2.0.min.js"></script>
<script src="https://cdn1.lncld.net/static/js/av-core-mini-0.6.1.js"></script>
<script>AV.initialize("{{site.leancloud.app_id}}", "{{site.leancloud.app_key}}");</script>
<script>
    function showHitCount(Counter) {
        var query = new AV.Query(Counter);
        var entries = [];
        var $visitors = $(".leancloud_visitors");
        $visitors.each(function () {
            entries.push( $(this).attr("id").trim() );
        });
        query.containedIn('url', entries);
        query.find()
                .done(function (results) {
                    console.log("results",results);
                    var COUNT_CONTAINER_REF = '.leancloud-visitors-count';
                    if (results.length === 0) {
                        $visitors.find(COUNT_CONTAINER_REF).text(0);
                        return;
                    }
                    for (var i = 0; i < results.length; i++) {
                        var item = results[i];
                        var url = item.get('url');
                        var hits = item.get('hits');
                        var element = document.getElementById(url);
                        $(element).find(COUNT_CONTAINER_REF).text(hits);
                    }
                    for(var i = 0; i < entries.length; i++) {
                        var url = entries[i];
                        var element = document.getElementById(url);
                        var countSpan = $(element).find(COUNT_CONTAINER_REF);
                        if( countSpan.text() == '') {
                            countSpan.text(0);
                        }
                    }
                })
                .fail(function (object, error) {
                    console.log("Error: " + error.code + " " + error.message);
                });
    }
    function addCount(Counter) {
        var $visitors = $(".leancloud_visitors");
        var url = $visitors.attr('id').trim();
        var title = $visitors.attr('data-flag-title').trim();
        var query = new AV.Query(Counter);
        query.equalTo("url", url);
        query.find({
            success: function(results) {
                if (results.length > 0) {
                    var counter = results[0];
                    counter.fetchWhenSave(true);
                    counter.increment("hits");
                    counter.save(null, {
                        success: function(counter) {
                            var $element = $(document.getElementById(url));
                            $element.find('.leancloud-visitors-count').text(counter.get('hits'));
                        },
                        error: function(counter, error) {
                            console.log('Failed to save Visitor num, with error message: ' + error.message);
                        }
                    });
                } else {
                    var newcounter = new Counter();
                    /* Set ACL */
                    var acl = new AV.ACL();
                    acl.setPublicReadAccess(true);
                    acl.setPublicWriteAccess(true);
                    newcounter.setACL(acl);
                    /* End Set ACL */
                    newcounter.set("title", title);
                    newcounter.set("url", url);
                    newcounter.set("hits", 1);
                    newcounter.save(null, {
                        success: function(newcounter) {
                            var $element = $(document.getElementById(url));
                            $element.find('.leancloud-visitors-count').text(newcounter.get('hits'));
                        },
                        error: function(newcounter, error) {
                            console.log('Failed to create');
                        }
                    });
                }
            },
            error: function(error) {
                console.log('Error:' + error.code + " " + error.message);
            }
        });
    }
    $(function() {
        var Counter = AV.Object.extend("Counter");
        if ($('.leancloud_visitors').length == 1) {
            // in post.html, so add 1 to hit counts
            addCount(Counter);
        } else if ($('.post-link').length > 1){
            // in index.html, there are many 'leancloud_visitors' and 'post-link', so just show hit counts.
            showHitCount(Counter);
        }
    });
</script>
{% endif %}
```

**4.3 _layouts/default.html**

default.html是所有页面（包括index.html和每一篇博文）的模板文件。 
在 _layouts/default.html 中添加 leancloud-analytics.html 。 
在default.html中添加，是为了让LeanCloud的代码出现在每一篇blog中。


```
{% include leancloud-analytics.html %}
```


**4.4 _layouts/post.html**
post.html是我所发的blog的布局模板。在post.html中有发布时间、作者、标签、分类和文章内容。文章点击数量需要在这里显示，所以需要加上如下代码：


```
{% if site.leancloud.enable %}
       <span id="{{ page.url }}" class="leancloud_visitors" data-flag-title="{{ page.title }}">
        <span class="post-meta-divider">|</span>
        <span class="post-meta-item-text"> Hits:  </span>
        <span class="leancloud-visitors-count"></span>
       </span>
   {% endif %}
```

   
需要注意的是，当前页面的信息记录在变量page中。page.url表示当前文章的地址，是以斜线开头的相对路径，例如blog/2016/12/18/test2。page.title是文章的标题，是为了在LeanCloud后台中看着方便的。当然，也可以用于判断是否是同一篇文章。

### 六、添加分类
随着文章的数目变多，如果没有一个合适的分类进行索引，不仅不利于我们自己检索内容，也不方便读者进行筛选。所以添加一个分类也是一个博客系统必备的功能。

操作方法：[参考文章](https://christianspecht.de/2014/10/25/separate-pages-per-tag-category-with-jekyll-without-plugins/)

好累。。。写个操作手册，比搭建网站还费时间，希望能给大家带来一些帮助。
