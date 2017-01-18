---
title: 为什么 Haskell 中 函数类型 不是 “Show” 类型类的实例
id: 432
categories:
  - 技术记录
date: 2016-02-03 10:01:41
tags:
---

# 问题

为什么函数类型不是 `show` 类型类的实例以显示参数和值的类型？

为什么我不能在 GHCi 或者 Hugs 中输入 `\x -&gt; x + x` ，在结果中看到同样的表达式形式作为输出。

为什么存在一个 Show 实例，却只打印函数的类型？

    Prelude&gt; :m + Text.Show.Functions
    Prelude Text.Show.Functions&gt; show Char.ord
    "&lt;function&gt;"
    `</pre>

    如何让 lambdabot 有如下显示：

    <pre>`dons &gt; ord
    lambdabot&gt;  &lt;Char -&gt; Int&gt;
    `</pre>

    # 答案

    ## 实践中的答案

    Haskell 编译器并不保持表达式原本的形式，而是将它们转成机器代码或者其他的底层表示。

    函数 `\x -&gt; x - x + x :: Int -&gt; Int` 也许被优化成 `\x -&gt; x :: Int -&gt; Int` 。没有那个地方l存储了变量名 `x` 。

    你也许会想，Haskell 是一门脚本语言，在运行时环境中维护着表达式的内容。情况并非如此，[Lambda 表达式](https://wiki.haskell.org/Lambda_abstraction)仅仅是[匿名函数](https://wiki.haskell.org/Anonymous_function)。你没可能在运行时环境中查询变量的名字，或者观察函数定义的结构。你也不能从程序员用户那里接收一个表达式，调用你程序中的变量，然后求值。即，Haskell 不是[反射的](https://wiki.haskell.org/index.php?title=Reflexive_language&amp;action=edit&amp;redlink=1 "Reflexive_language&amp;action=edit&amp;redlink=1")。一切都是可编译的。一个例外是 [hs-plugins](https://wiki.haskell.org/index.php?title=Hs-plugins&amp;action=edit&amp;redlink=1 "Hs-plugins&amp;action=edit&amp;redlink=1")

    ## 理论上的答案

    函数式编程是关于函数的。一个数学函数是由它的图通过二元组（参数，值）完全定义的。即：
    - ![\sqrt{\ } = &#123;(0,0), (1,1), (4,2), (9,3), \dots &#125;](https://wiki.haskell.org/wikiupload/math/7/a/9/7a95d49d189193b2987374c4bcd3d186.png)
    - ![ (\lambda x.\ x+x) = &#123;(0,0), (1,2), (2,4), (3,6), \dots &#125; ](https://wiki.haskell.org/wikiupload/math/e/a/f/eaf7cde16e0aa626caa2b0365e0a79b1.png)

    既然图 ![ \mathrm{show}(\lambda x.\ x+x) \ne \mathrm{show}(\lambda x.\ 2\cdot x) ](https://wiki.haskell.org/wikiupload/math/5/3/e/53ef58b99d9d9b4a63006980946855b9.png) 和 ![ \lambda x.\ 2\cdot x ](https://wiki.haskell.org/wikiupload/math/e/3/8/e38a5466b5a1b4d3c8339d700ac9d939.png) 是等价的，因为它们都表示了同样的函数。想象一下如果两个项都按照其形式被 GHCi 或 Hugs 显示，这就意味着等价的函数导致了不同的输出。交互式的 Haskell 环境使用通常的 `show` 函数，这也意味着 ![ \mathrm{show}(\lambda x.\ x+x) \ne \mathrm{show}(\lambda x.\ 2\cdot x) ](https://wiki.haskell.org/wikiupload/math/d/2/1/d21e1b708260732dd43095f9f7b1c881.png) 。这将打破[引用透明](https://wiki.haskell.org/Referential_transparency)。这也意味着显示函数的唯一可行方式是显示定义它们的图：

    <pre>`Prelude&gt; \x -&gt; x+x
    functionFromGraph [(0,0), (1,2), (2,4), (3,6),
    Interrupted.

能够实现这个功能的代码在 [universe-reverse-instances](http://hackage.haskell.org/package/universe-reverse-instances) 包中有。（这个包在安装 [universe](http://hackage.haskell.org/package/universe) 包时会被安装）

# 引用

[https://wiki.haskell.org/Show_instance_for_functions](https://wiki.haskell.org/Show_instance_for_functions)
[http://www.haskell.org/pipermail/haskell-cafe/2006-April/015161.html](http://www.haskell.org/pipermail/haskell-cafe/2006-April/015161.html)