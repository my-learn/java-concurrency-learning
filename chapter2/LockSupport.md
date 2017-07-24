对线程阻塞与唤醒，我们经常使用object的wait和notify，除了这种方式外，Java并发包还提供LockSupport实现对线程的挂起和恢复

LockSupport用来创建锁和其他同步类的基本线程阻塞原语。
因为这个类实现了锁的底层实现，各种的锁实现，比如ReentrantLock、Thread，都是基于LockSupport实现的

LockSupport中的park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程，而且park()和unpark()不会遇到“Thread.suspend 和 Thread.resume所可能引发的死锁”问题。因为park() 和 unpark()有许可的存在；调用 park() 的线程和另一个试图将其 unpark() 的线程之间的竞争将保持活性。
LockSupport是通过调用Unsafe函数中的接口实现阻塞和解除阻塞的。


# 示例
既然LockSupport和wait-notify实现一样的功能，我们先拿wait/notify写一个例子
```java
package com.gotoback.current;

public class WatiNotifyTest {

    public static void main(String[] args) {

        ThreadA ta = new ThreadA("ta");

        synchronized(ta) { // 通过synchronized(ta)获取“对象ta的同步锁”
            try {
                System.out.println(Thread.currentThread().getName()+" start ta");
                ta.start();

                System.out.println(Thread.currentThread().getName()+" block");
                // 主线程等待
                ta.wait();
                
                System.out.println(Thread.currentThread().getName()+" continue");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class ThreadA extends Thread{

        public ThreadA(String name) {
            super(name);
        }

        public void run() {
            synchronized (this) { // 通过synchronized(this)获取“当前对象的同步锁”
                System.out.println(Thread.currentThread().getName()+" wakup others");
                notify();    // 唤醒“当前对象上的等待线程”
            }
        }
    }
}

```
结果
```plain
main start ta
main block
ta wakup others
main continue

```
改成用LockSupport实现
```java
package com.gotoback.current;

import java.util.concurrent.locks.LockSupport;

public class LockSupportTest {

    private static Thread mainThread;

    public static void main(String[] args) {

        ThreadA ta = new ThreadA("ta");
        // 获取主线程
        mainThread = Thread.currentThread();

        System.out.println(Thread.currentThread().getName()+" start ta");
        ta.start();

        System.out.println(Thread.currentThread().getName()+" block");
        // 主线程阻塞
        LockSupport.park(mainThread);

        System.out.println(Thread.currentThread().getName()+" continue");
    }

    static class ThreadA extends Thread{

        public ThreadA(String name) {
            super(name);
        }

        public void run() {
            System.out.println(Thread.currentThread().getName()+" wakup others");
            // 唤醒“主线程”
            LockSupport.unpark(mainThread);
        }
    }
}

```

LockSupport源码分析
先上源码（略。。。）
这个类提供的都是静态方法，且无法被实例化。
LockSupport有两个私有的成员变量
```java
// Hotspot implementation via intrinsics API
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long parkBlockerOffset;
```
parkBlockerOffset作用：用于记录线程是被谁阻塞的。可以通过LockSupport的getBlocker获取到阻塞的对象，用于监控和分析线程。

# 对比
既然LockSupport和wait-notify实现一样的功能，自然就要拿他们两做对比了。
阻塞和唤醒是对于线程的，LockSupport的park/unpark更符合这个语义，以“线程”作为方法的参数，语义清晰，使用方便。而wait/notify的实现对线程的阻塞/唤醒是被动的，只能随机唤醒一个(notify)或唤醒所有(notifyAll)

