
`Object`中的锁

每个Object中，都存在两个池：
* 锁池(monitor set)：假设某个线程T正在持有某个对象的锁，其他线程想要执行这个对象的同步语句(synchronized)，亦即获取该对象的锁，则这个新的线程会进入锁池中，进行抢锁。
* 等待池(wait set)：

`wait(timeout)`
导致当前线程等待，直到另外一个线程调用该对象的`notify()`或`notifyAll()`或达到一个指定的超时时间。

notify()