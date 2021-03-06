# OSI 模型和 TCP/IP 模型


## 1. OSI 开放式系统互联参考模型

 **开放式系统互联模型**（英语：**O**pen **S**ystem **I**nterconnection Model，缩写：OSI；简称为**OSI模型**）是一种[概念模型](https://zh.wikipedia.org/wiki/概念模型)，由[国际标准化组织](https://zh.wikipedia.org/wiki/国际标准化组织)提出，一个试图使各种计算机在世界范围内互连为网络的标准框架。定义于ISO/IEC 7498-1。 

 ![OSI 开放式系统互联参考模型](https://i.loli.net/2019/12/09/TLr5NUxWHKmJszP.gif)



### 物理层（Physical Layer）

![](https://i.loli.net/2019/12/09/N2o1gwMl95GfFsD.png)

在局部局域网上传送[数据帧](https://zh.wikipedia.org/wiki/数据帧)（data frame），它负责管理电脑通信设备和网络媒体之间的互通， 定义物理设备标准，如网线的接口类型、光纤的接口类型、各种传输介质的传输速率 ，主要作用是传输比特流（就是由1、0转化为电流强弱来进行传输,到达目的地后在转化为1、0，也就是我们常说的数模转换与模数转换），这一层的数据叫做比特，单位是bit比特。包括了针脚、电压、线缆规范、集线器、中继器、网卡、主机接口卡、 调制解调器等。典型的协议有 RS 232C、RS 449/422/423、V.24 和 X.21、X.21bis

### 数据链路层**（Datalink Layer）** 

![](https://i.loli.net/2019/12/09/oSpjkPZy9ftYGVa.png)

定义了如何让格式化数据以进行传输，以及如何让控制对物理介质的访问，这一层通常还提供错误检测和纠正，以确保数据的可靠传输。交换机(二层)、网桥设备在这一层。数据链路层协议的代表包括： `HDLC` 、`PPP`、`STP`等。 

### 网络层 **（Network Layer）** 

![](https://i.loli.net/2019/12/09/wexCdOuZ2QH54B3.png)

在位于不同地理位置的网络中的两个主机系统之间提供连接和路径选择，Internet的发展使得从世界各站点访问信息的用户数大大增加，而网络层正是管理这种连接的层。网络层负责在源机器和目标机器之间建立它们所使用的路由。路由器在该层。协议有：`IP`、`ICMP`（互联网控制报文协议）、`ARP`（地址转换协议）、`RARP`（反向地址转换协议） 

###  **传输层（Transport Layer）** 

![](https://i.loli.net/2019/12/09/YQgGIThdiAEn23f.png)

OSI 模型中最重要的一层。定义了一些传输数据的协议和端口号（WWW端口80等），如：`TCP`（传输控制协议，传输效率低，可靠性强，用于传输可靠性要求高，数据量大的数据），`UDP`（用户数据报协议，与TCP特性恰恰相反，用于传输可靠性要求不高，数据量小的数据，如QQ聊天数据就是通过这种方式传输的）， 主要是将从下层接收的数据进行分段和传输，到达目的地址后再进行重组，常常把这一层数据叫做段。 

###  **会话层（Session Layer）** 

![](https://i.loli.net/2019/12/09/4hayBOXImwVbAct.png)

负责在网络中的两节点之间建立、维持和终止通信。 会话层的功能包括：建立通信链接，保持会话过程通信链接的畅通，同步两个节点之间的对话，决定通信是否被中断以及通信中断时决定从何处重新发送。

通过传输层（端口号：传输端口与接收端口）建立数据传输的通路，主要在你的系统之间发起会话或者接受会话请求（设备之间需要互相认识可以是IP也可以是MAC或者是主机名）。

常见的协议有 `ADSP`、`RPC` 等。

###  **表示层（Presentation Layer）** 

![](https://i.loli.net/2019/12/09/PRd9vurV1UCfqTw.png)

应用程序和网络之间的翻译官，在表示层，数据将按照网络能理解的方案进行格式化；这种格式化也因所使用网络的类型不同而不同。表示层管理数据的解密与加密，如系统口令的处理。可确保一个系统的应用层所发送的信息可以被另一个系统的应用层读取。例如，PC程序与另一台计算机进行通信，其中一台计算机使用扩展二一十进制交换码（EBCDIC），而另一台则使用美国信息交换标准码（ASCII）来表示相同的字符。如有必要，表示层会通过使用一种通格式来实现多种数据格式之间的转换。 常见的协议有`ASCII`、`SSL/TLS` 等。

###  **应用层（Application Layer）** 



![](https://i.loli.net/2019/12/09/qTcn6Y3klSOetdN.png)

是最靠近用户的OSI层，这一层为用户的应用程序（例如电子邮件、文件传输和终端仿真）提供网络服务 。常见的协议有 `HTTP`，`FTP`，`TELNET`、`SMTP` 等。

### OSI模型 数据传输流程概览

<img src="https://i.loli.net/2019/12/09/peH8BVr75dWE34c.png" alt="" style="zoom:67%;" />



## 2. TCP/IP四层网络模型

### 提出

OSI模型是一个定义良好的协议规范集，定义了开放系统的层次结构和之间的关系，但并没有提供一个可实现的方法，其并非一个标准，而是一个慨念型框架。TCP/IP是被广泛使用的OSI模型的实现。

### 简介

TCP/IP协议栈是美国国防部高级研究计划局计算机网（Advanced Research Projects Agency Network，ARPANET）和其后继因特网使用的参考模型。ARPANET是由美国国防部（U.S．Department of Defense，DoD）赞助的研究网络。最初，它只连接了美国境内的四所大学。随后的几年中，它通过租用的电话线连接了数百所大学和政府部门。最终ARPANET发展成为全球规模最大的互连网络-因特网。最初的ARPANET于1990年永久性地关闭。

TCP/IP是一组协议的代名词，它**还包括许多协议，组成了TCP/IP协议簇**。TCP/IP协议簇分为四层，IP位于协议簇的第二层(对应OSI的第三层)，TCP位于协议簇的第三层(对应OSI的第四层)。

### 层次

TCP/IP协议采用了4层的层级结构，每一层都呼叫它的下一层所提供的网络来完成自己的需求。分别为：

**应用层**：应用程序间沟通的层，如简单电子邮件传输（SMTP）、文件传输协议（FTP）、网络远程访问协议（Telnet）等。

**传输层**：在此层中，它提供了节点间的数据传送服务，如传输控制协议（TCP）、用户数据报协议（UDP）等，TCP和UDP给数据包加入传输数据并把它传输到下一层中，这一层负责传送数据，并且确定数据已被送达并接收。

**网络层**：负责提供基本的数据封包传送功能，让每一块数据包都能够到达目的主机（但不检查是否被正确接收），如网际协议（IP）。

**网络接口层**：对实际的网络媒体的管理，定义如何使用实际网络（如Ethernet、Serial Line等）来传送数据。

### 对应关系及协议整理

![](https://i.loli.net/2019/12/09/Ip71j6NFub3Ugr5.png)

### TCP/IP模型 数据传输流程概览

<img src="https://i.loli.net/2019/12/09/NWbI9otOVEwkJaL.png" alt="" style="zoom:50%;" />

## 3. 问题

### 交换机工作在OSI的哪一层？

**二层交换机** 工作在OSI的第二层数据链路层，，由于它们要对帧解码并使用帧信息将数据发送到正确的接收方，所以它们是工作在数据链路层的。
**三层交换机** 是工作在OSI的网络层，因为三层交换机有路由功能。

### 路由器工作在OSI的哪一层？

路由器工作在OSI七层模型的第3层，网络层。由于网络层处理，并智能指导数据传送，路由器连接网络各段，所以路由器属于网络层。在网络中，“路由”是基于编址方案、使用模式以及可达性来指引数据的发送。 网络层负责在源机器和目标机器之间建立它们所使用的路由。

### PING命令使用OSI哪一层协议？

ping命令使用的是ICMP协议，位于OSI七层网络模型中的第三层，网络层。



<center> ·End· </center>
