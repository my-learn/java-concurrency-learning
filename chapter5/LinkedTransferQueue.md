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
* size()：因为队列的异步特性，检测当前队列的元素个数需要逐一迭代，可能会得到一个不太准确的结果，尤其是在遍历时有可能队列发生更改。
* 批量操作：类似于addAll，removeAll, retainAll, containsAll, equals, toArray等方法，API不能保证一定会立刻执行。因此，我们在使用过程中，不能有所期待，这是一个具有异步特性的队列。

示例
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.LinkedTransferQueue;

public class LinkedTransferQueueDemo {
	static LinkedTransferQueue<String> lnkTransQueue = new LinkedTransferQueue<String>();

	public static void main(String[] args) {
		ExecutorService exService = Executors.newFixedThreadPool(2);
		Producer producer = new LinkedTransferQueueDemo().new Producer();
		Consumer consumer = new LinkedTransferQueueDemo().new Consumer();
		exService.execute(producer);
		exService.execute(consumer);
		exService.shutdown();
	}

	class Producer implements Runnable {
		@Override
		public void run() {
			for (int i = 0; i < 3; i++) {
				try {
					System.out.println("Producer is waiting to transfer...");
					lnkTransQueue.transfer("A" + i);
					System.out.println("producer transfered element: A" + i);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}

	class Consumer implements Runnable {
		@Override
		public void run() {
			for (int i = 0; i < 3; i++) {
				try {
					System.out.println("Consumer is waiting to take element...");
					String s = lnkTransQueue.take();
					System.out.println("Consumer received Element: " + s);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
}
```
在该示例中，有两个线程，消费者和生产者。生产者将元素传递给消费者，如果没有消费者，将等待元素被消费。打印结果如下
```plain
Consumer is waiting to take element...
Producer is waiting to transfer...
producer transfered element: A0
Consumer received Element: A0
Consumer is waiting to take element...
Producer is waiting to transfer...
producer transfered element: A1
Producer is waiting to transfer...
Consumer received Element: A1
Consumer is waiting to take element...
Consumer received Element: A2
producer transfered element: A2
```






