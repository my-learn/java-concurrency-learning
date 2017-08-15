Condition可以实现等待/通知的功能，但是比wait/notify功能更强大灵活。

一个锁可能关联一个或者多个条件，这些条件通过Condition接口声明。
Condition对象由Lock对象的newCondition()方法创建，调用condition.await（）来实现让线程等待，进入阻塞；调用condition.signal()唤醒线程，注意唤醒线程调用的condition必须是同一个Condition
和wait/notify一样，await（）和signal（）也是在同步代码区内执行。

Condition接口中的几个方法
await()：当线程调用条件的await()方法时，它将自动释放这个条件绑定的锁，其他某个线程才可以获取这个锁并且执行相同的操作，或者执行这个锁保护的另一个临界区代码。
* awat(long time,TimeUnit unit)：
* signal()：当一个线程调用了条件对象的singal()或者signalAll()方法后，一个或者多个在该条件上挂起的线程将被唤醒，但这并不能保证让他们挂起的条件已经满足，所以必须在while循环中调用await()，在条件成立之前不能离开这个循环。如果条件不成立，将再次调用await()。
* signalAll()：
* awatiUninterruptibly()：
* awaitUntil(Date date)：

必须小心使用await()和singal()方法。如果调用了一个条件的await()方法，却从不调用它的singal()方法，这个线程将永久休眠。

