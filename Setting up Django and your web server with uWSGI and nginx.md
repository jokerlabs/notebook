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
