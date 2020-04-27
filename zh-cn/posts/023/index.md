# Web 项目的一些调试技巧


<!--more-->

## 1. 响应状态码的含义

`200 OK`，表明请求已经成功. 默认情况下状态码为200的响应可以被缓存。

`302 Found`，重定向，服务器发送 `302` 状态码和一个新的 `url ` 让浏览器再发一次新的请求，实现功能的跳转。

`404 Not Found`，往往是路径配置有误。

`500 Internal Server Error`，服务器接受到了请求，但是在处理的过程中发生了问题。

详细信息请参考：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status



## 2. 服务端断点调试技巧

在IDEA中，打上断点后，需要以 Debug 方式运行程序。

按 `F8` 可以执行下一行

按 `F7` 可以进入到当前行执行的方法内部，查看更详细的调试信息

按 `F9` 可以一直向下执行直到遇到下一个断点处，注意在调试期间也可以在之后的程序代码中继续打断点，用于跳过一些繁杂的循环或者确定正确的程序段。

有些信息可能是以Hash值显示，这时可以在下方调试框中展开看信息信息：

![](https://i.loli.net/2019/12/28/ZM548VEUdLiYyaW.png)

如果断点太多，你还可以批量管理断点：

![](https://i.loli.net/2019/12/28/jRGKmuvIPdsegJk.png)



## 3. 客户端断点调试技巧

这里主要是指对 JS 的调试。以Chrome浏览器为例，调出如下页面，`F10`进入下一条语句，`F11`进入当前语句，`F8`到下一个断点或执行到底。

![](https://i.loli.net/2019/12/28/hBezrIyuPVgJNUb.png)



## 4. 设置日志级别，并将日志输出到不同的终端

Spring boot内置默认的日志工具 [Logback](https://logback.qos.ch) ，[参考手册](http://logback.qos.ch/manual/index.html)

```java
package org.slf4j; 
public interface Logger {
  // 日志级别，从低到高，等于或高于设定级别的日志将会在控制台输出
  public void trace(String message);
  public void debug(String message);
  public void info(String message); 
  public void warn(String message); 
  public void error(String message); 
}
```

logger日志级别被Spring Boot整合后，可直接在 application.properties 文件中配置

```properties
# 配置logger日志级别，以便查看sql语句带来的错误等，方便查错
logging.level.me.leihao.community = debug
# 项目上线后是没有控制台的，我们可以将日志打印到文件里
logging.file.name = E:/Test/community.log
```



在实际的项目开发中，日志文件是非常重要的，我们的设置会更复杂一些，会将不同级别的日志分成不同的文件，对单个文件如果太大也会进行拆分。

首先，先注释掉 application.properties 文件中配置的 logger 日志

然后，在 resources 目录下引入名为 `logback-spring.xml` 的配置文件（注意命名是Spring规定的，不要乱改）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!--你的项目名-->
    <contextName>community</contextName>
    <!--日志存储路径-->
    <property name="LOG_PATH" value="E:/Test/log"/>
    <!--子级目录，以区分不同项目-->
    <property name="APPDIR" value="community"/>

    <!-- error file -->
    <appender name="FILE_ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--分块规则-->
        <file>${LOG_PATH}/${APPDIR}/log_error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/${APPDIR}/error/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!--单个日志大小-->
                <maxFileSize>5MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志最长保留时间-->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <!--以追加方式存储日志-->
        <append>true</append>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--日志输出格式：日期、级别、线程、类、文件和行、提示-->
            <pattern>%d %level [%thread] %logger{10} [%file:%line] %msg%n</pattern>
            <!--字符集-->
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!--日志级别-->
            <level>error</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- warn file -->
    <appender name="FILE_WARN" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${APPDIR}/log_warn.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/${APPDIR}/warn/log-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>5MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d %level [%thread] %logger{10} [%file:%line] %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- info file -->
    <appender name="FILE_INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${APPDIR}/log_info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/${APPDIR}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>5MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d %level [%thread] %logger{10} [%file:%line] %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- console 打印到控制台 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d %level [%thread] %logger{10} [%file:%line] %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <!--控制台日志级别-->
            <level>debug</level>
        </filter>
    </appender>

    <!--对某个包日志级别的单独声明-->
    <logger name="me.leihao.community" level="debug"/>

    <root level="info">
        <appender-ref ref="FILE_ERROR"/>
        <appender-ref ref="FILE_WARN"/>
        <appender-ref ref="FILE_INFO"/>
        <appender-ref ref="STDOUT"/>
    </root>

</configuration>
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
