
# **《Bioinformatics Data Skills》**

## Lerning Notes
[Supplementary Materials](https://github.com/vsbuffalo/bds-files)
***

## Chapter 1 How to learn bioinformatics
* Reproducible and Robust
* Document Everything

## Chapter 2 Setting Up and Managing a Bioinformatics Project
* **Directory Structures** (*relative paths*、*devided in to subprojects*)
* **Poject Documentation** (*methds and workflows*、*data source*、*how to download*、*versions of database and software*)
* **Shell Expansion** (*brace expansion*、*parameter expansion*)
> * `mkdir -p zmays-snps/{data/seqs,scripts,analysis}`
> * str="aa.bb.cc.bb.ee.bin"
>> 1. 去头，从开头去除最短匹配前缀:      echo ${str#*.}              #bb.cc.bb.ee.bin
>> 2. 去头，从开头去除最长匹配前缀:      echo ${str##*.}            #bin
>> 3. 去尾，从结尾去除最短匹配后缀:      echo ${str%.*}             #aa.bb.cc.bb.ee
>> 4. 去尾，从结尾去除最长匹配后缀:      echo ${str%%.*}          #aa
>> 5. 删除第一个与"bb"匹配的字符串:     echo ${str/bb}             #aa..cc.bb.ee.bin
>> 6. 删除所有与"bb"匹配的字符串:       echo ${str//bb}            #aa..cc..ee.bin
>> 7. 将第一个"bb"替换成"gg":          echo ${str/bb/gg}        #aa.gg.cc.bb.ee.bin
>> 8. 将所有的"bb"替换成"gg":           echo ${str//bb/gg}       #aa.gg.cc.gg.ee.bin

* **Leading Zeros for Sorting**
* **Markdown**
 1. Syntax ([English Version](http://daringfireball.net/projects/markdown/syntax)、[Chinese Version](http://wowubuntu.com/markdown/#list))
 2. Software(Windows:[MarkdownPad2](http://www.markdownpad.com/); Linux:[Pandoc](http://www.pandoc.org/))

## Chapter 3 Remedical Unix Shell
> Philosophy: Write programs that do one thing and do it well.

* **tee**

## Chapter 4 Working with Remote Machines
* **storing** frequent ssh host
> ~/.ssh/config
>
> Host bio_serv
>
>     HostName 192.168.237.42
>     User cdarwin
>     Port 50453
> ssh bio_serv

* **ssh keys**

> ``` bash
> ssh-keygen -b 2084 
> (press two Enter)
> ```
>> This creates a private key at ~/.ssh/id_rsa and a public key at ~/.ssh/id_rsa.pub.
>> To use password-less authentication: 
>> `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`
>
> ssh-agent (wait to be learned)

* **Tmux**
![Book](http://i.imgur.com/LdrwOP4.jpg)
	1. Installation（by compiling source）
		1） install libevent
	``` bash
	wget https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz

	tar -xvzf libevent-2.1.8-stable.tar.gz

	cd libevent-2.1.8-stable

	./configure --prefix=/path/to/install/libevent

	make && make isntall
	```
		2) install Tmux
	``` bash
	wget https://github.com/tmux/tmux/releases/download/2.3/tmux-2.3.tar.gz

	tar -xvzf tmux-2.3.tar.gz

	cd tmux-2.3

	CFLAGS="-I/path/to/install/libevent/include" LDFLAGS="-L/path/to/install/libevent/lib" ./configure --prefix=/path/to/install/tmux

	make && make install

	echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/install/libevent/lib" >> ~/.bashrc

	. ~/.bashrc
	```
	2. Usage [[Manual](http://man.openbsd.org/OpenBSD-current/man1/tmux.1)]
		1) Configuration:
        ~/.tmux.conf
        > set -g mouse on ()
        2) Basic Usage:
		* the command prefix: Ctrl-b
		`$ tmux new-session -s basic` **or** `$ tmux new -s basic`
		`inside of tmux, press CTRL - b , then press t . A large clock will appear on the screen.`
		`Prefix d` --> detach
		`exit`
		`$ tmux list-sessions` **or** `$ tmux ls`
		`$ tmux kill-session -t basic`
		`$ tmux kill-session -t second_session`
		`$ tmux new -s windows -n shell`
		`Prefix %`(v) **or** `Prefix "`(H)
		`Prefix :` --> command mode
		> |Command   |Description                                                    |
		> |:--------:|:-------------------------------------------------------------:|
		> |`PREFIX d`|Detaches from the session, leaving the session running in the background.|
		> |`PREFIX :`| Enters Command mode.|
		> |`PREFIX c` | Creates a new window from within an existing tmux session. Shortcut for new-window.|
		> |`PREFIX n` | Moves to the next window.|
		> |`PREFIX p` | Moves to the previous window.|
		> |`PREFIX 0 … 9` | Selects windows by number.|
        > |`PREFIX w` | Displays a selectable list of windows in the current session.|
		> |`PREFIX f` | Searches for a window that contains the text you specify. Displays a selectable list of windows containing that text in the current session.|
		> |`PREFIX ,` | Displays a prompt to rename a window.|
		> |`PREFIX &` | Closes the current window after prompting for confirmation.|
		> |`PREFIX %`| Divides the current window in half vertically.|
		> |`PREFIX "` | Divides the current window in half horizontally.|
		> |`PREFIX o` | Cycles through open panes.|
		> |`PREFIX q` | Momentarily displays pane numbers in each pane.|
		> |`PREFIX x` | Closes the current pane after prompting for confirmation.|
		> |`PREFIX SPACE` | Cycles through the various pane layouts.|