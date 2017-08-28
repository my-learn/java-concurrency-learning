如果我们只需要某个类里的某个字段，那么就需要使用原子更新字段类。
    
atomic包提供了AtomicLongFieldUpdater、AtomicIntegerFieldUpdater、AtomicReferenceFieldUpdater 3个基于反射的实用工具，可以对指定类的指定 volatile 字段进行原子更新，使用上跟AtomicInteger差不多。

使用上的约束：
* 字段必须是public volatile
* 不能更新父类的字段
* 只能是实例变量，不能是类变量，也就是不能是static修饰
* 不能是final修饰的变量
* 对于AtomicIntegerFieldUpdater 和AtomicLongFieldUpdater 只能修改int/long类型的字段，不能修改其包装类型（Integer/Long）。如果要修改包装类型就需要使用AtomicReferenceFieldUpdater


下面是一个使用示例，** 注意其中的注释，很重要！ **
```java
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;

public class Test {

	private static AtomicIntegerFieldUpdater<User> u = AtomicIntegerFieldUpdater
			.newUpdater(User.class, "age");

	public static void main(String[] args) {
		User zs = new User("张三", 10);
		System.out.println(u.getAndIncrement(zs));//getAndIncrement返回的是旧值
		System.out.println(u.get(zs)); //当前的新值需要使用get获取
	}

	public static class User {
		private String name;
		public volatile int age;

		public User(String name, int age) {
			this.name = name;
			this.age = age;
		}

		public String getName() {
			return name;
		}

		public int getOld() {
			return age;
		}
	}
}
```
打印结果如下：
```plain
10
11

```


