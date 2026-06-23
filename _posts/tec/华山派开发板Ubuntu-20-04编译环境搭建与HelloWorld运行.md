---
title: 华山派开发板 Ubuntu 20.04 编译环境搭建与 HelloWorld 运行
tags:
  - embedded
  - cv1812h
  - sophon
  - tftp
  - ubuntu
categories:
  - tec
date: 2026-06-23 09:00:00
author: Hermes
---

> 基于硬十官方文档整理，从零搭建开发环境，并通过 TFTP 传输文件到开发板。

---

## 目录

1. [环境准备](#1-环境准备)
2. [虚拟机与 Ubuntu 安装](#2-虚拟机与-ubuntu-安装)
3. [共享文件夹配置](#3-共享文件夹配置)
4. [VSCode 安装与配置](#4-vscode-安装与配置)
5. [华山派编译环境搭建](#5-华山派编译环境搭建)
6. [交叉编译 HelloWorld](#6-交叉编译-helloworld)
7. [TFTP 传输并运行](#7-tftp-传输并运行)
8. [常见问题汇总](#8-常见问题汇总)

---

## 1. 环境准备

### 硬件

| 设备 | 说明 |
|---|---|
| 华山派开发板 | Type-C 供电，网口连接 |
| Windows 电脑 | 安装虚拟机软件 |
| 串口模块 | 用于串口登录开发板 |
| 网线 | 用于 TFTP 文件传输 |

### 软件版本

| 软件 | 版本 | 下载地址 |
|---|---|---|
| VMware Workstation | 16.x | [官方下载](https://www.vmware.com/) |
| Ubuntu | 20.04 LTS | [阿里云镜像](http://mirrors.aliyun.com/ubuntu-releases/20.04/) |
| SDK | sophpi-huashan | 网盘链接见下文 |

> ⚠️ **版本说明**：SDK 推荐使用 **Ubuntu 20.04 LTS**。高版本可能导致工具链不兼容。

---

## 2. 虚拟机与 Ubuntu 安装

### 2.1 VMware 新建虚拟机

1. 打开 VMware → **文件 → 新建虚拟机**
2. 选择 **典型** 配置
3. 选择 **稍后安装操作系统**
4. 客户机操作系统选择 **Linux**，版本选择 **Ubuntu 64 位**
5. 填写虚拟机名称，设置磁盘容量（建议 100GB 以上）
6. 点击 **自定义硬件**：
   - 关联 Ubuntu 20.04 ISO 镜像文件
   - 设置内存为 **8192 MB**（8GB）
   - 处理器核心数建议 **4 核以上**
7. 点击 **开启此虚拟机** 开始安装
8. 按默认选项逐步完成安装

### 2.2 安装 open-vm-tools

```bash
sudo apt update
sudo apt install open-vm-tools open-vm-tools-desktop -y
sudo reboot
```

---

## 3. 共享文件夹配置

用于 Ubuntu 与 Windows 之间互传文件。

### 3.1 VMware 设置

1. 虚拟机 → 设置 → 选项 → 共享文件夹
2. 选择 **总是启用**
3. 点击 **添加**，选择 Windows 上的文件夹

### 3.2 Ubuntu 挂载

```bash
# 创建挂载点
sudo mkdir -p /mnt/hgfs

# 挂载共享文件夹
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other

# 验证
ls /mnt/hgfs/
```

**开机自动挂载**（编辑 `/etc/fstab`）：

```bash
sudo nano /etc/fstab
# 末尾添加（uid/gid 替换为实际值）
.host:/ /mnt/hgfs fuse.vmhgfs-fuse allow_other,uid=1000,gid=1000,umask=022 0 0
```

---

## 4. VSCode 安装与配置

### 4.1 安装 VSCode

1. 打开 **Ubuntu Software**
2. 搜索 **VSCode**，点击安装

若下载失败，通过终端安装：

```bash
sudo apt install snap -y
sudo snap install --classic code
```

### 4.2 安装 C/C++ 插件

打开 VSCode，安装以下三个插件：
- **C/C++**
- **C/C++ Extension Pack**
- **C/C++ Themes**

---

## 5. 华山派编译环境搭建

### 5.1 下载 SDK

从网盘下载 `sophpi-huashan.tar.xz`：
链接: https://pan.baidu.com/s/1ouWi9yZ5pyaVqj8lvkh2YA 提取码: minq

通过共享文件夹将 SDK 转移到 Ubuntu。

### 5.2 创建并解压 SDK

```bash
# 创建工作目录
mkdir -p ~/huashan/workplace
cd ~/huashan/workplace

# 将 sophpi-huashan.tar.xz 放到此目录

# 解压 SDK（约 15 分钟）
tar -xf sophpi-huashan.tar.xz
cd sophpi-huashan

# 下载工具链（约 10 分钟）
./download_host-tools.sh

# 解压工具链
tar -xvf host-tools.tar.gz
```

### 5.3 安装编译依赖

```bash
sudo apt install -y \
    build-essential \
    libssl-dev \
    libncurses5-dev \
    bison \
    flex \
    bc \
    python3 \
    python3-pip \
    device-tree-compiler \
    u-boot-tools \
    swig \
    libpython3-dev \
    libglib2.0-dev \
    libpixman-1-dev
```

### 5.4 加载编译环境

```bash
cd cvi_mmf_sdk
source build/cvisetup.sh
```

正常显示可用板卡列表。

### 5.5 配置板卡

```bash
# 设置环境变量
export BOARD=cv1812h_wevb_0007a_emmc_huashan
export CHIP=cv1812h
export CHIP_ARCH=mars

# 执行板卡配置
defconfig cv1812h_wevb_0007a_emmc_huashan
```

> 📌 **注意**：在 Ubuntu 20.04 中，如果 `defconfig` 报错，需编辑 `build/common_functions.sh`，将 `defconfig()` 函数中的：
> ```bash
> _call_kconfig_script "${FUNCNAME[0]}" "${BUILD_PATH}/boards/${chip_arch}/${board}/${board}_defconfig"
> ```
> 改为：
> ```bash
> python3 "${BUILD_PATH}/scripts/defconfig.py" "${board}"
> ```

### 5.6 编译 SDK（约 60 分钟）

```bash
build_all
```

编译完成后，镜像文件位于：

```bash
cd install/soc_cv1812h_wevb_0007a_emmc_huashan/
ls
```

---

## 6. 交叉编译 HelloWorld

### 6.1 创建源码文件

```bash
cd ~/huashan/workplace
mkdir -p sample/helloworld
cd sample/helloworld

# 创建 helloworld.c
vim helloworld.c
```

写入以下代码：

```c
#include <stdio.h>

int main()
{
    printf("helloworld\n");
    return 0;
}
```

### 6.2 交叉编译

在 **cvi_mmf_sdk 目录下**配置工具链后，切换到 helloworld 目录编译：

```bash
# 进入 SDK 环境
cd ~/huashan/workplace/sophpi-huashan/cvi_mmf_sdk
source build/cvisetup.sh
defconfig cv1812h_wevb_0007a_emmc_huashan

# 进入 helloworld 目录
cd ~/huashan/workplace/sample/helloworld

# 交叉编译
riscv64-unknown-linux-musl-gcc helloworld.c -o helloworld_musllibc \
    -march=rv64imafdcvxthead -mcmodel=medany -mabi=lp64d
```

编译后生成可执行文件 `helloworld_musllibc`。

---

## 7. TFTP 传输并运行

### 7.1 Windows TFTP 服务端配置

1. 下载并安装 Tftpd64
2. 配置：
   - `Current Directory`：选择存放文件的文件夹
   - `Server interfaces`：选择电脑 IP（如 `192.168.150.3`）
3. 确保 Windows 防火墙放行 Tftpd64

### 7.2 硬件连接

| 连接 | 说明 |
|---|---|
| Type-C | 供电 |
| 串口模块 | 电脑连接开发板，用于串口登录 |
| 网线 | 电脑网口连接华山派网口 |

华山派默认 IP：**192.168.150.2**

### 7.3 通过共享文件夹转移文件

1. 将 `helloworld_musllibc` 复制到共享文件夹（`/mnt/hgfs/`）
2. 在 Windows 中进入共享文件夹
3. 打开终端（CMD 或 PowerShell）

### 7.4 使用 TFTP 传输到开发板

**在开发板串口终端中执行**：

```bash
# 进入可写目录
cd /mnt/data

# 从电脑下载文件（IP 替换为电脑 IP）
tftp -g -r helloworld_musllibc 192.168.150.3

# 查看文件
ls -la

# 添加执行权限
chmod 777 helloworld_musllibc

# 运行
./helloworld_musllibc
```

正常输出 `helloworld`。

### 7.5 开发板目录说明

| 目录 | 用途 |
|---|---|
| `/mnt/data` | 数据分区，可读写，推荐存放程序 |
| `/tmp` | 临时目录，重启后清除 |

---

## 8. 常见问题汇总

### Q1：TFTP 传输失败 Read-only file system

**解决**：切换到 `/mnt/data` 或 `/tmp` 目录。

### Q2：TFTP 连接不上

**检查清单**：

- [ ] Windows 防火墙是否放行 Tftpd64
- [ ] Tftpd64 的 `Server interfaces` 是否选择了正确的 IP
- [ ] 开发板能否 ping 通电脑 IP
- [ ] 网线是否连接正常

### Q3：`defconfig` 执行失败

Ubuntu 20.04 中可能需要修改脚本，参考 [5.5 节](#55-配置板卡)修改 `build/common_functions.sh`。

### Q4：编译 `build_all` 报错

确保已安装所有依赖，并确认 Ubuntu 版本为 20.04。

---

## 参考资料

- 硬十华山派课程：https://www.hw100k.com/
- SOPHON CV1812H 官方文档：https://sophon.cn/
- Tftpd64 官网：https://tftpd64.jounin.net/

---

*最后更新：2026-06-23*
