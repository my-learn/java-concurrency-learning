java.util.concurrent.atomic原子操作类包提供了一组原子变量类，用来实现单个变量的原子操作

原子类其内部实现不是简单的使用synchronized，而是基于Unsafe实现的包装类，核心操作是CAS原子操作，避免了synchronized的高额开销，执行效率大为提升。
（CAS操作：即compare and swap，指的是将预期值与当前变量的值比较(compare)，如果相等则使用新值替换(swap)当前变量，否则不作操作。CAS是基于硬件实现的锁机制，当然现在的CPU都支持CAS操作）


摘取一段AtomicInteger的源码
```java
public final boolean compareAndSet(int i, int expect, int update) {
	return compareAndSetRaw(checkedByteOffset(i), expect, update);
}

private boolean compareAndSetRaw(long offset, int expect, int update) {
	return unsafe.compareAndSwapInt(array, offset, expect, update);
}
```



java.util.concurrent.atomic包中共有12个原子类，可以分成4组
* 标量类（Scalar）：AtomicBoolean，AtomicInteger，AtomicLong，AtomicReference
* 数组类：AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray
* 更新器类：AtomicLongFieldUpdater，AtomicIntegerFieldUpdater，AtomicReferenceFieldUpdater
* 复合变量类：AtomicMarkableReference，AtomicStampedReference，AtomicStampedReference(解决CAS的ABA问题)

原子类通用方法
* set( )和get( )方法：可以原子地设定和获取atomic的数据。类似于volatile，保证数据会在主存中设置或读取
* lazySet()：延时设置变量值
* getAndSet( )方法 ：虽然get()和set()都是原子操作，但是放在一起执行的时候就是两个操作，并不是原子操作。该方法本质就是get()和set()合并后的原子操作
* compareAndSet( ) 和weakCompareAndSet( )方法：这2个方法接受2个参数，一个是期望数据(expected)，一个是新数据(new)；如果atomic里面的数据和期望数据一 致，则将新数据设定给atomic的数据，返回true，表明成功；否则就不设定，并返回false。




 