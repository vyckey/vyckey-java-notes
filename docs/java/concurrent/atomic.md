---
title: Atomic
tags: [java, concurrent, atom]
sidebar_label: Atomic
sidebar_position: 1
---

`Atomic` 是原子的意思，在计算机中，表示是一个不可中断的特性。

Java 中并发包`java.util.concurrent`的原子类都存放在`java.util.concurrent.atomic`包。根据操作的数据类型，可分为四类：

1. 基本类型：`AtomicInteger`，`AtomicLong`，`AtomicBoolean`。
2. 数组类型：`AtomicIntegerArray`，`AtomicLongArray`，`AtomicReferenceArray`。
3. 引用类型：`AtomicReference`，引用类型原子类；`AtomicMarkableReference`，原子更新带有标记的引用类型；`AtomicStampedReference`，原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
4. 对象属性修改类型：`AtomicIntegerFieldUpdater`，`AtomicLongFieldUpdater`，`AtomicReferenceFieldUpdater`。

# 四类原子类

## 基本类型原子类

上面提到的三个类使用方式几乎相同，所以这里仅以 `AtomicInteger` 为例子来介绍。

`AtomicInteger` 类常用方法：
```java
public final int get() //获取当前的值
public final int getAndSet(int newValue)//获取当前的值，并设置新的值
public final int getAndIncrement()//获取当前的值，并自增
public final int getAndDecrement() //获取当前的值，并自减
public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
public final void lazySet(int newValue)//最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
```

`AtomicInteger` 使用示例：
```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicIntegerTest {
    public static void main(String[] args) {
        int temvalue = 0;
        AtomicInteger i = new AtomicInteger(0);
        temvalue = i.getAndSet(3);
        System.out.println("temvalue:" + temvalue + ";  i:" + i); //temvalue:0;  i:3
        temvalue = i.getAndIncrement();
        System.out.println("temvalue:" + temvalue + ";  i:" + i); //temvalue:3;  i:4
        temvalue = i.getAndAdd(5);
        System.out.println("temvalue:" + temvalue + ";  i:" + i); //temvalue:4;  i:9
    }

}
```

`AtomicInteger` 类核心源码：
```java
    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;

    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```

`AtomicInteger` 类主要利用 CAS (compare and swap) + `volatile` 和 native 方法来保证原子操作，从而避免 `synchronized` 的高开销，执行效率大为提升。`UnSafe#objectFieldOffset()` 方法是一个本地方法，这个方法是用来拿到“原来的值”的内存地址。

## 数组类型原子类

上面提到的三个类使用方式几乎相同，所以这里仅以 `AtomicIntegerArray` 为例子来介绍。

`AtomicIntegerArray` 类常用方法：
```java
public final int get(int i) //获取 index=i 位置元素的值
public final int getAndSet(int i, int newValue)//返回 index=i 位置的当前的值，并将其设置为新值：newValue
public final int getAndIncrement(int i)//获取 index=i 位置元素的值，并让该位置的元素自增
public final int getAndDecrement(int i) //获取 index=i 位置元素的值，并让该位置的元素自减
public final int getAndAdd(int i, int delta) //获取 index=i 位置元素的值，并加上预期的值
boolean compareAndSet(int i, int expect, int update) //如果输入的数值等于预期值，则以原子方式将 index=i 位置的元素值设置为输入值（update）
public final void lazySet(int i, int newValue)//最终 将index=i 位置的元素设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
```

`AtomicIntegerArray` 类使用示例 :
```java
import java.util.concurrent.atomic.AtomicIntegerArray;

public class AtomicIntegerArrayTest {
    public static void main(String[] args) {
        int temvalue = 0;
        int[] nums = { 1, 2, 3, 4, 5, 6 };
        AtomicIntegerArray i = new AtomicIntegerArray(nums);
        for (int j = 0; j < nums.length; j++) {
            System.out.println(i.get(j));
        }
        temvalue = i.getAndSet(0, 2);
        System.out.println("temvalue:" + temvalue + ";  i:" + i);
        temvalue = i.getAndIncrement(0);
        System.out.println("temvalue:" + temvalue + ";  i:" + i);
        temvalue = i.getAndAdd(0, 5);
        System.out.println("temvalue:" + temvalue + ";  i:" + i);
    }
}
```

## 引用类型原子类

基本类型原子类只能更新一个变量，如果需要原子更新多个变量，需要使用引用类型原子类。

