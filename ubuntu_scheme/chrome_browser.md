## Chrome Browser on Ubuntu
<2017.1201> by Colin Wu

### Installation
1. start terminal:
`Ctrl + Alt + t` or `Win(Super)`
2. 将下载源加入到系统源列表：
`sudo wget http://www.linuxidc.com/files/repo/google-chrome.list -P /etc/apt/sources.list.d/`
3. 导入谷歌软件的公钥，用于下面步骤对下载软件进行验证。
`wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -`
4. 更新列表：
`sudo apt-get update`
5. 执行谷歌Chrom浏览器（稳定版）的安装：
`sudo apt-get install google-chrome-stable`
6. 启动谷歌浏览器：
`/usr/bin/google-chrome-stable`

### Sync Problems
```
nslookup www.google.com
nslookup clients2.google.com
nslookup chrome.google.com # 扩展中心
nslookup docs.google.com # 书签同步
nslookup clients4.google.com # 书签同步
nslookup tools.google.com
```
将地址写入到`/etc/hosts`里面

