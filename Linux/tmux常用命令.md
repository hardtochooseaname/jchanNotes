# tmux常用命令

Tmux（Terminal Multiplexer）是一个终端复用器，它允许在单个终端中运行多个会话和分屏。类似功能的工具还有[GNU screen](https://zhida.zhihu.com/search?content_id=239269541&content_type=Article&match_order=1&q=GNU+screen&zhida_source=entity)。它有两个优点：1) 多个终端界面之间切换变得容易；2) 可以把终端挂在后台，防止在使用[ssh](https://zhida.zhihu.com/search?content_id=239269541&content_type=Article&match_order=1&q=ssh&zhida_source=entity)时因断网而导致的终端丢失和程序中断。

## 安装

```text
sudo apt install tmux
```

## 基础概念

tmux按下面三个层级管理终端：

1. session：会话。tmux可以在前台和后台运行多个会话；
2. [window](https://zhida.zhihu.com/search?content_id=239269541&content_type=Article&match_order=1&q=window&zhida_source=entity)：窗口。一个会话可以包含多个窗口；
3. [panel](https://zhida.zhihu.com/search?content_id=239269541&content_type=Article&match_order=1&q=panel&zhida_source=entity)：窗格/分屏。一个窗口可以包含多个窗格。

## 常用命令

**1. 创建新会话：**

```text
tmux
```

如果想自己起名字

```text
tmux new -s <session name>
```

**2. 列出后台正在运行的会话：**

```text
tmux ls
```

**3. 进入后台正在运行的会话：**

```text
tmux a -t <session name>
```

**4. 重命名会话：**

```text
tmux rename -t <session name> <new-name>
```

## 常用快捷键

### **1. 会话快捷键**

- Ctrl + b 接 d：分离当前会话（退出会话界面挂在后台）
- Ctrl + b 接 $：重命名当前会话

### **2. 窗口快捷键**

- Ctrl + b 接 c：新建窗口
- Ctrl + b 接 n：跳转到下个窗口
- Ctrl + b 接 p：跳转到上个窗口
- Ctrl + b 接 <数字键>：跳转到指定窗口
- Ctrl + b 接 w：进入窗口选择列表
- Ctrl + b 接 ,：重命名当前窗口
- Ctrl + b 接 .：修改当前窗口序号

### 3. 分屏快捷键

- Ctrl + b 接 %：左右分屏
- Ctrl + b 接 "：上下分屏
- Ctrl + b 接 <方向键>：分屏跳转
- Ctrl + b 接 x：删除所在分屏
- Ctrl + b + <方向键>：调整分屏大小（按住`Ctrl + b`同时按方向键）
- Ctrl + b 接 z：所在分屏全屏，再按一次恢复
- Ctrl + b 接 {：左移分屏
- Ctrl + b 接 }：右移分屏
- Ctrl + b 接 Ctrl + o：上移分屏
- Ctrl + b 接 Alt + o：下移分屏
- Ctrl + b 接 <空格键>：切换分屏布局
- Ctrl + b 接 q：显示分屏序号和分辨率

### 4. 翻页

Ctrl + b 接 [：进入翻页模式，按 q 退出