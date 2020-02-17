# 使用 Spring AOP 统一记录日志


<!--more-->

## 需求

![需求](https://i.loli.net/2020/01/30/vaD453kgpyC9nI1.png)

如果我们每次都在业务组件中记录日志，但是记录日志并不是业务需求，它属于系统需求。



## AOP的概念

**Aspect Oriented Programing**

​	即面向方面（切面）编程。
​	AOP是一种编程思想，是对OOP的补充，可以进一步提高编程的效率。

![AOP的概念](https://i.loli.net/2020/01/30/hZgODSCvFdHEliQ.png)



## **AOP的术语**

![AOP的术语](https://i.loli.net/2020/01/30/FbLUmHqE5BWy7Nx.png)

**Target**：目标对象，程序中的每一个要处理的Bean我们称之为Target

**Joinpoint**：连接点，目标对象有很多地方能够被织入代码，其可以织入代码的地方，我们称之为连接点

**Aspect**：切面组件，AOP解决统一处理这些系统需求的方式是将代码定义到一个额外的Bean中，Pointcut声明代码到底要织入到哪些对象的哪些位置，Advice通知方法说明这个切面组件要处理怎样的逻辑。

**Weaving**：切面组件在程序运行之前或之后，通过框架织入到某些连接点之上，在编译时织入效率会很高，但是系统未运行时，可能有些条件我们并不能得到，就需要在运行时织入，运行时织入很方便灵活，但是效率不高。



## AOP的实现

**AspectJ**

​	AspectJ是**语言级**的实现（新的语言），它扩展了Java语言，定义了AOP语法。

​	AspectJ在**编译期**织入代码，支持各种类型的连接点，它有一个专门的编译器，用来生成遵守Java字节码规范的class文件。

**Spring AOP**

​	Spring AOP使用纯 **Java实现**，它**不需要专门的编译过程**，也**不需要特殊的类装载器**。

​	Spring AOP在 **运行时** 通过代理的方式织入代码，**只支持方法类型的连接点**。

​	Spring**支持**对**AspectJ**的集成。



## Spring AOP 代理

**JDK 动态代理**

​	Java提供的动态代理技术，可以在运行时创建**接口**的代理实例。

​	Spring AOP**默认**采用此种方式，在接口的代理实例中织入代码。

**CGLib 动态代理**

​	采用底层的**字节码技术**，在运行时创建**子类**代理实例。

​	当目标对象不存在接口时，Spring AOP会采用此种方式，在子类实例中织入代码。



## 示例 记录用户访问方法的信息

```java
@Component
// 切面组件
@Aspect
public class ServiceLogAspect {

    // 记录日志
    private static final Logger logger = LoggerFactory.getLogger(ServiceLogAspect.class);

    /**
     * 定义切点@Pointcut，其中筛选目标，
     * 第1个*表示返回值任意，
     * 第2个*表示所有组件，
     * 第3个*表示所有的方法，
     * (..)表示所有的参数
     */
    @Pointcut("execution(* me.halklein.community.service.*.*(..))")
    public void pointcut() {

    }

    // 在连接点开始@Before，pointcut()表示切点是什么
    @Before("pointcut()")
    public void before(JoinPoint joinPoint) {
        // 记录格式：用户[IP地址]，在[xxx]，访问了[me.halklein.community.service.xxx()]
        // 获取Request对象
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        // 获取访问者IP
        String ip = request.getRemoteHost();
        String now = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        // 获取类名，方法名
        String target = joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName();
        logger.info(String.format("用户[%s]，在[%s]，访问了[%s].", ip, now, target));
    }

}
```

**查看日志**

![](https://i.loli.net/2020/01/30/aCo5ALIVRyr6l9U.png)



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
