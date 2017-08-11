所有线程都隶属于一个线程组。可以是一个默认线程组，也可以是一个创建线程时明确指定的组。
 
说明：
在创建之初，线程被限制到一个组里，而且不能改变到一个不同的组。
若创建多个线程而不指定一个组，它们就会与创建它的线程属于同一个组。
 
使用很简单，如下所示
```java
ThreadGroup threadGroup = new ThreadGroup("tt");
Thread t1 = new Thread(threadGroup,myThread);
```
ThreadGroup也有很多方法
![](/chapter1/161.png)
具体使用可参考API文档！

线程组中不可控异常的处理
对于线程中不可控异常的处理的介绍,请参考文章 [Thread类详解](/chatper1/1-4.md)
```java
class MyTreadGroup extends ThreadGroup {
	public MyTreadGroup(String name) {
		super(name);
	}
	
	@Override
	public void uncaughtException(Thread t, Throwable e) {
		e.printStackTrace();
		interrupt(); // 中断其余的线程
	}
	
}
```

