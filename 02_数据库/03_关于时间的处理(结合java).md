# 好的范式是怎样的

数据库存储UTC时间戳(不带时区信息)

代码同样当UTC处理

UI拿到UTC时间之后，根据本地时区信息进行渲染

>奇怪的是网上居然没找到这方面的介绍

# mysql存储

使用TimeStamp类型时，数据库存储的是UTC时间，也就是绝对时间。

使用datagrip连接mysql进行验证

## 会话可以设置时区，从而从数据库查出不同的结果。

```
SET time_zone='+08:00';
select updated_at from cell_cluster;
```

结果：

```
2021-07-07 17:23:38
```



```
SET time_zone='+00:00';
select updated_at from cell_cluster;
```

结果：

```
2021-07-07 09:23:38
```

在会话中设置时区，等于是在根据时区进行渲染。



## 查看数据库中存储的值

数据库中存储的其实是一个utc时间戳，如果当数值查出来，那么无论设置时区是啥，查出的结果都一样：

```
SET time_zone='+00:00';
select unix_timestamp(updated_at) from cell_cluster;
```

查出的结果都是：

```
1625729508
```

对应于北京时间：

```
2021-07-07 17:23:38
```

# java代码的处理

## java用什么类型保存UTC时间

试了java.sql.timestamp和LocalDateTime

发现这两个类本身都带了时区信息，让问题变得很复杂，不符合预期。

> stack over flow找到一篇文章让用LocalDateTime
>
> <https://stackoverflow.com/a/63078938/13789176>



最后发现不带时区信息的java.time.Instant才是想要的。

```java
private Instant updatedAt;
```

再转换为时间戳返回到前台：

```
updatedAt.toEpochMilli()
```

## 怎么获取当前时刻的UTC时间

```
        System.out.println(System.currentTimeMillis());
        System.out.println(Instant.now().toEpochMilli());
```

这两个结果是一样的。都是UTC时间。



## 令人迷惑的serverTimezone

### 不设置serverTimezone

```
spring.datasource.url: jdbc:mariadb://localhost:3306/cellmdconsole
```

java代码获取到的Instant：

```
2021-07-08T09:23:38Z
```

### 设置serverTimezone=UTC

```
spring.datasource.url: jdbc:mariadb://localhost:3306/cellmdconsole?serverTimezone=UTC&useLegacyDatetimeCode=false
```

java代码不设置时区(系统默认为东8区)

java代码获取到的Instant：

```
2021-07-08T17:23:38Z 
```

可以看出已经转换为本地时间了，不是UTC时间。



### 设置serverTimezone=UTC并且设置java代码启动时属性：-Duser.timezone=UTC

java代码获取到的Instant：

```
2021-07-08T17:23:38Z 
```

serverTimezone这个参数到底表示什么意思，没找到答案。。也没搞明白为什么设置了会对获取到的Instant有影响。



# UI拿到UTC时间之后，根据本地所在时区进行渲染即可。



# 工具网站

[时间戳转换](https://www.bejson.com/convert/unix/)  需要注意的是，转换的结果是北京时间。

[Java – Display all ZoneId and its UTC offset](https://mkyong.com/java8/java-display-all-zoneid-and-its-utc-offset/)  各个时区的ZoneID。类似UTC这种，比如东八区可以用Asia/Shanghai表示。

