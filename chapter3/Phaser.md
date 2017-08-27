`Phaser`是jdk 1.7引入的线程同步辅助类，`Phaser`代表一个可重复使用的同步屏障(barrier)，功能类似`CyclicBarrier`和`CountDownLatch`，但是比他们更强大灵活。
<br />
首先解释下这个英文单词的意思，Phase翻译成中文就是阶段的意思。Phaser类机制是在每一个阶段结束的时候对线程进行同步，当所有的线程都到达该阶段，才能进入下一个阶段。 这个机制刚好说明Phaser侧重在“重用”二字上。
phase阶段初值为0，当所有的线程执行完本轮任务，同时开始下一轮任务时，意味着当前阶段已结束，进入到下一阶段，phase的值自动加1。

当我们有并发任务并且需要分解成几步执行的时候，这种机制就非常适合。 

相对于CyclicBarrier CountDownLatch来说，他们都只能在构造时指定成员参与数，而Phaser可以动态的添加或删除参与者。

# 构造函数
先从构造函数开始介绍，因为要开始使用Phase，得从构造函数开始初始化。Phase提供了4个构造函数：
* Phaser():创建一个Phaser，并且parties个数为0，以后我们可以通过register()、bulkRegister()方法来注册新的parties。
* Phaser(int parties):指定参与者个数，相当于直接regsiter了此数量的parties
* Phaser(Phaser parent, int parties):注册一个子Phase到父Phase中
* Phaser(Phaser parent):等价于Phaser(parent, 0)


# Phase提供的方法
* register():将一个新的参与者(party)注册到phase中，parties个数加一，这个新的参与者将被当成没有执完本阶段的线程。如果此时onAdvance方法正在执行，此方法将会等待它执行完毕后才会返回。此方法返回当前的phase周期数，如果Phaser已经中断，将会返回负数。
* bulkRegister(int parties)：跟register一样，只是可以注册多个party

* arrive()：参与者已经到达该phaser阶段，不需要该等待其他参与者都完成当前阶段，即该方法非阻塞的。如果没有register（即已register数量为0），调用此方法将会抛出异常，此方法返回当前phase周期数，如果Phaser已经终止，则返回负数。必须小心使用这个方法，因为它不会与其他线程同步。
* arriveAndDeregister():参与者已经到达该phaser阶段，并且减少参与者即parties个数减一，不需要该等待其他参与者都完成当前阶段。
* arriveAndAwaitAdvance()：参与者已经到达该phaser阶段，并且并等待其他参与者到达，才开始运行下面的代码。该方法等同于awaitAdvance(arrive());的效果

* awaitAdvance(int phase)：如果传入的阶段参数与当前阶段一致，这个方法会将当前线程至于休眠，直到这个阶段的所有参与者都运行完成。如果传入的阶段参数与当前阶段不一致，这个方法立即返回。
* awaitAdvanceInterruptibly(int phaser):这个方法跟awaitAdvance(int phase)一样，不同处是：该访问将会响应线程中断。会抛出interruptedException异常。
* awaitAdvanceInterruptibly(int phase,long timeout, TimeUnit unit):同上，可以指定一个等待时间，超时后抛出TimeoutException异常。

* forceTermination()：强制终止。当一个phaser没有参与者的时候，它就处于终止状态，使用forceTermination()方法来强制phaser进入终止状态，不管是否存在未注册的参与线程，当一个线程出现错误时，强制终止phaser是很有意义的。
* isTerminated(): 阶段是否已经结束
* onAdvance():这个后面再介绍
* forceTermination()：强制终止，此后Phaser对象将不可用，即register等方法将不再有效。此方法将会导致Queue中所有的waiter线程被唤醒。
* getArrivedParties():获取已经到达的parties个数。
* getPhase()：获取当前phase周期数。如果Phaser已经中断，则返回负值。
* getRegisteredParties()：获取已经注册的parties个数。
* getUnarrivedParties()：获取尚未到达的parties个数。


#　onAdvance
onAdvance是Phaser的一个重要方法，可以被重载。该方法源代码如下
```java
protected boolean onAdvance(int phase, int registeredParties) {
	return registeredParties == 0;
}
```
该方法默认的实现是在参与者数量为0的时候，返回true，表示该phaser状态为终止状态（isTermination()将返回true）。该方法在phaser阶段改变的时候会自动执行，相当于CyclicBarrier的构造函数中指定的第二个参数Runnable一样的效果。
这有两个参数phase表示当前阶段数，registeredParties表示注册的参与者数量。



