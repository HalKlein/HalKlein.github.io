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

## 为什么 Redis 能这么快？

官方称 Redis 能达到 10w+ QPS （ QPS 即 query per second ，每秒内查询次数 ）

### 它为何能这么快？

1. 完全基于内存，绝大部分请求是纯粹的内存操作，执行效率高
2. 数据结构简单，对数据操作也简单
3. 采用单线程，单线程也能处理高并发请求，想多核也可启动多实例
4. 使用多路 I/O 复用模型，非阻塞 IO

### 多路 I/O 复用模型

FD : File Descriptor，文件描述符

一个打开的文件通过唯一的描述符进行引用，该描述符是打开文件的元数据到文件本身的映射

**传统的阻塞 I/O 模型**

![image-20200220164714875](G:\BlogImg\image-20200220164714875.png)

Redis 采用的 I/O 多路复用函数：epoll / kqueue / evport / select ？

- 因地制宜

- 优先选择时间复杂度为 O(1) 的 I/0 多路复用函数作为底层实现，包括 Solaries 10 中的 evport、Linux 中的 epoll 和 macOS/FreeBSD 中的 kqueue，上述的这些函数都使用了内核内部的结构，并且能够服务几十万的文件描述符。但是如果当前编译环境没有上述函数，就会选择 select 作为备选方案，由于其在使用时会扫描全部监听的描述符，所以其时间复杂度较差 ，并且只能同时服务 1024 个文件描述符，所以一般并不会以 select 作为第一方案使用。
- 以时间复杂度为 O(n) 的 select 作为保底
- 基于 react 设计模式监听 I/O 事件

### Redis 底层数据类型基础

- 简单动态字符串
- 链表
- 字典
- 跳跃表
- 整数集合
- 压缩列表
- 对象

</br>

## 安装 for Windows

**下载地址**：https://github.com/microsoftarchive/redis/releases

**默认端口**：`6379`

Redis默认内置了16个库 ，可以通过 select 0-15 进行切换，默认进入0

```properties
# 进入 Redis 服务
$ redis-cli
# 选择库
127.0.0.1:6379> select 1
OK
# 连通性测试 ping - pong
127.0.0.1:6379> ping
PONG
```

**刷除库内的数据**：flushdb

注意Redis中key的命名提倡使用 `:` 连接而不是 `_`

</br>

## Redis常用数据类型的使用

### **字符串 （String）类型数据 测试**

基本数据类型，二进制安全（ 意思是可以包含多种数据类型 ）。

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

String 元素组成的字典，适合用于存储对象

```properties
# Hash存数据
127.0.0.1:6379> hset test:user id 1
(integer) 1
# Hash存数据
127.0.0.1:6379> hset test:user username HalKlein
(integer) 1
# 或者 hmset test:user id 1 username HalKlein
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

Redis 的列表可以看作一个横向的容器，

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

String 元素组成的无序集合，通过哈希表实现，集合是无序的，且内容是不能重复的。Redis 还特别提供了快速的求交集和并集的操作，可以用来实现比如 “ 共同关注 ” 这样的功能。

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

顾名思义，该集合是有序的，它会根据元素的 score 进行排序（默认由小到大）

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

### 其它的数据类型

Redis 还提供其它的一些特殊数据类型，如用于计数的 HyperLogLog ，用于支持存储地理位置信息的 Geo

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

## 从海量数据里查询某一固定前缀的key

首先应该弄清楚数据的规模！

### KEYS

如果用 Keys Pattern ：查找所有符合给定模式 pattern 的 key 

比如：keys ha* ，查找所有以 "ha" 开头的 Key

由于 Keys 指令是一次性返回所有匹配的 key，如果 key 的数量过大会使服务卡顿，对内存的消耗和Redis服务器来说都是一个隐患。

### SCAN cursor [MATCH pattern] [COUNT count]

- 基于游标的迭代器，需要基于上一次的游标延续之前的迭代过程 。
- 以 0 作为游标开始一次新的迭代，直到命令返回游标 0 完成一次遍历 。
- 但不保证每次执行都返回某个给定数量的元素，另外支持模糊查询 。
- 需要注意，一次返回的数量不可控，只能是大概率符合count参数 。

如：scan 0 match ha* count 10 ，意思是新的迭代查询以 "ha" 开头的 key ，并且期望每次大约返回 10 个key 。当执行之后返回的第一行数据，是新的游标地址，之后是一个 keys 数据集 。

注意，通过测试会发现返回的游标并不是每次递增的，也就是说，查询到的 key 有可能会出现重复，需要在外部程序中进行去重 。

</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
