---
title: call,apply,bind问题
date: 2025-03-26
tags: ["面试题"]
categories: ['技术']
author: QuHou
excerpt: 三者都可以改变this指向，apply只能传数组，bind不会立刻调用函数而是返回新函数，并且this不可以再被改变
---

```js
<script>
  function fn(...params) {
    console.log(this, params)
  }

  const obj = {
    name: '小明',
  }
  fn()
  fn.call(obj, 'hello', 1, 23)
  fn.apply(obj, ['hello', 1, 23])
  fn.bind(obj, 'hello', 1, 23)()
</script>

```