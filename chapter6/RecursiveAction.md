RecursiveAction用于任务没有返回结果的场景

下面演示一个例子：从数字0打印到20

先定义RecursiveAction任务
```java
import java.util.concurrent.RecursiveAction;

public class PrintTask extends RecursiveAction {
	// 每个任务最多只打印10个数字
	private static final int MAX = 10;

	private int start;
	private int end;

	public PrintTask(int start, int end) {
		this.start = start;
		this.end = end;
	}

	@Override
	protected void compute() {
		// 当end-start的值小于MAX时候，开始打印
		if (end - start < MAX) {
			for (int i = start; i < end; i++) {
				System.out.println(Thread.currentThread().getName() + "的i值:" + i);
			}
		} else {
			// 将大任务分解成两个小任务
			int middle = (start + end) / 2;
			PrintTask left = new PrintTask(start, middle);
			PrintTask right = new PrintTask(middle, end);
			// 并行执行两个小任务
			//left.fork();
			//right.fork();
			invokeAll(left,right); //同步方式执行任务
		}
	}
}
```

main方法
```java
public static void main(String[] args) throws Exception {
	ForkJoinPool forkJoinPool = new ForkJoinPool();
	// 提交一个RecursiveAction
	forkJoinPool.submit(new PrintTask(0, 200));

	// 阻塞当前线程直到ForkJoinPool中所有的任务都执行结束
	forkJoinPool.awaitTermination(2, TimeUnit.SECONDS);

	System.out.println("打印完成...");

	// 关闭线程池
	forkJoinPool.shutdown();

}
```
运行结果
```plain
ForkJoinPool-1-worker-3的i值:10
ForkJoinPool-1-worker-4的i值:0
ForkJoinPool-1-worker-2的i值:5
ForkJoinPool-1-worker-1的i值:15
ForkJoinPool-1-worker-2的i值:6
ForkJoinPool-1-worker-2的i值:7
ForkJoinPool-1-worker-2的i值:8
ForkJoinPool-1-worker-2的i值:9
ForkJoinPool-1-worker-4的i值:1
ForkJoinPool-1-worker-4的i值:2
ForkJoinPool-1-worker-3的i值:11
ForkJoinPool-1-worker-4的i值:3
ForkJoinPool-1-worker-1的i值:16
ForkJoinPool-1-worker-4的i值:4
ForkJoinPool-1-worker-3的i值:12
ForkJoinPool-1-worker-3的i值:13
ForkJoinPool-1-worker-1的i值:17
ForkJoinPool-1-worker-3的i值:14
ForkJoinPool-1-worker-1的i值:18
ForkJoinPool-1-worker-1的i值:19
打印完成...
```
从上面结果来看，ForkJoinPool启动了四个线程来执行这个打印任务，因为我的电脑是双核四线程，ForkJoinPool默认就是根据CPU数来分配线程的。
打印的数字是随机的，说明多个线程任务是并行执行的。


