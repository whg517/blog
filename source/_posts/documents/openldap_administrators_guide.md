---
title: OpenLDAP Software 2.4 Administrator's Guide
author: Kevin
date: 2017-11-20 14:52:25
updated: 2020-05-28 11:50:00
tags: openldap
categories: Documents
---

由于在使用 Openldap 苦于直接看以英文文档太麻烦，反正也是要阅读就边看边做笔记，于是有了次文档。记录内容呢也只是根据自己需要选择性翻译。
文章内容直接由 google 翻译软件翻译，可能存在不通顺之处，还望大家多多指出。在 markdown 转换到 html 过程中可能有部分尖括号引用内容，
导致丢失或者显示有误，请大家参照官方文档对比。

<!-- more -->

后期我会不定期优化文档，补全文档。

The OpenLDAP Project <<http://www.openldap.org/>> 

## 2 快速入门指南

以下是OpenLDAP软件2.4的快速入门指南，包括独立版 LDAP守护进程，*slapd*（8）。

这意味着让您了解安装和配置[OpenLDAP软件](http://www.openldap.org/software/)所需的基本步骤。应与本文档的其他章节，手册页以及发行版提供的其他材料（如`INSTALL`文档）或[OpenLDAP](http://www.openldap.org/)网站（[http://www.OpenLDAP.org](http://www.openldap.org/)）一起使用OpenLDAP软件常问问题（[http://www.OpenLDAP.org/faq/?file=2](http://www.openldap.org/faq/?file=2)）。

如果您打算认真运行OpenLDAP软件，那么在尝试安装软件之前，您应该查看所有这些文档。

**注意：**此快速入门指南不使用强身份验证，也不使用任何完整性或机密保护服务。这些服务在OpenLDAP管理员指南的其他章节中介绍。

### 2.1 获取软件

您可以按照OpenLDAP软件下载页面（<http://www.openldap.org/software/download/>）上的说明**获取软件**副本。建议新用户从最新*版本*开始。 

### 2.2 打开分发包打开

源目录，将目录更改到那里，然后使用以下命令解压缩分发：

```shell
gunzip -c openldap-VERSION.tgz | tar xvfB -
```

然后重新定位到分发目录中：

```shell
cd openldap-VERSION
```

您必须使用版本名称替换`VERSION`。 

### 2.3 审查文件 

您现在应该查看发行版`随附`的`版权`，`许可证`，`自述`文件和`INSTALL`文档。该`版权`和`许可`提供可接受的使用，复制和OpenLDAP软件的保修期限制信息。 

您还应该查看本文档的其他章节。特别是，本文档的“ [构建和安装OpenLDAP软件”](http://www.openldap.org/doc/admin24/install.html)一章提供了有关必备软件和安装步骤的详细信息。 

### 2.4 运行配置

您将需要运行提供的`配置`脚本来*配置*在系统上构建的分发。该`配置`脚本接受许多命令行选项启用或禁用可选的软件功能。通常默认值是可以的，但是您可能想要更改它们。要获取`配置`接受的选项的完整列表，请使用`--help`选项：

```shell
./configure --help
```

但是，考虑到您正在使用本指南，我们假设您是勇敢的，只需让`配置`确定什么是最好的：

```shell
./configure
```

假设`configure`不会不喜欢你的系统，你可以继续构建软件。如果`configure`没有抱怨，那么您可能需要去软件常见问题解答*安装* 部分（<http://www.openldap.org/faq/?file=8>）和/或实际阅读“ [构建和安装OpenLDAP软件”](http://www.openldap.org/doc/admin24/install.html)一章的文件。 

### 2.5 构建软件

下一步是构建软件。这一步有两个部分，首先我们构建依赖关系，然后编译软件：

```shell
make depend
make
```

两个都应该完成没有错误。 

2.6 测试构建

为了确保正确的构建，您应该运行测试套件（只需要几分钟）：

```shell
make test
```

适用于您的配置的测试将运行，它们应该通过。可能会跳过某些测试，如复制测试。 

### 2.7 安装软件

您现在可以安装该软件了; 这通常需要*超级用户*权限：

```shell
su root -c 'make install'
```

现在应该将所有内容安装在`/ usr / local下`（或者`configure`使用的任何安装前缀）。 

### 2.8 编辑配置文件

使用您喜欢的编辑器编辑提供的*slapd.ldif*示例（通常安装为`/usr/local/etc/openldap/slapd.ldif`）以包含MDB数据库定义的形式：

```
dn: olcDatabase=mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
OlcDbMaxSize: 1073741824
olcSuffix: dc=<MY-DOMAIN>,dc=<COM>
olcRootDN: cn=Manager,dc=<MY-DOMAIN>,dc=<COM>
olcRootPW: secret
olcDbDirectory: /usr/local/var/openldap-data
olcDbIndex: objectClass eq
```

请务必使用您域名的相应网域组件替换`<MY-DOMAIN>`和`<COM>`。例如，对于`example.com`，请使用：

```
dn: olcDatabase=mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
OlcDbMaxSize: 1073741824
olcSuffix: dc=example,dc=com
olcRootDN: cn=Manager,dc=example,dc=com
olcRootPW: secret
olcDbDirectory: /usr/local/var/openldap-data
olcDbIndex: objectClass eq
```

如果您的域包含其他组件，例如`eng.uni.edu.eu`，请使用：

```
dn: olcDatabase=mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
OlcDbMaxSize: 1073741824
olcSuffix: dc=eng,dc=uni,dc=edu,dc=eu
olcRootDN: cn=Manager,dc=eng,dc=uni,dc=edu,dc=eu
olcRootPW: secret
olcDbDirectory: /usr/local/var/openldap-data
olcDbIndex: objectClass eq
```

### 2.9 导入配置数据库

现在，您可以通过运行以下命令来导入您的配置数据库供*slapd*（8）使用：

```shell
su root -c /usr/local/sbin/slapadd -n 0 -F /usr/local/etc/slapd.d -l /usr/local/etc/openldap/slapd.ldif
```

### 2.10 启动SLAPD

您现在可以通过运行以下命令启动独立LDAP守护程序*slapd*（8）：

```shell
su root -c /usr/local/libexec/slapd -F /usr/local/etc/slapd.d
```

要检查服务器是否正在运行和配置，您可以使用*ldapsearch*（1）运行搜索。默认情况下，*ldapsearch*安装为`/usr/local/bin/ldapsearch`：

```
ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts
```

请注意在命令参数周围使用单引号来防止shell解释特殊字符。这应该返回：

```
dn:
namingContexts: dc=example,dc=com
```

有关运行*slapd*（8）的详细信息，请参见*slapd*（8）手册页和本文档的[Running slapd](http://www.openldap.org/doc/admin24/runningslapd.html)章节。 

### 2.11 将初始条目添加到您的目录

您可以使用*ldapadd*（1）向LDAP目录添加条目。*ldapadd*希望输入LDIF形成。我们将分两步进行：

1. 创建一个LDIF文件
2. 运行ldapadd

使用您喜欢的编辑器并创建一个包含以下内容的LDIF文件：

```
dn: dc=<MY-DOMAIN>,dc=<COM>
objectclass: dcObject
objectclass: organization
o: <MY ORGANIZATION>
dc: <MY-DOMAIN>

dn: cn=Manager,dc=<MY-DOMAIN>,dc=<COM>
objectclass: organizationalRole
cn: Manager
```

请务必使用您域名的相应网域组件替换`<MY-DOMAIN>`和`<COM>`。 应以组织名称替换`<我的组织>`。剪切和粘贴时，请确保从示例中修剪任何前导和尾随空格。

```
dn: dc=example,dc=com
objectclass: dcObject
objectclass: organization
o: Example Company
dc: example

dn: cn=Manager,dc=example,dc=com
objectclass: organizationalRole
cn: Manager
```

现在，您可以运行*ldapadd*（1）将这些条目插入到目录中。

```
ldapadd -x -D "cn=Manager,dc=<MY-DOMAIN>,dc=<COM>" -W -f example.ldif
```

请务必使用您域名的相应网域组件替换`<MY-DOMAIN>`和`<COM>`。系统将提示您输入`slapd.conf中`指定的“ `secret` ” 。例如，对于`example.com`，请使用：

```
ldapadd -x -D "cn=Manager,dc=example,dc=com" -W -f example.ldif
```

其中`example.ldif`是您上面创建的文件。

有关目录创建的其他信息，请参见本文档的“ [数据库创建和维护工具”](http://www.openldap.org/doc/admin24/dbtools.html)一章。 

### 2.12 看看它是否工作

现在我们准备好验证添加的条目在您的目录中。您可以使用任何LDAP客户端来执行此操作，但我们的示例使用*ldapsearch*（1）工具。请记住将`dc=example,dc=com`替换为您网站的正确值：

```
ldapsearch -x -b 'dc=example,dc=com' '(objectclass=*)'
```

此命令将搜索并检索数据库中的每个条目。

您现在可以使用*ldapadd*（1）或其他LDAP客户端添加更多条目，尝试各种配置选项，后端安排等。

请注意，默认情况下，*slapd*（8）数据库授予对*超级用户*（除了`rootdn`配置指令指定）之外的*所有人的读访问权限*。强烈建议您建立控制以限制对授权用户的访问。访问[控制](http://www.openldap.org/doc/admin24/access-control.html)章节讨论[访问控制](http://www.openldap.org/doc/admin24/access-control.html)。还鼓励您阅读[安全注意事项](http://www.openldap.org/doc/admin24/security.html)，[使用SASL](http://www.openldap.org/doc/admin24/sasl.html)和[使用TLS](http://www.openldap.org/doc/admin24/tls.html)部分。**``

以下章节提供了有关制作，安装和运行*slapd*（8）的更多详细信息。



## 5 配置 slapd

OpenLDAP 2.3和更高版本已经转换为使用动态运行时配置引擎 *slapd-config*（5）

- 完全启用了LDAP
- 使用标准LDAP操作进行管理
- 将其配置数据存储在 LDIF数据库，一般在`/usr/local/etc/openldap/slapd.d`目录下。
- 允许slapd的所有配置选项随时更改，通常不要求服务器重新启动，以使更改生效

本章介绍*slapd-config*（5）配置系统的一般格式，其次是常用设置的详细说明。

旧版本*slapd.conf*（5）文件仍然受支持，但其使用已被弃用，并且对其的支持将在未来的OpenLDAP版本中被撤销。在下一章将介绍通过*slapd.conf*（5）配置*slapd*（8）。

**注：**虽然*slapd-config*（5）系统存储了其作为（基于文本）LDIF文件的配置，你应该*永远不要*直接编辑任何LDIF文件。应通过LDAP操作执行配置更改，例如*ldapadd*（1），*ldapdelete*（1）或*ldapmodify*（1）。

### 5.1 配置布局

slapd配置存储为具有预定义模式和DIT的特殊LDAP目录。有特定的objectClasses用于承载全局配置选项，模式定义，后端和数据库定义以及各种其他项目。示例配置树如图5.1所示。

![图 5.1](https://qiniu.iclouds.work/FmRtJjELzSWkM7K8VC21jahAN4SS.png)

其他对象可以是配置的一部分，但是为了清楚起见，从图示中省略。

该 *slapd-config* 配置树有一个非常特殊的结构。树的根名为`cn=config` ，并包含全局配置设置。附加设置包含在单独的子条目中：

- 动态加载的模块
  - 只有使用`--enable-modules`选项来配置软件时，才可以使用这些。
- 模式定义
  - 该`cn=schema，cn=config`条目包含了系统架构（所有架构是硬编码的slapd）。 `cn=schema` 的
    子条目，`cn=config`包含从配置文件加载或在运行时添加的用户模式。
- 后端特定配置
- 数据库特定配置
  - 数据库条目的子节点定义了覆盖。
  - 数据库和叠加也可能有其他杂项的子项。

LDIF文件的常规规则适用于配置信息：以 `＃` 字符开头的注释行将被忽略。如果一行以单个空格开始，它被认为是前一行的延续（即使前一行是注释），并且单个前导空格被删除。条目用空白行分隔。



配置LDIF的一般布局如下：

```
# global configuration settings
dn: cn=config
objectClass: olcGlobal
cn: config
<global config settings>

# schema definitions
dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema
<system schema>

dn: cn={X}core,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: {X}core
<core schema>

# additional user-specified schema
...

# backend definitions
dn: olcBackend=<typeA>,cn=config
objectClass: olcBackendConfig
olcBackend: <typeA>
<backend-specific settings>

# database definitions
dn: olcDatabase={X}<typeA>,cn=config
objectClass: olcDatabaseConfig
olcDatabase: {X}<typeA>
<database-specific settings>

# subsequent definitions and settings
...
```

上面列出的一些条目的名称中有数字索引`“{X}”`。虽然大多数配置设置具有固有的顺序依赖性（即，一个设置必须在后续设置之前生效），LDAP数据库本质上是无序的。数字索引用于在配置数据库中强制执行一致的排序，以便保留所有排序依赖关系。在大多数情况下，不必提供指数; 它将根据创建条目的顺序自动生成。

配置指令被指定为各个属性的值。slapd配置中使用的大多数属性和对象类在其名称中具有前缀`“olc”` （OpenLDAP Configuration）。通常，属性和旧式`slapd.conf`配置关键字之间存在一一对应关系，使用关键字作为属性名称，并附加“olc”前缀。

配置指令可能会引用参数。如果是这样，这些参数就被空格分开。如果一个参数包含空格，参数应该用双引号括起来`“像这样”`。在下面的描述中，应该用实际文本替换的参数在括号`<>`中显示。

该分发包含将安装在`/usr/local/etc/openldap`目录中的示例配置文件。`/usr/local/etc/openldap/schema`目录中也提供了一些包含模式定义（属性类型和对象类）的文件。



### 5.2 配置指令

节详细介绍常用的配置指令。有关完整列表，请参阅*slapd-config*（5）手册页。本节将以自上而下的顺序处理配置指令，从`cn=config`条目中的全局伪指令开始。将对每个指令及其默认值（如果有）以及其使用示例进行描述。

#### 5.2.1. cn=config

此条目中包含的指令通常适用于整个服务器。他们大多数是系统或连接导向，而不是数据库相关。此条目必须具有`olcGlobal objectClass`。

##### 5.2.1.1 olcIdleTimeout: &lt;integer&gt;

指定强制关闭空闲客户端连接之前等待的秒数。值为0，默认值，禁用此功能。

##### 5.2.1.2 olcLogLevel: \<integer&gt;

此指令指定调试语句和操作统计信息应该被syslog的级别（当前记录到*syslogd*（8）`LOG_LOCAL4`设施）。您必须配置OpenLDAP `--enable-debug`（默认值）才能工作（除了始终启用的两个统计级别之外）。日志级别可以被指定为整数或关键字。可以使用多个日志级别，并且级别是相加的。要显示什么级别对应于什么样的调试，用`-d?`调用slapd 或查阅下表。\<level\>的可能值为：

表5.1：调试级别

| **Level** | **Keyword**    | **Description**                          |
| --------- | -------------- | ---------------------------------------- |
| -1        | any            | enable all debugging                     |
| 0         |                | no debugging                             |
| 1         | (0x1 trace)    | trace function calls                     |
| 2         | (0x2 packets)  | debug packet handling                    |
| 4         | (0x4 args)     | heavy trace debugging                    |
| 8         | (0x8 conns)    | connection management                    |
| 16        | (0x10 BER)     | print out packets sent and received      |
| 32        | (0x20 filter)  | search filter processing                 |
| 64        | (0x40 config)  | configuration processing                 |
| 128       | (0x80 ACL)     | access control list processing           |
| 256       | (0x100 stats)  | stats log connections/operations/results |
| 512       | (0x200 stats2) | stats log entries sent                   |
| 1024      | (0x400 shell)  | print communication with shell backends  |
| 2048      | (0x800 parse)  | print entry parsing debugging            |
| 16384     | (0x4000 sync)  | syncrepl consumer processing             |
| 32768     | (0x8000 none)  | only messages that get logged whatever log level is set |

期望的日志级别可以作为单个整数输入，该整数将（ORed）所需的级别（十进制或十六进制符号）组合为整数列表（在内部进行ORed操作），或作为显示的名称列表在括号之间，

```
olcLogLevel 129
olcLogLevel 0x81
olcLogLevel 128 1
olcLogLevel 0x80 0x1
olcLogLevel acl trace
```

是等同的

**例子：**

```
olcLogLevel -1
```

这将导致大量的调试信息被记录。

```
olcLogLevel conns filter
```

只需记录连接并搜索过滤器处理。

```
olcLogLevel none
```

记录无论配置的日志级别如何记录的消息。这不同于在不发生日志记录时将日志级别设置为0。至少需要“ `无”`级才能记录高优先级消息。

**默认：**

```
olcLogLevel stats
```

默认情况下配置基本统计记录。但是，如果未定义`olcLogLevel`，则不会发生日志记录（相当于0级）。

##### 5.2.1.3 olcReferral &lt;URI&gt;

此指令指定当slapd找不到本地数据库来处理请求时要传回的引用。

**例：**

```
olcReferral: ldap://root.openldap.org
```

这将在OpenLDAP项目中将非本地查询引用到全局根LDAP服务器。智能LDAP客户端可以在该服务器上重新询问他们的查询，但请注意，大多数这些客户端只会知道如何处理包含主机部分和可选的可分辨名称部分的简单LDAP URL。

##### 5.2.1.4 条目样例

```
dn: cn=config
objectClass: olcGlobal
cn: config
olcIdleTimeout: 30
olcLogLevel: Stats
olcReferral: ldap://root.openldap.org
```

#### 5.2.2 cn=module

如果在配置slapd时启用对动态加载模块的支持，则可以使用`cn=module`条目来指定要加载的模块集。模块条目必须具有`olcModuleList objectClass`。

##### 5.2.2.1 olcModuleLoad：&lt;filename&gt;

指定要加载的动态可加载模块的名称。文件名可以是绝对路径名或简单文件名。在`olcModulePath`指令指定的目录中搜索非绝对名称。

##### 5.2.2.2 olcModulePath: &lt;pathspec&gt;

指定要加载的动态可加载模块的名称。文件名可以是绝对路径名或简单文件名。在`olcModulePath`指令指定的目录中搜索非绝对名称。

##### 5.2.2.3 条目样例

```
dn: cn=module{0},cn=config
objectClass: olcModuleList
cn: module{0}
olcModuleLoad: /usr/local/lib/smbk5pwd.la

dn: cn=module{1},cn=config
objectClass: olcModuleList
cn: module{1}
olcModulePath: /usr/local/lib:/usr/local/lib/slapd
olcModuleLoad: accesslog.la
olcModuleLoad: pcache.la
```

#### 5.2.3 cn=schema

`cn=schema` 条目包含在slapd中硬编码的所有模式定义。因此，此条目中的值由slapd生成，因此在配置文件中不需要提供模式值。仍然必须定义该条目，以作为用户定义的模式添加到底部的基础。模式条目必须具有`olcSchemaConfig objectClass`。

##### 5.2.3.1 olcAttributeTypes: <[RFC4512](http://www.rfc-editor.org/rfc/rfc4512.txt) Attribute Type Description>

此指令定义属性类型。有关如何使用此指令的信息，请参阅“ [模式规范”](http://www.openldap.org/doc/admin24/schema.html)一章。

##### 5.2.3.2 olcObjectClasses: <[RFC4512](http://www.rfc-editor.org/rfc/rfc4512.txt) Object Class Description>

该指令定义了一个对象类。有关如何使用此指令的信息，请参阅“ [模式规范”](http://www.openldap.org/doc/admin24/schema.html)一章。

##### 5.2.3.3 条目样例

```
dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema

dn: cn=test,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: test
olcAttributeTypes: ( 1.1.1
  NAME 'testAttr'
  EQUALITY integerMatch
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 )
olcAttributeTypes: ( 1.1.2 NAME 'testTwo' EQUALITY caseIgnoreMatch
  SUBSTR caseIgnoreSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.44 )
olcObjectClasses: ( 1.1.3 NAME 'testObject'
  MAY ( testAttr $ testTwo ) AUXILIARY )
```

#### 5.2.4 后端特定指令

后端指令适用于所有相同类型的数据库实例，根据该指令，数据库指令可能会被覆盖。后端条目必须具有`olcBackendConfig objectClass`。

##### 5.2.4.1 olcBackend: &lt;type&gt;

该指令命名后端特定的配置条目。`<type>`应于表5.2中列出的支持的后端类型之一。

**表5.2：数据库后端**

| **类型**  | **描述**               |
| ------- | -------------------- |
| bdb     | Berkeley DB事务后端（已弃用） |
| config  | Slapd配置后端            |
| dnssrv  | DNS SRV后端            |
| hdb     | bdb后端的分层变体（已弃用）      |
| ldap    | 轻量级目录访问协议（Proxy）后端   |
| ldif    | 轻量级数据交换格式后端          |
| mdb     | 内存映射DB后端             |
| meta    | 元目录后端                |
| monitor | 监视后端                 |
| passwd  | 提供对*passwd的*只读访问（5）  |
| perl    | Perl可编程后端            |
| shell   | Shell（外部程序）后端        |
| sql     | SQL可编程后端             |

**例：**

```
olcBackend：bdb
```

没有为此条目定义其他指令。特定的后端类型可以为其特定用途定义附加属性，但是迄今为止还没有定义。因此，这些指令通常不会出现在任何实际配置中。

##### 5.2.4.2 条目样例

```
dn：olcBackend=bdb，cn=config
objectClass:olcBackendConfig
olcBackend:bdb
```

#### 5.2.5 数据库特定的指令

每个类型的数据库都支持本节中的指令。数据库条目必须具有`olcDatabaseConfig objectClass`。

##### 5.2.5.1 olcDatabase: \[\{&lt;index&gt;}]&lt;type&gt;

该指令命名一个特定的数据库实例。可以提供数字`{<index>}`来区分相同类型的多个数据库。通常可以省略索引，slapd将自动生成。`<type>`应于表5.2或列出的支持的后端类型之一`前端`类型。

该`前端`是用来保存应适用于所有其他数据库的数据库级选项一个特殊的数据库。后续数据库定义也可以覆盖一些前端设置。

在配置数据库也是特殊的; 无论是`配置`和`前端`数据库总是隐式创建的，即使他们没有明确配置，以及它们的任何其他数据库之前创建。

例：

```
olcDatabase: bdb
```

这标志着一个新的开始 BDB 数据库实例。

##### 5.2.5.2 olcAccess: to \<what\> [ by &lt;who&gt; [&lt;accesslevel&gt;\] [&lt;control&gt;] ]+

该指令通过一个或多个请求者（由\<who>指定）向一组条目和/或属性（由\<what>指定）访问（由\<accesslevel>指定）。有关基本用途，请参阅本指南的[访问控制](http://www.openldap.org/doc/admin24/access-control.html)部分。

**注意：**如果没有指定`olcAccess`指令，默认访问控制策略（`* by * read`）允许所有用户（通过身份验证和匿名）读取访问权限。

**注意：**在前端定义的访问控件附加到所有其他数据库的控件。

##### 5.2.5.3 olcReadonly { TRUE | FALSE }

该指令将数据库置于“只读”模式。任何修改数据库的尝试将返回“不愿意执行”的错误。如果设置在消费者身上，syncrepl发送的修改仍然会发生。

默认：

```
olcReadonly：FALSE
```

##### 5.2.5.4 olcRootDN: &lt;DN&gt;

此指令指定不受该数据库操作的访问控制或管理限制限制的DN。DN不需要引用此数据库中的条目，甚至不在目录中。DN可以指SASL身份。

基于入门的示例：

```
olcRootDN: "cn=Manager,dc=example,dc=com"
```

基于SASL的示例：

```
olcRootDN: "uid=root,cn=example.com,cn=digest-md5,cn=auth"
```

有关[SASL认证身份](http://www.openldap.org/doc/admin24/sasl.html#SASL Authentication)的信息，请参阅[SASL认证](http://www.openldap.org/doc/admin24/sasl.html#SASL Authentication)部分。

##### 5.2.5.5 olcRootPW: &lt;password&gt;

该指令可用于为rootdn指定DN的密码（当rootdn设置为数据库中的DN时）。

例：

```
olcRootPW: secret
```

还允许在[RFC2307](http://www.rfc-editor.org/rfc/rfc2307.txt)表单中提供密码的散列。 *slappasswd*（8）可用于生成密码哈希。

例：

```
olcRootPW: {SSHA}ZKKuqbEKJfKSXhUbHG3fG8MDn9j1v4QN
```

散列是使用命令`slappasswd -s secret`生成的。

##### 5.2.5.6 olcSizeLimit: &lt;integer&gt;

此指令指定从搜索操作返回的最大条目数。

默认：

```
olcSizeLimit: 500
```

关详细信息，请参阅本指南的[“限制”](http://www.openldap.org/doc/admin24/limits.html)部分和slapd-config（5）。

##### 5.2.5.7 olcSuffix: &lt;dn suffix&gt;

此指令指定将传递给此后端数据库的查询的DN后缀。可以给出多个后缀行，并且每个数据库定义通常需要至少一个。（一些后端类型，如`前端`和`监视器`使用硬编码后缀，可能不会在配置中被覆盖。）

例：

```
olcSuffix："dc=example,dc=com"
```

以`dc=example,dc=com`结尾的DN的查询将传递给此后端。

**注意：**

当选择要传递查询的后端时，slapd按照配置顺序查看每个数据库定义中的后缀值。因此，如果一个数据库后缀是另一个数据库后缀，则必须在配置中显示。

##### 5.2.5.8. olcSyncrepl

```
olcSyncrepl: rid=<replica ID>
                provider=ldap[s]://<hostname>[:port]
                [type=refreshOnly|refreshAndPersist]
                [interval=dd:hh:mm:ss]
                [retry=[<retry interval> <# of retries>]+]
                searchbase=<base DN>
                [filter=<filter str>]
                [scope=sub|one|base]
                [attrs=<attr list>]
                [attrsonly]
                [sizelimit=<limit>]
                [timelimit=<limit>]
                [schemachecking=on|off]
                [bindmethod=simple|sasl]
                [binddn=<DN>]
                [saslmech=<mech>]
                [authcid=<identity>]
                [authzid=<identity>]
                [credentials=<passwd>]
                [realm=<realm>]
                [secprops=<properties>]
                [starttls=yes|critical]
                [tls_cert=<file>]
                [tls_key=<file>]
                [tls_cacert=<file>]
                [tls_cacertdir=<path>]
                [tls_reqcert=never|allow|try|demand]
                [tls_cipher_suite=<ciphers>]
                [tls_crlcheck=none|peer|all]
                [logbase=<base DN>]
                [logfilter=<filter str>]
                [syncdata=default|accesslog|changelog]
```

该指令通过将当前的*slapd*（8）建立为运行syncrepl复制引擎的复制用户站点来将当前数据库指定为主内容的副本。主数据库位于由`provider`参数指定的复制提供程序站点。使用LDAP内容同步协议，副本数据库与主内容保持最新。有关[协议](http://www.rfc-editor.org/rfc/rfc4533.txt)的更多信息，请参阅[RFC4533](http://www.rfc-editor.org/rfc/rfc4533.txt)。

`RID`参数用于当前的识别`的syncrepl`复制消费者服务器内的指令，其中`<replica ID>`唯一地识别由当前所描述的的syncrepl规范`的syncrepl`指令。`<replica ID>`为非负数，长度不超过三位十进制数字。

`provider `参数指定包含主内容作为LDAP URI复制提供者站点。`provider `参数指定的方案，主机和可选地其中该提供者的slapd实例可以发现一个端口。域名或IP地址可以用于\<hostname>。示例是`ldap://provider.example.com：389`或`ldaps://192.168.1.1：636`。如果未指定\<port>，则使用标准LDAP端口号（389或636）。请注意，syncrepl使用消费者启动的协议，因此其规范位于消费者站点，而`replica `规范位于提供者站点。`syncrepl`和`replica `指令定义了两个独立的复制机制。它们不代表彼此的复制对等体。

使用搜索规范作为其结果集定义syncrepl副本的内容。消费者slapd将根据搜索规范将搜索请求发送给提供商slapd。搜索规范包括`searchbase`，`scope`，`filter`，`attrs `，`attrsonly`，`sizeLimit`和`timelimit `参数如在正常搜索规范。该`searchbase`参数没有默认值，必须指定。的`scope `默认为`sub`中，`filter `默认为`（objectclass=*） `，`attrs `默认为`“*，+”`来复制所有用户和操作属性，默认情况下，`attrsonly`未设置。双方的`sizeLimit`和`timelimit`默认为“无限制”，只有正整数或“无限制”可能被指定。

该 LDAP内容同步协议有两种操作类型：`refreshOnly`和`refreshAndPersist`。操作类型由`type`参数指定。在`refreshOnly`操作中，在每次同步操作结束之后的间隔时间周期性地重新调度下一个同步搜索操作。间隔由`interval`参数指定。默认设置为一天。在`refreshAndPersist`操作中，同步搜索在提供者*slapd*实例中保持不变。对主副本的进一步更新将生成`searchResultEntry` 到消费者slapd作为搜索响应的持续同步搜索。

如果在复制期间发生错误，消费者将尝试根据重试参数重新连接，该重试参数是\<retry interval>和<＃of retries> 的列表。例如，retry =“60 10 300 3”可让消费者重新开始前10次，每60秒重试一次，然后在停止重试之前每300秒重试三次。+在<＃重试次数>中意味着无限期的重试，直到成功。

可以通过打开`schemachecking`参数，在LDAP Sync使用者站点上强制执行模式检查。如果它被打开，则每个复制的条目将被检查其模式，因为条目存储在副本内容中。副本中的每个条目都应包含模式定义所需的属性。如果它被关闭，条目将被存储，而不检查模式一致性。默认是关闭。

该`binddn`参数给出了DN绑定为进行syncrepl搜索到提供者的slapd。它应该是具有对主数据库中的复制内容的读取访问权限的DN。

该`bindmethod`是`simple `还是`SASL`，取决于是否简单基于密码的认证或SASL连接到提供商*slapd*实例时要使用身份验证。

除非有足够的数据完整性和机密性保护（例如TLS或IPsec），否则不应使用简单认证。简单的身份验证需要指定`binddn`和`credentials`参数。

通常建议使用SASL认证。SASL身份验证需要使用`saslmech`参数来指定机制。根据机制，可以分别使用`authcid`和`credentials`来指定身份验证身份和/或凭据。所述`authzid`参数可以被用来指定一个授权身份。

`realm`参数指定其中的某些机制认证中的身份的境界。该`secprops`参数指定Cyrus  SASL安全属性。

该`STARTTLS`参数指定使用扩展操作认证的供应商之前建立TLS会话启动TLS的。如果提供了`critical `参数，则如果StartTLS请求失败，则该会话将中止。否则syncrepl会话继续而不使用TLS。tls_reqcert设置默认为`“demand”`，其他TLS设置默认与主slapd TLS设置相同。

消费者不是复制整个条目，而是可以查询数据修改的日志。这种操作模式称为*delta syncrepl*。除了上述参数之外，必须对要使用的日志适当地设置`logbase`和`logfilter`参数。所述`syncdata`参数必须设置为`“accesslog”`如果日志符合*slapo-ACCESSLOG*（5）日志格式，或`“更改日志”`如果日志符合过时的*更改日志*的格式。如果`syncdata`参数被省略或设置为`“default”，`则日志参数将被忽略。

*syncrepl* 复制机制是由支持*BDB*，*hdb*，以及 *mdb*后端。

有关如何使用此指令的详细信息，请参阅本指南的“ [LDAP同步复制”](http://www.openldap.org/doc/admin24/replication.html#LDAP Sync Replication)一章。

##### 5.2.5.9 olcTimeLimit: &lt;integer&gt;

此指令指定slapd将花费回答搜索请求的最大秒数（实时）。如果此时尚未完成请求，将返回指示超时时间的结果。

默认：

```
olcTimeLimit: 3600
```

有关详细信息，请参阅本指南的[“限制”](http://www.openldap.org/doc/admin24/limits.html)部分和slapd-config（5）。

##### 5.2.5.10 olcUpdateref: &lt;URL&gt;

该指令仅适用于从属slapd。它指定返回到在副本上提交更新请求的客户端的URL。如果指定多次，每个网址 被提供。

例：

```
olcUpdateref: ldap://master.example.net
```

##### 5.2.5.11 条目案例

```
dn: olcDatabase=frontend,cn=config
objectClass: olcDatabaseConfig
objectClass: olcFrontendConfig
olcDatabase: frontend
olcReadOnly: FALSE

dn: olcDatabase=config,cn=config
objectClass: olcDatabaseConfig
olcDatabase: config
olcRootDN: cn=Manager,dc=example,dc=com
```

#### 5.2.6 BDB and HDB Database Directives

此类别中的指令适用于 BDB 和 HDB 两个数据库。除了上面定义的通用数据库指令之外，它们还用于olcDatabase条目。有关BDB / HDB配置指令的完整参考，请参见*slapd-bdb*（5）。除了`olcDatabaseConfig objectClass`之外，BDB和HDB数据库条目必须分别具有`olcBdbConfig`和`olcHdbConfig objectClass`。

##### 5.2.6.1. olcDbDirectory: &lt;directory&gt;

此伪指令指定包含数据库和关联索引的BDB文件的目录。

默认：

```
olcDbDirectory: /usr/local/var/openldap-data
```

##### 5.2.6.2 olcDbCachesize：&lt;integer&gt;

此指令指定由BDB后端数据库实例维护的内存中缓存条目的大小。

默认：

```
olcDbCachesize: 1000
```

##### 5.2.6.3 olcDbCheckpoint:  &lt;kbyte&gt; &lt;min&gt;

此指令指定检查点BDB事务日志的频率。检查点操作将数据库缓冲区刷新到磁盘，并在日志中写入检查点记录。如果已经写入了`<kbyte>`数据或自上次检查点起已经过了`<min>`分钟，则检查点将发生。两个参数默认为零，在这种情况下，它们将被忽略。当`<min>`参数不为零时，内部任务将每分钟运行一分钟以执行检查点。有关详细信息，请参阅Berkeley DB参考指南。

例：

```
olcDbCheckpoint: 1024 10
```

##### 5.2.6.4 olcDbConfig: &lt;DB_CONFIG setting&gt;

此属性指定要放置在数据库目录的`DB_CONFIG`文件中的配置指令。在服务器启动时，如果尚未存在此类文件，则将创建`DB_CONFIG`文件，并将此属性中的设置写入。如果文件存在，其内容将被读取并显示在此属性中。该属性是多值的，以适应多个配置指令。没有提供默认值，但是在这里使用适当的设置来获得最佳的服务器性能是至关重要的。

对此属性所做的任何更改都将写入`DB_CONFIG`文件，并将导致数据库环境重置，以便更改可立即生效。如果环境缓存大并且最近没有被检查点，这种重置操作可能需要很长时间。在使用LDAP修改更改此属性之前，建议您使用Berkeley DB *db_checkpoint*实用程序手动执行单个检查点。

例：

```
olcDbConfig: set_cachesize 0 10485760 0
olcDbConfig: set_lg_bsize 2097512
olcDbConfig: set_lg_dir /var/tmp/bdb-log
olcDbConfig: set_flags DB_LOG_AUTOREMOVE
```

在本示例中，BDB缓存设置为10MB，BDB事务日志缓冲区大小设置为2MB，并将事务日志文件存储在`/var/tmp/bdb-log`目录中。另外还设置一个标志来告知BDB一旦它们的内容被检查点并且不再需要就删除事务日志文件。没有此设置，事务日志文件将继续累积，直到其他一些清除过程删除它们。有关详细信息，请参见`db_archive`命令的Berkeley DB文档。有关Berkeley DB标志的完整列表，请参阅 - <http://www.oracle.com/technology/documentation/berkeley-db/db/api_c/env_set_flags.html>

理想情况下，BDB缓存必须至少与数据库的工作集一样大，日志缓冲区大小应足够大以适应大多数事务而不会溢出，并且日志目录必须与主数据库文件在单独的物理磁盘上。数据库目录和日志目录应与用于常规系统活动（例如根，引导或交换文件系统）的磁盘分开。有关更多详细信息，请参阅FAQ-o-Matic和Berkeley DB文档。

##### 5.2.6.5 olcDbNosync: { TRUE | FALSE }

此选项会导致磁盘数据库内容在更改时不会立即与内存更改同步。将此选项设置为`TRUE`可能会牺牲数据完整性来提高性能。该指令与使用具有相同的效果

```
olcDbConfig: set_flags DB_TXN_NOSYNC
```

##### 5.2.6.6 olcDbIDLcacheSize：&lt;integer>

在索引插槽中指定内存中索引缓存的大小。默认值为零。较大的值将加快索引条目的频繁搜索。最佳大小将取决于数据库的数据和搜索特性，但使用三倍的条目缓存大小是一个很好的起点。

例：

```
olcDbIDLcacheSize: 3000
```

##### 5.2.6.7 olcDbIndex: {&lt;attrlist&gt; | default} [pres,eq,approx,sub,none\]

该指令指定为给定属性维护的索引。如果只给出一个`<attrlist>`，则会保留默认索引。索引关键字对应于可在LDAP搜索过滤器中使用的常见匹配类型。

例：

```
olcDbIndex: default pres,eq
olcDbIndex: uid
olcDbIndex: cn,sn pres,eq,sub
olcDbIndex: objectClass eq
```

第一行设置默认的索引集，以保持呈现和相等。第二行导致为`uid`属性类型维护默认（pres，eq）索引集。第三行导致为`cn`和`sn`属性类型维护当前，等于和子字符串索引。第四行导致`objectClass`属性类型的相等索引。

不等式匹配没有索引关键字。通常这些匹配不使用索引。然而，一些属性确实支持基于等式索引的不等式匹配的索引。

子串的索引可以更明确地指定为`subinitial`，`subany`，或`subfinal`，对应于字符串匹配滤波器的三种可能的组分。一个小数中间索引仅索引出现在属性值开头的子字符串。子索引仅索引出现在属性值末尾的子字符串，而子索引则会在值中任何位置发生子字符串索引。

请注意，默认情况下，为属性设置索引也会影响该属性的每个子类型。例如，在`name`属性上设置一个相等的索引会导致`cn`，`sn`和从`name`继承的每个其他属性被索引。

缺省情况下，没有维护索引。通常建议保持对象类的最小等同索引。

```
olcDbindex: objectClass eq
```

应根据在数据库中使用的最常见的搜索来配置其他索引。除非属性在数据库中很少出现，否则不应为属性配置存在索引，并且在正常使用目录期间，属性上的出现搜索会非常频繁地出现。大多数应用程序不使用存在搜索，因此通常存在索引不是很有用。

如果slapd运行时此设置发生更改，则将运行内部任务以生成已更改的索引数据。所有的服务器操作都能像索引器一样正常工作。如果在索引任务完成之前slapd已停止，则索引将必须使用slapindex工具手动完成。

##### 5.2.6.8 olcDbLinearIndex: { TRUE | FALSE }

如果此设置为`TRUE`， slapindex将一次为一个属性索引。默认设置为`FALSE`，在这种情况下，条目的所有索引属性都将同时处理。启用时，每个索引属性都被单独处理，使用遍历整个数据库的多次遍历。当数据库大小超过BDB缓存大小时，此选项可提高slapindex性能。当BDB缓存足够大时，不需要此选项并降低性能。同样默认情况下，slapadd执行完整的索引，因此不需要单独的slapindex运行。使用此选项，slapadd不进行索引编译，并且必须使用slapindex。

##### 5.2.6.9 olcDbMode: { &lt;octal> | \<symbolic> }

此指令指定新创建的数据库索引文件应具有的文件保护模式。这可以是`0600`或`-rw -------`

默认：

```
olcDbMode：0600
```

##### 5.2.6.10 olcDbSearchStack: &lt;integer&gt;

指定用于搜索过滤器评估的堆栈的深度。搜索过滤器在堆栈中进行评估，以适应嵌套的`AND` / `OR`条款。为每个服务器线程分配单个堆栈。堆栈的深度决定了如何复杂的过滤器可以被评估，而不需要额外的内存分配。比搜索堆栈深度嵌套的过滤器将导致为该特定搜索操作分配单独的堆栈。这些单独的分配可能对服务器性能造成严重的负面影响，但是指定太多堆栈也将消耗大量内存。每个搜索在32位计算机上每级使用512K字节，或64位计算机上每级的1024K字节。默认堆栈深度为16，因此在32位和64位机器上分别使用每个线程8MB或16MB。同样512KB大小的单个堆栈槽由编译时常数设置，如果需要可以更改;

默认：

```
olcDbSearchStack: 16
```

##### 5.2.6.11 olcDbShmKey: &lt;integer&gt;

指定共享内存BDB环境的密钥。默认情况下，BDB环境使用内存映射文件。如果指定了非零值，它将被用作标识将容纳环境的共享内存区域的键。

例：

```
olcDbShmKey: 42
```

##### 5.2.6.12 输出样例

```
dn: olcDatabase=hdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcHdbConfig
olcDatabase: hdb
olcSuffix: "dc=example,dc=com"
olcDbDirectory: /usr/local/var/openldap-data
olcDbCacheSize: 1000
olcDbCheckpoint: 1024 10
olcDbConfig: set_cachesize 0 10485760 0
olcDbConfig: set_lg_bsize 2097152
olcDbConfig: set_lg_dir /var/tmp/bdb-log
olcDbConfig: set_flags DB_LOG_AUTOREMOVE
olcDbIDLcacheSize: 3000
olcDbIndex: objectClass eq
```

### 5.3 配置举例

以下是一个示例配置，散布说明文字。它定义了两个数据库来处理不同的部分X.500树; 两者都是BDB数据库实例。显示的行号仅供参考，不包括在实际文件中。首先，全局配置部分：

```
1.    # example config file - global configuration entry
2.    dn: cn=config
3.    objectClass: olcGlobal
4.    cn: config
5.    olcReferral: ldap://root.openldap.org
6.
```

第1行是一条评论。第2-4行将其标识为全局配置条目。第5行上的`olcReferral：`指令意味着不属于下面定义的数据库之一的查询将被引用到在主机`root.openldap.org`上的标准端口（389）上运行的LDAP服务器。第6行是空白行，表示此条目的结尾。

```
 7.    # internal schema
 8.    dn: cn=schema,cn=config
 9.    objectClass: olcSchemaConfig
10.    cn: schema
11.
```

第7行是一条评论。第8-10行将此标识为模式子树的根。此条目中的实际模式定义将被硬编码为slapd，因此在此处未指定其他属性。第11行是空白行，表示此条目的结尾。

```
12.    # include the core schema
13.    include: file:///usr/local/etc/openldap/schema/core.ldif
14.
```

第12行是一条评论。第13行是以LDIF格式访问*核心*模式定义的LDIF include指令。第14行是空行。

接下来是数据库定义。第一个数据库是其全局应用于所有其他数据库的特殊`前端`数据库。

```
15.    # global database parameters
16.    dn: olcDatabase=frontend,cn=config
17.    objectClass: olcDatabaseConfig
18.    olcDatabase: frontend
19.    olcAccess: to * by * read
20.
```

第15行是一条评论。第16-18行将此条目标识为全局数据库条目。第19行是一个全局访问控制。它适用于所有条目（在任何适用的特定于数据库的访问控制之后）。第20行是空行。

下一个条目定义了配置后端。

```
21.    # set a rootpw for the config database so we can bind.
22.    # deny access to everyone else.
23.    dn: olcDatabase=config,cn=config
24.    objectClass: olcDatabaseConfig
25.    olcDatabase: config
26.    olcRootPW: {SSHA}XKYnrjvGT3wZFQrDD5040US592LxsdLy
27.    olcAccess: to * by * none
28.
```

第21-22行是评论。第23-25行将此条目标识为配置数据库条目。第26行定义了该数据库的*超级用户*密码。（DN默认为*“cn = config”*。）第27行拒绝对该数据库的所有访问，因此只有超级用户才能访问该数据库。（这已经是配置数据库中的默认访问权限，这里只是列出了说明，并且重申，除非明确配置超级用户身份验证的方式，否则配置数据库将无法访问。）

第28行是空行。

下一个条目定义了一个BDB后端，它将处理对树的“dc = example，dc = com”部分中的事物的查询。指数将被维护为几个属性，并且`userPassword`属性被保护免受未经授权的访问。

```
29.    # BDB definition for example.com
30.    dn: olcDatabase=bdb,cn=config
31.    objectClass: olcDatabaseConfig
32.    objectClass: olcBdbConfig
33.    olcDatabase: bdb
34.    olcSuffix: dc=example,dc=com
35.    olcDbDirectory: /usr/local/var/openldap-data
36.    olcRootDN: cn=Manager,dc=example,dc=com
37.    olcRootPW: secret
38.    olcDbIndex: uid pres,eq
39.    olcDbIndex: cn,sn pres,eq,approx,sub
40.    olcDbIndex: objectClass eq
41.    olcAccess: to attrs=userPassword
42.      by self write
43.      by anonymous auth
44.      by dn.base="cn=Admin,dc=example,dc=com" write
45.      by * none
46.    olcAccess: to *
47.      by self write
48.      by dn.base="cn=Admin,dc=example,dc=com" write
49.      by * read
```

第29行是一条评论。30-33行将此条目标识为BDB数据库配置条目。第34行指定查询传递给此数据库的DN后缀。第35行指定数据库文件将在其中生存的目录。

36和37行标识数据库*超级用户*条目和相关密码。此条目不受访问控制或大小或时间限制的限制。

第38至40行表示维持各种属性的索引。

第41至49行为此数据库中的条目指定访问控制。对于所有适用的条目，`userPassword`属性可由条目本身和“admin”条目写入。它可以用于认证/授权目的，但是不可读。所有其他属性可由条目和“admin”条目写入，但可由所有用户（经过身份验证或不被认证）读取。

第50行是空白行，表示此条目的结尾。

下一个条目定义另一个BDB数据库。这个处理涉及`dc=example,dc=net` subtree的查询，但是由与第一个数据库相同的实体进行管理。请注意，没有第60行，由于第19行的全局访问规则，读访问将被允许。

```
51.    # BDB definition for example.net
52.    dn: olcDatabase=bdb,cn=config
53.    objectClass: olcDatabaseConfig
54.    objectClass: olcBdbConfig
55.    olcDatabase: bdb
56.    olcSuffix: "dc=example,dc=net"
57.    olcDbDirectory: /usr/local/var/openldap-data-net
58.    olcRootDN: "cn=Manager,dc=example,dc=com"
59.    olcDbIndex: objectClass eq
60.    olcAccess: to * by users read
```

### 5.4 将旧样式*slapd.conf*（5）文件转换为*cn=config*格式

在转换为*cn=config*格式之前，您应该确保配置后端在您现有的配置文件中正确配置。虽然配置后端始终存在于slapd内，但默认情况下只能由其rootDN访问，并且没有分配默认凭据，除非您明确配置一种认证方式，否则将无法使用。

如果您还没有`database config`部分，请在`slapd.conf`的末尾添加如下内容

```
database config
rootpw VerySecret
```

**注意：**由于配置后端可用于将任意代码加载到slapd进程中，因此仔细保护用于访问它的任何凭据是非常重要的。由于简单的密码容易受到密码猜测攻击，通常最好省略rootpw，只能使用SASL身份验证配置rootDN。

现有的*slapd.conf*（5）文件可以使用*slaptest*（8）或任何照片工具转换为新格式：

```
slaptest -f /usr/local/etc/openldap/slapd.conf -F /usr/local/etc/openldap/slapd.d
```

测试您可以使用默认*rootdn*和*上面*配置的*rootpw*访问`cn=config`下的条目：

```
ldapsearch -x -D cn=config -w VerySecret -b cn=config
```

然后可以丢弃旧的*slapd.conf*（5）文件。如果不使用默认目录路径，请确保使用*-F*选项启动*slapd*（8）以指定配置目录。

**注意：**从slapd.conf格式转换为slapd.d格式时，任何包含的文件也将集成到生成的配置数据库中。

## 6 slapd配置文件

本章介绍通过*slapd.conf*（5）配置文件配置*slapd*（8）。 *slapd.conf*（5）已被弃用，只有当您的站点需要尚未更新的后台之一才能使用较新的*slapd-config*（5）系统时，才能使用它。在上一章中介绍了通过*slapd-config*（5）配置*slapd*（8）。

*slapd.conf*（5）文件通常安装在`/usr/local/etc/openldap` 目录。可以通过命令行选项将备用配置文件位置指定为*slapd*（8）。

### 6.1 配置文件格式

在*slapd.conf*（5）文件由三种类型的配置信息：全局的，特定后台和特定数据库的。首先指定全局信息，然后指定与特定后端类型相关联的信息，然后跟随特定数据库实例关联的信息。可以在后端和 / 或数据库指令中覆盖全局伪指令，并且数据库伪指令可以覆盖后端伪指令。

以“ `＃` ”字符开头的空行和注释行将被忽略。如果一行以空格开头，则被认为是上一行的延续（即使上一行是注释）。

slapd.conf的一般格式如下：

```
# global configuration directives
<global config directives>

# backend definition
backend <typeA>
<backend-specific directives>

# first database definition & config directives
database <typeA>
<database-specific directives>

# second database definition & config directives
database <typeB>
<database-specific directives>

# second database definition & config directives
database <typeA>
<database-specific directives>

# subsequent backend & database definitions & config directives
...
```

配置指令可能会引用参数。如果是这样，他们被空白分隔开来。如果一个参数包含空格，参数应该用双引号括起来`“像这样”`。如果参数包含双引号或反斜杠字符“ `\` ”，则该字符前面应该有一个反斜杠字符“ `\` ”。

该分发包含将安装在`/usr/local/etc/openldap`目录中的示例配置文件。`/usr/local/etc/openldap/schema`目录中也提供了一些包含模式定义（属性类型和对象类）的文件。

### 6.2 配置文件指令

本节详细介绍常用的配置指令。有关完整列表，请参阅*slapd.conf*（5）手册页。本节将配置文件指令分为全局，后端特定和特定于数据的类别，描述每个指令及其默认值（如果有），并给出其使用示例。

#### 6.2.1 全局指令

本节中描述的指令适用于所有后端和数据库，除非在后端或数据库定义中特别覆盖。应该用实际文本替换的参数显示在括号`<>`中。

##### 6.2.1.1 access to  [ by &lt;who&gt; [&lt;accesslevel&gt;\] [&lt;control&gt;] ]+

该指令通过一个或多个请求者（由&lt;who&gt;指定）向一组条目和/或属性（由&lt;what&gt;指定）访问（由&lt;accesslevel&gt;指定）。有关基本用途，请参阅本指南的[访问控制](http://www.openldap.org/doc/admin24/access-control.html)部分。

**注意：**如果没有指定`访问`指令，则默认访问控制策略`access to * by * read`，允许所有身份验证和匿名用户都可以读取访问权限。

#### 6.2.1.2 attributetype &lt;[RFC4512](http://www.rfc-editor.org/rfc/rfc4512.txt) Attribute Type Description&gt;

此指令定义属性类型。有关如何使用此指令的信息，请参阅“ [模式规范”](http://www.openldap.org/doc/admin24/schema.html)一章

#### 6.2.1.3 idletimeout &lt;integer&gt;

指定强制关闭空闲客户端连接之前等待的秒数。idletimeout为0，默认值为禁用此功能。

#### 6.2.1.4 include &lt;filename&gt;

该指令指定slapd应该在继续当前文件的下一行之前从给定文件中读取其他配置信息。包含的文件应遵循正常的slapd配置文件格式。该文件通常用于包含包含模式规范的文件。

**注意：**使用这个指令时应该小心 - 嵌套的include指令的数量没有很小的限制，没有进行环路检测。

#### 6.2.1.5. loglevel &lt;level&gt;

此指令指定调试语句和操作统计信息应该被syslog的级别（当前记录到*syslogd*（8）`LOG_LOCAL4`设施）。您必须配置OpenLDAP `--enable-debug`（默认值）才能工作（除了始终启用的两个统计级别之外）。日志级别可以被指定为整数或关键字。可以使用多个日志级别，并且级别是相加的。要显示什么号码对应于什么样的调试，用`-d?`调用slapd 或查阅下表。\<integer>的可能值为：

**表6.1：调试级别**

| **水平** | **描述**         | **关键词**        |
| ------ | -------------- | -------------- |
| -1     | 启用所有调试         | 任何             |
| 0      | 没有调试           |                |
| 1      | 跟踪函数调用         | （0x1跟踪）        |
| 2      | 调试数据包处理        | （0x2数据包）       |
| 4      | 重调查            | （0x4 args）     |
| 8      | 连接管理           | （0x8连接）        |
| 16     | 打印发送和接收的数据包    | （0x10 BER）     |
| 32     | 搜索过滤处理         | （0x20过滤器）      |
| 64     | 配置处理           | （0x40配置）       |
| 128    | 访问控制列表处理       | （0x80 ACL）     |
| 256    | 统计日志连接/操作/结果   | （0x100统计）      |
| 512    | 统计日志条目发送       | （0x200 stats2） |
| 1024   | 打印与后台的通讯       | （0x400 shell）  |
| 2048   | 打印输入解析调试       | （0x800解析）      |
| 16384  | syncrepl消费者处理  | （0x4000同步）     |
| 32768  | 只有设置了记录日志级别的消息 | （0x8000无）      |

期望的日志级别可以作为单个整数输入，该整数将（ORed）所需的级别（十进制或十六进制符号）组合为整数列表（在内部进行ORed操作），或作为显示的名称列表在括号之间，

```
loglevel 129
loglevel 0x81
loglevel 128 1
loglevel 0x80 0x1
loglevel acl跟踪
```

是等同的

例子：

```
loglevel -1
```

这将导致大量的调试信息被记录。

```
loglevel conns filter
```

只需记录连接并搜索过滤器处理。

```
loglevel none
```

记录无论配置的日志级别如何记录的消息。这不同于在不发生日志记录时将日志级别设置为0。至少需要“ `None ”`级才能记录高优先级消息。

默认：

```
loglevel stats
```

默认情况下配置基本统计记录。但是，如果未定义日志级别，则不会发生日志记录（相当于0级别）。

#### 6.2.1.6 objectclass &lt;[RFC4512](http://www.rfc-editor.org/rfc/rfc4512.txt) Object Class Description&gt;

该指令定义了一个对象类。有关如何使用此指令的信息，请参阅“ [模式规范”](http://www.openldap.org/doc/admin24/schema.html)一章。

#### 6.2.1.7 referral &lt;URI&gt;

此指令指定当slapd找不到本地数据库来处理请求时要传回的引用。

例：

```
referral ldap://root.openldap.org
```

这将在OpenLDAP项目中将非本地查询引用到全局根LDAP服务器。智能LDAP客户端可以在该服务器上重新询问他们的查询，但请注意，大多数这些客户端只会知道如何处理包含主机部分和可选的可分辨名称部分的简单LDAP URL。

#### 6.2.1.8 sizelimit &lt;integer&gt;

此指令指定从搜索操作返回的最大条目数。

默认：

```
sizelimit 500
```

有关详细信息，请参阅本指南的[“限制”](http://www.openldap.org/doc/admin24/limits.html)部分和*slapd.conf*（5）。

#### 6.2.1.9 timelimit &lt;integer&gt;

此指令指定slapd将花费回答搜索请求的最大秒数（实时）。如果此时尚未完成请求，将返回指示超时时间的结果。

默认：

```
timelimit 3600
```

有关详细信息，请参阅本指南的[“限制”](http://www.openldap.org/doc/admin24/limits.html)部分和*slapd.conf*（5）。

### 6.2.2 一般后端指令

本节中的指令仅适用于定义它们的后端。它们由每种后端支持。后端指令适用于所有相同类型的数据库实例，根据该指令，数据库指令可能会被覆盖。

#### 6.2.2.1 backend &lt;type&gt;

后端该指令标志着后端声明的开始。`<type>`应于表6.2中列出的支持的后端类型之一。

**表6.2：数据库后端**

| **类型**  | **描述**               |
| ------- | -------------------- |
| bdb     | Berkeley DB事务后端（已弃用） |
| dnssrv  | DNS SRV后端            |
| hdb     | bdb后端的分层变体（已弃用）      |
| ldap    | 轻量级目录访问协议（Proxy）后端   |
| mdb     | 内存映射DB后端             |
| meta    | 元目录后端                |
| monitor | 监视后端                 |
| passwd  | 提供对*passwd的*只读访问（5）  |
| perl    | Perl可编程后端            |
| shell   | Shell（外部程序）后端        |
| sql     | SQL可编程后端             |

例：

```
backend bdb
```

这标志着一个新的开始 BDB 后端定义

### 6.2.3 一般数据库指令

本节中的指令仅适用于定义它们的数据库。它们由每种类型的数据库支持。

#### 6.2.3.1 database &lt;type&gt;

该指令标志着数据库实例声明的开始。`<类型>`应于表6.2中列出的支持的后端类型之一。

例：

```
database bdb
```

#### 6.2.3.2 limits &lt;selector&gt; &lt;limit&gt; [&lt;limit&gt; [...\]]

根据操作的启动器或基本DN指定时间和大小限制。

有关详细信息，请参阅本指南的[“限制”](http://www.openldap.org/doc/admin24/limits.html)部分和*slapd.conf*（5）。

#### 6.2.3.3 readonly { on | off }

该指令将数据库置于“只读”模式。任何修改数据库的尝试将返回“不愿意执行”的错误。如果设置在消费者身上，syncrepl发送的修改仍然会发生。

默认：

```
readonly off
```

#### 6.2.3.4 rootdn &lt;DN&gt;

此指令指定不受该数据库操作的访问控制或管理限制限制的DN。DN不需要引用此数据库中的条目，甚至不在目录中。DN可以指SASL身份。

基于入门的示例：

```
rootdn "cn=Manager,dc=example,dc=com"
```

基于SASL的示例：

```
rootdn "uid=root,cn=example.com,cn=digest-md5,cn=auth"
```

有关[SASL认证身份](http://www.openldap.org/doc/admin24/sasl.html#SASL Authentication)的信息，请参阅[SASL认证](http://www.openldap.org/doc/admin24/sasl.html#SASL Authentication)部分。

#### 6.2.3.5 rootpw &lt;password&gt;

该指令可用于为rootdn指定DN的密码（当rootdn设置为数据库中的DN时）。

例：

```
rootpw secret
```

还可以在[RFC2307](http://www.rfc-editor.org/rfc/rfc2307.txt)表单中提供密码散列。 *slappasswd*（8）可用于生成密码哈希。

例：

```
rootpw {SSHA}ZKKuqbEKJfKSXhUbHG3fG8MDn9j1v4QN
```

散列是使用命令`slappasswd -s secret`生成的。

#### 6.2.3.6 suffix &lt;dn suffix&gt;

此指令指定将传递给此后端数据库的查询的DN后缀。可以给出多个后缀行，每个数据库定义至少需要一个。

例：

```
suffix "dc=example,dc=com"
```

以“`dc = example,dc = com`”结尾的DN的查询将传递给此后端。

**注意：**当选择要传递查询的后端时，slapd按照它们在文件中显示的顺序查看每个数据库定义中的后缀行。因此，如果一个数据库后缀是另一个数据库后缀，则它必须出现在配置文件中。

#### 6.2.3.7 syncrepl

```
syncrepl rid=<replica ID>
                provider=ldap[s]://<hostname>[:port]
                searchbase=<base DN>
                [type=refreshOnly|refreshAndPersist]
                [interval=dd:hh:mm:ss]
                [retry=[<retry interval> <# of retries>]+]
                [filter=<filter str>]
                [scope=sub|one|base]
                [attrs=<attr list>]
                [exattrs=<attr list>]
                [attrsonly]
                [sizelimit=<limit>]
                [timelimit=<limit>]
                [schemachecking=on|off]
                [network-timeout=<seconds>]
                [timeout=<seconds>]
                [bindmethod=simple|sasl]
                [binddn=<DN>]
                [saslmech=<mech>]
                [authcid=<identity>]
                [authzid=<identity>]
                [credentials=<passwd>]
                [realm=<realm>]
                [secprops=<properties>]
                [keepalive=<idle>:<probes>:<interval>]
                [starttls=yes|critical]
                [tls_cert=<file>]
                [tls_key=<file>]
                [tls_cacert=<file>]
                [tls_cacertdir=<path>]
                [tls_reqcert=never|allow|try|demand]
                [tls_cipher_suite=<ciphers>]
                [tls_crlcheck=none|peer|all]
                [tls_protocol_min=<major>[.<minor>]]
                [suffixmassage=<real DN>]
                [logbase=<base DN>]
                [logfilter=<filter str>]
                [syncdata=default|accesslog|changelog]
```

该指令通过将当前的*slapd*（8）建立为运行syncrepl复制引擎的复制用户站点来将当前数据库指定为主内容的副本。主数据库位于由`provider`参数指定的复制提供程序站点。使用LDAP内容同步协议，副本数据库与主内容保持最新。有关[协议](http://www.rfc-editor.org/rfc/rfc4533.txt)的更多信息，请参阅[RFC4533](http://www.rfc-editor.org/rfc/rfc4533.txt)。

`RID`参数用于当前的识别`的syncrepl`复制消费者服务器内的指令，其中`<replica ID>`唯一地识别由当前所描述的的syncrepl规范`syncrepl`指令。`<replica ID>`为非负数，长度不超过三位十进制数字。

`provider`参数指定包含主内容作为LDAP URI复制提供者站点。`provider`参数指定的方案，主机和可选地其中该提供者的slapd实例可以发现一个端口。域名或IP地址可以用于\<hostname>。示例是`ldap://provider.example.com:389`或`ldaps://192.168.1.1:636`。如果未指定\<port>，则使用标准LDAP端口号（389或636）。请注意，syncrepl使用消费者启动的协议，因此其规范位于消费者站点，而`副本`规范位于提供者站点。`syncrepl`和`副本`指令定义了两个独立的复制机制。它们不代表彼此的复制对等体。

使用搜索规范作为其结果集定义syncrepl副本的内容。消费者slapd将根据搜索规范将搜索请求发送给提供商slapd。搜索规范包括`searchbase`，`scope`，`filter`，`attrs`，`exattrs`，`attrsonly`，`sizeLimit`和`timelimit `参数如在正常搜索规范。该`searchbase`参数没有默认值，必须指定。scope 默认为`sub`中，`filter `默认为`（objectclass= *） `，`attrs`默认为`“*，+”`来复制所有用户和操作属性，默认情况下`attrsonly`是未设置的。双方`的sizeLimit`和`时限`默认为“无限制”，只有正整数或“无限制”可能被指定。该`exattrs`选项也可被用来指定应该从输入的条目被省略属性。

该 LDAP内容同步协议有两种操作类型：`refreshOnly`和`refreshAndPersist`。操作类型由`type`参数指定。在`refreshOnly`操作中，在每次同步操作结束之后的间隔时间周期性地重新调度下一个同步搜索操作。间隔由`interval`参数指定。默认设置为一天。在`refreshAndPersist`操作中，同步搜索在提供者*slapd*实例中保持不变。对主副本的进一步更新将生成`searchResultEntry` 到消费者slapd作为搜索响应的持续同步搜索。

如果在复制期间发生错误，消费者将尝试根据重试参数重新连接，该重试参数是\<retry interval>和<＃of retries> pair的列表。例如，retry =“60 10 300 3”可让消费者重新开始前10次，每60秒重试一次，然后在停止重试之前每300秒重试三次。+在<＃重试次数>中意味着无限期的重试，直到成功。

可以通过打开`schemachecking`参数，在LDAP Sync使用者站点上强制执行模式检查。如果它被打开，则每个复制的条目将被检查其模式，因为条目存储在副本内容中。副本中的每个条目都应包含模式定义所需的属性。如果它被关闭，条目将被存储，而不检查模式一致性。默认是关闭。

`network-timeout`参数设置，消费者将等待多长时间来建立网络连接到提供商。一旦建立连接，`timeout`参数确定消费者等待初始绑定请求完成的时间。这些参数的默认值来自*ldap.conf*（5）。

`binddn`参数给出了DN绑定为进行syncrepl搜索到提供者的slapd。它应该是具有对主数据库中的复制内容的读取访问权限的DN。

`bindmethod`是`simple`还是`SASL`，取决于是否简单基于密码的认证或SASL连接到提供商*slapd*实例时要使用身份验证。

除非有足够的数据完整性和机密性保护（例如TLS或IPsec），否则不应使用简单认证。简单的身份验证需要指定`binddn`和`credentials`参数。

通常建议使用SASL认证。SASL身份验证需要使用`saslmech`参数来指定机制。根据机制，可以分别使用`authcid`和`credentials`来指定身份验证身份和/或凭据。所述`authzid`参数可以被用来指定一个授权身份。

`realm`参数指定其中的某些机制认证中的身份的境界。该`secprops`参数指定赛勒斯SASL安全属性。

`keepalive `参数设定空闲，探针和间隔用于检查套接字是否是活的值; 空闲是TCP开始发送keepalive探测之前连接需要保持空闲的秒数; 探测器是TCP连接之前应发送的Keepalive探测器的最大数量; 间隔是个体保持性探测之间的间隔（以秒为单位）。只有一些系统支持定制这些值; 否则将忽略keepalive参数，并使用系统范围的设置。例如，在240秒的空闲活动之后，keepalive =“240：10：30”将发送10次，每30秒钟一次Keepalive。如果没有收到对探测器的响应，则连接将被丢弃。

`starttls `参数指定使用扩展操作认证的供应商之前建立TLS会话启动TLS的。如果提供了`critical `参数，则如果StartTLS请求失败，则该会话将中止。否则syncrepl会话继续而不使用TLS。tls_reqcert设置默认为`“demand”`，其他TLS设置默认与主slapd TLS设置相同。

`suffixmassage`参数允许用户从远程目录中其DN后缀从本地目录不同拉条目。与搜索库匹配的远程条目DN的部分将被后缀名称DN替换。

消费者不是复制整个条目，而是可以查询数据修改的日志。这种操作模式称为*delta syncrepl*。除了上述参数之外，必须对要使用的日志适当地设置`logbase`和`logfilter`参数。所述`syncdata`参数必须设置为`“accesslog”`如果日志符合*slapo-accesslog*（5）日志格式，或`“更改日志”`如果日志符合过时的*更改日志*的格式。如果`syncdata`参数被省略或设置为`“default”` ，则日志参数将被忽略。

*syncrepl* 复制机制是由支持*BDB*，*hdb*，以及*MDB*后端。

有关如何使用此指令的详细信息，请参阅本指南的“ [LDAP同步复制”](http://www.openldap.org/doc/admin24/replication.html#LDAP Sync Replication)一章。

#### 6.2.3.8 updateref &lt;URL&gt;

此伪指令仅适用于*从属*（或*阴影*）*slapd*（8）实例。它指定返回到在副本上提交更新请求的客户端的URL。如果指定多次，每个网址 被提供。

例：

```
updateref       ldap://master.example.net
```

### 6.2.4 BDB和HDB数据库指令

此类别中的指令仅适用于 BDB 和 HDB数据库。也就是说，它们必须遵循“数据库bdb”或“数据库hdb”行，并在任何后续的“后端”或“数据库”行之前。有关BDB / HDB配置指令的完整参考，请参见*slapd-bdb*（5）。

#### 6.2.4.1 directory &lt;directory&gt;

此伪指令指定包含数据库和关联索引的BDB文件的目录。

默认：

```
directory /usr/local/var/openldap-data
```

## 6.3 配置文件示例

以下是一个示例配置文件，其中插有说明文字。它定义了两个数据库来处理不同的部分X.500树; 两者都是BDB数据库实例。显示的行号仅供参考，不包括在实际文件中。首先，全局配置部分：

```
1.    # example config file - global configuration section
2.    include /usr/local/etc/schema/core.schema
3.    referral ldap://root.openldap.org
4.    access to * by * read
```

第1行是一条评论。第2行包含另一个包含*核心*模式定义的配置文件。第3行的`referral `指令意味着不属于下面定义的数据库之一的查询将被引用到在主机`root.openldap.org`上的标准端口（389）上运行的LDAP服务器。

第4行是一个全局访问控制。它适用于所有条目（在任何适用的特定于数据库的访问控制之后）。

配置文件的下一部分定义了一个BDB后端，它将处理对树的“`dc=example,dc=com`”部分中的内容的查询。数据库将被复制到两个从属 slapds，一个在叛逃，另一个在判断日。指数将被维护为几个属性，并且`userPassword`属性被保护免受未经授权的访问。

```
 5.    # BDB definition for the example.com
 6.    database bdb
 7.    suffix "dc=example,dc=com"
 8.    directory /usr/local/var/openldap-data
 9.    rootdn "cn=Manager,dc=example,dc=com"
10.    rootpw secret
11.    # indexed attribute definitions
12.    index uid pres,eq
13.    index cn,sn pres,eq,approx,sub
14.    index objectClass eq
15.    # database access control definitions
16.    access to attrs=userPassword
17.        by self write
18.        by anonymous auth
19.        by dn.base="cn=Admin,dc=example,dc=com" write
20.        by * none
21.    access to *
22.        by self write
23.        by dn.base="cn=Admin,dc=example,dc=com" write
24.        by * read
```

第5行是一条评论。数据库定义的开始由第6行的数据库关键字标记。第7行指定查询传递到此数据库的DN后缀。第8行指定数据库文件将在哪个目录。

第9行和第10行标识数据库*超级用户*条目和相关密码。此条目不受访问控制或大小或时间限制的限制。

第12至14行表示维持各种属性的索引。

第16到24行为此数据库中的条目指定访问控制。对于所有适用的条目，`userPassword`属性可由条目本身和“admin”条目写入。它可以用于认证/授权目的，但是不可读。所有其他属性可由条目和“admin”条目写入，但可由所有用户（经过身份验证或不被认证）读取。

示例配置文件的下一部分定义了另一个BDB数据库。这个处理涉及`dc=example,dc=net` subtree的查询`，`但是由与第一个数据库相同的实体进行管理。请注意，没有第39行，由于第4行的全局访问规则，读访问将被允许。

```
33.    # BDB definition for example.net
34.    database bdb
35.    suffix "dc=example,dc=net"
36.    directory /usr/local/var/openldap-data-net
37.    rootdn "cn=Manager,dc=example,dc=com"
38.    index objectClass eq
39.    access to * by users read
```

## 7 运行slapd

*slapd*（8）被设计为作为独立服务运行。这允许服务器利用缓存，管理底层数据库的并发问题，并节省系统资源。从*inetd*（8）运行*不是*一个选项。

### 7.1 命令行选项

*slapd*（8）支持手册页中详细说明的许多命令行选项。本节详细介绍了一些常用的选项。

```
-f <filename>
```

此选项指定slapd的备用配置文件。默认值通常是`/usr/local/etc/openldap/slapd.conf`。

```
-F <slapd-config-directory>
```

指定slapd配置目录。默认值为`/usr/local/etc/openldap/slapd.d`。

如果指定了`-f`和`-F`，则配置文件将被读取并转换为config目录格式并写入指定的目录。如果未指定任何选项，slapd将尝试读取默认配置目录，然后再尝试使用默认配置文件。如果存在有效的配置目录，那么默认配置文件将被忽略。所有使用配置选项的拍照工具都遵循同样的行为。

```
-h <URLs>
```

此选项指定备用侦听器配置。默认值是`ldap:///`这意味着LDAP 过度 TCP您可以指定特定的主机端口对或其他协议方案（如`ldaps://`或`ldapi://`）。

| **网址**    | **协议**        | **传输**        |
| --------- | ------------- | ------------- |
| LDAP：///  | LDAP          | TCP端口389      |
| LDAPS：/// | LDAP over SSL | TCP端口636      |
| ldapi：/// | LDAP          | IPC（Unix域套接字） |

例如，-h `ldaps:// ldap://127.0.0.1:666 `将创建两个监听器：一个用于默认`ldaps`上的所有接口上的（非标准）方案`ldaps：//`端口 636 ，另外一个用于端口666 上`localhost`（*loopback*）接口上的标准`ldap://`方案。主机可以使用主机名或 IPv4 要么 IPv6 地址。端口值必须是数字。

对于通过IPC的LDAP，可以在URL中编码Unix域套接字的路径名。请注意，目录分隔符必须与URL特殊的任何其他字符进行URL编码。因此，套接字`/usr/local/var/ldapi`必须编码为

```
ldapi://%2Fusr%2Flocal%2Fvar%2Fldapi
```

ldapi：在*使用LDAP机制*中详细描述[ [Chu-LDAPI](http://tools.ietf.org/html/draft-chu-ldap-ldapi-00) ]

请注意，ldapi：///传输没有被广泛实现：非OpenLDAP客户端可能无法使用它。

```
-n <service-name>
```

此选项指定用于记录和其他目的的服务名称。默认的服务名称是`slapd`。

```
-l <syslog-local-user>
```

此选项指定*syslog*（8）设施的本地用户。值可以是`LOCAL0`，`LOCAL1`，`LOCAL2`，...和`LOCAL7`。默认值为`LOCAL4`。所有系统可能不支持此选项。

```
-u user -g group
```

这些选项分别指定用户和组作为运行。 `用户`可以是用户名或uid。 `组`可以是组名或gid。

```
 -r directory
```

此选项指定运行时目录。打开侦听器之后，slapd将在读取任何配置文件或初始化任何后端之前将*chroot*（2）*chroot*（2）转到此目录。

```
-d <level> | ?
```

此选项将slapd调试级别设置为&lt;level&gt;。当级别是'？' 字符，打印各种调试级别并退出slapd，无论您提供任何其他选项。当前的调试级别是

**表7.1：调试级别**

| **水平** | **关键词**        | **描述**         |
| ------ | -------------- | -------------- |
| -1     | 任何             | 启用所有调试         |
| 0      |                | 没有调试           |
| 1      | （0x1跟踪）        | 跟踪函数调用         |
| 2      | （0x2数据包）       | 调试数据包处理        |
| 4      | （0x4 args）     | 重调查            |
| 8      | （0x8连接）        | 连接管理           |
| 16     | （0x10 BER）     | 打印发送和接收的数据包    |
| 32     | （0x20过滤器）      | 搜索过滤处理         |
| 64     | （0x40配置）       | 配置处理           |
| 128    | （0x80 ACL）     | 访问控制列表处理       |
| 256    | （0x100统计）      | 统计日志连接/操作/结果   |
| 512    | （0x200 stats2） | 统计日志条目发送       |
| 1024   | （0x400 shell）  | 打印与后台的通讯       |
| 2048   | （0x800解析）      | 打印输入解析调试       |
| 16384  | （0x4000同步）     | syncrepl消费者处理  |
| 32768  | （0x8000无）      | 只有设置了记录日志级别的消息 |

您可以通过为每个所需级别指定一次调试选项来启用多个级别。或者，由于调试级别相加，您可以自己做数学。也就是说，如果要跟踪函数调用并观察正在处理的配置文件，可以将级别设置为这两级的总和（在这种情况下为`-d 65`）。或者，您可以让slapd做数学，（例如`-d 1 -d 64`）。有关详细信息，请`参阅<ldap_log.h>`。

**注意：** slapd必须已经使用`--enable-debug`进行编译，以便在两个统计级别之外的任何调试信息都可用（默认值）定义。

### 7.2 开始slapd

一般来说，slapd是这样运行的：

```
/usr/local/libexec/slapd [<option>]*
```

其中`/usr/local/libexec `由`configure`确定，&lt;option&gt;是上述选项之一（或*slapd*（8））。除非你已经指定了一个调试级别（包括`0`级），否则slapd会自动从控制终端进行分支和分离，并在后台运行。

### 7.3 停止slapd

要安全地杀死*slapd*（8），你应该给出这样的命令

```
kill -INT `cat /usr/local/var/slapd.pid`
```

其中` /usr/local/var`由`configure`决定。

通过更加激烈的方法杀死slapd可能会导致信息丢失或数据库损坏。

## 8 访问控制

### 8.1 介绍

随着目录被越来越多的灵敏度数据填充，控制授予目录的访问种类变得越来越重要。例如，目录可能包含您可能需要通过合同或法律保护的机密性质的数据。或者，如果使用目录来控制对其他服务的访问，则对目录的不当访问可能会对您的站点的安全性造成攻击，从而对您的资产造成毁灭性破坏。

可以通过两种方法配置您的目录访问，第一种使用[slapd配置文件](https://www.openldap.org/doc/admin24/slapdconfig.html)，第二种使用*slapd-config*（5）格式（[配置slapd](https://www.openldap.org/doc/admin24/slapdconf2.html)）。

默认访问控制策略允许所有客户端读取。无论什么访问控制策略被定义，*rootdn*总是允许在一切和任何事情上的完全权限（即验证，搜索，比较，读取和写入）。

因此，在&lt;*by*&gt;子句中明确列出 *rootdn* 是无用的（并导致性能损失）。

以下部分将更详细地描述访问控制列表，并附有一些示例和建议。有关详细信息，请参阅*slapd.access*（5）。

### 8.2 通过静态配置访问控制

访问条目和属性由访问配置文件指令控制。访问线路的一般形式是：

```
<access directive> ::= access to <what>
    [by <who> [<access>] [<control>] ]+
<what> ::= * |
    [dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
    [filter=<ldapfilter>] [attrs=<attrlist>]
<basic-style> ::= regex | exact
<scope-style> ::= base | one | subtree | children
<attrlist> ::= <attr> [val[.<basic-style>]=<regex>] | <attr> , <attrlist>
<attr> ::= <attrname> | entry | children
<who> ::= * | [anonymous | users | self
        | dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
    [dnattr=<attrname>]
    [group[/<objectclass>[/<attrname>][.<basic-style>]]=<regex>]
    [peername[.<basic-style>]=<regex>]
    [sockname[.<basic-style>]=<regex>]
    [domain[.<basic-style>]=<regex>]
    [sockurl[.<basic-style>]=<regex>]
    [set=<setspec>]
    [aci=<attrname>]
<access> ::= [self]{<level>|<priv>}
<level> ::= none | disclose | auth | compare | search | read | write | manage
<priv> ::= {=|+|-}{m|w|r|s|c|x|d|0}+
<control> ::= [stop | continue | break]
```

其中`<what>`部分选择访问适用的条目和 / 或属性，`<who>`部分指定哪些实体被授予访问权限，并且`<access>`部分指定所授予的访问权限。支持多个`<who> <access> <control>`三元组，允许许多实体被授予对同一组条目和属性的不同访问权限。不是所有这些访问控制选项都在这里描述; 有关更多详细信息，请参阅*slapd.access*（5）手册页。

#### 8.2.1 什么来控制访问

访问规范的\<what>部分确定访问控制适用于的条目和属性。条目通常以两种方式进行选择：通过DN和过滤器。以下限定符通过DN选择条目：

```
to *
to dn[.<basic-style>]=<regex>
to dn.<scope-style>=<DN>
```

第一个表单用于选择所有条目。第二种形式可以用于通过将正则表达式与目标条目的*标准化DN*相匹配来选择条目。（本文档中没有进一步讨论第二种形式。）第三种形式用于选择DN请求范围内的条目。`<DN>`是可分辨名称的字符串表示形式，如[RFC4514所述](http://www.rfc-editor.org/rfc/rfc4514.txt)。

范围可以是`base`，`one`，`subtree`或`children`。当`基地`只提供DN的条目相匹配，`one`匹配，其父母为所提供的DN的条目，`subtree`的所有条目匹配，其根源在于所提供的DN子树，和`children`的所有条目匹配的DN下（但不是以项名为DN）。

例如，如果目录包含名为的条目：

```
0: o=suffix
1: cn=Manager,o=suffix
2: ou=people,o=suffix
3: uid=kdz,ou=people,o=suffix
4: cn=addresses,uid=kdz,ou=people,o=suffix
5: uid=hyc,ou=people,o=suffix
```

然后：

```
dn.base="ou=people,o=suffix" match 2;
dn.one="ou=people,o=suffix" match 3, and 5;
dn.subtree="ou=people,o=suffix" match 2, 3, 4, and 5; and
dn.children="ou=people,o=suffix" match 3, 4, and 5.
```

也可以使用过滤器选择条目：

```
to filter=<ldap filter>
```

其中`<ldap filter>`是LDAP搜索过滤器的字符串表示形式，如[RFC4515中所述](http://www.rfc-editor.org/rfc/rfc4515.txt)。例如：

```
to filter=(objectClass=person)
```

请注意，可以通过在`<what>`子句中包含两个限定符来由DN和过滤器选择条目。

```
to dn.one="ou=people,o=suffix" filter=(objectClass=person)
```

请注意，可以通过在`<what>`子句中包含两个限定符来由DN和过滤器选择条目。

```
attrs=<attribute list>
```

通过使用单个属性名称并使用值选择器来选择属性的特定值：

```
attrs=<attribute> val[.<style>]=<regex>
```

有两个特殊的*伪*属性`条目`和`子项`。要读取（并返回）目标条目，主体必须具有对目标*条目*属性的`读取`访问权限。要执行搜索，主题必须具有对搜索库*条目*属性的`搜索`访问权限。要添加或删除条目，主体必须`写入`到条目的访问`入口`属性，必须有`写`访问条目的父级的`子级`属性。要重命名的条目，主题必须`写`进入的访问`入口`属性并且具有对旧父级和新父级的`子级`属性的`写`访问权限。本节末尾的完整示例应该有助于清除。

最后，有一个特殊的条目选择器`“*”`用于选择任何条目。当没有提供其他`<what>`选择器时使用它。这相当于“ `dn=.*` ”

#### 8.2.2 谁授予访问权限

\<who>部分标识被授予访问权限的实体或实体。请注意，访问权限授予“实体”而不是“条目”。下表总结了实体说明符：

| **Specifier**                | **Entities**      |
| ---------------------------- | ----------------- |
| `*`                          | 全部，包括匿名和经过身份验证的用户 |
| `anonymous`                  | 匿名（未认证）用户         |
| `users`                      | 认证用户              |
| `self`                       | 与目标条目关联的用户        |
| `dn[.<basic-style>]=<regex>` | 用户匹配正则表达式         |
| `dn.<scope-style>=<DN>`      | DN范围内的用户          |

DN说明符的行为非常类似于&lt;what&gt;子句DN说明符。

还支持其他控制因素。例如，`<who>`可以被访问应用的条目中的DN值属性中列出的条目限制：

```
dnattr=<dn-valued attribute name>
```

dnattr规范用于授予访问其DN在条目的属性中的条目的访问权限（例如，授予访问组条目以列为组条目的所有者的任何用户的访问权限）。

在所有环境（或任何）环境中，某些因素可能不合适。例如，域名依赖于IP到域名查找。因为这些可以很容易被欺骗，因此应该避免领域因素。

#### 8.2.3 获得授权

授予的\<access>类型可以是以下之一：

**表6.4：访问级别**

| **Level**     | **权限**  | **说明**    |
| ------------- | ------- | --------- |
| none        = | 0       | 无法访问      |
| disclose    = | d       | 需要信息披露的错误 |
| auth        = | dx      | 需要验证（绑定）  |
| compare     = | cdx     | 需要比较      |
| search      = | scdx    | 需要应用搜索过滤器 |
| read        = | rscdx   | 需要阅读搜索结果  |
| write       = | wrscdx  | 需要修改/重命名  |
| manage      = | mwrscdx | 需要管理      |

每个级别意味着所有较低级别的访问。因此，例如，给予某人`写`访问的入口也授予他们`阅读`，`搜索`，`比较`，`权威性`和`公开`访问。但是，可以使用特权说明符来授予特定权限。

#### 8.2.4 访问控制评估

当评估某个请求者是否需要访问条目和/或属性时，slapd将条目和/或属性与配置文件中给出的`<what>`选择器进行比较。对于每个条目，数据库中提供的存储条目的访问控制（或全局访问指令，如果不保存在任何数据库中）首先应用，然后应用全局访问指令。然而，当处理访问列表时，由于全局访问列表被有效地附加到每个每个数据库列表中，所以如果结果列表不为空，则访问列表将以`* by * none`指令的隐式`访问`结束。如果没有适用于后端的访问指令，则使用默认读取。

在这个优先级中，访问指令按照它们在配置文件中出现的顺序进行检查。Slapd与第一个匹配条目和/或属性的`<what>`选择器停止。相应的访问指令是用于评估访问的一个slapd。

接下来，slapd将请求访问的实体与上面选择的访问指令中的`<who>`选择器按其出现顺序进行比较。它与第一个`<who>`选择符匹配请求者停止。这决定了请求访问的实体对入口和/或属性的访问。

最后，slapd将选定的`<access>`子句中授予的访问权限与客户端请求的访问进行比较。如果允许访问权限更大或相等，则授予访问权限。否则访问被拒绝。

访问指令的评估顺序使得它们在配置文件中的位置重要。如果一个访问指令比另一个访问指令更具体，那么它应该首先显示在配置文件中。类似地，如果一个`<who>`选择器比另一个更具体，那么它应该在access指令中首先出现。下面给出的访问控制示例应该有助于清楚。

#### 8.2.5 访问控制示例

上述访问控制设施相当强大。本节显示了用于描述目的的一些示例。

一个简单的例子：

```
access to * by * read
```

此访问指令授予对所有人的读访问权限。

```
access to *
    by self write
    by anonymous auth
    by * read
```

该指令允许用户修改其条目，允许匿名者对这些条目进行身份验证，并允许所有其他人读取这些条目。请注意，只有第一个`通过<who>`子句匹配。因此，匿名用户被授予`认证`，而不是`阅读`。最后一个条款也可以“ `被用户阅读` ”。

通常希望根据现有的保护级别来限制操作。以下显示如何使用安全强度因子（SSF）。

```
access to *
    by ssf=128 self write
    by ssf=64 anonymous auth
    by ssf=64 users read
```

如果已建立强度为128或更好的安全保护，允许用户修改自己的条目，允许对匿名用户进行身份验证，并在已建立64或更好的安全保护时读取访问权限。如果客户端尚未建立足够的安全保护，则将使用隐含`的* none`子句。

以下示例显示了使用样式说明符在两个访问指令中按顺序选择DN的条目。

```
access to dn.children="dc=example,dc=com"
     by * search
access to dn.children="dc=com"
     by * read
```

读取访问权限被授予`dc = com`子树下的条目，除了`dc = example，dc = com`子树之下的条目，授予搜索访问权限。由于两个访问指令都不匹配此DN，因此无法访问`dc = com`。如果这些访问指令的顺序被颠倒，则将永远不会达到尾随指令，因为`dc = example，dc = com`下的所有条目也都在`dc = com`条目下。

另请注意，如果没有`访问`指令与`<who>`子句匹配或否，**访问被拒绝**。也就是说，每个`对`指令的`访问都以``* none`子句隐含。当处理访问列表时，因为全局访问列表被有效地附加到每个数据库列表中，如果结果列表不为空，则访问列表将以`* by * none`指令的隐式`访问`结束。如果没有适用于后端的访问指令，则使用默认读取。

下一个例子再次显示了对访问指令和`<who>`子句的排序的重要性。它还显示了使用属性选择器来授予对特定属性和各种`<who>`选择器的访问权限。

```
access to dn.subtree="dc=example,dc=com" attrs=homePhone
    by self write
    by dn.children="dc=example,dc=com" search
    by peername.regex=IP=10\..+ read
access to dn.subtree="dc=example,dc=com"
    by self write
    by dn.children="dc=example,dc=com" search
    by anonymous auth
```

此示例适用于“ `dc = example，dc = com` ”子树中的条目。对于除`homePhone`之外的所有属性，一个条目可以写入自己，`example.com`条目下的条目可以由他们进行搜索，任何人除了认证/授权（总是匿名完成）之外没有访问权限（`由* none`隐含）。所述`HOMEPHONE`属性由条目，通过条目搜索下是可写`example.com`，通过从网络10连接的客户端可读的，否则不读取（隐式`由*无`）。所有其他访问被`* *`的隐式访问所拒绝。

有时，允许特定的DN从属性中添加或删除自己是有用的。例如，如果要创建一个组，并允许人们从成员属性中添加和删除其自己的DN，则可以使用如下所示的访问指令来实现：

```
access to attrs=member,entry
     by dnattr=member selfwrite
```

dnattr `<who>`选择器表示该访问适用于`成员`属性中列出的条目。自`写`访问选择器说，这样的成员只能从属性添加或删除自己的DN，而不是其他值。添加条目属性是必需的，因为访问该条目需要访问条目的任何属性。

### 8.3 通过动态配置访问控制

对slapd条目和属性的访问由olcAccess属性控制，其值是一系列访问指令。olcAccess配置的一般形式是：

```
olcAccess: <access directive>
<access directive> ::= to <what>
    [by <who> [<access>] [<control>] ]+
<what> ::= * |
    [dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
    [filter=<ldapfilter>] [attrs=<attrlist>]
<basic-style> ::= regex | exact
<scope-style> ::= base | one | subtree | children
<attrlist> ::= <attr> [val[.<basic-style>]=<regex>] | <attr> , <attrlist>
<attr> ::= <attrname> | entry | children
<who> ::= * | [anonymous | users | self
        | dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
    [dnattr=<attrname>]
    [group[/<objectclass>[/<attrname>][.<basic-style>]]=<regex>]
    [peername[.<basic-style>]=<regex>]
    [sockname[.<basic-style>]=<regex>]
    [domain[.<basic-style>]=<regex>]
    [sockurl[.<basic-style>]=<regex>]
    [set=<setspec>]
    [aci=<attrname>]
<access> ::= [self]{<level>|<priv>}
<level> ::= none | disclose | auth | compare | search | read | write | manage
<priv> ::= {=|+|-}{m|w|r|s|c|x|d|0}+
<control> ::= [stop | continue | break]
```

其中\<what>部分选择访问适用的条目和/或属性，`<who>`部分指定哪些实体被授予访问权限，并且`<access>`部分指定所授予的访问权限。支持多个`<who> <access> <control>`三元组，允许许多实体被授予对同一组条目和属性的不同访问权限。不是所有这些访问控制选项都在这里描述; 有关更多详细信息，请参阅*slapd.access*（5）手册页。

#### 8.3.1 什么来控制访问

访问规范的\<what>部分确定访问控制适用于的条目和属性。条目通常以两种方式进行选择：通过DN和过滤器。以下限定符通过DN选择条目：

```
to *
to dn[.<basic-style>]=<regex>
to dn.<scope-style>=<DN>
```

第一个表单用于选择所有条目。第二种形式可以用于通过将正则表达式与目标条目的*标准化DN*相匹配来选择条目。（本文档中没有进一步讨论第二种形式。）第三种形式用于选择DN请求范围内的条目。\<DN>是可分辨名称的字符串表示形式，如[RFC4514所述](http://www.rfc-editor.org/rfc/rfc4514.txt)。

范围可以是`base`，`one`，`subtree`或`children`。当`基地`只提供DN的条目相匹配，`one`匹配，其父母为所提供的DN的条目，`subtree`的所有条目匹配，其根源在于所提供的DN子树，和`children`的所有条目匹配的DN下（但不是以项名为DN）。

例如，如果目录包含名为的条目：

```
0: o=suffix
1: cn=Manager,o=suffix
2: ou=people,o=suffix
3: uid=kdz,ou=people,o=suffix
4: cn=addresses,uid=kdz,ou=people,o=suffix
5: uid=hyc,ou=people,o=suffix
```

然后：

```
dn.base="ou=people,o=suffix" match 2;
dn.one="ou=people,o=suffix" match 3, and 5;
dn.subtree="ou=people,o=suffix" match 2, 3, 4, and 5; and
dn.children="ou=people,o=suffix" match 3, 4, and 5.
```

也可以使用过滤器选择条目：

```
to filter=<ldap filter>
```

其中\<ldap filter>是LDAP搜索过滤器的字符串表示形式，如[RFC4515中所述](http://www.rfc-editor.org/rfc/rfc4515.txt)。例如：

```
to filter=(objectClass=person)
```

请注意，可以通过在\<what>子句中包含两个限定符来由DN和过滤器选择条目。

```
to dn.one="ou=people,o=suffix" filter=(objectClass=person)
```

通过在\<what>选择器中包含逗号分隔的属性名称列表来选择条目中的属性：

```
attrs=<attribute list>
```

通过使用单个属性名称并使用值选择器来选择属性的特定值：

```
attrs=<attribute> val[.<style>]=<regex>
```

有两个特殊的*伪*属性`条目`和`子项`。要读取（并返回）目标条目，主体必须具有对目标*条目*属性的`读取`访问权限。要执行搜索，主题必须具有对搜索库*条目*属性的`搜索`访问权限。要添加或删除条目，主体必须`写入`到条目的访问`入口`属性，必须有`写`访问条目的父级的`子级`属性。要重命名的条目，主题必须`写`进入的访问`入口`属性并且具有对旧父级和新父级的`子级`属性的`写`访问权限。本节末尾的完整示例应该有助于清除。

最后，有一个特殊的条目选择器`“*”`用于选择任何条目。当没有提供其他`<what>`选择器时使用它。这相当于“ `dn =。*` ”

#### 8.3.2 谁授予访问权限

\<who>部分标识被授予访问权限的实体或实体。请注意，访问权限授予“实体”而不是“条目”。下表总结了实体说明符：

**表5.3：访问实体指定符**

| **Specifier**                | **Entities**      |
| ---------------------------- | ----------------- |
| *                            | 全部，包括匿名和经过身份验证的用户 |
| anonymous                    | 匿名（未认证）用户         |
| users                        | 认证用户              |
| self                         | 与目标条目关联的用户        |
| dn[.\<basic-style>]=\<regex> | 用户匹配正则表达式         |
| dn.\<scope-style>=\<DN>      | DN范围内的用户          |

DN说明符的行为非常类似于\<what>子句DN说明符。

还支持其他控制因素。例如，`<who>`可以被访问应用的条目中的DN值属性中列出的条目限制：

```
dnattr=<dn-valued attribute name>
```

dnattr规范用于授予访问其DN在条目的属性中的条目的访问权限（例如，授予访问组条目以列为组条目的所有者的任何用户的访问权限）。

在所有环境（或任何）环境中，某些因素可能不合适。例如，域名依赖于IP到域名查找。因为这些可以很容易被欺骗，因此应该避免领域因素。

#### 8.3.3 获得授权

授予的\<access>类型可以是以下之一：

| Level    | 权限       | 描述        |
| -------- | -------- | --------- |
| none     | =0       | 无法访问      |
| disclose | =d       | 需要信息披露的错  |
| auth     | =dx      | 需要验证（绑定）  |
| compare  | =cdx     | 需要比较      |
| search   | =scdx    | 需要应用搜索过滤器 |
| read     | =rscdx   | 需要阅读搜索结果  |
| write    | =wrscdx  | 需要修改/重命名  |
| manage   | =mwrscdx | 需要管理      |

每个级别意味着所有较低级别的访问。因此，例如，给予某人`写`访问的入口也授予他们`阅读`，`搜索`，`比较`，`权威性`和`公开`访问。但是，可以使用特权说明符来授予特定权限。

#### 8.3.4 访问控制评估

当评估某个请求者是否需要访问条目和/或属性时，slapd将条目和/或属性与配置中给出的`<what>`选择器进行比较。对于每个条目，在数据库中提供的存储条目的访问控制（或全局访问指令，如果不保存在任何数据库中）首先应用，然后应用全局访问指令（保存在`前端`数据库定义中）。然而，当处理访问列表时，由于全局访问列表被有效地附加到每个每个数据库列表中，所以如果结果列表不为空，则访问列表将以`* by * none`指令的隐式`访问`结束。如果没有适用于后端的访问指令，则使用默认读取。

在此优先级内，按照配置属性中显示的顺序检查访问指令。Slapd与第一个匹配条目和/或属性的`<what>`选择器停止。相应的访问指令是用于评估访问的一个slapd。

接下来，slapd将请求访问的实体与上面选择的访问指令中的`<who>`选择器按其出现顺序进行比较。它与第一个`<who>`选择符匹配请求者停止。这决定了请求访问的实体对入口和/或属性的访问。

最后，slapd将选定的`<access>`子句中授予的访问权限与客户端请求的访问进行比较。如果允许访问权限更大或相等，则授予访问权限。否则访问被拒绝。

访问指令的评估顺序使得它们在配置文件中的位置重要。如果一个访问指令在其选择的条目方面比其他访问指令更具体，那么它应该首先显示在配置中。类似地，如果一个`<who>`选择器比另一个更具体，那么它应该在access指令中首先出现。下面给出的访问控制示例应该有助于清楚。

#### 8.3.5 访问控制示例

上述访问控制设施相当强大。本节显示了用于描述目的的一些示例。

一个简单的例子：

```
olcAccess: to * by * read
```

此访问指令授予对所有人的读访问权限。

```
olcAccess: to *
    by self write
    by anonymous auth
    by * read
```

该指令允许用户修改其条目，允许匿名者对这些条目进行身份验证，并允许所有其他人读取这些条目。请注意，只有第一个`通过<who>`子句匹配。因此，匿名用户被授予`认证`，而不是`阅读`。最后一个条款也可以“ `被用户阅读` ”。

通常希望根据现有的保护级别来限制操作。以下显示如何使用安全强度因子（SSF）。

```
olcAccess: to *
    by ssf=128 self write
    by ssf=64 anonymous auth
    by ssf=64 users read
```

该指令允许用户修改自己的条目，如果强度为128或更好的安全保护已建立，允许对匿名用户的身份验证访问，并且在强度达到64或更好的安全保护建立时读取访问权限。如果客户端尚未建立足够的安全保护，则将使用隐含的`by * none`子句。

以下示例显示了使用样式说明符在两个访问指令中选择DN的条目，其中排序很重要。

```
olcAccess: to dn.children="dc=example,dc=com"
     by * search
olcAccess: to dn.children="dc=com"
     by * read
```

读取访问权限被授予`dc = com`子树下的条目，除了`dc = example，dc = com`子树之下的条目，授予搜索访问权限。由于两个访问指令都不匹配此DN，因此无法访问`dc = com`。如果这些访问指令的顺序被颠倒，则将永远不会达到尾随指令，因为`dc = example，dc = com`下的所有条目也都在`dc = com`条目下。

另请注意，如果没有`olcAccess：to`指令匹配或否`由<who>`子句，**访问被拒绝**。当处理访问列表时，因为全局访问列表被有效地附加到每个数据库列表中，如果结果列表不为空，则访问列表将以`* by * none`指令的隐式`访问`结束。如果没有适用于后端的访问指令，则使用默认读取。

下一个例子再次显示了对访问指令和`<who>`子句的排序的重要性。它还显示了使用属性选择器来授予对特定属性和各种`<who>`选择器的访问权限。

```
olcAccess: to dn.subtree="dc=example,dc=com" attrs=homePhone
    by self write
    by dn.children=dc=example,dc=com" search
    by peername.regex=IP=10\..+ read
olcAccess: to dn.subtree="dc=example,dc=com"
    by self write
    by dn.children="dc=example,dc=com" search
    by anonymous auth
```

此示例适用于“ `dc = example，dc = com` ”子树中的条目。对于除`homePhone`之外的所有属性，一个条目可以写入自己，`example.com`条目下的条目可以由他们进行搜索，任何人除了认证/授权（总是匿名完成）之外没有访问权限（`由* none`隐含）。所述`HOMEPHONE`属性由条目，通过条目搜索下是可写`example.com`，通过从网络10连接的客户端可读的，否则不读取（隐式`由*无`）。所有其他访问被`* *`的隐式访问所拒绝。

有时，允许特定的DN从属性中添加或删除自己是有用的。例如，如果要创建一个组，并允许人们从成员属性中添加和删除其自己的DN，则可以使用如下所示的访问指令来实现：

```
olcAccess: to attrs=member,entry
     by dnattr=member selfwrite
```

dnattr `<who>`选择器表示该访问适用于`成员`属性中列出的条目。自`写`访问选择器说，这样的成员只能从属性添加或删除自己的DN，而不是其他值。添加条目属性是必需的，因为访问该条目需要访问条目的任何属性。

#### 8.3.6 访问控制订购

由于`olcAccess`指令的排序对其正确的评估至关重要，但LDAP属性通常不会保留其值的排序，因此OpenLDAP使用自定义模式扩展来维护这些值的固定排序。通过为每个值预先添加一个`“{X}”`数字索引来维护此顺序，与用于排序配置条目的方法类似。这些索引标签由slapd自动保存，在最初定义值时不需要指定。例如，创建设置时

```
olcAccess: to attrs=member,entry
     by dnattr=member selfwrite
olcAccess: to dn.children="dc=example,dc=com"
     by * search
olcAccess: to dn.children="dc=com"
     by * read
```

当您使用slapcat或ldapsearch读取它们时，它们将包含

```
olcAccess: {0}to attrs=member,entry
     by dnattr=member selfwrite
olcAccess: {1}to dn.children="dc=example,dc=com"
     by * search
olcAccess: {2}to dn.children="dc=com"
     by * read
```

数字索引可用于指定使用ldapmodify编辑访问规则时要更改的特定值。可以使用该索引来代替（或除了）实际的访问值。当管理多个访问规则时，使用此数字索引非常有用。

例如，如果我们需要更改上面的第二个规则来授予写访问权限而不是搜索，我们可以尝试这个LDIF：

```
changetype: modify
delete: olcAccess
olcAccess: to dn.children="dc=example,dc=com" by * search
-
add: olcAccess
olcAccess: to dn.children="dc=example,dc=com" by * write
-
```

但是，此示例**不能**保证现有值保持原始顺序，因此很可能会导致安全配置不正确。相反，应使用数字索引：

```
changetype: modify
delete: olcAccess
olcAccess: {1}
-
add: olcAccess
olcAccess: {1}to dn.children="dc=example,dc=com" by * write
-
```

此示例删除`olcAccess`属性（不管其值）的值＃1中的任何规则，并添加明确插入为值＃1的新值。结果将是

```
olcAccess: {0}to attrs=member,entry
     by dnattr=member selfwrite
olcAccess: {1}to dn.children="dc=example,dc=com"
     by * write
olcAccess: {2}to dn.children="dc=com"
     by * read
```

这正是目的。

### 8.4 访问控制常见示例

#### 8.4.1 基本ACL

一般应该从一些基本的ACL开始，如：

```
access to attrs=userPassword
    by self =xw
    by anonymous auth
    by * none


  access to *
    by self write
    by users read
    by * none
```

第一个ACL允许用户更新（而不是读取）他们的密码，匿名用户对此属性进行身份验证，并且（隐含地）拒绝所有对他人的访问。

第二个ACL允许用户完全访问其条目，经过身份验证的用户读取任何内容，并（隐式）拒绝对其他人（在本例中为匿名用户）的所有访问。

#### 8.4.2 匹配匿名和经过身份验证的用户

匿名用户具有空DN。虽然可以使用*dn.exact =“”*或*dn.regex =“^ $”*，但*slapd*（8））提供了一个匿名的简写，而应该使用它。

```
access to *
  by anonymous none
  by * read
```

拒绝所有访问匿名用户，同时授予他人阅读。

经过身份验证的用户拥有主题DN。虽然*dn.regex =“。+”*将匹配任何经过身份验证的用户，但OpenLDAP为用户提供了短用的代码。

```
access to *
  by users read
  by * none
```

此ACL在拒绝其他用户（即：匿名用户））时向已验证的用户授予读取权限。

#### 8.4.3 控制rootdn访问

您可以在*slapd.conf*（5）或*slapd.d中*指定*rootdn*，而不指定*rootpw*。那么你必须添加一个具有相同dn的实际目录条目，例如：

```
dn: cn=Manager,o=MyOrganization
cn: Manager
sn: Manager
objectClass: person
objectClass: top
userPassword: {SSHA}someSSHAdata
```

然后绑定为*rootdn*将需要对该DN进行常规绑定，这又需要对该条目的DN和*userPassword进行*身份验证访问，这可以通过ACL进行限制。例如：

```
access to dn.base="cn=Manager,o=MyOrganization"
  by peername.regex=127\.0\.0\.1 auth
  by peername.regex=192\.168\.0\..* auth
  by users none
  by * none
```

上面的ACL只允许使用roothost从localhost和192.168.0.0/24进行绑定。

#### 8.4.4 使用组管理访问

有几种方法可以做到这一点。这里说明了一种方法。考虑以下DIT布局：

```
+-dc=example,dc=com
+---cn=administrators,dc=example,dc=com
+---cn=fred blogs,dc=example,dc=com
```

和以下组对象（LDIF格式）：

```
dn: cn=administrators,dc=example,dc=com
cn: administrators of this region
objectclass: groupOfNames  (important for the group acl feature)
member: cn=fred blogs,dc=example,dc=com
member: cn=somebody else,dc=example,dc=com
```

然后，可以通过将适当*的group*子句添加到*slapd.conf*（5）中的访问指令，来授予对此组的成员的访问权限。例如，

```
access to dn.children="dc=example,dc=com"
    by self write
    by group.exact="cn=Administrators,dc=example,dc=com" write
    by * auth
```

像*dn*子句一样，还可以使用*expand*来根据目标的正则表达式匹配来扩展组名，也就是到*dn.regex*。例如，

```
access to dn.regex="(.+,)?ou=People,(dc=[^,]+,dc=[^,]+)$"
         attrs=children,entry,uid
    by group.expand="cn=Managers,$2" write
    by users read
    by * auth
```

以上图示假定要在*groupOfNames*对象类的*成员*属性类型中找到组成员。如果您需要使用不同的组对象和/或不同的属性类型，请使用以下*slapd.conf*（5）（缩写）语法：

```
access to <what>
        by group/<objectclass>/<attributename>=<DN> <access>
```

例如：

```
access to *
  by group/organizationalRole/roleOccupant="cn=Administrator,dc=example,dc=com" write
```

在这种情况下，我们有一个ObjectClass *organizationalRole*，其中包含*roleOccupant*属性中的管理员DN 。例如：

```
dn: cn=Administrator,dc=example,dc=com
cn: Administrator
objectclass: organizationalRole
roleOccupant: cn=Jane Doe,dc=example,dc=com
```

**注意：**指定的成员属性类型必须为DN或*NameAndOptionalUID*语法，并且指定的对象类SHOULD允许属性类型。

#### 8.4.5 授予对属性子集的访问权限

您可以通过在ACL *to*子句中指定属性名称列表来授予对一组属性的访问权限。为了有用，您还需要授予对该*条目*本身的访问权限。还要注意*儿童*如何控制添加，删除和重命名条目的能力。

```
# mail: self may write, authenticated users may read
access to attrs=mail
  by self write
  by users read
  by * none

# cn, sn: self my write, all may read
access to attrs=cn,sn
  by self write
  by * read

# immediate children: only self can add/delete entries under this entry
access to attrs=children
  by self write

# entry itself: self may write, all may read
access to attrs=entry
  by self write
  by * read

# other attributes: self may write, others have no access
access to *
  by self write
  by * none
```

也可以在此列表中指定ObjectClass名称，这将影响该*objectClass*所需和/或允许的所有属性。实际上，以前缀为*@* 是 *attrlist*中的名称直接被视为objectClass名称。一个名字作为前缀*！*也被视为一个对象类，但在这种情况下，访问规则影响不需要的，也不由允许的属性*对象类*。

#### 8.4.6 允许用户写入其下的所有条目

对于用户可以写入自己的记录及其所有子项的设置：

```
access to dn.regex="(.+,)?(uid=[^,]+,o=Company)$"
   by dn.exact,expand="$2" write
   by anonymous auth
```

（添加上面的更多示例）

#### 8.4.7 允许创建条目

假设你有这样的想法：

```
o=<basedn>
    ou=domains
        associatedDomain=<somedomain>
            ou=users
                uid=<someuserid>
                uid=<someotheruserid>
            ou=addressbooks
                uid=<someuserid>
                    cn=<someone>
                    cn=<someoneelse>
```

而另一个域\<someotherdomain>：

```
o=<basedn>
    ou=domains
        associatedDomain=<someotherdomain>
            ou=users
                uid=<someuserid>
                uid=<someotheruserid>
            ou=addressbooks
                uid=<someotheruserid>
                    cn=<someone>
                    cn=<someoneelse>
```

那么，如果你想用户*的uid = \<someuserid>*以**ONLY**创造自己的事项，你可以写一个这样的ACL：

```
# this rule lets users of "associatedDomain=<matcheddomain>"
# write under "ou=addressbook,associatedDomain=<matcheddomain>,ou=domains,o=<basedn>",
# i.e. a user can write ANY entry below its domain's address book;
# this permission is necessary, but not sufficient, the next
# will restrict this permission further


access to dn.regex="^ou=addressbook,associatedDomain=([^,]+),ou=domains,o=<basedn>$" attrs=children
        by dn.regex="^uid=([^,]+),ou=users,associatedDomain=$1,ou=domains,o=<basedn>$$" write
        by * none


# Note that above the "by" clause needs a "regex" style to make sure
# it expands to a DN that starts with a "uid=<someuserid>" pattern
# while substituting the associatedDomain submatch from the "what" clause.


# This rule lets a user with "uid=<matcheduid>" of "<associatedDomain=matcheddomain>"
# write (i.e. add, modify, delete) the entry whose DN is exactly
# "uid=<matcheduid>,ou=addressbook,associatedDomain=<matcheddomain>,ou=domains,o=<basedn>"
# and ANY entry as subtree of it


access to dn.regex="^(.+,)?uid=([^,]+),ou=addressbook,associatedDomain=([^,]+),ou=domains,o=<basedn>$"
        by dn.exact,expand="uid=$2,ou=users,associatedDomain=$3,ou=domains,o=<basedn>" write
        by * none


# Note that above the "by" clause uses the "exact" style with the "expand"
# modifier because now the whole pattern can be rebuilt by means of the
# submatches from the "what" clause, so a "regex" compilation and evaluation
# is no longer required.
```

#### 8.4.8 在Access Control中使用正则表达式的提示

当您打算使用正则表达式匹配时，始终使用*dn.regex = \<pattern>*。*dn = \<pattern>*单独默认为*dn.exact \<pattern>*。

当您想要至少匹配一个字符时，使用*（.+）*而不是*（.\*）*。*（.\*）也*匹配空字符串。

不要使用正则表达式，否则可以以更安全和更便宜的方式进行匹配。例子：

```
dn.regex=".*dc=example,dc=com"
```

是不安全和昂贵的：

- 不安全，因为包含*dc=example，dc=com的*任何字符串都不会匹配，而不是以所需的模式结束的字符串; 使用** dc=example，dc=com $*。
- 不安全，因为它将允许任何*attributeType*以*dc*结尾为字符串中第一个RDN的命名属性，例如自定义的attributeType *mydc*也会匹配。如果真的需要一个正则表达式，它只允许*dc=example，dc=com*或其任何子树，请使用*^(,+,)？dc = example，dc = com $*，这意味着：dc=...（如果有的话（括号内的模式后面的问号）必须以逗号结尾;
- 昂贵，因为如果你不需要submatches，你可以使用范围设计样式，例如

```
dn.subtree="dc=example,dc=com"
```

在匹配模式中包括*dc = example，dc = com*，

```
dn.children="dc=example,dc=com"
```

从匹配模式中排除*dc = example，dc = com*，或者

```
dn.onelevel="dc=example,dc=com"
```

只允许一个子级匹配。

因为*ou =（。+），ou =（。+），ou=addressbooks，o=basedn*将匹配*something=bla，ou=xxx，ou=yyy，ou=addressbooks，*总是在正则表达式中使用*^*和*$ *，o=basedn中，ou=addressbooks，o= BaseDN中，dc=some，dc=org

始终使用*（\[^，\] +）*来表示一个RDN，因为*（。+）*可以包含任意数量的RDN; 例如*ou =（。+），dc = example，dc = com*将匹配*ou = My，o = Org，dc = example，dc = com*，这可能不是你想要的。

不要将rootdn添加到by子句中。对于使用rootdn身份执行的操作，ACL甚至不会被处理（否则根本就没有理由定义rootdn）。

使用shorthands。用户指令与经过身份验证的用户匹配，匿名指令与匿名用户匹配。

如果您需要的是范围和/或子字符串替换，则不要使用*dn.regex*表单来执行<by>子句。使用范围设计样式（例如，*精确*，*一级*，*子级*或*子树*），并且样式修饰符将展开以引起子字符串扩展。

例如，

```
access to dn.regex=".+,dc=([^,]+),dc=([^,]+)$"
  by dn.regex="^[^,],ou=Admin,dc=$1,dc=$2$$" write
```

虽然正确，可以安全有效地代替

```
access to dn.regex=".+,(dc=[^,]+,dc=[^,]+)$"
  by dn.onelevel,expand="ou=Admin,$1" write
```

其中*\<what\>*子句中的正则表达式更紧凑，并且*\<by\>*子句中的正则表达式被具有子字符串扩展的更高效的一级方法范围取代。

#### 8.4.9 基于安全强度因素（ssf）授予和拒绝访问权限

您可以根据安全强度因子（SSF）限制访问

```
access to dn="cn=example,cn=edu"
      by * ssf=256 read
```

0（零）意味着没有保护，1表示完整性保护，56 DES或其他弱密码，112三重DES和其他强密码，128 RC4，Blowfish等现代强密码。

其他可能性：

```
transport_ssf=<n>
tls_ssf=<n>
sasl_ssf=<n>
```

建议使用256。

请参见*slapd.conf中*的信息（5）*SSF*。

#### 8.4.10 当事情不按预期工作时

考虑这个例子：

```
access to *
  by anonymous auth

access to *
  by self write

access to *
  by users read
```

您可能认为这将允许任何用户登录，阅读所有内容并更改自己的数据，如果他已经登录。但是在这个例子中，只有登录工作，并且ldapsearch不返回任何数据。问题是，SLAPD逐行执行其访问配置，并在访问规则的一部分找到匹配后立即停止（这里：***）

要获得我们想要的文件必须阅读：

```
access to *
  by anonymous auth
  by self write
  by users read
```

一般规则是：“特殊访问规则第一，通用访问规则最后”

另请参见*slapd.access*（5），loglevel 128和*slapacl*（8）来调试信息。

### 8.5 集 - 根据关系授予权限

集合通过示例最好地说明。以下部分将介绍几个ACL示例，以便于他们的理解。

（设置在访问控制FAQ条目：[http](http://www.openldap.org/faq/data/cache/1133.html) : [//www.openldap.org/faq/data/cache/1133.html](http://www.openldap.org/faq/data/cache/1133.html)）

**注意：**集合被认为是实验性的。

#### 8.5.1 群组

组的OpenLDAP ACL不会扩展组内的组，它们是具有另一组作为成员的组。例如：

```
dn: cn=sudoadm,ou=group,dc=example,dc=com
cn: sudoadm
objectClass: groupOfNames
member: uid=john,ou=people,dc=example,dc=com
member: cn=accountadm,ou=group,dc=example,dc=com

dn: cn=accountadm,ou=group,dc=example,dc=com
cn: accountadm
objectClass: groupOfNames
member: uid=mary,ou=people,dc=example,dc=com
```

如果我们使用具有上述条目的标准组ACL，并允许`sudoadm`组的成员写入某个地方，则不会包括`mary`：

```
access to dn.subtree="ou=sudoers,dc=example,dc=com"
        by group.exact="cn=sudoadm,ou=group,dc=example,dc=com" write
        by * read
```

使用集合，我们可以使ACL成为递归，并在组内考虑组。因此，对于每个成员，它进一步扩展：

```
 access to dn.subtree="ou=sudoers,dc=example,dc=com"
       by set="[cn=sudoadm,ou=group,dc=example,dc=com]/member* & user" write
       by * read
```

此设置ACL意味着：取`cn = sudoadm` DN，检查其`成员`属性（“ `*` ”表示递归），并将结果与经过身份验证的用户DN相交。如果结果不为空，则将ACL视为匹配，并授予写访问权限。

下图解释了如何构建这个集合：

![图XY：填充递归组集](https://qiniu.iclouds.work/FoylQKTYLfguYkfrAUxV1bcWeofE.png)

首先我们得到`uid = john` DN。此条目没有`成员`属性，因此扩展在此停止。现在我们来到`cn = accountadm`。这个确实有一个`成员`属性，这是`uid = mary`。该`UID =玛丽`项，然而，没有成员，所以我们在这里再次停止。最后的比较是：

```
{"uid=john,ou=people,dc=example,dc=com","uid=mary,ou=people,dc=example,dc=com"} & user
```

如果经过身份验证的用户的DN是其中的任何一个，则授予写访问权限。所以这集将包括`玛丽`中`sudoadm`组，她将被允许写访问。

#### 8.5.2 没有DN语法的组ACL

传统的组ACL，甚至上一个关于递归组的例子，都要求成员被指定为DN而不是用户名。

然而，使用集合，也可以在组ACL中使用简单的名称，如该示例所示。

假设我们要允许`sudoadm`组的成员写入我们树的`ou = suders`分支。但是我们现在的组定义是使用`memberUid`作为组成员：

```
dn: cn=sudoadm,ou=group,dc=example,dc=com
cn: sudoadm
objectClass: posixGroup
gidNumber: 1000
memberUid: john
```

使用这种类型的组，我们不能使用组ACL。但是使用一组ACL，我们可以授予所需的访问权限：

```
access to dn.subtree="ou=sudoers,dc=example,dc=com"
      by set="[cn=sudoadm,ou=group,dc=example,dc=com]/memberUid & user/uid" write
      by * read
```

我们使用一个简单的交集，我们将连接（和认证）用户的`uid`属性与组的`memberUid`属性进行比较。如果匹配，交叉路口不为空，ACL将授予写入权限。

当连接用户被认证为`uid = john，ou = people，dc = example，dc = com`时，此图说明了该集合：

![图XY：使用`memberUid进行`设置](https://qiniu.iclouds.work/Fn9FyhrGdS01P9RVlzfAuSJLfmUH.png)

在这种情况下，这是一场比赛。然而，如果它是`mary`验证的，她将被拒绝对`ou = sudoers的`写入权限，因为她的`uid`属性没有列在该组的`memberUid中`。

#### 8.5.3 以下参考

现在我们将展示一个非常强大的例子，说明可以用套来做什么。这个例子倾向于使OpenLDAP管理员在理解它及其含义之后微笑。

我们从一个用户条目开始：

```
dn: uid=john,ou=people,dc=example,dc=com
uid: john
objectClass: inetOrgPerson
givenName: John
sn: Smith
cn: john
manager: uid=mary,ou=people,dc=example,dc=com
```

编写ACL以允许管理员更新某些属性是非常简单的使用集合：

```
access to dn.exact="uid=john,ou=people,dc=example,dc=com"
   attrs=carLicense,homePhone,mobile,pager,telephoneNumber
   by self write
   by set="this/manager & user" write
   by * read
```

在该集合中，`这`扩展到被访问的条目，以便在访问约翰条目时，`该/ manager`扩展为`uid = mary，ou = people，dc = example，dc = com`。如果经理自己正在访问约翰的条目，则ACL将匹配，并且将授予对这些属性的写访问权限。

到目前为止，使用`dnattr`关键字可以获得同样的行为。然而，使用集合，我们可以进一步增强此ACL。假设我们想允许经理的秘书也更新这些属性。这是我们的做法：

```
access to dn.exact="uid=john,ou=people,dc=example,dc=com"
   attrs=carLicense,homePhone,mobile,pager,telephoneNumber
   by self write
   by set="this/manager & user" write
   by set="this/manager/secretary & user" write
   by * read
```

现在我们需要一张照片来帮助解释这里发生了什么（为了清楚起见，条目缩短）：

![图XY：设置跳过条目](https://qiniu.iclouds.work/FpRVHSwbdNwpOccDGg141q4ayWKR.png)

在这个例子中，简是玛丽的秘书，是约翰的经理。这个整体关系由`经理`和`秘书`属性定义，它们都是distinguishedName语法（即，完整的DN）。因此，当`uid = john`条目被访问时，`这个/ manager / secretary`集合变为`{“uid = jane，ou = people，dc = example，dc = com”` }（遵循图片中的引用）：

```
this = [uid=john,ou=people,dc=example,dc=com]
this/manager = \
  [uid=john,ou=people,dc=example,dc=com]/manager = uid=mary,ou=people,dc=example,dc=com
this/manager/secretary = \
  [uid=mary,ou=people,dc=example,dc=com]/secretary = uid=jane,ou=people,dc=example,dc=com
```

最终的结果是当Jane访问John的条目时，她将被授予对指定属性的写访问权限。更好的是，这将会发生在她访问的任何一个玛丽作为经理。

这是很酷和好，但也许给秘书太多的权力。也许我们需要进一步限制。例如，让我们只允许执行秘书有这样的权力：

```
access to dn.exact="uid=john,ou=people,dc=example,dc=com"
  attrs=carLicense,homePhone,mobile,pager,telephoneNumber
  by self write
  by set="this/manager & user" write
  by set="this/manager/secretary &
          [cn=executive,ou=group,dc=example,dc=com]/member* &
          user" write
  by * read
```

它与以前几乎是一样的ACL，但我们现在也要求连接用户是（可能嵌套的）`cn =executive`组的成员。

## 13 模式规范

本章介绍如何扩展*slapd*（8）使用的用户模式。本章假设读者熟悉了LDAP/X.500 信息模型。

第一部分，[分布式模式文件](https://www.openldap.org/doc/admin24/schema.html#Distributed Schema Files)详细说明了分发中提供的可选模式定义以及获取其他定义的位置。第二部分“ [扩展架构](https://www.openldap.org/doc/admin24/schema.html#Extending Schema) ”详细介绍了如何定义新的架构项目。

本章不讨论如何扩展*slapd*（8）使用的系统模式，因为这需要源代码修改。系统架构包括所有操作属性类型或允许或需要操作属性（直接或间接）的任何对象类。

### 13.1 分布式模式文件

OpenLDAP软件分布有一组模式规范供您使用。每个集合都在一个适合包含（使用`include`指令）的文件中定义在您的*slapd.conf*（5）文件中。这些模式文件通常安装在`/usr/local/etc/openldap/schema`目录中。

| **File**             | **Description**                    |
| -------------------- | ---------------------------------- |
| core.schema          | OpenLDAP *core* (required)         |
| cosine.schema        | Cosine and Internet X.500 (useful) |
| inetorgperson.schema | InetOrgPerson (useful)             |
| misc.schema          | Assorted (experimental)            |
| nis.schema           | Network Information Services (FYI) |
| openldap.schema      | OpenLDAP Project (experimental)    |

要使用任何这些模式文件，您只需要将所需的文件包含在*slapd.conf*（5）文件的全局定义部分中。例如：

```
# include schema
include /usr/local/etc/openldap/schema/core.schema
include /usr/local/etc/openldap/schema/cosine.schema
include /usr/local/etc/openldap/schema/inetorgperson.schema
```

其他文件可能可用。请咨询OpenLDAP常问问题（<http://www.openldap.org/faq/>）。

**注意：**不应修改提供的文件中定义的任何模式项。

### 13.2 扩展架构

可以扩展*slapd*（8）使用的模式，以支持其他语法，匹配规则，属性类型和对象类。本章详细介绍了如何使用slapd支持的语法和匹配规则添加用户应用程序属性类型和对象类。slapd也可以扩展以支持额外的语法，匹配规则和系统模式，但这需要一些编程，因此这里不讨论。

定义新模式有五个步骤：

1. 获取对象标识符
2. 选择名称前缀
3. 创建本地架构文件
4. 定义自定义属性类型（如有必要）
5. 定义自定义对象类

#### 13.2.1 对象标识符

每个架构元素由全局唯一标识 对象标识符（OID）。OID也用于识别其他对象。它们通常用协议描述ASN.1。特别是，它们被大量使用简单网络管理协议（SNMP）。由于OID是分级的，您的组织可以获得一个OID，并根据需要进行分支。例如，如果您的组织分配了OID `1.1`，则可以按如下方式分支树：

**表8.2：OID层次结构示例**

| **OID**   | **Assignment**     |
| --------- | ------------------ |
| 1.1       | Organization's OID |
| 1.1.1     | SNMP Elements      |
| 1.1.2     | LDAP Elements      |
| 1.1.2.1   | AttributeTypes     |
| 1.1.2.1.1 | x-my-Attribute     |
| 1.1.2.2   | ObjectClasses      |
| 1.1.2.2.1 | x-my-ObjectClass   |

当然，您可以自由地设计适合您组织OID下的组织需求的层次结构。不管你选择什么层次结构，你应该保留一个你所做的作业的注册表。这可以是一个简单的平面文件或更复杂的一些，例如*OpenLDAP OID Registry*（<http://www.openldap.org/faq/index.cgi?file=197>）。

有关对象标识符（以及列表服务）的更多信息，请参见<http://www.alvestrand.no/objectid/>。


为了获得注册的OID在*没有成本*，申请下OID [互联网编号分配机构](http://www.iana.org/)（ORG：IANA）维护*民营企业*的弧线。任何私营企业（组织）可以要求私营企业号（PEN）被分配在这个弧。只需填写IANA表单，[网址](http://pen.iana.org/pen/PenApplication.page)为<http://pen.iana.org/pen/PenApplication.page>，您的官方笔会通常在几天内发送给您。您的基地OID将像`1.3.6.1.4.1.X`这样的`X`，其中`X`是一个整数。

注意：使用此表单获取的PEN可用于任何目的，包括识别LDAP模式元素。

或者，OID名称空间可以从国家机构获得（例如，[ANSI](http://www.ansi.org/)，[BSI](http://www.bsi-global.com/)）。

#### 13.2.2 命名元素

除了为每个架构元素分配唯一的对标识符之外，还应为每个元素提供至少一个文本名称。名称应该向[IANA](http://www.iana.org/)注册，或者以“x-”为前缀，放在“私人使用”的名称空间中。

该名称应该是描述性的，不可能与其他模式元素的名称冲突。特别地，您选择的任何名称不应与现在或将来的标准曲目名称冲突（如果您以“x-”开头注册名称或使用名称），则可以保证。

请注意，您可以获得自己的注册名称前缀，以避免不必单独注册您的姓名。有关详细信息，请参阅[RFC4520](http://www.rfc-editor.org/rfc/rfc4520.txt)。

在下面的例子中，我们使用了一个短的前缀' `x-my-` '。这样一个短的前缀只适合于一个非常大的全球组织。一般来说，我们建议像“ `x-de-Firm-` ”（德国公司）或“ `x-com-Example` ”（与`example.com`相关的组织相关的元素）。

#### 13.2.3 本地架构文件

该`对象类`和`attributeTypes`配置文件指令可以被用来在目录中的条目定义模式规则。通常要创建一个文件来包含自定义模式项的定义。我们建议您在`/usr/local/etc/openldap/schema/local.schema中`创建一个文件`local.schema`，然后在其他模式`include`指令之后立即将此文件包含在*slapd.conf*（5）文件中。

```
# include schema
include /usr/local/etc/openldap/schema/core.schema
include /usr/local/etc/openldap/schema/cosine.schema
include /usr/local/etc/openldap/schema/inetorgperson.schema
# include local schema
include /usr/local/etc/openldap/schema/local.schema
```

#### 13.2.4 属性类型规范

该*属性类型*指令用于定义一个新的属性类型。该伪指令使用在子模式子条目中找到的attributeTypes属性使用的相同属性类型说明（如[RFC4512中](http://www.rfc-editor.org/rfc/rfc4512.txt)定义），例如：

```
attributetype <RFC4512 Attribute Type Description>
```

其中属性类型描述由以下定义 ABNF：

```
AttributeTypeDescription = "(" whsp
      numericoid whsp              ; AttributeType identifier
    [ "NAME" qdescrs ]             ; name used in AttributeType
    [ "DESC" qdstring ]            ; description
    [ "OBSOLETE" whsp ]
    [ "SUP" woid ]                 ; derived from this other
                                   ; AttributeType
    [ "EQUALITY" woid              ; Matching Rule name
    [ "ORDERING" woid              ; Matching Rule name
    [ "SUBSTR" woid ]              ; Matching Rule name
    [ "SYNTAX" whsp noidlen whsp ] ; Syntax OID
    [ "SINGLE-VALUE" whsp ]        ; default multi-valued
    [ "COLLECTIVE" whsp ]          ; default not collective
    [ "NO-USER-MODIFICATION" whsp ]; default user modifiable
    [ "USAGE" whsp AttributeUsage ]; default userApplications
    whsp ")"

AttributeUsage =
    "userApplications"     /
    "directoryOperation"   /
    "distributedOperation" / ; DSA-shared
    "dSAOperation"          ; DSA-specific, value depends on server
```

其中whsp是空格（' ``'），numericoid是点分十进制形式的全局唯一OID（例如`1.1.0`），qdescrs是一个或多个名称，wid是名称或OID，可选地后跟一个长度说明符（例如`{10` }）。

例如，属性类型`名称`和`cn`在`core.schema`中定义为：

```
attributeType ( 2.5.4.41 NAME 'name'
        DESC 'name(s) associated with the object'
        EQUALITY caseIgnoreMatch
        SUBSTR caseIgnoreSubstringsMatch
        SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{32768} )
attributeType ( 2.5.4.3 NAME ( 'cn' 'commonName' )
        DESC 'common name(s) associated with the object'
        SUP name )
```

请注意，每个定义属性的OID，提供一个简短的名称和一个简短的描述。每个名称都是OID的别名。返回结果时，*slapd*（8）返回第一个列出的名称。

第一个属性`name名称`包含`directoryString`（UTF-8编码Unicode）语法。语法由OID指定（1.3.6.1.4.1.1466.115.121.1.15标识directoryString语法）。指定了32768的长度建议。服务器应支持此长度的值，但可能支持更长的值。该字段不指定大小约束，因此在不强加这种大小限制的服务器（例如slapd）上被忽略。此外，相等和子串匹配使用案例忽略规则。以下是列出常用语法和匹配规则的表（*slapd*（8）支持这些和更多）。









## 15 使用SASL

OpenLDAP客户端和服务器能够通过认证 简单的认证和安全层 （SASL）框架，详见[RFC4422](http://www.rfc-editor.org/rfc/rfc4422.txt)。本章介绍如何在OpenLDAP中使用SASL。

有几种可以与SASL一起使用的行业标准认证机制，包括 GSSAPI 对于 Kerberos V， DIGEST-MD5，和 PLAIN 和 EXTERNAL 用于 传输层安全 （TLS）。

OpenLDAP软件提供的标准客户端工具（如*ldapsearch*（1）和*ldapmodify*（1））将默认尝试将用户认证为LDAP目录服务器使用SASL。基本身份验证服务可以由LDAP管理员设置几个步骤，允许用户对slapd服务器作为其LDAP条目进行身份验证。通过几个额外的步骤，可以允许一些用户和服务利用SASL的代理授权功能，允许他们自己进行身份验证，然后将其身份切换到另一个用户或服务的身份。

本章假设您已阅读*Cyrus SASL（系统管理员）*，并附带[Cyrus SASL](http://asg.web.cmu.edu/sasl/sasl-library.html)软件包（`doc / sysadmin.html`），并安装了Cyrus SASL。您应该使用Cyrus SASL `sample_client`和`sample_server`来测试您的SASL安装，然后再尝试使用OpenLDAP软件。

请注意，在以下文本中，术语*用户*用于描述通过LDAP客户端（如*ldapsearch*（1））连接到LDAP服务器的人员或应用程序实体。也就是说，术语*用户*不仅适用于使用LDAP客户端的个人，而且适用于在没有直接用户控制的情况下发出LDAP客户端操作的应用程序实体。例如，使用LDAP操作访问LDAP服务器中保存的信息的电子邮件服务器是应用程序实体。

### 15.1 SASL安全注意事项

SASL提供了许多不同的认证机制。本节简要概述了安全考虑。

一些机制，例如PLAIN和LOGIN，提供比LDAP *简单*认证更大的安全性。像LDAP *简单*认证一样，除非您有足够的安全保护，否则不应使用此类机制。建议这些机制只能与之配合使用传输层安全（TLS）。本文档不再详细讨论PLAIN和LOGIN的使用。

DIGEST-MD5机制是LDAPv3的强制实施认证机制。虽然与受信任的第三方认证系统相比，DIGEST-MD5不是一种强大的认证机制Kerberos的或公共密钥系统），它确实提供了对许多攻击的重大保护。不一样CRAM-MD5机制，防止选择明文攻击。DIGEST-MD5优于使用明文密码机制。CRAM-MD5机制不赞成使用DIGEST-MD5。下面讨论[DIGEST-MD5的](http://www.openldap.org/doc/admin24/sasl.html#DIGEST-MD5)使用。

GSSAPI机制使用 GSS-API Kerberos的V提供安全认证服务。KERBEROS_V4机制适用于使用Kerberos IV的机构。Kerberos被视为适合小型和大型企业的安全的分布式认证系统。下面讨论[GSSAPI](http://www.openldap.org/doc/admin24/sasl.html#GSSAPI)和[KERBEROS_V4](http://www.openldap.org/doc/admin24/sasl.html#KERBEROS_V4)使用。

EXTERNAL机制利用诸如下级网络服务提供的认证服务 传输层安全 （TLS）。与TLS X.509的公钥技术配合使用时，EXTERNAL提供强大的认证。TLS在“ [使用TLS”](http://www.openldap.org/doc/admin24/tls.html)一章中讨论。

EXTERNAL也可以与`ldapi：///`传输一起使用，因为Unix-domain sockets可以报告客户端进程的UID和GID。

还有其他强大的认证机制可供选择，包括 OTP （一次密码）和 SRP（安全远程密码）。这些机制在本文档中没有讨论。

### 15.2 SASL认证

获取基本的SASL认证运行涉及到几个步骤。第一步配置您的slapd服务器环境，以便可以使用您网站上的安全系统与客户端程序进行通信。这通常涉及设置服务密钥，公钥或其他形式的秘密。第二步涉及将身份验证身份映射到LDAPDN这取决于条目在目录中的布局。将在下一节中使用Kerberos V4作为示例机制给出第一步的说明。您站点身份验证机制所需的步骤将是类似的，但SASL可用的每种机制的指南超出了本章的范围。第二步在“ [映射身份验证身份](http://www.openldap.org/doc/admin24/sasl.html#Mapping Authentication Identities) ”一节中描述。

#### 15.2.1 GSSAPI

本节介绍SASL GSSAPI机制和Kerberos V与OpenLDAP的使用。将假设您已部署Kerberos V，您熟悉系统的操作，并且您的用户受到使用的培训。本节还假定您已经熟悉GSSAPI机制的使用，方法是阅读*配置GSSAPI和Cyrus SASL*（`doc / gssapi`文件中的*Cyrus SASL*提供），并成功实验了Cyrus提供的`sample_server`和`sample_client`应用程序。有关Kerberos的一般信息，请访问<http://web.mit.edu/kerberos/www/>。

要使用具有*slapd*（8）的GSSAPI机制，必须在服务运行的主机的领域内为*ldap*服务的主体创建一个服务密钥。例如，如果在`directory.example.com`上运行*slapd*，并且您的领域是`EXAMPLE.COM`，则需要使用主体创建服务密钥：

```
ldap/directory.example.com@EXAMPLE.COM
```

当*slapd*（8）运行时，它必须具有访问该键的权限。这通常通过将密钥放入keytab文件`/etc/krb5.keytab`。有关keytab位置设置的信息，请参阅Kerberos和Cyrus SASL文档。

要使用GSSAPI机制对目录进行身份验证，用户将在运行LDAP客户端之前获取授权单（TGT）。当使用OpenLDAP客户端工具时，用户可以通过将`-Y GSSAPI`指定为命令选项来`强制`使用GSSAPI机制。

为了进行认证和授权，*slapd*（8）将认证请求DN与以下形式相关联：

```
uid=<primary[/instance]>,cn=<realm>,cn=gssapi,cn=auth
```

继续我们的例子，具有Kerberos主体`kurt@EXAMPLE.COM`的用户将具有相关的DN：

```
uid=kurt,cn=example.com,cn=gssapi,cn=auth
```

并且主体`ursula/admin@FOREIGN.REALM`将具有关联的DN：

```
uid=ursula/admin,cn=foreign.realm,cn=gssapi,cn=auth
```

认证请求DN可以直接使用ACL和`groupOfNames` “member”属性，因为它是合法的LDAP DN格式。或者，可以在使用之前映射认证DN。有关详细信息，请参阅[映射身份验证身份](http://www.openldap.org/doc/admin24/sasl.html#Mapping Authentication Identities)部分。

#### 15.2.2 KERBEROS_V4

本节介绍使用SASL KERBEROS_V4机制与OpenLDAP。将假定您熟悉Kerberos IV安全系统的运行情况，并且您的站点部署了Kerberos IV。您的用户应该熟悉身份验证策略，如何在Kerberos票证缓存中接收凭据，以及如何刷新过期凭证。

**注意：** KERBEROS_V4和Kerberos IV不赞成使用GSSAPI和Kerberos V.

客户端程序需要能够获取会话密钥，以便在连接到LDAP服务器时使用。这允许LDAP服务器知道用户的身份，并允许客户端知道它正在连接到合法的服务器。如果要使用加密层，会话密钥也可以用来帮助协商该选项。

slapd服务器运行名为“ *ldap* ” 的服务，服务器将需要具有服务密钥的srvtab文件。SASL感知的客户端程序将获得带有用户票证授予票据（TGT）的“ldap”服务票证，其中票据的实例与OpenLDAP服务器的主机名相匹配。例如，如果您的`域名`为`EXAMPLE.COM`，并且slapd服务器正在名为`directory.example.com`的主机上运行，则服务器上的`/etc/srvtab`文件将具有服务密钥

```
 ldap.directory@EXAMPLE.COM
```

当LDAP客户端使用KERBEROS_IV机制将用户认证到目录时，它将从票据缓存或通过从Kerberos服务器获取一个新请求来为同一主体请求会话密钥。这将需要TGT在缓存中可用和有效。如果它不存在或已经过期，客户端可能打印出消息：

```
ldap_sasl_interactive_bind_s: Local error
```

获得服务票证后，将作为用户身份证明传递给LDAP服务器。服务器将使用SASL库调用从服务票证中提取身份和领域，并将其转换为表单的*身份验证请求DN*

```
uid=<username>,cn=<realm>,cn=<mechanism>,cn=auth
```

所以在上面的例子中，如果用户的名字是“adamson”，认证请求DN将是：

```
uid=adamsom,cn=example.com,cn=kerberos_v4,cn=auth
```

该认证请求DN可以直接使用ACL，也可以在使用前进行映射。有关详细信息，请参阅[映射身份验证身份](http://www.openldap.org/doc/admin24/sasl.html#Mapping Authentication Identities)部分。

#### 15.2.3 DIGEST-MD5

本节介绍使用SASL DIGEST-MD5机制，使用存储在目录本身或Cyrus SASL自己的数据库中的秘密。DIGEST-MD5依赖于客户端和服务器共享“秘密”，通常是密码。服务器产生一个挑战，并且客户端的响应证明它知道共享的秘密。这比通过电线发送秘密更安全。

赛勒斯SASL支持几种共享秘密机制。为此，它需要访问明文密码（不同于通过线路传递明文密码的机制，服务器可以存储散列版本的密码）。

服务器的共享密钥副本可以存储在Cyrus SASL自己的*sasldb*数据库中，在通过*saslauthd*访问的外部系统中或LDAP数据库本身中。在这两种情况下，应用文件访问控制和LDAP访问控制来防止密码暴露是非常重要的。本节讨论的配置和命令假设使用Cyrus SASL 2.1。

要使用存储在*sasldb中的*秘密，只需使用*saslpasswd2*命令添加用户：

```
saslpasswd2 -c <username>
```

必须使用*saslpasswd2*命令管理此类用户的密码。

要使用存储在LDAP目录中的秘密，请将明文密码放在`userPassword`属性中。有必要为`slapd.conf`添加一个选项，以确保使用LDAP密码修改操作设置的密码以明文形式存储：

```
password-hash   {CLEARTEXT}
```

以这种方式存储的密码可以使用*ldappasswd*（1）或简单地修改`userPassword`属性进行管理。无论存储密码在哪里，都需要从认证请求DN到用户DN的映射。

DIGEST-MD5机制产生以下形式的身份验证ID：

```
uid=<username>,cn=<realm>,cn=digest-md5,cn=auth
```

如果使用默认领域，则从ID中省略领域名称，给出：

```
uid=<username>,cn=digest-md5,cn=auth
```

有关可选的身份映射的信息，请参阅下面的[映射身份验证](http://www.openldap.org/doc/admin24/sasl.html#Mapping Authentication Identities)身份。

使用合适的映射，用户可以在执行LDAP操作时指定SASL ID，并且将使用存储在*sasldb*或目录本身中的密码来验证身份验证。例如，由目录条目标识的用户：

```
dn: cn=Andrew Findlay+uid=u000997,dc=example,dc=com
objectclass: inetOrgPerson
objectclass: person
sn: Findlay
uid: u000997
userPassword: secret
```

**注意：**在上述每种情况下，都没有提供授权身份（例如`-X`）。除非您正在尝试[SASL代理授权](http://www.openldap.org/doc/admin24/sasl.html#SASL Proxy Authorization)，否则不应指定授权身份。服务器将从认证身份推断授权身份（如下所述）。

#### 15.2.4 EXTERNAL

SASL EXTERNAL机制利用较低级协议进行的认证：通常 TLS 或Unix IPC

每个传输协议以其自己的格式返回验证身份：

##### 15.2.4.1 TLS认证身份格式

这是客户端证书的主题DN。请注意，DN由LDAP和X.509显示不同，因此颁发了证书

```
C=gb, O=The Example Organisation, CN=A Person
```

将产生以下认证身份：

```
cn=A Person,o=The Example Organisation,c=gb
```

请注意，您必须为TLSVerifyClient设置合适的值，以使服务器请求使用客户端证书。没有这个，SASL EXTERNAL机制将不会被提供。有关详细信息，请参阅[使用TLS](http://www.openldap.org/doc/admin24/tls.html)一章。

##### 15.2.4.2 IPC (ldapi:///) Identity Format

这是由客户端进程的Unix UID和GID组成的：

```
gidNumber=<number>+uidNumber=<number>,cn=peercred,cn=external,cn=auth
```

因此，以`root`身份运行的客户端进程将是：

```
gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
```

#### 15.2.5 映射身份验证身份

slapd服务器中的认证机制将使用SASL库调用来获取经过身份验证的用户的“用户名”，这取决于所使用的基本身份验证机制。该用户名位于身份验证机制的命名空间中，而不在正常的LDAP命名空间中。如上述部分所述，该用户名重新格式化为表单的认证请求DN

```
uid=<username>,cn=<realm>,cn=<mechanism>,cn=auth
```

或

```
uid=<username>,cn=<mechanism>,cn=auth
```

这取决于\<mechanism\>是否采用“realms”的概念。还要注意，如果认证中使用了默认领域，则会忽略该领域部分。

所述*ldapwhoami命令*（1）命令可以被用来确定与用户相关联的身份。确定映射的正确功能非常有用。

您不应该将上述表单的LDAP条目添加到LDAP数据库。您可能有机会为LDAP中的每个人员提供一个LDAP条目，并将其放置在目录树中，树不会从cn = auth开始。但是，如果您的站点在该用户的“用户名”和LDAP条目之间有明确的映射，则可以配置LDAP服务器以自动将认证请求DN映射到用户的*身份验证DN*。

**注意：**不要求认证请求DN和映射导致的用户认证DN指的是目录中保存的条目。但是，可以使用其他功能（见下文）。

LDAP管理员需要告诉slapd服务器如何将身份验证请求DN映射到用户的身份验证DN。这通过在*slapd.conf*（5）文件中添加一个或多个`authz-regexp`指令来完成。这个指令有两个参数：**

```
authz-regexp <搜索模式> <替换模式>
```

使用正则表达式函数*regcomp*（）和*regexec*（）将认证请求DN与搜索模式进行比较，如果匹配，则将其重写为替换模式。如果存在多个`authz-regexp`指令，则只使用其搜索模式与身份验证身份匹配的第一个。从替换模式输出的字符串应该是用户的认证DN或LDAP URL。如果替换字符串产生DN，则此DN命名的条目不需要由此服务器持有。如果替换字符串生成LDAP URL，则该LDAP URL必须评估为此服务器所保留的唯一一个条目。

搜索模式可以包含*regexec*（3C）中列出的任何正则表达式字符。注意的主要特征是点“”，星号“*”和开放的圆括号“（”和“）”。本质上，点匹配任何字符，星号允许紧接在前一个字符或模式的零个或多个重复，括号中的术语被记住替换模式。

替换模式将产生指向用户的DN或URL。与搜索模式中的括号中的字符串匹配的认证请求DN中的任何内容都存储在变量“\$ 1”中。该变量“$ 1”可以出现在替换模式中，并将被来自认证请求DN的字符串替换。如果搜索模式中有多组括号，则使用变量$ 2，$ 3等。

#### 15.2.6 直接映射

在可能的情况下，通常建议将认证请求DN直接映射到用户的DN。除了避免搜索用户DN的费用外，还允许将映射到指向不属于此服务器的条目的DN。

假设认证请求DN被写为：

```
uid=adamson,cn=example.com,cn=gssapi,cn=auth
```

用户的实际LDAP条目为：

```
uid=adamson,ou=people,dc=example,dc=com
```

那么*slapd.conf*（5）中的以下`authz-regexp`指令将提供直接映射。

```
authz-regexp
  uid=([^,]*),cn=example.com,cn=gssapi,cn=auth
  uid=$1,ou=people,dc=example,dc=com
```

一个更宽松的规则可以写成

```
authz-regexp
  uid=([^,]*),cn=[^,]*,cn=auth
  uid=$1,ou=people,dc=example,dc=com
```

但是，请仔细设定搜索模式，因为它可能会错误地允许人们认证为不能访问的DN。写一些严格的指令比一个具有安全漏洞的宽松指令更好。如果您的站点上只有一个身份验证机制，以及零个或一个使用的领域，则可以使用单个`authz-regexp`指令在身份验证身份和LDAP DN之间进行映射。

不要忘记允许省略领域的情况以及具有明确指定领域的情况。对于每种情况，这可能需要单独的`authz-regexp`指令，其中首先列出显式领域条目。

#### 15.2.7 基于搜索的映射

有一些情况下映射到LDAP URL可能是适当的。例如，一些站点可能具有位于LDAP树的多个区域中的人物对象，例如，如果存在`ou=accounting`树和`ou=engineering`树，并且人物之间散布着它们。或者，也许所需的映射必须基于用户信息中的信息。考虑需要将上述认证请求DN映射到其条目如下的用户：

```
dn: cn=Mark Adamson,ou=People,dc=Example,dc=COM
objectclass: person
cn: Mark Adamson
uid: adamson
```

认证请求DN中的信息不足以允许直接导出用户的DN，而是必须搜索用户的DN。对于这些情况，可以在`authz-regexp`指令中使用生成LDAP URL的替换模式。然后，此URL将用于执行LDAP数据库的内部搜索以查找该人的身份验证DN。

与其他网址相似的LDAP网址为形式

```
ldap://<host>/<base>?<attrs>?<scope>?<filter>
```

这包含执行LDAP搜索所需的所有元素：服务器的名称\<host\>，LDAP DN搜索库\<base\>，要检索的LDAP属性\<attrs\>，搜索范围\<scope\>是三个选项“base”，“one”或“sub”，最后是LDAP搜索过滤器\<filter\>。由于搜索是当前服务器中的LDAP DN，所以\<host\>部分应为空。\<attrs\>字段也被忽略，因为只有DN值得关注。这两个元素以URL的格式保留，以保持字符串中哪些信息的清晰度。

假设上述示例中的人实际上具有“adamson”的身份验证用户名，并且该信息保存在其LDAP条目的属性“uid”中。在`authz-regexp`指令可以写成

```
authz-regexp
  uid=([^,]*),cn=example.com,cn=gssapi,cn=auth
  ldap:///ou=people,dc=example,dc=com??one?(uid=$1)
```

这将启动对slapd服务器内的LDAP数据库的内部搜索。如果搜索只返回一个条目，则它被接受为用户的DN。如果返回多个条目，或者返回零条目，则认证失败，并将用户的连接作为认证请求DN绑定。

在URL中的搜索过滤器<filter>中使用的属性应该被编入索引，以便更快地搜索。如果不是，认证步骤单独可能会持续很长时间，用户可能会认为服务器关闭。

一个更复杂的站点可能有几个领域的使用，每个映射到目录中的不同的子树。这些可以用以下形式的语句处理：

```
# Match Engineering realm
authz-regexp
   uid=([^,]*),cn=engineering.example.com,cn=digest-md5,cn=auth
   ldap:///dc=eng,dc=example,dc=com??one?(&(uid=$1)(objectClass=person))

# Match Accounting realm
authz-regexp
   uid=([^,].*),cn=accounting.example.com,cn=digest-md5,cn=auth
   ldap:///dc=accounting,dc=example,dc=com??one?(&(uid=$1)(objectClass=person))

# Default realm is customers.example.com
authz-regexp
   uid=([^,]*),cn=digest-md5,cn=auth
   ldap:///dc=customers,dc=example,dc=com??one?(&(uid=$1)(objectClass=person))
```

请注意，首先处理明确命名的域，以避免域名成为UID的一部分。还要注意使用范围和过滤器来限制匹配到所需条目。

请注意，`authz-regexp`内部搜索需要访问控制。具体来说，认证身份必须具有`auth`访问权限。

有关更多详细信息，请参阅*slapd.conf*（5）。

### 15.3 SASL代理授权

SASL提供了一种被称为*代理授权*的功能，允许经过身份验证的用户请求他们代表另一个用户行事。此步骤发生在用户获得认证DN之后，并涉及向服务器发送授权标识。然后，服务器将决定是否允许发生授权。如果允许，则将用户的LDAP连接切换为具有从授权标识导出的绑定DN，并且LDAP会话继续访问新的授权DN。

允许进行授权的决定取决于运行LDAP的站点的规则和策略，因此不能由SASL单独进行。SASL库可以让服务器做出决定。LDAP管理员通过向LDAP数据库条目中添加信息来设置谁可以授权哪些身份的准则。默认情况下，授权功能被禁用，使用前必须由LDAP管理员明确配置。

#### 15.3.1 代理授权的使用

当一个实体需要代表许多其他用户行事时，这种服务是有用的。例如，用户可能被引导到网页以在其LDAP条目中对其个人信息进行更改。用户向Web服务器进行身份验证以建立其身份，但是由于该用户对其进行更改，因此，Web服务器CGI无法向LDAP服务器进行身份验证。相反，Web服务器将其自身认证为LDAP服务器作为服务标识，比如说，

```
cn=WebUpdate,dc=example,dc=com
```

然后将SASL授权给用户的DN。一旦这样授权，CGI会更改用户的LDAP条目，只要slapd服务器能够识别其ACL，则用户本身就在连接的另一端。用户可以直接连接到LDAP服务器，并自己认证，但这需要用户更多地了解LDAP客户端，网页提供的知识更容易。

还可以使用代理授权来限制对具有对数据库的更大访问权限的帐户的访问。这样的帐户，甚至是*slapd.conf*（5）中指定的根DN ，可以有一个严格的可以授权给该DN的人员列表。那么LDAP数据库的更改只能由该DN允许，并且为了成为该DN，用户必须首先认证为列表中的人员之一。这样可以更好地审核谁对LDAP数据库进行了更改。如果允许人们通过`rootpw `*slapd.conf*（5）指令或通过`userPassword`属性直接向特权帐户进行身份验证，则审核变得更加困难。

请注意，在成功的代理授权之后，LDAP连接的原始身份验证DN将被授权请求中的新DN覆盖。如果服务程序能够自己认证为自己的认证DN，然后授权给其他DN，并且计划在一个LDAP会话期间切换到几个不同的身份，则每次在授权给另一个DN之前需要对其进行身份验证（或使用不同的代理授权机制）。slapd服务器不记录服务程序切换到其他DN的能力。在Kerberos这样的认证机制上，由于用户的TGT和“ldap”

#### 15.3.2 SASL授权身份

SASL授权身份通过*ldapsearch*（1）的`-X`开关和其他工具或在*lutil_sasl_defaults*（）调用的`* authzid`参数中发送到LDAP服务器。身份可以是两种形式之一

```
u:<username>
```

或

```
dn:<dn>
```

在第一种形式中，<username>来自与上述身份验证身份相同的命名空间。它是由底层身份验证机制引用的用户的用户名。该表单的授权身份通过与认证过程相同的功能转换成DN格式，生成表单的*授权请求DN*

```
uid=<username>,cn=<realm>,cn=<mechanism>,cn=auth
```

然后，该授权请求DN通过相同的`authz-regexp`进程运行，以将其转换为数据库中的合法授权DN。如果由于LDAP URL中的搜索失败而无法转换，则授权请求将失败，并显示“不当访问”。否则，DN字符串现在是准备好批准的合法授权DN。

如果以第二种形式提供授权身份，则使用`“dn：”`前缀，前缀后面的字符串已经处于授权DN形式，准备进行批准。

#### 15.3.3 代理授权规则

一旦slapd具有授权DN，则开始实际的批准过程。LDAP管理员可以将两个属性放入LDAP条目以允许授权：

```
authzTo
authzFrom
```

两者都可以是多值的。该`authzTo`属性是源规则，将其置入与认证DN告诉DNS的认证DN被允许什么授权承担相关的条目。第二个属性是目标规则，它被放置到与请求的授权DN相关联的条目中，以告知哪些已验证的DN可以承担它。

选择要使用的授权策略属性取决于管理员。源代码规则首先在人员的身份验证DN条目中检查，并且如果`authzTo`规则中没有一个指定允许授权，则会检查授权DN条目中的`authzFrom`规则。如果两个案例都不指定请求被遵守，则该请求被拒绝。由于默认行为是拒绝授权请求，因此规则只指定允许请求; 没有消极的规则告诉什么授权拒绝。

两个属性中的值与`authz-regexp`伪指令的替换模式的输出形式相同：DN或LDAP URL。例如，如果`authzTo`值是DN，则该DN是经过身份验证的用户可以授权的DN。另一方面，如果`authzTo`值是LDAP URL，则该URL将用作LDAP数据库的内部搜索，并且已验证的用户可以成为搜索返回的任何DN。如果LDAP条目看起来像：

```
dn: cn=WebUpdate,dc=example,dc=com
authzTo: ldap:///dc=example,dc=com??sub?(objectclass=person)
```

那么认证为`cn = WebUpdate，dc = example，dc = com的`任何用户可以授权给具有objectClass of `Person`的搜索库`dc = example，dc = com`下的任何其他LDAP条目。``

##### 15.3.3.1 代理授权规则说明

`authzTo`或`authzFrom`属性中的LDAP URL 将返回一组DN。每个返回的DN将被检查。返回一个大集合的搜索可能会导致授权过程变得不舒服很长时间。此外，应该对由slapd建立索引的属性执行搜索。

为了帮助为`authzFrom`和`authzTo`生成更多清晰的规则，这些属性的值可以是其中具有正则表达式字符的DN。这意味着一个源规则

```
authzTo: dn.regex:^uid=[^,]*,dc=example,dc=com$
```

将允许经过身份验证的用户授权给与给定的正则表达式模式匹配的任何DN。这种正则表达式比较可以比LDAP搜索`（uid = *）`快得多。

另请注意，授权规则中的值必须是以下两种形式之一：LDAP URL或DN（带或不带正则表达式字符）。任何不以“ `ldap：//` ” 开头的内容都将被视为DN。不允许输入形式“ `u：<username>` ”的另一个授权身份作为授权规则。

##### 15.3.3.2 策略配置

决定使用哪种类型的规则，`authzFrom`或`authzTo`将取决于网站的情况。例如，如果可以成为给定身份的一组人可以容易地被写为搜索过滤器，则可以写入单个目的地规则。如果一组人不容易被搜索过滤器定义，而且一组人很小，可能会更好的是在每个被允许执行代理授权的人的条目中写入源规则。

默认情况下，禁用代理授权规则的处理。在`AuthZ的政策`指令必须在设定*的slapd.conf*（5）文件来启用授权。该指令可以设置为`无`对无规则（默认），`以`供源的规则，`从`为目的地的规则，或者`两者`的源和目标的规则。

源规则非常强大。如果普通用户有权在自己的条目中写入`authzTo`属性，那么他们可以编写允许他们作为其他人授权的规则。因此，当使用源规则时，`authzTo`属性应该使用只允许特权用户设置其值的ACL进行保护。



## 20 监测

*slapd*（8）支持可选LDAP监视界面，您可以使用它来获取有关*slapd*实例当前状态的信息。例如，该界面允许您确定当前有多少客户端连接到服务器。监控信息由专门的后端（*监视器*后端）提供。手册页，*slapd-monitor*（5）可用。

启用监视界面后，LDAP客户端可能被用于访问由*监视器*后端提供的信息，受访问和其他控制。

启用后，*监视器*后端将动态生成并返回对象，以响应 *cn=monitor* 子树中的搜索请求。每个对象包含有关服务器特定方面的信息。信息以用户应用程序和操作属性的组合方式进行。该信息可以使用*ldapsearch（1）*，任何通用LDAP浏览器或专门的监控工具进行访问。“ [访问监控信息”](https://www.openldap.org/doc/admin24/monitoringslapd.html#Accessing Monitoring Information)部分提供了有关如何使用*ldapsearch*（1）访问监视信息的简要教程监控信息 监控信息基础及其组织部分细节。

虽然支持监视器后端包含在slapd（8）的默认版本中，但该支持需要一些配置才能激活。这可以使用`cn=config`或 *slapd.conf*（5）完成。前者在中讨论过通过 cn=config 监视配置本章的一部分。后者[通过](https://www.openldap.org/doc/admin24/monitoringslapd.html#Monitor configuration via slapd.conf(5))本章的[slapd.conf（5）](https://www.openldap.org/doc/admin24/monitoringslapd.html#Monitor configuration via slapd.conf(5))部分在[Monitor配置中](https://www.openldap.org/doc/admin24/monitoringslapd.html#Monitor configuration via slapd.conf(5))讨论。这些部分假设监视器后端内置于*slapd中*（例如，`enable-monitor=yes` ，默认值）。如果监视器后端被构建为一个模块（例如，`enable-monitor=mod）`，则该模块必须加载。在“ [配置slapd](https://www.openldap.org/doc/admin24/slapdconf2.html)和[slapd配置文件”](https://www.openldap.org/doc/admin24/slapdconfig.html)章节中讨论了模块的加载。

### 20.1 通过cn=config(5)监控配置

*本节尚未写入。*



### 20.2 通过slapd.conf(5)监视配置

配置slapd.conf（5）来支持LDAP监控非常简单。

首先，确保您的*slapd.conf*（5）文件包含*core.schema*模式配置文件。该*显示器* 后端需要它。

其次，通过在现有数据库部分下方添加 *数据库监视器 *指令来实例化*监视器后端。例如：

```
database monitor
```

最后，根据需要添加其他全局或数据库指令。

像大多数其他数据库后端一样，监视器后端执行slapd（8）访问和其他管理控制。由于某些监视器信息可能很敏感，因此通常建议对 cn=monitor 进行访问，限于目录管理员及其监控代理。在*数据库监视器*指令下面添加*访问*指令是控制访问的一种清晰有效的方法。例如，在*数据库监视器*指令正下方添加以下*访问*指令将对监视信息的访问限制到指定的目录管理器。

```
access to *
        by dn.exact="cn=Manager,dc=example,dc=com
        by * none
```

有关*slapd*（8）访问控制的更多信息，请参阅[“slapd配置文件”](https://www.openldap.org/doc/admin24/slapdconfig.html)一章*的访问控制指令*部分和*slapd.access*（5）。

重新启动*slapd*（8）后，您可以开始探索`cn=config`中提供的监视信息，如本章“ [访问监控信息”](https://www.openldap.org/doc/admin24/monitoringslapd.html#Accessing Monitoring Information)部分所述。

可以验证slapd（8）是否被正确地配置为通过尝试读取`cn=monitor`对象来提供监视信息。例如，如果以下*ldapsearch*（1）命令返回cn=monitor对象（根据请求，没有属性），它正在运行。

```
ldapsearch -x -D 'cn=Manager,dc=example,dc=com' -W \
        -b 'cn=Monitor' -s base 1.1
```

请注意，与通用数据库后端不同，数据库后缀是硬编码的。它总是`cn =监视器`。所以不应该提供*后缀*指令。另请注意，通用数据库后端，显示器后端无法多次实例化。也就是说，在服务器配置中只能有一个（或零）`数据库监视器`出现。

### 20.3 访问监控信息

如前所述，当启用时，*监视器*后端将动态生成并响应*cn =监视器*子树中的搜索请求返回对象。每个对象包含有关服务器特定方面的信息。信息以用户应用程序和操作属性的组合方式进行。可以使用*ldapsearch（1）*，任何通用LDAP浏览器或专门的监视工具访问此信息。

本节提供了一个关于如何使用*ldapsearch*（1）访问监视信息的简要教程。

要检查任何特定的监视器对象，可以使用一个baseObject范围和一个`（objectClass = *）`过滤器对对象执行搜索操作。由于监视信息包含在用户应用程序和操作属性的组合中，所以应该要求返回所有用户应用程序属性（例如`'*'`）和所有操作属性（例如`'+'`）。例如，要读取`cn = Monitor`对象本身，可以使用*ldapsearch*（1）命令（修改为适合您的配置）：

```
ldapsearch -x -D 'cn=Manager,dc=example,dc=com' -W \
        -b 'cn=Monitor' -s base '(objectClass=*)' '*' '+'
```

当您的服务器运行时，应该产生类似于以下的输出：

```
dn: cn=Monitor
objectClass: monitorServer
structuralObjectClass: monitorServer
cn: Monitor
creatorsName:
modifiersName:
createTimestamp: 20061208223558Z
modifyTimestamp: 20061208223558Z
description: This subtree contains monitoring/managing objects.
description: This object contains information about this server.
description: Most of the information is held in operational attributes, which
 must be explicitly requested.
monitoredInfo: OpenLDAP: slapd 2.4 (Dec  7 2006 17:30:29)
entryDN: cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: TRUE
```

为了减少返回的不感兴趣的属性数量，在请求哪些属性要返回时，可以更有选择性。例如，可以请求返回由*monitorServer*对象类（例如`@objectClass`）允许的所有属性，而不是所有用户和所有操作属性：

```
ldapsearch -x -D 'cn=Manager,dc=example,dc=com' -W \
        -b 'cn=Monitor' -s base '(objectClass=*)' '@monitorServer'
```

这将限制输出如下：

```
dn: cn=Monitor
objectClass: monitorServer
cn: Monitor
description: This subtree contains monitoring/managing objects.
description: This object contains information about this server.
description: Most of the information is held in operational attributes, which
 must be explicitly requested.
monitoredInfo: OpenLDAP: slapd 2.X (Dec  7 2006 17:30:29)
```

要返回所有监控对象的名称，可以使用子树范围和`（objectClass = *）`过滤器执行`cn = Monitor`的搜索，并且不要求返回任何属性（例如，`1.1`）。````

```
ldapsearch -x -D 'cn=Manager,dc=example,dc=com' -W -b 'cn=Monitor' -s sub 1.1
```

如果运行此命令，您会发现*cn = Monitor*子树中有很多对象。以下部分介绍了一些常用的监视对象。

### 20.4 监控信息

所述*监视器*后端提供了丰富的用于监控在设置中的显示屏的对象的slapd（8）的有用信息。每个对象包含有关服务器特定方面的信息，例如后端，连接或线程。一些对象用作其他对象的容器，用于构造对象的层次结构。

在这个层次结构中，最高级的对象是{cn = Monitor}。虽然此对象主要用作其他对象的容器，其中大多数是容器，但此对象提供有关此服务器的信息。特别是它提供了slapd（8）版本字符串。例：

```
dn: cn=Monitor
monitoredInfo: OpenLDAP: slapd 2.X (Dec  7 2006 17:30:29)
```

**注意：**本节（及其子部分）中的示例已被修剪以仅显示关键信息。

#### 20.4.1 后端

该`cn=Backends`，`cn=Monitor `对象，本身提供了可用后端的列表。所有内置后端的可用后端列表以及模块加载的后端。例如：

```
dn: cn=Backends,cn=Monitor
monitoredInfo: config
monitoredInfo: ldif
monitoredInfo: monitor
monitoredInfo: bdb
monitoredInfo: hdb
```

这表示*config*，*ldif*，*monitor*，*bdb*和*hdb*后端可用。

`cn=Backends`，` cn=Monitor`对象也是可用的后端对象的容器。每个可用的后端对象包含有关特定后端的信息。例如：

```
dn: cn=Backend 0,cn=Backends,cn=Monitor
monitoredInfo: config
monitorRuntimeConfig: TRUE
supportedControl: 2.16.840.1.113730.3.4.2
seeAlso: cn=Database 0,cn=Databases,cn=Monitor

dn: cn=Backend 1,cn=Backends,cn=Monitor
monitoredInfo: ldif
monitorRuntimeConfig: TRUE
supportedControl: 2.16.840.1.113730.3.4.2

dn: cn=Backend 2,cn=Backends,cn=Monitor
monitoredInfo: monitor
monitorRuntimeConfig: TRUE
supportedControl: 2.16.840.1.113730.3.4.2
seeAlso: cn=Database 2,cn=Databases,cn=Monitor

dn: cn=Backend 3,cn=Backends,cn=Monitor
monitoredInfo: bdb
monitorRuntimeConfig: TRUE
supportedControl: 1.3.6.1.1.12
supportedControl: 2.16.840.1.113730.3.4.2
supportedControl: 1.3.6.1.4.1.4203.666.5.2
supportedControl: 1.2.840.113556.1.4.319
supportedControl: 1.3.6.1.1.13.1
supportedControl: 1.3.6.1.1.13.2
supportedControl: 1.3.6.1.4.1.4203.1.10.1
supportedControl: 1.2.840.113556.1.4.1413
supportedControl: 1.3.6.1.4.1.4203.666.11.7.2
seeAlso: cn=Database 1,cn=Databases,cn=Monitor

dn: cn=Backend 4,cn=Backends,cn=Monitor
monitoredInfo: hdb
monitorRuntimeConfig: TRUE
supportedControl: 1.3.6.1.1.12
supportedControl: 2.16.840.1.113730.3.4.2
supportedControl: 1.3.6.1.4.1.4203.666.5.2
supportedControl: 1.2.840.113556.1.4.319
supportedControl: 1.3.6.1.1.13.1
supportedControl: 1.3.6.1.1.13.2
supportedControl: 1.3.6.1.4.1.4203.1.10.1
supportedControl: 1.2.840.113556.1.4.1413
supportedControl: 1.3.6.1.4.1.4203.666.11.7.2
```

对于每个这些对象，monitorInfo指示对象中的哪个后端信息是关于的。例如，`cn=Backend 3`，`cn=Backends`，`cn=Monitor`对象包含（在示例中）有关*bdb*后端的信息。

| **属性**           | **描述**       |
| ---------------- | ------------ |
| monitoredInfo    | 后端名称         |
| supportedControl | 支持LDAP控制扩展   |
| seeAlso          | 这个后端实例的数据库对象 |

#### 20.4.2 Connections

主条目为空; 它应该包含一些关于连接数的统计信息。

为每个打开的连接创建动态子条目，并对该连接上的活动进行统计（稍后将对其进行详细说明）。有两个特殊的子条目分别显示总数和当前连接的数量。

例如：

总连接数：

```
dn: cn=Total,cn=Connections,cn=Monitor
structuralObjectClass: monitorCounterObject
monitorCounter: 4
entryDN: cn=Total,cn=Connections,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

当前连接：

```
dn: cn=Current,cn=Connections,cn=Monitor
structuralObjectClass: monitorCounterObject
monitorCounter: 2
entryDN: cn=Current,cn=Connections,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

#### 20.4.3 Databases

主条目包含每个配置的数据库的命名上下文; 对于每个数据库，子条目包含类型和命名上下文。

例如：

```
dn: cn=Database 2,cn=Databases,cn=Monitor
structuralObjectClass: monitoredObject
monitoredInfo: monitor
monitorIsShadow: FALSE
monitorContext: cn=Monitor
readOnly: FALSE
entryDN: cn=Database 2,cn=Databases,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

#### 20.4.4 Listener

它包含服务器当前正在侦听的设备的描述：

```
dn: cn=Listener 0,cn=Listeners,cn=Monitor
structuralObjectClass: monitoredObject
monitorConnectionLocalAddress: IP=0.0.0.0:389
entryDN: cn=Listener 0,cn=Listeners,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

#### 20.4.5 Log

它包含当前活动的日志项目。该*日志*子系统允许用户修改的操作*描述*属性，其值*必须*在admittable日志切换列表：

```
Trace
Packets
Args
Conns
BER
Filter
Config
ACL
Stats
Stats2
Shell
Parse
Sync
```

这些值可以添加，替换或删除; 它们会影响发送到syslog设备的消息。可以通过自定义模块添加自定义值。

#### 20.4.6 Operations

它显示了服务器执行的操作的一些统计信息：

```
Initiated
Completed
```

对于每个操作类型，即：

```
Bind
Unbind
Add
Delete
Modrdn
Modify
Compare
Search
Abandon
Extended
```

有太多的类型列出这里的例子，所以请尝试为自己使用 监控搜索示例

#### 20.4.7 Overlays

主条目包含运行时可用的覆盖类型; 每个叠加层的子条目包含叠加层的类型。

如果启用动态覆盖，它也应该包含已加载的模块：

```
# Overlays, Monitor
dn: cn=Overlays,cn=Monitor
structuralObjectClass: monitorContainer
monitoredInfo: syncprov
monitoredInfo: accesslog
monitoredInfo: glue
entryDN: cn=Overlays,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: TRUE
```

#### 20.4.8 SASL

目前空

#### 20.4.9 Statistics

它显示了服务器发送的数据的一些统计信息：

```
Bytes
PDU
Entries
Referrals
```

例如

```
# Entries, Statistics, Monitor
dn: cn=Entries,cn=Statistics,cn=Monitor
structuralObjectClass: monitorCounterObject
monitorCounter: 612248
entryDN: cn=Entries,cn=Statistics,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

#### 20.4.10 Threads

它包含启动时启用的最大线程数和当前的后备负载。

例如

```
# Max, Threads, Monitor
dn: cn=Max,cn=Threads,cn=Monitor
structuralObjectClass: monitoredObject
monitoredInfo: 16
entryDN: cn=Max,cn=Threads,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

#### 20.4.11 Time

它包含两个子条目，其中包含服务器的开始时间和当前时间。

例如

开始时间：

```
dn: cn=Start,cn=Time,cn=Monitor
structuralObjectClass: monitoredObject
monitorTimestamp: 20061205124040Z
entryDN: cn=Start,cn=Time,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

当前时间：

```
dn: cn=Current,cn=Time,cn=Monitor
structuralObjectClass: monitoredObject
monitorTimestamp: 20061207120624Z
entryDN: cn=Current,cn=Time,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

#### 20.4.12 TLS

目前空

### 20.4.13 Waiters

它包含当前读取服务器的数量。

例如

读服务员：

```
dn: cn=Read,cn=Waiters,cn=Monitor
structuralObjectClass: monitorCounterObject
monitorCounter: 7
entryDN: cn=Read,cn=Waiters,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

写服务员：

```
dn: cn=Write,cn=Waiters,cn=Monitor
structuralObjectClass: monitorCounterObject
monitorCounter: 0
entryDN: cn=Write,cn=Waiters,cn=Monitor
subschemaSubentry: cn=Subschema
hasSubordinates: FALSE
```

在这里添加新的监控事项，并讨论，引用手册页和示例

