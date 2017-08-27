我们知道，java中早期有一些线程安全的容器，比如Vector、Hashtable，这些同步容器在所有方法上都加了synchronized
那么，他们真的是线程安全的吗？

其实提出该疑问，就是在于对这个线程安全存在一些错误认识导致。
我们先看一段代码
```java
package test;
import java.util.Vector;
public class Test {
    static Vector<Integer> vector = new Vector<Integer>();
    public static void main(String[] args) throws InterruptedException {
        while(true) {
            for(int i=0;i<10;i++) {
            	vector.add(i);	
            }
            
            Thread thread1 = new Thread(){
                public void run() {
                    for(int i=0;i<vector.size();i++) {
                        vector.remove(i);
                    }
                };
            };
            
            Thread thread2 = new Thread(){
                public void run() {
                    for(int i=0;i<vector.size();i++){
                    	vector.get(i);
                    }
                        
                };
            };
            
            thread1.start();
            thread2.start();
            
        }
    }
}
```
运行的结果如下：
```plain
Exception in thread "Thread-8069" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 15
    at java.util.Vector.get(Unknown Source)
    at test.Test$2.run(Test.java:25)
Exception in thread "Thread-8837" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 20
    at java.util.Vector.get(Unknown Source)
    at test.Test$2.run(Test.java:25)
Exception in thread "Thread-8889" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 15
    at java.util.Vector.get(Unknown Source)
    at test.Test$2.run(Test.java:25)
```

我们分析下原因
运行时，程序可能会出现这种情况
当某个线程在某个时刻执行这句时：
```java
for(int i=0;i<vector.size();i++)
    vector.get(i);
```
假若此时vector的size方法返回的是10，i的值为9
然后另外一个线程执行了这句：
```java
for(int i=0;i<vector.size();i++)
    vector.remove(i);
```
将下标为9的元素删除了。
那么通过get方法访问下标为9的元素肯定就会出问题了。

归结原因
Vector的size方法和get方法在一个方法内同时调用时不是同步的，所以，修改起来就简单了
```java
public void run() {
	synchronized (Test.class) {   //进行额外的同步
		for(int i=0;i<vector.size();i++)
			vector.remove(i);
	}
};
```
另一个 地方vector.get(i);调用的地方也改成一样即可。

** 总结 **
java提供的线程安全容器，比如Vector、Collections.synchronizedMap、Collections.synchronizedList，他们的线程安全是有前提条件的，即：所有单个的操作都是线程安全的，但是多个操作组成的操作序列不能保证数据安全，因为在操作序列中控制流取决于前面操作的结果。
例如对于这样的功能put-if-absent语句块――如果一个条目不在Map 中，那么添加这个条目。该功能通过containsKey() 方法和 put() 方法组合起来。要保证原子性操作，就需要对该语句块加上同步锁。




