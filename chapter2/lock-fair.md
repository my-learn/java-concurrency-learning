在`ReentrantLock`类和`ReentrantReadWriteLock`类的构造器中，允许一个名为fair的boolean类型参数，是否使用公平模式。

# 关于公平模式
当有多个线程正在等待一把锁（ReentrantLock或者 ReentrantReadWriteLock），这个锁必须选择它们中间的一个来获得进入临界区，选择任意一个是没有任何标准的。


# 参数详解
* 默认值为 false，表示使用非公平模式。在这个模式下，会选择任意线程一个等待线程
* true表示使用公平模式。在这个模式下，将选择等待时间最长的线程

# 注意点
由于tryLock()方 法并不会使线程进入睡眠，即使Lock接口正在被使用，所以这个公平属性并不会影响它的功能。公平模式可以影响lock()和unlock()方法


#设置方法
```java
private Lock myLock = new ReentrantLock(true);
```

