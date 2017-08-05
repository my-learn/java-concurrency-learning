java.lang.ThreadLocal为解决多线程程序的并发问题提供了一种新的思路。ThreadLocal可用来存放线程的局部变量，每个线程都有单独的局部变量，彼此之间不会共享。
它并不是一个Thread，而是一个线程局部变量，也许把他命名为ThreadLocalVariable更合适。



# 方法
ThreadLocal<T>类主要包括以下几个方法：
```java
public T get()  //返回当前线程的局部变量
public void set(T value)
protected T initialValue() //返回当前线程的局部变量的初始值
public void remove()
```

这里只解释一下initialValue()方法，其他几个方法比较容易理解。
initialValue()方法为protected类型，它是为了被子类覆盖而特意提供的，该方法返回当前线程的局部变量的初始值。这个方法是一个延迟调用方法，当线程第一次调用ThreadLocal对象的get()或者set()方法时才执行，并且仅执行一次。在ThreadLocal类本身的实现中，initialValue()方法直接返回一个null，所以一般使用ThreadLocal，最好重写initialValue()方法，否则先调用set()方法会报空指针异常
```java
protected T initialValue() {
	return null;
}
```

# 示例
## 示例一
我们来看一个不使用ThreadLocal，程序运行会有什么问题
```java
public class Test {
    private Long num = 0l;
    //private ThreadLocal<Long> num = new ThreadLocal<Long>();

    public void set(Long val) {
    	num = val;
    	//num.set(val);
    }

    public long getLong() {
    	return num;
        //return num.get();
    }

    public static void main(String[] args) throws InterruptedException {
        final Test test = new Test();

        test.set(Thread.currentThread().getId());
        System.out.println("主线程["+Thread.currentThread().getName()+"] 给num设值,num="+test.getLong());

        Thread thread1 = new Thread(){
            public void run() {
                test.set(Thread.currentThread().getId());
                System.out.println("子线程["+Thread.currentThread().getName()+"] 给num设值,num="+test.getLong());
            };
        };
        thread1.start();
        thread1.join();
        System.out.println("等待线程1执行完...");

        System.out.println("主子线程["+Thread.currentThread().getName()+"] num="+test.getLong());
    }
}
```
打印结果如下：
```plain
主线程[main] 给num设值,num=1
子线程[Thread-0] 给num设值,num=9
等待线程1执行完...
主子线程[main] num=9
```
子线程修改了变量num的值，最后主线程获取到的num值发生改变了，这也说明在多线程下，变量访问是有问题的。
下面我们将变量num改成ThreadLocal，打开以上代码注释即可，运行结果如下：
``` plain 
主线程[main] 给num设值,num=1
子线程[Thread-0] 给num设值,num=9
等待线程1执行完...
主子线程[main] num=1
```
可以看到子线程修改了变量没有影响到main主线程，这也证明了通过ThreadLocal能达到在每个线程中创建变量副本的效果。

## 示例二
多个线程实现各自的计数
```plain 
class Counter {
	private static int count;
	private static ThreadLocal<Integer> serialCount = new ThreadLocal<Integer>() {
		@Override
		protected Integer initialValue() {
			return new Integer(count++);
		}
	};

	public static Integer get() {
		return serialCount.get().intValue();
	}

	public static void set(Integer value) {
		serialCount.set(value);
	}
}

class MyThread extends Thread {
	@Override
	public void run() {
		for (int i = 0; i < 3; i++) {
			int c = Counter.get();
			System.out.println(getName() + ":" + c);
			Counter.set(c += 1);
		}
	}
}

public class ConterTest extends Thread {
	public static void main(String[] args) {
		Thread t1 = new MyThread();
		Thread t2 = new MyThread();

		t1.start();
		t2.start();
	}
}

```

## 示例3
模拟数据库连接
```java
public class ConnectionUtil {
    private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>();
    private static Connection initConn = null; 
	
    static { // static块内的initConn初始化可以放在ThreadLocal的initialValue方法中
        try {
            initConn = DriverManager.getConnection("url, name and password");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    public Connection getConnection() {
        Connection conn = connectionHolder.get();
        if(conn == null){
			connectionHolder.set(initConn);
		} 
        return connectionHolder.get();
    }
    
}

```

## 示例
模拟session获取
```java
public class SessionUtil {
	private static final ThreadLocal<Session> threadSession = new ThreadLocal<Session>();

	public static Session getSession() throws InfrastructureException {
		Session s = threadSession.get();
		try {
			if (s == null) {
				s = getSessionFactory().openSession();
				threadSession.set(s);
			}
		} catch (HibernateException ex) {
			throw new InfrastructureException(ex);
		}
		return s;
	}
}
```

# 原理探究
ThreadLocal类中定义了静态内部类ThreadLocalMap，ThreadLocalMap中又定义了一个静态内部类Entry。
Thead类定义了一个ThreadLocal.ThreadLocalMap类型变量



# 自己实现一个ThreadLocal
既然知道了ThreadLocal的原理，下面我们就自己模拟实现一个ThreadLocal吧
```java
package test;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
public class MyThreadLocal<T> {
	private Map<Runnable,T> values = Collections.synchronizedMap(new HashMap<Runnable,T>());
	
	public T get(){
		Thread curThread = Thread.currentThread();
		T o = values.get(curThread);
		if(o==null && !values.containsKey(curThread)){
			o = initialValue();
			values.put(curThread, o);
		}
		return o;
	}
	
	public void set(T newValue){
		values.put(Thread.currentThread(), newValue);
	}
	
	protected T initialValue() {
		return null;
	}
}
```
可以自己定义一个静态的Map，将当前的Thread作为key，创建的session作为值，put到Map中，应该也行，这也是一般人的想法，但事实上，ThreadLocal的实现刚好相反，它是在每个线程中有个一个Map，而将ThreadLocal实例作为key，这样每个Map中的项数很少。
每个ThreadLocal当然只能放一个对象，要是需要放其他的对象，就再new一个新的ThreadLocal出来，这个新的ThreadLocal作为key，需要放的对象作为value，放到ThreadLocalMap中。



