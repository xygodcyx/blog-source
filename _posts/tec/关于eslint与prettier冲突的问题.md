---
title: 关于eslint与prettier冲突的问题
date: 2025-04-12
tags: ['坑']
categories: ['技术']
author: QuHou
excerpt: eslint配置与prettier格式化的冲突
---

# 问题描述
当使用eslint时又使用了prettier就有可能会发生配置冲突的问题
此时可以关闭prettier直接使用eslint的格式化配置即可
# 解决方案
``` json
"editor.codeActionsOnSave": {
    "source.fixAll.eslint": "always" // 启用 ESLint 自动修复
}
```
