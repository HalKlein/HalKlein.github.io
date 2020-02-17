# 采坑小记


<!--more-->

## 1. 提高 git clone 的速度

之前建博客的时候 git clone 感觉挺快的，现在找了几个项目练习有几十M大，发现git clone的速度就50k/s左右实在是受不了！自己是有小飞机的，开了全局代理，居然还是很慢

最后找到原因：git 的代理要单独配置，命令如下：

```c
git config --global http.https://github.com.proxy socks5://127.0.0.1:1086
git config --global https.https://github.com.proxy socks5://127.0.0.1:1086
```

socks5代理端口在这里查看：



MacOS 未知来源：sudo spctl --master-disable



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>


