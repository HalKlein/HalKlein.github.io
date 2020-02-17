# Kafka 入门


<!--more-->

## Kafka简介

早先的时候Kafka只是一个消息队列，只是用来发消息，后来经过不断的完善，扩充自己的功能，慢慢的其功能就不再仅仅局限于消息队列了，所以其官网自称：Apache Kafka® is *a distributed streaming platform* 。其应用也十分广泛，消息系统、日志收集、用户行为追踪、流式处理等。

</br>

## Kafka的特点

高吞吐量、消息持久化、高可靠性、高扩展性。

**消息持久化**是指，Kafka会将消息存到硬盘等永久介质上，这也是Kafka能够构建TB级别消息系统的基础。误区也在这里，硬盘存取效率这么低存到硬盘上不会影响性能吗？其实，硬盘在顺序读写时其性能是很高的，甚至可能高于对内存的随机读写，Kafka就利用了这一点，它对硬盘的读写都是顺序的，这样既保持了高性能又保证了足够大的空间。

**高可靠性**，主要体现在它是一个分布式的服务器，可以做集群部署。

</br>

## Kafka相关术语

**Broker**：Kafka服务器 。

**Zookeeper**：是一个独立的应用和概念，用来管理集群，Kafka需要做集群，就可以使用Zookeeper 。

**Topic**：消息队列实现的方式大致有两种：点对点的方式，发布订阅方式；Kafka采用发布订阅方式，生产者把消息发布到的空间就叫Topic 。

**Partition**：分区，对Topic的分区，可以对多个分区进行多线程写入，提高并发能力  。

![Partition](https://i.loli.net/2020/02/02/uyw86jHIfW5ahsD.png)

**Offset**：消息在Partition内存放的索引

![Offset](https://i.loli.net/2020/02/02/2dwDHtVxsqeEG8S.png)

**Replica**：副本，对数据做备份，Kafka是分布式的所以为了保证可靠性数据不会只存一份，需要通过副本的形式对数据存储很多份，每个分区有多个副本。分为 Leader Replica（主副本） 和 Follower Replica （随从副本），如果主副本失效了，那么集群会从众多随从副本中选一个新的作为主副本。

</br>

## 安装Kafka

http://kafka.apache.org/downloads

下载的Kafka的包中，解压后bin目录能看到里面内置了用于Linux的.sh的命令，也有用于Windows的.bat命令。

**配置Zookeeper.properties`**

配置Kafka内置的Zookeeper的配置文件 `zookeeper.properties` ，主要是配置Zookeeper运行时产生的数据的存放位置

```properties
dataDir=e:/Test/data/zookeeper
```

**配置server.properties**

Kafka日志存放位置：

```properties
log.dirs=e:/Test/log/kafka-logs
```

</br>

## 使用Kafka，示例

**1.先启动Zookeeper**

打开一个cmd窗口，cd到Kafka安装目录，执行（使用指定配置文件，启动Zookeeper）

```properties
$ bin\windows\zookeeper-server-start.bat config\zookeeper.properties
```

**2.启动Kafka**

另外打开一个cmd窗口，cd到Kafka安装目录，执行（使用指定配置文件，启动Kafka）

```properties
$ bin\windows\kafka-server-start.bat config\server.properties
```

**3.使用Kafka**

另外打开一个cmd窗口，cd到Kafka的bin/windows目录下，执行以下命令创建主题Topic

```properties
# 在 bootstrap-server服务器主机地址和端口 上创建1个副本1个分区的主题‘test’
$ kafka-topics.bat --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```

查看所有主题，验证上条命令是否执行成功

```properties
$ kafka-topics.bat --list --bootstrap-server localhost:9092
```

**4.往主题上发送消息（生产者身份）**

执行如下命令

```properties
$ kafka-console-producer.bat --broker-list localhost:9092 --topic test
>Hello
>World
>
```

**5.获取消息（消费着身份）**

另外打开一个cmd窗口，cd到Kafka的bin/windows目录下，执行如下命令，看看能不能得到消息Hello和World

```properties
$ kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
Hello
World

```

这时，在生产者命令窗口继续发消息比如'Hello Kafka'

```properties
>Hello
>World
>Hello Kafka
>
```

再打开消费者命令窗口，就可以看到收到了消息‘Hello Kafka’

```properties
Hello
World
Hello Kafka

```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
