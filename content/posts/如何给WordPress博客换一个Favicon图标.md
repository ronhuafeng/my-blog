---
title: 如何给WordPress博客换一个Favicon图标
id: 9
categories:
  - 技术记录
date: 2012-06-03 16:56:18
tags:
---

整个过程大约分为以下几步

1.  制作favicon.ico

    *   在网站[http://www.favicon.cc/](http://www.favicon.cc/ "favicon.cc")里上传一张图片，通过点击“Import Image”
    *   然后就可以根据各种选项对图片进行调整
    *   最后通过点击“Download Favicon”进行下载
    *   注：如果你有天赋可以自己设计图标，那么在那个编辑器里创作也是可以的

2.  上传到博客目录，比如我上传到mydomain/blog目录下了，链接为[http://www.formalscience.com/blog/favicon.ico](http://www.formalscience.com/blog/favicon.ico "favicon.ico")
3.  修改对应主题内的header.php，在&lt;head&gt;和&lt;/head&gt;之间添加如下代码：

`&lt;LINK rel=icon type=image/x-icon href="http://yourdomain/favicon.ico"&gt;`

`&lt;LINK rel="shortcut icon" type=image/x-icon href="yourdomain/favicon.ico"&gt;`

对于我的博客来说就是

`&lt;LINK rel=icon type=image/x-icon href="http://www.formalscience.com/blog/favicon.ico"&gt;`

`&lt;LINK rel="shortcut icon" type=image/x-icon href="http://www.formalscience.com/blog/favicon.ico"&gt;`

完成以上步骤后清空下浏览器缓存就可以看到效果了。

参考链接：

[ 给WP地址栏添加一个Favicon图标](http://www.xmlas.com/wordpress-add-favicon-ico.html "ref 1")

[WordPress 博客添加个性标志favicon.ico](http://www.boke8.net/wordpress-favicon-ico.html "ref 2")