# The Jupyter notebook
***
<2017.1116> by Colin Wu colinabc@qq.com
## Installation
```
conda install jupyter
```
## Create Password
1. Auto creation: version >= 5.0
```
jupyter notebook password
Enter passwd: ****
Verify password: ****
[NotebookPasswordApp] Wrote hashed password to /Users/you/.jupyter/jupyter_notebook_config.json
```
2. Manually creation
```
ipython
In [1]: from notebook.auth import passwd
In [2]: passwd()
Enter password:
Verify password:
Out[2]: 'sha1:67c9e60bb8b6:9ffede0825894254b2e042ea597d771089e11aed'
```
## Configuration
```
jupyter notebook --generate-config # generate ~/.jupyter/jupyter_notebook_config.py
vim ~/.jupyter/jupyter_notebook_config.py
```
Ucomment the following lines and fixing them:
```
c.NotebookApp.ip='*'
c.NotebookApp.password = u'sha:ce...'
c.NotebookApp.open_browser = False
c.NotebookApp.port =8890
```

## Start Service
remote server: `jupyter notebook` or `jupyter notebook --allow-root` for `root` user

local: start the browser and use `http://address_of_remote:8890` to log in jupyter notebook

## Usage:
Refer to http://blog.csdn.net/tina_ttl/article/details/51031113