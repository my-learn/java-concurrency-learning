Condition可以用来替代传统的Object的wait()、notify()实现线程间的协作。Condition比wait/notify功能更强大灵活。

一个锁可能关联一个或者多个条件，这些条件通过Condition接口声明。
Condition对象由Lock对象的newCondition()方法创建，调用condition.await（）来实现让线程等待，进入阻塞；调用condition.signal()唤醒线程，注意唤醒线程调用的condition必须是同一个Condition
和wait/notify一样，await（）和signal（）也是在同步代码区内执行，所以await()、signal()都必须在lock.lock()和lock.unlock之间才可以使用。


Condition接口中的几个方法
await()：当线程调用条件的await()方法时，它将自动释放这个条件绑定的锁，其他某个线程才可以获取这个锁并且执行相同的操作，或者执行这个锁保护的另一个临界区代码。
* await(long time,TimeUnit unit)：进入阻塞，等待被唤醒
* signal()/signalAll()：当一个线程调用了条件对象的singal()或者signalAll()方法后，一个或者多个在该条件上挂起的线程将被唤醒，但这并不能保证让他们挂起的条件已经满足，所以必须在while循环中调用await()，在条件成立之前不能离开这个循环。如果条件不成立，将再次调用await()。
* awatiUninterruptibly()：
* awaitUntil(Date date)：

必须小心使用await()和singal()方法。如果调用了一个条件的await()方法，却从不调用它的singal()方法，这个线程将永久休眠。

# 示例
```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Test {

	public static void main(String[] args) {
		final Lock lock = new ReentrantLock();
		final Condition condition = lock.newCondition();

		new Thread(new Runnable() {
			@Override
			public void run() {
				lock.lock();
				System.out.println(Thread.currentThread().getName() + "拿到了锁");
				System.out.println(Thread.currentThread().getName() + "等待信号");
				try {
					condition.await();
					System.out.println(Thread.currentThread().getName() + "拿到信号");
				} catch (InterruptedException e) {
					e.printStackTrace();
				} finally {
					lock.unlock();
				}

			}
		}, "线程1").start();

		new Thread(new Runnable() {
			@Override
			public void run() {
				lock.lock();
				System.out.println(Thread.currentThread().getName() + "拿到了锁");

				try {
					Thread.sleep(3000);
					System.out.println(Thread.currentThread().getName() + "发出信号");
					condition.signalAll();
				} catch (InterruptedException e) {
					e.printStackTrace();
				} finally {
					lock.unlock();

				}
			}
		}, "线程2").start();
	}
}
```
执行结果：
```plain 
线程1拿到了锁
线程1等待信号
线程2拿到了锁
线程2发出信号
线程1拿到信号
```
