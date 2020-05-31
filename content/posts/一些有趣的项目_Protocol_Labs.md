---
title: 一些有趣的项目 Protocol Labs
categories:
  - 技术记录
date: 2018-01-12 21:37:21
tags:
  - note
---

最初在 3Blue1Brown 发布的一个介绍区块链原理的视频中看到了这个组织的连接。发现比较有意思，给大家分享一下~

项目的使命：

> We believe the internet has become humanity's most important technology. We build protocols, systems, and tools to improve how it works. Today, we are focused on how we store, locate, and move information.

> 我们相信互联网已成为人类最重要的技术。我们构建提升互联网工作能力的协议、系统和工具。当前我们集中在如何存储、定位和移动信息的工作上。

这段文字翻译得有点机器翻译风格。

项目地址：https://protocol.ai/projects/ ，目前上面有 5 个项目：

1. **Filecoin**
  - 加密货币，Miners 通过向网络提供存储空间来获取 `Filecoin` ，使用者通过消耗 `Filecoin` 来在**去中心化的**网络中存储加密后的文件。
2. **IPFS** (InterPlanetary File System)
  - 一种新型协议，用来使网络去中心化。IPFS 通过内容寻址和数字签名来创建完全去中心化和分布式的应用。IPFS 使得网络更快、更安全以及更加开放。
  - 这是一段 YouTube 上的介绍视频：https://www.youtube.com/watch?v=8CMxDNuuAiQ ，介绍了 IPFS 的一些基本使用方法。根据我的理解，这是通过 content-address（immutable hash） 访问的分布式加密文件系统，可以通过命令行、网页界面等多种方式进行访问，有点类似 Samba，不过是分布式的。Siraj Raval 制作的一个视频：https://www.youtube.com/watch?v=BA2rHlbB5i0 ，也对 IPFS 进行了介绍，主要对 **Why** 的部分进行阐述。
    - 带宽，多个客户端对中心节点访问
    - 延迟
    - 弹性 Resiliency，中心节点失效（网络断开或者数据删除）后无法进行数据访问
    - 中心化 Centralization，主流网站掌控所有数据，用户无从得知数据的使用方式，此外会受到政府或者其他势力的干扰。
 - 使用的技术：Chord、DHT、bit swap(bittorrent mechanism)、MerkleDAG
3. **Libp2p**
  - 一个模块化的网络栈，把一系列传输协议和 peer-to-peer 协议整合在一起，方便开发者构建大型、健壮的 p2p 网络
4. **IPLD**
  - 去中心化网络（content-addressable web）的数据模型，它通过加密哈希值的方式连接了所有数据，使得数据的遍历和彼此链接更加容易。网站的示意图中连接了 bitcoin、以太坊、IPFS、Git Repo 等。
5. **Multiformats**
  - 这个项目是面向未来验证系统（future-proof systems）的协议集合， 自描述的格式可以让你的系统可以互操作和具有可升级性。
