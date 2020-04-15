---
title: Centos 7 安装 Jupyter Notebook
author: wanghuagang
date: 2018-08-03 16:04:00
updated: 2017-11-27 11:47:25
tags:
- centos
- Jupyter notebook
categories: 
- 运维
---

本文在 Python36 virtualenv virtualenvwrapper supervisor 环境已经配置完成的基础上操作的。如果你的环境还没准备好，可以参照此文根据自己环境调整相关操作。如果你的环境已经配置完成可以按照此文愉快的安装。如果你想安装前面提到的环境可以参考下面几篇文章。

本文安装 Jupyter Notebook 是我在 Centos 7 服务器上安装的一次记录过程。此方式可以作为本地环境使用，也可以作为服务器环境部署。当然对于本次 Jupyter Notebook 的部署方式，也是我现在常用的项目部署方式----通过 supervisor 部署，而结合 virtualenvwrapper 能够更方便的控制虚拟环境。下面开始吧

<!--more-->

## 环境准备

创建 jupyter 虚拟环境

```bash
mkvirtualenv jupyter
```

## 安装

```bash
pip install ipython jupyter
```

## 配置

### 生成 jupyter 配置文件

一般会直接生成在家目录 `~/.jupyter/jupyter_notebook_config.py`

```bash
jupyter notebook --generate-config
```

### 生成必要配置参数

为了服务器数据安全，我们采用设置密码和设置 https 保证安全登录。当然我们的 https 证书是自己颁发的。

#### 生成加密密码

进入 ipython ，生成 sha1 密码。

```
from notebook.auth import passwd
passwd()
Enter password:
Verify password:
Out[3]: 'sha1:5c82f59da5a3:a68e8360e2cab6336fbbfa3d141f9a617aec7dba'
```

输入两次密码。密码是不回显的！

#### 生成自签名证书

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout ~/.jupyter/jkey.key -out ~/.jupyter/jcert.pem
```

会提示输入一些内容，如果你不知道该填什么的话，那就回车默认好了~

#### 修改配置文件

修改文件 `/.jupyter/jupyter_notebook_config.py`

```bash
c.NotebookApp.password = 'sha1:5c82f59da5a3:a68e8360e2cab6336fbbfa3d141f9a617aec7dba'
c.NotebookApp.port = 8888
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.notebook_dir = '/opt/jupyte_notebook/'
c.NotebookApp.certfile = '/home/user/.jupyter/jcert.pem'
c.NotebookApp.keyfile = '/home/user/.jupyter/jkey.key'
```

**注意**

1. `password` 要填你上面密码生成的 hash 值
2. `notebook_dir` 置了jupyter的notebook路径，即访问jupyter首页时，看到的文件列表就是该目录下的。记得授权给启动用户，要不然可能会出现无法访问
3. 最后两条配置的文件路径记得要写对应文件的绝对路径！

## 启动

**增加防火墙端口**

如果你使用了 firewalld 记得开发端口

```bash
firewall-cmd --add-port=8888/tcp --permanent
firewall-cmd --reload
```

### 测试启动

执行命令启动。

```bash
jupyter notebook
```

然后浏览器访问页面


### supervisord 启动

编辑文件 `/etc/supervisord.d/jupyter_notebook.ini` 填入如下配置

```conf
[program:jupyter_notebook]
command = /home/user/.virtualenvs/jupyter/bin/jupyter notebook
autostart = true
startsecs = 5
autorestart = true
startretries = 3
user = kevin
redirect_stderr = true
stdout_logfile_maxbytes = 20MB
stdout_logfile_backups = 10
stdout_logfile = /var/log/supervisor/jupyter_notebook/stdout.log
```

**注意：**

1. `command` 为你虚拟环境中的启动命令
2. `user` 用哪个用户启动。如果公网能访问不建议使用 root 用户
3. `stdout_logfile` 日志存放目录。记得把权限给启动用户。因为会生成备份日志。建议单独放一个文件夹，然后授权给此用户。免得日志写满了没有权限备份日志和建立新文件。这里配置不好的话 supervisor 页面的 `tail -f` 可能看不到实时打出的日志。

执行 `supervisorctl`进入 supervisor 交互，重新加载 supervisor 配置文件。

```
> upload
```

查看状态

```
status
```

当然这两个操作都可以在 web 页面上进行。

如果有出错请查看相关日志

-----------

**TODO：**

1. 使用 nginx 代理
2. jupyterhub 多用户管理
