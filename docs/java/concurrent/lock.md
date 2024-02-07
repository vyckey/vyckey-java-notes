---
title: Lock
tags: [java, concurrent, lock]
sidebar_label: Lock
sidebar_position: 1
---

# 锁有什么用？

在多线程或多进程中访问共享资源，如果没有锁会导致程序出现不可预期的问题，数据不一致，并发死锁等问题。

-----

# 乐观锁和悲观锁

**乐观锁**和**悲观锁**是并发控制中的两种不同的策略。

* **乐观锁**：假设并发冲突的概率很小，因此在读取数据时不加锁，只有在更新数据时才进行锁定。在更新数据时，先读取数据，判断数据是否被其他线程修改过，如果没有则进行更新，否则放弃更新并进行重试。**乐观锁适用于读多写少的场景，可以提高并发性能。**
* **悲观锁**：假设并发冲突的概率很大，因此在读取数据时就进行锁定，防止其他线程修改数据。在更新数据时，先进行锁定，然后进行更新，更新完成后释放锁。**悲观锁适用于写多读少的场景，可以保证数据的一致性。**

## 乐观锁的实现方法

### 版本号

版本号实现乐观锁的要点：当版本相等或大于时更新。

```sql
update xxx_table(`version`, ...) set (`version`+1, ...) where `version` >= ${some_version}..
```

假设有一个用户表，其中包含用户的基本信息和版本号字段。当用户信息被修改时，版本号会自动加1。在更新用户信息时，使用版本号实现乐观锁可以避免并发修改导致数据不一致的问题。

例如，假设用户A的信息如下：

| id   | name | age  | version |
| ---- | ---- | ---- | ------- |
| 1    | Tom  | 20   | 1       |

此时，用户B也想修改用户A的信息，并且同时发起了更新请求，此时用户A和用户B的信息如下：

| id   | name | age  | version | success |
| ---- | ---- | ---- | ------- | --- |
| 1    | Tom  | 20   | 2       | true |
| 1    | Tom  | 20   | 1       | false |

假设用户A的信息被先更新了，此时版本号变成了2，用户B的更新请求会失败，因为版本号不一致。此时用户B需要重新获取最新的版本号，并且重新发起更新请求。

### CAS算法

CAS（Compare and Swap）算法是一种并发控制算法，用于解决多线程并发访问共享资源时的数据一致性问题。它通过比较共享变量的值和期望值是否相等来判断是否有其他线程已经修改了共享变量的值，如果相等，则将共享变量的值修改为新值，否则不做任何操作。CAS算法是一种乐观锁，它不需要像悲观锁一样在每次访问共享资源时都进行加锁和解锁操作，因此可以提高并发性能。CAS算法常用于实现无锁数据结构和并发控制。

```
CAS(V,E,N) // 其中V：需要操作的共享变量，E：预期值，N：新值
```

当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新 V 的值，否则不会执行任何操作（比较和替换是一个原子操作）。一般情况下是一个自旋操作，即不断的重试。一般来说，会有CPU的原子指令来支持CAS操作。

#### 什么是ABA问题？

ABA问题是指当一个线程A读取该变量的值，然后另一个线程B修改了该变量的值并将其改回原来的值，最后线程A再次读取该变量的值时，它会发现该变量的值没有发生变化，因此认为没有其他线程修改过该变量，而实际上变量的值已经被修改了。这种情况称为ABA问题，因为变量的值从A变成了B再变回A。ABA问题可能会导致程序出现错误，例如在CAS（Compare and Swap）操作中，它可能会误判变量的值没有发生变化，从而造成线程安全问题。为了解决ABA问题，可以采用版本号等机制来避免。

#### 自旋锁详解

自旋是一种如果出现线程竞争，但是此处的逻辑线程执行起来非常快，某些没有获取到资源的线程也能在不久后的将来获取到资源并执行时，OS让没有获取到资源的线程执行几个**空循环**的操作等待资源的情况。而自旋这种操作也的确可以提升效率，因为自旋锁只是在逻辑上阻塞了线程，在用户态的情况下让线程停止了执行，**并没有真正意义上在内核态中挂起对应的内核线程**，从而可以减少很多内核挂起/放下线程耗费的资源。但问题是**当线程越来越多竞争很激烈时，占用CPU的时间变长会导致性能急剧下降**，因此Java虚拟机内部一般对于自旋锁有一定的次数限制，可能是50或者100次循环后就放弃，直接让OS挂起内核线程，让出CPU资源。

