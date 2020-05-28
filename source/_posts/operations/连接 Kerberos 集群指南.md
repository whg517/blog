---
title: 连接 Kerberos 集群指南
author: wanghuagang
date: 2019-05-06 12:30:00
updated: 2020-05-28 11:50:00
tags: 
 - Kerberos
categories:
 - 运维
---

本文主要介绍客户端在使用 Kerberos 的时候如何连接 Kerberos 集群，使用 Kerberos 安全认证。其中包含 Windows 客户端， Linux 客户端安装使用记录。

<!--more-->

## Kerberos 介绍

Kerberos 简介

![kerberos_pic](https://gss2.bdstatic.com/-fo3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268%3Bg%3D0/sign=9213b35de9f81a4c2632ebcfef110764/a5c27d1ed21b0ef485f59977dfc451da81cb3efd.jpg)

**Kerberos**是一种计算机网络授权协议，用来在非安全网络中，对个人通信以安全的手段进行身份认证。这个词又指[麻省理工学院](https://baike.baidu.com/item/麻省理工学院)为这个协议开发的一套计算机软件。

先来看看Kerberos协议的前提条件：

如下图所示，Client与KDC， KDC与Service 在协议工作前已经有了各自的共享密钥，并且由于协议中的消息无法穿透防火墙，这些条件就限制了Kerberos协议往往用于一个组织的内部。

![kerberos](http://qiniu.iclouds.work/Fmaqh-qBfq-rXFuXcSUNxplf9Ibu.png)


**过程**

Kerberos协议分为两个部分：

1. Client向KDC发送自己的身份信息，KDC从Ticket Granting Service得到TGT(ticket-granting ticket)， 并用协议开始前Client与KDC之间的密钥将TGT加密回复给Client。

   此时只有真正的Client才能利用它与KDC之间的密钥将加密后的TGT解密，从而获得TGT。

   （此过程避免了Client直接向KDC发送密码，以求通过验证的不安全方式）

2. Client利用之前获得的TGT向KDC请求其他Service的Ticket，从而通过其他Service的身份鉴别。

   Kerberos协议的重点在于第二部分，简介如下：

   ![kerberos](http://qiniu.iclouds.work/Fkxqfdj7TUx0InXKX3w6QkbNawBp.png)

   - Client将之前获得TGT和要请求的服务信息(服务名等)发送给KDC，KDC中的Ticket Granting Service将为Client和Service之间生成一个Session Key用于Service对Client的身份鉴别。然后KDC将这个Session Key和用户名，用户地址（IP），服务名，有效期, 时间戳一起包装成一个Ticket(这些信息最终用于Service对Client的身份鉴别)发送给Service， 不过Kerberos协议并没有直接将Ticket发送给Service，而是通过Client转发给Service.所以有了第二步。
   - 此时KDC将刚才的Ticket转发给Client。由于这个Ticket是要给Service的，不能让Client看到，所以KDC用协议开始前KDC与Service之间的密钥将Ticket加密后再发送给Client。同时为了让Client和Service之间共享那个秘密(KDC在第一步为它们创建的Session Key)， KDC用Client与它之间的密钥将Session Key加密随加密的Ticket一起返回给Client。
   - 为了完成Ticket的传递，Client将刚才收到的Ticket转发到Service. 由于Client不知道KDC与Service之间的密钥，所以它无法算改Ticket中的信息。同时Client将收到的Session Key解密出来，然后将自己的用户名，用户地址（IP）打包成Authenticator用Session Key加密也发送给Service。
   - Service 收到Ticket后利用它与KDC之间的密钥将Ticket中的信息解密出来，从而获得Session Key和用户名，用户地址（IP），服务名，有效期。然后再用Session Key将Authenticator解密从而获得用户名，用户地址（IP）将其与之前Ticket中解密出来的用户名，用户地址（IP）做比较从而验证Client的身份。
   - 如果Service有返回结果，将其返回给Client。

## Kerberos Client 配置

### Ubuntu

准备

同步时间

```bash
timedatectl set-timezone "Asia/Shanghai" # 设置系统时区
timedatectl set-ntp true # 联网自动同步到 NTP 时间服务器
```



根据 [Ubuntu 官方文档](https://help.ubuntu.com/lts/serverguide/kerberos.html.en) **Kerberos Linux Client** 一节操作

1. 安装

```bash
sudo apt install krb5-user
```

2. 修改配置

修改 `/etc/krb5.conf` ，配置如下

```
[libdefaults]
  renew_lifetime = 7d
  forwardable = true
  default_realm = TENDATA.CN
  ticket_lifetime = 24h
  dns_lookup_realm = false
  dns_lookup_kdc = false
  default_ccache_name = /tmp/krb5cc_%{uid}
  #default_tgs_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5
  #default_tkt_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5

[logging]
  default = FILE:/var/log/krb5kdc.log
  admin_server = FILE:/var/log/kadmind.log
  kdc = FILE:/var/log/krb5kdc.log

[realms]
  TENDATA.CN = {
    admin_server = m1.node.hadoop
    kdc = 192.168.10.1
    kdc = 192.168.10.2
  }
```

3. 配置域名解析

修改 hosts 文件，增加如下解析记录。

```
192.168.10.1  m1.node.hadoop
192.168.10.2  m2.node.hadoop
192.168.10.3  m3.node.hadoop

192.168.10.10 d1.node.hadoop
192.168.10.11 d2.node.hadoop
192.168.10.12 d3.node.hadoop
```

4. 使用 Kerberos Client

使用 `kinit` 获取 tgt 

```
kevin@ubuntu-kevin:~$ kinit whg
Password for whg@TENDATA.CN: 

kevin@ubuntu-kevin:~$ klist
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: whg@TENDATA.CN

Valid starting       Expires              Service principal
2019-05-06T09:45:56  2019-05-07T09:45:55  krbtgt/TENDATA.CN@TENDATA.CN
```

使用 `kdestroy` 销毁 tgt

```
kevin@ubuntu-kevin:~$ kdestroy 

kevin@ubuntu-kevin:~$ klist 
klist: No credentials cache found (filename: /tmp/krb5cc_1000)
```

命令使用参数和其他命令请阅读 [Kerberos 用户命令](https://web.mit.edu/kerberos/krb5-latest/doc/user/user_commands/index.html) 

5. 访问相关服务

使用 `curl` 访问 Kerberos 认证的页面。

正常访问不会出现 `Error 401 Authentication required` 。

```
kevin@ubuntu-kevin:~$ curl -k --negotiate -u: -H "Content-Type: application/json" -X GET http://m1.node.hadoop:50070/
<!--
   Licensed to the Apache Software Foundation (ASF) under one or more
   contributor license agreements.  See the NOTICE file distributed with
   this work for additional information regarding copyright ownership.
   The ASF licenses this file to You under the Apache License, Version 2.0
   (the "License"); you may not use this file except in compliance with
   the License.  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="REFRESH" content="0;url=dfshealth.html" />
<title>Hadoop Administration</title>
</head>
</html>
```

或者访问 `curl -L -k --negotiate -u: -H "Content-Type: application/json" -X GET http://m1.node.hadoop:50070/webhdfs/v1/?op=LISTSTATUS` 正常出现 HDFS 文件列表(JSON 格式数据)，没有 401 错误即代表正常访问



**Error 401 Authentication required**

```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html;charset=utf-8"/>
<title>Error 401 Authentication required</title>
</head>
<body><h2>HTTP ERROR 401</h2>
<p>Problem accessing /webhdfs/v1/. Reason:
<pre>    Authentication required</pre></p>
</body>
```

### Centos

准备

同步时间

```bash
timedatectl set-timezone "Asia/Shanghai" # 设置系统时区
timedatectl set-ntp true # 联网自动同步到 NTP 时间服务器
```

1. 安装 Kerberos Client

```bash
yum -y install krb5-workstation
```

其余步骤和 Ubuntu 操作相同。



### Windows

Windows 默认已经支持 Kerberos，但需要使用 加入 Active Directory ，以获取 tgt。在使用 openladp 的情况下需要配置 AD proxy，这种方式比较麻烦。MIT 提供 [Kerberos for Windows](<https://web.mit.edu/kerberos/dist/index.html>) 来更方便的使用 Kerberos 认证。

没有使用 Windows 默认 kerberos 支持，当有程序调用Windows 集成身份验证的时候也就会认证失败。例如 IE，Chrome 浏览器登陆 Kerberos 认证的网站时，会出现 `HTTP ERROR 403` 错误。

参考文章：[Configuring Kerberos Authentication for Windows](<https://www.simba.com/products/Hive/doc/ODBC_InstallGuide/win/content/odbc/hi/kerberos.htm>) 

注意：在选择 kfw 版本应该与后面安装浏览器版本一致，推荐安装 64-bit kfw，因为会向下兼容

1. 下载安装

[Kerberos for Windows](<https://web.mit.edu/kerberos/dist/index.html>)

安装时推荐选择 “Complete” 安装

![kerberos_for_windows_install](http://qiniu.iclouds.work/FldFbui6VkQsUeIxVUS9zBIpFXKd.png)

2. 配置

kfw 64位 默认安装在 `C:\Program Files\MIT\Kerberos` 下，当然这不是重点。 kfw 会加载 `C:\ProgramData\MIT\Kerberos5\krb5.ini` 的配置文件。

> `ProgramData` 目录默认是隐藏的。

修改该配置文件内容为：

```
[libdefaults]
  renew_lifetime = 7d
  forwardable = true
  default_realm = TENDATA.CN
  ticket_lifetime = 24h
  dns_lookup_realm = false
  dns_lookup_kdc = false
  default_ccache_name = /tmp/krb5cc_%{uid}
  #default_tgs_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5
  #default_tkt_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5

[logging]
  default = FILE:/var/log/krb5kdc.log
  admin_server = FILE:/var/log/kadmind.log
  kdc = FILE:/var/log/krb5kdc.log

[realms]
  TENDATA.CN = {
    admin_server = m1.node.hadoop
    kdc = 192.168.10.1
    kdc = 192.168.10.2
  }
```

3. 配置系统属性：

打开 `控制面板` -> 选择 `系统和安全` -> 选择 `系统` -> 选择 `高级系统设置` 打开 `系统属性` 面板 -> 点击 `高级` 选项卡 -> 打开环境变量

新建系统变量，变量名称为 `KRB5CCNAME` 变量值为 `C:\temp\krb5cache`  。前提时 `C:\temp\` 目录存在，`krb5cache` 不需要创建，在使用 `kinit` 命令的时候会自动生成该文件。

如果 `krb5.ini` 的配置文件不在默认目录可以使用变量名 `KRB5_CONFIG` 指定，变量值为该文件的绝对路径

4. 使用

在 CMD 中使用 `kinit` 获取 tgt。

由于 Windows 支持 Kerberos ，所有部分命令默认时 Windows 的。比如 `klist` 。

修改环境变量，让 kfw 命令覆盖 Windows。

编辑 `Path` 环境变量，将 `kerberos` 变量上移至最上面即可。

![up_kerberos_bin](http://qiniu.iclouds.work/Fhof90F5SjgN4s9uGD-JU9-52gdx.png)

在 CMD 中使用 `klist` 即可查看以获取的 tgt

**注意：**

由于 Java 也实现了 kerberos client 认证机制，在 JDK 的 `bin` 目录提供了 `kinit` 、 `klist` 等相关命令，在配置Path 环境变量时，Kerberos 的优先级应该比 JDK 的要高。

**403 ** 分析：

在没有安装 kfw 时，系统已经具有 `klist` 命令，该命令位于 `C:\WINDOWS\system32` 。

```
C:\WINDOWS\system32>klist

当前登录 ID 是 0:0x73218

缓存的票证: (0)

C:\WINDOWS\system32>klist sessions

当前登录 ID 是 0:0x73218
[0] 会话 0 0:0x136dc999 DESKTOP-5KM5T43\Administrator NTLM:Network
[1] 会话 0 0:0x134bf7a2 DESKTOP-5KM5T43\Administrator NTLM:Network
```

相关命令使用前查看 [klist](<https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/klist>) 

### JVM

根据 [Java Platform, Standard Edition Security Developer’s Guide](<https://docs.oracle.com/javase/10/security/introduction-jaas-and-java-gss-api-tutorials1.htm#JSSEC-GUID-202DE2FD-E2CA-44D1-9391-50127913538E>) 描述，Java 为开发人员提供了 JAAS 和 Java GSS-API 来使用 Kerberos 认证。前者是直接认证，后者是使用调用底层认证。

**注意：** 根据 JCE 描述，由于一些国家的进口管制限制，一些版本的 JDK 或者 JRE 允许强，但有限的加密策略。所以如果需要无限长度加密策略，需要下载 `Java Cryptography Extension (JCE) ` 支持。详细描述请查看 JCE 压缩包中的 README.txt。而 Kerberos 的加密正式需要无限长度支持。下载地址为 [Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files 8 Download](<https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html>) 。下载后解压到 `JAVA_HOME\jre\lib\security` 即可。

1. 配置

[Kerberos Requirements](<https://docs.oracle.com/javase/10/security/kerberos-requirements1.htm#JSSEC-GUID-EAA2758B-3071-4CDA-AEF1-D76F5271E998>) 中描述了 JDK 是如何使用 Kerberos 的。它支持系统属性值，和 `krb5.conf` 配置文件读取。为了便于使用，下面仅配置使用 `krb5.conf` 

**注意：** 系统属性值比配置有更高的优先级！

JDK 查找配置文件的算法如下：

- 系统属性 ``java.security.krb5.conf` 指定文件位置

- 查找配置文件位置 (java-home 为 JDK 安装目录)

  - `<java-home>\conf\security` (Windows) 
  - `<java-home>/conf/security`  (Solaris, Linux, and macOS)

- 如果上述没有找到，则查找如下位置：

  - `/etc/krb5/krb5.conf` (Solaris)
  - `C:\Windows\krb5.ini` (Windows)

  - `/etc/krb5.conf` (Linux)
  - `~/Library/Preferences/edu.mit.Kerberos`, `/Library/Preferences/edu.mit.Kerberos`, or `/etc/krb5.conf` (macOS)

所以，在 Windows 中，应该把 `krb5.conf` 更名为 `krb5.ini` 放在 `C:\Windows\krb5.ini` 。

2. 验证

在 CMD 下，进入到 `JAVA_HOME\bin\` 中，执行 `kinit` 和 `klist` 。关于命令的具体使用，请查看 [Security Tools and Commands](<https://docs.oracle.com/javase/10/tools/security-tools-and-commands.htm#JSWOR691>)



## 浏览器访问 Kerberos 服务



参考文章 [How to configure supported browsers for Kerberos and NTLM](https://ping.force.com/Support/PingFederate/Integrations/How-to-configure-supported-browsers-for-Kerberos-NTLM) 

PingFederate Integrated Windows Authentication (IWA)适配器支持Kerberos和NTLM身份验证协议，但是需要配置一些浏览器来使用它们。

对于Kerberos和NTLM身份验证，PingFederate IWA适配器使用SPNEGO(Simple and Protected GSS-API Negotiation)机制协商Kerberos或NTLM作为底层身份验证协议。下面的每个浏览器都支持SPNEGO，但是由于浏览器和操作系统的结合，存在的差异可能会影响在每个实例中协商的协议。

The PingFederate IWA Adapter supports the following browsers:

- Internet Explorer
- Mozilla Firefox
- Google Chrome
- Apple Safari

### Internet Explorer

鉴于 Windows 原因，不进行配置。

### Firefox

> 当前 Firefox 版本 64.0.2(64-bit) 

参考 [Configuring Firefox for Negotiate Authentication](http://people.redhat.com/mikeb/negotiate/) 、 [Integrated Authentication](<https://developer.mozilla.org/en-US/docs/Mozilla/Integrated_authentication) 

#### Linux

1. 在地址栏中键入about:config，点击 `I accept the risk !` ，进入配置页面

![firefox_config](http://qiniu.iclouds.work/FtiywmQmIg8ILbI9UX6kr9VWZePz.png)

2. 配置 SPNEGO authentication

配置 `network.negotiate-auth.trusted-uris` 为允许使用浏览器进行SPNEGO身份验证的站点， 支持统配。多个站点使用逗号隔开。

![forefox_config_negotiate](http://qiniu.iclouds.work/FukOO6s6wCNDX5vE-BuJwWmTcQvF.png)

然后使用 `kinit` 获取认证 tgt 之后，打开 Chrome 访问服务页面。没有出现 401 即为正常。

#### Windows

注意：Firefox 浏览器版本应与前面安装的 kfw 版本一致，否则会出现 401 。如果 kfw 为 64bit 则不用关注这一点。

和 Linux 相同，修改 Firefox 配置

```
network.negotiate-auth.trusted-uris
network.auth.use-sspi
```

然后使用 `kinit` 获取认证 tgt 之后，打开 Chrome 访问服务页面。没有出现 401 即为正常。

### Chrome

#### Linux

根据 [Chrome HTTP authentication(https://www.chromium.org/developers/design-documents/http-authentication) 中 **Integrated Authentication** 描述 Chrome 是支持 Kerberos 和 NTLM 认证的。默认不会启用，只有从代理接收身份验证协商或者在白名单的服务器协商时，才会启用继承身份验证。

##### Chrome 策略组启用白名单

添加白名单可以通过 [AuthServerWhitelist](https://dev.chromium.org/administrators/policy-list-3#AuthServerWhitelist) 策略设置。多个白名单用以逗号分隔。

参考：[Linux Quick Start](<https://dev.chromium.org/administrators/linux-quick-start>) 

对于Chromium，策略配置文件位于/etc/chromium之下，对于谷歌Chrome，策略配置文件位于/etc/opt/chrome之下。在这些目录中保存着两组策略:一组是管理员需要并强制执行的，另一组是推荐给用户但不是必需的。这两组在:

```
/etc/opt/chrome/policies/managed/
/etc/opt/chrome/policies/recommended/
```

如果这些目录不存在，请创建它们:

```
>mkdir /etc/opt/chrome/policies
>mkdir /etc/opt/chrome/policies/managed
>mkdir /etc/opt/chrome/policies/recommended
```

确保非管理员用户无法写入/托管下的文件;否则，他们可能只是覆盖您的策略来获得他们想要的配置!

```
>chmod -w /etc/opt/chrome/policies/managed
```

要设置所需的策略，请在 `/etc/opt/chrome/policies/managed/` 下创建一个名为 `test_policy.json` 的文件。

```
>touch /etc/opt/chrome/policies/managed/test_policy.json
```

在这个文件中，放入以下内容:

```
{
          "AuthServerWhitelist": ".node.hadoop"
}
```

就是这样!下次在该机器上启动谷歌Chrome时，主页将被锁定到这个值。

在地址栏输入 `chrome:policy` 查看当前策略

![chrome_policy](http://qiniu.iclouds.work/FhCrKMifE6Nj6ZlQGMqlcRtVKZ1C.png)

在使用 `kinit` 获取认证 tgt 之后，打开 Chrome 访问服务页面。没有出现 401 即为正常。

##### 使用命令行参数启动

根据 [Run Chromium with flags](<https://www.chromium.org/developers/how-tos/run-chromium-with-flags>) 中描述，Chromium(和Chrome)支持接受命令行标志(或“开关”)，以便启用特定功能或修改默认功能。

当前可用参数可以查看 [List of Chromium Command Line Switches](<https://peter.sh/experiments/chromium-command-line-switches/>) 。可以使用 [--auth-server-whitelist ](<https://peter.sh/experiments/chromium-command-line-switches/#auth-server-whitelist>) 参数启用白名单功能。

```bash
chrome --auth-server-whitelist=".node.hadoop"
```

#### Windows

由于 Chrome 自身不带有 Gssapi，而是调用操作系统类库，而且依赖于 `Internet 选项` 配置。因此不在配置。

具体内容请参考 [Chrome HTTP authentication](<https://www.chromium.org/developers/design-documents/http-authentication>) 中 `Negotiate external libraries` 部分



