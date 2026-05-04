---
title: 如何为vitepress导出的data数据添加类型提示
date: 2025-03-28
tags: ['vitepress']
categories: ['技术']
author: QuHou
excerpt: 用declare声明data数据，然后显示导出data
---

# 核心代码：

```js
declare const data: Archive[]
export { data }
```
