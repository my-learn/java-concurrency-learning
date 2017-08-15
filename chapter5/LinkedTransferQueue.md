`java.util.concurrent.TransferQueue`是jdk1.7新增的接口，继承自BlockingQueue，所以它具备阻塞队列的所有特性，除此还提供5个新的API功能。

TransferQueue中定义了5个方法
* transfer(E e)：若当前存在一个正在等待获取的消费者线程（使用take()或者poll()函数），即立刻移交之；否则，会插入当前元素e到队列尾部，并且等待进入阻塞状态，直到到有消费者线程取走该元素。
* tryTransfer(E e)：若当前存在一个正在等待获取的消费者线程，使用该方法会即刻转移/传输对象元素e；若不存在，则返回false，并且不进入队列。这是一个不阻塞的操作。
* tryTransfer(E e, long timeout, TimeUnit unit)：若当前存在一个正在等待获取的消费者线程，会立即传输给它;否则将插入元素e到队列尾部，并且等待被消费者线程获取消费掉；若在指定的时间内元素e无法被消费者线程获取，则返回false，同时该元素被移除。
* hasWaitingConsumer()：判断是否存在消费者线程。
* getWaitingConsumerCount()：获取所有等待获取元素的消费线程数量。




# LinkedTransferQueue
LinkedTransferQueue是TransferQueue唯一的实现类。它是基于链表的无界TransferQueue，功能：A producer will wait for the consumer to consume the elements in this queue.
它是一个非常有用的类，很多开源项目都用了这个类，比如Netty，另外在xmemcached中yan4j框架中也有同名类`com.google.code.yanf4j.util.LinkedTransferQueue`，它备注了是从Netty中拷贝过来的，作者还是大神Doug Lea。

* put():生产者将元素放入queue中，该方法不会抛出异常，因为LinkedTransferQueue没有size大小限制，即无界的。
* take():从queue中获取元素，如果为空，则一直等待
* size()：因为队列的异步特性，检测当前队列的元素个数需要逐一迭代，可能会得到一个不太准确的结果，尤其是在遍历时有可能队列发生更改，同时API不能保证一定会立刻执行。**尽量避免使用**
* 批量操作：类似于addAll，removeAll, retainAll, containsAll, equals, toArray等方法，API不能保证一定会立刻执行。**尽量避免使用**

SynchronousQueue的底层也是用类似transfer思路实现的，它定义了一个静态内部类Transferer，并提供transfer方法，我们知道SynchronousQueue本身不存在容量,只能进行线程之间的元素传送，当执行SynchronousQueue的offer操作时，如果没有其他线程执行poll，则直接返回false，这里程之间元素传送正是通过transfer方法完成的。
看看poll()源码就知道了
```java
public E poll() {
    return (E)transferer.transfer(null, true, 0);
}
```

transfer算法比较复杂，大致的理解是采用所谓双重数据结构(dual data structures)。之所以叫双重，其原因是方法都是通过两个步骤完成：保留与完成。比如消费者线程从一个队列中取元素，发现队列为空，他就生成一个空元素放入队列,所谓空元素就是数据项字段为空。然后消费者线程在这个字段上旅转等待。这叫保留。直到一个生产者线程意欲向队例中放入一个元素，这里他发现最前面的元素的数据项字段为NULL，他就直接把自已数据填充到这个元素中，即完成了元素的传送。

