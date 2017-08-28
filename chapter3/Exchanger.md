Exchanger 用于两个并发线程之间在一个同步点进行数据交换。 这两个线程通过exchange方法交换数据，如果第一个线程先执行exchange方法，它会一直等待第二个线程也执行exchange方法，当两个线程都达到同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

需要注意几点：Exchanger 只能用于两个线程之间交换数据，调用exchange会导致阻塞，直到另一个线程达到同步调交换数据，也就是调用exchange，如果只有一个线程调用了一次exchange，会导致死锁。

该类只有一个方法
V exchange(V x):等待其他线程到达交换点，然后将数据给别人，同时自己从别人那拿数据。它还有一个支持超时的重载方法。

api文档上给了一个在两个线程间交换buffer数读取据的示例，有兴趣的可以看下，这里我给出一个类似的例子：10块布换一匹马的示例
```java
import java.util.concurrent.Exchanger;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExchangeTest {
	public static void main(String[] args) {
		ExecutorService service = Executors.newCachedThreadPool();
		final Exchanger<String> exchanger = new Exchanger<String>();

		service.execute(new Runnable() {
			
			String data = "10块布";

			@Override
			public void run() {
				try {
					System.out.println("线程" + Thread.currentThread().getName() + "正在来的路上");
					Thread.sleep((long) Math.random() * 10000);
					System.out.println("线程" + Thread.currentThread().getName() + "到达指定地点，准备交换");
					String data2 = exchanger.exchange(data);
					System.out.println("线程 " + Thread.currentThread().getName() + "用"+data+"换回的 " + data2);
				} catch (Exception e) {
					e.printStackTrace();
				}

			}
		});

		service.execute(new Runnable() {
			
			String data = "一匹马";

			@Override
			public void run() {
				try {
					System.out.println("线程" + Thread.currentThread().getName() + "正在来的路上");
					Thread.sleep((long) (Math.random() * 10000));
					System.out.println("线程" + Thread.currentThread().getName() + "到达指定地点，准备交换");
					String data2 = exchanger.exchange(data);
					System.out.println("线程 " + Thread.currentThread().getName() + "用"+data+"换回的 " + data2);
				} catch (Exception e) {
					e.printStackTrace();
				}

			}
		});
		
		service.shutdown();
	}
}
```
打印结果：
```plain
线程pool-1-thread-1正在来的路上
线程pool-1-thread-1到达指定地点，准备交换
线程pool-1-thread-2正在来的路上
线程pool-1-thread-2到达指定地点，准备交换
线程 pool-1-thread-2用一匹马换回的 10块布
线程 pool-1-thread-1用10块布换回的 一匹马
```

用Exchanger 也能模拟生产者-消费者
```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Exchanger;

public class ExchangeTest2 {
	public static void main(String[] args) {
		Exchanger<List<String>> exchanger = new Exchanger<>();
		new Thread(new Producer(exchanger)).start();
		new Thread(new Consumer(exchanger)).start();

	}
}

class Producer implements Runnable {
	private List<String> goods = new ArrayList<>();
	private Exchanger<List<String>> exchanger;

	public Producer(Exchanger<List<String>> exchanger) {
		this.exchanger = exchanger;
	}

	@Override
	public void run() {
		for (int j = 1; j <= 3; j++) { // 生成3次
			System.out.println("========开始第" + j + "次生产========");
			for (int i = 1; i <= 5; i++) { // 每次生产5个商品
				goods.add("商品" + j + "_" + i);
				System.out.println("生成了商品，编号" + j + "_" + i);
			}

			try {
				goods = exchanger.exchange(goods);
				System.out.println("交换完后，生产者商品库存量为【" + goods.size() + "】，说明商品都被消费了");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}

	}

}

class Consumer implements Runnable {
	private List<String> goods = new ArrayList<>();
	private Exchanger<List<String>> exchanger;

	public Consumer(Exchanger<List<String>> exchanger) {
		this.exchanger = exchanger;
	}

	@Override
	public void run() {
		for (int i = 1; i <= 3; i++) {
			try {
				goods = exchanger.exchange(goods);
				System.out.println("交换完后，消费者获取到的商品数量为【" + goods.size() + "】");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}
```
打印结果如下：
```plain
========开始第1次生产========
生成了商品，编号1_1
生成了商品，编号1_2
生成了商品，编号1_3
生成了商品，编号1_4
生成了商品，编号1_5
交换完后，消费者获取到的商品数量为【5】
交换完后，生产者商品库存量为【0】，说明商品都被消费了
========开始第2次生产========
生成了商品，编号2_1
生成了商品，编号2_2
生成了商品，编号2_3
生成了商品，编号2_4
生成了商品，编号2_5
交换完后，生产者商品库存量为【5】，说明商品都被消费了
交换完后，消费者获取到的商品数量为【5】
========开始第3次生产========
生成了商品，编号3_1
生成了商品，编号3_2
生成了商品，编号3_3
生成了商品，编号3_4
生成了商品，编号3_5
交换完后，生产者商品库存量为【5】，说明商品都被消费了
交换完后，消费者获取到的商品数量为【10】
```
