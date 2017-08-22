AtomicInteger提供的方法除了前面一节介绍的原子类通用方法，还提供了一些加法，减法原子操作，AtomicLong也是一样的
getAndIncrement( )：以原子方式将当前值加 1，相当于线程安全的i++操作。
incrementAndGet( )：以原子方式将当前值加 1， 相当于线程安全的++i操作。
getAndDecrement( )：以原子方式将当前值减 1， 相当于线程安全的i--操作。
decrementAndGet ( )：以原子方式将当前值减 1，相当于线程安全的--i操作。
addAndGet( )： 以原子方式将给定值与当前值相加， 实际上就是等于线程安全的i =i+delta操作。
getAndAdd( )：以原子方式将给定值与当前值相加， 相当于线程安全的t=i;i+=delta;return t;操作。
（注:i++并不是原子操作！）

下面我们演示一个并发存钱取钱的示例，通过线程安全的和非线程安全的对比来说明原子类对程序开发带来的便利性

1.先定义账号系统
```java
/**
 * 账号系统
 * 
 * @author Administrator
 *
 */
public interface Account {

	public long getBalance();

	public void setBalance(long balance);

	public void addAmount(long amount);

	public void substractAmount(long amount);

}
```
2.再定以两个实现类，分别是非线程安全和线程安全的
```java
/**
 * 非线程安全的账号系统
 * @author Administrator
 *
 */
public class UnSafetyAccount implements Account {
	
	//账户余额
	private long balance;
	
	public long getBalance() {
		return this.balance;
	}

	public void setBalance(long balance) {
		this.balance = balance;
	}
	
	public void addAmount(long amount){
		this.balance += amount;
	}
	
	public void substractAmount(long amount){
		this.balance -= amount;
	}
}
```
第二个实现类
```java
/**
 * 线程安全的账号系统
 * @author Administrator
 *
 */
public class SafetyAccount implements Account {
	
	//账户余额
	private AtomicLong balance;
	
	public SafetyAccount (){
		balance = new AtomicLong();
	}

	public long getBalance() {
		return this.balance.get();
	}

	public void setBalance(long balance) {
		this.balance.set(balance);;
	}
	
	public void addAmount(long amount){
		this.balance.getAndAdd(amount);
	}
	
	public void substractAmount(long amount){
		this.balance.getAndAdd(-amount);
	}
}
```
3.在定义Bank存钱，ATM取钱两个线程
```java
/**
 * 模拟存钱
 * @author Administrator
 *
 */
public class Bank implements Runnable{
	
	private Account account;
	
	public Bank(Account account){
		this.account = account;
	}

	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			account.addAmount(100);
		}
		
	}

}
```
```java
/**
 * 模拟取钱
 * @author Administrator
 *
 */
public class ATM implements Runnable {
	private Account account;

	public ATM(Account account){
		this.account = account;
	}

	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			account.substractAmount(100);
		}

	}
}
```
4.最后就是测试类了
为了让程序更容易出现错误，我将存钱取钱线程数设为10000，同时加入了yield，提高cpu切换的频率，更容易产生并发
```java
public class Test {

	public static void main(String[] args) throws InterruptedException, ExecutionException {
		Account account = new UnSafetyAccount();
		account.setBalance(10000);
		
		List<Thread> tlist = new ArrayList<Thread>();
		
		for (int i = 0; i < 10000; i++) {
			Bank bank = new Bank(account);
			Thread t1 = new Thread(bank);
			tlist.add(t1);
			t1.start();
			t1.yield();
		}
		
		for (int i = 0; i < 10000; i++) {
			ATM atm = new ATM(account);
			Thread t2 = new Thread(atm);
			tlist.add(t2);
			t2.start();
			t2.yield();
		}
		
		for (int i = 0; i < tlist.size(); i++) {
			tlist.get(i).join();
		}
		
		System.out.println("余额:"+account.getBalance());
	}

}
```
这里用的是UnSafetyAccount，即非线程安全的，尝试多执行几次你会发现出现最后的账号余额不是10000，理论来说存钱取钱次数是一样的，余额应该是10000才对。
将Account account = new UnSafetyAccount();
改成Account account = new SafetyAccount();
余额则一直都是10000，这才是正确的。

由此例可以看出AtomicInteger原子类在编写并发程序带来的便利性。如果不用AtomicInteger，那只能synchronized做同步了。




