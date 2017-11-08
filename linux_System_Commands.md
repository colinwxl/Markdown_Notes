##[linux] system command
1. 查看系统版本
cat /etc/redhat-release

2. 免密钥ssh登录(ubuntu)
`ssh-keygen`，执行过程中它先要求你确认保存公钥的位置(默认为: `~/.ssh/id_rsa`)，然后它会让你重复输入一个密码两次，如果不想在使用公钥的时候输入密码，可以留空。执行完毕后，就会生成数据Client端的一对密钥。SSH密钥默认储存在账户的家目录下的`.ssh`目录中。关键是看有没有用`xxx_rsa`和`xxx_rsa.pub`来命名的一对文件，有`.pub`后缀的文件就是公钥，另一个文件是密钥。

```
cd ~/.ssh
ls -al
```
检查是否存在`authorized_keys`的文件，不存在：`touch authorized_keys`，将`xxx_rsa.pub`里的密钥复制到`authoried_keys`中，`chmod 600 authorized_keys`