---
title: tmux + tmuxifier 打造终极Linux运维终端工具
description: 分享tmux和tmuxifier基本知识及我的用法。
toc: true
categories:
  - linux
tags:
  - tmux
  - tools
date: 2017-04-09 20:40:02
updated: 2017-04-09 20:40:02
---

# tmux + tmuxifier 打造终极Linux运维终端工具

对于日常需要操作和管理大量线上机器的我们，有时候在一个终端上远程和操作大量线上服务器的时候是件头疼而繁琐的事情。虽然平时有自己的脚本简化远程服务器操作，但平时在Linux系统终端中使用时仍然仍然很是蛋疼（相信用linux电脑工作的或多或少都体会过：打开机器，不断的开终端，不断的敲远程命令，选择终端时还要不断的点鼠标找窗口等等。。。），特别在临时处理问题时感觉特浪费时间在这些上。最近在想针对windows的Xshell软件的终端功能，花了一些时间研究了一下在Linux上类似的使用，我通过tmux+tmuxifier的结合解决了需求（网上还有比较主流的tmux+tmuxinator），使用tmuxifier是因为这个是纯bash脚本程序。本文分享下tmux和tmuxifier基本知识及我的用法。

## tmux简介和使用
tmux是screen的替代品。

官网： https://tmux.github.io/

### ◆ tmux结构

tmux使用C/S模型，包含概念：
- server：首次运行tmux命令后即启动一个tmux服务，在所有session结束之前一直处在提供服务状态，client实际通过socket方式连接server。一个server服务可以包含多个session。
- session：一个session可以包含多个window，是一组窗口的集合，通常用来概括同一个任务或一个项目。session可以有自己的名字便于任务之前的切换。
- window：单个可见窗口。window有自己的编号，一个window中可以有多个pane。
- pane：窗格，在window中被划分成的小块的窗口。
- client：使用tmux命令连接到server中的虚拟终端。多个client可以连接到同一个server中，甚至是同一个session、window或pane中。

开始盗图了~~

