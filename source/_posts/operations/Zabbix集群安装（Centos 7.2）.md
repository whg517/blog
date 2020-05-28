---
title: Zabbix 集群安装（Centos 7.2）
author: wanghuagang
date: 2018-04-02 18:59:00
updated: 2020-05-28 11:50:00
tags:
- centos
- zabbix
categories: 
- 运维
---

本文主要介绍 Zabbix 集群安装记录。操作环境以 Centos 7 和 Zabbix 3.4 安装三节点集群。

<!--more-->

## 环境说明

1. centos 7.2 操作系统
2. zibbix 3.4.2

**集群环境**

 hostname | ip
 --|--
 server | 192.168.80.101
 agent1 | 192.168.80.102
 agent2 | 192.168.80.103

**server 主机安装的服务**

1. LNMP 一键安装包，安装了 Nginx、Mariadb、PHP
2. 安装 zabbix web 管理页面
3. 安装 Zabbix-server 节点
4. 安装 Zabbix-agent 节点

**agent1 主机 安装的服务**

agent 主机就是需要监控的主机，所以只要安装 zabbix-agent 服务就可以了。

然后可以通过 Web 管理页面将 agent 节点添加进去做监控

## 安装

### server 节点

#### 安装 LNMP / LAMP

执行 lnmp 一键安装包

```bash
wget http://soft.vpser.net/lnmp/lnmp1.5.tar.gz -cO lnmp1.5.tar.gz && tar zxf lnmp1.5.tar.gz && cd lnmp1.5 && ./install.sh lnmp
```

安装 

- mariadb
- php
- nginx

**调整参数**

vim /usr/local/php/etc/php.ini

修改 393 行参数为 300

```conf
393 max_input_time = 300
```

重新加载 php。即可 继续

```bash
lnmp php-fpm reload
```

#### 安装 Zabbix3.4

1. 下载

```bash
wget -O zabbix-3.4.11.tar.gz  https://excellmedia.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/3.4.11/zabbix-3.4.11.tar.gz
```

2. 安装依赖库

```bash
yum -y install net-snmp-devel libxml2-devel libcurl-devel libevent libevent-devel gcc
```

3. 解压编译

在编译时可以指定在编译内容，这里在 server 主机同时编译了 zabbix-server 和 zabbix-agent 服务

```bash
tar zxvf zabbix-3.4.11.tar.gz
cd zabbix-3.4.11
./configure --prefix=/usr/local/zabbix --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2
```

如果要使用加密，需增加 `--with-openssl` 参数进行编译

**错误解决**

`error: MySQL library not found`

```bash
yum install -y mariadb-devel
```

`error: Invalid Net-SNMP directory - unable to find net-snmp-config`

解决

```bash
yum install net-snmp-devel
```

4. 安装

```bash
make && make install
```

5. 创建用户和用户组

```bash
groupadd zabbix
useradd -r -g zabbix zabbix
chown -R zabbix:zabbix /usr/local/zabbix
```

5. 准备 mysql

```sql
mysql -uroot -p
create database if not exists zabbix default character set utf8 collate utf8_general_ci;
grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
flush privileges;
exit;
```

**导入顺序不能错**

```bash
mysql -uzabbix -pzabbix zabbix < database/mysql/schema.sql
mysql -uzabbix -pzabbix zabbix < database/mysql/images.sql
mysql -uzabbix -pzabbix zabbix < database/mysql/data.sql
```

在使用完全分离的集群的搭建的时候，下面两点注意下：

- 如果只是代理可以不用导入 `database/mysql/images.sql`
- 如果只是代理可以不用导入 `database/mysql/data.sql`

6. 创建日志目录

```bash
mkdir /var/log/zabbix
chown zabbix:zabbix /var/log/zabbix
```

7. 配置

**配置 zabbix-server 服务**

`vim /usr/local/zabbix/etc/zabbix_server.conf`

```conf
LogFile=/var/log/zabbix/server.log
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPort=3306
DBPassword=zabbix
DBSocket=/tmp/mysql.sock
```

**配置 zabbix-agent 服务**

注意 agent 被他们写成 `agentd`

vim /usr/local/zabbix/etc/zabbix_agentd.conf

```conf
LogFile=/var/log/zabbix/agentd.log
Server=127.0.0.1
ServerActive=127.0.0.1
Hostname=server
```

这里有几点要注意的：

- `DBSocket=/tmp/mysql.sock` 这一项要打开，要不然会连不上数据库
- `Hostname` 最好指定一个名词，如果在 Web 上出现问题方便查找

8. 配置启动服务

```bash
ln -s /usr/local/zabbix/sbin/* /usr/local/sbin/
ln -s /usr/local/zabbix/bin/* /usr/local/bin/

cp /root/zabbix-3.4.11/misc/init.d/tru64/zabbix_* /etc/init.d/
chmod +x /etc/init.d/zabbix_*
```

9. 启动 server 和 agent

```bash
/etc/init.d/zabbix_server start
/etc/init.d/zabbix_agentd start
```

10. 注册到服务中

```bash
chkconfig --add zabbix_server
chkconfig --add zabbix_agentd
```

配置开机自启

```bash
chkconfig zabbix_server on
chkconfig zabbix_agentd on
```

后面启动的时候可以使用服务方式启动

```bash
service zabbix_server start
service zabbix_server stop
```

如果报错 `service zabbix_server does not support chkconfig`

只需要编辑对应的文件在 `#!/bin/sh` 后 加入一行内容就好了

```sh
#!/bin/sh
# chkconfig: 2345 10 90
```

检查端口

