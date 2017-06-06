### 执行命令

bundle exec jekyll serve


### 五、添加阅读量

一般的博客系统，都会有帖子的阅读数量，可以查看自己文章有多少人阅读了。jekyll好像暂时没有相应的插件支持，但是我们可以通过LeanCloud进行支持。

LeanCloud 介绍：LeanCloud 是行业领先的一站式后端云服务提供商，专注于为移动开发者提供一流的工具、平台和服务。LeanCloud提供了很强大的功能，本文中用到的只是LeanCloud中的数据存储功能，而且没有涉及复杂的数据处理。

思路：在进入文章的时候，调用LeanCloud的api，实现具体文章的点击数目添加。并存储到LeanCloud云平台。

1、注册LeanCloud

2、创建应用，申请appid和appkey

需要注意：App ID和App Key要用在你的博客中，所以基本上等同于是公开的。所以需要做一些安全设置，以防止App ID和 App Key被滥用

千万不要泄露Master Key，拥有Master Key相当于拥有了root权限。

3 创建Class

名字自定义，名字限制：只能包含字母、数字、下划线，必须以字母开头。 

我们可以创建了名为Counter的Class，采用默认的“限制写入”这种ACL权限设置

4 修改代码

4.1 在_config.yml文件中，添加如下代码：

leancloud:
  enable: true
  app_id: edcgBgescCWpVAgimGW1nbfL-gzGzoHsz
  app_key: 2c30PCULFMUSgBVNOwNKpqeD
  
4.2 添加leancloud-analytics.html文件

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

4.3 _layouts/default.html

default.html是所有页面（包括index.html和每一篇博文）的模板文件。 
在 _layouts/default.html 中添加 leancloud-analytics.html 。 
在default.html中添加，是为了让LeanCloud的代码出现在每一篇blog中。

{% include leancloud-analytics.html %}

4.4 _layouts/post.html
post.html是我所发的blog的布局模板。在post.html中有发布时间、作者、标签、分类和文章内容。文章点击数量需要在这里显示，所以需要加上如下代码：

{% if site.leancloud.enable %}
       <span id="{{ page.url }}" class="leancloud_visitors" data-flag-title="{{ page.title }}">
        <span class="post-meta-divider">|</span>
        <span class="post-meta-item-text"> Hits:  </span>
        <span class="leancloud-visitors-count"></span>
       </span>
   {% endif %}
   
需要注意的是，当前页面的信息记录在变量page中。page.url表示当前文章的地址，是以斜线开头的相对路径，例如blog/2016/12/18/test2。page.title是文章的标题，是为了在LeanCloud后台中看着方便的。当然，也可以用于判断是否是同一篇文章。