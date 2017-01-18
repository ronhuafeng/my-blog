---
title: '基于OpenShift搭建MediaWiki'
id: 73
categories:
  - 技术记录
date: 2012-11-12 15:45:19
tags:
---

1.按照[OpenShift](https://openshift.redhat.com/app/ "OpenShift")建立用用的过程新建一个PHP 5.3应用，可以通过网页方式创建（推荐）；本地完成一些初始化工作（参考官方[教程](https://openshift.redhat.com/community/get-started "OpenShift本地设置的一些步骤")）

2.按照应用页面给出的git地址，把代码库clone到本地，然后把MediaWiki代码push到Master库中。（[git使用方法](http://rogerdudler.github.com/git-guide/index.zh.html "git使用方法")）

3.在应用页面点击那个“Add a Cartridge”，然后加上一个MySql数据库（应该是这样，我在命令行中添加的）和一个phpMyAdmin；把获得的数据库root用户名和密码记下来；
但是如何重新访问这些Cartridge，我还不知道啊，也许从命令行可以过去。

4.进入phpMyAdmin进行管理，左上方有MediaWiki连接数据库要使用的数据库地址（这个地方寻找地址卡了好久，直到安装了phpMyAdmin才得到的sql连接地址，应该还有其他方法的）

5.进入应用URL进行MediaWiki的初始化工作，剩下的就很简单了