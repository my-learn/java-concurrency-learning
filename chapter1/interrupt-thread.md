Java的中断机制很容易被人理解为是中断线程的，其实理解不完全对。每一个线程都有一个中断状态，调用线程对象的interrupt方法并不一定中断了正在运行的线程，而仅仅它通知线程在合适的时机中断自己，也即将线程状态改为“中断状态”，改变中断状态后再如何处理就需要用户自己去控制了，比如结束线程、传递异常、忽略中断再继续任务等等


# Thread提供的中断相关的方法
* void interrupt():用于中断线程。调用该方法的线程的状态为将被置为"中断"状态，但不会停止线程，一旦线程被置为“中断状态”，就会抛出interruptedException中断异常。需要用户自己做中断处理，也就是处理interruptedException这个异常
* boolean isInterrupted()、isInterrupted(boolean)：检查当前线程是否被中断
* static boolean interrupted：这是一个静态方法，该方法会将中断标示位清除，所以不要用该方法判断线程是否已经中断，而是用isInterrupted方法

interrupt跟interrupted区别在于interrupted用来判断当前线是否被中断，而isInterrupted可以用来判断其他线程是否被中断，所以
`while(！isInterrupted())`也可以换成`while(！Thread.interrupted())`

isInterrupted()和interrupted()方法有一个很大的区别。isInterrupted()不能改变interrupted属性的值，但是后者能设置interrupted属性为false。因为interrupted()是一个静态方法，更推荐使用isInterrupted()方法。
当你主动调用了线程实例的interrupt()方法， 就可以用isInterrupted()捕获到。

# 为什么stop方法被废弃
参考[Why Are Thread.stop, Thread.suspend, Thread.resume and Runtime.runFinalizersOnExit Deprecated?](http://docs.oracle.com/javase/1.5.0/docs/guide/misc/threadPrimitiveDeprecation.html)


# 响应中断
如果一个线程处于了阻塞状态,同时能够响应中断，则在线程检测到中断标识为true时，会在这些阻塞方法调用处抛出InterruptedException异常，并且在抛出异常后立即将线程的中断标别位清除，即重新设置为false。抛出异常是为了线程从阻塞状态醒过来，并在结束线程前让程序员有足够的时间来处理中断请求。
当前在实际业务代码中，你也可以忽略掉中断异常。

能够响应中断的阻塞线程，比如Thread.sleep、thread.join、可中断的I/O通道、condition.await、BlockingQueue.put等

synchronized在获锁的过程中是不能被中断的，`reentrantLock.lock()`也不能，但是`reentrantLock.tryLock(long timeout, TimeUnit unit)`方法会在指定时间内获取不到所抛出InterruptedException异常

# 处理不可中断的阻塞
并非所有的可阻塞方法或者阻塞机制都能相应中断，比如前面提到的synchronized获取锁，还有Object.wait()、tcp通信socket数据传输，I/O操作等，interrupt中断对他们不起任务作用，我们就有必要使用某种机制来中断由于这些代码导致的线程阻塞
首先`synchronized`获取导致的阻塞无解，除非改成使用`reentrantLock.tryLock(long timeout, TimeUnit unit)`就可中断，
其次，java提供了socket、I/O上的close方法，该方法会抛出异常就可解除阻塞。

另外，jdk 1.4提供InterruptibleChannel接口、实现该接口的通道是可中断的。
如果一个线程在调用实现了InterruptibleChannel接口的代码上阻塞，一个线程调用了该阻塞线程的 interrupt 方法，将会导致该通道被关闭，同时已阻塞线程接将会收到ClosedByInterruptException，并且设置已阻塞线程的中断状态。


# 示例
前面说了很多关于中断的东西，还是来看看代码吧

Tread的sleep可以响应中断
```java
public class InterruptTest {
	public static void main(String[] args) throws Exception {

		Thread t = new Thread() {
			@Override
			public void run() {
				try {
					sleep(50000); // 延迟50秒
				} catch (InterruptedException e) {
					System.out.println("捕获到InterruptedException异常，异常信息：" + e.getMessage());
				}
			}
		};
		t.start();
		System.out.println("在50秒之内按任意键中断线程!");
		System.in.read();
		t.interrupt();
		t.join();
		System.out.println("线程已经退出!");
	}
}
```
不可中断的阻塞
```java
import java.io.IOException;

public class InterruptTest {
	public static void main(String[] args) throws IOException, InterruptedException {

		Thread t = new Thread() {
			@Override
			public void run() {
				while(!Thread.currentThread().isInterrupted()){
					long c = 1;
					for (int i = 1; i < Integer.MAX_VALUE; i++) {
						c = i*c; //做乘法，耗时
					}
				}
				
				 // 线程被中断后，会跳出while循环
				System.out.println("线程被中断");
				
			}
		};
		t.start();
		System.out.println("在50秒之内按任意键中断线程!");
		System.in.read();
		t.interrupt();
		t.join();
		System.out.println("线程已经退出!");
	}
}
```

还有一种方法控制线程中断，利用InterruptedException异常
这对于线程实现了复制的算法并且分布在几个方法中，控制线程中断更方便。
由Thread实例发起interrupt()中断。
```java
if（Thread.interrupted()）{
    throw new InterruptedException();
}
```

```java
try {
    ...
} catch(InterruptedException e) {
 ...
}
```

