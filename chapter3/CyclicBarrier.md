CyclicBarrier和CountDownLatch一样，都是线程的计数器
从字面意思可以把它看成是一个可循环使用的屏障（同步点），它可以让一组线程到达一个屏障时，如果还有其他线程未到达屏障，则一直被阻塞，直到最后一个线程到达屏障，屏障才会放行，所有被屏障拦截的线程才会继续干活。


CyclicBarrier初始化时需要传入一个数字参数，表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞并等待被唤醒。
初始化时还可以传入一个Runnable的参数， 此Runnable任务在CyclicBarrier的数目达到后，所有其它线程被唤醒前被执行。

下面看一个基本示例
```java
package com.gotoback.current;

import java.util.Random;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierTest {

    private static final int THREAD_NUM = 3;
    
    public static class WorkerThread implements Runnable{

        CyclicBarrier barrier;
        int no;
        
        public WorkerThread(int no,CyclicBarrier b){
        	this.no = no;
            this.barrier = b;
        }
        
        @Override
        public void run() {
            try{
                System.out.println("线程"+no+"到达屏障点，阻塞...");
                //线程在这里等待，直到所有线程都到达barrier。
                barrier.await();
                int workTime = new Random().nextInt(10);
                System.out.println("线程"+no+"通过屏障点，开始工作");
                Thread.sleep(workTime*1000);
            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args) {
        CyclicBarrier cb = new CyclicBarrier(THREAD_NUM, new Runnable() {
            //当所有线程到达barrier时执行
            @Override
            public void run() {
                System.out.println("所有线程到达屏障点");
                
            }
        });
        
        for(int i=0;i<THREAD_NUM;i++){
            new Thread(new WorkerThread(i+1,cb)).start();
        }
    }

}

```
控制台打印
```plain
线程1到达屏障点，阻塞...
线程3到达屏障点，阻塞...
线程2到达屏障点，阻塞...
所有线程到达屏障点
线程2通过屏障点，开始工作
线程1通过屏障点，开始工作
线程3通过屏障点，开始工作

````

其他方法
getNumberWaiting：获得CyclicBarrier阻塞的线程数量
isBroken方法用来知道阻塞的线程是否被中断
```java
package com.gotoback.current;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierTest3 {
	static CyclicBarrier barrier = new CyclicBarrier(2);

	public static void main(String[] args) throws InterruptedException, BrokenBarrierException {
		Thread thread = new Thread(new Runnable() {

			@Override
			public void run() {
				try {
					System.out.println("当前barrier阻塞的线程数"+barrier.getNumberWaiting());
					barrier.await();
				} catch (Exception e) {
				}
			}
		});
		thread.start();
		thread.interrupt();
		try {
			barrier.await();
		} catch (Exception e) {
			System.out.println("阻塞的线程是否被中断"+barrier.isBroken()); //打印true
		}
	}
}

```

CyclicBarrier和CountDownLatch的区别
CountDownLatch的计数器只能使用一次,而CyclicBarrier的计数器可以使用reset() 方法重置。所以CyclicBarrier实现当错误发生错误时，重置计数器，并让线程们重新执行一次这样的业务场景

既然CyclicBarrier是循环的计数器，不仅体现在用reset重置上，而且在使用时，自动重置的，什么意思？比如CyclicBarrier初始计数器是3，当调用barrier.await()三次后，所有阻塞线程唤醒，但是你可以再次调用barrier.await()，CyclicBarrier又会从3开始计数。每次到达3后，再下次钓鱼那个await自动从0开始计数。
下面看一个经典的旅行团例子来理解这个特性（示例代码来自网络）
```java
package com.gotoback.current;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CyclicBarrierTest2 {
	// 徒步需要的时间: Shenzhen, Guangzhou, Shaoguan, Changsha, Wuhan
	private static int[] timeWalk = { 5, 8, 15, 15, 10 };
	// 自驾游
	private static int[] timeSelf = { 1, 3, 4, 4, 5 };
	// 旅游大巴
	private static int[] timeBus = { 2, 4, 6, 6, 7 };

	static String now() {
		SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
		return sdf.format(new Date()) + ": ";
	}

	static class Tour implements Runnable {
		private int[] times;
		private CyclicBarrier barrier;
		private String tourName;

		public Tour(CyclicBarrier barrier, String tourName, int[] times) {
			this.times = times;
			this.tourName = tourName;
			this.barrier = barrier;
		}

		public void run() {
			try {
				Thread.sleep(times[0] * 1000);
				System.out.println(now() + tourName + " 到达深圳");
				barrier.await();
				Thread.sleep(times[1] * 1000);
				System.out.println(now() + tourName + " 到达广州");
				barrier.await();
				Thread.sleep(times[2] * 1000);
				System.out.println(now() + tourName + " 到达韶关");
				barrier.await();
				Thread.sleep(times[3] * 1000);
				System.out.println(now() + tourName + " 达到长沙");
				barrier.await();
				Thread.sleep(times[4] * 1000);
				System.out.println(now() + tourName + " 达到武汉");
				barrier.await();
			} catch (InterruptedException e) {
			} catch (BrokenBarrierException e) {
			}
		}
	}

	public static void main(String[] args) {
		// 三个旅行团
		CyclicBarrier barrier = new CyclicBarrier(3);
		ExecutorService exec = Executors.newFixedThreadPool(3);
		exec.submit(new Tour(barrier, "WalkTour", timeWalk));
		exec.submit(new Tour(barrier, "SelfTour", timeSelf));
		// 当我们把下面的这段代码注释后，会发现，程序阻塞了，无法继续运行下去。
		exec.submit(new Tour(barrier, "BusTour", timeBus));
		exec.shutdown();
	}

}

```
控制台打印
```plain
11:39:47: SelfTour 到达深圳
11:39:48: BusTour 到达深圳
11:39:51: WalkTour 到达深圳
11:39:54: SelfTour 到达广州
11:39:55: BusTour 到达广州
11:39:59: WalkTour 到达广州
11:40:03: SelfTour 到达韶关
11:40:05: BusTour 到达韶关
11:40:14: WalkTour 到达韶关
11:40:18: SelfTour 达到长沙
11:40:20: BusTour 达到长沙
11:40:29: WalkTour 达到长沙
11:40:34: SelfTour 达到武汉
11:40:36: BusTour 达到武汉
11:40:39: WalkTour 达到武汉

```

