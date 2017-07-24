使用isInterrupted方法和interrupted方法
继承了Thread类后，就有了isInterrupted()方法，可以用来检查线程是否已被中断，
Thread类静态方法interrupted方法，用来检查** 当前执行的线程 **是否被中断。
isInterrupted()和interrupted()方法有一个很大的区别。isInterrupted()不能改变interrupted属性的值，但是后者能设置interrupted属性为false。因为interrupted()是一个静态方法，更推荐使用isInterrupted()方法。
当你主动调用了线程实例的interrupt()方法， 就可以用isInterrupted()捕获到。

还有一种方法控制线程中断，利用InterruptedException异常
这对于线程实现了复制的算法并且分布在几个方法中，控制线程中断更方便。
1.由Thread实例发起interrupt()中断。
2.
```java
2..
if（Thread.interrupted()）{
    throw new InterruptedException();
}
```
3.
```java
try {
    ...
} catch(InterruptedException e) {
 ...
}
```

