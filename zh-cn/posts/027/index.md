# IDEA 引入依赖失败的解决办法


<!--more-->

## 问题

今天学习异步消息，用到 FastJson 对 json 进行解析和生成，但是在 `pom.xml` 中引入下面的依赖描述时，等待许久也无法下载完成。之后尝试进行科学上网，并开启全局代理，依然是下载失败或下载不完全。

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>fastjson</artifactId>
   <version>1.2.62</version>
</dependency>
```

</br>

## 解决办法

1.在 [MVNrepository](https://mvnrepository.com/) 官网直接找到对应的 jar 包链接，[点击进行下载](https://repo1.maven.org/maven2/com/alibaba/fastjson/1.2.62/fastjson-1.2.62.jar)

![MVNrepository](https://i.loli.net/2020/01/27/gOTSzU6IrYm1WtA.png)

2.找到下载jar包的所在目录，使用maven命令直接进行安装，例如：

```properties
mvn install:install-file -Dfile=E:\Download\Chrome\fastjson-1.2.62.jar -DgroupId=com.alibaba -DartifactId=fastjson -Dversion=1.2.62 -Dpackaging=jar
```

![对应参数](https://i.loli.net/2020/01/27/2AB9iKW6RIdGCXT.png)

安装成功后显示存放路径

![安装成功](https://i.loli.net/2020/01/27/L4Hk3bv5F1CrwaM.png)



重启IDEA，重新加载依赖，就可以成功了！同理，其它的依赖应该也可以这样操作。

</br>

## Update 2020/01/30

博主最后找到了原因（来自初学者的无奈），是因为Maven的配置路径被修改了，可能是因为我之前对IDEA进行了一次更新，导致部分配置丢失，恢复了默认配置。

![修改为本地的Maven，配置文件改为本地Maven的配置文件](https://i.loli.net/2020/01/30/goHQhP8tO1C4a7Y.png)

其实这很容易发现，当SpringBoot添加依赖的时候，点击下方的进度条查看详细，你就可以看到该包的下载路径，如果不是来自于[maven.aliyun.com](maven.aliyun.com)，是来自apache、maven.org这些的话下载会非常慢甚至失败！

所以你需要在IDEA的setting中配置Maven为用户自己的Maven，但是主要的是Maven的配置文件无论用自带的还是自己的Mavan你都需要添加阿里云的镜像，这样在国内导入依赖的时候会快很多。

在本地Maven配置文件settings.xml中修改（或添加）aliyun镜像：

```xml
<mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>https://maven.aliyun.com/repository/central</url>
</mirror>
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
