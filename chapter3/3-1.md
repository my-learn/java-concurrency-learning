它是一种计数器，用来保护一个或者多个共享资源的访问。
如果线程要访问一个共享资源，它必须先获得信号量，调用acquire，信号量计数器减1；调用release，信号量计数器加1。计数器大于0，才允许访问共享资源，否则acquire一直阻塞，直到信号量计数器大于0

# 使用步骤
通过acquire()方法获得信号量
使用共享资源执行必要的操作
通过release()方法释放信号量

# Semaphore类方法
+ acquire()：获得信号量，只有获得了信号量，才能操作共享资源。
+ acquireUninterruptibly()：获得信号量，跟acquire的区别在于当线程被阻塞时，可能会被中断，acquire会抛出InterruptedException异常，而acquireUninterruptibly方法会忽略线程中断不会抛出异常。
+ tryAcquire()：试图获得信号量。如果能获得就返回true，否则返回false。我们可以根据返回值来做出恰当的处理。
+ release()：释信号量
+ availablePermits():返回Semaphore当前可用的信号量，该方法只能用于调试测试，不能用于逻辑判断，比如尝试以下的代码：

``` java
// 有打印机可用
if (printer.availablePermits() > 0) {
	System.out.println("打印任务" + this.id + "准备打印，有空闲打印机");
} else {
	System.out.println("打印任务" + this.id + "准备打印，没有空闲打印机，排队等待~");
}
// 获取信号量
printer.acquire();
...
```
printer.availablePermits()和printer.acquire()并不是原子操作，可能当调用printer.availablePermits()时还有可用信号量，但是当调用printer.acquire()时，已经无信号量可用。


# 信号量的公平性
构造函数可以传入一个参数，false-非公平模式，true-公平模式。默认是false。
公平模式选择等待共享资源时间最长的那个线程

# 示例
下面打印机同时有多个打印任务，但是只有2台打印机产生资源竞争的场景
打印任务
```java
package com.gotoback.current;

import java.util.concurrent.Semaphore;

public class PrintJob extends Thread {
	private Semaphore printer;
	private int id;

	public PrintJob(int i, Semaphore printer) {
		this.id = i;
		this.printer = printer;
	}

	public void run() {
		try {
			// 占有打印机
			printer.acquire();
			int time = (int) (Math.random() * 10 * 1000);
			System.out.println("打印任务" + this.id + "开始打印... 预计耗时"+time/1000+"秒");
			Thread.sleep(time);
			System.out.println("打印任务" + this.id + "打印完成");
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			// 归还打印机
			printer.release();
		}
	}

	
}
```
测试类
```java
package com.gotoback.current;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaPhoreTest {
	public static void main(String args[]) {
		ExecutorService list = Executors.newCachedThreadPool();
		Semaphore printer = new Semaphore(2);// 只有两台打印机
		// 有十个打印任务
		for (int i = 0; i < 10; i++) {
			list.submit(new PrintJob(i + 1, printer));
		}
		list.shutdown();
		
		//小技巧，等待所有打印任务完成
		printer.acquireUninterruptibly(2);
		System.out.println("全部打印任务执行完毕");
		printer.release(2);
	}
}

```
结果如下：
```plain
打印任务2开始打印... 预计耗时8秒
打印任务1开始打印... 预计耗时7秒
打印任务1打印完成
打印任务3开始打印... 预计耗时2秒
打印任务2打印完成
打印任务4开始打印... 预计耗时9秒
打印任务3打印完成
打印任务5开始打印... 预计耗时7秒
打印任务4打印完成
打印任务6开始打印... 预计耗时0秒
打印任务5打印完成
打印任务7开始打印... 预计耗时0秒
打印任务6打印完成
打印任务8开始打印... 预计耗时4秒
打印任务7打印完成
打印任务9开始打印... 预计耗时4秒
打印任务8打印完成
打印任务10开始打印... 预计耗时8秒
打印任务9打印完成
打印任务10打印完成
全部打印任务执行完毕

```