#### 只能保证一个共享变量的原子操作

当对一个共享变量执行操作时CAS能保证其原子性，如果对多个共享变量进行操作,CAS就不能保证其原子性。有一个解决方案是利用对象整合多个共享变量，即一个类中的成员变量就是这几个共享变量。然后将这个对象做CAS操作就可以保证其原子性。atomic中提供了 `AtomicReference` 来保证引用对象之间的原子性。

## 悲观锁

Java中的悲观锁有以下几种：

1. `synchronized` 关键字：通过在方法或代码块前加上 `synchronized` 关键字，实现对共享资源的互斥访问。
2. `ReentrantLock` 类：`ReentrantLock` 是JDK提供的一种可重入锁，它提供了比 `synchronized` 更多的灵活性和功能，如可中断锁、公平锁等。
3. `Semaphore` 类： `Semaphore` 是一种计数信号量，它可以控制同时访问某个资源的线程数量。
4. `ReadWriteLock` 接口： `ReadWriteLock` 接口提供了读写锁的实现，它允许多个线程同时读取共享资源，但只允许一个线程写入共享资源，从而提高了并发性能。
5. `StampedLock` 类： `StampedLock` 是JDK8新增的一种锁机制，它支持乐观读、悲观读和写锁，可以根据实际情况选择不同的锁机制来提高并发性能。

------

# synchronized

`synchronized` 是Java中的关键字，用于实现同步机制。当一个方法或代码块被 `synchronized` 修饰时，它在同一时刻只能被一个线程访问，其他线程需要等待。这样可以保证多个线程访问共享资源时的数据安全性和一致性。 `synchronized` 可以用于修饰方法、代码块和静态方法。

## 使用场景

* 实例方法：`public synchronized void func() {}` 。
* 静态方法：`public static synchronized void func() {}` 。
* 实例代码块：`synchronized(this) {}` 。
* 静态代码块：`synchronized(SomeClass.class) {}` 。

## 反编译synchronized加以理解

现在来进一步分析synchronized的具体底层实现，编写如下示例代码：

```java
public class SynchronizedTest {
    public synchronized void method1() {
        System.out.println("synchronized method");
    }

    public void method2() {
        synchronized (SynchronizedTest.class) {
            System.out.println("synchronized block");
        }
    }
}
```

使用 `javap -v SynchronizedTest.class` 命令查看字节码信息，下面是精简过的字节码信息：

```
public synchronized void method1();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String synchronized method
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return

  public void method2();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #5                  // class com/hoily/service/whale/infrastructure/SynchronizedTest
         2: dup
         3: astore_1
         4: monitorenter
         5: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         8: ldc           #6                  // String synchronized block
        10: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        13: aload_1
        14: monitorexit
        15: goto          23
        18: astore_2
        19: aload_1
        20: monitorexit
        21: aload_2
        22: athrow
        23: return
    Exception table:
         from    to  target type
             5    15    18   any
            18    21    18   any
```

对于同步方法，没有特别的指令标识，但方法有一个 `ACC_SYNCHRONIZED` 的标识。

对于同步代码块，使用 `monitorenter` 指令获得锁，然后指定同步中代码块，最后通过 `monitorexit` 释放锁。

关于 `monitor` 指令，详见[链接](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.monitorenter)

每个对象都与一个监视器相关联。 当且仅当监视器有所有者时，它才会被锁定。 执行 `monitorenter` 的线程试图获得与 `objectref` 关联的监视器的所有权，如下所示：

1. 如果与 `objectref` 关联的监视器的条目计数为零，则线程进入监视器并将其条目计数设置为一。 线程就是监视器的所有者。
2. 如果线程已经拥有与 `objectref` 关联的监视器，它会重新进入监视器，并增加其条目计数。
3. 如果另一个线程已经拥有与 `objectref` 关联的监视器，则该线程将阻塞，直到监视器的条目计数为零，然后再次尝试获得所有权。

关于 `monitorexit` 指令，详见[链接](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.monitorexit)

1. 执行 `monitorexit` 的线程必须是与 `objectref` 引用的实例关联的监视器的所有者。
2. 线程递减与 `objectref` 关联的监视器的条目计数。 如果结果是条目计数的值为零，则线程退出监视器并且不再是它的所有者。 其他阻塞进入监视器的线程被允许尝试这样做。

