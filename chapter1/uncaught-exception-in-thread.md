我们看到run方法不能throws异常，如果报出一个运行时异常怎么办？Java提供一种在线程对象里捕获和处理运行时异常的一种机制。
```java
thread.setUncaughtExceptionHandler(new     
    Thread.UncaughtExceptionHandler() {
			
	@Override
	public void uncaughtException(Thread t, Throwable e) {
		System.out.println("报错了...");
		
	}
})
```
随便写个代码，抛一个运行时异常就行了，就能验证这个功能。
Thread类还有另一个方法可以处理未捕获到的异常，即静态方法`setDefaultUncaughtExceptionHandler()`。这个方法在应用程序中为所有的线程对象创建了一个异常处理器。

当线程抛出一个未捕获的异常并且没有被捕获时** (这种情况只可能是运行时异常) **，JVM将为异常寻找以下三种可能的处理器：
首先，它查找线程对象的未捕获异常处理器。如果找不到，JVM继续查找线程对象所在的线程组（ThreadGroup）的未捕获异常处理器。如果还是找不到（参考文章《线程组》），如同本节所讲的，JVM将继续查找默认的未捕获异常处理器。
如果没有一个处理器存在，JVM则将堆栈异常记录打印到控制台，并退出程序。


