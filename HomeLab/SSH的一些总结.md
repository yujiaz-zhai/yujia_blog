# SSH远程登录的一些总结

> SSH远程登录作为开发时常用到的操作，如果想要优雅地辗转本地、云服务器、自建工作站等多端，总结一下SSH的基本操作还是很有必要的

## 通用操作

### 一、SSH登录

假设现在你有一台带有公网ip的云服务器，或者是你有一台和你现在使用的电脑处在同一网段的设备及其ip，那么要如何登录到这台设备上？

通常的做法是：

```shell
ssh -p 22 User@HostName
```

其中：

`user`: 登录的用户

`host`: 登录的主机，这里通常是主机的ip地址

`ssh`默认的端口号为22，因此如果你开放了其他端口供ssh登录，只需要更改`-p`后面的端口号即可，如果保持默认22端口，那么一般简介的写法可以省略`-p`

第一次登录到某台机器时，通常会触发提示：

```
The authenticity of host '123.57.47.211 (123.57.47.211)' can't be established.
ECDSA key fingerprint is SHA256:iy237yysfCe013/l+kpDGfEG9xxHxm0dnxnAbJTPpG8.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

这里输入`yes`	向下进行即可，此时对方主机的信息就会保存在本地机器的`~/.ssh/known_hosts`中，下次登录就不会触发提示。

后续的操作就是密码登录

这种方法似乎不太优雅，因为实际操作起来的时候每次登录都要输入一大串代码，体验感不好，为了优化一下体验，可以做一下**地址的映射**

首先进入`~/.ssh`，创建并修改`config`文件

```shell
cd ~/.ssh | vim config
```

`config`:

```
Host MyServer
	HostName xxx.xxx.xxx.xxx
	User username
	Port xxxx
```

下次登录时可以直接输入即可直接进入输密码步骤

```
ssh MyServer
```

体验感优化了不少，但是每次使用密码登录还是不太优雅，不太符合SSH的设计初衷，因此可以配置免密登录

### 二、免密登录

在本地的机器上运行

```shell
ssh-keygen
```

然后一直回车即可，你会发现在`~/.ssh`文件夹下多了`id_rsa`和`id_rsa.pub`文件，其中：

`id_rsa`: 这是本地机器的私钥，不能泄露

`id_rsa.pub`: 本地机器的公钥，配置免密登录时需要传给对方机器

这里需要简单说一下双钥验证的逻辑，如果你要连接到SSH服务器上，客户端软件就会向服务器发出请求，请求用你的密匙进行安全验证。服务器收到请求之后，先在该服务器上你的主目录下寻找你的公用密匙，然后把它和你发送过来的公用密匙进行比较。如果两个密匙一致，服务器就用公用密匙加密"质询"(challenge)并把它发送给客户端软件。客户端软件收到"质询"之后就可以用你的私人密匙解密再把它发送给服务器

第二种级别不仅加密所有传送的数据，而且"中间人"这种攻击方式也是不可能的(因为他没有你的私人密匙)

所以配置免密登录的前提就是先让对方机器 ‘认识’ 你，因此需要先传送本地机器的公钥到对方机器，可以使用

```shell
ssh-copy-id MyServer
```

或者你可以直接复制本地机器的公钥到对方机器的`~/.ssh/authorized_keys`文件中

两种方法的结果是一致的，都是在`authorized_keys`中记录公钥

此后下次登录就不需要传递口令了

### **特别的：对于远程登录Windows系统配置免密登录**

因为做内网穿透登录到Windows时配置免密登录遇到了些麻烦，特总结在这里

> 这里感谢B站的BUPT的大佬[大啊好我r中之](https://space.bilibili.com/239828907)，详情转https://www.bilibili.com/video/BV13L411w7XU/?spm_id_from=333.999.0.0&vd_source=37f658bc9088523a078f185569af336c

首先先确定Windows上是否安装了

```
OpenSSH Authentication Agent
OpenSSH Server
```

可以使用下方命令来检查，如果安装了但是没有打开（第二行命令运行后显示Stopped），需要去计算机管理->服务中手动开启这两项服务

```
ssh -V
Get-Service -Name *ssh*
```

并确保防火墙打开了22端口

```
netstat -an | findstr :22
```

检查工作做完后，进入`C:\ProgramData\ssh\`修改`sshd_config`文件

确保一下三行没有被注释

```
PubkeyAuthentication yes #是否使用双钥验证
AuthorizedKeysFile .ssh/authorized_keys  #钥匙来自的文件地址
PasswordAuthentication yes #是否允许密码登录
```

确保以下三行被注释了

```
#Match Group administrators
#AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

然后重启`OpenSSH Server`服务

这样你就可以尝试使用上面的通用方法配置免密登录，如果在使用`ssh-copy-id`出现乱码而且配置不成功的话，选用另一种手动复制公钥的方法即可解决