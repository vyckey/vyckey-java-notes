
# Object中的内置锁

每个 `Object` 中，都存在两个池：
* **锁池(monitor set)**：假设某个线程T正在持有某个对象的锁，其他线程想要执行这个对象的同步语句( `synchronized` )，亦即获取该对象的锁，则这个新的线程会进入锁池中，进行抢锁。
* **等待池(wait set)**：假设一个线程T调用了某个对象的 `wait()` 方法，线程T就会释放该对象的锁，同时线程T就进入到了该对象的等待池中。

```java
public class Object {
    public final native void notify();

    public final native void notifyAll();

    public final native void wait(long timeout) throws InterruptedException;

    public final void wait(long timeout, int nanos) throws InterruptedException {
        // ...
    }

    public final void wait() throws InterruptedException {
        wait(0);
    }
}
```

## wait()方法

导致**当前线程等待**，直到另外一个线程调用该对象的 `notify()` 或 `notifyAll()` 或达到一个指定的超时时间。

当前线程必须拥有该对象的监视器。

此方法导致当前线程（称为 T）将自身置于此对象的等待集中，然后放弃对此对象的任何和所有同步声明。 线程 T 出于线程调度目的而被禁用并处于休眠状态，直到发生以下四种情况之一：

1. **某个其他线程调用了该对象的 `notify()` 方法，而线程 T 恰好被任意选择为要被唤醒的线程。**
2. **其他某个线程为此对象调用 `notifyAll()` 方法。**
3. **一些其他线程中断线程 T。**
4. **指定的实时时间或多或少已经过去。但是，如果超时为零，则不会考虑实时，线程只是等待直到收到通知。**

然后从该对象的等待集中删除线程 T，并重新启用线程调度。 然后它以通常的方式与其他线程竞争在对象上同步的权利； 一旦它获得了对象的控制权，它对该对象的所有同步声明都将恢复到之前的状态——即，恢复到调用 `wait()` 方法时的状态。 线程 T 然后从 `wait()` 方法的调用返回。 因此，从 `wait()` 方法返回时，对象和线程 T 的同步状态与调用 `wait()` 方法时的状态完全相同。

线程也可以在没有被通知、中断或超时的情况下被唤醒，这就是所谓的**虚假唤醒**。 虽然这在实践中很少发生，但应用程序必须通过测试应该导致线程被唤醒的条件来防止它，如果条件不满足则继续等待。 换句话说，等待应该总是在循环中发生，就像这样：

```java
synchronized (obj) {
    while (<condition does not hold>)
        obj.wait(timeout);
    ... // Perform action appropriate to condition
}
```

## notify()方法

**唤醒**在此对象的监视器上**等待的单个线程**。 如果有任何线程正在等待该对象，则选择唤醒其中一个线程。**选择是任意的**，由实现自行决定。 线程通过调用其中一种等待方法在对象的监视器上等待。

在当前线程放弃对该对象的锁定之前，被唤醒的线程将无法继续。被唤醒的线程将以通常的方式与可能正在积极竞争同步该对象的任何其他线程进行竞争。`notify` 后，当前线程不会马上释放该对象锁，`wait` 所在的线程并不能马上获取该对象锁，要等到程序退出 `synchronized` 代码块后，当前线程才会释放锁，`wait` 所在的线程也才可以获取该对象锁。

此方法只能由作为此对象监视器所有者的线程调用。 线程通过以下三种方式之一成为对象监视器的所有者：

1. 通过执行该对象的同步实例方法 `synchronized(this){}`。
2. 通过执行在对象上同步的同步语句的主体 ``。
3. 对于类类型的对象，通过执行该类的同步静态方法 `static synchronized(T.class)`。

## notifyAll()方法

唤醒在此对象的监视器上等待的所有线程。 线程通过调用其中一种等待方法在对象的监视器上等待。

在当前线程放弃对该对象的锁定之前，被唤醒的线程将无法继续。 被唤醒的线程将以通常的方式与可能正在积极竞争同步该对象的任何其他线程进行竞争。此方法只能由作为此对象监视器所有者的线程调用。

`notifyAll()` 使所有原来在该对象上 `wait()` 的线程统统退出 `wait()` 的状态（即全部被唤醒，不再等待 `notify()` 或 `notifyAll()`，但由于此时还没有获取到该对象锁，因此还不能继续往下执行），变成等待获取该对象上的锁，一旦该对象锁被释放（`notifyAll` 线程退出调用了 `notifyAll()` 的 `synchronized` 代码块的时候），他们就会去竞争。如果其中一个线程获得了该对象锁，它就会继续往下执行，在它退出 `synchronized` 代码块，释放锁后，其他的已经被唤醒的线程将会继续竞争获取该锁，一直进行下去，直到所有被唤醒的线程都执行完毕。

## wait()、notify()、notifyAll()为什么必须放在synchronized中？

JVM 在运行时会强制检查 `wait()` 和 `notify()` 有没有在 `synchronized` 代码中，如果没有的话就会报非法监视器状态异 `IllegalMonitorStateException` 。根本原因就是为了防止多线程并发运行时，程序的执行混乱问题。

假设 `wait()` 和 `notify()` 可以不加锁，我们用它们来实现一个自定义阻塞队列。 这里的阻塞队列是指读操作阻塞，也就是当读取数据时，如果有数据就返回数据，如果没有数据则阻塞等待数据。代码示例如下：

```java
public class MyBlockingQueue {
    private Queue<String> queue = new LinkedList<>();

    public void put(String data) {
        queue.add(data); 
        // 唤醒线程继续执行（这里的线程指的是执行 take 方法的线程）
        notify(); // ③
    }

    /**
     * 如果队列里面有数据则返回数据，如果没有数据就阻塞等待数据（阻塞式执行）
     */
    public String take() throws InterruptedException {
        // 使用 while 判断是否有数据（这里使用 while 而非 if 是为了防止虚假唤醒）
        while (queue.isEmpty()) { // ①  
            // 没有任务，先阻塞等待
            wait(); // ②
        }
        return queue.remove(); // 返回数据
    }
}
```

| 步骤 | 线程1 | 线程2 |
| --- | --- | --- |
| 1	| 执行步骤 ① 判断当前队列中没有数据	| |
| 2 | | 执行步骤 ③ 将数据添加到队列，并唤醒线程1继续执行 |
| 3 | 执行步骤 ② 线程 1 进入休眠状态 | |

如果不添加 `synchronized`，按照上面的步骤，线程 1 执行完判断之后，尚未执行休眠之前，此时另一个线程添加数据到队列中。然而这时线程 1 已经执行过判断了，所以就会直接进入休眠状态，从而导致队列中的那条数据永久性不能被读取。而添加了 `synchronized` 则不会出现这种问题。



Unsafe

ReentrantLock

Semaphore

ReentrantReadWriteLock

SynchronousQueue


# 参考资料

* [JavaGuide - AQS详解](https://javaguide.cn/java/concurrent/aqs.html)
* [掘金 - 为什么wait和notify必须放在synchronized中？](https://juejin.cn/post/7067322092936495112)