`AtomicReference` 类使用示例 :

```java
import java.util.concurrent.atomic.AtomicReference;

public class AtomicReferenceTest {
    public static void main(String[] args) {
        AtomicReference < Person > ar = new AtomicReference < Person > ();
        Person person = new Person("SnailClimb", 22);
        ar.set(person);
        Person updatePerson = new Person("Daisy", 20);
        ar.compareAndSet(person, updatePerson);

        System.out.println(ar.get().getName());
        System.out.println(ar.get().getAge());
    }
}

@Data
static class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }
}
```

`AtomicStampedReference` 类使用示例 :
```java
import java.util.concurrent.atomic.AtomicStampedReference;

public class AtomicStampedReferenceDemo {
    public static void main(String[] args) {
        // 实例化、取当前值和 stamp 值
        final Integer initialRef = 0, initialStamp = 0;
        final AtomicStampedReference<Integer> asr = new AtomicStampedReference<>(initialRef, initialStamp);
        System.out.println("currentValue=" + asr.getReference() + ", currentStamp=" + asr.getStamp());

        // compare and set
        final Integer newReference = 666, newStamp = 999;
        final boolean casResult = asr.compareAndSet(initialRef, newReference, initialStamp, newStamp);
        System.out.println("currentValue=" + asr.getReference()
                + ", currentStamp=" + asr.getStamp()
                + ", casResult=" + casResult);

        // 获取当前的值和当前的 stamp 值
        int[] arr = new int[1];
        final Integer currentValue = asr.get(arr);
        final int currentStamp = arr[0];
        System.out.println("currentValue=" + currentValue + ", currentStamp=" + currentStamp);

        // 单独设置 stamp 值
        final boolean attemptStampResult = asr.attemptStamp(newReference, 88);
        System.out.println("currentValue=" + asr.getReference()
                + ", currentStamp=" + asr.getStamp()
                + ", attemptStampResult=" + attemptStampResult);

        // 重新设置当前值和 stamp 值
        asr.set(initialRef, initialStamp);
        System.out.println("currentValue=" + asr.getReference() + ", currentStamp=" + asr.getStamp());

        // [不推荐使用，除非搞清楚注释的意思了] weak compare and set
        // 困惑！weakCompareAndSet 这个方法最终还是调用 compareAndSet 方法。[版本: jdk-8u191]
        // 但是注释上写着 "May fail spuriously and does not provide ordering guarantees,
        // so is only rarely an appropriate alternative to compareAndSet."
        // todo 感觉有可能是 jvm 通过方法名在 native 方法里面做了转发
        final boolean wCasResult = asr.weakCompareAndSet(initialRef, newReference, initialStamp, newStamp);
        System.out.println("currentValue=" + asr.getReference()
                + ", currentStamp=" + asr.getStamp()
                + ", wCasResult=" + wCasResult);
    }
}
```
输出结果如下：
```
currentValue=0, currentStamp=0
currentValue=666, currentStamp=999, casResult=true
currentValue=666, currentStamp=999
currentValue=666, currentStamp=88, attemptStampResult=true
currentValue=0, currentStamp=0
currentValue=666, currentStamp=999, wCasResult=true
```

