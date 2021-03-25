---
title: MIT Kerberos Documentation (1.15.2)
author: Kevin
date: 2017-11-20 15:59:25
updated: 2020-05-28 11:50:00
tags: kerberos
categories: Documents
---

本文使用 Google 翻译 Kerberos 官方文档。如有不通顺或错误之处，请大家及时之处，不对的地方请大家参照官方文档

[MIT Kerberos Documentation (1.15.2)](http://web.mit.edu/kerberos/krb5-latest/doc/)

<!-- more -->

[TOC]

# 2 对于管理员

## 1.1 安装指南

### 1.1.1 安装KDC

在生产环境中设置Kerberos时，最好将多个从属KDC与主KDC一起使用，以确保Kerberized服务的持续可用性。每个KDC都包含Kerberos数据库的副本。
主KDC包含领域数据库的可写拷贝，它会定期复制到从属KDC。所有数据库更改（如密码更改）都在主KDC上进行。当主KDC不可用时，从属KDC提供Kerberos票证授予
服务，但不提供数据库管理。麻省理工学院建议您安装所有KDC，以便能够作为主机或其中一个从机。这将使您能够轻松地切换主KDC与其中一个从站（如果需要）
（请参阅[切换主从KDC](http://web.mit.edu/kerberos/krb5-latest/doc/admin/install_kdc.html#switch-master-slave)）。
此安装过程基于该建议。

**警告**

- Kerberos系统依赖于正确的时间信息的可用性。确保主机和所有从机KDC都正确同步时钟。
- 最好在有限访问的安全和专用硬件上安装和运行KDC。如果您的KDC也是文件服务器，FTP服务器，Web服务器，甚至只是客户端机器，那么通过任何这些区域中的安全漏洞获取root访问权限的用户可能会访问Kerberos数据库。

#### 1.1.1.1 安装和配置主KDC

从操作系统提供的软件包或源中安装Kerberos（请参阅[单个树中的构建](http://web.mit.edu/kerberos/krb5-latest/doc/build/doing_build.html#do-build)）。

注意

为了本文档的目的，我们将使用以下名称：

```
kerberos.mit.edu    - master KDC
kerberos-1.mit.edu  - slave KDC
ATHENA.MIT.EDU      - realm name
.k5.ATHENA.MIT.EDU  - stash file
admin/admin         - admin principal
```

有关此主题文件的默认名称和位置，请参阅[MIT Kerberos默认值](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#mitk5defaults)。调整系统环境的名称和路径。

#### 1.1.1.2 编辑KDC配置文件

修改配置文件[krb5.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#krb5-conf-5)和 [kdc.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)，以反映您的领域的正确信息（如域名域映射和Kerberos服务器名称）。（有关这些文件的推荐默认位置，请参阅[MIT Kerberos默认值](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#mitk5defaults)）。

配置中的大多数标签都具有默认值，对大多数站点都有效。[krb5.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#krb5-conf-5)文件中有一些标签必须指定其值，本节将对这些标签进行 说明。

如果这些配置文件的位置与默认配置文件的位置不同，请将**KRB5_CONFIG**和**KRB5_KDC_PROFILE**环境变量分别设置为指向krb5.conf和kdc.conf。例如：

```
export KRB5_CONFIG=/yourdir/krb5.conf
export KRB5_KDC_PROFILE=/yourdir/kdc.conf
```

##### krb5.conf

如果您不使用DNS TXT记录（请参阅将[主机名映射到Kerberos领域](http://web.mit.edu/kerberos/krb5-latest/doc/admin/realm_config.html#mapping-hostnames)），必须在[libdefaults](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#libdefaults) 部分中指定**default_realm**。如果您不使用DNS URI或SRV记录（请参阅 [KDC](http://web.mit.edu/kerberos/krb5-latest/doc/admin/realm_config.html#kdc-hostnames)和[KDC发现的](http://web.mit.edu/kerberos/krb5-latest/doc/admin/realm_config.html#kdc-discovery)[主机名](http://web.mit.edu/kerberos/krb5-latest/doc/admin/realm_config.html#kdc-hostnames)），则必须在[域](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#realms)部分中包含每个 *领域* 的 **kdc** 标签。要在每个领域与kadmin服务器进行通信， 必须在[域](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#realms)部分中设置**admin_server**标记 。

一个例子krb5.conf文件：

```conf
[libdefaults]
    default_realm = ATHENA.MIT.EDU

[realms]
    ATHENA.MIT.EDU = {
        kdc = kerberos.mit.edu
        kdc = kerberos-1.mit.edu
        admin_server = kerberos.mit.edu
    }
```

##### kdc.conf

The kdc.conf file can be used to control the listening ports of theKDC and kadmind, as well as realm-specific defaults, the database typeand location, and logging.

An example kdc.conf file:

```conf
[kdcdefaults]
    kdc_listen = 88
    kdc_tcp_listen = 88

[realms]
    ATHENA.MIT.EDU = {
        kadmind_port = 749
        max_life = 12h 0m 0s
        max_renewable_life = 7d 0h 0m 0s
        master_key_type = aes256-cts
        supported_enctypes = aes256-cts:normal aes128-cts:normal
        # If the default location does not suit your setup,
        # explicitly configure the following values:
        #    database_name = /var/krb5kdc/principal
        #    key_stash_file = /var/krb5kdc/.k5.ATHENA.MIT.EDU
        #    acl_file = /var/krb5kdc/kadm5.acl
    }

[logging]
    # By default, the KDC and kadmind will log output using
    # syslog.  You can instead send log output to files like this:
    kdc = FILE:/var/log/krb5kdc.log
    admin_server = FILE:/var/log/kadmin.log
    default = FILE:/var/log/krb5lib.log
```

替换`ATHENA.MIT.EDU`和`kerberos.mit.edu分别`与您的Kerberos领域和服务器的名称。

**注意：**

您必须对**database_name**， **key_stash_file**和**acl_file**使用的目标目录（这些目录必须存在）具有写入权限。

#### 1.1.1.2  创建KDC数据库

您将在主KDC上使用[kdb5_util](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kdb5_util.html#kdb5-util-8)命令创建Kerberos数据库和可选的[存储文件](http://web.mit.edu/kerberos/krb5-latest/doc/basic/stash_file_def.html#stash-definition)。

注意

如果您选择不安装存储文件，KDC将在每次启动时提示您输入主密钥。这意味着KDC无法自动启动，例如在系统重启后。

[kdb5_util](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kdb5_util.html#kdb5-util-8)将提示您输入Kerberos数据库的主密码。该密码可以是任何字符串。一个好的密码是你可以记住的密码，但没有人能猜到。不良密码的例子是可以在字典中找到的字词，任何常见或受欢迎的名称，特别是着名人物（或卡通人物），您的用户名（任何形式）（例如，前进，后退，重复两次等），以及本手册中出现的任何样本密码。如果本手册中没有出现的密码的一个例子是“MITiys4K5！”，表示“MIT是Kerberos 5的来源”（这是每个单词的第一个字母，用数字“ 4“表示”for“，最后包括标点符号。）

以下是使用[kdb5_util](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kdb5_util.html#kdb5-util-8)命令在主KDC上创建Kerberos数据库和存储文件的示例。用您的Kerberos领域的名称替换`ATHENA.MIT.EDU`：

```
shell% kdb5_util create -r ATHENA.MIT.EDU -s

Initializing database '/usr/local/var/krb5kdc/principal' for realm 'ATHENA.MIT.EDU',
master key name 'K/M@ATHENA.MIT.EDU'
You will be prompted for the database Master Password.
It is important that you NOT FORGET this password.
Enter KDC database master key:  <= Type the master password.
Re-enter KDC database master key to verify:  <= Type it again.
shell%
```

这将在[LOCALSTATEDIR ](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths) `/krb5kdc`（或在[kdc.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)中[指定](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)的位置）创建五个文件：

- 两个Kerberos数据库文件，`principal`和`principal.ok`
- Kerberos管理数据库文件`principal.kadm5`
- 管理数据库锁文件，`principal.kadm5.lock`
- 在此示例中为`.k5.ATHENA.MIT.EDU`的存储文件。如果您不想使用存储文件，请运行上述命令而不使用**-s**选项。

有关管理Kerberos数据库的更多信息，请参阅Kerberos数据库 [上的操作](http://web.mit.edu/kerberos/krb5-latest/doc/admin/database.html#db-operations)。

#### 1.1.1.3 将管理员添加到ACL文件

接下来，您需要创建一个访问控制列表（ACL）文件，并将至少一个管理员的Kerberos主体放在其中。该文件由[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)守护进程用于控制哪些主体可以查看和对Kerberos数据库文件进行特权修改。ACL文件名由[kdc.conf中](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)的**acl_file** 变量决定; 默认值为[LOCALSTATEDIR](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths) `/ krb5kdc/kadm5.acl`。

有关Kerberos ACL文件的更多信息，请参阅[kadm5.acl](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kadm5_acl.html#kadm5-acl-5)。

#### 1.1.1.4 将管理员添加到Kerberos数据库

接下来，您需要向Kerberos数据库添加管理主体（即，允许管理Kerberos数据库的主体）。您现在*必须*至少添加一个主体，以允许通过网络进行Kerberos管理守护程序kadmind和kadmin程序之间的通信，以便进一步管理。为此，请使用主KDC上的kadmin.local实用程序。kadmin.local旨在在主KDC主机上运行，而不对管理服务器使用Kerberos身份验证; 相反，它必须对本地文件系统上的Kerberos数据库具有读写访问权限。

您创建的管理员主体应该是您添加到ACL文件的[管理员](http://web.mit.edu/kerberos/krb5-latest/doc/admin/install_kdc.html#admin-acl)（请参阅[将管理员添加到ACL文件](http://web.mit.edu/kerberos/krb5-latest/doc/admin/install_kdc.html#admin-acl)）。

在以下示例中，将 创建管理主体`admin/admin`：

```
shell% kadmin.local

kadmin.local: addprinc admin/admin@ATHENA.MIT.EDU

WARNING: no policy specified for "admin/admin@ATHENA.MIT.EDU";
assigning "default".
Enter password for principal admin/admin@ATHENA.MIT.EDU:  <= Enter a password.
Re-enter password for principal admin/admin@ATHENA.MIT.EDU:  <= Type it again.
Principal "admin/admin@ATHENA.MIT.EDU" created.
kadmin.local:
```

#### 1.1.1.5 在主KDC上启动Kerberos守护进程

此时，您可以在主KDC上启动Kerberos KDC（[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)）和管理守护程序。要这样做，键入：

```
shell% krb5kdc
shell% kadmind
```

每个服务器守护进程将在后台进行分支和运行。

注意

假设您希望这些守护进程在启动时自动启动，可以将它们添加到KDC的`/ etc / rc`或 `/ etc / inittab`文件中。为了做到这一点，你需要一个 [藏书文件](http://web.mit.edu/kerberos/krb5-latest/doc/basic/stash_file_def.html#stash-definition)。

您可以通过在[krb5.conf中](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#krb5-conf-5)定义的日志记录位置中检查其启动消息来验证它们是否正常启动 （请参阅[logging](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#logging)）。例如：

```
shell% tail /var/log/krb5kdc.log
Dec 02 12:35:47 beeblebrox krb5kdc[3187](info): commencing operation
shell% tail /var/log/kadmin.log
Dec 02 12:35:52 beeblebrox kadmind[3189](info): starting
```

启动时守护程序遇到的任何错误也将列在日志记录输出中。

作为附加验证，请检查[kinit是否](http://web.mit.edu/kerberos/krb5-latest/doc/user/user_commands/kinit.html#kinit-1)成功执行了您在上一步中创建的主体（[将管理员添加到Kerberos数据库](http://web.mit.edu/kerberos/krb5-latest/doc/admin/install_kdc.html#addadmin-kdb)）。跑：

```
shell% kinit admin/admin@ATHENA.MIT.EDU
```

#### 1.1.1.6 安装从KDC

您现在可以开始配置从属KDC。

**注意：**

假设您正在设置KDC，以便您可以轻松地使用其中一个从站切换主KDC，则应在主KDC和从KDC上执行以上步骤，除非另有说明。

##### 为从KDC创建主机keytabs

每个KDC需要Kerberos数据库中的`主机`密钥。将数据库转储文件从主KDC传播到辅助KDC服务器时，这些密钥用于相互验证。

在主KDC上，连接到管理界面，并为每个KDC的`主机`服务创建主机主体。例如，如果主KDC被称为`kerberos.mit.edu`，并且您有一个名为`kerberos-1.mit.edu`的从KDC ，您将键入以下内容：

```
shell% kadmin
kadmin: addprinc -randkey host/kerberos.mit.edu
NOTICE: no policy specified for "host/kerberos.mit.edu@ATHENA.MIT.EDU"; assigning "default"
Principal "host/kerberos.mit.edu@ATHENA.MIT.EDU" created.

kadmin: addprinc -randkey host/kerberos-1.mit.edu
NOTICE: no policy specified for "host/kerberos-1.mit.edu@ATHENA.MIT.EDU"; assigning "default"
Principal "host/kerberos-1.mit.edu@ATHENA.MIT.EDU" created.
```

将主KDC服务器置于Kerberos数据库中并不是绝对必要的，但如果希望能够与其中一个从站交换主KDC，则可以方便。

接下来，为所有参与的KDC 提取`主机`随机密钥，并将其存储在每个主机的默认密钥表文件中。理想情况下，您应该在自己的KDC上本地提取每个keytab。如果这不可行，您应该使用加密会话来通过网络发送它们。要直接在名为`kerberos-1.mit.edu`的从属KDC上提取密钥表 ，您将执行以下命令：

```
kadmin: ktadd host/kerberos-1.mit.edu
Entry for principal host/kerberos-1.mit.edu with kvno 2, encryption
    type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/kerberos-1.mit.edu with kvno 2, encryption
    type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/kerberos-1.mit.edu with kvno 2, encryption
    type des3-cbc-sha1 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/kerberos-1.mit.edu with kvno 2, encryption
    type arcfour-hmac added to keytab FILE:/etc/krb5.keytab.
```

如果您正在为主KDC上的从属KDC提供一个名为`kerberos-1.mit.edu`的密钥表 ，那么您应该为该机器的keytab使用专用的临时密钥表文件：

```
kadmin: ktadd -k /tmp/kerberos-1.keytab host/kerberos-1.mit.edu
Entry for principal host/kerberos-1.mit.edu with kvno 2, encryption
    type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/kerberos-1.mit.edu with kvno 2, encryption
    type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
```

文件`/tmp/kerberos-1.keytab`可以作为`/etc/krb5.keytab`安装 在主机`kerberos-1.mit.edu上`。

##### 配置从KDC

数据库传播复制主数据库的内容，但不传播配置文件，存储文件或kadm5 ACL文件。以下文件必须手动复制到每个从站（有关这些文件的默认位置，请参阅 [MIT Kerberos默认值](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#mitk5defaults)）：

- 的krb5.conf
- kdc.conf中
- 的kadm5.acl
- 主密钥存档文件

将复制的文件移动到相应的目录中，与主KDC完全相同。kadm5.acl只需要允许从站与主KDC交换。

数据库通过[kpropd](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kpropd.html#kpropd-8)守护进程从主KDC传播到从属KDC 。您必须明确指定允许在从机上使用新数据库提供Kerberos转储更新的主体。在KDC状态目录中创建一个名为kpropd.acl的文件，其中包含每个KDC 的`主机`主体：

```
host/kerberos.mit.edu@ATHENA.MIT.EDU
host/kerberos-1.mit.edu@ATHENA.MIT.EDU
```

**注意：**

如果您希望在一段时间内切换主从KDC，请在所有KDC上的kpropd.acl文件中列出所有参与的KDC服务器的主机主体。否则，您只需要在KDC的kpropd.acl文件中列出主KDC的主机主体。

然后，将以下行添加到每个KDC 上的`/etc/inetd.conf中`（调整路径为kpropd）：

```
krb5_prop stream tcp nowait root /usr/local/sbin/kpropd kpropd
```

您还需要在每个KDC上的`/etc/services中`添加以下行（如果尚未存在）（假定使用默认端口）：

```
krb5_prop       754/tcp               # Kerberos slave propagation
```

重新启动inetd守护进程。

或者，将[kpropd](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kpropd.html#kpropd-8)作为独立守护进程启动。启用增量传播时，这是必需的。

现在，从KDC能够接受数据库传播，您需要从主服务器传播数据库。

注意：不要启动从机KDC; 您仍然没有主数据库的副本。

##### 将数据库传播到每个从属

首先，在主KDC上创建数据库的转储文件，如下所示：

```
shell% kdb5_util dump /usr/local/var/krb5kdc/slave_datatrans
```

然后，将数据库手动传播到每个从属KDC，如下例所示：

```
shell% kprop -f /usr/local/var/krb5kdc/slave_datatrans kerberos-1.mit.edu

Database propagation to kerberos-1.mit.edu: SUCCEEDED
```

您将需要一个脚本来转储和传播数据库。以下是一个Bourne shell脚本的例子。

**注意：**

请记住，您需要 使用KDC状态目录的名称替换`/usr/local/var/krb5kdc`。

```
#!/bin/sh

kdclist = "kerberos-1.mit.edu kerberos-2.mit.edu"

kdb5_util dump /usr/local/var/krb5kdc/slave_datatrans

for kdc in $kdclist
do
    kprop -f /usr/local/var/krb5kdc/slave_datatrans $kdc
done
```

您将需要设置一个cron作业以在您之前决定的时间间隔内运行此脚本（请参阅[数据库传播](http://web.mit.edu/kerberos/krb5-latest/doc/admin/realm_config.html#db-prop)）。

现在，从KDC具有Kerberos数据库的副本，您可以启动krb5kdc守护程序：

```
shell% krb5kdc
```

与主KDC一样，您可能希望将此命令添加到KDC的`/etc/rc`或`/etc/inittab`文件中，以便在启动时自动启动krb5kdc守护程序。

##### 安装失败了？

您可能会遇到以下错误消息。有关可能的原因和解决方案的更详细的讨论，请单击错误链接重定向到[故障排除](http://web.mit.edu/kerberos/krb5-latest/doc/admin/troubleshoot.html#troubleshoot) 部分。

1. [kprop：连接到服务器时没有到主机的路由](http://web.mit.edu/kerberos/krb5-latest/doc/admin/troubleshoot.html#kprop-no-route)
2. [kprop：连接到服务器时拒绝连接](http://web.mit.edu/kerberos/krb5-latest/doc/admin/troubleshoot.html#kprop-con-refused)
3. [kprop：服务器拒绝身份验证（在sendauth交换期间），同时向服务器进行身份验证](http://web.mit.edu/kerberos/krb5-latest/doc/admin/troubleshoot.html#kprop-sendauth-exchange)

#### 1.1.1.7 将Kerberos主体添加到数据库

一旦KDC建立并运行，就可以使用 [kadmin](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmin_local.html#kadmin-1)将用户，主机和其他服务的[主体](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmin_local.html#kadmin-1)加载到Kerberos数据库。此过程在[添加，修改和删除主体中](http://web.mit.edu/kerberos/krb5-latest/doc/admin/database.html#add-mod-del-princs)有详细描述。

您可能偶尔想使用您的一个从属KDC作为主人。如果升 级主KDC，或者主KDC有磁盘崩溃，可能会发生这种情况。有关说明，请参阅以下部分。

#### 1.1.1.9 开关主机和从KDC

您可能偶尔想使用您的一个从属KDC作为主人。如果升级主KDC，或者主KDC有磁盘崩溃，可能会发生这种情况。

假设您已将所有KDC配置为能够用作主KDC或从KDC（如本文档所推荐）），则您需要做的是进行更换：

如果主KDC仍在运行，请在*旧主* KDC 上执行以下操作：

1. 杀死kadmind进程。
2. 禁用传播数据库的cron作业。
3. 手动运行数据库传播脚本，以确保从站全部拥有数据库的最新副本（请参阅 [将数据库传播到每个从站KDC](http://web.mit.edu/kerberos/krb5-latest/doc/admin/install_kdc.html#kprop-to-slaves)）。

在*新*主人KDC上：

1. 启动[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)守护程序（请参阅[在主KDC上启动Kerberos守护程序](http://web.mit.edu/kerberos/krb5-latest/doc/admin/install_kdc.html#start-kdc-daemons)）。
2. 设置cron作业以传播数据库（请参阅 [将数据库传播到每个从属KDC](http://web.mit.edu/kerberos/krb5-latest/doc/admin/install_kdc.html#kprop-to-slaves)）。
3. 切换旧的和新的主KDC的CNAME。如果不能这样做，您需要在Kerberos领域的每台客户端机器上更改[krb5.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#krb5-conf-5)文件。

#### 1.1.1.10 增量数据库传播

如果您希望Kerberos数据库变大，您可能希望将增量传播设置为从属KDC。有关 详细信息，请参阅[增量数据库传播](http://web.mit.edu/kerberos/krb5-latest/doc/admin/database.html#incr-db-prop)。

### 1.1.2 安装和配置UNIX客户机

Kerberized客户端程序包括[kinit](http://web.mit.edu/kerberos/krb5-latest/doc/user/user_commands/kinit.html#kinit-1)， [klist](http://web.mit.edu/kerberos/krb5-latest/doc/user/user_commands/klist.html#klist-1)，[kdestroy](http://web.mit.edu/kerberos/krb5-latest/doc/user/user_commands/kdestroy.html#kdestroy-1)和[kpasswd](http://web.mit.edu/kerberos/krb5-latest/doc/user/user_commands/kpasswd.html#kpasswd-1)。所有这些程序都在目录[BINDIR中](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)。

您通常可以通过使用PAM将Kerberos与登录系统集成在客户端计算机上。细节因操作系统而异，应在操作系统文档中介绍。如果这样做，您需要确保您的用户知道在登录时使用其Kerberos密码。

您还需要教育您的用户使用票证管理程序kinit，klist和kdestroy。如果您没有将Kerberos密码更改集成到本机密码程序中（通常通过PAM），您将需要教育用户使用kpasswd代替其非Kerberos对等体passwd。

#### 1.1.2.1客户机配置文件

运行Kerberos的每台机器都应该有一个[krb5.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#krb5-conf-5)文件。至少应在[libdefaults](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#libdefaults) 中定义**default_realm**设置 。如果您没有使用DNS SRV记录（[KDC的主机名](http://web.mit.edu/kerberos/krb5-latest/doc/admin/realm_config.html#kdc-hostnames)）或URI记录（[KDC Discovery](http://web.mit.edu/kerberos/krb5-latest/doc/admin/realm_config.html#kdc-discovery)），它还必须包含一个[域](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#realms) 部分，其中包含您所在领域的KDC的信息。

考虑将**rdns**设置为false，以减少对服务主机名的正确DNS信息的依赖。关闭此标志表示通过转发名称解析（将您的域名添加到不合格的主机名，并在DNS中解析CNAME记录），服务主机名将被规范化，但不能通过反向地址查找。只有历史原因，此标志的默认值为true。

如果您期望用户经常使用可转发凭证登录到远程主机（例如，使用ssh），请考虑将可**转发**设置 为true，以便用户默认获得可转发的故障单。否则用户将需要使用`kinit -f`来获得可前往的票证。

考虑调整**ticket_lifetime**设置以匹配用户可能的会话长度。例如，如果您的大多数用户将登录八小时工作日，则可以将默认值设置为十小时，以便在早晨获得的票据在工作日结束后不久即将到期。用户仍然可以在必要时手动请求更长的票据，最多允许每个用户在KDC上的主体记录允许的最大值。

如果客户机主机可以访问不同领域的服务，定义[domain_realm](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#domain-realm)映射可能是有用的，以便客户端知道哪些主机属于哪个领域。但是，如果您的客户端和KDC正在运行1.7或更高版本，那么在客户端机器上保留此部分，只需将其定义在KDC的krb5.conf中即可。

### 1.1.3 UNIX应用服务器

应用服务器是通过网络提供一个或多个服务的主机。应用服务器可以是“安全的”或“不安全的”。安全的主机被设置为要求从连接到它的每个客户端进行认证。“不安全”的主机仍将提供Kerberos身份验证，但也允许未经身份验证的客户端进行连接。

如果您在所有客户机上安装了Kerberos V5，MIT建议您使主机安全，以利用Kerberos身份验证提供的安全性。但是，如果某些客户端没有安装Kerberos V5，则可以运行不安全的服务器，并且还可以利用Kerberos V5的单点登录功能。

#### 1.1.3.1 keytab文件

所有Kerberos服务器计算机都需要一个keytab文件才能向KDC进行身份验证。默认情况下，在类UNIX系统上，此文件名为[DEFKTNAME](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)。keytab文件是主机密钥的本地副本。keytab文件是入侵的潜在入口点，如果被破坏，将允许不受限制地访问其主机。keytab文件只能由root读取，只能存在于本机的本地磁盘上。该文件不应该是机器的任何备份的一部分，除非访问备份数据与访问机器的root密码一样紧密。

为了生成主机的密钥表，主机必须在Kerberos数据库中具有一个主体。在主机的[添加，修改和删除](http://web.mit.edu/kerberos/krb5-latest/doc/admin/database.html#add-mod-del-princs)过程中，对主机数据库的添加过程进行了详细描述。（请参阅 [为从属KDC创建主机](http://web.mit.edu/kerberos/krb5-latest/doc/admin/install_kdc.html#slave-host-key)密钥表进行简要说明。）keytab是通过运行[kadmin](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmin_local.html#kadmin-1)并发出[ktadd](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmin_local.html#ktadd) 命令生成的。

例如，要生成一个密钥表文件，以允许主机 `trillium.mit.edu`为服务主机，FTP，和流行，管理员认证`joeadmin`会发出命令（在 `trillium.mit.edu`）：

```
trillium% kadmin
kadmin5: ktadd host/trillium.mit.edu ftp/trillium.mit.edu
    pop/trillium.mit.edu
kadmin: Entry for principal host/trillium.mit.edu@ATHENA.MIT.EDU with
    kvno 3, encryption type DES-CBC-CRC added to keytab
    FILE:/etc/krb5.keytab.
kadmin: Entry for principal ftp/trillium.mit.edu@ATHENA.MIT.EDU with
    kvno 3, encryption type DES-CBC-CRC added to keytab
    FILE:/etc/krb5.keytab.
kadmin: Entry for principal pop/trillium.mit.edu@ATHENA.MIT.EDU with
    kvno 3, encryption type DES-CBC-CRC added to keytab
    FILE:/etc/krb5.keytab.
kadmin5: quit
trillium%
```

#### 1.1.3.2关于安全主机的一些建议

Kerberos V5可以保护您的主机免受某些类型的入侵攻击，但是可以安装Kerberos V5，并且仍然使您的主机容易受到攻击。显然，安装指南不是试图为每一次可能的攻击列出一个详尽的反措施清单，但值得注意的是一些较大的漏洞和如何关闭它们。

我们建议安全机器的备份排除keytab文件（[DEFKTNAME](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)）。如果不可能，备份应至少在本地完成，而不是通过网络完成，并且备份磁带应该是物理上安全的。

keytab文件和由root运行的任何程序（包括Kerberos V5二进制文件）应保存在本地磁盘上。keytab文件只能由root读取。

## 1.2 配置文件

Kerberos使用配置文件来允许管理员在每个机器的基础上指定设置。 [krb5.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#krb5-conf-5)适用于使用Kerboros库的所有应用程序，在客户端和服务器上。对于KDC特定应用程序，可以在[kdc.conf中](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)指定其他设置 ; 这两个文件被合并到直接访问KDC数据库的应用程序使用的配置文件。 [kadm5.acl](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kadm5_acl.html#kadm5-acl-5) 也仅用于KDC，它控制修改KDC数据库的权限。

### 1.2.1 krb5.conf

krb5.conf文件包含Kerberos配置信息，包括感兴趣的Kerberos领域的KDC和管理服务器的位置，默认为当前领域和Kerberos应用程序，并将主机名映射到Kerberos领域。通常，您应该将krb5.conf文件安装在`/ etc`目录中 。您可以通过设置环境变量**KRB5_CONFIG**覆盖默认位置。可以在**KRB5_CONFIG中**指定多个冒号分隔的文件名; 所有存在的文件将被读取。从版本1.14开始，目录名也可以在**KRB5_CONFIG中**指定; 名称中仅包含字母数字字符，破折号或下划线的目录中的所有文件将被读取。

#### 1.2.1.1 结构

krb5.conf文件以Windows INI文件的样式设置。部分由节名称以方括号表示。每个部分可能包含零个或多个关系，形式如下：

```
foo = bar
```

或者：

```
fubar = {
    foo = bar
    baz = quux
}
```

在一行的末尾放置“\*”表示这是 标签的*最终*值。这意味着，此配置文件的其余部分和任何其他配置文件都不会检查此标签的任何其他值。

例如，如果你有以下几行：

```
foo = bar*
foo = baz
```

那么`foo`（`baz`）的第二个值将永远不会被读取。

krb5.conf文件可以包括在一行开头使用以下指令之一的其他文件：

```
include FILENAME
includedir DIRNAME
```

*FILENAME*或*DIRNAME*应该是绝对路径。命名的文件或目录必须存在并且可读。包括一个目录包括目录中的所有文件，其名称仅由字母数字字符，破折号或下划线组成。从版本1.15开始，名称以“.conf”结尾的文件也包括在内，除非名称以“.”开头。包含的配置文件在语法上独立于其父级，因此每个包含的文件必须以段头开头。

krb5.conf文件可以指定从可加载模块而不是文件本身获取配置，使用以下指令在任何段头之前的行开头：

```
module MODULEPATH:RESIDUAL
```

*MODULEPATH*可能相对于krb5安装的库路径，也可能是绝对路径。 在初始化时间*内向*模块提供*RESIDUAL*。如果krb5.conf使用模块指令，那么[kdc.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)也应该使用一个。

#### 1.2.1.2部分

|                                          |                        |
| ---------------------------------------- | ---------------------- |
| [libdefaults](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#libdefaults) | Kerberos V5库使用的设置      |
| [realms](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#realms) | 领域特定的联系信息和设置           |
| [domain_realm](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#domain-realm) | 将服务器主机名映射到Kerberos领域   |
| [capaths](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#capaths) | 非层次交叉领域的认证路径           |
| [appdefaults](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#appdefaults) | 某些Kerberos V5应用程序使用的设置 |
| [plugins](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#plugins) | 控制插件模块注册               |

另外，krb5.conf可能包含 [kdc.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5) 中描述的任何关系 ，但不是推荐的做法。

##### [libdefaults]

libdefaults部分可能包含以下任何关系：

- **allow_weak_crypto**

如果这个标志被设置为false，那么弱加密类型（如指出的[加密类型](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#encryption-types)在[kdc.conf中](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)）将被过滤掉的名单**default_tgs_enctypes**， **default_tkt_enctypes**和**permitted_enctypes**。此标记的默认值为false，这可能会导致现有Kerberos基础结构中的身份验证失败，该基础架构不支持强密码。受影响环境中的用户应将此标记设置为true，直到其基础架构采用更强的密码。

- **ap_req_checksum_type**

一个整数，用于指定在认证者中使用的AP-REQ校验和的类型。该变量应该被取消设置，因此将使用正在使用的加密密钥的相应校验和。如果向后兼容性需要特定的校验和类型，则可以进行设置。有关可能的值及其含义，请参阅**kdc_req_checksum_type** 配置选项。

- **canonicalize**

如果此标志设置为true，则向KDC发出的初始票据请求将请求客户主体名称的规范化，并且将接受不同于客户主体的答复，而不是所请求的主体将被接受。默认值为false。

- **ccache_type**

此参数确定由[kinit](http://web.mit.edu/kerberos/krb5-latest/doc/user/user_commands/kinit.html#kinit-1)或其他程序创建的凭据高速缓存类型的格式。默认值为4，表示最新的格式。可以使用较小的值与与同一主机上的凭据缓存交互的非常旧的Kerberos实现的兼容性。

- **clockskew**

在假设Kerberos消息无效之前，设置库允许的最大允许数量的clockskew（以秒为单位）。默认值为300秒，或五分钟。

在评估票的开始和到期时间时，也会使用clockskew设置。例如，如果持续时间比**clockskew**设置更短，那么仍然可以使用到达其到期时间的票证（如果是可更新的票证，则更新）。

- **default_ccache_name**

此关系指定默认凭据缓存的名称。默认值为[DEFCCNAME](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)。此关系需要参数扩展（见下文）。新版本1.11。

- **default_client_keytab_name**

此关系指定用于获取客户端凭据的默认密钥表的名称。默认值为[DEFCKTNAME](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)。此关系需要参数扩展（见下文）。新版本1.11。

- **default_keytab_name**

此关系指定了应用程序服务器（如sshd）使用的默认密钥表名称。默认值为[DEFKTNAME](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)。此关系需要参数扩展（见下文）。

- **default_realm**

标识客户端的默认Kerberos领域。将其值设置为您的Kerberos领域。如果未设置此值，则在调用诸如[kinit的](http://web.mit.edu/kerberos/krb5-latest/doc/user/user_commands/kinit.html#kinit-1)程序时，必须为每个Kerberos主体指定一个域。

- **default_tgs_enctypes**

标识在制作TGS-REQ时客户端应该请求的会话密钥加密类型的支持列表，按照从最高到最低的优先顺序。列表可以用逗号或空格分隔。见[加密类型](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#encryption-types)在 [kdc.conf中](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)对这个标签接收到的值的列表。默认值为`aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96 aes128-cts-hmac-sha256-128 aes256-cts-hmac-sha384-192 des3-cbc-sha1 arcfour-hmac-md5 山茶花256 -cts-cmac camellia128 -cts-cmac des-cbc-crc des-cbc-md5 des-cbc-md4`，但是单DES加密类型将从该列表中隐式删除，如果 **allow_weak_crypto**为false。

除非具体的向后兼容性目的要求，否则不要设置此; 这种设置的陈旧值可以防止客户端在升级库时利用新的更强的enctypes。

- **default_tkt_enctypes**

标识在制作AS-REQ时客户端应该请求的会话密钥加密类型的支持列表，按照从最高到最低的优先顺序。格式与default_tgs_enctypes相同。该标签的默认值为 `aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96aes128-cts-hmac-sha256-128 aes256-cts-hmac-sha384-192 des3-cbc-sha1 arcfour- hmac-md5 camellia256-cts-cmac camellia128 -cts-cmac des-cbc-crc des-cbc-md5 des-cbc-md4`，但是如果**allow_weak_crypto的**值为false ，则单DES加密类型将从该列表中隐式删除。

除非具体的向后兼容性目的要求，否则不要设置此; 这种设置的陈旧值可以防止客户端在升级库时利用新的更强的enctypes。

- **dns_canonicalize_hostname**

指示名称查找是否用于规范化用于服务主体名称的主机名。将此标志设置为false可以通过减少对DNS的依赖来提高安全性，但这意味着短名称不会被规范化为完全限定的主机名。默认值为true。

- **dns_lookup_kdc**

指示是否应该使用DNS SRV记录来定位KDCs和其他服务器的领域，如果它们没有列在领域的krb5.conf信息中。（请注意，为了与kadmind联系，请注意，admin_server条目必须在krb5.conf域中，因为kadmin的DNS实现不完整。）

启用此选项会打开一种拒绝服务攻击，如果有人欺骗DNS记录并将您重定向到另一个服务器。然而，这并不比拒绝服务更糟糕，因为这个假的KDC将无法解码您发送的任何东西（除了没有加密数据的初始票据请求之外），假的KDC发送的任何东西都不会在没有验证的情况下被信任使用一些秘密，它不会知道。

- **dns_uri_lookup**

指示是否应该使用DNS URI记录来定位KDCs和其他服务器的领域，如果它们没有列在领域的krb5.conf信息中。如果没有找到URI记录，SRV记录将用作回退。默认值为true。新版本1.15。

- **err_fmt**

此关系允许自定义错误消息格式化。如果设置了一个值，错误消息将通过将％M的正常错误消息和％C的错误代码替换为格式。

- **extra_addresses**

这允许计算机使用多个本地地址，以便允许Kerberos在使用NAT的网络中工作，同时仍然使用受地址限制的故障单。地址应以逗号分隔的列表。如果**noaddresses**为true，则此选项不起作用 。

- **forwardable**

如果这个标志是真的，如果KDC允许，默认情况下，初始票可以转发。默认值为false。

- **ignore_acceptor_hostname**

当接受基于主机的服务主体的GSSAPI或krb5安全上下文时，忽略调用应用程序传递的任何主机名，并允许客户端对与服务名称和领域名称（如果给定）匹配的密钥表中的任何服务主体进行身份验证。此选项可以提高多宿主主机上的服务器应用程序的管理灵活性，但可能危及虚拟主机环境的安全性。默认值为false。1.10版新功能

- **k5login_authoritative**

如果此标志为真，则如果 存在[.k5login](http://web.mit.edu/kerberos/krb5-latest/doc/user/user_config/k5login.html#k5login-5)文件，则主体必须列在本地用户的k5login文件中才能被授予登录访问权限。如果此标志为假，即使存在k5login文件但未列出主体，仍可通过其他机制授予委托人登录访问权限。默认值为true。

- **k5login_directory**

如果设置，库将在命名目录中查找本地用户的k5login文件，文件名与本地用户名对应。如果未设置，库将在用户的主目录中查找k5login文件，文件名为.k5login。出于安全原因，.k5login文件必须由本地用户或root拥有。

- **kcm_mach_service**

仅在OS X上，确定KCM凭据高速缓存类型用于联系KCM守护程序的引导服务的名称。如果值为`-`，则不会使用Mach RPC与KCM守护程序联系。默认值为`org.h5l.kcm`。

- **kcm_socket**

确定用于访问KCM守护程序的KCM凭据缓存类型的Unix域套接字的路径。如果值为 `-`，则Unix域套接字将不会用于联系KCM守护程序。默认值为 `/var/run/.heim_org.h5l.kcm-socket`。

- **kdc_default_options**

请求初始票证的默认KDC选项（为多个值计数）。默认设置为0x00000010（KDC_OPT_RENEWABLE_OK）。

- **kdc_timesync**

该关系的接受值为1或0.如果不为零，客户端计算机将计算它们的时间与KDC在票证中的时间戳中返回的时间之间的差异，并在请求时使用此值来校正不准确的系统时钟服务票或认证服务。此修正因子仅由Kerberos库使用; 它不用于更改系统时钟。默认值为1。

- **kdc_req_checksum_type**

一个整数，用于指定用于KDC请求的校验和的类型，以便与很旧的KDC实现兼容。此值仅用于DES密钥; 其他键使用这些键的首选校验和类型。

可能的值及其含义如下。

|      |                         |
| ---- | ----------------------- |
| 1    | CRC32                   |
| 2    | RSA MD4                 |
| 3    | RSA MD4 DES             |
| 4    | DES CBC                 |
| 7    | RSA MD5                 |
| 8    | RSA MD5 DES             |
| 9    | NIST SHA                |
| 12   | HMAC SHA1 DES3          |
| -138 | Microsoft MD5 HMAC校验和类型 |


- **noaddresses**


如果此标志为真，则不会对设置的地址限制进行初始票证的请求，从而允许在NAT之间使用票证。默认值为true。

- **permitted_enctypes**


标识允许在会话密钥加密中使用的所有加密类型。该标签的默认值为 `aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96 aes128-cts-hmac-sha256-128 aes256-cts-hmac-sha384-192 des3-cbc-sha1 arcfour- hmac-md5 camellia256-cts-cmac camellia128 -cts-cmac des-cbc-crc des-cbc-md5 des-cbc-md4`，但是如果**allow_weak_crypto的**值为false ，则单DES加密类型将从该列表中隐式删除。

- **plugin_base_dir**


如果设置，则确定krb5插件所在的基本目录。默认值是krb5库目录的`krb5 / plugins`子目录。

- **preferred_preauth_types**


这允许您设置客户端将尝试的首选预认证类型，其他可能由KDC发布的其他类型。此设置的默认值为“17,16,15,14”，这将强制libkrb5尝试使用PKINIT（如果支持）。

- **proxiable**


如果这个标志是真的，如果KDC允许，默认情况下初始门票将可以启用。默认值为false。

- **rdns**


如果这个标志是真的，除了前向名称查找之外，还将使用反向名称查找来规范化用于服务主体名称的主机名。如果**dns_canonicalize_hostname**设置为false，则此标志无效。默认值为true。

- **realm_try_domains**


指示主机的域组件是否应用于确定主机的Kerberos领域。此变量的值为整数：-1表示不搜索，0表示尝试主机的域本身，1表示也尝试域的直接父项等等。库用于定位Kerberos领域的通常机制用于确定域是否是有效领域，如果设置了**dns_lookup_kdc**，则可能涉及到DNS的咨询。默认情况下不是搜索域组件。

- **renew_lifetime**


（[持续时间](http://web.mit.edu/kerberos/krb5-latest/doc/basic/date_format.html#duration)字符串）设置初始票据请求的默认可更新生命周期。默认值为0。

- **safe_checksum_type**


一个整数，指定用于KRB-SAFE请求的校验和的类型。默认设置为8（RSA MD5 DES）。为了与与DCE 1.1或更早版本的Kerberos库链接的应用程序兼容，请使用3的值来使用RSA MD4 DES。当该值与会话密钥类型不兼容时，该字段将被忽略。有关 可能的值及其含义，请参阅**kdc_req_checksum_type**配置选项。

- **ticket_lifetime**


（[持续时间](http://web.mit.edu/kerberos/krb5-latest/doc/basic/date_format.html#duration)字符串）设置初始票据请求的默认生命周期。默认值为1天。

- **udp_preference_limit**


当向KDC发送消息时，如果消息的大小高于**udp_preference_limit**，则库将尝试在UDP之前使用TCP 。如果消息小于 **udp_preference_limit**，则UDP将在TCP之前尝试。无论大小如何，如果第一次尝试失败，将尝试两个协议。

- **verify_ap_req_nofail**


如果此标志为真，则如果客户端计算机没有密钥表，则尝试验证初始凭据将失败。默认值为false。

- ​

##### [realms]

文件的[realms]部分中的每个标签都是Kerberos领域的名称。标签的值是具有定义该特定领域的属性的关系的子部分。对于每个领域，可以在领域的子部分中指定以下标签：

- **admin_server**


标识管理服务器正在运行的主机。通常，这是主Kerberos服务器。为了与 领域的[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)服务器进行通信，必须为此标签赋予一个值。

- **auth_to_local**


此标签允许您设置将主体名称映射到本地用户名的一般规则。如果正在翻译的主体名称没有显式映射，则将使用它。可能的值是：

​	**RULE**:*exp*

​	本地名称将从制定*EXP*。

​	对于格式*EXP*是*[ ***Ñ ***：***字符串***（***正则表达式***）S / ***图案***/ ***更换***/克**。整数*n*表示目标主体应具有多少个组件。如果此匹配，则一个字符串将被从形成*的字符串*，替换主要为境界`$ 0`和*Ñ* “日本金的成分 `$ N`（例如，如果主要是`输入johndoe /管理员`然后 `[2：$ 2 $ 1foo ]`将导致字符串 `adminjohndoefoo`）。如果此字符串与*regexp*匹配**，那么`s // [g]`替换命令将在字符串上运行。可选**克**将导致取代是全球性的过*串*，而不是在只更换第一匹配*串*。

​	**DEFAULT**

​	主体名称将用作本地用户名。如果主体具有多个组件或不在默认领域，则此规则不适用，转换将失败。

例如：

```
[realms]
    ATHENA.MIT.EDU = {
        auth_to_local = RULE:[2:$1](johndoe)s/^.*$/guest/
        auth_to_local = RULE:[2:$1;$2](^.*;admin$)s/;admin$//
        auth_to_local = RULE:[2:$2](^.*;root)s/^.*$/root/
        auto_to_local = DEFAULT
    }
```

将导致任何没有`root`或`admin的`主体作为使用默认规则进行翻译的第二个组件。`管理员`的第二个组成部分的主体将成为其第一个组成部分。 `root`将被用作具有`root`的第二个组件的任何主体的本地名称。这两个规则的例外是任何主体`johndoe / *`，它将始终获取本地名称`guest`。

- **auth_to_local_names**


本小节允许您将主体名称的显式映射设置为本地用户名。标签是映射名称，值是相应的本地用户名。

- **default_domain**


此标记指定在将Kerberos 4服务主体转换为Kerberos 5主体时（例如，将`rcmd.hostname`转换为 `host / hostname.domain`）时，用于扩展主机名的域。

- **http_anchors**


当通过HTTPS代理访问KDC和kpasswd服务器时，可以使用此标记来指定应信任的CA证书的位置，以发出代理服务器的证书。如果未指定，则使用系统级默认的CA证书集。

值的语法与**pkinit_anchors**标记的值相似 ：

​	**FILE:** *filename*

​	*filename*被认为是OpenSSL风格的ca-bundle文件的名称。

​	**DIR:** *dirname*

​	*dirname*被假定为包含CA证书的目录。目录中的所有文件将被检查; 如果它们包含证书（以PEM格式），则将被使用。

​	**ENV:** *envvar*

​	*envvar*指定已设置为符合先前值之一的值的环境变量的名称。例如， `ENV：X509_PROXY_CA`，其中环境变量`X509_PROXY_CA`已设置为`FILE：/tmp/my_proxy.pem`。

- **kdc**


为该领域运行KDC的主机的名称或地址。可以包括与冒号的主机名分隔的可选端口号。如果名称或地址包含冒号（例如，如果是IPv6地址），请将其括在方括号中以区分冒号和端口分隔符。为使您的计算机能够与每个领域的KDC进行通信，必须在配置文件中的每个领域中给予该标签一个值，或者必须有指定KDC的DNS SRV记录。

- **kpasswd_server**


指向执行所有密码更改的服务器。如果没有这样的条目，**admin_server** 主机上的端口464 将被尝试。

- **master_kdc**


标识主KDC（s）。目前，仅在一种情况下使用此标签：如果由于密码无效而尝试获取凭证失败，则客户端软件将尝试联系主KDC，以防用户密码刚刚更改，更新的数据库具有尚未传播到从属服务器。

- **v4_instance_convert**


此小节允许管理员配置**default_domain**映射规则的例外。它包含V4实例（标签名称），它应该转换为某个特定主机名（标记值）作为Kerberos V5主体名称中的第二个组件。

- **v4_realm**


当将V5主体名称转换为V4主体名称时，此关系由krb524库例程使用。当V4域名和V5域名不相同时使用，但仍然共享相同的主体名称和密码。标记值是Kerberos V4领域名称。

##### [domain_realm]

[domain_realm]部分提供从域名或主机名到Kerberos领域名称的翻译。标签名称可以是主机名或域名，其中域名由句点（`.`）的前缀指示。该关系的值是该特定主机或域的Kerberos域名。主机名关系隐含地提供相应的域名关系，除非提供了明确的域名关系。可以在[领域](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#realms)部分或使用DNS SRV记录来标识Kerberos领域。主机名和域名应小写。例如：

```
[domain_realm]
    crash.mit.edu = TEST.ATHENA.MIT.EDU
    .dev.mit.edu = TEST.ATHENA.MIT.EDU
    mit.edu = ATHENA.MIT.EDU
```

将名称为`crash.mit.edu`的主机映射到 `TEST.ATHENA.MIT.EDU`领域。第二个条目将域`dev.mit.edu`下的所有主机映射到`TEST.ATHENA.MIT.EDU`域中，而不是名为`dev.mit.edu`的主机。主机由第三项，它映射主机匹配`mit.edu`和域下的所有主机`mit.edu`不前的规则匹配到的境界`ATHENA.MIT.EDU`。

如果没有翻译条目适用于用于服务票证请求的服务主体的主机名，库将尝试从客户端领域的KDC获得适当领域的引用。如果这不成功，主机的领域被认为是转换为大写的主机名的域部分，除非[libdefaults]中的**realm_try_domains**设置导致使用不同的父域。

##### [capaths]

为了执行直接（非分层）交叉领域认证，需要进行配置来确定领域之间的认证路径。

客户端将使用此部分来查找其领域和服务器领域之间的认证路径。服务器将使用此部分来验证客户端使用的身份验证路径，方法是检查接收到的故障单的转接字段。

每个参与的客户端都有一个标签，每个标签都有每个服务器领域的子标签。子标签的值是可以参与跨域认证的中间域。如果存在多于一个中间领域，则可能会重复子标签。值“。”表示两个领域直接共享密钥，不允许中间域参与。

只有客户端或服务器上需要的那些条目才需要存在。一个客户端需要一个标签为其本地领域与子标签所有的服务器领域将需要验证。服务器领域的子标签需要一个服务器的每个领域的标签。

例如，`ANL.GOV`，`PNL.GOV`和`NERSC.GOV`都希望使用`ES.NET`领域作为中间领域。ANL具有`TEST.ANL.GOV`的子`域`，它将使用`NERSC.GOV进行`认证， 但不会`PNL.GOV`。`ANL.GOV`系统的[capaths]部分将如下所示：

```
[capaths]
    ANL.GOV = {
        TEST.ANL.GOV = .
        PNL.GOV = ES.NET
        NERSC.GOV = ES.NET
        ES.NET = .
    }
    TEST.ANL.GOV = {
        ANL.GOV = .
    }
    PNL.GOV = {
        ANL.GOV = ES.NET
    }
    NERSC.GOV = {
        ANL.GOV = ES.NET
    }
    ES.NET = {
        ANL.GOV = .
    }
```

`NERSC.GOV` 系统上使用的配置文件的[capaths]部分将如下所示：

```
[capaths]
    NERSC.GOV = {
        ANL.GOV = ES.NET
        TEST.ANL.GOV = ES.NET
        TEST.ANL.GOV = ANL.GOV
        PNL.GOV = ES.NET
        ES.NET = .
    }
    ANL.GOV = {
        NERSC.GOV = ES.NET
    }
    PNL.GOV = {
        NERSC.GOV = ES.NET
    }
    ES.NET = {
        NERSC.GOV = .
    }
    TEST.ANL.GOV = {
        NERSC.GOV = ANL.GOV
        NERSC.GOV = ES.NET
    }
```

当标记中使用子标签多于一次时，客户端将使用值的顺序来确定路径。值的顺序对于服务器来说并不重要。

##### [appdefaults] 

[appdefaults]部分中的每个标签命名一个Kerberos V5应用程序或某些Kerberos V5应用程序使用的选项。标签的值定义了该应用程序的默认行为。

例如：

```
[appdefaults]
    telnet = {
        ATHENA.MIT.EDU = {
            option1 = false
        }
    }
    telnet = {
        option1 = true
        option2 = true
    }
    ATHENA.MIT.EDU = {
        option2 = false
    }
    option2 = true
```

指定选项值的上述四种方式按优先级递减的顺序显示。在这个例子中，如果telnet在领域EXAMPLE.COM中运行，默认情况下应该将option1和option2设置为true。然而，`ATHENA.MIT.EDU`领域的telnet程序 应该将`option1`设置为false，将 `option2`设置为true。默认情况下，ATHENA.MIT.EDU中的任何其他程序应该将`option2`设置为false。任何在其他领域运行的程序都应该将`option2`设置为true。

每个应用程序的可指定选项的列表可以在该应用程序的手册页中找到。此处指定的应用程序默认值将由[领域](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#realms)部分中指定的应用程序覆盖。

##### [plugins]

 - [pwqual](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#pwqual) interface
 - [kadm5_hook](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#kadm5-hook) interface
 - [clpreauth](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#clpreauth) and [kdcpreauth](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#kdcpreauth) interfaces


#### 1.2.1.3 参数扩展

从版本1.11开始，几个变量（如 **default_keytab_name**）允许扩展参数。有效参数有：

|                   |                           |
| ----------------- | ------------------------- |
| %{TEMP}           | 临时目录                      |
| %{uid}            | Unix真正的UID或Windows SID    |
| %{euid}           | Unix有效的用户ID或Windows SID   |
| %{USERID}         | 与％{uid}相同                 |
| %{null}           | 空字符串                      |
| %{LIBDIR}         | 安装库目录                     |
| %{BINDIR}         | 安装二进制目录                   |
| %{SBINDIR}        | 安装admin二进制目录              |
| %{username}       | （Unix）有效用户名的用户名           |
| %{APPDATA}        | （Windows）当前用户的漫游应用程序数据    |
| %{COMMON_APPDATA} | （Windows）所有用户的应用程序数据      |
| %{LOCAL_APPDATA}  | （Windows）当前用户的本地应用程序数据    |
| %{SYSTEM}         | （Windows）Windows系统文件夹     |
| %{WINDOWS}        | （Windows）Windows系统文件夹     |
| %{USERCONFIG}     | （Windows）Windows系统文件夹     |
| %{COMMONCONFIG}   | （Windows）常用MIT krb5配置文件目录 |


#### 1.2.1.4 示例krb5.conf文件

以下是通用krb5.conf文件的示例：

```
[libdefaults]
    default_realm = ATHENA.MIT.EDU
    dns_lookup_kdc = true
    dns_lookup_realm = false

[realms]
    ATHENA.MIT.EDU = {
        kdc = kerberos.mit.edu
        kdc = kerberos-1.mit.edu
        kdc = kerberos-2.mit.edu
        admin_server = kerberos.mit.edu
        master_kdc = kerberos.mit.edu
    }
    EXAMPLE.COM = {
        kdc = kerberos.example.com
        kdc = kerberos-1.example.com
        admin_server = kerberos.example.com
    }

[domain_realm]
    mit.edu = ATHENA.MIT.EDU

[capaths]
    ATHENA.MIT.EDU = {
           EXAMPLE.COM = .
    }
    EXAMPLE.COM = {
           ATHENA.MIT.EDU = .
    }
```



### 1.2.2 kdc.conf

kdc.conf文件补充了通常仅在KDC上使用的程序的[krb5.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#krb5-conf-5)，例如[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)和 [kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)守护程序以及[kdb5_util](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kdb5_util.html#kdb5-util-8)程序。这里记录的关系也可以在krb5.conf中指定; 对于所提到的KDC程序，krb5.conf和kdc.conf将被合并到单个配置文件中。

通常，kdc.conf文件位于KDC状态目录 [LOCALSTATEDIR ](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)* `/ krb5kdc中`。您可以通过设置环境变量**KRB5_KDC_PROFILE**来覆盖默认位置。

请注意，您需要重新启动KDC守护程序，以使任何配置更改生效。

#### 1.2.2.1 结构

kdc.conf文件的设置格式与[krb5.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#krb5-conf-5)文件相同 。

#### 1.2.2.2 部分

kdc.conf文件可能包含以下部分：

|                                          |                         |
| ---------------------------------------- | ----------------------- |
| [kdcdefaults](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdcdefaults) | KDC行为的默认值               |
| [realms](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-realms) | 领域特定的数据库配置和设置           |
| [dbdefaults](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#dbdefaults) | 默认数据库设置                 |
| [dbmodules](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#dbmodules) | 每数据库设置                  |
| [logging](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#logging) | 控制Kerberos守护程序执行日志记录的方式 |


##### [kdcdefaults]

除了两个例外，[kdcdefaults]部分中的关系指定了领域变量的默认值，如果[realms]部分不包含标签的关系，则使用它们。有关这些关系的定义，请参阅 [领域](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-realms) 部分。

- **host_based_services**
- **kdc_listen**
- **kdc_ports**
- **kdc_tcp_listen**
- **kdc_tcp_ports**
- **no_host_referral**
- **restrict_anonymous_to_tgt**



- **kdc_max_dgram_reply_size**

指定可以通过UDP发送的最大数据包大小。默认值为4096字节。

- **kdc_tcp_listen_backlog**

（整数）设置KDC守护程序的侦听队列长度的大小。该值可能受到OS设置的限制。默认值为5。

##### [realms]

[realms]部分中的每个标签都是Kerberos领域的名称。标签的值是关系为该特定领域定义KDC参数的子部分。以下示例显示如何为ATHENA.MIT.EDU领域定义一个参数：

```
[realms]
    ATHENA.MIT.EDU = {
        max_renewable_life = 7d 0h 0m 0s
    }
```

以下标签可能在[域}部分中指定：

- **acl_file**

（String。）[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)用于确定允许哪些主体在Kerberos数据库中有哪些权限的访问控制列表文件的位置 。默认值为 [LOCALSTATEDIR ](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)* `/krb5kdc/kadm5.acl`。有关Kerberos ACL文件的更多信息，请参阅[kadm5.acl](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kadm5_acl.html#kadm5-acl-5)。

- **database_module**

（String。）此关系指示可加载数据库使用的数据库特定参数的[dbmodules](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#dbmodules)下的配置部分的名称。默认值是域名。如果此配置部分不存在，则将为所有数据库参数使用默认值。

- **database_name**

（String，deprecated。）此关系指定了该领域的Kerberos数据库的位置，如果正在使用DB2模块，并且[dbmodules](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#dbmodules)配置部分未指定数据库名称。默认值为[LOCALSTATEDIR ](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)* `/krb5kdc/principal`。

- **default_principal_expiration**

（[绝对时间](http://web.mit.edu/kerberos/krb5-latest/doc/basic/date_format.html#abstime)字符串。）指定在此领域中创建的主体的默认有效期。默认值为0，表示没有到期日。

- **default_principal_flags**

（标志字符串。）指定在此领域中创建的主体的默认属性。该字符串的格式是逗号分隔的标志列表，在应该被禁用的每个标志之前的每个标志之前使用'+'和' - '。该**postdateable**，**forwardable**，**tgt-based**， **renewable**，**proxiable**，**dup-skey**，**allow-tickets**，和 **service** 标志默认为启用。

有几个可能的标志：

​	**allow-tickets**

​	启用此标志表示KDC将颁发此委托人的门票。禁用此标志基本上禁用此领域内的主体。

​	**dup-skey**

​	启用此标志允许主体获取另一个用户的会话密钥，允许该主体的用户到用户身份验证。

​	**forwardable**

​	启用此标志允许校长获得可转发的门票。

​	**hwauth**

​	如果启用此标志，则在接收任何票据之前，必须使用硬件设备对主体进行预认证。

​	**no-auth-data-required**

​	启用此标志可防止将PAC或AD-SIGNEDPATH数据添加到主体的服务单。

​	**ok-as-delegate**

​	如果此标志已启用，则会提示客户端在对服务进行身份验证时可以委任凭据。

​	**ok-to-auth-as-delegate**

​	启用此标志允许校长使用S4USelf门票。

​	**postdateable**

​	启用此标志允许校长获得可更新的门票。

​	**preauth**

​	如果在客户机主体上启用了此标志，那么在接收任何票据之前，需要该主体对KDC进行预认证。在服务主体上，启用此标志意味着此主体的服务票据将仅发送给具有预验证位的TGT的客户端。

​	**proxiable**

​	启用此标志允许主体获取代理机票。

​	**pwchange**

​	启用此标志将强制更改此主体的密码。

​	**pwservice**

​	如果启用此标志，则将该主体标记为密码更改服务。这只能在特殊情况下使用，例如，如果用户的密码已经过期，则用户必须通过正常密码认证才能获得该主体的票据，以便能够更改密码。

​	**renewable**

​	启用此标志允许校长获得可更新的门票。

​	**service**

​	启用此标志允许KDC颁发此主体的服务票证。

​	**tgt-based**

​	启用此标志允许主体基于票证授予票而获取票据，而不是重复用于获取TGT的身份验证过程。

- **dict_file**

（String。）包含不允许作为密码的字符串的字典文件的位置。该文件应该每行包含一个字符串，而不需要额外的空格。如果没有指定或者没有分配给主体的策略，则不会执行字典检查密码。

- **host_based_services**

（空格或逗号分隔的列表。）列出即使服务器主体未被客户端标记为主机的服务，这些服务将获得基于主机的引用处理。

- **iprop_enable**

（布尔值。）指定是否启用增量数据库传播。默认值为false。

- **iprop_master_ulogsize**

（整数）指定要为增量传播保留的最大日志条目数。默认值为1000.在版本1.11之前，最大值为2500。

- **iprop_slave_poll**

（Delta时间字符串。）指定从机KDC轮询主机的新更新的频率。默认值为`2m`（即两分钟）。

- **iprop_listen**

（Whitespace-或逗号分隔的列表。）指定iprop RPC监听地址和/或端口的[kadmind的](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)守护进程。每个条目可能是一个接口地址，一个端口号，或一个以冒号分隔的地址和端口号。如果地址包含冒号，请将其括在方括号中。如果没有指定地址，则使用通配符地址。如果kadmind无法绑定到任何指定的地址，它将无法启动。默认值（当**iprop_enable**为true时）将绑定到**iprop_port中**指定的端口上的通配符地址。新版本1.15。

- **iprop_port**

（端口号。）指定用于增量传播的端口号。当**iprop_enable**为true时，从配置文件需要此关系，主配置文件中需要此关系或**iprop_listen**，因为没有默认端口号。**iprop_listen**条目中指定的端口号将覆盖[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)守护程序的此端口号。

- **iprop_resync_timeout**

（Delta时间字符串。）指定等待完整传播完成的时间量。这在配置文件中是可选的，仅由从属KDC使用。默认值为5分钟（`5米`）。新版本1.11。

- **iprop_logfile**

（文件名。）指定要存储领域数据库的更新日志文件的位置。默认是使用 **数据库名称**从KRB5配置文件的realms部分的条目，以`.ulog`追加。（注意：如果在“领域”部分中未指定**database_name**，可能是因为正在使用LDAP数据库后端，或者在[dbmodules]部分中指定了文件名，则会使用**database_name**的硬编码默认值 。的**iprop_logfile** 默认值不会使用[dbmodules]部分的值。）

- **kadmind_listen**

（空格或逗号分隔的列表。）指定[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)守护程序的kadmin RPC侦听地址和/或端口。每个条目可能是一个接口地址，一个端口号，或一个以冒号分隔的地址和端口号。如果地址包含冒号，请将其括在方括号中。如果没有指定地址，则使用通配符地址。如果kadmind无法绑定到任何指定的地址，它将无法启动。默认值是绑定到**kadmind_port**或标准kadmin端口（749）中指定的端口上的通配符地址。新版本1.15。

- **kadmind_port**

（端口号。）指定[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8) 守护程序侦听此域的端口。在**kadmind_listen**条目中指定的端口号 将覆盖此端口号。kadmind的分配端口是749，默认情况下使用。

- **key_stash_file**

（String。）指定存储主密钥的位置（通过kdb5_util隐藏）。默认值为[LOCALSTATEDIR](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)`/ krb5kdc/.k5.REALM`，其中*REALM*是Kerberos *域*。

- **kdc_listen**

（空格或逗号分隔的列表。）指定[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)守护程序的UDP侦听地址和/或端口。每个条目可能是一个接口地址，一个端口号，或一个以冒号分隔的地址和端口号。如果地址包含冒号，请将其括在方括号中。如果没有指定地址，则使用通配符地址。如果没有指定端口，则使用标准端口（88）。如果KDC守护程序无法绑定到任何指定的地址，将无法启动。默认是绑定到标准端口上的通配符地址。新版本1.15。

- **kdc_ports**

（不推荐使用空格或逗号分隔的列表）。在释放1.15之前，此关系列出了[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)守护程序侦听UDP请求的端口 。在版本1.15及更高版本中， 如果没有定义该关系，它与**kdc_listen**具有相同的含义。

- **kdc_tcp_listen**

（空格或逗号分隔的列表。）指定[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)守护程序的TCP侦听地址和/或端口。每个条目可能是一个接口地址，一个端口号，或一个以冒号分隔的地址和端口号。如果地址包含冒号，请将其括在方括号中。如果没有指定地址，则使用通配符地址。如果没有指定端口，则使用标准端口（88）。要禁用TCP侦听，请使用`kdc_tcp_listen = “”`将此关系设置为空字符串。如果KDC守护程序无法绑定到任何指定的地址，将无法启动。默认是绑定到标准端口上的通配符地址。新版本1.15。

- **kdc_tcp_ports**

（不推荐使用空格或逗号分隔的列表）。在释放1.15之前，此关系列出了[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)守护程序侦听UDP请求的端口 。在版本1.15及更高版本中，如果没有定义该关系，则与**kdc_tcp_listen**具有相同的含义。

- **kpasswd_listen**

（逗号分隔列表。）指定[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)守护程序的kpasswd侦听地址和/或端口。每个条目可能是一个接口地址，一个端口号，或一个以冒号分隔的地址和端口号。如果地址包含冒号，请将其括在方括号中。如果没有指定地址，则使用通配符地址。如果kadmind无法绑定到任何指定的地址，它将无法启动。默认值是绑定到**kpasswd_port中**指定的端口或标准kpasswd端口（464）上的通配符地址。新版本1.15。

- **kpasswd_port**

（端口号。）指定[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8) 守护程序侦听此域的密码更改请求的端口。在**kpasswd_listen**条目中指定的端口号将覆盖此端口号。密码更改请求的分配端口为464，默认使用。

- **master_key_name**

（String。）指定与主键相关联的主体的名称。默认值是`K/M`。

- **master_key_type**

（键类型字符串）指定主键的键类型。其默认值为`aes256-cts-hmac-sha1-96`。有关所有可能值的列表，请参阅[加密类型](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#encryption-types)。

- **max_life**

（[持续时间](http://web.mit.edu/kerberos/krb5-latest/doc/basic/date_format.html#duration)字符串）指定票证在该领域中有效的最长时间段。默认值为24小时。

- **max_renewable_life**

[持续时间](http://web.mit.edu/kerberos/krb5-latest/doc/basic/date_format.html#duration)字符串）指定在此领域可以更新有效票证的最长时间段。默认值为0。

- **no_host_referral**

（布尔值）。如果设置为true，则KDC将假定服务主体支持des-cbc-crc以进行会话密钥协商。如果**allow_weak_crypto**在[libdefaults](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#libdefaults)是假的，或者如果DES-CBC-CRC没有被允许的加密类型，则该变量没有效果。默认为true。新版本1.11。

- **reject_bad_transit**

（布尔值）如果设置为true，KDC将根据从其[krb5.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/krb5_conf.html#krb5-conf-5)文件的领域名称和capaths部分计算的传输路径，检查跨领域门票的转换范围 列表; 如果要颁发的票证中的路径包含不在计算路径中的任何领域，则不会发出票证，并将错误返回给客户端。如果这个值设置为false，那么这样的票据将会被发出，并且将被留给应用服务器来验证领域的传输路径。

如果在传入请求中设置了禁用转移检查标志，则完全不执行此检查。使用 **reject_bad_transit**选项将导致这样的票据请求总是被拒绝。

此传输路径检查和配置文件选项目前仅适用于TGS请求。

默认值为true。

- **restrict_anonymous_to_tgt**

（布尔值）如果设置为true，则KDC将拒绝来自匿名主体的票单请求，而不是领域的票证授予服务。此选项允许启用匿名PKINIT以用作FAST装甲门票，而不允许对服务进行匿名身份验证。默认值为false。新版本1.9。

- **supported_enctypes**

（*键*列表：*salt* strings。）指定此领域的主体的默认键/盐组合。通过[kadmin](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmin_local.html#kadmin-1)创建的任何主体将具有这些类型的键。该标签的默认值为`aes256-cts-hmac-sha1-96：normal aes128-cts-hmac-sha1-96：normal des3-cbc-sha1：normal arcfour-hmac-md5：normal`。有关可能值的[列表](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#keysalt-lists)，请参阅[Keysalt列表](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#keysalt-lists)。

##### [dbdefaults]

[dbdefaults]部分指定某些数据库参数的默认值，如果[dbmodules]子部分不包含该标记的关系，则使用该值。有关这些关系的定义，请参阅[dbmodules](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#dbmodules)部分。

- **ldap_kerberos_container_dn**
- **ldap_kdc_dn**
- **ldap_kdc_sasl_authcid**
- **ldap_kdc_sasl_authzid**
- **ldap_kdc_sasl_mech**
- **ldap_kdc_sasl_realm**
- **ldap_kadmind_dn**
- **ldap_kadmind_sasl_authcid**
- **ldap_kadmind_sasl_authzid**
- **ldap_kadmind_sasl_mech**
- **ldap_kadmind_sasl_realm**
- **ldap_service_password_file**
- **ldap_servers**
- **ldap_conns_per_server**

##### [dbmodules]

[dbmodules]部分包含KDC数据库和数据库模块使用的参数。[dbmodules]部分中的每个标签都是由领域的**database_module**参数指定的Kerberos领域或部分名称 。以下示例显示如何为ATHENA.MIT.EDU领域定义一个数据库参数：

```
[dbmodules]
    ATHENA.MIT.EDU = {
        disable_last_success = true
    }
```

可以在[dbmodules]子部分中指定以下标签：

- **database_name**

此DB2特定标记指示数据库在文件系统中的位置。默认值为[LOCALSTATEDIR ](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)* `/krb5kdc/principal`。

- **db_library**

此标记指示可加载数据库模块的名称。该值应为DB2模块的`db2`和LDAP模块的`kldap`值。

- **disable_last_success**

如果设置为`true`，则将KDC更新抑制为需要进行身份验证的主要条目的“上次成功身份验证”字段。设置此标志可能会提高性能。（不需要预认证的主要条目从不更新“上一次成功认证”字段）。在1.9版中首先介绍。

- **disable_lockout**

如果设置为`true`，请将KDC更新抑制为需要进行身份验证的主体条目的“最终验证失败”和“失败密码尝试”字段。设置此标志可能会提高性能，但也会禁用帐户锁定。在1.9版中首先介绍。

- **ldap_conns_per_server**

此LDAP特定标记指示要为每个LDAP服务器维护的连接数。

- **ldap_kdc_dn** and **ldap_kadmind_dn**

这些特定于LDAP的标签表示绑定到LDAP服务器的默认DN。该[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)守护进程使用**ldap_kdc_dn**，而[kadmind的](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)守护程序和其他行政程序使用**ldap_kadmind_dn**。kadmind DN必须具有读取和写入LDAP数据库中的Kerberos数据的权限。除非**disable_lockout**和**disable_last_success**为true ，否则KDC DN必须具有相同的权限， 在这种情况下，它只需要具有读取Kerberos数据的权限。如果使用**ldap_kdc_sasl_mech**或**ldap_kadmind_sasl_mech**设置SASL机制，则忽略这些标记 。

- **ldap_kdc_sasl_mech** and **ldap_kadmind_sasl_mech**

这些特定于LDAP的标签指定在绑定到LDAP服务器时要使用的SASL机制（例如 `EXTERNAL`）。新版本1.13。

- **ldap_kdc_sasl_authcid** and **ldap_kadmind_sasl_authcid**

这些特定于LDAP的标签指定绑定到LDAP服务器时要使用的SASL身份验证身份。并不是所有的SASL机制都需要身份认证。如果SASL机制需要秘密（例如`DIGEST-MD5`的密码），这些标签还会确定**ldap_service_password_file**中隐藏密码的名称 。新版本1.13。

- **ldap_kdc_sasl_authzid** and **ldap_kadmind_sasl_authzid**

这些特定于LDAP的标签指定绑定到LDAP服务器时要使用的SASL授权标识。在大多数情况下，它们不需要指定。新版本1.13。

- **ldap_kdc_sasl_realm** and **ldap_kadmind_sasl_realm**

这些特定于LDAP的标签指定绑定到LDAP服务器时要使用的SASL领域。在大多数情况下，他们不需要设置。新版本1.13。

- **ldap_kerberos_container_dn**

此LDAP特定标记表示领域对象将位于的容器对象的DN。

- **ldap_servers**

此LDAP特定标记指示Kerberos服务器可以连接到的LDAP服务器的列表。LDAP服务器的列表是空格分隔的。LDAP服务器由LDAP URI指定。建议使用`ldapi：`或`ldaps：` URL连接到LDAP服务器。

- **ldap_service_password_file**

此LDAP特定标记指示包含**ldap_kdc_dn**和**ldap_kadmind_dn**对象的密码（由`kdb5_ldap_utilstashsrvpw`创建） 或SASL身份验证的 **ldap_kdc_sasl_authcid**或**ldap_kadmind_sasl_authcid**名称的文件。该文件必须保持安全。

- **unlockiter**

如果设置为`true`，则此DB2特定标记会导致迭代操作在处理每个主体时释放数据库锁。将此标志设置为`true`可以防止在大型数据库转储过程中KDC或kadmin操作的扩展阻塞。首先介绍1.13版本。



可以在[dbmodules]部分中直接指定以下标签，以控制从哪个数据库模块加载：

- **db_module_dir**

该标签控制插件系统查找数据库模块的位置。该值应该是绝对路径。

##### [logging]

[logging]部分指示[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)和 [kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)如何执行日志记录。它可能包含以下关系：

- **admin_server**

指定[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)如何执行日志记录。

- **kdc**

指定[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)如何执行日志记录。

- **default**

指定守护进程如何在没有特定于守护进程的关系的情况下执行日志记录。

- **debug**

（布尔值。）指定调试消息是否包含在SYSLOG以外的日志输出中。调试消息始终包含在系统日志输出中，因为syslog执行自己的优先级过滤。默认值为false。新版本1.15。



测井规格可能有以下形式：

- **FILE=***filename* or **FILE:***filename*

此值将导致守护程序的日志消息转到 *文件名*。如果使用`=`表单，文件将被覆盖。如果使用`：`表单，该文件将附加到。

- **STDERR**

此值会导致守护程序的日志消息进入其标准错误流。

- **CONSOLE**

此值会导致守护程序的日志消息进入控制台，如果系统支持该消息。

- **DEVICE=\<devicename>**

这将导致守护程序的日志消息进入指定的设备。

- **SYSLOG[:*severity[:*facility]*]


这将导致守护程序的日志消息进入系统日志。

severity参数指定系统日志消息的默认严重性。这可能是syslog（3）调用支持的以下任何严重性，减去`LOG_`前缀：**EMERG**， **ALERT**，**CRIT**，**ERR**，**WARNING**，**NOTICE**，**INFO**和**DEBUG**。

facility参数指定记录消息的工具。这可能是syslog（3）调用减去LOG_前缀支持的以下任何设备：**KERN**， **USER**，**MAIL**，**DAEMON**，**AUTH**，**LPR**，**NEWS**， **UUCP**，**CRON**和**LOCAL0**到**LOCAL7**。

如果没有指定严重性，默认值为**ERR**。如果没有指定工具，默认值为**AUTH**。

在以下示例中，来自KDC的日志消息将转到控制台和设备LOG_DAEMON下的系统日志，默认严重性为LOG_INFO; 并且来自管理服务器的记录消息将附加到文件 `/var/adm/kadmin.log`并发送到设备`/dev/tty04`。

```
[logging]
    kdc = CONSOLE
    kdc = SYSLOG:INFO:DAEMON
    admin_server = FILE:/var/adm/kadmin.log
    admin_server = DEVICE=/dev/tty04
```

##### [otp]

[otp]的每个小节是OTP令牌类型的名称。小节内的标签定义了将一次性密码请求转发到RADIUS服务器所需的配置。

对于每个令牌类型，可以指定以下标签：

- **server**

这是发送RADIUS请求的服务器。它可以是可选端口的主机名，可选端口的IP地址或Unix域套接字地址。默认值为 [LOCALSTATEDIR](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)`/ krb5kdc/<name>.socket`。

- **secret**


此标记表示包含用于加密RADIUS数据包的秘密的文件名（可能与[LOCALSTATEDIR](http://web.mit.edu/kerberos/krb5-latest/doc/mitK5defaults.html#paths)`/ krb5kdc有关`）。秘密应该自己出现在文件的第一行; 线上的前导和尾随空格将被删除。如果**服务器的**值为Unix域套接字地址，则此标记是可选的，如果未指定，则将使用空的秘密。否则，此标记是必需的。

- **timeout**


一个整数，指定KDC应尝试联系RADIUS服务器的时间（以秒为单位）。此标记是所有重试的总时间，应小于OTP值保持有效的时间。默认值为5秒。

- **retries**


此标签指定要对RADIUS服务器进行重试次数。默认是3次重试（4次尝试）。

- **strip_realm**


如果此标记为`true`，则没有领域的主体将被传递到RADIUS服务器。否则，将包括这个领域。默认值为`true`。

- **indicator**


如果使用该标记类型进行身份验证，则此标签指定要包含在故障单中的认证指示符。此选项可以多次指定。（新版本1.14。）

在以下示例中，请求通过UDP发送到远程服务器：

```conf
[otp]
    MyRemoteTokenType = {
        server = radius.mydomain.com:1812
        secret = SEmfiajf42$
        timeout = 15
        retries = 5
        strip_realm = true
    }
```

`默认`值为`DEFAULT的`默认令牌类型是为每个主配置未指定令牌类型而定义的。其配置如下所示。您可以将此令牌类型覆盖适用于您的情况：

```conf
[otp]
    DEFAULT = {
        strip_realm = false
    }
```

#### 1.2.2.3加密类型

需要加密类型列表的配置文件中的任何标签都可以设置为以下字符串的某种组合。标记为“弱”的加密类型可用于兼容性，但不推荐使用。

| -                                        | -                                        |
| ---------------------------------------- | ---------------------------------------- |
| des-cbc-crc                              | DES cbc模式与CRC-32（弱）                      |
| des-cbc-md4                              | DES cbc模式与RSA-MD4（弱）                     |
| des-cbc-md5                              | DES cbc模式与RSA-MD5（弱）                     |
| des-cbc-raw                              | DES cbc模式raw（weak）                       |
| des3-cbc-raw                             | 三重DES cbc模式（弱）                           |
| des3-cbc-sha1 des3-hmac-sha1 des3-cbc-sha1-kd | 三重DES cbc模式与HMAC / sha1                  |
| des-hmac-sha1                            | DES与HMAC / sha1（弱）                       |
| aes256-cts-hmac-sha1-96 aes256-cts aes256-sha1 | 具有96位SHA-1 HMAC的AES-256 CTS模式            |
| aes128-cts-hmac-sha1-96 aes128-cts aes128-sha1 | 具有96位SHA-1 HMAC的AES-128 CTS模式            |
| aes256-cts-hmac-sha384-192 aes256-sha2   | 具有192位SHA-384 HMAC的AES-256 CTS模式         |
| aes128-cts-hmac-sha256-128 aes128-sha2   | 具有128位SHA-256 HMAC的AES-128 CTS模式         |
| arcfour-hmac rc4-hmac arcfour-hmac-md5   | RC4与HMAC / MD5                           |
| arcfour-hmac-exp rc4-hmac-exp arcfour-hmac-md5-exp | Exportable RC4 with HMAC/MD5 (weak)      |
| camellia256-cts-cmac camellia256-cts     | 可导出的RC4与HMAC / MD5（弱）                    |
| camellia128-cts-cmac camellia128-cts     | Camellia-128 CTS模式与CMAC                  |
| des                                      | DES系列：des-cbc-crc，des-cbc-md5和des-cbc-md4（弱） |
| des3                                     | 三重DES系列：des3-cbc-sha1                    |
| aes                                      | TAES系列：aes256-cts-hmac-sha1-96，aes128-cts-hmac-sha1-96，aes256-cts-hmac-sha384-192和aes128-cts-hmac-sha256-128 |
| rc4                                      | RC4系列：arcfour-hmac                       |
| camellia                                 | Camellia 系列: camellia256-cts-cmac 和camellia128-cts-cmac |


字符串**DEFAULT**可用于引用所讨论变量的默认类型。可以从当前列表中删除类型或族，前缀为减号（“ - ”）。类型或家庭可以加上加号（“+”）作为对称性; 它具有与列出类型或家族相同的含义。例如，“ `DEFAULT-des` ”将是删除DES类型的默认加密类型集，“ `des3 DEFAULT` ”将是默认的三重DES类型的加密类型移动到前端。

虽然所有Kerberos操作都支持**aes128-cts**和**aes256-cts**，但是我们的GSSAPI实现（krb5-1.3.1及更早版本）的旧版本不支持它们。不支持AES支持的运行krb5版本的服务不得在KDC数据库中给出这些加密类型的密钥。

该**AES128-SHA2**和**AES256-SHA2**加密类型，脱模1.15新。不支持这些较新加密类型的krb5版本的服务不能在KDC数据库中给出这些加密类型的密钥。

#### 1.2.2.4 示例kdc.conf文件

以下是kdc.conf文件的示例：

```conf
[kdcdefaults]
    kdc_listen = 88
    kdc_tcp_listen = 88
[realms]
    ATHENA.MIT.EDU = {
        kadmind_port = 749
        max_life = 12h 0m 0s
        max_renewable_life = 7d 0h 0m 0s
        master_key_type = aes256-cts-hmac-sha1-96
        supported_enctypes = aes256-cts-hmac-sha1-96:normal aes128-cts-hmac-sha1-96:normal
        database_module = openldap_ldapconf
    }

[logging]
    kdc = FILE:/usr/local/var/krb5kdc/kdc.log
    admin_server = FILE:/usr/local/var/krb5kdc/kadmin.log

[dbdefaults]
    ldap_kerberos_container_dn = cn=krbcontainer,dc=mit,dc=edu

[dbmodules]
    openldap_ldapconf = {
        db_library = kldap
        disable_last_success = true
        ldap_kdc_dn = "cn=krbadmin,dc=mit,dc=edu"
            # this object needs to have read rights on
            # the realm container and principal subtrees
        ldap_kadmind_dn = "cn=krbadmin,dc=mit,dc=edu"
            # this object needs to have read and write rights on
            # the realm container and principal subtrees
        ldap_service_password_file = /etc/kerberos/service.keyfile
        ldap_servers = ldaps://kerberos.mit.edu
        ldap_conns_per_server = 5
    }
```

### 1.3 使用OpenLDAP后端配置Kerberos

1. 在OpenLDAP服务器和客户端上设置SSL，以确保KDC服务和LDAP服务器位于不同机器上时的安全通信。 如果LDAP服务器和KDC服务在同一台机器上运行，则可以使用`ldapi：//`。

   - 在OpenLDAP服务器上设置SSL：
     - 使用OpenSSL工具获取CA证书

     - 配置OpenLDAP服务器以使用SSL / TLS

       对于后者，您需要在*slapd.conf*文件中指定CA证书位置的位置。

       有关更多信息，请参阅以下链接：[http](http://www.openldap.org/doc/admin23/tls.html) :[//www.openldap.org/doc/admin23/tls.html](http://www.openldap.org/doc/admin23/tls.html)

   - 在OpenLDAP客户端上设置SSL

     - 对于KDC和Admin Server，您需要在ldap.conf中进行客户端配置。例如：

```conf
TLS_CACERT /etc/openldap/certs/cacert.pem
```

2. 通过提供其存储位置，在LDAP服务器的配置文件（slapd.conf）中包含Kerberos模式文件（kerberos.schema）：

```conf
include /etc/openldap/schema/kerberos.schema
```

3. 选择要将[krb5kdc](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/krb5kdc.html#krb5kdc-8)和[kadmind](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kadmind.html#kadmind-8)服务器绑定到LDAP服务器的DN ，并在必要时创建它们。这些DN将使用[kdc.conf中](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)的**ldap_kdc_dn**和**ldap_kadmind_dn** 指令进行[指定](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5) ; 他们的密码可以用“ `kdb5_ldap_util stashsrvpw` ”和使用**ldap_service_password_file**指令指定的结果文件进行**存储**。

4. 为全局Kerberos容器条目选择一个DN（但不要在此时创建该条目）。将使用[kdc.conf中](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)的**ldap_kerberos_container_dn**指令指定此DN 。将在此DN下创建领域容器条目。主体条目可能存在于领域容器（默认值）下方，也可能存在于从领域容器引用的单独树中。

5. 配置LDAP服务器ACL以使KDC和kadmin服务器DN读取和写入Kerberos数据。如果在领域的[dbmodules](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#dbmodules)子部分中**disable_last_success**和**disable_lockout**都设置为true ，则KDC DN仅需要对Kerberos数据的读取访问。

   示例访问控制信息：

```conf
access to dn.base=""
    by * read

access to dn.base="cn=Subschema"
    by * read

access to attrs=userPassword,userPKCS12
    by self write
    by * auth

access to attrs=shadowLastChange
    by self write
    by * read

# 提供对领域容器的访问
access to dn.subtree= "cn=EXAMPLE.COM,cn=krbcontainer,dc=example,dc=com"
    by dn.exact="cn=kdc-service,dc=example,dc=com" write
    by dn.exact="cn=adm-service,dc=example,dc=com" write
    by * none

# 提供对主体的访问，如果不在领域容器下
access to dn.subtree= "ou=users,dc=example,dc=com"
    by dn.exact="cn=kdc-service,dc=example,dc=com" write
    by dn.exact="cn=adm-service,dc=example,dc=com" write
    by * none

access to *
    by * read
```

6. 启动LDAP服务器，如下所示：

```bash
slapd -h "ldapi:/// ldaps:///"
```

7. 修改[kdc.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)文件以包括下列LDAP特定项目：

```conf
realms
    database_module

dbmodules
    db_library
    db_module_dir
    ldap_kdc_dn
    ldap_kadmind_dn
    ldap_service_password_file
    ldap_servers
    ldap_conns_per_server
```

8. 使用[kdb5_ldap_util](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kdb5_ldap_util.html#kdb5-ldap-util-8)创建领域（请参阅 [创建Kerberos领域](http://web.mit.edu/kerberos/krb5-latest/doc/admin/database.html#ldap-create-realm)）：

```bash
kdb5_ldap_util -D cn=admin,dc=example,dc=com create -subtrees ou=users,dc=example,dc=com -r EXAMPLE.COM -s
```

如果主体将存在于与领域容器的单独子树中，则使用**-subtrees**选项。在执行命令之前，请确保存在上述的子树 `（ou = users，dc = example，dc = com）`。如果主体将存在于领域容器下方，请忽略**-subtrees**选项，而不用担心创建主体子树。

有关详细信息，请参阅[LDAP数据库上的操作](http://web.mit.edu/kerberos/krb5-latest/doc/admin/database.html#ops-on-ldap)一节。

在配置文件中指定的**ldap_kerberos_container_dn**下创建域对象 。此操作也将创建Kerberos容器，如果不存在的话。这将用于存储与所有领域相关的信息。

9. 使用[kdb5_ldap_util](http://web.mit.edu/kerberos/krb5-latest/doc/admin/admin_commands/kdb5_ldap_util.html#kdb5-ldap-util-8) **stashsrvpw**命令将KDC和管理服务使用的[服务对象的密码保存](http://web.mit.edu/kerberos/krb5-latest/doc/admin/database.html#stash-ldap)为绑定到LDAP服务器 （请参阅“ [冻结服务对象的密码”](http://web.mit.edu/kerberos/krb5-latest/doc/admin/database.html#stash-ldap)）。对象DN应与[kdc.conf](http://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_files/kdc_conf.html#kdc-conf-5)文件中指定的**ldap_kdc_dn**和**ldap_kadmind_dn**值 相同 ：

```bash
kdb5_ldap_util -D cn=admin,dc=example,dc=com stashsrvpw -f /etc/kerberos/service.keyfile cn=krbadmin,dc=example,dc=com
```

10. 将`krbPrincipalName`添加到slapd.conf中的索引，以加快访问速度。

使用LDAP后端可以提供主体条目的别名。目前，我们不提供用于创建别名的机制，因此必须通过直接操作LDAP条目来完成。

具有别名的条目包含*krbPrincipalName*属性的多个值 。由于LDAP属性值未排序，因此必须使用*krbCanonicalName*属性指定哪个主体名称是规范的。因此，要为条目创建别名，首先将该条目的*krbCanonicalName*属性设置为规范主体名称（该名称应与预先存在的*krbPrincipalName*值相同），然后为别名添加其他 *krbPrincipalName*属性。

当客户端请求规范化时，主体别名仅由KDC返回。通常要求服务主体规范化; 对于客户主体，通常需要显式标志（例如，`kinit -C`），并且仅对初始票据请求执行规范化。

**也可以看看**

[Ubuntu 10.4上的LDAP后端（lucid）](http://web.mit.edu/kerberos/krb5-latest/doc/admin/advanced/ldapbackend.html#ldap-be-ubuntu)
