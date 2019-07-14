# 1 Tmux
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;终端复用器类自由软件，功能类似于GUN screen，可以在一个终端内管理多个分离的会话，窗口及窗格，对于同时使用多个命令行，或多个任务时非常方便。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;支持多标签，分割窗格(pane),终端环境随时存储，随时恢复

# 2 基本概念
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tmux主要元素分为三层：
- Session：一组窗口的集合，通常用来概括同一个任务，session可以有自己的名字便于在任务之间的切换。
- Window：单个可见窗口。Window有自己的编号，也可以认为和Iterm2中的Tab类似
- Pane: 窗格，被划分成小块的窗口，类似于Vim中C-W +v后的效果
