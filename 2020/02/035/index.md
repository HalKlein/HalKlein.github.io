# Spring 整合 Kafka


<!--more-->

## 引入依赖

https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka

为更好的兼容性，使用父pom中声明的版本即可，不要单独声明版本

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

</br>

## 配置Kafka

```properties
# 配置 Kafka
# 服务器地址及端口
spring.kafka.bootstrap-servers=localhost:9092
# 消费者分组ID，对应consumer.properties文件中的配置
spring.kafka.consumer.group-id=community-consumer-group
# 是否自动提交消费者的偏移量
spring.kafka.consumer.enable-auto-commit=true
# 自动提交的频率（单位：ms）
spring.kafka.consumer.auto-commit-interval=3000
# 当消费者找不到指定Topic时忽略该异常，Kafka 2.3之后新增，
# 否则可能启动不起项目，报错：java.lang.IllegalStateException: Topic(s) [xxx] is/are not present and missingTopicsFatal is true
spring.kafka.listener.missing-topics-fatal=false
```

</br>

## 访问Kafka

### 先启用服务

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

如果你修改过配置文件，记得重启服务才能生效。

### 开发测试类

```java
@SpringBootTest
@ContextConfiguration(classes = CommunityApplication.class)
public class KafkaTests {

    @Autowired
    private KafkaProducer kafkaProducer;

    @Test
    public void testKafka() {

        // 生产者发信息
        kafkaProducer.sendMessage("test", "你好！");
        kafkaProducer.sendMessage("test", "在吗？");

        // 阻塞，等待
        try {
            Thread.sleep(1000 * 10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

}

// 生产者
@Component
class KafkaProducer {

    @Autowired
    private KafkaTemplate kafkaTemplate;

    // 发消息
    public void sendMessage(String topic, String content) {
        kafkaTemplate.send(topic, content);
    }

}

// 消费者
@Component
class KafkaConsumer {

    // 收消息
    @KafkaListener(topics = {"test"})
    // 收到消息会封装到ConsumerRecord
    public void handleMessage(ConsumerRecord record) {
        System.out.println(record.value());
    }

}
```

**运行测试类主方法**

```properties
你好！
在吗？
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
