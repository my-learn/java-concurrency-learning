java.lang.ThreadLocal可用来存放线程的局部变量，每个线程都有单独的局部变量，彼此之间不会共享。
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
ThreadLocal实现的核心原理就是将变量添加到Thread的一个变量中存储。下面看具体实现
![](/chapter1/110.png)
我们看到ThreadLocal的类图（注：ThreadLocalMap是ThreadLocal的内部类，Entry是ThreadLocalMap的内部类。）
Thread有一个类型为ThreadLocalMap的变量`threadLocals`，用于存储该Thread拥有的用户变量，变量具体怎么存储交给了ThreadLocalMap，我们再看ThreadLocalMap，它有一个类型为Entry的数组变量table，用于存储用户变量，这里是数组，自然是可以存储多个用户变量了。一个Entry带代表了一个变量
一个用户变量又怎么存储的呢？我们还要再看看Entry，看他的构造函数就能一目了然
```java
static class Entry extends WeakReference<ThreadLocal> {
	/** The value associated with this ThreadLocal. */
	Object value;

	Entry(ThreadLocal k, Object v) {
		super(k);
		value = v;
	}
}
```
key类型为ThreadLocal，值就是用户设置的值。

再看看代码
最基本的使用
```java
private ThreadLocal<Long> num = new ThreadLocal<Long>();
num.set(123456l);
```
我们直接从set方法入手
```java
public void set(T value) {
	Thread t = Thread.currentThread();
	ThreadLocalMap map = getMap(t);
	if (map != null)
		map.set(this, value);
	else
		createMap(t, value);
}
```
这里顺便给出getMap(t)代码
```java
ThreadLocalMap getMap(Thread t) {
	return t.threadLocals;
}
```
set方法的逻辑：当前Thread的threadLocals变量是否为null？，假设不为null，进入if,看到ThreadLocalMap#set方法
```java
private void set(ThreadLocal key, Object value) {

	// We don't use a fast path as with get() because it is at
	// least as common to use set() to create new entries as
	// it is to replace existing ones, in which case, a fast
	// path would fail more often than not.

	Entry[] tab = table;
	int len = tab.length;
	int i = key.threadLocalHashCode & (len-1);

	for (Entry e = tab[i];
		 e != null;
		 e = tab[i = nextIndex(i, len)]) {
		ThreadLocal k = e.get();

		if (k == key) {
			e.value = value;
			return;
		}

		if (k == null) {
			replaceStaleEntry(key, value, i);
			return;
		}
	}

	tab[i] = new Entry(key, value);
	int sz = ++size;
	if (!cleanSomeSlots(i, sz) && sz >= threshold)
		rehash();
}
```
还是比较容易看得懂的

如果想更直观的看到数据是如何存储的，打个断点看看就能明白
为了方便看到Thread的变量threadLocals的值，加如下两行代码，断点打在System.out.println这行上
```java
public void set(Long val) {
//    	num = val;
	num.set(val);
	num2.set(333l);
	Thread tTmp = Thread.currentThread();
	System.out.println("a");
}
```
查看tTmp的threadLocals的值
![](/chapter1/1110_2.png)



# 自己实现一个ThreadLocal
ThreadLocal的设计有点特别，它是在每个线程中有个一个Map，而将ThreadLocal实例作为key。而一般人想到的方式可能会是定义一个静态的Map，将当前的Thread作为key,put到Map中。ThreadLocal的这种设计可以使每个Map中的项数很少。
下面我们就自己模拟用静态Map实现一个简单的ThreadLocal吧
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




