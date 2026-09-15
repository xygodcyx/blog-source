---
title: Arch系统下使用微信时fcitx5输入法的缩放非常小的解决方法
tags:
  - 'Linux'
  - 'Arch'
categories:
  - 'Linux'
date: 2026-09-15 14:05:35
---

编辑`~/.Xresources`文件，输入以下内容：
``` xml
"Xft.dpi" "192"
```
运行`xrdb -merge ~/.Xresources`使配置生效

在`niri`或其它桌面管理器的配置文件里设置启动参数：

```sh
spawn-at-startup "xrdb" "-merge" ".Xresources"
```

问题解决
