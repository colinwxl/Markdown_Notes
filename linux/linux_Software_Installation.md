## Linux Software Installation
```
echo "PATH=/home/wuxiaolong/tools/bin:$PATH" >> ~/.bashrc
```
### make command
[Parameter]
-j [N], --jobs[=N]  Allow N jobs at once; infinite jobs with no arg.  
如果-j后不跟任何数字，则不限制处理器并行编译的任务数。
### [Lynx](http://lynx.browser.org/)
version:[Lynx2.8.8](https://invisible-mirror.net/archives/lynx/tarballs/lynx2.8.8rel.2.tar.gz)
```
wget https://invisible-mirror.net/archives/lynx/tarballs/lynx2.8.8rel.2.tar.gz --no-check-certificate
tar -xzvf lynx2.8.8rel.2.tar.gz
cd lynx2-8-8
./configure --prefix=/home/wuxiaolong/tools/lynx2.8.8
make && make install
cp ~/tools/lynx2.8.8/bin/lynx ../../bin
```

### linux上python解释器中的自动补全
`~/.pythonrc.py`
```
import rlcompleter, readline
readline.parse_and_bind('tab:complete')
```
`~/.bashrc`
```
export PYTHONSTARTUP=~/.pythonrc.py
```
python启动时就会使用PYTHONSTARTUP变量指定的文件中的初始化代码。

### [curl](https://curl.haxx.se/)
version:[curl-7.55.1](https://curl.haxx.se/download/curl-7.55.1.tar.gz)
```
# can not use wget to download, use winscp upload instead
# three steps
```

### [VirtualBox 5.2](https://www.virtualbox.org/)
1. Step 1, Setup Apt Repository
Firstly edit `/etc/apt/sources.list` file and add one of the following lines according to your distribution to your system. You can find your system distribution codename using `lsb_release -c` command from a terminal.
```
For Ubuntu 17.04 ("Zesty")
deb http://download.virtualbox.org/virtualbox/debian zesty contrib
For Ubuntu 16.04 ("Xenial")
deb http://download.virtualbox.org/virtualbox/debian xenial contrib
For Ubuntu 14.04 ("Trusty")
deb http://download.virtualbox.org/virtualbox/debian trusty contrib
For Debian 9 ("Stretch")
deb http://download.virtualbox.org/virtualbox/debian stretch contrib
For Debian 8 ("Jessie") 
deb http://download.virtualbox.org/virtualbox/debian jessie contrib
```

2. Step 2，Import VirtualBox Package Sign Key
After adding the required apt repository on your system, download and import the Oracle public key for apt-secure using following commands.
```
https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
```

3. Step, Install Oracle VirtualBox
After completing above steps, let’s install VirtualBox using following commands. If you have already installed any older version of VirtualBox, Below command will update it automatically.
```
sudo apt-get update
sudo apt-get install virtualbox-5.2
```

4. Step 4, Launch VirtualBox
We can use dashboard shortcuts to start VirtualBox or simply run following command from a terminal.
```
virtualbox
```