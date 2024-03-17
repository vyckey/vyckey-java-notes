---
title: Program
tags: [java, concurrent]
sidebar_label: Program
sidebar_position: 10
---

# 参考资料

# 生产者消费者模式

## 使用 `wait()` 和 `notifyAll()` 实现

```java
public class ProducerConsumerTest {
    @Test
    public void test() {
        for (int i = 0; i < 6; i++) {
            Thread producer = new Thread(this::produce);
            producer.setName("生产者" + i);
            producer.start();

            Thread consumer = new Thread(this::consume);
            consumer.setName("消费者" + i);
            consumer.start();
        }
    }

    private final LinkedList<Object> bufferList = new LinkedList<>();
    private final int maxCount = 5;

    private void produce() {
        synchronized (bufferList) {
            while (bufferList.size() >= maxCount) {
                try {
                    System.out.println(Thread.currentThread().getName() + ": 缓冲区已满");
                    bufferList.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }

            Object object = new Object();
            bufferList.addLast(object);
            System.out.println(Thread.currentThread().getName() + ": 生产者生产了一条数据" + object + "，缓冲区共" + bufferList.size() + "条");
            bufferList.notifyAll();
        }
    }

    private void consume() {
        synchronized (bufferList) {
            while (bufferList.size() == 0) {
                try {
                    System.out.println(Thread.currentThread().getName() + ": 缓冲区已空");
                    bufferList.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }

            Object object = bufferList.removeFirst();
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println(Thread.currentThread().getName() + ": 消费者消费了一条数据" + object + "，缓冲区共" + bufferList.size() + "条");
            bufferList.notifyAll();
        }
    }
}
```

输出结果：
```
生产者0: 生产者生产了一条数据java.lang.Object@62f1365，缓冲区共1条
生产者1: 生产者生产了一条数据java.lang.Object@a563326，缓冲区共2条
消费者1: 消费者消费了一条数据java.lang.Object@62f1365，缓冲区共1条
消费者5: 消费者消费了一条数据java.lang.Object@a563326，缓冲区共0条
生产者5: 生产者生产了一条数据java.lang.Object@40485833，缓冲区共1条
消费者2: 消费者消费了一条数据java.lang.Object@40485833，缓冲区共0条
消费者4: 缓冲区已空
消费者0: 缓冲区已空
生产者4: 生产者生产了一条数据java.lang.Object@6d3dde3e，缓冲区共1条
消费者3: 消费者消费了一条数据java.lang.Object@6d3dde3e，缓冲区共0条
生产者3: 生产者生产了一条数据java.lang.Object@3d1c1384，缓冲区共1条
生产者2: 生产者生产了一条数据java.lang.Object@2452581c，缓冲区共2条
消费者0: 消费者消费了一条数据java.lang.Object@3d1c1384，缓冲区共1条
消费者4: 消费者消费了一条数据java.lang.Object@2452581c，缓冲区共0条
```

## 使用 `BlockingQueue` 实现

