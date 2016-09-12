---
layout: post
title:  "开篇第一博，教教你们怎么样搭建自己的博客"
date:   2016-09-12
excerpt: "经常苦恼到底在哪个网站建博客，但由于自己懒，所以计划一直搁置。最近了解了github pages和jekyll。于是重新开始我的捣(zhuang)鼓(bi)之路."
project: blog
tag:
- jekyll 
- moon
- blog
- github pages
- theme
comments: true
---

经常苦恼到底在哪个网站建博客，但由于自己懒，所以计划一直搁置。最近了解了`github pages`和`jekyll`。于是重新开始我的捣(zhuang)鼓(bi)之路.

由于Github的不定期抽风，于是我选择了国内的`coding pages`代替。然后在[`jekyll themes`](http://jekyllthemes.org/)上找到了这款模板

![Moon Homepage](/assets/img/theme.png)    
    
<center><b>Moon</b> is a minimal, one column jekyll theme.</center>

为了避免当你看到这篇博客的时候，这个模版已经更新了，我fork了一份到我的github。你可以clone到自己的本地，然后进行修改。[clone](https://github.com/okou19900722/Moon)

`jekyll`是一个简单的免费的Blog生成工具，类似`WordPress`。但是和`WordPress`又有很大的不同，原因是`jekyll`只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如`Disqus`。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。

本地使用`jekyll`需要一个`ruby`环境，网上有很多教程，但我没有配置成功，最终在`github`上找到了一个一键环境[`PortableJekyll`](https://github.com/madhur/PortableJekyll)，下载下来大约800多M。解压后双击目录下的`setpath.cmd`，打开的cmd命令行就是已经设置好环境变量的窗口。

切换到`jekyll`模板的目录下或者通过以下命令创建新的站点
{% highlight cmd %}
jekyll new name
{% endhighlight %}
然后通过以下命令启动`jekyll`服务
{% highlight cmd %}
jekyll s --watch
{% endhighlight %}
watch是用来时间监控文件，文件修改之后，热更新站点。但是好像`_config.yml`修改之后必须重启。`jekyll`服务启动之后，如下图
![Success Photo](/assets/img/post1_1.png)
这时访问[`http://localhost:4000`](http://localhost:4000)可以查看站点