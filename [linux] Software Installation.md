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