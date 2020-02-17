# Spring 整合 Redis


<!--more-->

## 引入依赖

[**spring-boot-starter-data-redis**](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis)

```xml
<!-- SpringBoot整合Redis，这里可以不要写version，因为在spring-boot-starter-parent中默认已经指定，
并且Spring自己测试过与当前兼容的Redis版本，这样不容易出错 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

</br>

## 配置Redis数据库参数

```properties
# 配置 Redis
# 使用的库（0-15）
spring.redis.database=11
# 主机地址
spring.redis.host=localhost
# 端口号，默认（6379）
spring.redis.port=6379
```

</br>

## 编写Redis配置类，构造RedisTemplate

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;

// Redis 配置类
@Configuration
public class RedisConfig {

    /**
     * RedisAutoConfiguration中默认是RedisTemplate<Object, Object>，但Redis中的key都是String类型的
     * 虽然说Object具有通用性，但是多数时候不便于我们使用，这里直接就改为String类型的key好了
     *
     * @factory redisTemplate要想访问数据库，我们就需要将Redis数据库的连接工厂注入进来
     * @return
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);

        // 指定key的序列化方式为String
        template.setKeySerializer(RedisSerializer.string());
        // 指定普通的value的序列化方式为JSON
        template.setValueSerializer(RedisSerializer.json());
        // Hash比较特殊单独设置
        // 指定Hash的key的序列化方式
        template.setHashKeySerializer(RedisSerializer.string());
        // 指定Hash的value的序列化方式
        template.setHashValueSerializer(RedisSerializer.json());

        // 使参数生效
        template.afterPropertiesSet();
        return template;
    }

}
```

</br>

## 访问Redis

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.BoundValueOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.ContextConfiguration;

import java.util.concurrent.TimeUnit;

@SpringBootTest
//使用正式环境里面的配置类
@ContextConfiguration(classes = CommunityApplication.class)
public class RedisTests {

    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * Redis 字符串测试
     */
    @Test
    public void testStrings() {
        String redisKey = "test:count";

        // 存数据
        redisTemplate.opsForValue().set(redisKey, 1);
        // 获取数据
        System.out.println(redisTemplate.opsForValue().get(redisKey));
        System.out.println(redisTemplate.opsForValue().increment(redisKey));
        System.out.println(redisTemplate.opsForValue().decrement(redisKey));
    }


    /**
     * Redis Hash测试
     */
    @Test
    public void testHash() {
        String redisKey = "test:user";

        // 存数据
        redisTemplate.opsForHash().put(redisKey, "id", 1);
        redisTemplate.opsForHash().put(redisKey, "username", "HalKlein");

        // 取数据
        System.out.println(redisTemplate.opsForHash().get(redisKey, "id"));
        System.out.println(redisTemplate.opsForHash().get(redisKey, "username"));

    }


    /**
     * Redis List测试
     */
    @Test
    public void testLists() {
        String redisKey = "test:ids";

        // 存数据
        redisTemplate.opsForList().leftPush(redisKey, 101);
        redisTemplate.opsForList().leftPush(redisKey, 102);
        redisTemplate.opsForList().leftPush(redisKey, 103);

        // 统计数据量
        System.out.println(redisTemplate.opsForList().size(redisKey));
        // 取数据
        System.out.println(redisTemplate.opsForList().index(redisKey, 0));
        System.out.println(redisTemplate.opsForList().range(redisKey, 0, 2));

        // 弹出数据
        System.out.println(redisTemplate.opsForList().leftPop(redisKey));
        System.out.println(redisTemplate.opsForList().leftPop(redisKey));
        System.out.println(redisTemplate.opsForList().leftPop(redisKey));
    }


    /**
     * Redis set集合测试
     */
    @Test
    public void testSets() {
        String redisKey = "test:teachers";

        // 存数据
        redisTemplate.opsForSet().add(redisKey, "小明", "小李", "张三", "小王");

        // 统计数据量
        System.out.println(redisTemplate.opsForSet().size(redisKey));
        // 弹出数据
        System.out.println(redisTemplate.opsForSet().pop(redisKey));
        // 显示数据
        System.out.println(redisTemplate.opsForSet().members(redisKey));
    }


