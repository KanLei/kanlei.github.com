---
layout: post
title: "拜占庭将军问题与共识算法"
description: ""
category: 
tags: [blockchain]
---
{% include JB/setup %}


### 内容简介

本文主要解释了什么是拜占庭将军问题，以及如何解决拜占庭将军问题；什么是共识算法，不同共识算法之间的区别是什么。

### 两个将军问题

在介绍拜占庭将军问题之前，我们先来了解一下两个将军问题。

> 两只军队驻扎在山谷的两侧，他们共同的敌人则驻扎在山谷的中间，两个将军必须带领军队同时出击方能击败山谷中的敌军，只有一方进军则必定失败。两个将军之间交流完全依赖于信使，且山谷是信使们传递消息的必经之路，那么两个将军攻击敌人的时间达成一致呢？

在计算机网络中中，该问题被描述为，两个结点在不安全的信道环境下如何达成一致？最终，两个将军问题被证明是无法解决的问题，由于通信链路的不安全性，无法保证多个实体间的最终一致性。

### 拜占庭将军问题

拜占庭将军问题则是在两个将军问题之上的延伸，首先，拜占庭将军问题假设信道是安全的，而将军中有可能出现叛徒。问题描述如下：

> 一组拜占庭将军率领军队围困一座城堡，至少有 n/2 + 1 位将军共同出击才能击败城堡中的敌军，否则即为失败。将军中存在叛徒，叛徒可能会传递假的消息给其他的将军。如何保证最终的攻击不会失败？

在分布式计算中，该问题被描述为不同的结点之间通过交换信息达成共识，而当结点发生错误或者传递出错误消息时，如何才能保证整个系统的一致性？


### POW

POW(Proof of Work)是比特币系统给出的解决共识问题的方案。在比特币系统中，Satoshi Nakamoto 提出了一种让所有计算结点通过消耗计算资源完成一道计算题目，从而提升结点伪造的困难度，前提是假设全网中诚实的结点掌握着绝大多数算力，否则即会出现 51% 攻击。

首先比特币网络中的某一个结点生成一笔交易并广播到全网，同时挖矿结点在不断接收来自网络上的交易，验证交易合法性之后会将它们添加到一个区块中，结点会通过消耗计算资源得到一个HASH值用于表示自己的工作量，接着将该区块传播给全网，网络中的其它结点接收到该区块并校验，验证通过后则将其链接到当前的链上，从而构成了一条由区块链接而成的区块链。

#### 双花攻击

双花攻击即一笔交易被花费两次，当攻击者掌握着全网51%算力时，即有可能造成双花攻击。在比特币之前也存在一些电子现金货币的尝试，但都依赖于中心机构来保证不会出现双花问题，而比特币提出了一种以去中心化的方式，通过对每个区块中增加时间戳，来保证时间最早的花费才是有效的交易。

假设攻击者发送 100BTC 给商户，这笔交易被矿工确认并打包到区块1024中，在大约经过1小时，区块1024之后又新增了5个区块，这时商户确认收到 100BTC，然后将物品发给攻击者，当攻击者收货物品后立即又发送了一笔新的交易，将 100BTC 发送给自己，这笔交易不会通过挖矿节点验证，因为攻击者的账户中已经没有这 100BTC，攻击者因此选择分叉，重新构造一个包含这笔交易的区块1024，并将其链接到1023区块之后，这时存在两条链，但挖矿节点并不会切换到攻击者分叉的这条链上，因为矿工们会选择最长的链作为标准，除非攻击者拥有全网51%的算力，才有可能让新分叉链的长度追赶上原始链的长度。

#### 分叉问题

由于挖矿是由全网结点共同进行的，因此难免会出现某一时刻不同的挖矿结点会同时挖出最新的区块，并广播给网络中的其余结点，这时就形成了分叉。比特币系统采用的方案时，所有的结点会自动切换到工作量最多的链上，即最长的分叉，由于两条分叉上的算力不是平衡的，所以最终保证只有最长的一条链有效。

#### 存储容量



#### 交易速度

比特币的交易速度目前是平均每10分钟一个区块，大概xxx笔交易，交易速度过慢也是比特币被诟病的原因之一。由于出块时间较长，所以很多项目尝试调整出块速度，比如以太坊，以太坊的出块速度约[17秒](https://etherscan.io/chart/blocktime)，但以太坊不单单是调整了区块速度，以太坊旨在创建一个可编程的分布式网络，能够在区块链上实现应用程序，而不是仅仅是用于转账。

### dBFT

> dBFT 全称 Delegated Byzantine Fault Tolerant，是一种通过代理投票来实现大规模节点参与共识的拜占庭容错型共识机制。NEO 管理代币的持有者通过投票，可以选出其所支持的记账人。随后由被选出的记账人团体通过 BFT 算法，来达成共识并生成新的区块。投票在 NEO 网络持续进行而非按照固定任期。
 
dBFT 的优势在于不会产生分叉，所有节点会针对某一区块会达成最终一致，但其交易速度依然有限。




[*Two Generals' Problem*](https://en.wikipedia.org/wiki/Two_Generals%27_Problem)  
[*Byzantine fault tolerance*](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance#Byzantine_Generals'_Problem)  
[*Bitcoin*](https://bitcoin.org/bitcoin.pdf)  
[*dBFT*](https://steemit.com/neo/@basiccrypto/neo-s-consensus-protocol-how-delegated-byzantine-fault-tolerance-works)  
[*Ethereum White Paper*](https://github.com/ethereum/wiki/wiki/White-Paper)