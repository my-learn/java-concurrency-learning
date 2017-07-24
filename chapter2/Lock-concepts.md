http://www.cnblogs.com/dolphin0520/p/3923167.html

Java中的ReentrantLock和synchronized两种锁定机制的对比
http://blog.csdn.net/fw0124/article/details/6672522
或
http://www.ibm.com/developerworks/cn/java/j-jtp10264/index.html

可重入
当某线程请求一个由其他线程持有的锁时，发出请求的线程就会阻塞。然而，由于内置锁(synchronized)是可重入的，因此如果某个线程试图获得一个已经由它自己持有的锁时，那么这个请求就会成功。synchronized同步块对同一条线程来说是可重入的，不会出现把自己锁死的问题。