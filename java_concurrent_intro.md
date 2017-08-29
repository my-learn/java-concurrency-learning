Java从jdk 1.5开始，提供了java.util.concurrent并发包，它提供了许多线程安全、高性能的并发构建块，简化了多线程变成，使开发者可以很容易的开发出线程安全、可伸缩性、高性能、可读性和可靠性的程序。

java.util.concurrent包的类都来自于[JSR-166](https://www.jcp.org/en/jsr/detail?id=166),
JSR 166: Concurrency Utilities 官方的描述“The JSR proposes a set of medium-level utilities that provide functionality commonly needed in concurrent programs. ”。
作者是大名鼎鼎的Doug Lea，这个包的前身可以在[这里](http://gee.cs.oswego.edu/dl/classes/EDU/oswego/cs/dl/util/concurrent/intro.html)找到,
API手册：http://gee.cs.oswego.edu/dl/jsr166/dist/docs/
[Doug Lea维护的blog](http://g.oswego.edu/dl/concurrency-interest/)

先上图Doug Lea大神，来膜拜一下

![](/Doug_Lea.jpg)


并发库结构图（来自网络）
![](/structure-1.png)
![](/structure-2.jpg)