`AtomicMarkableReference` 类使用示例 :
```java
import java.util.concurrent.atomic.AtomicMarkableReference;

public class AtomicMarkableReferenceDemo {
    public static void main(String[] args) {
        // 实例化、取当前值和 mark 值
        final Boolean initialRef = null, initialMark = false;
        final AtomicMarkableReference<Boolean> amr = new AtomicMarkableReference<>(initialRef, initialMark);
        System.out.println("currentValue=" + amr.getReference() + ", currentMark=" + amr.isMarked());

        // compare and set
        final Boolean newReference1 = true, newMark1 = true;
        final boolean casResult = amr.compareAndSet(initialRef, newReference1, initialMark, newMark1);
        System.out.println("currentValue=" + amr.getReference()
                + ", currentMark=" + amr.isMarked()
                + ", casResult=" + casResult);

        // 获取当前的值和当前的 mark 值
        boolean[] arr = new boolean[1];
        final Boolean currentValue = amr.get(arr);
        final boolean currentMark = arr[0];
        System.out.println("currentValue=" + currentValue + ", currentMark=" + currentMark);

        // 单独设置 mark 值
        final boolean attemptMarkResult = amr.attemptMark(newReference1, false);
        System.out.println("currentValue=" + amr.getReference()
                + ", currentMark=" + amr.isMarked()
                + ", attemptMarkResult=" + attemptMarkResult);

        // 重新设置当前值和 mark 值
        amr.set(initialRef, initialMark);
        System.out.println("currentValue=" + amr.getReference() + ", currentMark=" + amr.isMarked());

        // [不推荐使用，除非搞清楚注释的意思了] weak compare and set
        // 困惑！weakCompareAndSet 这个方法最终还是调用 compareAndSet 方法。[版本: jdk-8u191]
        // 但是注释上写着 "May fail spuriously and does not provide ordering guarantees,
        // so is only rarely an appropriate alternative to compareAndSet."
        // todo 感觉有可能是 jvm 通过方法名在 native 方法里面做了转发
        final boolean wCasResult = amr.weakCompareAndSet(initialRef, newReference1, initialMark, newMark1);
        System.out.println("currentValue=" + amr.getReference()
                + ", currentMark=" + amr.isMarked()
                + ", wCasResult=" + wCasResult);
    }
}
```
输出结果如下：
```
currentValue=null, currentMark=false
currentValue=true, currentMark=true, casResult=true
currentValue=true, currentMark=true
currentValue=true, currentMark=false, attemptMarkResult=true
currentValue=null, currentMark=false
currentValue=true, currentMark=true, wCasResult=true
```

## 对象的属性修改类型原子类

如果需要原子更新某个类里的某个字段时，需要用到对象的属性修改类型原子类。要想原子地更新对象的属性需要两步：
1. 因为对象的属性修改类型原子类都是抽象类，所以每次使用都必须使用静态方法 newUpdater()创建一个更新器，并且需要设置想要更新的类和属性。
2. 更新的对象属性必须使用 public volatile 修饰符。

三个类提供的方法几乎相同，所以我们这里以 `AtomicIntegerFieldUpdater` 为例子来介绍。

`AtomicIntegerFieldUpdater` 类使用示例：
```java
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;

public class AtomicIntegerFieldUpdaterTest {
	public static void main(String[] args) {
		AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");

		User user = new User("Java", 22);
		System.out.println(a.getAndIncrement(user));// 22
		System.out.println(a.get(user));// 23
	}
}

@Data
static class User {
	private String name;
	public volatile int age;

	public User(String name, int age) {
		super();
		this.name = name;
		this.age = age;
	}
}
```

# Unsafe一探究竟

Java中的 `Unsafe` 类是一个非常特殊的类，它提供了一些不安全的操作，这些操作通常是Java语言中不允许的，比如直接操作内存、线程挂起、CAS操作等。`Unsafe` 类主要用于Java的底层实现，比如JVM、NIO等，一般情况下不建议开发者直接使用Unsafe类。 `Unsafe` 类的存在是为了方便Java底层的实现，同时也提供了一些高级的操作，比如原子操作、内存分配等，这些操作可以在某些情况下提高程序的性能。

## CAS操作

 `Unsafe` 类提供了最基本的CAS操作的API。

```java
public final native boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);
public final native boolean compareAndSwapInt(Object o, long offset,int expected,int x);
public final native boolean compareAndSwapLong(Object o, long offset,long expected,long x);
```

## 内存操作

### 直接内存操作

```java
/分配内存指定大小的内存
public native long allocateMemory(long bytes);
//根据给定的内存地址address设置重新分配指定大小的内存
public native long reallocateMemory(long address, long bytes);
//用于释放allocateMemory和reallocateMemory申请的内存
public native void freeMemory(long address);
//将指定对象的给定offset偏移量内存块中的所有字节设置为固定值
public native void setMemory(Object o, long offset, long bytes, byte value);
//设置给定内存地址的值
public native void putAddress(long address, long x);
//获取指定内存地址的值
public native long getAddress(long address);

//设置给定内存地址的long值
public native void putLong(long address, long x);
//获取指定内存地址的long值
public native long getLong(long address);
//设置或获取指定内存的byte值
public native byte  getByte(long address);
public native void  putByte(long address, byte x);
//其他基本数据类型(long,char,float,double,short等)的操作与putByte及getByte相同
.......... 省略代码
//操作系统的内存页大小
public native int pageSize();
```

### 对象以及相关操作

