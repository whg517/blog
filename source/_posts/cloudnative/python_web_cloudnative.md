---
title: Python Web CloudNative [未完]
author: kevin
date: 2020-05-28 11:50:00
updated: 2020-05-28 11:57:00
tags: 
- python 
- fastapi
- react
- gitlab-ci
categories: CloudNative
---

现在 CloudNative 那么火，当前基础设施环境也越来越成熟。诸如 Docker，K8s 到现在 DCI 标准的出现，使得 DevOps 越来越方便。本文以个人开发时总结的
一套 Python Web 开发在使用 gitalb-ci 和 Docker 环境上的实践。当然这只是第一版，以后也会在使用中不断优化开发管理和运维流程。

<!-- more -->

## 一、说明

本项目以单 Web 项目为示例，以抛砖引玉。项目不注重业务功能，以最简单业务功能，模拟 Web 访问场景。但项目开发和组织是结合当前个人总结的开发实践来做的。如有缺陷，还望斧正。

项目以前后端分离。后端采用 [fastapi](https://fastapi.tiangolo.com/) 框架开发，现在仅提供一个访问端点，后续优化可能会逐步增加数据库访问等依赖服务，尽可能模拟真实中小型 Web 开发场景。前端项目采用 [react](https://reactjs.org/) 开发，使用 nodejs 构建成静态文件后由 nginx 加载后对外提供访问地址。前端页面调用后端服务。

## 二、后端开发

项目以 `httpbin` 命名，目的是在做示例项目之后，该项目可以像正常 [httpbin](https://httpbin.org/) 一样可用。

项目 Python 环境为 `Python3.7` 。主要是 `fastapi` 是一个 [ASGI](https://asgi.readthedocs.io/en/latest/) Web 框架，在 `Python 3.7` 异步支持更好。

### 1. 项目初始化

安装 [cookiecutter](https://github.com/cookiecutter/cookiecutter)

```shell script
pip install cookiecutter
```

使用已有项目模板创建项目，这个项目模板也是个人在实践过程中总结的通用 Python 项目模板。

如果无法使用，可以手动创建

```
cookiecutter http://git.tendata.com.cn/tendata/bigdata/cookiecutter-tendata-python.git
```

项目模板结构如下：

```
.
├── httpbin
│   └── __init__.py
├── Pipfile
├── README.md
├── setup.cfg
├── setup.py
├── tests
│   └── __init__.py
└── tox.ini
```

#### 1.1 项目内容

如果已经使用模板生成可以跳过

`httpbin/__init__.py`

```
__version__ = '0.1.0'
```



`Pipfile`

```toml
[[source]]
name = "pypi"
url = "https://pypi.org/simple"
verify_ssl = true

[dev-packages]
tox = "*"
isort = "*"
flake8 = "*"
pytest = "*"

[packages]

[requires]
python_version = "3.7"

```

项目依赖使用 [pipenv](https://github.com/pypa/pipenv) 管理。初始模板带有四个开发环境时要使用的库。



`README.md`

```markdown
# httpbin

This is a demo restful app developed using [fastapi web framework](https://fastapi.tiangolo.com/)
```



`setup.cfg`

```ini
[metadata]
name = httpbin
version = attr: httpbin.__version__
author = wanghuagang
author_email = huagang517@126.com
description = This is a demo resuful project
keywords = ['httpbin', 'fastaip', 'restful']
long_description = file: READMD.md
long_description_content_type="text/markdown",
classifiers =
    Operating System :: OS Independent
    Programming Language :: Python :: 3.7

[options]
python_requires= >=3.6
zip_safe = False
include_package_data = True
packages = find:
install_requires =
    fastapi
    uvicorn

[options.packages.find]
exclude = 
    tests
    doc

[flake8]
max-line-length = 120
exclude =
    build
    .tox
    .git,

[isort]
not_skip = __init__.py
skip =
    .tox

[tool:pytest]
testpaths = tests
python_files = tests.py test_*.py *_tests.py
```

为了减少源数据散落各处，项目打包源数据放在 `setup.cfg` 中，这是 [setuptools 30.3.0](https://setuptools.readthedocs.io/en/latest/setuptools.html?highlight=setup.cfg#configuring-setup-using-setup-cfg-files) 后新增的功能，所以在使用的时候，要注意版本。此文件中还放了其他库的要配置源数据。

项目依赖 `install_requires` 中包含了两个库，一个是 Web 框架 `fastapi` ，两一个是 `ASGI` 库 `uvicorn` 。这是运行 `fastapi` 程序需要用到的。



> 注意： `version = attr: httpbin.__version__` 使用了 [`attr:`](https://setuptools.readthedocs.io/en/latest/setuptools.html?highlight=setup.cfg#specifying-values) 特殊指令，其作用相当于 `import` ，如果 `httpbin/__init__.py` 中导入了第三方依赖，在执行 `tox` 或者在没有外部依赖的环境下执行 `python setup.py` 打包的时候会报缺少外部依赖的问题。因为依赖调用，而环境中又没有安装。此时要么把依赖从 `httpbin/__init__.py` 中移出去，要么在 `setuo.py` 中使用 `open` 方法读取 `httpbin/__init__.py` 文件，然后用正则匹配出对版本号。`re.search(r"__version__ = ['\"]([^'\"]+)['\"]", open(['httpbin', '__version__.py']).read(), re.M).group(1)`



`setup.py`

```python
import setuptools

setuptools.setup()
```

这个文件就不需要写太多参数啦。



`tox.ini`

```ini
[tox]
skipsdist = true
envlist = py37

[testenv:py37]
usedevelop = true
deps = 
    pipenv
commands =
    pipenv install -d
    pytest
    isort --recursive --check-only --diff
    flake8
```



至此，项目模板就是这样。



#### 1.2 初始开发环境

使用 pipenv 初始化虚拟环境

```shell
pipenv install
```

使用 pipenv 安装 `fastapi`

```shell
pipenv install fastapi
```

#### 1.3 初始化git

```shell
git init
```

初始化 git 仓库，用于版本管理。这里记得创建 `.gitignore` 文件。



### 2. 项目开发

#### 1.1 接口和路由

新建文件 `httpbin/routers.py` ，文件内容如下：

```python
from datetime import datetime

from fastapi.routing import APIRouter

router = APIRouter()


@router.get('/')
def index():
    return {'now': datetime.now().strftime('%Y-%m-%d %H:%D:%S')}
```

这里仅创建了一个接口，接口默认返回 JSON 数据。

#### 1.2 主程序

新建 `httpbin/server.py` ，内容如下：

```python
from fastapi import FastAPI, __version__

from httpbin.routers import router

app = FastAPI(
    title='httpbin',
    description='This is demo fastapi project',
    version=__version__
)

app.include_router(router)
```

首先初始化一个 `FastAPI` 对象，然后把前面定义的路由加载进去。

### 3. 测试

测试框架使用 [pytest](https://docs.pytest.org/en/latest/) ,由于其强大的的特性和丰富的扩展插件，在编写测试的上更高效。

#### 3.1. 配置测试

新建 `tests/conftest.py` ，内容如下：

```python
import pytest
from starlette.testclient import TestClient

from httpbin.server import app


@pytest.fixture
def client():
    yield TestClient(app)
```

`conftest.py` 为 pytest 的配置文件，在这里定义其他测试所依赖的内容。

`fixture` 是一个很强大的功能，在使用 `yield` 关键字的时候可以让你直接在编写测试方法的情况下获得 `setup` 和 `teardown` 的效果。其 `scope` 参数可以配置 `function` 、 `class` 或者其他级别。具体请参考文档。

测试接口使用 `starlette.TestClient` ，其依赖 `requests` 库，所以要记得安装

```shell
pipenv install -d requests
```



#### 3.2.编写测试文件



新建测试文件 `tests/test_api.py` ，内容如下：

```python
import json


def test_index(client):
    response = client.get('/')
    assert response.ok
    assert response.status_code == 200
    assert json.loads(response.text).get('now')
```



#### 3.3. 运行测试

```shell
pytest
```



### 4. 代码质量提升

#### 4.1 运行 isort

[isort](https://github.com/timothycrosley/isort) 是一款自动格式化到包顺序的工具，运行后根据提示操作。格式化完成后，每个文件的导包会根据配置格式化。运行后有助于提升代码风格。

```ini
isort
```

#### 4.2 运行 flake8

[flake8](https://flake8.pycqa.org/en/latest/)  是一款检测你的代码是否符合 [PEP8](https://www.python.org/dev/peps/pep-0008/#indentation) 规范的工具。运行后有助于提升代码质量。

```shell
flake8
```

### 5. 自动化

[tox](https://tox.readthedocs.io/en/latest/index.html) 是一个自动化工具，通过配置可以编排需要执行的操作。

```shell
tox
```

在 `tox.ini` 文件中已经编写了对应的规则。一般在最后确定开发完成之后再运行一遍，用于最后检查项目是否还存在问题。



### 6. 运行项目

以可编辑模式安装项目

```bash
pip install -e .
```

运行

```shell
uvicorn httpbin.server:app --port 8000 --host 127.0.0.1
```

端口和地址可以不指定。

访问地址 [http://127.0.0.1:8000](http://127.0.0.1:8000) 即可访问首页内容。

`fastapi` 还提供了 [`openapi 3.0规范`](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md)  的接口文档页面和对应的 schema 。访问 [http"//127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 页面就能看到当前所有接口，方便前后端对接和调试。



在运行过程中如果发现异常问题，要及时排查和调整。修复后重复运行 `tox` 对项目重新自动化检测。



### 7. 提交代码

提交代码前一定要运行一次 `tox` 。

```bash
git add .
git commit -m "feat: 完善 httpbin 基本功能，测试正常。"
```



## 三、后端-云原生

### 1. 增加 Dockerfile

创建 ``Dockerfile` ，文件内容如下：

```dockerfile
# ##########################################################
# Build stage.
# Build python distribute package.
FROM python:3.7 as build

ENV PIP_INDEX_URL=https://mirrors.aliyun.com/pypi/simple/

WORKDIR /app

ADD Pipfile Pipfile.lock  ./

RUN pip install --no-cache-dir -U pip \
    && pip3 install --no-cache-dir pipenv \
    && pipenv install --clear --system \
    && rm -rf ~/.cache/pipenv

ADD httpbin ./httpbin
ADD tests ./tests
ADD setup.cfg setup.py README.md ./

RUN python setup.py bdist_wheel

# ##########################################################
# Distribute stage
FROM python:3.7

LABEL name=httpbin

ENV PIP_INDEX_URL=https://mirrors.aliyun.com/pypi/simple/

# Copy build artifact from upstream stage.
COPY --from=0 /app/dist/ /tmp/dist/

RUN pip install --no-cache-dir -U pip \
    && pip install --no-cache-dir /tmp/dist/*.whl

ENTRYPOINT ["uvicorn"]
CMD ["httpbin:app",  "--port", "80", "--host", "0.0.0.0"]
```

为了减少生成镜像的层数，构建过程采用多[阶段构建](https://docs.docker.com/develop/develop-images/multistage-build/)，最后阶段为发布的镜像，使用的依赖在前一阶段构建。

记得生成 `.dockerignore` 文件，排除不必要的内容，尽量缩减镜像层数和每层的大小。

#### 1.1 测试 Docker

构建 Docker 镜像

```bash
docker build -t httpbin:latest .
```

构建完成后会生成对应镜像



#### 1.2 运行

```shell
docker run --rm httpbin
```



### 2. 增加 ci/cd 

我使用的是 gitlab，所以后面 gitlab-ci 内容。以后有机会会增加其他 ci 工具。

新建 `.gitlab-ci.yml` ，内容如下

```yml
stages:
  - test
  - build
  - upload

# This base py37 env, you should extend it in your stage.
# Use start with dot (.) to hide stage, ci will ignore it.
.py37:
  image: python:3.7
  script:
    - pip install -U pip

test:
  stage: test
  extends:
    - .py37
  script:
    - pip install -U tox
    - tox

build whl:
  stage: build
  image: python:3.7
  script:
    - python setup.py bdist_wheel
  artifacts:
    expire_in: 7 days
    paths:
      - ./dist/*.whl

# Build docker image only when release.
build image:
  stage: build
  image: docker:19
  when: on_success
  only:
    refs:
      - tags
  script:
    - docker build -t $CI_PROJECT_TITLE:$CI_COMMIT_REF_NAME -t $CI_PROJECT_TITLE:latest .

# Upload to pypi only when release
upload twine:
  stage: upload
  extends:
    - .py37
  when: on_success
  only:
    refs:
      - tags
    variables:
      - $TWINE_USERNAME
      - $TWINE_PASSWORD
  script:
    - pip install -U twine
    - twine upload dist/*.whl
```

此 CI 逻辑为一般提交只运行测试和构建 whl 文件，当发布的时候会根据 release 的版本号，构建 Docker 镜像，同时将归档文件上传到 pypi 索引服务器。
