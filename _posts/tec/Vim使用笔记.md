---
title: Vim使用笔记
date: 2025-06-30
tags: ['vim', '效率工具']
categories: ['技术']
author: QuHou
excerpt: vim学习笔记
---

# vim 使用笔记

## 基本的模式切换：

normal 模式：光标移动模式
insert 模式：字符插入模式
visual 模式：框选模式

## 光标的移动

```text
f{char} 光标跳到下一个{char}所在位置
F{char} 光标跳到上一个{char}所在位置
t{char} 光标跳到下一个{char}的前一个字符的位置
T{char} 光标跳到上一个{char}的后一个字符的位置
;重复上次查找动作
,反向重复上次查找动作
```

## 操作符

```text
d(delete) 删除

c(change) 修改（删除并进入模式）

y(yank) 复制

v(visual) 选中并进入visual模式

反向重复上次查找动作
```

## 动作

i(inner),a(around)
动作是和操作符一起执行的,先执行操作符,然后执行动作(框选字符)

## 大小写切换

~ 对光标所在的字母进行大小写切换
{number}~ 对{number}个字符进行大小写切换
g~~ 对整行代码进行大小写切换
gUU 将整行代码转为大写
guu 将整行代码转为小写
guiw 将标所在的单词转为小写
gUiw 将光标所在的单词转为大写

## easymotion
两次leader + s : 全局搜索模式
两次leader + f : 向下搜索
两次leader + F : 向上搜索
......(跟光标的移动的命令一致)

## vim-surround
快速替换成对出现的字符('',"",{},[],()等等)
ysiw" 给一个单词的周围插入""
ds" 在周围删除"
cs"t 把"替换成html标签
