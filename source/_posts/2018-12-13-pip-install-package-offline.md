---
title: pip 离线安装包
date: 2018-12-13
tags:
  - python
categories:
  - 技术记录
  - tech-notes
---

## 1. 使用场景

在没有网络的设备上使用 pip 安装包。下面以 `sklearn` 包为例展示如何在没有网络的环境下安装包。

## 2. 下载包到本地缓存

首先进入一个目录，在这个例子里是 `/Users/bef0rewind/Downloads/pip-tmp` 目录。

```
pip download sklearn
```

我这里下载到了一个缓存目录 `/Users/bef0rewind/Downloads/pip-tmp`，随便选一个就好。`pip download` 只会下载对应的包，不会进行安装。

此时使用，`pip freeze` 可以看到已经安装的包，如果之前没有安装过 `sklearn`，显示的列表里是没有这个包的。

## 3. 断网安装

为了展示没有网络的情况下如何安装，我断开网络进行了验证。

```
pip install --no-index --find-links=/Users/bef0rewind/Downloads/pip-tmp sklearn
```

其中 `--find-links` 要 `pip` 从指定的目录里寻找安装包。

## 4. 其他

如果要用 Python3，而系统默认的版本是 Python 2，则可以将 `pip` 命令换成 `pip3`。

