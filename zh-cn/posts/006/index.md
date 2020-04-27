# TCP 三次握手


<!--more-->

## 1. TCP(Transmission Control Protocol)　传输控制协议 

 TCP是主机对主机层的传输控制协议，提供可靠的连接服务，采用三次握手确认建立一个连接。

TCP工作在网络 OSI 的七层模型中的第四层（Transport层），IP在第三层（Network层），ARP在第二层（Data Link层）；在第二层上的数据，我们把它叫Frame，在第三层上的数据叫Packet，第四层的数据叫Segment。 数据从应用层发下来，会在每一层都会加上头部信息，进行封装，然后再发送到数据接收端。  

- 一种**面向连接的、可靠的、基于字节流的**传输层通信协议
- 将应用层的数据流分割成报文段并发送给目标节点的TCP层
- 数据包都有序号，对方收到则发送ACK确认，未收到则重传
- 使用校验和来检验数据在传输过程中是否有误
- 在一个 TCP 连接中，仅有两方进行彼此通信。广播和多播不能用于 TCP
- TCP 给数据分节进行排序，并使用累积确认保证数据的顺序不变和非重复
- TCP 使用**滑动窗口机制**来实现流量控制，通过动态改变窗口的大小进行拥塞控制

在进行数据传输之前，需要在传输数据的两端（客户端和服务器端）创建一个连接，这个连接由一对插口地址唯一标识，即是在IP报文首部的源IP地址、目的IP地址，以及TCP数据报首部的源端口地址和目的端口地址 

