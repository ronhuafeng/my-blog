---
title: 为什么用Clojure，以及一点感悟
id: 111
categories:
  - 技术记录
date: 2013-09-02 12:12:39
tags:
---

<span style="color: #cc99ff;">**1.Java互操作**</span>

Clojure和Java互操作还是比较简单的，网上有很多案例。

首先讲下Java中调用Clojure。

一种方式是用过clojure.lang中的一些方法引入一个命名空间中的函数和变量，然后使用，这种方法就要注意ClassPath和ClassLoader的问题，这在Eclipse的RCP开发中还是比较烦的。另外要注意如果有Clojure源码文件被编译成Class文件了，调用这些文件的Clojure.jar和编译出这些文件的Clojure.jar版本一定要一致。

另外一种方式就是使用 “:aot :all“ 命令把所有Clojure源码编译成JVM的Class文件，然后调用genclass方法产生的类和函数。这个可以比较方便地把接口暴露出来。[这篇文章](https://kotka.de/blog/2010/02/gen-class_how_it_works_and_how_to_use_it.html "这篇文章")针对genclass的各种用法讲得很好。

在Clojure中调用Java就很简单了，随便一搜就是一大堆，官方文档也有很多例子。

<span style="color: #cc99ff;">**2.编程模式**</span>

Clojure本身就是JVM上的一个Lisp方言，所以编程模式主要就是函数式。当然我对函数式这个术语理解也不是很透彻，目前理解的函数式语言的特点就是：函数是第一公民、尽量保持无状态操作、有利于并行化操作、具有较强的作为meta language的能力（可以用来构建DSL）。这个可以看一下[SCIP（计算机程序的构造和解释）](http://book.douban.com/subject/1148282/ "SCIP（计算机程序的构造和解释）")这本书。

<span style="color: #cc99ff;">**3.初衷**</span>

我本来想用Scala的好嘛！其实，当时主要想做[BIP](http://www-verimag.imag.fr/New-BIP-tools.html?lang=en "BIP")（这个还是高端大气上档次的）这个形式化语言的一个解释器，然后依靠那个不靠谱的方案和一个还不存在的同步形式化语言解释器进行对接，构建一个仿真环境。

其实同为JVM上寄生的函数式语言，明显Scala做这个工作更有优势一点：基于OCaml（这个的前身ML我之前用过）、有方便的类型模式匹配机制（这个机制在写解释器时应该比较有用）。Clojure我也调研过，但是当时是没有这个想法用它的。但是呢，但是呢，亚马逊给我推送了一封邮件，里面就有《[Clojure编程](http://book.douban.com/subject/21661495/ "《Clojure编程》")》这本书，然后我还顺着广告点过去买了。于是在我刚开始做这个工作的时候，Clojure就被选中了。

都是缘分啊！

**<span style="color: #cc99ff;">4.困难</span>**

困难一部分来自对Clojure一些库不了解，对一些常见的实现习惯不了解，后来实践中逐渐搜索学习，也就掌握了。

另外一些来自于和实验室主项目集成的时候，Eclipse RCP项目的ClassLoader比较诡异，我之前也没怎么了解过，碰到了各种”ClassNotFound“的错误，后来经过查找还是解决了。[这个是我的一个Evernote笔记](evernote:///view/1204641/s10/3c46b4c5-4bde-4be6-b9c8-753fae940d21/3c46b4c5-4bde-4be6-b9c8-753fae940d21/http:// "这个是我的一个Evernote笔记")。

**<span style="color: #cc99ff;">5.编辑环境的的配置和各种尝试</span>**

我先后试过Emacs和Intellij IDEA。

Emacs我还是很喜欢的，特别是里面的ParEdit插件，用来进行Lisp中大量的括号相关的操作简直无敌了，就是要先学习一下快捷键，效率会提升很多。但是有一个缺点（我能说Emacs的缺点么？），配置起来麻烦。虽然各种插件我都装上了，用起来也不错，但是过程中遇到的各种陷阱还是很多的，网上的教程也不总是和预想中的一样有效。我还是一直坚持使用Emacs的。后来实验室的另一个工作要在Windows上进行，在Windows下我一直无法用Emacs启动Clojure的REPL，因此转向了神器——Intellij IDEA。

神器只要下个LaClojure的插件和Leiningen插件就好了。各种只能提示啦，测试啦都可以很方便使用，我还把keymap改成了Emacs式的，算是一个念想。不过神器下面没有ParEdit插件，没法那么方便地处理括号，有些不开心。

**<span style="color: #cc99ff;">6.总结</span>**

在学校的一个优势呢，就是可以尝试新的东西（虽然想让我赶快完成项目的老板不这样想）。以前我也学习Lisp，ML等函数式语言，但是一直没有实际动手做个东西。这次虽然使用Clojure做这个解释器有些冲动的成分在里面，整体而言还是很满意的。最后解释器效率可能有些问题（我写的程序有问题，和语言无关），但是程序工作起来还比较正确（至少那些单元测试都还能通过），修改起来也比较容易（还需要一些重构）。我对于Java和Clojure的了解也都稍微深刻了那么一点（是在命名空间这方面）。

下次要不要试试其他的语言呢？

**<span style="color: #cc99ff;">7.项目</span>**

主要是做了一个BIP的解释器，还没有做完，只是完成了核心的语义解释的一部分。前段的语法解析器没有做，不过这个有现成的。

**<span style="color: #cc99ff;">8.经验</span>**

实践出真知！

这首歌我最近一直在单曲循环。

&nbsp;
<object width="257" height="33" classid="clsid:d27cdb6e-ae6d-11cf-96b8-444553540000" codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=6,0,40,0"><param name="src" value="http://www.xiami.com/widget/3240498_59168/singlePlayer.swf" /><param name="wmode" value="transparent" /><embed width="257" height="33" type="application/x-shockwave-flash" src="http://www.xiami.com/widget/3240498_59168/singlePlayer.swf" wmode="transparent" /></object>