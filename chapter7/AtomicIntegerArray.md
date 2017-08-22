原子数组提供了对原子变量(AtomicInteger、AtomicLong)的数组的原子操作

下面演示一下怎么使用
创建两个线程，一个对数组元素增加值，另个对数组元素减少值
```java
import java.util.concurrent.atomic.AtomicIntegerArray;

public class Add implements Runnable {

	private AtomicIntegerArray intArr;

	public Add(AtomicIntegerArray intArr) {
		this.intArr = intArr;
	}

	@Override
	public void run() {
		for (int i = 0; i < intArr.length(); i++) {
			intArr.getAndIncrement(i);
		}
	}

}
```
第二个线程
```java
public class Subtract implements Runnable {

	private AtomicIntegerArray intArr;

	public Subtract(AtomicIntegerArray intArr) {
		this.intArr = intArr;
	}

	@Override
	public void run() {
		for (int i = 0; i < intArr.length(); i++) {
			intArr.getAndDecrement(i);
		}
	}
}
```
测试类
```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicIntegerArray;

public class Test2 {
	public static void main(String[] args) throws Exception {
		AtomicIntegerArray intArr = new AtomicIntegerArray(1000);
		
		Add inc = new Add(intArr);
		Subtract dec = new  Subtract(intArr);
		
		List<Thread> tlist = new ArrayList<Thread>();
		
		for (int i = 0; i < 1000; i++) {
			Thread t1 = new Thread(inc);
			Thread t2 = new Thread(dec);
			
			t1.start();
			t1.yield();
			t2.start();
			t2.yield();
			
			tlist.add(t1);
			tlist.add(t2);
		}

		for (int i = 0; i < tlist.size(); i++) {
			tlist.get(i).join();
		}
		
		// 如果存在不为0的元素，证明整个add、substract操作是线程安全的
		for (int i = 0; i < intArr.length(); i++) {
			if(intArr.get(i)!=0)
				System.out.println("存在不为0的元素 "+intArr.get(i));
		}
	}
}
```
我们看到使用了AtomicIntegerArray原子数组后，在并发情况下，对数组元素操作是线程安全的