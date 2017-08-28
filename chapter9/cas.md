CAS是并发库无锁设计的的核心基础，在并发库中的类中随处可见，了解CAS，才能很好的理解它的无锁设计。

CAS(Compare And Swap)是一种硬件上对并发的支持，是处理器中提供的一种特殊指令，用于管理对共享数据的并发访问。


CAS是一种无的非阻塞算法的实现，它包含了3个操作数：
* 需要读取的内存只值V
* 进行比较的值A
* 待写入的新值B

当且仅当V的值等于A时，CAS将通过用新值B来更新V的值，否则不会执行任何操作。
从设计思想上来看，synchronized是一种悲观锁，即我要用我先占用，别人不准动，即使我现在可能还没正式使用；而CAS则是一种乐观锁，用的时候去问下有没有人在用啊？没有，那好，我来用。

CAS 算法是 Unsafe 的 native 实现的，例如AtomicInteger中的compareAndSet
```java
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```


不过CAS有一个问题，即无法解决ABA的问题，但是并发库通过代码手段解决了。说明并发库还是相当厉害的。
解决ABA问题 通常通过版本号解决，每次修改版本号加1；比较时候 如果版本号一直则可以修改，否则不能修改！

CAS并不是无阻塞，至少在硬件上做了处理，而不是阻塞在语言层面上。虽然CAS很高效，但并非所有需要线程安全的地方都要用CAS，CAS只适合一些粒度比较小的场景，比如计算器，状态值等，毕竟锁机制的存在也不是空谈的。

