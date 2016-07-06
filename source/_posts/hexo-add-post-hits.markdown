---
title: hexo yilia主题增加文章阅读数
date: 2016-06-30 21:18:14
tags:
- hexo
- yilia
---

hexo是一个静态页面，诸如统计，留言之类的动态内容都需要依赖第三方服务，或者你自己有服务器资源也可以自己写个，
真心不难，对于文章阅读数统计，有几个第三方服务提供商：

1. leancloud
2. firebase
3. busuanzi

经过对比，个人任务leancloud最优，免费账号提供的资源足够用了;cdn在国内，对于国内用户方位速度较快;可以在归档页面显示
文章阅读数。就像这样
![文章浏览数 效果图](/img/post/post-archieve-hits.png)

下面结合网上众多的教程，写下在yilia主题下增加文章阅读数的过程。

<!-- more -->

## 准备leancloud应用

完成leancloud账户注册，然后新建应用：

![新建应用](/img/post/leancloud-app.png)

然后新创建一个Class，起名就叫Counter吧，权限选择`无限制`

![创建Class](/img/post/create-app.png)

然后取得你的应用信息，待会会用。

![getAppInfo](/img/post/getappkey.png)

## 修改yilia主题

在增改yilia主题文件文件之前，先改下主目录的配置文件`_config.yml`

### 修改`_config.yml`配置

```bash
leancloud_visitors:
  enable: true
  app_id: your_app_id
  app_key: your_app_key
```

### 增加Counter插件
在主题的layout目录下新建立一个_widget目录，然后新建文件`Counter.ejs`,当然你可以按照你的意思在将这个代码放到
符合你意愿的文件中。

```html
<!-- jQuery文件-->
<script src="//cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
<!-- 你可以用官方的cdn，那就把这里注释掉 -->
<!-- <script src="https://cdn1.lncld.net/static/js/av-core-mini-0.6.1.js"></script> -->
<%- js('js/av-core-mini-0.6.1') %>
<script>AV.initialize("<%= theme.leancloud_visitors.app_id %>", "<%= theme.leancloud_visitors.app_key%>");</script>
<!-- 注意这里，一会要把这个js文件引入进来 -->
<%- js('js/Counter') %>
```

### 新建Counter.js
在主题（这里是yilia）的`source/js`目录下，新建`Counter.js`。内容如下：

```js
// 这里是显示 归档等地方的 点击数的
function showTime(Counter) {
    var query = new AV.Query(Counter);
    $(".leancloud_visitors").each(function () {
        var url = $(this).attr("id").trim();
        query.equalTo("url", url);
        query.find({
            success: function (results) {
                if (results.length == 0) {
                    var content = '0 ' + $(document.getElementById(url)).text();
                    $(document.getElementById(url)).text(content);
                    return;
                }
                for (var i = 0; i < results.length; i++) {
                    var object = results[i];
                    var content = object.get('time') + ' ' + $(document.getElementById(url)).text();
                    $(document.getElementById(url)).text(content);
                }
            },
            error: function (object, error) {
                console.log("Error: " + error.code + " " + error.message);
            }
        });

    });
}

function addCount(Counter) {
    var Counter = AV.Object.extend("Counter");
    // 注意 选择子 选择的类名，如果你想自定义类名，别忘记修改这里
    url = $(".leancloud_visitors").attr('id').trim();
    title = $(".leancloud_visitors").attr('data-flag-title').trim();
    
    var query = new AV.Query(Counter);
    query.equalTo("url", url);
    query.find({
        success: function (results) {
            console.log(results);
            // 如果命中，浏览数+1
            if (results.length > 0) {
                var counter = results[0];
                console.log(counter.get('time'));
                counter.fetchWhenSave(true);
                counter.increment("time");
                counter.save(null, {
                    success: function (counter) {
                        var content = counter.get('time') + ' ' + $(document.getElementById(url)).text();
                        $(document.getElementById(url)).text(content);
                    },
                    // 注意，这里如果请求leancloud，为了不至于显示不了数字，改成如下的代码
                    error: function (counter, error) {
                        var content = counter.get('time') + ' ' + $(document.getElementById(url)).text();
                        $(document.getElementById(url)).text(content);
                        //console.log('Failed to save Visitor num, with error message: ' + error.message);
                    }
                });
            } else {
            // 没有命中，新建counter项
                var newcounter = new Counter();
                newcounter.set("title", title);
                newcounter.set("url", url);
                newcounter.set("time", 1);
                newcounter.save(null, {
                    success: function (newcounter) {
                        console.log("newcounter.get('time')=" + newcounter.get('time'));
                        var content = newcounter.get('time') + ' ' + $(document.getElementById(url)).text();
                        $(document.getElementById(url)).text(content);
                    },
                    error: function (newcounter, error) {
                        console.log('Failed to create');
                    }
                });
            }
        },
        error: function (error) {
            console.log('Error:' + error.code + " " + error.message);
        }
    });
}
$(function () {
    var Counter = AV.Object.extend("Counter");
    if ($('.leancloud_visitors').length == 1) {
        console.log($('.leancloud_visitors'));
        addCount(Counter);
    } else if ($('.post-title-link').length >= 1) {
    // 注意边界值
        showTime(Counter);
    }
});
```

