Thread提供了几个跟中断相关的方法
* void interrupt():用于中断线程。调用该方法的线程的状态为将被置为"中断"状态，但不会停止线程，一旦线程被置为“中断状态”，就会抛出interruptedException中断异常。需要用户自己做中断处理，也就是处理interruptedException这个异常
* boolean isInterrupted()、isInterrupted(boolean)：
* static boolean interrupted：这是一个静态方法，该方法会将中断标示位清除，所以不要用该方法判断线程是否已经中断，而是用isInterrupted方法

interrupt跟interrupted区别在于interrupted用来判断当前线是否被中断，而isInterrupted可以用来判断其他线程是否被中断，所以
`while(！isInterrupted())`也可以换成`while(！Thread.interrupted())`

isInterrupted()和interrupted()方法有一个很大的区别。isInterrupted()不能改变interrupted属性的值，但是后者能设置interrupted属性为false。因为interrupted()是一个静态方法，更推荐使用isInterrupted()方法。
当你主动调用了线程实例的interrupt()方法， 就可以用isInterrupted()捕获到。

# 为什么stop方法被废弃
参考[Why Are Thread.stop, Thread.suspend, Thread.resume and Runtime.runFinalizersOnExit Deprecated?](http://docs.oracle.com/javase/1.5.0/docs/guide/misc/threadPrimitiveDeprecation.html)


响应中断
interrupt只会对那些能够支持响应中断的代码才起作用，比如Thread.sleep或BlockingQueue.put等


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

