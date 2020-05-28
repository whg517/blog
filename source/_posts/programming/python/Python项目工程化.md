---
title: Python 项目工程化
author: Kevin
date: 2018-10-12 16:38:00
updated: 2020-05-28 11:50:00
tags:
- Python
categories: 开发
---

如果你是新手，请先参考 [The Hitchhiker’s Guide to Python!](https://docs.python-guide.org/) 。
当然它有[中文版](https://pythonguidecn.readthedocs.io/zh/latest/) 

<!-- more -->

## 一、环境准备

### 1. 语言版本

Python version 3.6

[Python 3.6](https://www.python.org/downloads/release/python-366/) / [Anaconda 5.2.0(Python 3.6.5)](https://repo.anaconda.com/archive/) / [Mininconda3-4.5.4(Python 3.6)](https://repo.continuum.io/miniconda/)

在这里推荐使用 Miniconda 或者 Anaconda 安装 Python。方便管理 Python。上面各处的版本均为对应对心的 Python 3.6 版本。在使用 conda 安装 Python 的时候，推荐将 conda 注册到 PATH 中。

### 2. 虚拟环境工具

#### virtualenv

建议开发中使用

#### virtualenvwrapper

集中化管理虚拟环境，建议部署的时候使用。

#### pipenv

类似于 Node.js 的 npm 或者 Ruby 的 bundler 的项目依赖管理器。它是一种更高级的工具，但是还不是很稳定。

直到在写这篇文章，pipenv 仍会因为 pip 18.1 的更新出现不可预测的 Bug。暂时不推荐使用。

### 3. 开发工具

#### Pycharm / IntelliJ IDEA

两者都有 JetBrains 开发。[Pycharm](http://www.jetbrains.com/pycharm/) 专注 Python 开发。而后者更侧重于 Java 开发。IDEA可以通过 Python 插件使用前者大多数特性。

推荐 Pycharm。

工具依赖 Java 环境，运行前请确保系统存在 JRE (Java Runtime Environment)。

#### Visual Studio Code

[Visual Studio Code IDE](https://code.visualstudio.com/) 是由微软开发的一款免费的、轻量的、开源的 IDE。支持 Mac、Windows 和 Linux。可通过安装 [Python 插件](https://marketplace.visualstudio.com/items?itemName=ms-python.python) 使其支持 Python 开发。

## 二、开发规范

### 1. 语言使用规范

- [Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/)
- [The Zen of Python](https://www.python.org/dev/peps/pep-0020/)
- [Google Python Style Guide](https://github.com/google/styleguide/blob/gh-pages/pyguide.md) 

前两者是 Python 指定的语言规范和风格规范。最后一个是 Google 社区根据Python官方规范和使用过程中总结的经验结合在一起推出的一套Python开源规范。

### 2. 项目开发规范

### 3. 项目结构

#### 3.1 一般项目结构

```
project_name
|-- docs
|-- project_name
    |-- project_module
|-- tests
|-- scripts
|-- setup.cfg
|-- setup.py
|-- tox.ini
|-- README.md
```

#### 3.2 Django 项目结构

```
project_name
|-- docs
|-- apps
    |-- app1
|-- static
|-- templates
|-- project_name
|-- manage.py
|-- pytest.ini
|-- scripts
|-- setup.cfg
|-- setup.py
|-- tox.ini
|-- README.md
```

