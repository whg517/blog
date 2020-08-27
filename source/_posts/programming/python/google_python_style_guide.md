
---
title: Google Python 风格指南
author: Kevin
date: 2020-08-04 10:06:00
updated: 2020-08-04 11:50:00
tags:
- Python
categories: 开发
---

<a id="s1-background"></a>
<a id="1-background"></a>

<a id="background"></a>
## 1 背景

在 Google ， Python 是主要使用的动态语言之一。该风格之南表述了一些在 Python 编程中 **应该做** 和 **不应该** 的事情。

为了帮你正确地格式化代码，我们准备了 [Vim 的配置文件](https://google.github.io/styleguide/google_python_style.vim) 。
对于 Emacs ，使用默认的应该没什么问题。

很多个团队在使用 [yapf](https://github.com/google/yapf/) 来自动格式化来避免在格式上的争论。


<a id="s2-python-language-rules"></a>
<a id="2-python-language-rules"></a>

<a id="python-language-rules"></a>
## 2 Python 语言规范


<a id="s2.1-lint"></a>
<a id="21-lint"></a>

<a id="lint"></a>
### 2.1 Lint

在你的代码上运行 `pylint` 


<a id="s2.1.1-definition"></a>
<a id="211-definition"></a>

<a id="definition"></a>
#### 2.1.1 定义

`pylint` 是一个可以在 Python 源代码中发现 Bugs 和风格问题的工具。它可以发现一些通常由编译器捕获的问题，像 C 和 c++ 等
动态程度较低的语言。由于Python的动态特性，一些警告可能是不正确的;然而，虚假得警告应该相当少见。

#### 2.1.2 Pros

捕获容易出错的错误，例如错别字，在分配前使用vars等。

#### 2.1.3 Cons

`pylint` 并不完美。要利用它，有时我们需要：

- 围绕它写
- 禁止其警告
- 对其进行改进

#### 2.1.4 Decision

确定在你的代码中运行  `pylint` 。

如果不适当，请禁用警告，以免隐藏其他问题。要取消显示警告，可以设置行级注释：

```python
dict = 'something awful'  # Bad Idea... pylint: disable=redefined-builtin
```

`pylint` 警告均由符号名称(`empty-docstring`) 标识， Google 特定警告以 `g-` 开头。

如果从符号名称中看不到禁用的原因，请添加说明。以这种方式进行禁用的优势在于，我们可以轻松地搜索抑制并重新进行禁用。

要获取 `pylint` 警告列表可以这样操作：

```shell
pylint --list-msgs
```

获取特定消息的详细信息，执行：

```shell
pylint --help-msg=C6409
```

推荐使用 `pylint：disable` 而不是使用旧的 `pylint：disable-msg` 形式。

可以通过删除函数开头的变量来消除未使用的参数警告。记得添加注释解释为什么删除。 `Unused` 就可以了。例如：

```python
def viking_cafe_order(spam, beans, eggs=None):
    del beans, eggs  # Unused by vikings.
    return spam + spam + spam
```

取消警告的其他常见形式包括使用 `_` 作为未使用参数的标识符，或在参数名称前加上 `unused_` 前缀，
或将其分配给 `_` 。虽然可以这么做，但不再鼓励。一些中断调用程序按名称传递参数，并且不强制要求参数实际上未使用。

### 2.2 Imports

`import` 语句仅用在包和模块，不要用在类和函数上。对于 [typing module](#typing-imports) 导入是特例。

#### 2.2.1 定义

Re-usability mechanism for sharing code from one module to another.

#### 2.2.2 优点

命名空间管理约定很简单。每个标识符的来源以一致的方式表示； `x.Obj` 表示 `Obj` 对象定义在 `x` 模块中。

#### 2.2.3 缺点

模块名称仍然可以冲突。一些模块名称很长。

#### 2.2.4 判断

* 使用 `import x` 导入一个包或模块。
* 使用 `from x import y ` ， `x` 包的前缀， `y` 是不带



<a id="typing-imports"></a>

