# Socket 入门


<!--more-->

## 进程间通信

网络中进程进行通信的前提是每个进程都要有唯一的标识，在本地计算机中我们可以用PID来唯一标识一个进程，可以多个计算机处在网络中时，不同计算机的进程PID冲突的可能性就很大。怎么办呢？我们知道IP地址可以唯一标识主机，而TCP协议和端口号可以唯一标识主机的一个进程，这样我们就可以用 IP+协议+端口号 来唯一标识网络中的一个进程了。

使用TCP/IP协议的应用程序通常采用应用编程接口：UNIX BSD的套接字（socket）和UNIX System V的TLI（已经被淘汰），来实现网络进程之间的通信。就目前而言，几乎所有的应用程序都是采用socket。

## 什么是Socket？

网络套接字（英语：Network socket；又译网络套接字、网络接口、网络插槽）在计算机科学中是电脑网络中行程间数据流的端点。使用以网际协议（Internet Protocol）为通信基础的网络套接字，称为网际套接字（Internet socket）。因为网际协议的流行，现代绝大多数的网络套接字，都是属于网际套接字。

socket起源于Unix，而Unix/Linux基本哲学之一就是“一切皆文件”，都可以用“打开open –> 读写write/read –> 关闭close”模式来操作。我的理解就是Socket就是该模式的一个实现，socket即是一种特殊的文件，一些socket函数就是对其进行的操作（读/写IO、打开、关闭）。



![img](https://i.loli.net/2019/12/09/tl96CpP3Qv4fMEV.jpg)



Socket与TCP/IP协议并没有必然的联系，Socket编程接口在设计的时候就希望能适应其它的网络协议，所以Socket的出现是为了我们更好地使用TCP/IP协议栈，是对TCP/IP协议的抽象，是操作系统对外开放的接口。

## socket通信流程

以使用TCP协议通讯的socket为例，其交互流程大概是这样子的



![Socket通信流程](https://i.loli.net/2019/12/09/aJRALE8cFmZonkg.png)Socket通信流程



## 实列

### **编写一个网络应用程序，包含客户端和服务端，客户端向服务端发送一个字符串，服务端收到字符串并打印到命令行同时向客户端返回字符串长度，最后客户端输出收到的字符串长度，分别用TCP和UDP实现。**

#### TCP方式：

服务端

```java
public class TCPServer {
    public static void main(String[] args) throws Exception {
        //创建socket,并将socket绑定到65000端口
        ServerSocket ss = new ServerSocket(65000);
        //死循环，使得socket一直等待并处理客户端发送过来的请求
        while (true) {
            //监听65000端口，直到客户端返回连接信息后才返回
            Socket socket = ss.accept();
            //获取客户端的请求信息后，执行相关业务逻辑
            new LengthCalculator(socket).start();
        }
    }
}
```

服务端业务逻辑

```java
public class LengthCalculator extends Thread {
    //以socket为成员变量
    private Socket socket;
    public LengthCalculator(Socket socket) {
        this.socket = socket;
    }
    @Override
    public void run() {
        try {
            //获取socket的输出流
            OutputStream os = socket.getOutputStream();
            //获取socket的输入流
            InputStream is = socket.getInputStream();
            int ch = 0;
            byte[] buff = new byte[1024];
            //buff主要用来读取输入的内容，存成byte数组，ch主要用来获取读取数组的长度
            ch = is.read(buff);
            //将接收流的byte数组转换成字符串，这里获取的内容是客户端发送过来的字符串参数
            String content = new String(buff, 0, ch);
            System.out.println(content);
            //往输出流里写入获得的字符串的长度，回发给客户端
            os.write(String.valueOf(content.length()).getBytes());
            //不要忘记关闭输入输出流以及socket
            is.close();
            os.close();
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

客户端

```java
public class TCPClient {
    public static void main(String[] args) throws Exception {
        //创建socket，并指定连接的是本机的端口号为65000的服务器socket
        Socket socket = new Socket("127.0.0.1", 65000);
        //获取输出流
        OutputStream os = socket.getOutputStream();
        //获取输入流
        InputStream is = socket.getInputStream();
        //将要传递给server的字符串参数转换成byte数组，并将数组写入到输出流中
        os.write(new String("hello world").getBytes());
        int ch = 0;
        byte[] buff = new byte[1024];
        //buff主要用来读取输入的内容，存成byte数组，ch主要用来获取读取数组的长度
        ch = is.read(buff);
        //将接收流的byte数组转换成字符串，这里是从服务端回发回来的字符串参数的长度
        String content = new String(buff, 0, ch);
        System.out.println(content);
        //不要忘记关闭输入输出流以及socket
        is.close();
        os.close();
        socket.close();
    }
}
```

运行



![img](https://i.loli.net/2019/12/09/Rg5GzaCweFlMtjL.png)

![img](https://i.loli.net/2019/12/09/iITvbCjR18d2HMY.png)



**UDP方式：**

服务端

```java
public class UDPServer {
    public static void main(String[] args) throws Exception {
        // 服务端接受客户端发送的数据报
        DatagramSocket socket = new DatagramSocket(65001); //监听的端口号
        byte[] buff = new byte[100]; //存储从客户端接受到的内容
        DatagramPacket packet = new DatagramPacket(buff, buff.length);
        //接受客户端发送过来的内容，并将内容封装进DatagramPacket对象中
        socket.receive(packet);
        byte[] data = packet.getData(); //从DatagramPacket对象中获取到真正存储的数据
        //将数据从二进制转换成字符串形式
        String content = new String(data, 0, packet.getLength());
        System.out.println(content);
        //将要发送给客户端的数据转换成二进制
        byte[] sendedContent = String.valueOf(content.length()).getBytes();
        // 服务端给客户端发送数据报
        //从DatagramPacket对象中获取到数据的来源地址与端口号
        DatagramPacket packetToClient = new DatagramPacket(sendedContent,
                sendedContent.length, packet.getAddress(), packet.getPort());
        socket.send(packetToClient); //发送数据给客户端
    }
```

客户端

```java
public class UDPClient {
    public static void main(String[] args) throws Exception {
        // 客户端发数据报给服务端
        DatagramSocket socket = new DatagramSocket();
        // 要发送给服务端的数据
        byte[] buf = "Hello World".getBytes();
        // 将IP地址封装成InetAddress对象
        InetAddress address = InetAddress.getByName("127.0.0.1");
        // 将要发送给服务端的数据封装成DatagramPacket对象 需要填写上ip地址与端口号
        DatagramPacket packet = new DatagramPacket(buf, buf.length, address,
                65001);
        // 发送数据给服务端
        socket.send(packet);
        // 客户端接受服务端发送过来的数据报
        byte[] data = new byte[100];
        // 创建DatagramPacket对象用来存储服务端发送过来的数据
        DatagramPacket receivedPacket = new DatagramPacket(data, data.length);
        // 将接受到的数据存储到DatagramPacket对象中
        socket.receive(receivedPacket);
        // 将服务器端发送过来的数据取出来并打印到控制台
        String content = new String(receivedPacket.getData(), 0,
                receivedPacket.getLength());
        System.out.println(content);
    }
}
```

运行



![img](https://i.loli.net/2019/12/09/RuBrkvAVp5aFTx3.png)



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
