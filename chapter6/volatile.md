volatile是一个轻量级的锁机制。用volatile修饰的变量，线程在每次使用变量的时候，都会读取变量修改后的最的值。但是volatile很容易被误用，产生意想不到的结果。
大部分情况下volatile 变量是无法替代锁的。
因为volatile有禁止指令重排序的语义，保证了可见性，但是无法保证原子性


volatile 变量具有 synchronized 的可见性特性，但是不具备原子性。所以一般使用volatile必须具备以下2个条件：
* 对变量的写操作不依赖于当前值。
* 该变量没有包含在具有其他变量的不变式中。

错误的使用
volatile不能用作计数器，因为i++不是原子操作，示例
```java
public class Counter {

	public static int count = 0;

	public static void inc() {
		try {
			Thread.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		count++;
	}

	public static void main(String[] args) {
		for (int i = 0; i < 1000; i++) {
			new Thread(new Runnable() {
				@Override
				public void run() {
					Counter.inc();
				}
			}).start();
		}

		System.out.println("运行结果:Counter.count=" + Counter.count);
	}
}
```
每次运行的记过都不一样，即使给count 加上volatile修饰，也是一样的。

使用场景
状态标记量
```java
volatile boolean falg;  
  
public void shutdown() {   
    falg = true;   
}  
  
public void doWork() {   
    while (!falg) {   
        // do stuff  
    }  
}  
```

相关链接
http://www.importnew.com/18126.html
http://www.cnblogs.com/aigongsi/archive/2012/04/01/2429166.html
http://blog.csdn.net/vking_wang/article/details/9982709
https://www.ibm.com/developerworks/cn/java/j-jtp06197.html



