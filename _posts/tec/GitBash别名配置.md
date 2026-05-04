---
title: GitBash别名配置
date: 2025-03-25
tags: ["技术笔记"]
categories: ['技术']
author: QuHou
---



1.打开Git Bash，查看本机根目录下是否存在.bash_profile文件，若不存在，则创建该文件：

2.编辑.bash_profile文件，在文件末尾添加以下内容：

``` shell
alias cls='clear'
alias rl='source ~/.bash_profile'
alias bc='code ~/.bash_profile'

alias i='ni'
alias d='nr dev'

alias c='code'
alias cc='cd E:'

alias rm='rm -rf'

alias e='explorer.exe .'

alias ..='cd ..'
alias ...='cd ../../'
alias ....='cd ../../../'
alias .....='cd ../../../../'
```

3.保存并关闭.bash_profile文件。

4.重新打开Git Bash，输入命令：

``` shell
source ~/.bash_profile
```

即可生效。