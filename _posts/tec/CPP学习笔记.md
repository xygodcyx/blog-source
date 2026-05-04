---
title: CPP学习笔记
tags:
  - 'CPP'
categories:
  - '技术'
date: 2025-09-17 15:55:46
---

以实现植物大战僵尸为目标学习C++

---

## 类的使用

### 头文件应如何定义

是否要重写父类的定义的函数这一决策要在头文件里完成，.cpp只负责实现.h里定义的函数

`Plant.h`:

```c
{% raw %}
#pragma once

#include "Entity.h"

class Plant : public Entity
{
public:
 explicit Plant(const BaseData& baseData);
 void init(int plantType, int x, int y, int frameCount) override;
 void update() override;
 void updateAnim() override;
 void mainAttackTimer() override;
 void mainAttack() override;
};
{% endraw %}
```

所以Plant.cpp需要实现Entity定义的所有方法

但SunFlower并不需要，所以`SunFlower.h`可以写成：

```c
{% raw %}
#pragma once

#include "Plant.h"

class SunFlower : public Plant
{
public:
 SunFlower();
 void mainAttack() override;
};
{% endraw %}
```

这样一来，SunFlower.cpp只需要实现最终需要关心的mainAttack函数即可，其它植物也只需要实现自己的mainAttack，比如生产阳光、发射豌豆等等
