## Tmux
<2017-10-12>
### Introduction
> tmux is a terminal multiplexer: it enables a number of terminals to be created, accessed, and controlled from a single screen. tmux may be detached from a screen and continue running in the background, then later reattached.
> 
> 上面是官方给出的解释，主要说明了tmux的两个强大功能。
> 
> 1. 能够通过一个屏幕，创建并管理多个终端。
> 2. 断线后连接，所有的工作得以保存。

网上有一篇文章说，有关Tmux的教程多达4257篇，这还只是粗略统计的，这足矣看出Tmux真牛B。

### Why Tmux
首先我们先看一张图：
![Tmux1](http://i.imgur.com/iUqD1Ug.png)

这是小编自己配置的Tmux工作环境。

为啥不使用纵向分屏(tmux里为horizon),是为了方便复制shell脚本里面的代码哦。

很多人看到这,就会觉得Tmux就是个分屏器。其实不仅如此，tmux在很多方面都很有用。就很linux工作者而言，由于 tmux 允许随时随地断开或重新接入会话（Session），所以最大的作用就是在远程服务器上持久地保存工作状态。

举个简单的例子：
小明今天打开了很多个终端，进行多项工作。下班时间到了，他只需要detach一下tmux的session，然后就可以下班做自己的事情。等到第二天上班时间，他只需要再attach以下之前的session，就可以看到之前所有的工作内容。这样是不是很方便呢？

### How Tmux
![tmux2](http://i.imgur.com/xJmMCVW.png)
tmux使用C/S模型构建，主要包括以下单元模块：

* 一个tmux命令执行后启动一个tmux服务
* 一个tmux服务可以拥有多个session，一个session可以看作是tmux管理下的伪终端的一个集合
* 一个session可能会有多个window与之关联，每个window都是一个伪终端，会占据整个屏幕
* 一个window可以被分割成多个pane

#### Installation (by compiling source)
由于小编并没有root权限，所以只能以手动编译源代码的方式安装。

1） install libevent
```
    wget https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz
    
    tar -xvzf libevent-2.1.8-stable.tar.gz
    cd libevent-2.1.8-stable
    
    ./configure --prefix=/path/to/install/libevent
    make && make isntall
```

2) install Tmux
```
	wget https://github.com/tmux/tmux/releases/download/2.5/tmux-2.5.tar.gz
	tar -xvzf tmux-2.5.tar.gz
	cd tmux-2.5

	CFLAGS="-I/path/to/install/libevent/include" LDFLAGS="-L/path/to/install/libevent/lib" ./configure --prefix=/path/to/install/tmux
	make && make install

	echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/install/libevent/lib" >> ~/.bashrc
	echo "export PATH=/path/to/install/tmux/bin:$PATH" >> ~/.bashrc
	echo 'alias tmux="tmux -2"' >> ~/.bashrc
	. ~/.bashrc
```

#### Basic Usage
##### session
1. 如果上面的安装没有问题，那么我们就可以在终端上敲`tmux`，这样就开启了一个tmux会话（session）。你可以按ctrl+b，然后按d，先断开这个会话，并在稍后再重新接入。
2. 显示所有会话：
`tmux ls`
3. 也可以在新建会话的同时指定名字：
`tmux new -s session-name -n window-name`
这样后面就可以直接使用该会话名称来接入会话：
`tmux a -t session-name`
4. 关闭sessions
<1> type `exit` within a session/window/pane to destroy it
<2> detach and type `tmux kill-session -t session-name`

##### window
1. If you end up with more than nine windows, you can use `Prefix w` to display a visual menu of your windows so you can select the one you’d like. You can also use `Prefix f` to find a window that contains a string of text. Typing the text and pressing Enter displays a list of windows containing that text.
2. To close a window, you can either type “exit” into the prompt in the window, or you can use Prefix &, which displays a confirmation message in the status bar before killing off the window. If you accept, your previous window comes into focus. To completely close out the tmux session, you have to close all the windows in the session.

##### pane
1. To cycle through the panes, press `Prefix o`. You can also use `Prefix`, followed by the `Up`, `Down`, `Left`, or `Right` keys to move around the panes.
2. 
#### Configuration
tmux的个人配置文件为~/.tmux.conf
具体的如何配置，大家可以参考下这本书：

![Book](http://i.imgur.com/eiyciC6.jpg)

以下是小编本人的配置文件内容：

	# Setting the prefix from C-b to C-a
	set -g prefix C-a
	
	# Free the original Ctrl-b prefix keybinding
	unbind C-b
 
	# Setting the delay between prefix and command
	# set -s escape-time 1

	# Set the base index for windows to 1 instead of 0
	# set -g base-index 1
 
	# Define Prefix r so it reloads the .tmux.conf file in the current session
	bind r source-file ~/.tmux.conf \; display "Reloaded!"
 
	# Ensure that we can send Ctrl-a to other apps
	bind C-a send-prefix
 
	# splitting panes with | and -
	bind | split-window -h
	bind - split-window -v
 
	# moving between panes with Prefix h,j,k,l
	# bind h select -pane -L
	# bind j select -pane -D
	# bind k select -pane -U
	# bind l select -pane -R
 
	# mouse support - set to on if you want to use the mouse
	set -g mouse off

	# mouse options for selecting pane
	set -g mouse on
	set -g default-terminal "screen-256color"
	set -g status-style bg=black,fg=white
	set -g window-status-style fg=cyan,bg=black
	setw -g window-status-current-style fg=white,bold,bg=red
 
	# colors for pane borders
	setw -g pane-border-style fg=green,bg=black
	setw -g pane-active-border-style fg=white,bg=yellow

1. colors
add `[ -z ​"​$TMUX​"​ ] && export TERM=xterm-256color` to `~/.bashrc`. This conditional statement ensures that the TERM variable is only set outside of tmux, since tmux sets its own terminal.
2. status line
The status line consists of three components: a left panel, the window list, and a right panel. 

|Variable	  |Description|
|:-----------:|:-----------------------------------------:|
|#H           |Hostname of local host|
|#h           |Hostname of local host without the domain name|
|#F           |Current window flag|
|#I           |Current window index|
|#P           |Current pane index|
|#S           |Current session name|
|#T           |Current window title|
|#W           |Current window name|
|##           |A literal #|
|#(shell-command)|First line of the shell command’s output|
|#[attributes]|Color or attribute change|

#### Shortcuts

|Command          |Description                                                  |
|:---------------:|:-----------------------------------------------------------:|
|`PREFIX d`       |Detaches from the session, leaving the session running in the background.|
|`PREFIX :`       | Enters Command mode.|
|`PREFIX c`       | Creates a new window from within an existing tmux session. Shortcut for new-|
|`PREFIX n`       | Moves to the next window.|
|`PREFIX p`       | Moves to the previous window.|
|`PREFIX 0 … 9`   | Selects windows by number.|
|`PREFIX w`       | Displays a selectable list of windows in the current session.|
|`PREFIX f`       | Searches for a window that contains the text you specify. Displays a selectable list of windows containing that text in the current session.|
|`PREFIX ,`       | Displays a prompt to rename a window.|
|`PREFIX &`       | Closes the current window after prompting for confirmation.|
|`PREFIX %`       | Divides the current window in half vertically.|
|`PREFIX "`       | Divides the current window in half horizontally.|
|`PREFIX o`       | Cycles through open panes.|
|`PREFIX q`       | Momentarily displays pane numbers in each pane.|
|`PREFIX x`       | Closes the current pane after prompting for confirmation.|
|`PREFIX SPACE`   | Cycles through the various pane layouts.|

#### Pair Programming
> There are two ways to work with remote users. The first method involves creating a new user account that you and others share. You set up tmux and your development environment under that account and use it as a shared workspace. The second approach uses tmux’s sockets so you can have a second user connect to your tmux session without having to share your user account

