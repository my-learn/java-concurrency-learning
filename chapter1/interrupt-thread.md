Thread提供了interrupt静态方法来中断当前执行的线程。但是单独调用interrupt方法不能中断正在运行过程中的线程，只能中断阻塞过程中的线程，使其抛出一个异常，也就说，它可以用来中断一个正处于阻塞状态的线程。
通过isInterrupted()方法检查线程是否已被中断，
Thread类静态方法interrupted方法，用来检查** 当前执行的线程 **是否被中断。

isInterrupted()和interrupted()方法有一个很大的区别。isInterrupted()不能改变interrupted属性的值，但是后者能设置interrupted属性为false。因为interrupted()是一个静态方法，更推荐使用isInterrupted()方法。
当你主动调用了线程实例的interrupt()方法， 就可以用isInterrupted()捕获到。

还有一种方法控制线程中断，利用InterruptedException异常
这对于线程实现了复制的算法并且分布在几个方法中，控制线程中断更方便。
由Thread实例发起interrupt()中断。
```java
if（Thread.interrupted()）{
    throw new InterruptedException();
}
```

```java
try {
    ...
} catch(InterruptedException e) {
 ...
}
```

