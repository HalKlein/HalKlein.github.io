# TCP 滑动窗口


<!--more-->

## 1. RTT 和 RTO

- RTT（round trip time） 往返时延 ：发送一个数据包到收到对应的ACK，所花费的时间。
- RTO（retransamission timeout） ：重传时间间隔，由RTT计算得出，不是写死的。

由TCP的重传机制，我们知道Timeout的设置对重置非常重要，设置长了会影响重发效率，设置短了可能导致还没丢失就又重传了。 而网络状况更是复杂，所以我们没有办法设置一个死的Timeout值 。为了动态地设置，TCP引入了RTT， 这样发送端就大约知道需要多少的时间，从而可以方便地设置Timeout——RTO（ Retransmission TimeOut ） ， 以让重传机制更高效。 [RFC793](http://tools.ietf.org/html/rfc793) 中有对算法详细的定义。

## 2. TCP的滑动窗口

我们都知道，TCP必需要解决的可靠传输以及包乱序（reordering）的问题，所以，TCP必需要知道网络实际的数据处理带宽或是数据处理速度，这样才不会引起网络拥塞，导致丢包。 所以，TCP引入了一些技术和设计来做网络流控，Sliding Window是其中一个技术。TCP头里有一个字段叫Window，又叫Advertised-Window，这个字段是接收端告诉发送端自己还有多少缓冲区可以接收数据。于是发送端就可以根据这个接收端的处理能力来发送数据，而不会导致接收端处理不过来。

### **下面，模拟TCP缓冲区：**



![img](https://i.loli.net/2019/12/09/JUC5zmwxbLkyvFq.png)



- 发送端的LastByteAcked指向了被接收端Ack过的位置（表示成功发送确认），LastByteSent表示发出去了，但还没有收到成功确认的Ack，LastByteWritten指向的是上层应用正在写的地方。
- 接收端LastByteRead指向了TCP缓冲区中读到的位置，NextByteExpected指向的地方是收到的连续包的最后一个位置，LastByteRcved指向的是收到的包的最后一个位置，我们可以看到中间有些数据还没有到达，所以有数据空白区。

### **窗口数据计算过程：**

- 接收端在给发送端回ACK中，会汇报自己的 **AdvertisedWindow** = MaxRcvBuffer – （LastByteRcvd – Last Byte Read），（LastByteRcvd – LastByteRead）表示接收到的数据已占用缓存的大小;
- 发送方根据Advertised Window的值，要保证已发送未确认的数据<= Advertised Window的大小，待发送数据大小**EffectiveWindow **= AdvertisedWindow – (LastByteSent – LastByteAcked)

### 滑动窗口的基本原理

**TCP会话的发送方，滑动窗口示意图**



![发送方滑动窗口示意图](https://i.loli.net/2019/12/09/cnU7JxGLl1hySEe.png)发送方滑动窗口示意图



**TCP会话的接收方，滑动窗口示意图**



![接收方滑动窗口示意图](https://i.loli.net/2019/12/09/rFw3h7ES8Ze2VAb.png)接收方滑动窗口示意图



需要指出的是，滑动窗口的大小可以依据一定的策略动态调整。应用会根据自身处理能力的变化，通过对本端TCP接收窗口大小的控制来实现对端的发送窗口进行流量限制。



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
