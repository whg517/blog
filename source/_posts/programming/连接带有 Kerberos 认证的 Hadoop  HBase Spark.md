---
title: 连接带有 Kerberos 认证的 Hadoop Hbase Hive Spark
author: Kevin
date: 2017-11-21 18:18:08
updated: 2017-11-27 11:47:25
tags:
- kerberos
- hadoop
- hbase
- hive
- spark
categories: Hadoop
---

> 说明：本文操作环境为使用 Windows 远程连接集群环境，通过代码带入 Kerberos 认证信息操作集群。
>
> 项目为 Maven Project，添加依赖仅限于测试连接，如做额外开发可能会缺少依赖。请自行添加

## 操作环境：

- CentOS：7
- Ambari：2.5.1.0
- HDP：2.6.2.0-205
- 客户机：Windows 10


## 准备文件：

Kerberos 配置文件。该文件一般位于 Kerberos 服务器的 `/etc/krb5.conf` 位置。将其下载到本地。

Hadoop 的配置文件。

<!-- more -->

## Java 连接 Hadoop

[Hadoop Authentication](https://www.2cto.com/kf/201204/126021.html)

上面这篇博客讲解了 java 连接带有 Kerberos 的 HDFS 流程。可以看一下。

新建一个 Maven 项目

![maven_project](http://ono3vb8rf.bkt.clouddn.com/FgvE_RlTJ6YsxhMw7m9lbDSKLPvd.png)

添加 HDFS 依赖

```xml
        <!-- HDFS dependences -->
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.7.4</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.4</version>
        </dependency>
```

将 `krb5.conf` `tendata.keytab`  放到项目根目录

![](http://ono3vb8rf.bkt.clouddn.com/Fvr3toNl75tVQV6siXCnKq1bdQ_g.png)

实例代码

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.LocatedFileStatus;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.RemoteIterator;
import org.apache.hadoop.security.UserGroupInformation;

import java.io.IOException;

/**
 * Created by Administrator on 2017/11/21
 */
public class HadoopAuth {
    static Configuration conf = new Configuration();

    static{
        System.setProperty("java.security.krb5.conf", "krb5.conf");
        conf.set("HadoopAuth.security.authentication", "kerberos");
        conf.set("fs.defaultFS", "hdfs://192.168.10.1");
        UserGroupInformation.setConfiguration(conf);
        try {
            UserGroupInformation.loginUserFromKeytab("tendata@TENDATA.CN", "tendata.keytab");
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    public static void main(String[] args) throws IOException {
        FileSystem fs = FileSystem.get(conf);
        // 传入当前认证用户有操作权限的路径
        Path path = new Path("/user/tendata/testdir");
        if (! fs.exists(path)){
            boolean mkdirs = fs.mkdirs(path);
            System.out.println("mkdir " + mkdirs );
        }

        boolean delete = fs.delete(path, true);
        System.out.println("delete " + delete);
    }
}

```



## Java 连接 HBase



添加 HBase 依赖

```xml
		<!-- HBase dependence -->
        <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-client -->
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>1.1.12</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.phoenix/phoenix-core -->
        <dependency>
            <groupId>org.apache.phoenix</groupId>
            <artifactId>phoenix-core</artifactId>
            <version>4.7.0-HBase-1.1</version>
        </dependency>
```

实例代码

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.ClusterStatus;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.ServerName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.security.UserGroupInformation;

import java.io.IOException;

/**
 * Created by Administrator on 2017/11/21
 */
public class HBaseAuth {

    static String zNode = "m1.node.hadoop,m2.node.hadoop,m3.node.hadoop";
    static String zPort = "2181";

    public static void main(String[] args) throws IOException {
        Configuration conf = HBaseConfiguration.create();

        // krb5.conf 文件位置随意。当不在项目中的时候需要引用绝对路径
        System.setProperty("java.security.krb5.conf", "krb5.conf");     // 配置系统使用 krb5.conf

        conf.set("hadoop.security.authentication", "Kerberos");     // 指定 HDFS 安全认证方式为 Kerberos。

        // 下面的对 hdfs-site.xml  的配置都可以在集群配置文件中找得到的
        conf.set("hbase.security.authentication", "kerberos");      // 指定 HBase 安全认证方式为 Kerberos。
        conf.set("hbase.master.kerberos.principal", "hbase/_HOST@TENDATA.CN");
        conf.set("hbase.zookeeper.property.clientPort", zPort);     // Zookeeper 端口
        conf.set("hbase.zookeeper.quorum", zNode);      // Zookeeper 节点

        /*
        在查看 Zookeeper 根目录信息会有两个 hbase 开头的目录
        /hbase-secure   在配置安全集群下，HBase 会使用这个
        /hbase-unsecure 在没有配置安全集群下，使用此目录
         */
        conf.set("zookeeper.znode.parent", "/hbase-secure");    // 指定 HBase 在 Zookeeper 目录

        // 通过hadoop security下中的 UserGroupInformation类来实现使用keytab文件登录

        UserGroupInformation.setConfiguration(conf);

        // 设置登录的kerberos principal和对应的 keytab 文件，其中 keytab 文件需要kdc管理员生成给到开发人员
        // 文件如果没法放在项目目录中，则需要引用路径
        UserGroupInformation.loginUserFromKeytab("tendata@TENDATA.CN", "tendata.keytab");

        // 与HBase数据库的连接对象
        Connection connection = ConnectionFactory.createConnection(conf);
        // 数据库元数据操作对象
        Admin admin = connection.getAdmin();

        System.out.println("---------------获取集群信息-----------------");
        ClusterStatus clusterStatus = admin.getClusterStatus();
        String id = clusterStatus.getClusterId();
        String hbaseVersion = clusterStatus.getHBaseVersion();
        ServerName master = clusterStatus.getMaster();
        System.out.println("id:\t" + id +"\n" +
                            "version:\t" + hbaseVersion + "\n" +
                            "master:\t" + master
        );
    }
}

```

## Java 连接 Hive

添加 Hive 依赖

```xml
		<!-- Hive dependence -->
        <!-- https://mvnrepository.com/artifact/org.apache.hive/hive-jdbc -->
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-jdbc</artifactId>
            <version>1.2.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hive/hive-common -->
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-common</artifactId>
            <version>1.2.1</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient -->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5</version>
        </dependency>
```

实例代码

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.security.UserGroupInformation;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

/**
 * Created by Administrator on 2017/11/21
 */
public class HiveAuth {

    private static String driverName = "org.apache.hive.jdbc.HiveDriver";   // Hive JDBC 驱动
    // url ，注意后面带有 principal
    // 这里使用 hive2 连接
    private static String url = "jdbc:hive2://m1.node.hadoop:10000/default;principal=hive/m1.node.hadoop@TENDATA.CN;";
    private static String user = "hive";
    private static String password = "";

    public static void main(String[] args) throws Exception {
        // krb5.conf 文件位置随意。当不在项目中的时候需要引用绝对路径
        System.setProperty("java.security.krb5.conf", "krb5.conf");     // 配置系统使用 krb5.conf

        Configuration conf = new Configuration();
        conf.set("hadoop.security.authentication", "kerberos"); // 指定 hadoop 安全认证方式为 Kerberos。

        // 通过hadoop security下中的 UserGroupInformation类来实现使用keytab文件登录
        UserGroupInformation.setConfiguration(conf);

        // 设置登录的 kerberos principal和对应的 keytab 文件，其中 keytab 文件需要kdc管理员生成给到开发人员
        // 文件如果没法放在项目目录中，则需要引用路径
        UserGroupInformation.loginUserFromKeytab("tendata@TENDATA.CN", "tendata.keytab");

        Class.forName(driverName);
        Connection conn = DriverManager.getConnection(url, user, password);
        Statement stmt = conn.createStatement();


        ResultSet res = stmt.executeQuery("show databases");
        while (res.next()) {
            System.out.println(res.getString(1));
        }
    }
}

```

## Java 连接 Spark

使用 Java 连接 Spark 应该是少引用了某个参数，导致指定的安全认证参数传不进去。一直认定是 SIMPLE。

索性直接引入 hadoop 的配置文件。

在 Resource 目录下放入 `core-site.xml` `yarn-site.xml` 两个文件。

![](http://ono3vb8rf.bkt.clouddn.com/FqF9JVuTAUIj3RWpkc0lP59QtHIa.png)

添加 Spark 依赖

```xml
        <!-- Spark dependence -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>1.6.3</version>
        </dependency>
```



实例代码

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.security.UserGroupInformation;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

import java.io.IOException;
import java.util.List;

/**
 * Created by Administrator on 2017/11/21
 */
public class SparkAuth {

    static{
        // 这里要给系统加载 krb5 的配置，否则系统会使用默认 remal
        System.setProperty("java.security.krb5.conf", "krb5.conf");
        Configuration configuration = new Configuration();
        // 下面的对 hdfs-site.xml  的配置都可以在集群配置文件中找得到的
        configuration.set("hadoop.security.authentication", "kerberos");     // 指定 hadoop 安全认证方式为 Kerberos。
        configuration.set("dfs.namenode.kerberos.principal.pattern", "nn/*@TENDATA.CN");
        configuration.set("fs.defaultFS", "hdfs://192.168.10.1");

        // 通过hadoop security下中的 UserGroupInformation类来实现使用keytab文件登录
        UserGroupInformation.setConfiguration(configuration);
        try {
            // 设置登录的kerberos principal和对应的 keytab 文件，其中 keytab 文件需要kdc管理员生成给到开发人员
            // 文件如果没法放在项目目录中，则需要引用路径
            UserGroupInformation.loginUserFromKeytab("tendata@TENDATA.CN", "tendata.keytab");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        SparkConf conf = new SparkConf().setAppName("Test_Java_E_Kevin").setMaster("local");

        JavaSparkContext sc = new JavaSparkContext(conf);

        JavaRDD<String> stringJavaRDD = sc.textFile("hdfs://m1.node.hadoop:8020/user/tendata/tmp/tmp-kevin/example/data/example_data_id_name.txt", 1);

        List<String> collect = stringJavaRDD.collect();

        for (String i : collect) {
            System.out.println(i);
        }
    }
}

```

## Scala 连接 Spark

针对于 Scala Spark 的环境请自行配置，具体连接代码如下。

需要在 Resource 目录下引用前面提到的两个配置文件

```scala
import org.apache.hadoop.conf.Configuration
import org.apache.hadoop.security.UserGroupInformation
import org.apache.spark.sql.SQLContext
import org.apache.spark.{SparkContext, SparkConf}

object Put{
  def main(args: Array[String]) {
    System.setProperty("java.security.krb5.conf", "krb5.conf")

    val conf = new Configuration()
    conf.set("hadoop.security.authentication", "Kerberos")
    conf.set("fs.defaultFS", "hdfs://192.168.10.1")
    conf.set("dfs.namenode.kerberos.principal.pattern", "nn/*@TENDATA.CN")
    UserGroupInformation.setConfiguration(conf)
    UserGroupInformation.loginUserFromKeytab("tendata@TENDATA.CN", "tendata.keytab")

    val sparkConf = new SparkConf().setAppName("Test_SparkReadHDFS_Kevin").setMaster("local")
    val sc = new SparkContext(sparkConf)
    val scRDD = sc.textFile("hdfs://m1.node.hadoop:8020/user/tendata/tmp/tmp-kevin/example/data/example_data_id_name.txt", 1);
    scRDD.collect().foreach(x => println(x))
  }
}
```

## 错误总结

##### 1.  Can't get Kerberos realm

```
Exception in thread "main" java.lang.IllegalArgumentException: Can't get Kerberos realm
	at org.apache.hadoop.security.HadoopKerberosName.setConfiguration(HadoopKerberosName.java:65)
	at org.apache.hadoop.security.UserGroupInformation.initialize(UserGroupInformation.java:249)
	at org.apache.hadoop.security.UserGroupInformation.setConfiguration(UserGroupInformation.java:285)
	at HdfsConnKerberos.HDFSClient.main(HDFSClient.java:43)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.apache.hadoop.security.authentication.util.KerberosUtil.getDefaultRealm(KerberosUtil.java:84)
	at org.apache.hadoop.security.HadoopKerberosName.setConfiguration(HadoopKerberosName.java:63)
	... 8 more
Caused by: KrbException: Cannot locate default realm
	at sun.security.krb5.Config.getDefaultRealm(Config.java:1006)
	... 14 more
Caused by: KrbException: Generic error (description in e-text) (60) - Unable to locate Kerberos realm
	at sun.security.krb5.Config.getRealmFromDNS(Config.java:1102)
	at sun.security.krb5.Config.getDefaultRealm(Config.java:987)
	... 14 more
```

主要几点

```
Caused by: KrbException: Cannot locate default realm

Caused by: KrbException: Generic error (description in e-text) (60) - Unable to locate Kerberos realm
```



主要原因在代码中没有添加 `krb5.conf` 这个配置，所以检查这个配置文件的是否存在和文件内容的正确性

##### 2. org.apache.hadoop.hbase.exceptions.ConnectionClosingException: Call to m1.node.hadoop/192.168.10.1:16000 failed

读取 HBase 信息，连接到 HBase 没反应，重复出现下面的信息。

Call exception, tries=10, retries=31, started=48283 ms ago, cancelled=false, msg=com.google.protobuf.ServiceException: org.apache.hadoop.hbase.exceptions.ConnectionClosingException: Call to m1.node.hadoop/192.168.10.1:16000 failed on local exception: org.apache.hadoop.hbase.exceptions.ConnectionClosingException: Connection to m1.node.hadoop/192.168.10.1:16000 is closing. Call id=10, waitTime=1

```
2017-10-13 15:26:02,021 INFO  [main] zookeeper.ZooKeeper (ZooKeeper.java:<init>(438)) - Initiating client connection, connectString=m1.node.hadoop:2181,m2.node.hadoop:2181,m3.node.hadoop:2181 sessionTimeout=180000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@65b3f4a4
2017-10-13 15:26:02,077 INFO  [main-SendThread(m3.node.hadoop:2181)] zookeeper.ClientCnxn (ClientCnxn.java:logStartConnect(1019)) - Opening socket connection to server m3.node.hadoop/192.168.10.3:2181. Will not attempt to authenticate using SASL (unknown error)
2017-10-13 15:26:02,079 INFO  [main-SendThread(m3.node.hadoop:2181)] zookeeper.ClientCnxn (ClientCnxn.java:primeConnection(864)) - Socket connection established, initiating session, client: /192.168.2.199:59415, server: m3.node.hadoop/192.168.10.3:2181
2017-10-13 15:26:02,104 INFO  [main-SendThread(m3.node.hadoop:2181)] zookeeper.ClientCnxn (ClientCnxn.java:onConnected(1279)) - Session establishment complete on server m3.node.hadoop/192.168.10.3:2181, sessionid = 0x35efece455700dd, negotiated timeout = 40000
2017-10-13 15:26:05,349 WARN  [main]  (NetworkAddressUtils.java:getLocalIpAddress(389)) - Your hostname, DESKTOP-5KM5T43 resolves to a loopback/non-reachable address: fe80:0:0:0:3433:e0f1:9aa7:18da%net4, but we couldn't find any external IP address!
2017-10-13 15:26:06,524 WARN  [main] shortcircuit.DomainSocketFactory (DomainSocketFactory.java:<init>(117)) - The short-circuit local reads feature cannot be used because UNIX Domain sockets are not available on Windows.
2017-10-13 15:26:54,855 INFO  [main] client.RpcRetryingCaller (RpcRetryingCaller.java:callWithRetries(146)) - Call exception, tries=10, retries=31, started=48283 ms ago, cancelled=false, msg=com.google.protobuf.ServiceException: org.apache.hadoop.hbase.exceptions.ConnectionClosingException: Call to m1.node.hadoop/192.168.10.1:16000 failed on local exception: org.apache.hadoop.hbase.exceptions.ConnectionClosingException: Connection to m1.node.hadoop/192.168.10.1:16000 is closing. Call id=10, waitTime=1 
2017-10-13 15:27:15,012 INFO  [main] client.RpcRetryingCaller (RpcRetryingCaller.java:callWithRetries(146)) - Call exception, tries=11, retries=31, started=68440 ms ago, cancelled=false, msg=com.google.protobuf.ServiceException: org.apache.hadoop.hbase.exceptions.ConnectionClosingException: Call to m1.node.hadoop/192.168.10.1:16000 failed on local exception: org.apache.hadoop.hbase.exceptions.ConnectionClosingException: Connection to m1.node.hadoop/192.168.10.1:16000 is closing. Call id=11, waitTime=1 

```

这种情况是在创建 Configuration 的格式不对造成。

下面两种获取 Configuration 对象是不同的。

如果使用第一种 conf 操作 HBase，则会出现上述错误。

而正确的创建方式应该是第二种。

```java
Configuration conf = new Configuration();
Configuration conf = HBaseConfiguration.create();
```

##### 3.  [org.apache.hadoop.hbase.client.RpcRetryingCaller] - Call exception, tries=11, retries=35, started=48413 ms ago, cancelled=false, msg=

连接 HBase 没有反应，重复出现 Call exception

```
2017-11-21 17:45:11,982 INFO [org.apache.zookeeper.ZooKeeper] - Initiating client connection, connectString=m1.node.hadoop:2181,m2.node.hadoop:2181,m3.node.hadoop:2181 sessionTimeout=90000 watcher=hconnection-0x66d189790x0, quorum=m1.node.hadoop:2181,m2.node.hadoop:2181,m3.node.hadoop:2181, baseZNode=/hbase-secure
2017-11-21 17:45:12,049 INFO [org.apache.zookeeper.ClientCnxn] - Opening socket connection to server m2.node.hadoop/192.168.10.2:2181. Will not attempt to authenticate using SASL (unknown error)
2017-11-21 17:45:12,050 INFO [org.apache.zookeeper.ClientCnxn] - Socket connection established to m2.node.hadoop/192.168.10.2:2181, initiating session
2017-11-21 17:45:12,065 INFO [org.apache.zookeeper.ClientCnxn] - Session establishment complete on server m2.node.hadoop/192.168.10.2:2181, sessionid = 0x25fdc95c59d001d, negotiated timeout = 40000
2017-11-21 17:45:15,386 WARN [] - Your hostname, DESKTOP-5KM5T43 resolves to a loopback/non-reachable address: fe80:0:0:0:3c56:c61d:8b18:745%net4, but we couldn't find any external IP address!
---------------获取集群信息-----------------
2017-11-21 17:45:54,986 INFO [org.apache.hadoop.hbase.client.RpcRetryingCaller] - Call exception, tries=10, retries=35, started=38401 ms ago, cancelled=false, msg=
2017-11-21 17:46:04,998 INFO [org.apache.hadoop.hbase.client.RpcRetryingCaller] - Call exception, tries=11, retries=35, started=48413 ms ago, cancelled=false, msg=
```

这是没有指定 HBase 安全认证导致的。

增加如下配置即可

```JAVA
conf.set("hbase.security.authentication", "kerberos");      // 指定 HBase 安全认证方式为 Kerberos。
```

##### 4. java.io.IOException: java.lang.reflect.InvocationTargetException

连接 HBase 出现 `org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory` 类找不到

```java
Exception in thread "main" java.io.IOException: java.lang.reflect.InvocationTargetException
	at org.apache.hadoop.hbase.client.ConnectionFactory.createConnection(ConnectionFactory.java:240)
	at org.apache.hadoop.hbase.client.ConnectionFactory.createConnection(ConnectionFactory.java:218)
	at org.apache.hadoop.hbase.client.ConnectionFactory.createConnection(ConnectionFactory.java:119)
	at HBaseAuth.main(HBaseAuth.java:33)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:422)
	at org.apache.hadoop.hbase.client.ConnectionFactory.createConnection(ConnectionFactory.java:238)
	... 8 more
Caused by: java.lang.UnsupportedOperationException: Unable to find org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory
	at org.apache.hadoop.hbase.util.ReflectionUtils.instantiateWithCustomCtor(ReflectionUtils.java:36)
	at org.apache.hadoop.hbase.ipc.RpcControllerFactory.instantiate(RpcControllerFactory.java:58)
	at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.createAsyncProcess(ConnectionManager.java:2256)
	at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.<init>(ConnectionManager.java:691)
	at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.<init>(ConnectionManager.java:631)
	... 13 more
Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:331)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:264)
	at org.apache.hadoop.hbase.util.ReflectionUtils.instantiateWithCustomCtor(ReflectionUtils.java:32)
	... 17 more
```

https://www.cnblogs.com/itboys/p/6862366.html

1. 检查应用开发工程的配置文件hbase-site.xml中是否包含配置项hbase.rpc.controllerfactory.class。

   ```
   <name>hbase.rpc.controllerfactory.class</name>
   <value>org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory</value>
   ```

2. 如果当前的应用开发工程配置项中包含该配置项，则应用开发程序还需要引入Jar包“phoenix-core-4.4.0-HBase-1.0.jar”。此Jar包可以从HBase客户端安装目录下的“HBase/hbase/lib”获取。

3. 如果不想引入该Jar包，请将应用开发工程的配置文件“hbase-site.xml”中的配置“hbase.rpc.controllerfactory.class”删除掉。

##### 5. SIMPLE authentication is not enabled.  Available:[TOKEN, KERBEROS]

这个问题我只在使用 Spark 的情况下出现过。

```
Exception in thread "main" org.apache.hadoop.security.AccessControlException: SIMPLE authentication is not enabled.  Available:[TOKEN, KERBEROS]
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:422)
	at org.apache.hadoop.ipc.RemoteException.instantiateException(RemoteException.java:106)
	at org.apache.hadoop.ipc.RemoteException.unwrapRemoteException(RemoteException.java:73)
	at org.apache.hadoop.hdfs.DFSClient.getFileInfo(DFSClient.java:2110)
	at org.apache.hadoop.hdfs.DistributedFileSystem$22.doCall(DistributedFileSystem.java:1305)
	at org.apache.hadoop.hdfs.DistributedFileSystem$22.doCall(DistributedFileSystem.java:1301)
	at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
	at org.apache.hadoop.hdfs.DistributedFileSystem.getFileStatus(DistributedFileSystem.java:1317)
	at org.apache.hadoop.fs.Globber.getFileStatus(Globber.java:57)
	at org.apache.hadoop.fs.Globber.glob(Globber.java:252)
	at org.apache.hadoop.fs.FileSystem.globStatus(FileSystem.java:1674)
	at org.apache.hadoop.mapred.FileInputFormat.singleThreadedListStatus(FileInputFormat.java:259)
	at org.apache.hadoop.mapred.FileInputFormat.listStatus(FileInputFormat.java:229)
	at org.apache.hadoop.mapred.FileInputFormat.getSplits(FileInputFormat.java:315)
	at org.apache.spark.rdd.HadoopRDD.getPartitions(HadoopRDD.scala:200)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:248)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:246)
	at scala.Option.getOrElse(Option.scala:121)
	at org.apache.spark.rdd.RDD.partitions(RDD.scala:246)
	at org.apache.spark.rdd.MapPartitionsRDD.getPartitions(MapPartitionsRDD.scala:35)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:248)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:246)
	at scala.Option.getOrElse(Option.scala:121)
	at org.apache.spark.rdd.RDD.partitions(RDD.scala:246)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1911)
	at org.apache.spark.rdd.RDD$$anonfun$foreach$1.apply(RDD.scala:875)
	at org.apache.spark.rdd.RDD$$anonfun$foreach$1.apply(RDD.scala:873)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:112)
	at org.apache.spark.rdd.RDD.withScope(RDD.scala:358)
	at org.apache.spark.rdd.RDD.foreach(RDD.scala:873)
	at HDFS.SparkHDFS$.main(SparkHDFS.scala:43)
	at HDFS.SparkHDFS.main(SparkHDFS.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.security.AccessControlException): SIMPLE authentication is not enabled.  Available:[TOKEN, KERBEROS]
	at org.apache.hadoop.ipc.Client.call(Client.java:1475)
	at org.apache.hadoop.ipc.Client.call(Client.java:1412)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:229)
	at com.sun.proxy.$Proxy20.getFileInfo(Unknown Source)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.getFileInfo(ClientNamenodeProtocolTranslatorPB.java:771)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:191)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:102)
	at com.sun.proxy.$Proxy21.getFileInfo(Unknown Source)
	at org.apache.hadoop.hdfs.DFSClient.getFileInfo(DFSClient.java:2108)
	... 34 more
```

直接在 Resource 目录下加入 `core-site.xml` 配置文件即可。

当人加入了上述配置文件后，重新运行又会出现下面的错误

##### 6.  Can't get Master Kerberos principal for use as renewer

记不得这个错误有没有在其他地方出现过了。反正现在在使用 Spark 的时候出现了。

```
Exception in thread "main" java.io.IOException: Can't get Master Kerberos principal for use as renewer
	at org.apache.hadoop.mapreduce.security.TokenCache.obtainTokensForNamenodesInternal(TokenCache.java:116)
	at org.apache.hadoop.mapreduce.security.TokenCache.obtainTokensForNamenodesInternal(TokenCache.java:100)
	at org.apache.hadoop.mapreduce.security.TokenCache.obtainTokensForNamenodes(TokenCache.java:80)
	at org.apache.hadoop.mapred.FileInputFormat.listStatus(FileInputFormat.java:205)
	at org.apache.hadoop.mapred.FileInputFormat.getSplits(FileInputFormat.java:313)
	at org.apache.spark.rdd.HadoopRDD.getPartitions(HadoopRDD.scala:202)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:239)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:237)
	at scala.Option.getOrElse(Option.scala:121)
	at org.apache.spark.rdd.RDD.partitions(RDD.scala:237)
	at org.apache.spark.rdd.MapPartitionsRDD.getPartitions(MapPartitionsRDD.scala:35)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:239)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:237)
	at scala.Option.getOrElse(Option.scala:121)
	at org.apache.spark.rdd.RDD.partitions(RDD.scala:237)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1929)
	at org.apache.spark.rdd.RDD$$anonfun$collect$1.apply(RDD.scala:927)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:150)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:111)
	at org.apache.spark.rdd.RDD.withScope(RDD.scala:316)
	at org.apache.spark.rdd.RDD.collect(RDD.scala:926)
	at org.apache.spark.api.java.JavaRDDLike$class.collect(JavaRDDLike.scala:339)
	at org.apache.spark.api.java.AbstractJavaRDDLike.collect(JavaRDDLike.scala:46)
	at SparkAuth.main(SparkAuth.java:43)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
```

解决办法：

在 Resource 目录下引入 `yarn-site.xml` 配置文件

**针对前两个错误，主要在 Spark 中出现。本人猜测，可能是因为在 Windows 端运行 local 模式程序的时候，本地作为 Driver，当 Executor 端 真正去访问 HDFS 中的资源的时候， Executor 并没有拿到认证身份。所以，在加入配置文件后， Executor 端会通过配置去相应位置使用 keytab 获取 kgt ，然后正常访问集群中的资源。**

本人才疏学浅，说法可能有误，上述说法仅代表本人观点。如有不正确还望及时联系纠正。

E-mail: kiven517@126.com

##### 7. 终极解决问题

在 Resource 目录引入所使用的服务的所有配置。比如 hadoop 的四个配置文件。

然后使用代码直接加载配置

```java
configuration.addResource("core-site.xml");
configuration.addResource("hdfs-site.xml");
configuration.addResource("mapred-site.xml");
configuration.addResource("yarn-site.xml");
```
