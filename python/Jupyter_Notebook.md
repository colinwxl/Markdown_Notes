# The Jupyter notebook
***
<2017.1116> by Colin Wu colinabc@qq.com
## Installation
```sh
conda install jupyter
```
## Create Password
1. Auto creation: version >= 5.0
```py
jupyter notebook password
Enter passwd: ****
Verify password: ****
[NotebookPasswordApp] Wrote hashed password to /Users/you/.jupyter/jupyter_notebook_config.json
```
2. Manually creation
```sh
ipython
In [1]: from notebook.auth import passwd
In [2]: passwd()
Enter password:
Verify password:
Out[2]: 'sha1:67c9e60bb8b6:9ffede0825894254b2e042ea597d771089e11aed'
```
## Configuration
```sh
jupyter notebook --generate-config # generate ~/.jupyter/jupyter_notebook_config.py
vim ~/.jupyter/jupyter_notebook_config.py
```
Ucomment the following lines and fixing them:
```vim
# c.NotebookApp.ip='*' # Errors Occur
c.NotebookApp.ip='0.0.0.0' # listen on all IPs
c.NotebookApp.allow_origin = '*' # allow all origins
c.NotebookApp.password = u'sha:ce...'
c.NotebookApp.open_browser = False
c.NotebookApp.port =8890
```

## Start Service
remote server: `jupyter notebook` or `jupyter notebook --allow-root` for `root` user

local: start the browser and use `http://address_of_remote:8890` to log in jupyter notebook

## Usage:
Refer to http://blog.csdn.net/tina_ttl/article/details/51031113

## Some skills:
1. 内省：[函数|模块]? 可查看帮助文档；[函数|模块]?? 可查看对应的Python源代码
2. 魔法命令：%magic, %timeit, %who, %debug,...一样支持内省
3. ipdb (pip install ipdb)
```python
import ipdb
ipdb.set_trace()
```