ABA问题可以先自行百度。

在很多情况下，ABA问题不是什么大问题，可以忽略，但是有些特殊情况对值监视比较敏感的场景，例如 现在有个赠送优惠卷活动，为了刺激用户消费，账号余额低于20块的就赠送给一个100元优惠卷。ABA问题体现在，如果用户充过钱又消费出现的低于20块，那就不应该赠送优惠卷。
当然解决ABA问题，本身是可以从业务上解决的，比如先判断用户有没有消费过，没消费过同时余额低于20，就赠送优惠卷。


atomic包中提供的两个类AtomicMarkableReference、AtomicStampedReference用来解决ABA问题的

AtomicMarkableReference通过内部一个&lt; reference,int&gt;结构实现的，可用于表示与否的状态;
AtomicStampedReference通过内部一个&lt; reference,int&gt;结构实现的，可用来作为带版本号的原子引用;

其实AtomicStampedReference完全可以代替AtomicMarkableReference，但是AtomicMarkableReference是用boolean表示状态与否变化，更符合语义点。

在演示这两个类的使用前，先看看以下代码演示产生ABA问题
```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicReference;

public class Test0 {

	public final static AtomicReference<String> ATOMIC_REFERENCE = new AtomicReference<String>("aa");

	public static void main(String[] args) {
		List<Thread> list = new ArrayList<Thread>();
		for (int i = 0; i < 100; i++) {
			Thread t = new Thread() {
				public void run() {
					try {
						Thread.sleep(Math.abs((int) (Math.random() * 100)));
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					if (ATOMIC_REFERENCE.compareAndSet("aa", "bb")) {
						System.out.println(Thread.currentThread().getName() + "将值修改为了bb");
					}
				}
			};
			t.start();

			list.add(t);

		}
		Thread t2 = new Thread() {
			public void run() {
				while (!ATOMIC_REFERENCE.compareAndSet("bb", "aa"))
					;
				System.out.println(Thread.currentThread().getName() + "将值改为aa");
			}
		};
		t2.start();
		list.add(t2);

		for (int i = 0; i < list.size(); i++) {
			try {
				list.get(i).join();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}

		System.out.println("最后的值为" + ATOMIC_REFERENCE.get());

	}
}
```
结果打印：
```plain
Thread-0将值修改为了bb
Thread-100将值改为aa
Thread-26将值修改为了bb
最后的值为bb

```
上面的代码中，线程t将aa改成bb，线程t2将bb改成aa，从结果来看，t执行了compareAndSet("aa", "bb")成功了两次，第一次执行时的预期值aa和第二次执行时的预期值aa并不是同一个，因为第二个aa是线程t2赋值进来的。

下面使用AtomicStampedReference解决该问题
```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicStampedReference;

public class Test {

	public final static AtomicStampedReference<String> ATOMIC_REFERENCE = new AtomicStampedReference<String>("aa", 1);

	public static void main(String[] args) {
		List<Thread> list = new ArrayList<Thread>();
		final int stamp = ATOMIC_REFERENCE.getStamp();
		for (int i = 0; i < 100; i++) {
			Thread t = new Thread() {
				public void run() {
					try {
						Thread.sleep(Math.abs((int) (Math.random() * 100)));
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					if (ATOMIC_REFERENCE.compareAndSet("aa", "bb", stamp, stamp + 1)) {
						System.out.println(Thread.currentThread().getName() + "将值修改为了bb");
					}
				}
			};
			t.start();

			list.add(t);

		}
		Thread t2 = new Thread() {
			int stamp = ATOMIC_REFERENCE.getStamp();

			public void run() {
				while (!ATOMIC_REFERENCE.compareAndSet("bb", "aa", stamp, stamp + 1))
					;
				System.out.println(Thread.currentThread().getName() + "将值改为aa");
			}
		};
		t2.start();
		list.add(t2);

		for (int i = 0; i < list.size(); i++) {
			try {
				list.get(i).join();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}

		System.out.println("最后的值为" + ATOMIC_REFERENCE.get(new int[1]));

	}
}
```
结果打印
```plain
Thread-66将值修改为了bb
Thread-100将值改为aa
最后的值为aa

```
线程t第二次执行compareAndSet("aa", "bb", stamp, stamp + 1)时，stamp值不一样，就任务不是预期值，不执行set了

AtomicMarkableReference也可以解决该问题
```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicMarkableReference;

public class Test2 {

	public final static AtomicMarkableReference<String> ATOMIC_REFERENCE = new AtomicMarkableReference<String>("aa",false);

	public static void main(String[] args) {
		List<Thread> list = new ArrayList<Thread>();
		for (int i = 0; i < 100; i++) {
			Thread t = new Thread() {
				public void run() {
					try {
						Thread.sleep(Math.abs((int) (Math.random() * 100)));
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					if (ATOMIC_REFERENCE.compareAndSet("aa", "bb", false, true)) {
						System.out.println(Thread.currentThread().getName() + "将值修改为了bb");
					}
				}
			};
			t.start();

			list.add(t);

		}
		Thread t2 = new Thread() {
			public void run() {
				while (!ATOMIC_REFERENCE.compareAndSet("bb", "aa", true, true))
					;
				System.out.println(Thread.currentThread().getName() + "将值改为aa");
			}
		};
		t2.start();
		list.add(t2);

		for (int i = 0; i < list.size(); i++) {
			try {
				list.get(i).join();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}

		System.out.println("最后的值为" + ATOMIC_REFERENCE.get(new boolean[1]));

	}
}
```
打印结果跟AtomicStampedReference是一样的

其实，AtomicMarkableReference 并不能完全解决ABA问题，因为他只有一个boolean来标记是否更改，就两个状态，可能实际需要更多的状态判断，而AtomicStampedReference对数据赋予了版本号的能力，控制数据更灵活。