### 修改head.ejs

在head.ejs中的`</head>`加上如下代码就好，以便能够加载到我们上面定义的`Counter.ejs`页面组件。

```js
    <!-- 增加点击次数插件-->
    <% if (theme.leancloud_visitors.enable){ %>
        <%- partial('../_widget/Counter') %>
    <% } %>
```

### 修改post/tag.ejs

这个看你个人意愿，想把点击数放到什么地方了，按照我的审美，我决定把他放在tag后面，如上面的效果图哈。

在文件中，append一下内容：

```html
<% if (theme.leancloud_visitors.enable) { %>
<!-- 这个counter-tag counter的样式，你们得自己弄啊.. 我就不想贴样式代码了.. -->
<div class="counter-tag counter">
    <!-- 别忘记这个类名... post-title-link -->
    <span id="<%- url_for(post.path) %>" class="leancloud_visitors post-title-link"
          style="font-size: 12px" data-flag-title="<%= post.title %>">
         &nbsp;
    <!-- 这里给出浏览数,定义在language中哈-->
        <%= __('post.visitors') %>
    </span>
</div>
<% } %>
```

### 修改语言配置文件

给你喜欢的语言增加上下面的配置吧，我都写成这样了..

```text
post:
  ...
  visitors: Hits
  ...
```

### 调整样式

最后你得按照你的主题调整下样式吧~ 这个我花费的时间最长来，比如如何显示那个眼睛啊~ 什么是Font awesome..
写styl文件，定义counter-tag counter的样式，还是那句话，你可以自定义类名，自定义样式。

### 折腾

你肯定得琢磨个把小时，好好享受这个过程吧。

## 部署应用

到了这里你肯定做好了吧:>,那么就把它托管到github上吧,有问题随时交流~~

> 参考

1. [使用 LeanCloud 为 Hexo 博客添加文章浏览量统计组件](https://forum.leancloud.cn/t/yong-hu-fen-xiang-shi-yong-leancloud-wei-hexo-bo-ke-tian-jia-wen-zhang-liu-lan-liang-tong-ji-zu-jian/280)
2. [Hexo的NexT主题个性化：添加文章阅读量](http://www.jeyzhang.com/hexo-next-add-post-views.html)
3. [为hexo博客添加访问次数统计功能](http://ibruce.info/2013/12/22/count-views-of-hexo/)
3. [FontAwesome参考1](https://astronautweb.co/snippet/font-awesome/)
4. [FontAwesome参考2](http://www.bootcss.com/p/font-awesome/)
5. [使用FontAwesome fa-eye](http://fontawesome.io/icon/eye/)