# 示例
定义一个生产者
```java
import java.util.Random;
import java.util.UUID;
import java.util.concurrent.TransferQueue;

public class LinkedTransferQueueProducer implements Runnable {

	protected TransferQueue<String> transferQueue;
	final Random random = new Random();

	public LinkedTransferQueueProducer(TransferQueue<String> queue) {
		this.transferQueue = queue;
	}

	@Override
	public void run() {
		while (true) {
			try {
				String data = UUID.randomUUID().toString();
				System.out.println("Put: " + data);
				transferQueue.transfer(data);
				Thread.sleep(500);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}

```
定义一个消费者
```java
import java.util.concurrent.TransferQueue;

public class LinkedTransferQueueConsumer implements Runnable {

	protected TransferQueue<String> transferQueue;

	public LinkedTransferQueueConsumer(TransferQueue<String> queue) {
		this.transferQueue = queue;
	}

	@Override
	public void run() {
		while (true) {
			try {
				String data = transferQueue.take();
				System.out.println(Thread.currentThread().getName()
						+ " take(): " + data);
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}
```
测试类
```java
import java.util.concurrent.LinkedTransferQueue;
import java.util.concurrent.TransferQueue;

public class LinkedTransferQueueExample {
	public static void main(String[] args) {
		final TransferQueue<String> transferQueue = new LinkedTransferQueue<String>();
		
		LinkedTransferQueueProducer queueProducer = new LinkedTransferQueueProducer(transferQueue);
		new Thread(queueProducer).start();

		LinkedTransferQueueConsumer queueConsumer1 = new LinkedTransferQueueConsumer(transferQueue);
		new Thread(queueConsumer1).start();

		LinkedTransferQueueConsumer queueConsumer2 = new LinkedTransferQueueConsumer(transferQueue);
		new Thread(queueConsumer2).start();

	}
}

```
运行结果
```plain
Put: 819c357c-3990-4db5-adfc-011ba0d95195
Thread-1 take(): 819c357c-3990-4db5-adfc-011ba0d95195
Put: c37dd2d2-6130-43ad-9851-275546a2a410
Thread-2 take(): c37dd2d2-6130-43ad-9851-275546a2a410
Put: 483b45c5-25b3-48d3-a3a3-979c896ae905
Thread-1 take(): 483b45c5-25b3-48d3-a3a3-979c896ae905
Put: e472a655-53ed-4931-a0b6-653d4a2bd060
Thread-2 take(): e472a655-53ed-4931-a0b6-653d4a2bd060
Put: 9cf2575c-96e1-4f18-bb3c-9f287a9b2ffa
Thread-1 take(): 9cf2575c-96e1-4f18-bb3c-9f287a9b2ffa
```
示例代码引自<http://javapapers.com/java/java-linkedtransferqueue/>



# TransferQueue vs SynchronousQueue
我们看到`LinkedTransferQueue`和`SynchronousQueue`都能达到相同的功能，那么他们有什么区别呢？

该文章([post by Alex Miller](http://puredanger.github.io/tech.puredanger.com/2009/02/28/java-7-transferqueue/))提及到
> ** TransferQueue ** is more generic and useful than SynchronousQueue however as it allows you to flexibly decide whether to use normal BlockingQueue semantics or a guaranteed hand-off. In the case where items are already in the queue, calling transfer will guarantee that all existing queue items will be processed before the transferred item.

> ** SynchronousQueue ** implementation uses dual queues (for waiting producers and waiting consumers) and protects both queues with a single lock. The LinkedTransferQueue implementation uses CAS operations to form a nonblocking implementation and that is at the heart of avoiding serialization bottlenecks.

TransferQueue相比SynchronousQueue更好用，因为你可以决定是使用BlockingQueue的方法（译者注：例如put方法）还是确保一次传递完成（译者注：即transfer方法）。

而且[Doug Lea](http://cs.oswego.edu/pipermail/concurrency-interest/2009-February/005888.html) 也解释了开发TransferQueue的动机：从功能角度来讲，LinkedTransferQueue实际上是ConcurrentLinkedQueue、SynchronousQueue（公平模式）和LinkedBlockingQueue的超集。而且LinkedTransferQueue更好用，因为它不仅仅综合了这几个类的功能，同时也提供了更高效的实现。

参考
http://ifeve.com/java-transfer-queue/
http://blog.csdn.net/yjian2008/article/details/16951811 
