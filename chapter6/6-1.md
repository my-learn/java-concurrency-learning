TimeUnit是java.util.concurrent包下面的一个类,它表示给定单元粒度的时间段，并提供在这些单元中进行跨单元转换、执行计时、延迟操作功能。

常用的颗粒度
```java
TimeUnit.DAYS          //天
TimeUnit.HOURS         //小时
TimeUnit.MINUTES       //分钟
TimeUnit.SECONDS       //秒
TimeUnit.MILLISECONDS  //毫秒
```

时间颗粒度转换 
```java
public long toMillis(long d)    //转化成毫秒
public long toSeconds(long d)  //转化成秒
public long toMinutes(long d)  //转化成分钟
public long toHours(long d)    //转化成小时
public long toDays(long d)     //转化天
```

例子
```java
TimeUnit.SECONDS.sleep(5)  线程休眠5秒 
TimeUnit.SECONDS.toMillis(1)     1秒转换为毫秒数  
TimeUnit.SECONDS.toMinutes(60)   60秒转换为分钟数  
TimeUnit.SECONDS.convert(1, TimeUnit.MINUTES) 1分钟转换为秒数 
TimeUnit.DAYS.toHours(1) 将1天转化为小时
TimeUnit.HOURS.toSeconds(1)  1小时有3600秒
TimeUnit.HOURS.convert(3,TimeUnit.DAYS) 把3天转化成小时
```

用的比较多的还是 `TimeUnit.SECONDS.sleep(5) `,以前都是用Thread.sleep(5)实现的，不过并没有本质的区别，因为TimeUnit中的sleep就是调用的Thread.sleep方法，TimeUnit中sleep源码如下：
```java
public void sleep(long timeout) throws InterruptedException {
	if (timeout > 0) {
		long ms = toMillis(timeout);
		int ns = excessNanos(timeout, ms);
		Thread.sleep(ms, ns);
	}
}
```

