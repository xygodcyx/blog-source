---
title: godot多人同步中is_multiplayer_authority的理解
tags:
  - 'godot'
  - '游戏开发'
categories:
  - ''
date: 2026-05-31 13:55:30
---

最近笔者在开发一款多人联机游戏，其中频繁出现is_multiplayer_authority()函数

在此之前其实自行实现过多人游戏的完整链路开发，但对godot的上层实现很是好奇因为它封装了大量的底层逻辑，以至于在使用时完全感觉不出来自己有在做“同步”工作，所以现在写下一篇笔记记录一下心得

---

## 理解multiplayer_peer

这是多人游戏鉴权的核心，每个游戏玩家（peer）都会生成一个唯一的unique_id，其中充当服务端的玩家unique_id为1，其它玩家的unique_id为随机数

authority则是每个节点的默认属性初始为1，会被继承

is_multiplayer_authority会判断authority和unique_id是否一致，一致说明是权威客户端（一般是服务器）

有时会将权威下放，比如在收集移动时，通常会单独使用一个MultiplayerSynchronizer作为同步玩家输入
并对MultiplayerSynchronizer进行set_multiplayer_authority操作，其中authority被设置为每个客户端的unique_id，这样就实现了每个客户端都有独立的输入控制，但实际移动的操作（改变速度、根据输入改变方向等等）还是只有服务器有权限，因为只有它是authority==unique_id

至于其它节点的鉴权也是一样的，所有的节点都有authority，会有unique_id进行比对，也都能知道当前是不是权威客户端（当前一般都是服务器，除非手动进行了set_multiplayer_authority操作）

而rpc调用则会根据修饰符的条件进行调用："authority","call_local","unreliable"， 就表示只能在权威客户端调用（一般是服务器），在本地调用，可以不可靠
