---
title: 测试博客
author: Kevin
---



# 一级标题

## 二级标题

### 三级标题

#### 四级标题

##### 五级标题



- Java 代码

```java
class Test{
  public static void main(String[] args){
  	int a = 1;
    String b = "a";
    System.out.println(b + a)
  }
}
```

<!-- more -->


- Python 代码

```python
import pyspark
a = 0
b = 1
c = a + b
print "a + b = %s" % c
```



- shell

```bash
yum -y install openssh-clients
a = 'a'
cat <<- EOF >/root/test.txt
aaa
[logging]
  path = FILE:/var/log/test.log
EOF
sed -i '/^aaa/,$d' /root/test.txt
```



- scala

```scala
object ReduceByKey{
  def main(args: Array[String]) {

    val list = List("aaa", "bbb", "ccc", "123", "ddd", "eee", "aaa", "bbb", "ddd", "aaa", "123", "123", "ddd")

    val conf = new SparkConf().setAppName("ReduceByKey").setMaster("local")
    val sc = new SparkContext(conf)

    val listRdd = sc.parallelize(list)
    val list_VRdd = listRdd.map(x => (x, 1))

    val countRdd = list_VRdd.reduceByKey(_+_)
    countRdd.foreach(i => println(i))


  }
}
```



- sql

```sql
CREATE TABLE Persons
(
Id_P int,
LastName varchar(255),
)
```

