---
title: sql中like语句的处理问题
date: 2025-04-03
tags: ['nodejs', 'sql']
categories: ['技术']
author: QuHou
---

在写 sql 语句时，如果涉及 like 查询，需要在 values 里写具体的匹配语法，不能直接在字符串里写

```nodejs
 const rows = await query(
      'SELECT * FROM students WHERE `name` LIKE ? OR `grade` LIKE ?',
      [`%${keyword}%`, `%${keyword}%`]
)
```