# 示例
先看一个最简单的示例，通过Phaser控制多个线程的同步时机
```java
import java.util.Random;
import java.util.concurrent.Phaser;

public class PhaserTest1 {
	// 参与的parties个数  
	private static int PARTIES = 3;
	
	public static void main(String[] args) throws InterruptedException {
		Phaser phaser = new Phaser(PARTIES);
		for (int i = 0; i < PARTIES; i++) {
			new MyThread(phaser).start();
		}
	}

}

class MyThread extends Thread {
	private Phaser phaser;

	public MyThread(Phaser phaser) {
		this.phaser = phaser;
	}

	@Override
	public void run() {
		System.out.println(Thread.currentThread().getName() + "任务开始工作");
		int workTime = new Random().nextInt(10-3+1)+3;//随机3-10秒，方便看效果
		try {
			Thread.sleep(workTime * 1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(Thread.currentThread().getName() + "任务处理完成，耗时"+workTime+"秒， 等待其他任务执行完成 ");
		// 等待其他任务执行完成，这儿会阻塞
		phaser.arriveAndAwaitAdvance();
		System.out.println(Thread.currentThread().getName() + " 进入下一阶段，如果没有下一阶段则结束 ");
	}
}
```
打印结果：
```plain
Thread-0任务开始工作
Thread-2任务开始工作
Thread-1任务开始工作
Thread-0任务处理完成，耗时3秒， 等待其他任务执行完成 
Thread-2任务处理完成，耗时6秒， 等待其他任务执行完成 
Thread-1任务处理完成，耗时8秒， 等待其他任务执行完成 
Thread-0 进入下一阶段，如果没有下一阶段则结束 
Thread-2 进入下一阶段，如果没有下一阶段则结束 
Thread-1 进入下一阶段，如果没有下一阶段则结束 
```
通过查看arriveAndAwaitAdvance()源码可以知道该方法不会抛出异常，如果在等待时希望可中断，或者可超时，可以使用awaitAdvanceInterruptibly方法代替



前面的示例只有一个阶段同步，下面演示3个阶段同步
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Phaser;

public class PhaserTest2 {
	// 指定参与的parties个数
	private static int PAETIES = 5;

	public static void main(String[] args) {
		// 可以在创建时不指定parties,而是在运行时，随时注册和注销新的parties
		final Phaser phaser = new Phaser();
		ExecutorService executor = Executors.newFixedThreadPool(PAETIES);
		for (int i = 0; i < PAETIES; i++) {
			phaser.register();// 每创建一个线程任务，我们就注册一个party
			executor.execute(new Runnable() {
				@Override
				public void run() {
					try {
						int i = 0;
						// 运行三个阶段
						while (i < 3 && !phaser.isTerminated()) {
							System.out.println(Thread.currentThread().getName() + "到达 phase:" + phaser.getPhase());
							Thread.sleep(1000);
							// 等待同一阶段内其他Task到达，才可以进入新的阶段
							phaser.arriveAndAwaitAdvance();
							i++;
						}
					} catch (Exception e) {
						e.printStackTrace();
					} finally {
						phaser.arriveAndDeregister();
					}
				}
			});
		}
		
		executor.shutdown();
	}
}
```
打印结果:
```java
pool-1-thread-1到达 phase:0
pool-1-thread-2到达 phase:0
pool-1-thread-5到达 phase:0
pool-1-thread-3到达 phase:0
pool-1-thread-4到达 phase:0
pool-1-thread-4到达 phase:1
pool-1-thread-3到达 phase:1
pool-1-thread-5到达 phase:1
pool-1-thread-1到达 phase:1
pool-1-thread-2到达 phase:1
pool-1-thread-2到达 phase:2
pool-1-thread-3到达 phase:2
pool-1-thread-1到达 phase:2
pool-1-thread-4到达 phase:2
pool-1-thread-5到达 phase:2
```

在某些外部条件满足时，才真正开始任务的执行
用CountDownLatch能够实现该场景，不过我们现在使用Phaser实现



再演示一个多阶段同步的示例
```java
import java.util.concurrent.Phaser;
import java.util.concurrent.TimeUnit;

public class PhaserTest3 {

	private static int PAETIES = 3;

	public static void main(String[] args) throws InterruptedException {
		final Phaser phaser = new Phaser(PAETIES);
		for (int i = 0; i < PAETIES; i++) {
			TimeUnit.SECONDS.sleep(1);
			Thread t = new Thread(new MyThread(phaser));
			t.start();
		}

	}

