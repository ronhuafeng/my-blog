---
title: 使用 Nvidia 显卡加速机器学习算法的一些资料
date: 2018-01-13 13:14:09
categories:
  - 技术记录
tags: 
---

Nnidia 显卡可以用来加速机器学习算法（特别是深度学习），但安装驱动过程中总会碰到这样或者那样的问题。
一个难点是安装库的时候没有下载链接，比如 Nvidia 的 Cuda/cuDNN 主页经常会出现这样的提示：

```
NVIDIA Developer Site is under going maintenance.
      The site will be back by shortly.
      We apologize for any inconvenience.
```
虽然不能按照官方路径进行下载，但经过搜索总能找到一些入口。下面是我收集的一些链接：

- cuDNN 
  - 下载页面：https://developer.nvidia.com/rdp/cudnn-download 
  - 这个页面需要注册 Nvidia 开发者账号并登录，当前包含从 `cuDNN v5.1` 到 `cuDNN 7.0.5` 的版本
- CUDA
  - 下载页面：https://developer.nvidia.com/cuda-toolkit-archive
  - 这个页面需要注册 Nvidia 开发者账号并登录，当前包含从 `CUDA Toolkit 1.0` 到 `CUDA Toolkit 9.0` 的版本，最新的 `CUDA Toolkit 9.1` 的链接仍然指向*正在维护*的提示页面
  - Ubuntu 16.04 `CUDA Toolkit 9.1` 下载链接：http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_9.1.85-1_amd64.deb 
  - 此外可以在这个页面找到其他 Linux 发行版的安装包：http://developer.download.nvidia.com/compute/cuda/repos/
  - Windows 10 版本的 `CUDA Toolkit 9.1` 我成功下载过一次，但是和当前的 `TensorFlow 1.4` 不兼容，刚才发现之前组合出来的下载链接无效了。如果找到链接会补充上。当前可用 `cuDNN v6.1` 和 `CUDA Toolkit 8.0 GA2` 版本。

