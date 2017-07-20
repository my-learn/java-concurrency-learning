ThreadLocal类中定义了静态内部类ThreadLocalMap，ThreadLocalMap中又定义了一个静态内部类Entry。
Thead类定义了一个ThreadLocal.ThreadLocalMap类型变量

java.lang.ThreadLocal可用来存放线程的局部变量，每个线程都有单独的局部变量，彼此之间不会共享
ThreadLocal<T>类主要包括以下几个方法：
```java
public T get()  //返回当前线程的局部变量
public void set(T value)
protected T initialValue() //返回当前线程的局部变量的初始值
public void remove()
```

这里只解释一下initialValue()方法，其他几个方法比较容易理解。
initialValue()方法为protected类型，它是为了被子类覆盖而特意提供的，该方法返回当前线程的局部变量的初始值。这个方法是一个延迟调用方法，当线程第一次调用ThreadLocal对象的get()或者set()方法时才执行，并且仅执行一次。在ThreadLocal类本身的实现中，initialValue()方法直接返回I一个null。
```java
protected T initialValue() {
	return null;
}
```
我们来看个示例
```java
public class Test {
    ThreadLocal<Long> longLocal = new ThreadLocal<Long>();
    ThreadLocal<String> stringLocal = new ThreadLocal<String>();
 
     
    public void set() {
        longLocal.set(Thread.currentThread().getId());
        stringLocal.set(Thread.currentThread().getName());
    }
     
    public long getLong() {
        return longLocal.get();
    }
     
    public String getString() {
        return stringLocal.get();
    }
     
    public static void main(String[] args) throws InterruptedException {
        final Test test = new Test();
         
         
        test.set();
        System.out.println(test.getLong());
        System.out.println(test.getString());
     
         
        Thread thread1 = new Thread(){
            public void run() {
                test.set();
                System.out.println(test.getLong());
                System.out.println(test.getString());
            };
        };
        thread1.start();
        thread1.join();
         
        System.out.println(test.getLong());
        System.out.println(test.getString());
    }
}
```
打印结果如下：
```plain
1
main
9
Thread-0
1
main
```
子线程修改了值，没有影响到main主线程。这也证明了通过ThreadLocal能达到在每个线程中创建变量副本的效果


一般使用ThreadLocal，最好重写initialValue()方法，否则先调用set()方法会报空指针异常
比如上例中，在定义ThreadLocal变量时可以改成
```java
ThreadLocal<Long> longLocal = new ThreadLocal<Long>(){
	protected Long initialValue() {
		return Thread.currentThread().getId();
	};
};
ThreadLocal<String> stringLocal = new ThreadLocal<String>(){;
	protected String initialValue() {
		return Thread.currentThread().getName();
	};
};
```

再来举几个示例
示例1
```java
package test;
class Counter {
	private static int count;
	private static ThreadLocal<Integer> serialCount = new ThreadLocal<Integer>() {
		@Override
		protected Integer initialValue() {
			return new Integer(count++);
		};
	};
	public static Integer get() {
		return serialCount.get().intValue();
	};
	public static void set(Integer value) {
		serialCount.set(Integer.valueOf(value));
	};
}
public class LocalTester extends Thread {
	 @Override
	public void run() {
		for (int i = 0; i < 3; i++) {
			int c= Counter.get();
			System.out.println(getName()+":"+c);
			Counter.set(c+=1);
		}
	}
	 
	 public static void main(String[] args) {
		Thread t1 = new LocalTester();
		Thread t2 = new LocalTester();
		
		t1.start();
		t2.start();
	}
}
```
打印结果如下
```plain
Thread-0:0
Thread-1:1
Thread-1:2
Thread-1:3
Thread-0:1
Thread-0:2
```

示例2
```java
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>() {
	public Connection initialValue() {
		return DriverManager.getConnection(DB_URL);
	}
};
 
public static Connection getConnection() {
	return connectionHolder.get();
}
```

示例3
```java
private static final ThreadLocal threadSession = new ThreadLocal();
 
public static Session getSession() throws InfrastructureException {
    Session s = (Session) threadSession.get();
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
```

自己实现一个ThreadLocal
我们来模拟实现一个ThreadLocal
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



