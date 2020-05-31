---
title: Spin工具的Eclipse集成
id: 54
categories:
  - 技术记录
date: 2012-08-19 02:40:45
tags:
---

Spin是Promela建模语言的解释器和验证工具。它本身有一个图形化的工具iSpin，不过这个图形化实在有些简陋。下面介绍一个Eclipse的插件，这样我们可以使用Eclipse的编辑环境来使用Spin。

首先，在Eclipse的install new software中添加
[http://matrix.uni-mb.si/fileadmin/datoteke/znanost/orodja/ep4s/update-site/](http://matrix.uni-mb.si/fileadmin/datoteke/znanost/orodja/ep4s/update-site/)
把Group items by category选项去掉，否则看不到可安装的插件；安装此插件。

然后，在菜单栏中选择Spin项，进入Spin Configuration，[![image](http://www.formalscience.com/blog/wp-content/uploads/2012/08/image_thumb.png "image")](http://www.formalscience.com/blog/wp-content/uploads/2012/08/image.png)

Spin工具可以从其[官网](http://spinroot.com/spin/Man/README.html)下载。

C Complier里的gcc可以用cygwin安装，记得选择直接的版本，不要选择gcc.exe，否则可能出现不能验证通过的错误。

基本上这样就算安装完成了，如果你有什么问题，可以给我留言。