关于 `ACC_SYNCHRONIZED` ，详见[链接](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10)

1. 作为方法调用和返回的一部分，方法级同步是隐式执行的。 同步方法在运行时常量池的 `method_info` 结构中通过 `ACC_SYNCHRONIZED` 标志进行区分，该标志由方法调用指令检查。
2. 当调用设置了 `ACC_SYNCHRONIZED` 的方法时，执行线程进入监视器，调用方法本身，然后退出监视器，无论方法调用是正常完成还是突然完成。 在执行线程拥有监视器期间，没有其他线程可以进入它。 如果在调用 `synchronized` 方法时抛出异常，并且 `synchronized` 方法没有处理该异常，则在 `synchronized` 方法重新抛出异常之前自动退出该方法的监视器。

## monitor监视器

Monitor监视器可以理解为一种同步工具，或者说是同步机制，它通常被描述成一个对象。操作系统的**管程**是概念原理， `ObjectMonitor` 是它的原理实现。

### 操作系统的管程

* 管程是一种程序结构，结构内的多个子程序（对象或模块）形成的多个工作线程互斥访问共享资源。
* 这些共享资源一般是硬件设备或一群变量。管程实现了在一个时间点，最多只有一个线程在执行管程的某个子程序。
* 与那些通过修改数据结构实现互斥访问的并发程序设计相比，管程实现很大程度上简化了程序设计。
* 管程提供了一种机制，线程可以临时放弃互斥访问，等待某些条件得到满足后，重新获得执行权恢复它的互斥访问。

### ObjectMonitor

