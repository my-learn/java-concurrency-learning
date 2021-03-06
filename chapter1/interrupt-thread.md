Java的中断机制很容易被人理解为是中断线程的，使线程停止，其实理解不完全对。每一个线程都有一个中断状态，调用线程对象的interrupt方法并不一定中断了正在运行的线程，而仅仅发出一个中断信号，通知线程在合适的时机中断自己，也即将线程状态改为“中断状态”，改变中断状态后用户可以做后续处理工作，比如结束线程、传递异常、忽略中断再继续任务等等


# Thread提供的中断相关的方法
* void interrupt():用于中断线程。调用该方法的线程的状态为将被置为"中断"状态，但不会停止线程，一旦线程被置为“中断状态”，就会抛出interruptedException中断异常。需要用户自己做中断处理，也就是处理interruptedException这个异常
* boolean isInterrupted()、isInterrupted(boolean)：检查当前线程是否被中断
* static boolean interrupted()：这是一个静态方法，该方法会将中断标示位清除，所以不要用该方法判断线程是否已经中断，而是用isInterrupted方法

interrupt跟interrupted区别在于interrupted用来判断当前线是否被中断，而isInterrupted可以用来判断其他线程是否被中断，所以
`while(！isInterrupted())`也可以换成`while(！Thread.interrupted())`

isInterrupted()和interrupted()方法有一个很大的区别。isInterrupted()不能改变interrupted属性的值，但是后者能设置interrupted属性为false。因为interrupted()是一个静态方法，更推荐使用isInterrupted()方法。
当你主动调用了线程实例的interrupt()方法， 就可以用isInterrupted()捕获到。


# 会检查中断状态并响应的阻塞
如果一个线程处于了阻塞状态,同时能够响应中断，则在线程检测到中断标识为true时，会在这些阻塞方法调用处抛出InterruptedException异常，并且在抛出异常后立即将线程的中断标别位清除，即重新设置为false。抛出异常是为了线程从阻塞状态醒过来，并在结束线程前让程序员有足够的时间来处理中断请求。
当前在实际业务代码中，你也可以忽略掉中断异常。

能够响应中断的阻塞线程，比如Thread.sleep()、thread.join()、Object.wait()、可中断的I/O通道、condition.await()、BlockingQueue.put()/take()等

synchronized在获锁的过程中是不能被中断的，`reentrantLock.lock()`也不能，但是`reentrantLock.tryLock(long timeout, TimeUnit unit)`方法会在指定时间内获取不到所抛出InterruptedException异常

# 不可响应中断的阻塞
并非所有的可阻塞方法或者阻塞机制都能相应中断，比如前面提到的synchronized获取锁，还tcp通信socket数据传输，I/O操作等，interrupt中断对他们不起任务作用，我们就有必要使用某种机制来中断由于这些代码导致的线程阻塞
首先`synchronized`获取导致的阻塞无解，除非改成使用`reentrantLock.tryLock(long timeout, TimeUnit unit)`就可中断，
其次，java提供了socket、I/O上的close方法，该方法会抛出异常就可解除阻塞。

另外，jdk 1.4提供InterruptibleChannel接口、实现该接口的通道是可中断的。
如果一个线程在调用实现了InterruptibleChannel接口的代码上阻塞，一个线程调用了该阻塞线程的 interrupt 方法，将会导致该通道被关闭，同时已阻塞线程接将会收到ClosedByInterruptException，并且设置已阻塞线程的中断状态。


# 示例
前面说了很多关于中断的东西，还是来看看代码吧

Thread的sleep可以响应中断
```java
public class InterruptTest {
	public static void main(String[] args) throws Exception {
		MyThread myThread = new MyThread("myThread");
		myThread.start();

		TimeUnit.SECONDS.sleep(2);

		myThread.interrupt();
		System.out.printf("Thread:%s interrupted status is %s", myThread.getName(), myThread.isInterrupted());
	}

	static class MyThread extends Thread {
		public MyThread(String name) {
			super(name);
		}

		@Override
		public void run() {
			// 中断被阻塞状态（sleep、wait、join 等状态）的线程，会抛出异常 InterruptedException
			// 抛出异常 InterruptedException 前，JVM 会先将中断状态重置为默认状态 false
			try {
				TimeUnit.SECONDS.sleep(10);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```
控制打印false，同时打印错误堆栈


不能中断处于非阻塞状态的线程
将前面示例的run()方法改成
```java
@Override
public void run() {
	// 线程一直在运行状态，没有停止或者阻塞等
	// 调用了 interrupt() 方法，中断状态置为 true，但不会影响线程的继续运行
	while (true) {

	}
}
```
控制打印true，同时程序不会终止。


将上面示例的run()方法改成
```java
@Override
public void run() {
    int i = 0;
    while(i<Integer.MAX_VALUE){
        System.out.println(i+" while循环");
        i++;
    }
}
```
效果更明显了，主线程调用了interrupt，控制台还是不停的打印。
while循环会一直运行直到变量i的值超出Integer.MAX_VALUE。所以说直接调用interrupt方法不能中断正在运行中的线程。

但是如果配合isInterrupted()能够中断正在运行的线程，因为调用interrupt方法相当于将中断标志位置为true，那么可以通过调用isInterrupted()判断中断标志是否被置位来中断线程的执行。上面的run()改成：
```java
public class Test {
	public static void main(String[] args) throws IOException {
		Test test = new Test();
		MyThread thread = test.new MyThread();
		thread.start();
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
		}
		thread.interrupt();
	}

	class MyThread extends Thread {
		@Override
		public void run() {
			int i = 0;
			while (!isInterrupted() && i < Integer.MAX_VALUE) {
				System.out.println(i + " while循环");
				i++;
			}
		}
	}
}
```
运行会发现，打印若干个值之后，while循环就停止打印了。

但是一般情况下不建议通过这种方式来中断线程，而是用状态标志，这个前面一节已经介绍了。还是上面的代码，改成：
```java
public class Test {
	public static void main(String[] args) throws IOException {
		Test test = new Test();
		MyThread thread = test.new MyThread();
		thread.start();
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
		}
		thread.setStop(true);
	}

	class MyThread extends Thread {
		private volatile boolean isStop = false;
		
		@Override
		public void run() {
			int i = 0;
			while (!isStop && i < Integer.MAX_VALUE) {
				System.out.println(i + " while循环");
				i++;
			}
		}

		public void setStop(boolean isStop) {
			this.isStop = isStop;
		}
		
	}
}
```
同样的也能实现中断线程

最佳方案就是!isStop和!isInterrupted()联合使用，因为类似调用sleep()方法处于阻塞状态下的线程无法通过!isStop终止。

