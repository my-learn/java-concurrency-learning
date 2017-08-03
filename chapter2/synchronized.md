<!-- toc -->

# 资源加锁介绍
在并发情况下，多个线程同时访问一个临界资源时，会导致资源竞争，出现意想不到的结果，所以必须要保证在并发情况下临界资源在同一个时刻只允许一个线程访问。
通常做法就是对该临界资源加锁即可。在java中，可以使用关键字synchronized加锁
加锁的资源，其他线程无法访问，会阻塞等待，直到被解锁

# synchronized介绍
Java中的每个对象都有一个锁（lock），或者叫做监视器（monitor），当一个线程访问某个对象的synchronized方法时，将该对象上锁，其他任何线程都无法再去访问该对象的synchronized方法了。直到之前的那个线程执行方法完毕后（或者是抛出了异常），才将该对象的锁释放掉，其他线程才有可能再去访问该对象的synchronized方法。
**注意这时候是给对象上锁，如果是不同的对象，则各个对象之间没有限制关系。**
**synchronized关键字会降低程序的性能。**

# synchronized同步方法或者同步块
在Java中，可以使用synchronized关键字来标记一个方法或者代码块，当synchronized关键字修饰一个方法的时候，该方法叫做同步方法。
当某个线程调用该对象的synchronized方法或者访问synchronized代码块时，这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法，只有等待这个方法执行完毕或者代码块执行完毕，这个线程才会释放该对象的锁，其他线程才能执行这个方法或者代码块。
## synchronized方法
```java
public synchronized void insert(User user){
	...
}
```

## synchronized代码块
synchronized代码块类似于以下这种形式
```java
synchronized(synObject) {
	 
}
```
当在某个线程中执行这段代码块，该线程会获取对象synObject的锁，从而使得其他线程无法同时访问该代码块。
synObject可以是this，代表获取当前对象的锁，也可以是类中的一个属性，代表获取该属性的锁。通常我们使用this关键字来引用正在执行的方法所属的对象。
比如:
```java
public void insert(User user){
	synchronized (this) { // 这里的this指当前对象
		...
	}
}
```
或者
```java
// 先定义一个对象
private Object object = new Object();
public void insert(User user){
	synchronized (object) {
		...
	}
}
```
也或者写成缺省形式
```java
public void insert(User user){
	synchronized ("") {
		...
	}
}
```
从上面可以看出，synchronized代码块使用起来比synchronized方法要灵活得多。因为也许一个方法中只有一部分代码只需要同步，如果此时对整个方法用synchronized进行同步，会影响程序执行效率。而使用synchronized代码块就可以避免这个问题，synchronized代码块可以实现只对需要同步的地方进行同步。

# 类级别的synchronized
每个类也会有一个锁，它可以用来控制对static数据成员的并发访问
因为静态方法不属于对象，而是属于类，所以它会将这个方法所在的类的Class对象上锁。一个类不管生成多少个对象，它们所对应的是同一个Class对象。
```java
public synchronized static void insert1() {
	System.out.println("执行insert1");
}
```
类级别
```java
public void insert(User user){
	synchronized (类名.class) { // 这里的类名指当前类
		...
	}
}
```

# synchronized关键字使用总结
1. 当一个线程正在访问一个对象的synchronized方法，那么其他线程不能访问该对象的其他synchronized方法。这个原因很简单，因为一个对象只有一把锁，当一个线程获取了该对象的锁之后，其他线程无法获取该对象的锁，所以无法访问该对象的其他synchronized方法。
2. 当一个线程正在访问一个对象的synchronized方法，那么其他线程能访问该对象的非synchronized方法。这个原因很简单，访问非synchronized方法不需要获得该对象的锁，假如一个方法没用synchronized关键字修饰，说明它不会使用到临界资源，那么其他线程是可以访问这个方法的，
3. 如果另一个线程访问的是同一个类的不同对象，这两个线程都不会被阻塞。因为他们访问的是不同的对象，所以不存在互斥问题。
4. 如果一个线程执行一个对象的非static synchronized方法，另外一个线程需要执行这个对象所属类的static synchronized方法，此时不会发生互斥现象，因为访问static synchronized方法占用的是类锁，而访问非static synchronized方法占用的是对象锁，所以不存在互斥现象。
5. 对于synchronized方法或者synchronized代码块，当出现异常时，JVM会自动释放当前线程占用的锁，因此不会由于异常导致出现死锁现象。
6. 可以递归调用被synchronized声明的方法。当线程访问一个对象的同步方法时，他还可以调用这个对象的其他的同步方法，也包含正在执行的方法，而不必再次去获取这个方法的访问权。