```java
public class ProducerConsumerTest {
    @Test
    public void test() {
        for (int i = 0; i < 6; i++) {
            Thread producer = new Thread(this::produce);
            producer.setName("生产者" + i);
            producer.start();

            Thread consumer = new Thread(this::consume);
            consumer.setName("消费者" + i);
            consumer.start();
        }
    }

    private final BlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(10);
    private final AtomicInteger counter = new AtomicInteger();

    private final int maxCount = 5;

    private void produce() {
        Object object = new Object();
        try {
            blockingQueue.put(object);
            System.out.println(Thread.currentThread().getName() + ": 生产者生产了一条数据" + object + "，缓冲区共" + counter.incrementAndGet() + "条");
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    private void consume() {
        Object object = null;
        try {
            object = blockingQueue.take();
            System.out.println(Thread.currentThread().getName() + ": 消费者消费了一条数据" + object + "，缓冲区共" + counter.decrementAndGet() + "条");
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 使用 `Semaphore` 实现

```java
public class ProducerConsumerTest {
    @Test
    public void test() {
        for (int i = 0; i < 6; i++) {
            Thread producer = new Thread(this::produce);
            producer.setName("生产者" + i);
            producer.start();

            Thread consumer = new Thread(this::consume);
            consumer.setName("消费者" + i);
            consumer.start();
        }
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    private final Semaphore notFull = new Semaphore(10);
    private final Semaphore notEmpty = new Semaphore(0);
    private final Semaphore mutex = new Semaphore(1);
    private final List<Object> bufferList = new LinkedList<>();

    private void produce() {
        try {
            notFull.acquire();
            mutex.acquire();
            Object object = new Object();
            bufferList.add(object);
            System.out.println(Thread.currentThread().getName() + ": 生产者生产了一条数据" + object + "，缓冲区共" + bufferList.size() + "条");
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            mutex.release();
            notEmpty.release();
        }
    }

    private void consume() {
        try {
            notEmpty.acquire();
            mutex.acquire();
            Object object = bufferList.remove(0);
            System.out.println(Thread.currentThread().getName() + ": 消费者消费了一条数据" + object + "，缓冲区共" + bufferList.size() + "条");
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            mutex.release();
            notFull.release();
        }
    }
}
```

## CountDownLatch

> 实现学生一起比赛跑步的程序，CountDownLatch 初始化为学生数量的线程，鸣枪后，每个学生就是一条线程，来完成各自的任务，当第一个学生跑完全程后，CountDownLatch 就会减一，直到所有的学生完成后，CountDownLatch 会变为 0 ，接下来再一起宣布跑步成绩。

```java
public class StudentRunRace {
    private final int studentCount;
    private final CountDownLatch stopLatch;
    private final CountDownLatch runLatch;

    StudentRunRace(int studentCount) {
        this.studentCount = studentCount;
        this.stopLatch = new CountDownLatch(1);
        this.runLatch = new CountDownLatch(studentCount);
    }

    private void waitSignal() throws Exception {
        System.out.println("选手" + Thread.currentThread().getName() + "正在等待裁判发布口令");
        stopLatch.await();
        System.out.println("选手" + Thread.currentThread().getName() + "已接受裁判口令");
        Thread.sleep((long) (Math.random() * 10000));
        System.out.println("选手" + Thread.currentThread().getName() + "到达终点");
        runLatch.countDown();
    }

    private void waitStop() throws Exception {
        Thread.sleep((long) (Math.random() * 10000));
        System.out.println("裁判" + Thread.currentThread().getName() + "即将发布口令");
        stopLatch.countDown();
        System.out.println("裁判" + Thread.currentThread().getName() + "已发送口令，正在等待所有选手到达终点");
        runLatch.await();
        System.out.println("所有选手都到达终点");
        System.out.println("裁判" + Thread.currentThread().getName() + "汇总成绩排名");
    }

    public void startRace() {
        ExecutorService service = Executors.newCachedThreadPool();
        for (int i = 0; i < studentCount; i++) {
            service.execute(() -> {
                try {
                    waitSignal();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            });
        }
        try {
            waitStop();
        } catch (Exception e) {
            e.printStackTrace();
        }
        service.shutdown();
    }

    public static void main(String[] args) {
        StudentRunRace studentRunRace = new StudentRunRace(10);
        studentRunRace.startRace();
    }
}
```

## 多线程循环打印数字

问题：创建 `n` （其中 `n>=2` ）个线程，每个线程 `i`（其中 `i>=1` ） 轮流打印数字 `i` ，并循环打印。

## 使用 `wait()` 和 `notifyAll()` 实现

```java
public class NumberPrinter {
    private final Object lock = new Object();
    private final int threadNum;
    private Integer integer = 1;
    private int printThreadId = 1;

    public NumberPrinter(int threadNum) {
        this.threadNum = threadNum;
    }

    public static void main(String[] args) throws InterruptedException {
        new NumberPrinter(3).print();

        Thread.sleep(5000);
    }

    public void print() {
        for (int i = 0; i < threadNum; i++) {
            startThread(i + 1);
        }
    }

    private void startThread(int threadId) {
        Thread thread = new Thread(() -> {
            while (true) {
                synchronized (lock) {
                    while (printThreadId != threadId) {
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                    }

                    System.out.println(Thread.currentThread().getName() + ":" + integer++);
                    printThreadId++;
                    if (integer > threadNum) {
                        integer = 1;
                        printThreadId = 1;
                    }

                    lock.notifyAll();
                }
            }
        }, "Thread" + threadId);
        thread.start();
    }
}
```

* [掘金 - Java实现生产者和消费者的5种方式](https://juejin.cn/post/6844903486895865864)