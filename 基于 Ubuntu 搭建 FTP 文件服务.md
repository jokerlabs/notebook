## 基于 Ubuntu 搭建 FTP 文件服务
FTP 是一个很实用的文件传输协议，方便在客户端和服务器之间进行文件的传输。本笔记使用 vsftpd 来搭建一个 FTP 服务，并且创建专有的 FTP 登录账户，保障服务器安全。
### 软件环境
腾讯云服务器+Ubuntu 18.04LTS 64bit

### 安装并启动 FTP 服务

#### 安装 VSFTPD
使用 `apt-get` 安装 `vsftpd`：
```
sudo apt-get install vsftpd -y
```

`vsftpd` 是在 Linux 上被广泛使用的 FTP 服务器，根据其官网介绍，它可能是 UNIX-like 系统下最安全和快速的 FTP 服务器软件。

#### 启动 VSFTPD
安装完成后 VSFTPD 会自动启动，通过 `netstat `命令可以看到系统已经[监听了 21 端口]：
```
sudo netstat -nltp | grep 21
```
如果没有启动，可以手动开启 VSFTPD 服务：
```
sudo systemctl start vsftpd.service
```
FTP 协议默认使用 21 端口作为服务端口

### 配置用户访问目录

#### 新建用户主目录
```
sudo mkdir /home/uftp
```
执行完后，在这里 `/home/uftp` 就能看到新建的文件夹 `uftp` 了。

#### 创建登录欢迎文件：
```
sudo touch /home/uftp/welcome.txt
```
方便用户登录后可以看到欢迎信息，并且确定用户确实登录到了主目录上。

用户的主目录是用户通过 FTP 登录后看到的根目录

#### 新建用户 uftp 并设置密码
创建一个用户 `uftp`：
```
sudo useradd -d /home/uftp -s /bin/bash uftp
```
为用户 `uftp` 设置密码：
```
sudo passwd uftp
```
删除掉 `pam.d` 中 `vsftpd`，因为该配置文件会导致使用用户名登录 ftp 失败：
```
sudo rm /etc/pam.d/vsftpd
```

为了方便后面的实验步骤，不建议使用其它的用户名

请记住设置的密码以用于后续步骤

#### 限制该用户仅能通过 FTP 访问
限制用户 `uftp` 只能通过 FTP 访问服务器，而不能直接登录服务器：
```
sudo usermod -s /sbin/nologin uftp
```
#### 修改 vsftpd 配置
```
sudo chmod a+w /etc/vsftpd.conf
```
修改 `/etc/vsftpd.conf` 文件中的配置（直接将如下配置添加到配置文件最下方）：
```
# 限制用户对主目录以外目录访问
chroot_local_user=YES

# 指定一个 userlist 存放允许访问 ftp 的用户列表
chroot_list_enable=YES

# 记录允许访问 ftp 用户列表
chroot_list_file=/etc/vsftpd.chroot_list

# 不配置可能导致莫名的530问题
seccomp_sandbox=NO

# 允许文件上传
write_enable=YES

# 使用utf8编码
utf8_filesystem=YES
```
新建文件 `/etc/vsftpd.chroot_list`，用于存放允许访问 ftp 的用户：
```
sudo touch /etc/vsftpd.chroot_list
sudo chmod a+w /etc/vsftpd.chroot_list
```
修改 `/etc/vsftpd.chroot_list` ，加入刚刚创建的用户：

示例代码：`/etc/vsftpd.chroot_list`
```
uftp
```
#### 设置访问权限
设置主目录访问权限（只读）：
```
sudo chmod a-w /home/uftp
```
新建公共目录，并设置权限（读写）：
```
sudo mkdir /home/uftp/public && sudo chmod 777 -R /home/uftp/public
```
重启vsftpd 服务：
```
sudo systemctl restart vsftpd.service
```

### 访问 FTP 服务

FTP 服务已安装并配置完成，下面我们来使用该 FTP 服务

#### 访问 FTP 服务
根据您个人的工作环境，选择一种方式来访问已经搭建的 FTP 服务

#### 通过 FTP 客户端工具访问
FTP 客户端工具众多，下面推荐两个常用的：

FileZilla - 跨平台的 FTP 客户端，支持 Windows 和 Mac

FlashFXP- FlashFXP是一款功能强大的FXP/FTP软件，集成了其它优秀的FTP软件的优点
下载和安装 FTP 客户端后，使用下面的凭据进行连接即可：

主机：
```
<您的 CVM IP 地址>
```
用户：
```
uftp
```
输入密码后，如果能够正常连接，那么大功告成，您可以开始使用属于您自己的 FTP 服务器了！

接下来，请上传任意一张图片到您的 FTP 服务器上的 uftp 的 public 目录下，然后，就可以在 `/home/uftp/public` 中看到了。

#### 通过 Windows 资源管理器访问
Windows 用户可以复制下面的[链接]到资源管理器的地址栏访问：
```
ftp://uftp:你的密码@<您的 CVM IP 地址>
```
如果您申请了域名，可以将Ip 地址替换为对应的域名作为访问凭据

如果您申请了域名，可以将链接中的 Ip 地址替换为对应的域名访问 FTP 服务

大功告成

### 参考资料
<https://cloud.tencent.com/developer/labs/lab/10123#stage-1-step-2-21>

