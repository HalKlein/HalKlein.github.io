# 常用的元注解


<!--more-->

## 常用的元注解

当我们需要自定义注解时，要用到元注解定义我们自己的注解（所谓元注解，就是注解其它注解的注解），其中常见的元注解有以下四个：

**@Target**

由于标识我们自定义注解的使用范围。从源码可以看到可以赋值为ElementType。

```java
package java.lang.annotation;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Target {
    ElementType[] value();
}
```

其中ElementType定义如下

```java
public enum ElementType {
    TYPE,	// 类、接口、枚举类
    FIELD,	// 成员变量（包括：枚举常量）
    METHOD,	// 成员方法
    PARAMETER,	// 方法参数
    CONSTRUCTOR, 	// 构造方法
    LOCAL_VARIABLE,	// 局部变量
    ANNOTATION_TYPE,	// 注解类
    PACKAGE,	// 包
    TYPE_PARAMETER,	// 类型参数，JDK 1.8 新增
    TYPE_USE,	// 使用类型的任何地方，JDK 1.8 新增
    MODULE;	// 允许在模块上使用注解，JDK 1.9新增

    private ElementType() {
    }
}
```



**@Retention**

用于说明自定义注解的有效时间或者说保留的时间（如：编译时有效、运行时有效等）

```java
package java.lang.annotation;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Retention {
    RetentionPolicy value();
}
```

```Java
package java.lang.annotation;

public enum RetentionPolicy {
    SOURCE,	//源文件
    CLASS,	//编译时（默认）
    RUNTIME;	//运行时

    private RetentionPolicy() {
    }
}
```



**@Document**

声明在生成帮助文档的时候要不要把这个自定义注解信息也带上。

```java
package java.lang.annotation;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Documented {
}
```



**@Inherited**

使被它修饰的自定义注解具有继承性。子类继承该自定义注解之后也具有该注解。

```java
package java.lang.annotation;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Inherited {
}
```



**PS**

一般来说，我们自定义注解，@Target和@Retention是都会用到的，用来指定作用的范围和时机



## 如何读取注解

通过AnnotatedElement接口实现对自定义注解的解析

Method.getDeclaredAnnotations（）
Method.getAnnotation（Class<T> annotationClass）

**看个例子：**

```Java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    // 如果拦截到的是一个方法 : instanceof HandlerMethod
    if (handler instanceof HandlerMethod) {
        // 转型，方便获取内容
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Method method = handlerMethod.getMethod();
        // 取注解，取含有自定义注解：LoginRequired，的方法
        LoginRequired loginRequired = method.getAnnotation(LoginRequired.class);
        // 取注解后发现不为空，说明该方法需要登录才能访问，并且检测不到当前用户
        if (loginRequired != null && hostHolder.getUser() == null) {
            // 使用response重定向到登录页
            response.sendRedirect(request.getContextPath() + "/login");
            // 拒绝后续访问
            return false;
        }
    }
    return true;
}
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>

