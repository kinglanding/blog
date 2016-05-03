# blog

这个repo是kinglanding.github.io的内容本身。

从octopress转换到hexo，不同的框架对markdown的支持是不同的。如果要在hexo中支持<!-- more -->这个标签，需要将markdown的渲染引擎换成 $hexo-renderer-markdown-it$ .

在根目录的_config.ymal中，将

```text
markdown:
  render:
    html: true
```
重新生成下博客内容，👌 搞定啦～
