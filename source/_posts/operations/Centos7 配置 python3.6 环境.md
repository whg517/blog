---
title: Centos7 配置 Python36 环境
author: wanghuagang
date: 2018-04-02 18:59:00
updated: 2017-11-27 11:47:25
tags:
- centos
- python
categories: 
- 运维
---

### 安装

> 由于 Linux 和 python 起源都比较早，所以不要把 python2.x 卸载，否则会出现系统不可逆损坏！

```bash
yum install python36
```

`python` 命令默认是系统 python2.x 软连接到 `python2.x` 上的，请勿将该软链接置到 python3.x 上，否可可能会出现系统问题，当然可以通过 livecd 启动然后把软链接更改回来。

**后文中的 python 都是指系统中 python2.x**

```
[root@localhost ~]# python3.6 --version
Python 3.6.3
```

<!--more-->

### python3 软链接
为了方便使用 python3.6，我们做一个 python3 的软链接到 python3.6 上

python36 是链接到 python3.6 上的

```bash
cd /usr/bin/
ln -s python36 python3
```

检查环境

```
[root@localhost bin]# python3 --version
Python 3.6.3
```

### 配置 pip 使用国内源

为了更快速更方便的安装 Python 库，我们配置国内 pypi 镜像来加速。

国内可加速的镜像有 [阿里云](https://opsx.alibaba.com/mirror) [豆瓣](https://pypi.doubanio.com/simple/) [清华源](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)

> 阿里云速度比较快，但对于更新时效性，清华源的 pypi 更及时，五分钟一次，阿里是每天一次。不过在我看来软件更新来说，我感觉版本迭代也没那么快吧，我们也不一定要用最新版本。所以我选择速度快的。

在文件

`~/.pip/pip.conf`

中添加或修改:

```conf
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```

如果出现 `ssl` 问题，请将 `http` 换成 `https` 或添加 `trusted-host` 参数

### 安装 pip3.6

这个东西有点坑，python3.6 之前，可以通过 yum 安装 `python35-pip` 。 python36 还没那个包。不过我们可以通过以下方法来安装 pip36 模块

```bash
wget https://bootstrap.pypa.io/get-pip.py
```

> 如果没有 `wget` 请执行 `yum install wget`

执行

```bash
python3.6 get-pip.py
```

安装完成后使用

```
[root@localhost ~]# pip3 --version
pip 9.0.3 from /usr/local/lib/python3.6/site-packages (python 3.6)
```

如果你的系统没有为 python 安装 pip ，`pip --version` 也应该为 pyhon3.6 。当然即使有了，可以修改 `/usr/local/bin/pip` 文件指向 pip3 或者使用软链接。

**后文中 pip 都是指 pip3**

### 安装虚拟环境 Virtualenvwrapper

详细内容参考 [Virtualenv和Virtualenvwrapper的配置使用](https://blog.csdn.net/leafage_m/article/details/72854559) 

里面包含 Winwows 和 Linux 的安装使用说明

> virtualenv is a tool to create isolated Python environments.

virtualenv是用来创建一个独立的Python虚拟环境的工具，通过virtualenv可以创建一个拥有独立的python版本和安装库的虚拟开发环境。这样一来我们就可以在虚拟环境中安装各种各种所需要的库，从而不会造成本地的库过多所引起的使用混乱。同时也可以创建不同的python版本来完成不同的需求开发。

对应的Virtualenvwrapper是在使用virtualenv的一个扩展。

> virtualenvwrapper is a set of extensions to Ian Bicking’s virtualenv tool. The extensions include wrappers for creating and deleting virtual environments and otherwise managing your development workflow, making it easier to work on more than one project at a time without introducing conflicts in their dependencies.

通过wrapper可以方便的管理虚拟环境。

```bash
pip install virtualenv
pip install virtualenvwrapper
```

但是安装之后并不能直接使用，我们需要配置之后才能使用相关命令。

配置环境变量

```bash
vim /etc/profile
```

添加如下内容

```conf
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source /usr/local/bin/virtualenvwrapper.sh
```

生效环境变量

```bash
source /etc/profile
```

加入上面的环境变量是为了防止 python 没有安装 virtualenvwrapper，而报 `/usr/bin/python: No module named virtualenvwrapper` 提示。当然如果你安装了可以不加

其他用户使用，请注销登录后使用该用户登录。以便初始化 `source /usr/local/bin/virtualenvwrapper.sh` 。

第一条是配置 virtualenvwrapper 默认解释器为 python3 ，第二条是用 virtualenvwrapper

#### 使用 virtualenvwrapper

1. 创建 python 虚拟环境

```
[root@localhost ~]# mkvirtualenv testenv
Using base prefix '/usr'
New python executable in /root/.virtualenvs/testenv/bin/python3.6
Also creating executable in /root/.virtualenvs/testenv/bin/python
Installing setuptools, pip, wheel...done.
virtualenvwrapper.user_scripts creating /root/.virtualenvs/testenv/bin/predeactivate
virtualenvwrapper.user_scripts creating /root/.virtualenvs/testenv/bin/postdeactivate
virtualenvwrapper.user_scripts creating /root/.virtualenvs/testenv/bin/preactivate
virtualenvwrapper.user_scripts creating /root/.virtualenvs/testenv/bin/postactivate
virtualenvwrapper.user_scripts creating /root/.virtualenvs/testenv/bin/get_env_details
(testenv) [root@localhost ~]# 
(testenv) [root@localhost ~]# ll ~/.virtualenvs/testenv/
total 8
drwxr-xr-x 2 root root 4096 Apr  3 03:24 bin
drwxr-xr-x 2 root root   24 Apr  3 03:24 include
drwxr-xr-x 3 root root   23 Apr  3 03:24 lib
lrwxrwxrwx 1 root root    3 Apr  3 03:24 lib64 -> lib
-rw-r--r-- 1 root root   60 Apr  3 03:24 pip-selfcheck.json

(testenv) [root@localhost ~]# 
```

初次创建完成虚拟环境，会自动使用。退出虚拟环境请使用 `deactivate`

2. 使用虚拟环境

使用 `workon` 命令，直接 `tab` 提示即可列出当前虚拟环境。指定名称后使用

```
[root@localhost ~]# workon test
testenv   test_env  
```

3. 关于 virtualenv 和 virtualenvwrapper 的更多使用方法请参考官方文档。另外再推荐一个虚拟环境管理工具 `pipenv` 。请自行权衡利弊后选择使用