```java
//传入一个对象的class并创建该实例对象，但不会调用构造方法
public native Object allocateInstance(Class cls) throws InstantiationException;

//获取字段f在实例对象中的偏移量
public native long objectFieldOffset(Field f);
//静态属性的偏移量，用于在对应的Class对象中读写静态属性
public native long staticFieldOffset(Field f);
//返回值就是f.getDeclaringClass()
public native Object staticFieldBase(Field f);


//获得给定对象偏移量上的int值，所谓的偏移量可以简单理解为指针指向该变量的内存地址，
//通过偏移量便可得到该对象的变量，进行各种操作
public native int getInt(Object o, long offset);
//设置给定对象上偏移量的int值
public native void putInt(Object o, long offset, int x);

//获得给定对象偏移量上的引用类型的值
public native Object getObject(Object o, long offset);
//设置给定对象偏移量上的引用类型的值
public native void putObject(Object o, long offset, Object x);
//其他基本数据类型(long,char,byte,float,double)的操作与getInthe及putInt相同

//设置给定对象的int值，使用volatile语义，即设置后立马更新到内存对其他线程可见
public native void  putIntVolatile(Object o, long offset, int x);
//获得给定对象的指定偏移量offset的int值，使用volatile语义，总能获取到最新的int值。
public native int getIntVolatile(Object o, long offset);

//其他基本数据类型(long,char,byte,float,double)的操作与putIntVolatile
//及getIntVolatile相同，引用类型putObjectVolatile也一样。
..........省略代码

//与putIntVolatile一样，但要求被操作字段必须有volatile修饰
public native void putOrderedInt(Object o,long offset,int x);
```

### 数组以及相关操作

```java
//获取数组第一个元素的偏移地址
public native int arrayBaseOffset(Class arrayClass);
//数组中一个元素占据的内存空间,arrayBaseOffset与arrayIndexScale配合使用，可定位数组中每个元素在内存中的位置
public native int arrayIndexScale(Class arrayClass);
```

## 线程操作

将一个线程进行挂起是通过 `park` 方法实现的，调用 `park` 后，线程将一直阻塞直到超时或者中断等条件出现。 `unpark` 可以终止一个挂起的线程，使其恢复正常。Java对线程的挂起操作被封装在 `LockSupport` 类中，`LockSupport` 类中有各种版本 `pack` 方法，其底层实现最终还是使用 `Unsafe.park()` 方法和 `Unsafe.unpark()` 方法来实现。

```java
//线程调用该方法，线程将一直阻塞直到超时，或者是中断条件出现。  
public native void park(boolean isAbsolute, long time);  

//终止挂起的线程，恢复正常.java.util.concurrent包中挂起操作都是在LockSupport类实现的，其底层正是使用这两个方法，  
public native void unpark(Object thread); 
```

## 内存屏障

```java
//在该方法之前的所有读操作，一定在load屏障之前执行完成
public native void loadFence();
//在该方法之前的所有写操作，一定在store屏障之前执行完成
public native void storeFence();
//在该方法之前的所有读写操作，一定在full屏障之前执行完成，这个内存屏障相当于上面两个的合体功能
public native void fullFence();
```

## 其他操作

```java
//获取持有锁，已不建议使用
@Deprecated
public native void monitorEnter(Object var1);
//释放锁，已不建议使用
@Deprecated
public native void monitorExit(Object var1);
//尝试获取锁，已不建议使用
@Deprecated
public native boolean tryMonitorEnter(Object var1);

//获取本机内存的页数，这个值永远都是2的幂次方  
public native int pageSize();  

//告诉虚拟机定义了一个没有安全检查的类，默认情况下这个类加载器和保护域来着调用者类  
public native Class defineClass(String name, byte[] b, int off, int len, ClassLoader loader, ProtectionDomain protectionDomain);  

//加载一个匿名类
public native Class defineAnonymousClass(Class hostClass, byte[] data, Object[] cpPatches);
//判断是否需要加载一个类
public native boolean shouldBeInitialized(Class<?> c);
//确保类一定被加载 
public native  void ensureClassInitialized(Class<?> c);
```

# 参考资料

* [JavaGuide - Atomic原子类总结](https://javaguide.cn/java/concurrent/atomic-classes.html)
* [掘金 - (四)深入理解Java并发编程之无锁CAS机制、魔法类Unsafe、原子包Atomic](https://juejin.cn/post/7078545790594973733)