	static class MyThread implements Runnable {
		private Phaser phaser;

		public MyThread(Phaser phaser) {
			this.phaser = phaser;
		}

		@Override
		public void run() {
			System.out.printf("当前阶段%s-->当前线程%s\n", phaser.getPhase(), Thread.currentThread().getName());
			phaser.arriveAndAwaitAdvance();// 等待所有参与者完成当前阶段
			System.out.printf("当前阶段%s-->当前线程%s\n", phaser.getPhase(), Thread.currentThread().getName());
			phaser.arriveAndAwaitAdvance();// 等待所有参与者完成当前阶段
			System.out.printf("当前阶段%s-->当前线程%s\n", phaser.getPhase(), Thread.currentThread().getName());
			phaser.arriveAndDeregister();// 注销当前线程

		}

	}
}
```
打印结果
```plain
当前阶段0-->当前线程Thread-0
当前阶段0-->当前线程Thread-1
当前阶段0-->当前线程Thread-2
当前阶段1-->当前线程Thread-2
当前阶段1-->当前线程Thread-0
当前阶段1-->当前线程Thread-1
当前阶段2-->当前线程Thread-1
当前阶段2-->当前线程Thread-0
当前阶段2-->当前线程Thread-2
```

最后举一个使用onAdvance的示例
有3个人争选驸马，公主决定准备3个比赛，来考验这三个人，下面我们就模拟这个这个场景。
这个场景有需要注意的地方，就是第一个比赛，必须要等三个人都完成了得出了结果才能开始下一个比赛
```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.Phaser;

public class PhaserTest4 {
	private static int PARTIES = 3;
	
	public static void main(String[] args) throws InterruptedException {
		final Phaser phaser = new Phaser(PARTIES){
			@Override
			protected boolean onAdvance(int phase, int registeredParties) {
				if(registeredParties == 0){ //参与者为0，则当前phaser可以终止了
					System.out.println("三场比赛已结束！！！");
					return true;
				} else {
					switch(phase){
					case 0:
						System.out.println("第一场比赛结束！");
						return false;
					case 1:
						System.out.println("第二场比赛结束！");
						return false;
					case 2:
						System.out.println("第三场比赛结束！");
						return false;
							
					}
				}
				return super.onAdvance(phase, registeredParties);
			}
		};
		List<Thread> list = new ArrayList<Thread>(PARTIES);
		
		for (int i = 0; i < PARTIES; i++) {
			Thread t = new Thread(new Prince(i+"号白马王子",phaser));
			list.add(t);
			t.start();
		}
		
		for(Thread t : list){
			t.join();
		}
		
		System.out.println("phaser最终状态："+phaser.isTerminated());
	}
	
	static class Prince implements Runnable {
		private String name;
		private Phaser phaser ;
		
		public Prince(String name,Phaser phaser){
			this.name = name;
			this.phaser = phaser;
		}

		@Override
		public void run() {
			match(1);
			match(2);
			match(3);
		}
		
		private void match(int matchNo){
			System.out.printf("%s参加第%d项比赛\n",name,matchNo);
			int workTime = new Random().nextInt(10-3+1)+3;//随机3-10秒，方便看效果
			try {
				Thread.sleep(workTime * 1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.printf("%s完成第%d项比赛，耗时%d秒\n",name,matchNo,workTime);
			
			if(matchNo==3){ //最后一个比赛完成，取消线程
				phaser.arriveAndDeregister();
			} else {
				phaser.arriveAndAwaitAdvance();//等待其他人完成比赛
			}
		}
		
	}
}
```
打印结果如下：
```plain
0号白马王子参加第1项比赛
1号白马王子参加第1项比赛
2号白马王子参加第1项比赛
1号白马王子完成第1项比赛，耗时6秒
2号白马王子完成第1项比赛，耗时8秒
0号白马王子完成第1项比赛，耗时9秒
第一场比赛结束！
2号白马王子参加第2项比赛
1号白马王子参加第2项比赛
0号白马王子参加第2项比赛
0号白马王子完成第2项比赛，耗时5秒
2号白马王子完成第2项比赛，耗时6秒
1号白马王子完成第2项比赛，耗时8秒
第二场比赛结束！
0号白马王子参加第3项比赛
2号白马王子参加第3项比赛
1号白马王子参加第3项比赛
0号白马王子完成第3项比赛，耗时5秒
1号白马王子完成第3项比赛，耗时9秒
2号白马王子完成第3项比赛，耗时10秒
三场比赛已结束！！！
phaser最终状态：true
```