```
[root@server zabbix-3.4.11]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      912/nginx: master p 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      895/sshd            
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      11967/zabbix_agentd 
tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      11907/zabbix_server 
tcp6       0      0 :::3306                 :::*                    LISTEN      1383/mysqld         
tcp6       0      0 :::22                   :::*                    LISTEN      895/sshd            
tcp6       0      0 :::10050                :::*                    LISTEN      11967/zabbix_agentd 
tcp6       0      0 :::10051                :::*                    LISTEN      11907/zabbix_server
```

看看 `zabbix_server` / `zabbix_agentd` 是否都起来了。如果没有可用查看日志检查

#### 部署 Web

```bash
mkdir /home/wwwroot/default/zabbix
cd /root/zabbix-3.4.11/frontends/php
cp -a . /home/wwwroot/default/zabbix/
```

##### 访问 Web 初始化引导

访问zabbix：http://ip/zabbix/index.php ，初始化安装的默认账号：Admin，密码：zabbix

### agent 节点安装

1. 下载

```bash
wget -O zabbix-3.4.11.tar.gz  https://excellmedia.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/3.4.11/zabbix-3.4.11.tar.gz
```

2. 安装依赖库

```bash
yum -y install net-snmp-devel libxml2-devel libcurl-devel libevent libevent-devel gcc
```

3. 解压编译

在编译时可以指定在编译内容，这里在 server 主机同时编译了 zabbix-server 和 zabbix-agent 服务

```bash
tar zxvf zabbix-3.4.11.tar.gz
cd zabbix-3.4.11
./configure --prefix=/usr/local/zabbix --enable-agent
```

如果要使用加密，需增加 `--with-openssl` 参数进行编译。当然前提是 server 节点支持加密

4. 安装

```bash
make && make install
```

5. 创建用户和用户组

```bash
groupadd zabbix
useradd -r -g zabbix zabbix
chown -R zabbix:zabbix /usr/local/zabbix
```

6. 创建日志目录

```bash
mkdir /var/log/zabbix
chown zabbix:zabbix /var/log/zabbix
```

**可选**：加密配置

生成 PSK string ，并保存 `/usr/local/zabbix/etc/zabbix_agentd.psk`

```bash
openssl rand -hex 32 > /usr/local/zabbix/etc/zabbix_agentd.psk
```

授权

```bash
chown zabbix:zabbix /usr/local/zabbix/etc/zabbix_agentd.psk
chmod 644 /usr/local/zabbix/etc/zabbix_agentd.psk
```

7. 配置

**配置 zabbix-agent 服务**

注意 agent 被他们写成 `agentd`

`vim /usr/local/zabbix/etc/zabbix_agentd.conf`

```conf
LogFile=/var/log/zabbix/agentd.log
Server=192.168.80.101
ServerActive=192.168.80.101
Hostname=agent1
```

**加密配置**

```conf
TLSConnect=psk
TLSAccept=psk
TLSPSKFile=/usr/local/zabbix/etc/zabbix_agentd.psk
TLSPSKIdentity=PSK001
```

这里有几点要注意的：

- 要正确配置 server ip
- `Hostname` 最好指定一个名词，如果在 Web 上出现问题方便查找

8. 配置启动服务

```bash
ln -s /usr/local/zabbix/sbin/* /usr/local/sbin/
ln -s /usr/local/zabbix/bin/* /usr/local/bin/

cp /root/zabbix-3.4.11/misc/init.d/tru64/zabbix_agentd /etc/init.d/
chmod +x /etc/init.d/zabbix_agentd
```

9. 启动 server 和 agent

```bash
/etc/init.d/zabbix_agentd start
```

配置开机自启

```bash
chkconfig zabbix_agentd on
```

如果报错 `service zabbix_agentd does not support chkconfig`

只需要编辑对应的文件在 `#!/bin/sh` 后 加入一行内容就好了

```sh
#!/bin/sh
# chkconfig: 2345 10 90
```

**加密检测**

```bash
zabbix_get-s127.0.0.1-k"system.cpu.load[all,avg1]"--tls-connect=psk--tls-psk-identity="PSK001"--tls-psk-file=/usr/local/zabbix/etc/zabbix_agentd.psk
```

10. 在 Web 页面添加到主机中

-----------------

记录

通过 yum 安装后的服务启动文件

cat /usr/lib/systemd/system/zabbix-server.service

```conf
[Unit]
Description=Zabbix Server
After=syslog.target
After=network.target

[Service]
Environment="CONFFILE=/etc/zabbix/zabbix_server.conf"
EnvironmentFile=-/etc/sysconfig/zabbix-server
Type=forking
Restart=on-failure
PIDFile=/run/zabbix/zabbix_server.pid
KillMode=control-group
ExecStart=/usr/sbin/zabbix_server -c $CONFFILE
ExecStop=/bin/kill -SIGTERM $MAINPID
RestartSec=10s
TimeoutSec=0

[Install]
WantedBy=multi-user.target
```

cat /usr/lib/systemd/system/zabbix-agent.service 

```conf
[Unit]
Description=Zabbix Agent
After=syslog.target
After=network.target

[Service]
Environment="CONFFILE=/etc/zabbix/zabbix_agentd.conf"
EnvironmentFile=-/etc/sysconfig/zabbix-agent
Type=forking
Restart=on-failure
PIDFile=/run/zabbix/zabbix_agentd.pid
KillMode=control-group
ExecStart=/usr/sbin/zabbix_agentd -c $CONFFILE
ExecStop=/bin/kill -SIGTERM $MAINPID
RestartSec=10s

[Install]
WantedBy=multi-user.target
```