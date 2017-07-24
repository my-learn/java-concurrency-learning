# 前言
`Lock`功能同`synchronized`一样，都是用于对资源加锁，那么，既然已经有了synchronized，为什么还需要Lock ？
synchronized的不足：
1. 导致线程一直等待，没有一种机制能让控制他在得不到锁时中断操作
2. 读写分离。这是Lock引入的机制。什么意思？就是多个线程读不需要锁，只有在写时，才需要锁。

Lock的出现就是为了弥补这些不足。
当然我们需要注意它们的不同点：
1. synchronized是java中的关键字，而Lock只是一个类
2. 采用synchronized不需要用户去手动释放锁，而使用Lock时必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象

# Lock介绍
Lock是一个接口，在java.util.concurrent.locks包中，接口中声明了如下接口：
```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
	
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```
在使用时，为了防止死锁，一般的写法：
```java
Lock lock = new ReentrantLock();
lock.lock();
try{
    //处理任务
}catch(Exception ex){
     
}finally{
    lock.unlock();   //释放锁
}
```

Lock接口中6个方法，我们来介绍下
+ **lock():**获取锁，如果锁已被其他线程获取，则进行等待
lockInterruptibly()：当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过
+ **lock.lockInterruptibly()**想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()
```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }  
}
```
**注意，当一个线程获取了锁之后，是不会被interrupt()方法中断的。因为本身在前面的文章中讲过单独调用interrupt()方法不能中断正在运行过程中的线程，只能中断阻塞过程中的线程**。
因此当通过lockInterruptibly()方法获取某个锁时，如果不能获取到，只有进行等待的情况下，是可以响应中断的。
而用synchronized修饰的话，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。
+ **tryLock()：**该方法会立即返回，他不会将线程置入休眠。它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待
+ **tryLock(long time, TimeUnit unit)：**在指定的时间内如果还拿不到锁，就返回false；如果一开始拿到锁或者在等待期间内拿到了锁，则返回true
一般情况下通过tryLock来获取锁时是这样使用的
```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){
         
     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```
+ **unlock():**释放锁
+ **newCondition()：**





# 读写锁
## ReadWriteLock
`ReadWriteLock`是一个接口，在它里面只定义了两个方法：
```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading.
     */
    Lock readLock();
 
    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing.
     */
    Lock writeLock();
}
```
一个用来获取读锁，一个用来获取写锁。也就是说将文件的读写操作分开，分成2个锁来分配给线程，从而使得多个线程可以同时进行读操作。
它只有一个实现类`ReentrantReadWriteLock`

## ReentrantReadWriteLock
`ReentrantReadWriteLock`里面提供了很多丰富的方法，不过最主要的有两个方法：readLock()和writeLock()用来获取读锁和写锁。
下面通过几个例子来看一下ReentrantReadWriteLock具体用法。
假如有多个线程要同时进行读操作的话，先看一下synchronized达到的效果：
```java
public class Test {
    private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
     
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
    }  
     
    public synchronized void get(Thread thread) {
        long start = System.currentTimeMillis();
        while(System.currentTimeMillis() - start <= 1) {
            System.out.println(thread.getName()+"正在进行读操作");
        }
        System.out.println(thread.getName()+"读操作完毕");
    }
}
```
这段程序的输出结果会是，直到thread1执行完读操作之后，才会打印thread2执行读操作的信息。
```plain
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0读操作完毕
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1读操作完毕
```
而改成用读写锁的话：
```java
public class Test {
    private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
     
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
    }  
     
    public void get(Thread thread) {
        rwl.readLock().lock();
        try {
            long start = System.currentTimeMillis();
             
            while(System.currentTimeMillis() - start <= 1) {
                System.out.println(thread.getName()+"正在进行读操作");
            }
            System.out.println(thread.getName()+"读操作完毕");
        } finally {
            rwl.readLock().unlock();
        }
    }
}
```
此时打印的结果为：
```plain
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-0读操作完毕
Thread-1读操作完毕
```
说明thread1和thread2在同时进行读操作。
这样就大大提升了读操作的效率。
不过要注意的是，如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。
如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。


# Lock和synchronized的选择
总结来说，Lock和synchronized有以下几点不同：
1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；
2. synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；
3. Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；
4. 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。
5. Lock可以提高多个线程进行读操作的效率。
在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized。所以说，在具体使用时要根据适当情况选择。

# 修改锁的公平性
在ReentrantLock类和 ReentrantReadWriteLock类的构造器中，允许一个名为fair的boolean类型参数，它允许你来控制这些类的行为。默认值为 false，这将启用非公平模式。在这个模式中，当有多个线程正在等待一把锁（ReentrantLock或者 ReentrantReadWriteLock），这个锁必须选择它们中间的一个来获得进入临界区，选择任意一个是没有任何标准的。true值将开启公平 模式。在这个模式中，当有多个线程正在等待一把锁（ReentrantLock或者ReentrantReadWriteLock），这个锁必须选择它们 中间的一个来获得进入临界区，它将选择等待时间最长的线程。考虑到之前解释的行为只是使用lock()和unlock()方法。由于tryLock()方 法并不会使线程进入睡眠，即使Lock接口正在被使用，这个公平属性并不会影响它的功能。
```java
private Lock myLock = new ReentrantLock(true);
```

# 锁的相关概念介绍
转载地址：<http://www.cnblogs.com/dolphin0520/p/3923167.html>