## 2. TCP头部结构 
![](https://i.loli.net/2019/12/09/dLhNgTlbYtBA3FW.jpg) 

1. `Source port`源端口、`Destination port`目的端口，他们各占用2个字节。TCP和UDP的数据包都是不包含IP地址信息的，因为那是IP层要处理的，但是TCP和UDP都会有源端口和目的端口。在计算机本地我们可以用PID唯一标识一个进程，但是在网络中不同计算机的进程可能会有相同的PID，可以通过在传输层中使用协议端口号来解决，这样我们就可以用 IP+协议+端口号来唯一标识网络中的一个进程.

2. `Sequence Number`序号：Seq序号，占4个字节，用来标识从TCP源端向目的端发送的字节流，发起方发送数据时对此进行标记。是包的序号，用来解决网络包乱序（reordering）问题。

3. `Acknowledgement Number`确认序号：Ack序号，占4个字节，只有ACK标志位为1时，确认序号字段才有效，Ack=Seq+1。

4. `Data offset`数据偏移：由于头部有可选字段长度不固定，因此它指出TCP报文的数据离TCP报文起始有多远。

5. `Reserved`保留域：保留以后使用的。

6. `TCP Flags` 标志位：常见的有6个，即URG、ACK、PSH、RST、SYN、FIN等，具体含义如下：

   `URG` 紧急指针标志：为1时有效，为0则忽略。

   `ACK` **确认序号标志**：为1时确认号有效，为0标识报文中不含确认信息。 TCP 规定在连接建立后传送的所有报文段都必须把 ACK 置为一。 

   `PSH` push标志：为1时，指示接收方接收到报文后应该尽快将这个报文交给应用层程序，而不是在缓冲区排队。

   `RST` 重置连接标志：重置连接，重置由于主机崩溃或其它原因而出现的异常连接，或者用于拒绝非法的报文段和拒绝连接请求。

   `SYN` **同步序号**：用于建立连接过程。 当SYN=1，ACK=0时，表示当前报文段是一个连接请求报文。当SYN=1，ACK=1时，表示当前报文段是一个同意建立连接的应答报文。 

   `FIN` **finish标志**：用于释放连接。为1时， 该字段为一表示此报文段是一个释放连接的请求报文 

7. ` Window Size`窗口大小，指滑动窗口的大小，用于告知发送端接受方的缓存大小，以此控制发送端发送数据的速率从而进行流量控制 。

8. `Checksum`校验和，指奇偶校验，对整个TCP报文以16位进行计算所得，由发送端计算和存储并由接收端验证。

9. `Urgent Pointer` 紧急指针，当TCP Flags中的`URG`为1时才有效，指出本TCP报文中紧急数据的字节数。

10. `TCP Options` 可选项，其长度可变，定义一些其它的可选参数。



## 3. TCP三次握手

“握手”是为了**建立连接**，TCP三次握手的流程图如下：

![](https://i.loli.net/2019/12/09/tbrGiOqwZ6Jpe3K.jpg) 

其中，前两次握手是不允许携带数据的，第三握手可以携带数据，如果第三次握手不携带数据就可以不消耗序号。

**第一次握手**

建立连接时， Client将标志位SYN置为1，随机产生一个值seq=x ， 并将该数据包发送给Server ，Client进入SYN_SEND状态，等待Server 确认；

**第二次握手**

Server 收到数据包后由标志位SYN=1知道Client请求建立连接，Server将标志位SYN和ACK都置为1  ， ack=x+1 ，同时随机产生一个值seq=y ， 并将该数据包发送给Client以确认连接请求，Server进入SYN_RCVD状态。 

**第三次握手**

Client收到Server的SYN+ACK包， 检查ack是否为y+1，标志位ACK是否为1，如果正确则将标志位ACK置为1，ack=y+1，seq=x+1， 并将该数据包发送给Server ;  Server检查ack是否为y+1，ACK是否为1，如果正确则连接建立成功，Client和Server进入ESTABLISHED状态，完成三次握手，随后Client与Server之间可以开始传输数据了。 

**注意**

- 不要将确认序号ack与标志位中的ACK搞混了。
- 确认方ack=发起方seq+1，两端配对。



## 4. 三次握手抓包分析

筛选过滤，将我们访问的地址作为筛选的 目标地址 和 源地址，筛选出 源地址 是因为三次握手中，访问的地址也会作为源地址向我们的计算机发送请求。

```c
ip.dst==61.128.252.10 or ip.src==61.128.252.10
```

![](https://i.loli.net/2019/12/09/pbmqrR3fvd2xls4.png)

顺便查阅归纳下，**Wireshark的过滤规则**：

**1. 过滤IP**

```c
ip.src eq 192.168.1.108 or ip.dst eq 192.168.1.108
或者ip.addr eq 192.168.1.107
```

**2. 过滤端口**

```c
tcp.port eq 80 //不管端口是源端口的还是目标端口都显示
tcp.port eq 80 or udp.port eq 80
tcp.dstport == 80 //只显tcp协议的目标端口80
tcp.srcport == 80 //只显tcp协议的源端口80
tcp.port >= 1 and tcp.port <= 80 //端口范围过滤
```

**3. 过滤协议**

```c
直接输入协议名：tcp、udp、arp、icmp、http、smtp、ftp、dns、msnms、ip、ssl、oicq、bootp等
排除协议过滤：如 !arp 或者 not arp
```

**4. 过滤MAC**

```c
eth.dst == 48:7d:2e:98:61:d1 或 eth.dst==48-7d-2e-98-61-d1 //过滤目标mac
eth.src eq 48:7d:2e:98:61:d1 //过滤来源mac
```

小于 less than（lt）

等于 eq

大于等于 ge

不等 ne

**5. 包长度过滤**

```c
udp.length == 26 //这个长度是指udp本身固定长度8加上udp下面那块数据包之和
tcp.len >= 7   //指的是ip数据包(tcp下面那块数据),不包括tcp本身
ip.len == 94 //除了以太网头固定长度14,其它都算是ip.len,即从ip本身到最后
frame.len == 119 //整个数据包长度,从eth开始到最后
```

**6. http模式过滤**

```c
例子:
http.request.method == “GET”
http.request.method == “POST”
http.request.uri == “/img/logo.png”
http contains “GET”
http contains “HTTP/1.”
http.request.method == “GET” && http contains “Host: “

http.host == xxx.com	// 过滤 host
http.response == 1	// 过滤所有的 http 响应包
http.response.code == 302	// 过滤状态码 202
http.request.method==POST 	// 过滤 POST 请求包
http.cookie contains xxx	// cookie 包含 xxx
http.request.uri=="/robots.txt"	//过滤请求的uri，取值是域名后的部分
http.request.full_uri=="http://1.com"	// 过滤含域名的整个url
http.server contains "nginx"	//过滤http头中server字段含有nginx字符的数据包
http.content_type == "text/html"	//过滤content_type是text/html
http.content_encoding == "gzip"	//过滤content_encoding是gzip的http包
http.transfer_encoding == "chunked"	//根据transfer_encoding过滤
http.content_length == 279
http.content_length_header == "279"	//根据content_length的数值过滤
http.request.version == "HTTP/1.1"	//过滤HTTP/1.1版本的http包，包括请求和响应
```



## 5. 问题

### 为什么需要三次握手才能建立起连接

为了初始化Sequence Number的初始值，通信双方要互相通知对方自己的Sequence Number，作为以后数据通信的序号，以保证接收到的数据不会因为网络上的问题而乱序，即TCP会用这个序号来拼接数据， 确保数据能够完整传输。 



### 首次握手的隐患--SYN超时

**原因**

三次握手过程中，Server收到Client的SYN，回复SYN-ACK后（ Server收到Client的ACK之前的TCP连接称为半连接（`half-open connect`），此时Server处于SYN_RCVD状态 ）， Cient就掉线了，Server端没有收到Client的ACK确认。 于是 Server会不断重试直到超时，在Linux系统中会默认进行5次重试（每次重试隔时间从1s开始每次都翻售 ，即1+2+4+6+8+16+32=63秒）TCP才断开连接。

 **SYN攻击**

Client在短时间内伪造大量不存在的IP地址，并向Server不断地发送SYN包，Server回复确认包，并等待Client的确认，由于源地址是不存在的，因此，Server需要不断重发直至超时，这些伪造的SYN包将长时间占用未连接队列，把Server的syn连接的队列耗尽 ，导致正常的SYN请求因为队列满而被丢弃，从而引起网络堵塞甚至系统瘫痪。 

**检测**

SYN攻击时一种典型的DDOS攻击，检测SYN攻击的方式非常简单，即当Server上有大量半连接状态且源IP地址是随机的，则可以断定遭到SYN攻击了，使用如下命令可以可以查看`SYN_RECV`状态：

```c
# netstat -nap | grep SYN_RECV
```

**针对SYN Flood的防护措施**

Linux下给了一个叫`tcp_syncookies`的参数来应对这个事——当SYN队列满了后，TCP会通过源地址端口、目标地址端口和时间戳打造出一个特别的`Sequence Number`发回去（又叫SYN Cookie），如果是攻击者则不会有响应，如果是正常连接，则会把这个 `SYN Cookie`发回来，然后服务端可以通过cookie建连接（即使你不在SYN队列中）。请注意，慎用`tcp_syncookies`来处理正常的大负载的连接的情况。因为，`synccookies`是妥协版的TCP协议，并不严谨。 对于正常的请求，可以调整以下三个参数：

- `tcp_synack_retries` ，用于减少重试次数；
- `tcp_max_syn_backlog`，用于增大SYN连接数；
- `tcp_abort_on_overflow` ，处理不过来，直接拒绝连接了。 

**保活机制**

如果建立连接后，Client出现故障，向对方发送保活探测报文，如果未收到响应则继续发送，尝试次数达到保活探测数仍未收到响应则中断连接



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
