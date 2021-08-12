# 远程服务器连接


## 1. 连接

通过SSH，首先需要确定自己已经安装了openSSH，本次是在Windows环境下进行。选用自己习惯的软件即可，我用过putty, Xshell, VScode等等，由于~~自己太懒~~配置不好vim，而且终端看久了眼疼，就想采用VS Code来进行连接。

### 通用流程

1. 安装SSH和VS code，并在VS code的扩展商店里添加Remote SSH相关插件。

2. 生成密钥。在.ssh目录下可以用`ssh-keygen`命令生成公钥和私钥，并将公钥传到远程服务器的根目录下.ssh文件夹中。

3. 连接远程主机(例如通过`ssh username@ip -p port`命令)，在.ssh目录下通过`cat xxx.pub >> authorized_keys`命令将公钥添加到authorized_keys中。

4. 采用私钥进行登录(例如`ssh username@ip -p port -i id_rsa`命令)。点击VS code中Remote SSH的设置键，通常选第一个configuration file进入，根据需要填写下面的参数信息：

   ```config
   Host: 连接的主机的名称，可以自己起名
   Hostname: remote server的IP地址
   Port: 用于登录远程服务器的端口
   User: 用于登录远程主机的用户名
   IdentityFile: 私钥在本地的路径
   ProxyCommand: 需要执行的指令
   ```

   完成并保存后就可以发现在左侧添加上的新的远程服务器，点击连接即可。

### 跳板机

有时候例如自己实验的的服务器是设置在内网环境中的，用自己的客户端在公共环境无法直接连接，通常采用跳板机来解决这个问题。一般是先通过ssh连接到跳板机之后，再跳转到所需的服务器，同样是只能用vim使用命令行环境，由于配置起来比较麻烦，使用不便，还是想用VS code来提高便捷性。配置的过程与通用流程稍有不同，因为涉及到两个机器，所以在configuration files中要按照如下方式填写：

```config
Host JumpMachine             #跳板机名称
    HostName XXX.XXX.XXX.XXX #跳板机IP 
    Port XXX                 #跳板机ssh端口
    User root                #跳板机用户名
    IdentityFile xx\xx\xx..
    
Host TargetMachine           #远程服务器名称
    HostName XXX.XXX.XXX.XXX #远程服务器IP
    Port XXX                 #远程服务器ssh端口
    User root                #远程服务器用户名
    ProxyCommand ssh(替换为自己安装的ssh的路径，例如C:\Windows\System32\OpenSSH\ssh.exe) -W %h:%p JumpMachine
```

同样的，为了省去密码等，可以发送公钥到跳板机和服务器，并添加到authorized_keys中(并在参数中添加`IdentityFile`)。(**注：所有前序机器的id_rsa.pub都要添加到最终机器上。比如说有3台机ABC，其中B是跳板机。那么A的.pub要在B跟C上分别导入一次，B的.pub要在C上导入一次，共3次，才能完全实现免密登录远程服务器**)

为了防止发送公钥时覆盖了目标机器上的authorized_keys文件，可以用`ssh-copy-id`命令来复制公钥

```bash
ssh-copy-id -i id_rsa.pub "-p 跳板机的ssh端口 用户名@跳板机IP"
```

将笔记本的公钥也同样的发送到远程服务器中

可以先使用scp将公钥发送至跳板机

```bash
scp -P 跳板机ssh端口 id_rsa.pub  跳板机用户名@跳板机IP:~/.ssh/temp
```


再通过ssh连接至跳板机，并切换到~/.ssh目录下，将其发送至远程服务器

```bash
scp -P 远程服务器ssh端口 temp 远程服务器用户名@远程服务器IP:~/.ssh/temp
```

最后通过ssh连接至远程服务器，切换到~/.ssh目录下，手动将公钥拼接到authorized_keys中

```bash
cat temp >> authorized_keys
```

配置保存后就可以在左边的SSH TARGETS中看到配置好的JumpMachine和TargetMachine，选择TargetMachine进行连接即可。



## 2. 文件传输

个人~~由于懒而且记不住命令~~还是比较习惯有GUI，平时比较常用的是[WinSCP](https://winscp.net/eng/docs/lang:chs),这是一个在Windows环境下使用的SSH的开源图形化SFTP客户端，同时支持SCP协议。它可以在本地与远程计算机之间安全的传输文件，并可以直接编辑文件。它可以很直观的看到文件的层级结构和控制文件的传输而无需依赖其它插件。

### 基本使用

新建会话主机名填入IP，端口号填入自己的端口号，再输入用户名和密码，保存并登陆之后应该可以看到文件的界面。

### 跳板机

在session界面点击高级选项，在高级设置中找到`隧道/Tunnel`，勾选上使用SSH隧道，并输入跳转机的IP、端口号以及用户名和密码，如果不输入密码，可以在`Tunnel`界面下方添加自己的私钥文件(需要是ppk，如果不是可以用PuTTYgen来生成/转换)。登录界面的信息填最终想通过跳转机连接到的服务器的配置即可。

## 参考文献

1. [VS Code Remote SSH配置](https://zhuanlan.zhihu.com/p/68577071)
2. [vscode通过跳板机连接远程服务器_huitailangyz的博客-CSDN博客](https://blog.csdn.net/huitailangyz/article/details/106392021)
3. [使用WinScp连接远程服务器和传输文件 - 你要 - 博客园 (cnblogs.com)](https://www.cnblogs.com/fuyaozhishang/p/8033849.html)
4. [winscp通过跳板机访问远程服务器（使用秘钥的方式传输文件） - 华为云 (huaweicloud.com)](https://www.huaweicloud.com/articles/6a1b96826310fba7855e727ec591159f.html)

