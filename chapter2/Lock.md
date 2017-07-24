# 前言
`Lock`功能同`synchronized`一样，都是用于对资源加锁，那么，既然已经有了synchronized，为什么还需要Lock ？
synchronized的不足：
1. 导致线程一直等待，没有一种机制能让控制他在得不到锁时中断操作
2. 读写分离。这是Lock引入的机制。什么意思？就是多个线程读不需要锁，只有在写时，才需要锁。
3. synchronized是非公平锁，无法保证等待时间最久的线程优先执行，而ReentrantLock可以设置锁的公平性

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

# 方法
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

# Lock概览
Lock相关的类和接口都在java.util.concurrent.lock包下，该包整体结构
![](/chapter2/diagram-1.png)

重要的接口Lock和ReadWriteLock，实际基本使用他们的实现类ReentrantLock、ReentrantReadWriteLock。


# 锁的相关概念介绍
转载地址：<http://www.cnblogs.com/dolphin0520/p/3923167.html>



