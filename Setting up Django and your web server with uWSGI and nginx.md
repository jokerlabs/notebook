## 用uWSGI和nginx部署Django Web服务器

本教程针对想要设置生产Web服务器的Django用户。它将指导您完成设置Django所需的步骤，以便与uWSGI和nginx完美配合。它涵盖了所有三个组件，提供了完整的Web应用程序和服务器软件堆栈。

Django是一个高级Python Web框架，鼓励快速开发和干净，实用的设计。

nginx（发音为engine-x）是一个免费的，开源的，高性能的HTTP服务器和反向代理，以及一个IMAP / POP3代理服务器。

## 相关概念
Web服务器面向外部世界。它可以直接从文件系统提供文件（HTML，图像，CSS等）。但是，它不能直接与Django应用程序对话; 它需要运行应用程序的东西，从Web客户端（例如浏览器）提供请求并返回响应。

Web服务器网关接口 - WSGI - 完成这项工作，WSGI是Python标准。

uWSGI是一个WSGI实现。在本教程中，我们将设置uWSGI，以便创建一个Unix套接字，并通过uwsgi协议提供对Web服务器的响应。最后，我们完整的组件堆栈将如下所示：

```the web client <-> the web server <-> the socket <-> uwsgi <-> Django```



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

### 安装django, 创建项目
```
# 创建并激活虚拟环境之后，就可以安装Django了
(venv)project_name$ pip install django==2.2

# 创建项目
(venv)project_name$ django-admin.py startproject www .
(venv)project_name$ ls
www venv manage.py

# 此时django项目就创建完了，可以查看项目，显示Django欢迎界面，说明Django成功运行
(venv)project_name$ python manage.py runserver
```


## 安装uWSGI并配置
### 安装uWSGI
```
(venv)project_name$ pip install uwsgi
```
### 测试uWSGI
创建一个`uwsgi_test.py` 文件，写入以下内容
```python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello Uwsgi"]
```
运行uWSGI：
```
uwsgi --http :8001 --wsgi-file uwsgi_test.py
```
找个浏览器，访问`http://<your ip adress>:8001/`, 如果显示`Hello Uwsgi`, 说明uWSGI正常运行, 以下堆栈有效
```
the web client <-> uWSGI <-> Python
```
### 测试你的Django项目
现在我们希望uWSGI做同样的事情，但是要运行Django站点而不是`uwsgi_test.py`模块
```
uwsgi --http :8001 --module www.wsgi
```
将浏览器指向服务器：`<你的ip地址>:8001`，如果django页面出现，说明以下堆栈可以正常运行。
```
the web client <-> uWSGI <-> Django
```


## 安装nginx
### 安装
```
sudo apt insatll nginx
```
安装后nginx会自动启动，如果不行请运行以下命令
```
sudo /etc/init.d/nginx
```
在浏览器指向服务器：`<你的ip地址>`，显示nginx欢迎界面，表示nginx成功安装并启动

### 配置nginx
您将需要该uwsgi_params文件，该文件nginx 位于uWSGI发行版的目录中，或者来自 <https://github.com/nginx/nginx/blob/master/conf/uwsgi_params>将其复制到项目目录中.

前往/etc/nginx/目录，查看nginx.conf（nginx基础配置），发现里面有这么两行，意思就是包含conf.d文件夹中所有以conf后缀的配置和site-enabled文件夹中的内容
```
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```
我们不更改`nginx.conf`基础配置，只需要修改`conf.d`目录下的`conf`文件即可，进入`conf.d`文件夹，修改`default.conf`文件，没有的话就新建一个
```shell
# nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name example.com; # substitute your machine's IP address or FQDN
    charset     utf-8;
    
    # nginx log
    access_log      /var/www/<PROJECT_NAME>/nginx_access.log;
    error_log       /var/www/<PROJECT_NAME>/nginx_error.log;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /path/to/your/mysite/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /path/to/your/mysite/uwsgi_params; # the uwsgi_params file you installed
    }
}
```
重启nginx
```
sudo /etc/init.d/nginx restart
```
### nginx的和uWSGI的测试
让我们让nginx与“hello world” `uwsgi_test.py`应用程序对话:
```
uwsgi --socket :8001 --wsgi-file uwsgi_test.py
```
- `socket: 8001`: 使用协议uwsgi，端口8001

在浏览器访问：`<你的ip地址>:8000`，显示Hello Uwsgi说明nginx与uWSGI成功通信，以下堆栈成功：
```
the web client <-> the web server <-> the socket <-> uWSGI <-> Python
```
### 用uwsgi和nginx运行Django应用程序
让我们运行我们的Django应用程序：
```
uwsgi --socket ：8001 --module www.wsgi
```
在浏览器输入：`<你的ip地址>： 8000`， 就可以看到我们的Django页面啦！

### 修改端口 
http的默认端口:80被nginx的欢迎页面程序所占用，我们先解除80端口占用：
配置文件目录：'/etc/nginx/sites-enables/', 修改default文件，这个文件是默认加载的配置文件，编辑：
```
server {
	# listen 80 default_server;  ## 这里默认欢迎页面是80端口，把这个端口改成一个不用的端口号
	# listen [::]:80 default_server;   ## 这里默认欢迎页面是80端口，改成一个不用的端口号
    listen 80 default_server; 
	listen [::]:80 default_server; 
}
```
然后修改/etc/nginx/conf.d/default.conf这个我们创建的文件，把端口号改为80
```
server {
    # the port your site will be served on
    listen      80;
```

重启nginx，就可以直接用我们的ip地址或者域名访问Django网站了。

## 编写并上传Django项目
到这里就大功告成了，接下来就是编写我们的Django应用，用ftp工具上传，实现我们想要实现的功能。祝你好运。如果有任何问题，请随时跟我通过电子邮件讨论：
<xuqjia@163.com>，感谢阅读。