![tmux_cs_pic.png](http://images.cnblogs.com/cnblogs_com/itech/linux/tmux.png "tmux C/S结构图")

![tmux_view_pic.jpg](http://7xqkau.com1.z0.glb.clouddn.com/16-4-5/3631366.jpg "tmux界面结构图")

### ◆ 配置
tmux默认会先从/etc/tmux.conf加载系统级的配置项，然后从~/.tmux.conf 加载用户级的配置项。

如果使用编译安装则，默认在PREFIX/etc中。

也可以使用参数-f指定一个配置文件。
```sh
tmux -f /path/to/tmux.conf
```

可以在man帮助中获取OPTIONS进行配置。

__tmux.conf__

```
# 提示信息的持续时间；设置足够的时间以避免看不清提示，单位为毫秒
set-option -g display-time 5000

# 窗口的初始序号；默认为0，这里设置为1
set-option -g base-index 1

# 状态栏右方的内容；这里的设置将得到类似23:59的显示
set-option -g status-right "#(date +'%F %H:%M')"

# 支持鼠标
#set-option -g mouse on

```

### ◆ tmux使用

__通过以下方式获得使用帮助：__

（1）Shell命令(COMMANDS)：

```bash
man tmux
tmux lscm
tmux <subcmd> --help
```

（2）窗口及窗格快捷键命令(KEY BINDINGS)：

```
按住前缀激活指令后放开，然后按？
默认：Ctrl+b ?
```

__常用操作__

（1）session操作

```
# COMMANDS 新建session
tmux
tmux new-session -s [session-name] [-n window-name] [command]

# KEY BINDINGS 新建session
C-b : new-ssession
C-b : new-session -s SESSION-NAME [-n WINDOW-NAME] [command]


# COMMANDS 显示session信息
tmux ls

# KEY BINDINGS 显示/选择session信息
C-b s 方向键选择+Enter


# COMMANDS 连接session
tmux a
tmux attach-session [-t target-session]


# KEY BINDINGS rename session
C-b $ 输入新的SESSION名称+Enter


# COMMANDS 关闭session
tmux kill-session [-aC] [-t target-session]

# KEY BINDINGS 关闭session
C-b : kill-session
```

（2）window操作

```
# COMMANDS 新建window
tmux new-window [-n window-name] [-t target-window] [command]

# KEY BINDINGS 新建window
C-b c
C-b : new-window [-n window-name] [-t target-window] [command]


# COMMANDS 显示window信息
tmux lsw [-a] [-F format] [-t target-session]

# KEY BINDINGS 显示window信息
C-b : lsw


# KEY BINDINGS 查找/选择window
C-b w 方向键选择+Enter
C-b ' 输入WINDOW名称+Enter
C-b f 输入WINDOW名称+Enter
C-b l
C-b n
C-b p

# KEY BINDINGS rename window
C-b , 输入新的WINDOW名称+Enter

# KEY BINDINGS 关闭window
C-b & 回答y/n?
```

（3）pane操作

```
# KEY BINDINGS 切分创建pane
C-b %
C-b "


# KEY BINDINGS 显示pane编号信息
C-b q


# KEY BINDINGS 选择pane
C-b o
C-b ;
C-b 方向键


# KEY BINDINGS 置换pane
C-b {
C-b }

# KEY BINDINGS 关闭pane
C-b x 回答y/n?
```

## tmuxifier简介和使用
tmuxifier是基于bash的tmux会话管理工具。

Github: https://github.com/jimeh/tmuxifier 

### ◆ tmuxifier命令

tmuxifier在安装和配置文成后，直接是shell命令：

```bash
usage: tmuxifier <command> [<args>]

Some useful tmuxifier commands are:
   <command>      <alias>
   load-session   s        Load the specified session layout.
   load-window    w        Load the specified window layout into current session.
   list           l        List all session and window layouts.
   list-sessions  ls       List session layouts.
   list-windows   lw       List window layouts.
   new-session    ns       Create new session layout and open it with $EDITOR.
   new-window     nw       Create new window layout and open it with $EDITOR.
   edit-session   es       Edit specified session layout with $EDITOR.
   edit-window    ew       Edit specified window layout with $EDITOR.
   commands                List all tmuxifier commands (including internal).
   version                 Print Tmuxifier version.
   help                    Show this message.

See 'tmuxifier help <command>' for information on a specific command.

```

tmuxifier结合session/window配置文件可以很灵活的运用在终端shell和tmux服务中。

### ◆ tmuxifier的会话和窗口配置

tmuxifier通过layouts配置文件来保存session和window在tmux服务终端的行为状态。

layouts配置文件默认保存在tmuxifier_home下的layouts目录中。

可以使用TMUXIFIER_LAYOUT_PATH进行指定该目录：

```bash
export TMUXIFIER_LAYOUT_PATH="$HOME/.tmux-layouts"
```

layouts可以使用如下命令创建：

```bash
tmuxifier ns SESSION-NAME
tmuxifier nw WINDOW-NAME
```

执行命令后，tmuxifier会通过shell环境变量EDITOR设置的命令打开layouts模版配置文件提供编辑。EDITOR可以指定为vim等：export EDITOR=vim

创建的layouts配置文件为shell脚本文件保指定的layouts目录中，会在tmuxifier的load的命令中被source执行。
- session的layouts配置文件为：SESSION-HOME.session.sh
- window的layouts配置文件为：WINDOW-NAME.window.sh

最后，用于创建layouts配置文件的模版文件在tmuxifier_home下的templates目录中，可以预先编辑session.sh和window.sh这两个模版文件来定制模版。

## tmux + tmuxifier 打造Linux终端工具

我通过tmux+tmuxifier打造的终端工具是这样的：

__远程机器连接管理__
- 使用tmuxifier配置线上服务器的session和window的layout。
- session主要用于在本地shell终端中一次性打开整个项目的所有服务器，以方便进行快速操作和切换机器。
- window主要用于在tmux服务终端切分pane或新建window时快速连接远程机子，我session的配置都会基于加载window的配置。

__日常使用__  
- 平常远程机子操作都在tmux终端中进行，所有工作会话窗口都在tmux中进行管理。
- 本地shell终端可以随时退出和连接tmux服务，也可以多个终端同时连接tmux服务中，甚至是连到同个window或pane。
- 使用tmuxifier命令来管理和连接远程机子终端。session用来批量开启远程机器终端，window用来辅助在tmux终端中快速连接服务器。

__layout templates__

session.sh

```bash
# Set a custom session root path. Default is `$HOME`.
# Must be called before `initialize_session`.
#session_root "~/Projects/{{SESSION_NAME}}"

if initialize_session "{{SESSION_NAME}}"; then

  # 创建新window并替换local shell的方式
  #new_window "hdfs79_132" "ssh 219.153.36.79"

  # 读取window方式连接服务器配置方式
  new_window "tmux_local"
  load_window "WINDOW_NAME1"

  new_window "tmux_local"
  load_window "WINDOW_NAME2"

  select_window 1
fi
finalize_and_go_to_session
```

window.sh

```
# Set window root path. Default is `$session_root`.
# Must be called before `new_window`.
#window_root "~/Projects/{{WINDOW_NAME}}"

tmuxifier-tmux rename-window "{{WINDOW_NAME}}"
# 在当前pane中执行 用于创建链接,前面添加exec可替换当前shell
tmuxifier-tmux send-keys "ssh IP"
tmuxifier-tmux send-keys "C-m"

# Split window into panes.
#split_v 20
#split_h 50

# ----- Set active pane. -----
#select_pane 0
```

注意：这样配置的layout只能这么用，不然在tmux服务终端中执行可能会有问题：
```tmuxifier s session-name```命令只能在本地终端中使用,加载命令时，如果已有同名tmux session则跳转至该名称的session中而不是创建新的session。
```tmuxifier w window-name```命令只能在tmux终端中使用。

完~~


# 参考资料

http://www.cnblogs.com/itech/archive/2012/12/17/2822170.html  
http://www.maxincai.com/post/tmux/  
http://blog.jobbole.com/87278/  
http://blog.jobbole.com/87584/  
http://www.kancloud.cn/kancloud/tmux/62459  


