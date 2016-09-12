---
layout: post
title:  "创建属于自己的字体"
date:   2016-09-12
excerpt: "上一篇说到下载了Moon模版创建自己的博客，Moon模版使用FontAwesome字体，虽然这个字体很全面，包括了github,facebook,twitter等站的图标，甚至国内的微博、qq都有，但毕竟是外国人搞的东西，相对国外来说小众的，只服务国人的网站就找不到对应的字符了."
project: blog
tag:
- moon
comments: true
---

上一篇说到下载了`Moon`模版创建自己的博客，Moon模版使用`FontAwesome`字体，虽然这个字体很全面，包括了`github`,`facebook`,`twitter`等站的图标，甚至国内的`微博`、`qq`都有，但毕竟是外国人搞的东西，相对国外来说小众的，只服务国人的网站就找不到对应的字符了.

本来是准备在首页放个`coding`的标，所以就百度字体的制作方法，找到了一款叫`FontCreator`的软件，安装之后打开`FontAwesome`字体，也可以点`File`->`New`创建新的字体，我在这里只教大家怎么在已有的字体中添加新的字符，毕竟如果创造新的字体库，那么需要改动的地方就更多了。

使用`FontCreator`打开字体之后，点击如下图所示按钮添加一个字符

![Create Character](/assets/img/posts/2/1.png)

在新打开的页面中选择没有占用的字符，下图所示的是已经被占用的字符的颜色

![create character](/assets/img/posts/2/2.png)

双击选择的空白字符，最窗口最下面`Codepoints`栏自动插入了选中字符的16进制编码，点击`ok`确定

![create character](/assets/img/posts/2/3.png)

在下面的窗口中就可以看到新创建的字符了，选中创建的字符，单击鼠标右键，选择`Glyph Properties`可以修改字符的属性

![create character](/assets/img/posts/2/4.png)

修改名字

![create character](/assets/img/posts/2/5.png)

然后双击刚刚创建的字符，可以创建内容。可以选择`Import Image`从图片中创建

![create character](/assets/img/posts/2/6.png)

选择好图片，就会出现下图，直接点击`Generate`则创建好了

![create character](/assets/img/posts/2/7.png)

下图就是导入图片之后的内容，软件会默认将白色的部分变成透明，所以可以不用自己ps图片去扣

![create character](/assets/img/posts/2/8.png)

关掉界面就可以了，鼠标移动到字符上，可以看到该字符的信息。前面我们选择的那个16进制就是他的编码，在程序中有用，比如这里的`$F2A2`

![create character](/assets/img/posts/2/9.png)

按下图的指示保存当前字体

![create character](/assets/img/posts/2/10.png)

然后可以用百度搜索web字体转换，我使用的是这个网站[点击跳转](https://everythingfonts.com/font-face)，`Pick Font File`选择刚刚保存的ttf字体文件，然后点击`Convert`导出字体

下载完成之后，解压，按`/assets/fonts/FontAwesome`目录中的文件一一对应替换。这个网站好像会把`-`给去掉了，自己手动加上.

然后修改`/_sass/vendor/font-awesome/_variables.scss`文件，添加

{% highlight css %}
$fa-var-coding: "\f2a0";
{% endhighlight %}

然后修改`/_sass/vendor/font-awesome/_icons.scss`文件，添加
{% highlight css %}
.#{$fa-css-prefix}-coding:before { content: $fa-var-coding; }
{% endhighlight %}

这样就ok了，以后就可以在需要用到的地方直接使用`fa-coding`来显示`coding`的图标了，因为字体是矢量的，多大都不会失真了。

由于主页位置有限，最终还是没有放coding的图标