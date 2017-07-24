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

# LockSupport源码分析
先上源码（略。。。）
这个类提供的都是静态方法，且无法被实例化。
LockSupport有两个私有的成员变量
```java
// Hotspot implementation via intrinsics API
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long parkBlockerOffset;
```
parkBlockerOffset作用：挂起线程对象的偏移地址，对应的是Thread类的parkBlocker。用来记录线程是被谁堵塞的，当程序出现问题时候，通过线程监控分析工具可以找出问题所在。

# 对比
LockSupport阻塞和唤醒线程直接操作的就是线程，所以更符合语义。而Object的wait/notify它并不是直接对线程操作，它是被动的方法，它需要一个object来进行线程的挂起或唤醒，只能随机唤醒一个(notify)或唤醒所有(notifyAll)。
park/unpark使用起来会更加的灵活、方便。因为在调用对象的wait之前当前线程必须先获得该对象的监视器（synchronized），被唤醒之后需要重新获取到监视器才能继续执行。而LockSupport则不需要，它可以随意进行park或者unpark。

