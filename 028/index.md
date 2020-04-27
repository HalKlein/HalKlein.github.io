# Spring事务管理


<!--more-->

## 什么是事务

事务是由N步数据库操作序列组成的逻辑执行单元，这系列操作要么全执行，要么全放弃执行。

## 事务的特性（ACID）

![事务的特性](https://i.loli.net/2020/01/28/NesIWh9K742wjdP.png)

原子性（Atomicity）：事务是应用中不可再分的最小执行体。

一致性（Consistency）：事务执行的结果，须使数据从一个一致性状态，变为另一个一致性状态。

隔离性（Isolation）：各个事务的执行互不干扰，任何事务的内部操作对其他的事务都是隔离的。

持久性（Durability）：事务一旦提交，对数据所做的任何改变都要记录到永久存储器中。



## 事务的隔离性

- 常见的并发异常

  第一类丢失更新、第二类丢失更新。

  脏读、不可重复读、幻读。

- 常见的隔离级别

  Read Uncommitted：读取未提交的数据。

  Read Committed：读取已提交的数据。

  Repeatable Read：可重复读。

  Serializable：串行化 （性能低）。

### 常见的并发异常

**第一类丢失更新**

某一个事务的回滚，导致另外一个事务已更新的数据丢失了。

![第一类丢失更新](https://i.loli.net/2020/01/28/r3B2UXDYupGqRtb.png)



**第二类丢失更新**

某一个事务的提交，导致另外一个事务已更新的数据丢失了。

![第二类丢失更新](https://i.loli.net/2020/01/28/PHW9FnDOIZ6gRid.png)



**脏读**

某一个事务，读取了另外一个事务未提交的数据。

![脏读](https://i.loli.net/2020/01/28/uDXo7y28nI3ZVBs.png)



**不可重复读**

某一个事务，对同一个数据前后读取的结果不一致。

![不可重复读](https://i.loli.net/2020/01/28/ZdOCKQrPpyBYE36.png)



**幻读**

某一个事务，对同一个表前后查询到的行数不一致。

![幻读](https://i.loli.net/2020/01/28/CoYSLuRg2tQPWMf.png)



### 常见的隔离级别

![常见的隔离级别](https://i.loli.net/2020/01/28/1lMsQTVYJGynihg.png)

一般来说，我们选择中间两种。第一中几乎没有改善；第四种由于性能非常低，一般是不选用的，除非是金融类业务，政府部分数据等才考虑。



## 实现机制

### 悲观锁（数据库）

**共享锁（S锁）**

事务A对某数据加了共享锁后，其他事务只能对该数据加共享锁，但不能加排他锁。

**排他锁（X锁）**

事务A对某数据加了排他锁后，其他事务对该数据既不能加共享锁，也不能加排他锁。

### 乐观锁（自定义）

**版本号、时间戳等**

在更新数据前，检查版本号是否发生变化。若变化则取消本次更新，否则就更新数据（版本号+1）。



## [Spring 事务管理](https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#spring-data-tier)

- 声明式事务

  通过XML配置，声明某方法的事务特征。

  通过注解，声明某方法的事务特征。

- 编程式事务

  通过 TransactionTemplate 管理事务，并通过它执行数据库的操作。

声明式事务基于 AOP，将具体业务逻辑与事务处理解耦。声明式事务管理使业务代码逻辑不受污染，因此在实际使用中声明式事务用的比较多。声明式事务有两种方式，一种是在配置文件（xml）中做相关的事务规则声明，另一种是基于@Transactional 注解的方式，注释配置是目前流行的使用方式。

##### @Transactional 注解的属性信息

| 属性名           | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| name             | 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。 |
| propagation      | 事务的传播行为，默认值为 REQUIRED。                          |
| isolation        | 事务的隔离度，默认值采用 DEFAULT。                           |
| timeout          | 事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。 |
| read-only        | 指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。 |
| rollback-for     | 用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。 |
| no-rollback- for | 抛出 no-rollback-for 指定的异常类型，不回滚事务。            |

### 声明式事务示例

**业务方法**

```java
@Service
/*Spring容器管理的Bean默认是单例的，如果需要每次getBean都新建一个实例，需要加上如下
@Scope("singleton")是默认的（单例的）*/
@Scope("prototype") // 非单例，（极少使用）
public class AlphaService {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private DiscussPostMapper discussPostMapper;

    /**
     * Spring 事务管理测试
     * Spring 事务管理，加注解：@Transactional，设置事务级别：isolation = Isolation.READ_COMMITTED
     * propagation 事务的传播机制，解决事务交叉问题。如【 业务方法A 去调用 业务方法B 】他们都加有事务注解，那么B的事务应该以谁为准？的问题
     *      REQUIRED 支持当前事务（外部事务，上例事务A），如果不存在则创建新事务
     *      REQUIRES_NEW 创建一个新的事务，并且暂定当前事务（外部事务）
     *      NESTED 如果存在当前事务（外部事务），则嵌套在该事务中执行（B有独立的提交和回滚），否则就和 REQUIRED 一样
     *
     * @return
     */
    @Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
    public Object save1() {
        // 新增用户
        User user = new User();
        user.setUsername("alpha");
        user.setSalt(CommunityUtil.generateUUID().substring(0, 5));
        user.setPassword(CommunityUtil.md5("123") + user.getSalt());
        user.setEmail("alpha@qq.com");
        user.setHeaderUrl("http://image.nowcoder.com/99t.png");
        user.setCreateTime(new Date());
        userMapper.insertUser(user);

        // 新增帖子
        DiscussPost post = new DiscussPost();
        post.setUserId(user.getId());
        post.setTitle("Hello");
        post.setContent("新人报道");
        post.setCreateTime(new Date());
        discussPostMapper.insertDiscussPost(post);

        // 人工制造错误，用于测试
        Integer.valueOf("abc");

        return "OK";
    }

}
```

**测试类**

```java
@SpringBootTest
@ContextConfiguration(classes = CommunityApplication.class)
public class TransactionTests {

    @Autowired
    private AlphaService alphaService;

    @Test
    public void testSave1() {
        Object obj = alphaService.save1();
        System.out.println(obj);
    }

}
```

**测试结果**

没有在数据库中新增用户，也没有新增一条帖子。



### 编程式事务示例

**业务方法**

```java
@Service
/*Spring容器管理的Bean默认是单例的，如果需要每次getBean都新建一个实例，需要加上如下
@Scope("singleton")是默认的（单例的）*/
@Scope("prototype") //非单例，（极少使用）
public class AlphaService {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private DiscussPostMapper discussPostMapper;

    @Autowired
    private TransactionTemplate transactionTemplate;

    
    /**
     * 编程式事务示例
     * 1. 注入 TransactionTemplate ，这个Bean是Spring自动创建并装配到容器中的，所以直接注入即可
     * @return
     */
    public Object save2(){
        // 设置事务级别
        transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
        // 设置传播机制
        transactionTemplate.setPropagationBehavior(transactionTemplate.PROPAGATION_REQUIRED);

        return transactionTemplate.execute(new TransactionCallback<Object>() {
            @Override
            public Object doInTransaction(TransactionStatus transactionStatus) {
                // 新增用户
                User user = new User();
                user.setUsername("beta");
                user.setSalt(CommunityUtil.generateUUID().substring(0, 5));
                user.setPassword(CommunityUtil.md5("123") + user.getSalt());
                user.setEmail("beta@qq.com");
                user.setHeaderUrl("http://image.nowcoder.com/999t.png");
                user.setCreateTime(new Date());
                userMapper.insertUser(user);

                // 新增帖子
                DiscussPost post = new DiscussPost();
                post.setUserId(user.getId());
                post.setTitle("你好");
                post.setContent("我是新人");
                post.setCreateTime(new Date());
                discussPostMapper.insertDiscussPost(post);

                // 人工制造错误，用于测试
                Integer.valueOf("abc");

                return "OK";
            }
        });
    }

}
```

**测试类**

```java
@SpringBootTest
@ContextConfiguration(classes = CommunityApplication.class)
public class TransactionTests {

    @Autowired
    private AlphaService alphaService;


    @Test
    public void testSave2() {
        Object obj = alphaService.save2();
        System.out.println(obj);
    }

}
```

**测试结果**

同上，没有在数据库中新增用户，也没有新增一条帖子。

**PS**

一般情况下我们选用简单的声明式事务，但是如果有些业务方法过于复杂，而我们只希望管理其中一部分事务，则选用编程式事务。



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
