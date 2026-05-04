---
title: http请求的实现
date: 2025-04-01
tags: ['浏览器', 'http']
categories: ['技术']
author: QuHou
excerpt: http协议
---

```js Request类的构造函数
class Request {
  constructor(options) {
    this.method = options.method || 'GET'
    this.host = options.host
    this.path = options.path || '/'
    this.port = options.port || 80
    this.headers = options.headers || {}
    this.body = options.body || {}

    if (!this.headers['Content-Type']) {
      this.headers['Content-Type'] = 'application/x-www-form-urlencoded'
    }

    if (this.headers['Content-Type'] === 'text/json') {
      this.bodyText = JSON.stringify(this.body)
    } else if (
      this.headers['Content-Type'] === 'application/x-www-form-urlencoded'
    ) {
      this.bodyText = Object.keys(this.body)
        .map((k) => `${k}=${encodeURIComponent(this.body[k])}`)
        .join('&')
    }

    this.headers['Content-Length'] = this.bodyText.length
  }
}
```
