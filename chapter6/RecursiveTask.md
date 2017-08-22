RecursiveTask用于任务有返回结果的场景

前面在介绍Future时，演示了并行计算数组的和的示例，这里改成用Fork/Join框架实现
```java
import java.util.concurrent.ExecutionException;
import java.util.concurrent.RecursiveTask;

/**
 * 并行计算数组的和
 * @author Administrator
 *
 */
public class CalculatorTask extends RecursiveTask<Integer> {
	 private static final int MAX = 10;   //每次只能处理10个
	 
	private int[] numbers;
	private int start;
	private int end;

	public CalculatorTask(int start, int end, int[] numbers) {
		this.start = start;
		this.end = end;
		this.numbers = numbers;
	}

	// 计算
	@Override
	protected Integer compute() {
		int sum = 0;
		if (start - end < MAX) { 
			for (int i = start; i < end; i++) {
				sum += i;
			}
		} else {
			// 将大任务拆分多个子任务计算
			int middle = (start + end) / 2;
			CalculatorTask left = new CalculatorTask(start, middle,numbers);
			CalculatorTask right = new CalculatorTask(middle + 1, end,numbers);
			
			// 并行执行两个任务
			//left.fork();
			//right.fork();
			// 把两个任务的结果合并起来  
			//sum = left.join() + right.join();
			
			invokeAll(left,right);
			
			try {
				sum = left.get()+right.get();
			} catch(InterruptedException | ExecutionException e){
				e.printStackTrace();
			}
		}
		return sum;
	}

}
```
main
```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
	int[] numbers = new int[] { 1, 2, 3, 4, 5, 6, 7, 8, 10, 11, 12, 13 };

	ForkJoinPool forkJoinPool = new ForkJoinPool();// 对线程池的扩展
	CalculatorTask task = new CalculatorTask(1, numbers.length, numbers);
	Future<Integer> result = forkJoinPool.submit(task);
	
	//获取计算结果
	System.out.println(result.get()); // get会导致阻塞
	//也可以用task.join();来等待task任务完成，但是它跟get是有区别的，主要区别在于get不能被中断
	
	// 关闭线程池
	forkJoinPool.shutdown();
	
	if(task.isCompletedNormally()){
		System.out.println("任务成功执行完成");
	}
}
```

运行结果：
```plain
66
任务成功执行完成

```