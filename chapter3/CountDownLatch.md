类似倒计时的功能，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
它初始化时需要传入一个整数，这个整数就是线程要等待完成的操作的数目。

CountDownLatch类的方法
+ CountDownLatch(int count) ：构造函数，需要传入一个初始值，即定义必须等待的先行完成的操作的数目
+ await()：当调用await时候，调用线程处于等待挂起状态，直至计数器变成0再继续
+ await(long timeout, TimeUnit unit)：可以指定等待时间，否则线程被中断
+ countDown()：每个被等待的事件在完成的时候调用，计数器减一

在当前计数到达0之前，调用await 方法会一直受阻塞，当调用countDown()方法后，内部计数器将减1，当计数器到达0的时候，CountDownLatch对象将唤起所有在awati()方法上等待的线程。

**注意：
1. 当计数器到达0后，再次执行await将不会有效，因为CountDownLatch的barrier是是不可重用的 
2. 计数无法被重置，只能重新new一个。如果需要重置计数，可以使用 CyclicBarrier
**
大体原理图（来自网络）
![](/chapter3/CountDownLatch-1.png)


在使用CountDownLatch时，注意避免死锁。最好在finally块中调用countDown
一个被await阻塞的线程在以下三种情况下恢复：
+ 调用 countDown() 方法，计数到达零
+ 其他某个线程中断当前线程
+ 超出await指定的等待时间

CountDownLatch有很多使用场景，比如控制线程顺序执行、程序启动请求处理类要等待所有资源加载完成、将一个任务分解成多个子任务完成等。

下面看几个实例
1.基本示例
该示例可以知道CountDownLatch的使用效果：可以让主线程等待所有子线程完成再继续执行
```java
import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CountDownLatchTest {

	public static void main(String[] args) throws InterruptedException {
		// barrier个数
		int count = 5;
		final CountDownLatch latch = new CountDownLatch(count);
		ExecutorService executor = Executors.newFixedThreadPool(count);

		for (int i = 0; i < count; i++) {
			executor.execute(new Runnable() {
				@Override
				public void run() {
					System.out.println(Thread.currentThread().getName() + "任务开始工作");
					int workTime = new Random().nextInt(10 - 3 + 1) + 3;// 随机3-10秒，方便看效果
					try {
						Thread.sleep(workTime * 1000);

						System.out.println(Thread.currentThread().getName() + "完成工作，耗时" + workTime + "秒!");
					} catch (InterruptedException e) {
						e.printStackTrace();
					} finally {
						latch.countDown();
					}
				}
			});
		}
		
		// 主线程阻塞，直到所有的barrier到达
		latch.await();
		executor.shutdown();

		System.out.println("所有线程执行完成！");
	}
}
```
执行结果
```plain
pool-1-thread-2任务开始工作
pool-1-thread-4任务开始工作
pool-1-thread-5任务开始工作
pool-1-thread-1任务开始工作
pool-1-thread-3任务开始工作
pool-1-thread-1完成工作，耗时6秒!
pool-1-thread-2完成工作，耗时6秒!
pool-1-thread-5完成工作，耗时7秒!
pool-1-thread-3完成工作，耗时10秒!
pool-1-thread-4完成工作，耗时10秒!
所有线程执行完成！
```

2.一个工作分配给两个工人协作完成
再演示几个比较有实际业务意义的示例
```java
package com.gotoback.current;

import java.util.Random;
import java.util.concurrent.CountDownLatch;

public class CountDownLatchTest {

	public static void main(String[] args) throws InterruptedException {
		CountDownLatch latch = new CountDownLatch(2);// 一个工作分配给两个工人协作完成
		Work work1 = new Work("张三", latch);
		Work work2 = new Work("李四", latch);
		work1.start(); // 工人一开始
		work2.start(); // 工人二开始工作
		latch.await();// 等待所有工人完成工作
		System.out.println("工作完成");
	}

	static class Work extends Thread {
		String workerName; // 员工
		int workTime; // 工作时间
		CountDownLatch latch;

		public Work(String workerName, CountDownLatch latch) {
			this.workerName = workerName;
			this.latch = latch;
		}

		public void run() {
			System.out.println("工人 " + workerName + " 开始工作... ");
			int workTime = new Random().nextInt(10);
			try {
				Thread.sleep(workTime * 1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("工人 " + workerName + " 完成工作，耗时" + workTime + "秒");
			latch.countDown();// 工人完成工作，计数器减一

		}

	}

}

```
控制台打印
```plain
工人 张三 开始工作... 
工人 李四 开始工作... 
工人 李四 完成工作，耗时2秒
工人 张三 完成工作，耗时3秒
工作完成

```

3.一个会议需要等待所有参会人员加入才能开始会议

```java
package com.gotoback.current;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CountDownLatchTest2 {
	public static void main(String[] args) throws InterruptedException {
		// 会议开始的锁，只是一个简单的开关，所以初始化计数器为1
		final CountDownLatch conference = new CountDownLatch(1);
		// 参会人员的锁，有10个人，所以初始化计数器为10
		final CountDownLatch participants = new CountDownLatch(10);
		// 十名参会人员
		final ExecutorService exec = Executors.newFixedThreadPool(10);
		for (int index = 0; index < 10; index++) {
			final int NO = index + 1;
			Runnable run = new Runnable() {
				public void run() {
					try {
						conference.await();// 一直阻塞，会议还没开始
						Thread.sleep((long) (Math.random() * 10 * 1000));
						System.out.println("No." + NO + " 进入视频会议");
					} catch (InterruptedException e) {
						e.printStackTrace();
					} finally {
						participants.countDown(); //通知有一个人员加入会议
					}
				}
			};
			exec.submit(run);
		}
		
		
		System.out.println("会议开始，等待参会人员加入...");
		conference.countDown();  //会议开始，参会人员可以加入了
		
		
		participants.await();//一直阻塞，直到全部参会人员加入
		System.out.println("人员到齐，会议正式开始");
		exec.shutdown();
	}
}

```
控制台打印
```plain
会议开始，等待参会人员加入...
No.10 进入视频会议
No.8 进入视频会议
No.4 进入视频会议
No.2 进入视频会议
No.6 进入视频会议
No.3 进入视频会议
No.1 进入视频会议
No.9 进入视频会议
No.7 进入视频会议
No.5 进入视频会议
人员到齐，会议正式开始

```

