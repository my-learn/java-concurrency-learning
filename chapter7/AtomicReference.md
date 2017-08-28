AtomicReference 可以实现对象引用的原子更新操作。

下面看示例代码：
```java
import java.util.concurrent.atomic.AtomicReference;

public class ARTest {

	public static void main(String[] args) {

		Person p1 = new Person(1);
		Person p2 = new Person(2);

		// 新建AtomicReference对象，初始化它的值为p1对象
		AtomicReference<Person> ar = new AtomicReference<Person>(p1);
		// 通过CAS设置ar。如果ar的值为p1的话，则将其设置为p2。
		ar.compareAndSet(p1, p2);

		Person p3 = ar.get();
		System.out.println("p3 id=" + p3.getId()); //打印 id=2
	}
}

class Person {
	volatile long id;

	public Person(long id) {
		this.id = id;
	}

	public long getId() {
		return id;
	}

	public void setId(long id) {
		this.id = id;
	}

}
```

使用上跟AtomicInteger使用上没有太多的区别


下面演示一下使用AtomicReference实现一个线程安全的堆栈
```java
import java.util.Stack;
import java.util.concurrent.atomic.AtomicReference;

/**
 * 使用AtomicReference实现一个线程安全的堆栈
 * @author Administrator
 *
 * @param <T>
 */
public class ConcurrentStack<T> extends Stack<T>{
	
	private AtomicReference<Node<T>> stacks = new AtomicReference<Node<T>>();

	@Override
	public T push(T e) {
		Node<T> oldNode, newNode;
		for (;;) {
			oldNode = stacks.get();
			newNode = new Node<T>(e, oldNode);
			if (stacks.compareAndSet(oldNode, newNode)) {
				return e;
			}
		}
	}
	
	@Override
	public T pop() {
		Node<T> oldNode, newNode;
		for (;;) {
			oldNode = stacks.get();
			newNode = oldNode.next;
			if (stacks.compareAndSet(oldNode, newNode)) {
				return oldNode.object;
			}
		}
	}

	private static final class Node<T> {
		private T object;
		private Node<T> next;

		private Node(T object, Node<T> next) {
			this.object = object;
			this.next = next;
		}
	}
}
```
其实这个例子然并暖，因为已经有现成的功能了，使用LinkedBlockingDeque就可以了
