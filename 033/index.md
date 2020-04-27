# 阻塞队列 BlockingQueue


<!--more-->

## BlockingQueue

![BlockingQueue](https://i.loli.net/2020/02/01/YD12so6nTyWihrx.png)

阻塞队列是Java自带的核心API，具体来说就是一个BlockingQueue接口，主要用来解决线程通信的问题。当然我们有很多解决线程通信问题的方法，只不过BlockingQueue更简单易用些，其主要有两个方法put、take来解决问题，用于存队列中存和取一个数据。比如上图，线程1与线程2通信，线程1调用put方法将数据放入阻塞队列中，然后线程2调用take方法从阻塞队列中取值，这样处理问题的方式满足**生产者消费者模式**，线程1是生产者，线程2是消费者，BlockingQueue其实就是在生产者和消费者之间构建了一个缓冲区，避免了两个线程之间直接打交道，带来的问题（生产的一直生产，等待的一直等待，造成CPU资源的浪费）。

</br>

## BlockingQueue的实现类

BlockingQueue是一个接口，不能直接使用，JDK中带了很多它的实现类，它们各有特点和优势。

- ​	ArrayBlockingQueue，数组实现
- ​	LinkedBlockingQueue，链表实现
- ​	PriorityBlockingQueue、SynchronousQueue、DelayQueue等。

更多查阅 Java API ：https://docs.oracle.com/javase/8/docs/api/

</br>

### ArrayBlockingQueue阻塞队列测试

```java
import java.util.Random;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class BlockingQueueTests {

    public static void main(String[] args) {
        // 使用ArrayBlockingQueue进行测试，长度：10
        BlockingQueue queue = new ArrayBlockingQueue(10);

        new Thread(new Producer(queue)).start();

        new Thread(new Consumer(queue)).start();
        new Thread(new Consumer(queue)).start();
        new Thread(new Consumer(queue)).start();
    }

}

// 生产者线程
class Producer implements Runnable {

    private BlockingQueue<Integer> queue;

    public Producer(BlockingQueue<Integer> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 100; i++) {
                // 设置间隔
                Thread.sleep(20);
                queue.put(i);
                System.out.println(Thread.currentThread().getName() + " 生产: " + queue.size());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

// 消费者线程
class Consumer implements Runnable {

    private BlockingQueue<Integer> queue;

    public Consumer(BlockingQueue<Integer> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            while (true) {
                // 设置间隔
                Thread.sleep(new Random().nextInt(1000));
                queue.take();
                System.out.println(Thread.currentThread().getName() + " 消费: " + queue.size());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**运行结果**

```properties
Thread-0 生产: 1
Thread-0 生产: 2
Thread-1 消费: 1
Thread-0 生产: 2
Thread-0 生产: 3
Thread-0 生产: 4
Thread-0 生产: 5
Thread-0 生产: 6
Thread-0 生产: 7
Thread-0 生产: 8
Thread-0 生产: 9
Thread-0 生产: 10
Thread-1 消费: 9
Thread-0 生产: 10
Thread-3 消费: 9
Thread-0 生产: 10
...
...
...
Thread-3 消费: 9
Thread-0 生产: 10
Thread-1 消费: 9
Thread-0 生产: 10
Thread-3 消费: 9
Thread-3 消费: 8
Thread-2 消费: 7
Thread-1 消费: 6
Thread-2 消费: 5
Thread-2 消费: 4
Thread-1 消费: 3
Thread-2 消费: 2
Thread-2 消费: 1
Thread-2 消费: 0
```

可以看到，生产者线程一直在生产数据，但是达到10个时停止生产，3个消费者线程，错杂消费着数据，直到最后生产者停止生产，消费者最终将数据消费完，此时程序并没有结束，消费者线程还在等待着数据。ArrayBlockingQueue实现了这个过程中数据的缓冲。



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
