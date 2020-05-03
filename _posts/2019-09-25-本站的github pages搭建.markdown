---
layout:     post
title:      "本站的github pages搭建"
subtitle:   " \"github pages\""
date:       2019-09-25 15:00:00
author:     "Conerlius"
category: 其他
keywords: jekyll,github pages
tags:
    - jekyll
---

* content
{:toc}

## 前提说明
此搭建方式是基于github pages来搭建的，而github pages就是搭建了jekyll，所以这里仅仅只是简单的搭建，如果有大佬要自己写样式，请忽略本文！
## Blog模板
在[知乎](https://www.zhihu.com/question/20223939/answer/29742210)上有不少这样的问答，本文用的就是[林安亚的博客](https://github.com/lay1010/lay1010.github.io)

![jpg](/images/jekyll_github_pages_1.jpg)

## 搭建
### 模板上行
本文没有使用git的fork，因为我是从黄玄大佬的blog样式换成这个样式的。这里就简单说说github pages的创建；
#### github pages创建
打开[github](https://github.com/)，然后创建一个名为`XXX.github.io`的仓
![jpg](/images/jekyll_github_pages_2.jpg)
``https://github.com/XXX?tab=repositories``这里的XXX要和`XXX.github.io`一样，**否则你所创建的pages生成的link就会带有其他的前缀了！**

---

创建完成后，通过git 把自己的github.io clone到本地；然后下载本文上面提供的模板。
> 注意：虽然这个模板上写着是就，建议去行的特性看，但是在2019年9月的时候，我还是不建议使用新的，还是用旧的！
> 
![jpg](/images/jekyll_github_pages_3.jpg)

将下载完成的模板解压，除了红色框出来的三个不用拷贝外，其他的都拷贝到自己的github.io下；
> 拷贝完成后，记得在自己的github.io下创建一个`_post`的目录

到此，blog的文件基本齐全了，下面就是要修改配置了。

### 配置修改
* _config.yml
> 这里只需要把一些原作者的文本改成自己的就可以了

### 集成google analytics
* 先去[google analytics](https://analytics.google.com/analytics/web/)注册一个账号，然后创建一个app，将获取到的id填写到`_config.yml`中
  
```
# google analytics 设置
ga:
  id: UA-XXXXXXXX-X
  url: conerlius.github.io
```

### 集成disqus
* 先去[disqus](https://disqus.com)注册一个账号，然后创建一个app，将获取到的`shortname`填写到`disqus.html`中
  
```
<div id="disqus_thread"></div>
<script type="text/javascript">
    /* * * CONFIGURATION VARIABLES * * */
    var disqus_shortname = 'XXXXXXX';
    
    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>

```

这里就已经全部集成完成了！