# Spring 统一处理异常


<!--more-->

在系统出现错误时，我们希望系统给用户图形化的反馈，而不是一些状态错误信息。

![](https://i.loli.net/2020/01/30/WfvGOd1js3tcQMe.png)

各个层次都会出现错误，但其实最终都会抛出异常给表现层。所以我们只用在表现层中捕获异常，处理异常，就能处理系统中所有的异常。



## **SpringBoot自动化处理异常**

SpringBoot统一处理异常，我们需要在templates目录下新建`error`目录，然后在里面新建错误页面文件，且文件的名字一定要对应错误状态码。就可以简单实现对异常的统一处理。



## **Spring处理异常**

但是SpringBoot统一处理异常，并不是很详细，有时候并不能满足我们的需要，我们可能需要进行更加详细和复杂的操作，这时可以使用Spring进行更详细的配置。

**@ControllerAdvice**

​	用于修饰类，表示该类是Controller的全局配置类。

​	在此类中，可以对Controller进行如下三种全局配置：**异常处理方案、绑定数据方案、绑定参数方案**。

**@ExceptionHandler**

​	用于修饰方法，该方法会在Controller出现异常**后**被调用，用于**处理捕获到的异常**。

**@ModelAttribute**

​	用于修饰方法，该方法会在Controller方法执行**前**被调用，用于为Model对象**绑定参数**。

**@DataBinder**

​	用于修饰方法，该方法会在Controller方法执行**前**被调用，用于**绑定参数**的转换器。

在controller包下advice包，里面新建ExceptionAdvice类，用于处理全局异常

```java
/**
 * Controller异常全局配置类
 * annotations = Controller.class 只扫描Controller组件
 */
@ControllerAdvice(annotations = Controller.class)
public class ExceptionAdvice {

    private static final Logger logger = LoggerFactory.getLogger(ExceptionAdvice.class);

    /**
     * @ExceptionHandler 声明该方法用于处理异常
     * Exception.class 处理所有异常
     */
    @ExceptionHandler({Exception.class})
    public void handleException(Exception e, HttpServletRequest request, HttpServletResponse response) throws IOException {
        logger.error("服务器发生异常: ", e.getMessage());
        // 遍历详细的异常栈信息
        for (StackTraceElement element : e.getStackTrace()) {
            logger.error(element.toString());
        }

        // 重定向
        // 判断是普通请求还是异步请求
        String xRequestWith = request.getHeader("x-requested-with");

        if ("XMLHttpRequest".equals(xRequestWith)) {
            // XMLHttpRequest 异步请求

            // application/json 返回字符串后，浏览器会自动转换为json对象
            // application/plain 返回一个普通字符串，可以是json格式，浏览器需要认为转换为js对象
            response.setContentType("application/plain;charset=utf-8");
            // 输出
            PrintWriter writer = response.getWriter();
            writer.write(CommunityUtil.getJsonString(1, "服务器异常！"));
        } else {
            // 是一个普通请求，发生错误，我们进行重定向
            response.sendRedirect(request.getContextPath() + "/error");
        }
    }

}
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
