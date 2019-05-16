## 用用uWSGI和nginx部署Django Web服务器

本教程针对想要设置生产Web服务器的Django用户。它将指导您完成设置Django所需的步骤，以便与uWSGI和nginx完美配合。它涵盖了所有三个组件，提供了完整的Web应用程序和服务器软件堆栈。

Django是一个高级Python Web框架，鼓励快速开发和干净，实用的设计。

nginx（发音为engine-x）是一个免费的，开源的，高性能的HTTP服务器和反向代理，以及一个IMAP / POP3代理服务器。

### 相关软件版本
```
Django==2.2
Python==3.6.7
nginx==1.17.0
uwsgi==2.0.17
Ubuntu-Server:18.04LTS
```
## 准备工作
### 连接到服务器
执行操作（以下操作建议用root权限操作，避免安装过程中出现的各种权限不够问题）
```
sudo apt update
sudo apt upgrade
```
### 安装python环境
```
# 系统自带Python3环境，更新之后可以使用最新版
# 需要安装python3-pip, 再通过pip3安装virtualenv包， 用来创建python3虚拟环境
sudo apt install python3-pip
sudo pip3 install virtualenv
```
### 安装python虚拟环境
创建虚拟环境可以很好的将项目和其他项目分离
```
# 为项目创建一个文件夹
sudo mkdir /home/project_name/
sudo cd /home/project_name
# 创建虚拟环境
project_name$ sudo virtualenv -p /usr/bin/python3 venv
# 激活虚拟环境
project_name$ source venv/bin/activate
# 激活虚拟环境之后，系统会进入虚拟的python环境
(venv)project_name$
# y要停止使用虚拟环境，可执行命令deactivate
(venv)project_name$ deactivate
```