    /**
     * Redis SortedSets有序集合测试
     */
    @Test
    public void testSortedSets() {
        String redisKey = "test:students";

        // 存数据
        redisTemplate.opsForZSet().add(redisKey, "悟空", 100);
        redisTemplate.opsForZSet().add(redisKey, "八戒", 80);
        redisTemplate.opsForZSet().add(redisKey, "沙僧", 70);

        // 取数据
        System.out.println(redisTemplate.opsForZSet().zCard(redisKey));
        System.out.println(redisTemplate.opsForZSet().score(redisKey, "八戒"));
        System.out.println(redisTemplate.opsForZSet().reverseRank(redisKey, "悟空"));
        System.out.println(redisTemplate.opsForZSet().range(redisKey, 0, 1));
    }


    /**
     * Redis Key相关命令测试
     */
    @Test
    public void testKeys() {
        // 删除Key
        redisTemplate.delete("test:user");

        // 某个Key存在吗？
        System.out.println(redisTemplate.hasKey("test:user"));

        // Key的有效时间
        redisTemplate.expire("test:students", 10, TimeUnit.SECONDS);
    }


    // 绑定Key，便于多次重复访问同一个key
    @Test
    public void testBoundOperations() {
        String redisKey = "test:count";

        // 绑定Key
        BoundValueOperations operations = redisTemplate.boundValueOps(redisKey);

        System.out.println(operations.increment());
        System.out.println(operations.increment());
        System.out.println(operations.increment());
        System.out.println(operations.decrement());

        System.out.println(operations.get());
    }


}
```

</br>

## Redis事务管理

![Redis事务](https://i.loli.net/2020/01/31/SpVmEIsaOig7Cj6.png)

Redis的事务，是将事务中产生的命令都添加到事务队列中，待整个事务提交后再统一出队按序进行执行。所以事务中的命令不会立即执行，比如 你在事务中做了一个查询，这个查询不会立即返回结果。由于这样的机制，通常对于Redis的事务管理，我们不会使用声明式事务，更多的使用编程式事务。

另外，需要注意的是，Redis中的事务是**不会回滚**（roll back）的，有一条命令出现错误，之前的命令不会回滚，会面的命令也将继续执行，因为Redis认为这些错误是由程序员自身的错误引起的并且回滚也不能实质性的解决问题，Redis选择了这样更加简单高效的事务处理方式。

### **Redis事务相关的命令**

| 命令原型              | 命令描述                                                     |
| --------------------- | ------------------------------------------------------------ |
| MULTI                 | 用于标记事务的开始，其后执行的命令都将被存入命令队列，直到执行EXEC时，这些命令才会被原子的执行。 |
| EXEC                  | 执行在一个事务内命令队列中的所有命令，同时将当前连接的状态恢复为正常状态，即非事务状态。如果在事务中执行了WATCH命令，那么只有当WATCH所监控的Keys没有被修改的前提下，EXEC命令才能执行事务队列中的所有命令，否则EXEC将放弃当前事务中的所有命令。 |
| DISCARD               | 回滚事务队列中的所有命令，同时再将当前连接的状态恢复为正常状态，即非事务状态。如果WATCH命令被使用，该命令将UNWATCH所有的Keys。 |
| WATCH *key [key ...]* | 在MULTI命令执行之前，可以指定待监控的Keys，然而在执行EXEC之前，如果被监控的Keys发生修改，EXEC将放弃执行该事务队列中的所有命令。 |
| UNWATCH               | 取消当前事务中指定监控的Keys，如果执行了EXEC或DISCARD命令，则无需再手工执行该命令了，因为在此之后，事务中所有被监控的Keys都将自动取消。 |

### 示例

```java
/**
 * Redis编程式事务测试
 */
@Test
public void testTransactional() {
    Object obj = redisTemplate.execute(new SessionCallback() {
        @Override
        public Object execute(RedisOperations redisOperations) throws DataAccessException {
            String redisKey = "test:tx";
            // 启用事务
            redisOperations.multi();

            // 事务逻辑
            redisOperations.opsForSet().add(redisKey,"张三");
            redisOperations.opsForSet().add(redisKey,"李四");
            redisOperations.opsForSet().add(redisKey,"王五");
			// 该查询在事务内，不会有效
            System.out.println(redisOperations.opsForSet().members(redisKey));

            // 提交事务并返回
            return redisOperations.exec();
        }
    });

    System.out.println(obj);
}
```

**输出**

```properties
[]
[1, 1, 1, [王五, 李四, 张三]]
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