每个对象都存在一个与之关联的monitor，线程对monitor持有的方式以及持有时机决定了 `synchronized` 的锁状态以及 `synchronized` 的状态升级方式。monitor是通过C++中 `ObjectMonitor` 实现，代码可以通过[openjdk hotspot链接](https://hg.openjdk.org/jdk8u/jdk8u/hotspot/)进行下载openjdk中hotspot版本的源码，具体文件路径在 `src\share\vm\runtime\objectMonitor.hpp` ，具体源码为：

```cpp
// initialize the monitor, exception the semaphore, all other fields
// are simple integers or pointers​
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; // 记录owner线程获取锁的次数
    _waiters      = 0,
    _recursions   = 0; // 锁的重入次数
    _object       = NULL;
    _owner        = NULL; // 指向持有ObjectMonitor的对象的线程
    **_WaitSet**  = NULL; // 存放处于wait状态的线程队列
    _WaitSetLock  = 0;
    _Responsible  = NULL;
    _succ         = NULL;
    _cxq          = NULL;
    FreeNext      = NULL;
    **_EntryList** = NULL; // 存放处于鞥带锁block状态的线程队列
    _SpinFreq     = 0;
    _SpinClock    = 0;
    OwnerIsThread = 0;
    _previous_owner_tid = 0;
}
```

**工作原理**

Java Monitor 的工作机理如图所示：

![](../../../static/images/java/concurrent/object_monitor.awebp)

1. 想要获取 `monitor` 的线程,首先会进入 `_EntryList` 队列。
2. 当某个线程获取到对象的 `monitor` 后,进入 `_Owner` 区域，设置为当前线程,同时计数器 `_count` 加 `1`。
3. 如果线程调用了 `wait()` 方法，则会进入 `_WaitSet` 队列。它会释放 `monitor` 锁，即 `_owner` 赋值为 `null` , `_count` 自减 `1` ,进入 `_WaitSet` 队列阻塞等待。
4. 如果其他线程调用 `notify()` / `notifyAll()` ，会唤醒 `_WaitSet` 中的某个线程，该线程再次尝试获取 `monitor` 锁，成功即进入 `_Owner` 区域。
5. 同步方法执行完毕了，线程退出临界区，会将 `monitor` 的 `owner` 设为 `null`，并释放监视锁。

上述的过程等价于：

```java
synchronized(this) {  //进入_EntryList队列
    // do something
    this.wait();  //进入_WaitSet队列
}
```

### 对象与ObjectMonitor的关联

对象是如何跟monitor关联的呢？直接先看图：

![](../../../static/images/java/concurrent/object_monitor__object.awebp)

对象头主要包括两部分数据：
* **Mark Word** : 用于存储对象自身的运行时数据，它是实现轻量级锁和偏向锁的关键。
* **Class Pointer** : 是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例

**Mark Word** 用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等。

在32位的HotSpot虚拟机中，如果对象处于未被锁定的状态下，那么**Mark Word**的32bit空间里的25位用于存储对象哈希码，4bit用于存储对象分代年龄，2bit用于存储锁标志位，1bit固定为0，表示非偏向锁。其他状态如上图所示。

`synchronized` 是重量级锁，也就是说 `synchronized` 的对象锁，**Mark Word**锁标识位为 `10` ，其中指针指向的是Monitor对象的起始地址。

## 锁类型

Java 提供了很方便的线程同步的内置锁 `synchronized` ，可以极大地简化并发模型。但早期 `synchronized` 的实现性能较低，比较耗费资源，Java 6 之后对 `synchronized` 的实现做了进一步的升级和优化，提高了并发性能。

### 偏向锁

偏向锁解决优化锁性能的**出发点是“不存在多线程锁竞争，总是由同一线程获取锁的”** 经验数据。

偏向锁的处理过程是，如果一个线程获得了锁，那么锁就进入偏向模式，此时 `Mark Word` 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。换句通俗易懂的话说：偏向锁其中的“偏”是偏心的偏。它的意思就是说，这个锁会**偏向于第一个获得它的线程，在接下来的执行过程中，假如该锁没有被其他线程所获取，没有其他线程来竞争该锁，那么持有偏向锁的线程将永远不需要进行同步操作**。在此线程之后的执行过程中，如果再次进入或者退出同一段同步块代码，并不再需要去进行加锁或者解锁操作，而是会做以下的步骤：

1. Load-and-test，也就是简单判断一下当前线程id是否与Markword当中的线程id是否一致。
2. 如果一致，则说明此线程已经成功获得了锁，继续执行下面的代码。
3. 如果不一致，则要检查一下对象是否还是可偏向，即“是否偏向锁”标志位的值。
4. 如果还未偏向，则利用CAS操作来竞争锁，也即是第一次获取锁时的操作。

但是当第二个线程来尝试获取锁时，如果此对象已经偏向了，并且不是偏向自己，则说明存在了竞争。此时会根据该锁的线程竞争情况，可能会产生偏向撤销，重新偏向，但大部分情况下就是膨胀成轻量级锁了。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果，毕竟极有可能连续多次是同一个线程申请相同的锁。但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失。

**偏向锁膨胀过程**

1. 当第一个线程进入的时候发现是匿名偏向状态，则会用 `CAS` 指令把 `mark word` 中的 `threadid` 替换为当前线程的id如果替换成功，则证明成功拿到锁，失败则锁膨胀;
2. 当线程第二次进入同步块时，如果发现线程id和对象头中的偏向线程id一致，则经过一些比较之后，在当前线程栈的 `lock record` 中添加一个空的 `Displaced Mark Word` ，由于操作的是私有线程栈，所以不需要 `CAS` 操作， `synchronized` 带来的开销基本可以忽略;
3. 当其他线程进入同步块中时，发现偏向线程不是当前线程，则进入到撤销偏向锁的逻辑，当达到全局安全点时，锁开始膨胀为轻量级锁，原来的线程仍然持有锁，如果发现偏向线程挂了，那么就把偏向锁撤销，并将对象头内的 `MarkWord` 改为无锁状态，锁膨胀，
4. 但是需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁。

**偏向锁撤销过程**

1. 在一个安全点停止拥有锁的线程。
2. 遍历线程栈，如果存在锁记录的话，需要修复锁记录和 `Markword` ，使其变成无锁状态。
3. 唤醒当前线程，将当前锁升级成轻量级锁。

所以，如果程序中大量同步代码块大多数情况下都是有两个及以上的线程竞争的话，那么偏向锁就会是一种累赘，对于这种情况，我们可以一开始就通过 `XX:-UseBiasedLocking` 把偏向锁这个默认功能给关闭，从而做到性能上的优化。

### 轻量级锁

倘若偏向锁失败，`synchronized` 并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段，此时 `MarkWord` 的结构也变为轻量级锁的结构。

轻量级锁能够提升锁性能的**出发点是“对绝大部分的锁，在整个同步周期内都不存在竞争”**，这个依据是根据一些经验得到的。

**轻量级锁膨胀过程**

当锁膨胀为轻量级锁时，首先判断是否有线程持有锁(判断 `mark work`)，如果是，则在当前线程栈中创建一个 `lock record` 复制 `mark word`并且 `CAS` 的把当前线程栈的 `lock record` 的地址放到对象头中(细节：之前持有偏向锁的线程会优先进行 `CAS` 并将锁信息指针更改到对象头内 `mark word` 中)，如果成功，则说明获取到轻量级锁，如果失败，则说明锁已经被占用了，此时记录线程的重入次数(把 `lock record` 的 `mark word` 设置为 `null` )，锁会自旋（自适应自旋），确保在竞争不激烈的情况下仍然可以不膨胀为重量级锁从而减少消耗，如果 `CAS` 失败，则说明线程出现竞争，需要膨胀为重量级的锁。

轻量级锁主要有两种类型：

#### 自旋锁

自旋锁是指当一个线程尝试获取某个锁时，如果该锁已被其他线程占用，就**一直循环检测锁是否被释放**，而不是进入线程挂起或睡眠状态，避免了内核态与用户态的切换，线程阻塞与线程切换，很大程度地提高了锁的性能。

当然，自旋锁也是有适用范围的，自旋锁适用于锁保护的临界区很小的情况，临界区很小的话，锁占用的时间就很短。对于竞争线程多处理器少，大量线程自旋会比较浪费。自旋锁也会占用CPU，对于计算密集型任务，也会比较占用资源。

#### 自适应自旋锁

**自适应自旋解决的是“锁竞争时间不确定”的问题**。JVM很难感知到确切的锁竞争时间，而交给用户分析就违反了JVM的设计初衷。自适应自旋假定不同线程持有同一个锁对象的时间基本相当，竞争程度趋于稳定，因此，可以根据上一次自旋的时间与结果调整下一次自旋的时间。可以参考如下策略：

* 如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将允许自旋等待持续相对更长的时间，比如100个循环。
* 相反的，如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能减少自旋时间甚至省略自旋过程，以避免浪费处理器资源。

然而，自适应自旋也不能彻底地解决竞争时间不确定的问题，如果默认的自旋次数设置不合理（过高或过低），那么自适应的过程将很难收敛到合适的值。

### 重量级锁

在JDK 1.6之前， `synchronized` 的实现使用了监视器模型（monitor），监视器锁可以认为直接对应底层操作系统中的互斥量（mutex）。这种同步方式的成本非常高，包括系统调用引起的内核态与用户态切换、线程阻塞造成的线程切换等。因此被称这种锁为“重量级锁”。

### 样例代码分析

```java
public class ObjectHead {
    public static void main(String[] args) throws InterruptedException {
        /** 
        无锁态：虚拟机刚启动时 new 出来的对象处于无锁状态
        **/
        Object obj = new Object();
        // 查看对象内部信息
        System.out.println(ClassLayout.parseInstance(obj).toPrintable());


        /** 
        匿名偏向锁：休眠4S后再创建出来的对象处于匿名偏向锁状态
        PS：当一个线程在执行被synchronized关键字修饰的代码或方法时，如果看到该锁
        对象是处于匿名偏向锁状态的（标志位为偏向锁但是对象头中MrakWord内threadID
        为空），那么这个线程将会利用cas机制把自己的线程ID设置到mrakword中，此后
        如果没有其他线程来竞争该锁，那么这个线程再执行被需要获取该锁的代码将不需
        要经过任何获取锁和释放锁的过程。
        **/
        Thread.sleep(4000);
        Object obj1 = new Object();
        System.out.println(ClassLayout.parseInstance(obj1).toPrintable());

        /** 
        轻量级锁：对于真正的无锁态对象obj加锁之后的对象处于轻量级锁状态
        **/
        synchronized (obj) {
            // 查看对象内部信息
            System.out.println(ClassLayout.parseInstance(obj).toPrintable());
        }

        /** 
        重量级锁：调用wait方法之后锁对象直接膨胀为重量级锁状态
        **/
        new Thread(()->{
            try {
                obj.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        Thread.sleep(1);
        synchronized (obj) {
            // 查看对象内部信息
            System.out.println(ClassLayout.parseInstance(obj).toPrintable());
        }
    }
}
```
输出结果：
```
java.lang.Object object internals:  锁标志位状态：001：真正意义上无锁状态
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           e5 01 00 20 (11100101 00000001 00000000 00100000) (536871397)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

java.lang.Object object internals:  锁标志位状态：101：匿名偏向锁状态
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           05 00 00 00 (00000101 00000000 00000000 00000000) (5)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           e5 01 00 20 (11100101 00000001 00000000 00100000) (536871397)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

java.lang.Object object internals:  锁标志位状态：000：轻量级锁状态
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           18 f5 41 01 (00011000 11110101 01000001 00000001) (21099800)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           e5 01 00 20 (11100101 00000001 00000000 00100000) (536871397)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

java.lang.Object object internals:  锁标志位状态：010：重量级锁状态
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           5a de db 17 (01011010 11011110 11011011 00010111) (400285274)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           e5 01 00 20 (11100101 00000001 00000000 00100000) (536871397)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

/**抛出异常原因：违法的监控状态异常。当某个线程试图等待一个自己并不拥有的对象（Obj）的监控器或者通知其他线程等待该对象（Obj）的监控器时，抛出该异常。**/
Exception in thread "Thread-0": java.lang.IllegalMonitorStateException
	at java.lang.Object.wait(Native Method)
	at java.lang.Object.wait(Object.java:502)
	at com.sixstar.springbootvolatilesynchronized.Synchronized.ObjectHead.lambda$main$0(ObjectHead.java:27)
	at java.lang.Thread.run(Thread.java:748)
```

### 三种锁的总结

| 锁类型 | 优点 | 缺点 | 适用场景 |
| --- | --- | --- | --- |
| 偏向锁 | 加锁和解锁不需要额外的消耗，和执行非同步代码相比仅存在纳秒级别的差距 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗 | 适用只有一个线程访问的同步块场景 |
| 轻量级锁 | 竞争的线程不会阻塞，提高了程序的相应速度 | 如果始终得不到锁竞争的线程，使用自旋会消耗CPU | 追求响应时间，同步块执行非常快 |
| 重量级锁 | 线程竞争不使用自旋，不会消耗CPU | 线程阻塞，相应时间缓慢 | 追求吞吐量，同步块执行较慢 |

## 锁优化

在JDK1.6之前， `synchronized` 的实现才会直接调用 `ObjectMonitor` 的enter和exit，这种锁被称之为重量级锁。从JDK6开始，HotSpot虚拟机开发团队对Java中的锁进行优化，如增加了适应性自旋、锁消除、锁粗化、轻量级锁和偏向锁等优化策略。


### 锁消除

锁削除是指虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行削除。

日常代码开发中，有一些开发者，在没并发情况下，也使用加锁。如没并发可能，直接上来就 `ConcurrentHashMap` 。这种情况下锁消除就可以避免锁的资源消耗。

### 锁粗化

锁粗话概念比较好理解，就是将**多个连续的加锁、解锁操作连接在一起**，扩展成一个范围更大的锁。

为何需要锁租化？在使用同步锁的时候，需要让同步块的作用范围尽可能小—仅在共享数据的实际作用域中才进行同步，这样做的目的是 为了使需要同步的操作数量尽可能缩小，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。但是如果一系列的连续加锁解锁操作，可能会导致不必要的性能损耗，所以引入锁粗话的概念。

锁租化比喻，举个例子，买门票进动物园。老师带一群小朋友去参观，验票员如果知道他们是个集体，就可以把他们看成一个整体（锁租化），一次性验票过，而不需要一个个找他们验票。


-----

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

-----

# JDK 中内置的工具锁

## ReentrantLock

## ReentrantReadWriteLock

## Semaphore

## CountDownLatch

`CountDownLatch` 可以使一个或多个线程等待，直到其他线程的一系列操作完成，用来实现多线程屏障。

SynchronousQueue


# 参考资料

* [掘金 - (四)深入理解Java并发编程之无锁CAS机制、魔法类Unsafe、原子包Atomic](https://juejin.cn/post/7078545790594973733)
* [JavaGuide - AQS详解](https://javaguide.cn/java/concurrent/aqs.html)
* [掘金 - Synchronized解析—如果你愿意一层一层剥开我的心](https://juejin.cn/post/6844903918653145102)
* [掘金 - 为什么wait和notify必须放在synchronized中？](https://juejin.cn/post/7067322092936495112)
* [掘金 - 浅谈偏向锁、轻量级锁、重量级锁](https://juejin.cn/post/6844903550586191885)
* [掘金 - (二)彻底理解Java并发编程之Synchronized关键字实现原理剖析](https://juejin.cn/post/6977744582725681182)