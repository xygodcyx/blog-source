---
title: this指向问题
date: 2025-03-26
tags: ['面试题']
categories: ['技术']
author: QuHou
excerpt: 普通函数谁定义this指向谁，箭头函数无this，在哪调用this指向谁
---

```js
<script>
  class C1 {
    constructor() {
      window.setTimeout(function () {
        // 普通函数的this执行遵循谁调用指向谁的原则
        console.log(this)
      })
      window.setTimeout(() => {
        // 箭头函数没有this，会向上找this
        console.log(this)
      })
    }
  }
  new C1()
</script>
```

