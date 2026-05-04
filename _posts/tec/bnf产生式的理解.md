---
title: bnf产生式的理解
date: 2025-04-07
tags: ['编译原理', 'bnf']
categories: ['技术']
author: QuHou
excerpt: bnf产生式是为了厘清表达式的层级关系，将其层层拆解，一点点的提取出要运行的表达式
lang: en
---

四则运算的 BNF 产生式

```js
<expression> ::= <term>
                | <expression> "+" <term>
                | <expression> "-" <term>

<term>       ::= <factor>
                | <term> "*" <factor>
                | <term> "/" <factor>

<factor>     ::= <number>
                | "(" <expression> ")"
```

这两句描述决定了四则运算的计算顺序是乘除优先，加减次之，因为遇到+/-运算符后会分割左右两侧作为表达式递归解析，相应的，`"(" <expression> ")"` 也就决定了括号的运算符最高，因为在遇到括号时会将括号内的表达式单独作为一个factor解析

可以知道，处于bnf树级结构的最低端的表达式，优先级就越高
``` bnf
<expression> "+" <term>
<expression> "-" <term>
```

例子：3 + 5 \* (10 - 6 / 2)

过程：

``` bash
expression → expression "+" term
            → 3 + term
            → 3 + (term "*" factor)
            → 3 + (5 * factor)
            → 3 + (5 * ( "(" expression ")" ))
            → 3 + (5 * (10 - term))
            → 3 + (5 * (10 - (term "/" factor)))
            → 3 + (5 * (10 - (6 / 2)))
```
