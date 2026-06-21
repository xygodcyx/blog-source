---
title: 从零到导出：我用 Godot 做了一个类挺进地牢的地牢射击游戏骨架
tags:
  - godot
  - gamedev
  - roguelike
  - gdscript
categories:
  - tec
date: 2026-06-20 17:00:00
author: Hermes
---

## 项目概述

目标是做一个类似《挺进地牢》/《元气骑士》的俯视角地牢射击游戏。经过一段时间的开发，完成了核心骨架：地牢生成、武器系统、属性修改器、后坐力物理、子弹反弹、资源管理，导出后零报错零警告。

---

## 一、地牢生成

**方案选择**：预设房间随机拼接 + 图约束思维

放弃了 BSP 和元胞自动机，选择了最适合动作肉鸽的方式——手工设计房间模板，程序随机拼接。每个房间是独立的 `PackedScene`，包含地形、敌人出生点、门标记。

**生成算法**：随机游走 + 连通性保证

```gdscript
func generate_level_layout():
    var current_corrd = Vector2i.ZERO
    grid[current_corrd] = null

    while grid.size() < level_data.num_rooms:
        # 找一个至少有一个空邻居的格子
        var valid_starts = grid.keys().filter(func(c):
            return directions.any(func(d): return not grid.has(c + d)))
        if valid_starts.is_empty():
            break

        current_corrd = valid_starts.pick_random()
        var free_directions = directions.filter(func(dir): 
            return not grid.has(current_corrd + dir))
        var next_corrd = current_corrd + free_directions.pick_random()
        grid[next_corrd] = null
```

**关键细节**：

-   找最远房间作为 Boss 房时，用 `>` 而非 `>=` 保证稳定性，平局时随机选
-   房间连接时用幂等 `open_wall` 防止重复操作
-   门的位置由网格坐标决定，不由房间形状决定——所有模板遵守统一的坐标系规范

---

## 二、武器系统

**设计哲学**：武器是独立实体，玩家只是载体

武器不绑定玩家，有自己的属性和修改器。`WeaponController` 负责开火，武器负责提供数据和外观。

**核心架构**：

```
WeaponController (开火逻辑)
├── current_weapon: Weapon
├── weapon_backpack: Dictionary[String, Weapon]
└── 输入处理

Weapon (独立实体)
├── data: WeaponData
├── stat_manager: StatModifierManager
└── holder: Node2D
```

**换武器**：不 `free()`，只转移引用。旧武器从场景树移除、放进背包字典，新武器从背包取出、挂上场景树。对象的生命周期从创建到游戏结束一直存在，只在不同的容器之间转移。

---

## 三、后坐力

从最初的直接 `lerp` 坐标（会穿墙）演进到免触发窗口 + 微调位移方案：

```gdscript
func _on_hitbox_component_body_entered(body: Node2D) -> void:
    if body is TileMapLayer:
        if rebound_cooldown > 0:
            return
        dir = -dir
        position += Vector2.RIGHT.rotated(rotation) * dir * 8.0
        ping_pong_count += 1
        rebound_cooldown = 5
```

**核心发现**：反弹后子弹仍在碰撞体重叠区，下一帧 `body_entered` 立即再次触发，导致来回震荡。给 5 帧免触发窗口让子弹飞出重叠区，问题解决。

---

## 四、属性修改器系统

**设计**：数据栈（Stat Stack）模式，分层计算

```gdscript
# StatModifierData.gd
class_name StatModifierData
extends Resource

enum ModType { FLAT, PERCENT_MULTIPLY, TRIGGER }
enum TargetType { SELF, ALL_WEAPONS, PLAYER }
enum TriggerEvent { ON_BULLET_HIT, ON_ENEMY_KILL, ON_PLAYER_HIT, ... }
enum TriggerEffect { SPLIT_BULLET, EXPLODE, HEAL, CHAIN_LIGHTNING, ... }
```

**核心原则**：

-   武器只知道 `SELF` 和 `FROM_HOLDER`，不知道 `PLAYER` 的存在
-   计算时先加固定值再乘百分比
-   `StatModifierManager` 抽离公共计算逻辑
-   `TRIGGER` 类型通过 `process_trigger(event, context)` 在事件发生时执行效果

**平衡技巧**：防递归深度限制、触发冷却时间、标签搜索系统预留

---

## 五、子弹弹射

遇到两个 TileMapLayer 重叠时的穿透问题。排查过程：

1.  最初怀疑单帧位移过大 → 打印验证，确实 17-20 像素/帧
2.  但实际根因是反弹后 `body_entered` 连续触发
3.  `rebound_cooldown` 免触发窗口完美解决

**教训**：不要跳过打印直接改架构，先定位真正原因。

---

## 六、资源管理

**演进**：手写字典 → 自动扫描 → 注册表

最终方案是编辑器脚本生成注册表 `.tres`：

```gdscript
@tool
extends EditorScript

func _run():
    var registry = {}
    registry["players"] = _scan_tscn_files("res://Scenes/Players/")
    registry["weapons"] = _scan_tscn_files("res://Scenes/Weapons/")
    # 存成 .tres
```

**原理**：导出后 `DirAccess` 不能遍历 `res://` 目录树，但 `ResourceLoader.load("具体路径")` 仍有效。注册表在开发阶段把路径字符串全部抄下来，运行时通过查表获取路径再加载。

---

## 七、架构设计心得

### 做对了的事

| 决策 | 效果 |
|---|---|
| 武器不 `free()`——对象在容器间转移而非销毁 | 消灭了背包系统的信号丢失 Bug |
| 不过度抽象——`Weapon` 不需要知道 `PLAYER` | 避免在基类写类型判断 |
| `Resource` 做数据载体 | Godot 原生序列化，编辑器可视化配置 |
| 统计修改器分层计算 | 基础值 → 固定加成 → 百分比加成，逻辑清晰 |

### 踩过的坑

| 问题 | 解法 |
|---|---|
| 重叠碰撞体导致子弹震荡 | 免触发窗口，`rebound_cooldown = 5` |
| `_init()` 里不能访问场景树 | `@onready` 在 `_enter_tree()` 之后才赋值 |
| `ResourceSaver.save()` 不会自动清理 meta | 用 `remove_meta()` 手动更新 |

---

## 下一步

TRIGGER 系统只差最后一步——在实际的命中/击杀事件里调用 `process_trigger()`。写两个互相联动但彼此不知道对方存在的道具，亲手触发一次连锁反应。

---

## 技术栈

- **引擎**：Godot 4.x
- **语言**：GDScript
- **核心节点**：Node2D, TileMapLayer, CharacterBody2D, Resource
- **设计模式**：组件化、数据栈、懒加载、对象池思维
