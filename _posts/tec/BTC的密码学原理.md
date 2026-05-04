---
title: BTC的密码学原理
date: 2025-03-31
tags: ['web3']
categories: ['技术']
author: QuHou
---

## 主要用到两个密码学原理：签名和哈希

### 哈希

交易的过程并不加密，交易账号的地址和金额公开透明

该过程会产生哈希碰撞
`h(x) = h(y)`
一般无法找到两个会产生哈希碰撞的 x 和 y

collision resistance
指的是无法找到一个(m)'与原始的 h(m)相同

理论上 collision resistance 是不存在的
但在实践上 collision resistance 被验证正确

有被验证错误的哈希函数

哈希函数有 hiding 性质：即无法从`h(x) -> x`

实现 hiding 的关键是 x 取值足够大且分布足够均匀

digital equivalent of a sealed envelope

根据哈希的 hiding 性质，可以用 hash 作为 sealed envelope

puzzle friendly 是指事先无法知道哪个数值会算出此 hash 值

挖矿的原理：不断的暴力检索，使得 H(block header) <= target

例如只有 xxxx 的区域才是正确的 成为 target space

`xxxx|||||||||||||||||||||||||||`

挖矿很难，但是验证很简单，验证时只需要根据 block header 算一次 hash 即可

比特币用的 hash 函数是：SHA-256：`Search Hash Algorithm`

### 签名

使用加密货币需要拥有一个账户，可以自己开户，是去中心化的

public key，private key

f

### BTC 协议

数字货币的协议

double speeding attack 双花攻击

### 数据结构

要说明支付方钱的来源

哈希指针指向币的来源

一场交易中 A->B
所有人都要知道 A 的公钥，以验证 A 的币的来源的合法性

A 的公钥由铸币提供

block header

version
hash of previous block header
merkle root hash
target
nonce

block body

transaction list

账本内容

distributed consensus 分布式共识
distributed hash table 分布式哈希表

FLP impossibility result

在一个 asynchronous 异步系统中，如果时延没用上线，那只要有一个成员 faulty 了，那么就打不成共识

CAP Theorem

三者只能取其二

1. Consistency
2. Availability
3. Oartition tolerance


Paxos 协议

1. √


投票权

memership