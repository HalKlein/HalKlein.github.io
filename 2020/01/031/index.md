# Redis 高性能存储入门


<!--more-->

## Redis简介

![](https://i.loli.net/2020/01/30/TSIZAhWOYXGcjVn.png)

- Redis是一款基于键值对的**NoSQL**数据库，它的值支持多种数据结构：
  字符串（strings）、哈希（hashes）、列表（lists）、集合（sets）、有序集合（sorted sets）等。

- Redis将所有的数据都存放在内存中，所以它的读写性能十分惊人。
  同时，Redis还可以将内存中的数据以快照 (RDB，适合空闲的时候进行定时备份) 或日志 (AOF，可以实时存储，但恢复速度慢)  的形式保存到硬盘上，以保证数据的安全性。

- Redis典型的应用场景包括：**缓存**、**排行榜**、**计数器**、**社交网络**、**消息队列** (但Redis不是专业的消息队列工具) 等。

  官网：https://redis.io 

  Windows平台：https://github.com/microsoftarchive/redis

</br>

## 安装 for Windows

**下载地址**：https://github.com/microsoftarchive/redis/releases

**默认端口**：`6379`

Redis默认内置了16个库 ，可以通过 select 0-15 进行切换，默认进入0

```properties
$ redis-cli
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379>
```

**刷除库内的数据**：flushdb

注意Redis中key的命名提倡使用 `:` 连接而不是 `_`

</br>

## Redis常用数据类型的使用

### **字符串 （String）类型数据 测试**

```properties
# 存String测试
127.0.0.1:6379> set author HalKlein
OK
# 查询String数据
127.0.0.1:6379> get author
"HalKlein"
127.0.0.1:6379>
127.0.0.1:6379> set count 1
OK
127.0.0.1:6379> get count
"1"
# 加1
127.0.0.1:6379> incr count
(integer) 2
# 减1
127.0.0.1:6379> decr count
(integer) 1
127.0.0.1:6379>

```

</br>

### 哈希（hashes）类型数据 测试

```properties
# Hash存数据
127.0.0.1:6379> hset test:user id 1
(integer) 1
# Hash存数据
127.0.0.1:6379> hset test:user username HalKlein
(integer) 1
# Hash取数据
127.0.0.1:6379> hget test:user username
"HalKlein"
# Hash取数据
127.0.0.1:6379> hget test:user id
"1"
127.0.0.1:6379>
```

</br>

### 列表（lists）类型数据 测试

![Redis列表](https://i.loli.net/2020/01/30/hIRMju15YPQnHFg.png)

Redis的列表可以看作一个横向的容器，

我们往里面存数据时，支持从左边往里“装”，也支持从右边往里“装”，

取数据时，支持从左边取数据，也支持从右边取数据，

根据这样的设计，如果我们的数据从一端进，从另一端出，就可以看作是一个**队列**；

如果我们的数据从一端进，也从该端出，就可以看作是一个**栈**；

#### **List左进右出演示**

```properties
# List左端进数据
127.0.0.1:6379> lpush test:ids 101 102 103
(integer) 3
# List长度查询
127.0.0.1:6379> llen test:ids
(integer) 3
# 根据List下标查数据
127.0.0.1:6379> lindex test:ids 0
"103"
# 查询List某范围之间的数据
127.0.0.1:6379> lrange test:ids 0 2
1) "103"
2) "102"
3) "101"
# 从右端弹出数据
127.0.0.1:6379> rpop test:ids
"101"
# 从右端弹出数据
127.0.0.1:6379> rpop test:ids
"102"
# List长度查询
127.0.0.1:6379> llen test:ids
(integer) 1
127.0.0.1:6379>
```

</br>

### 集合（sets）类型数据 测试

集合是无序的，且内容是不能重复的。

```properties
# 集合中添加数据
127.0.0.1:6379> sadd test:student aaa bbb ccc ddd eee
(integer) 5
# 统计集合中元素个数
127.0.0.1:6379> scard test:student
(integer) 5
# 从集合中随机弹出一个元素
127.0.0.1:6379> spop test:student
"aaa"
# 从集合中随机弹出一个元素
127.0.0.1:6379> spop test:student
"eee"
# 查看集合中的元素
127.0.0.1:6379> smembers test:student
1) "bbb"
2) "ccc"
3) "ddd"
127.0.0.1:6379>
```

</br>

### 有序集合（sorted sets）类型数据 测试

顾名思义，该集合是有序的，它会根据元素的score进行排序（默认由小到大）

```properties
# 有序集合添加数据
127.0.0.1:6379> zadd test:teachers 20 aaa 10 bbb 50 ccc 30 ddd 40 eee
(integer) 5
# 统计有序集合中元素数量
127.0.0.1:6379> zcard test:teachers
(integer) 5
# 查询有序集合中的元素的score
127.0.0.1:6379> zscore test:teachers eee
"40"
# 查询有序集合中元素的位置排名
127.0.0.1:6379> zrank test:teachers eee
(integer) 3
# 查询某个范围间的所有元素
127.0.0.1:6379> zrange test:teachers 0 3
1) "bbb"
2) "aaa"
3) "ddd"
4) "eee"
127.0.0.1:6379>
```

</br>

## 其它的一些常用命令

```properties
# 查询所有的key
127.0.0.1:6379> keys *
1) "author"
2) "test:user"
3) "count"
4) "test:ids"
5) "test:student"
6) "test:teachers"
# 查询以 ‘test’开头的key
127.0.0.1:6379> keys test*
1) "test:user"
2) "test:ids"
3) "test:student"
4) "test:teachers"
# 查询某个key对应的value的值的类型
127.0.0.1:6379> type test:user
hash
# 查询某个key是否存在
127.0.0.1:6379> exists test:user
(integer) 1
# 删除某个key，会连带value一起删除
127.0.0.1:6379> del test:user
(integer) 1
# 再次查询某个key是否存在
127.0.0.1:6379> exists test:user
(integer) 0
# 指定某个key的有效时间
127.0.0.1:6379> expire test:student 10
(integer) 1
# 查询所有的key
127.0.0.1:6379> keys *
1) "author"
2) "count"
3) "test:ids"
4) "test:student"
5) "test:teachers"
# 10多秒后，再次查询所有的key
127.0.0.1:6379> keys *
1) "author"
2) "count"
3) "test:ids"
4) "test:teachers"
127.0.0.1:6379>
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
