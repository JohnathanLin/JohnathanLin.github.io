---
title: "Various Locking Mechanisms in Java Concurrency Programming"
date: 2023-12-04T17:27:44+08:00
draft: false
categories: ["Tech", "Note"]
---
> Introduction: This article aims to introduce the relevant usage of synchronized, ReentrantLock, and Condition in Java.

# Locking with Synchronized
Synchronized can be applied to instance methods, static methods, and code blocks. When used to modify a code block, it can either lock on a specific object or on a class (.class).

## Synchronized is a Non-Fair Lock
The following code utilizes synchronized to lock on a variable accessible by multiple threads, achieving the orderly printing of numbers. Additionally, it calculates the frequency of number printing for different threads:
```java
package com.windypath.lockcondition;

public class Syn {
    int count = 0;
    final Object sth = new Object();
    void play() {
        int loopTimes = 1000;
        SynThread t1 = new SynThread(loopTimes, "t1");
        SynThread t2 = new SynThread(loopTimes, "t2");
        SynThread t3 = new SynThread(loopTimes, "t3");
        SynThread t4 = new SynThread(loopTimes, "t4");
        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }

    public static void main(String[] args) {
        Syn syn = new Syn();
        syn.play();
    }
    class SynThread extends Thread {
        int loopTimes;

        public SynThread(int loopTimes, String threadName) {
            super(threadName);
            this.loopTimes = loopTimes;
        }
        @Override
        public void run() {
            int times = 0;
            while (count <= 200000) {
                synchronized (sth) {
                    count++;
//                    System.out.println(getName() + " 输出 " + count);
                    times++;
                }
            }
            System.out.println(getName() + "一共输出了 " + times + " 次");
        }
    }
}
```
Output：
```
t2一共输出了 103061 次
t1一共输出了 37174 次
t4一共输出了 33751 次
t3一共输出了 26018 次
```
It can be observed that the number of times thread t2 outputs is more than the sum of the other three threads. This is because synchronized is a non-fair lock.
## Synchronized's Waiting Queue
The waiting queue for an object locked with synchronized is located in the _waitSet of the ObjectMonitor. This ObjectMonitor is a low-level native (C/C++) component.

## Synchronized Lock Upgrade
However, heavyweight locks are not acquired right from the start. Instead, the optimization begins with biased locking. It only upgrades to lightweight locks in the presence of contention, and only when numerous threads are involved in lock contention does it escalate from a lightweight lock to a heavyweight lock.

The locked object uses the MarkWord in its object header to store lock information.


The storage structure of a Java object in memory consists of three parts:

- Object header
- Instance variables
- Padding bytes

Among them, the object header mainly stores some runtime data:
- MarkWord
- Class Metadata Address (points to the object type data)
- Array Length (if it's an array, it records the length)

The lock information is recorded in the MarkWord of the object header. The following diagram illustrates the different bits of information in the MarkWord for different locks:
![](/images/java_lock_bit_detail.jpg)


- Biased Lock
Biased locking is designed to avoid the resource consumption associated with higher-level locks, such as lightweight locks, when a lock is acquired in a non-multithreaded environment.

The term "biased" means that the locked object is biased toward a specific thread. Its object header stores the ID of the biased thread.

- Lightweight Lock
The lightweight lock is not intended to replace heavyweight locks. Its purpose is to reduce the performance overhead incurred by traditional heavyweight locks in the absence of multithreaded competition. Before explaining the execution process of the lightweight lock, it's essential to understand that the lightweight lock is suitable for scenarios where threads alternately execute synchronized blocks.

## Differences Between Lightweight Lock and Biased Lock
The process of acquiring a lightweight lock involves multiple CAS (Compare-And-Swap) operations, whereas a biased lock requires only one CAS operation.
Lightweight locks are suitable for scenarios where threads alternately execute synchronized blocks. In contrast, biased locks are designed to further improve performance when only one thread executes the synchronized block.

## Observation of Synchronized Lock Escalation
Attempt to use an object, and have multiple threads acquire and release a synchronized lock on it at different time intervals to observe its lock state.

thread1: Acquires and releases immediately.

thread2: Waits 500ms to acquire, then holds for 1500ms before releasing.

thread3: Waits 1000ms to acquire, then releases immediately.


At this point, there will be lock contention between thread2 and thread3.

Source Code：
```java
package com.windypath;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.openjdk.jol.info.ClassLayout;

/**
 * 
 * Observing the process of synchronized transitioning from biased lock to lightweight lock to heavyweight lock.
 * Using log4j2
 */

public class BiasdLock {
    final static Logger log = LogManager.getLogger();
    public static void main(String[] args) throws InterruptedException {
        log.debug(Thread.currentThread().getName() + "最开始的状态:\n"
                + ClassLayout.parseInstance(new Object()).toPrintable());
        // After starting, the HotSpot Virtual Machine has a 4-second delay before enabling biased locking mode for each newly created object.
        Thread.sleep(4000);
        // 创建一个对象，用于多个不同的线程上锁用
        Object obj = new Object();
        log.debug(Thread.currentThread().getName() + "等待4秒后的状态（新对象）:\n"
                + ClassLayout.parseInstance(obj).toPrintable());
        //线程1，马上上锁马上释放
        new Thread(() -> {
            log.debug(
                    Thread.currentThread().getName() + "开始执行准备获取锁:\n" + ClassLayout.parseInstance(obj).toPrintable());
            synchronized (obj) {
                log.debug(Thread.currentThread().getName() + "获取锁执行中:\n"
                        + ClassLayout.parseInstance(obj).toPrintable());
            }
            log.debug(Thread.currentThread().getName() + "释放锁:\n" + ClassLayout.parseInstance(obj).toPrintable());
        }, "thread1").start();
        // 线程2，等线程1释放锁后再上锁
        new Thread(() -> {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug(
                    Thread.currentThread().getName() + "开始执行准备获取锁:\n" + ClassLayout.parseInstance(obj).toPrintable());
            synchronized (obj) {
                log.debug(Thread.currentThread().getName() + "获取锁执行中:\n"
                        + ClassLayout.parseInstance(obj).toPrintable());
                try {
                    Thread.sleep(1500);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

            }
            log.debug(Thread.currentThread().getName() + "释放锁:\n" + ClassLayout.parseInstance(obj).toPrintable());
        }, "thread2").start();
        // 线程3，在线程2拥有锁的时候尝试上锁
        new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug(
                    Thread.currentThread().getName() + "开始执行准备获取锁:\n" + ClassLayout.parseInstance(obj).toPrintable());
            synchronized (obj) {
                log.debug(Thread.currentThread().getName() + "获取锁执行中:\n"
                        + ClassLayout.parseInstance(obj).toPrintable());
            }
            log.debug(Thread.currentThread().getName() + "释放锁:\n" + ClassLayout.parseInstance(obj).toPrintable());
        }, "thread3").start();
        //主线程等待所有线程运行结束，查看状态
        Thread.sleep(5000);
        log.debug(Thread.currentThread().getName() + "结束状态:\n" + ClassLayout.parseInstance(obj).toPrintable());
    }
}
```
Output(Briefly):
```
15:53:58.436 [main] main最开始的状态:non-biasable
15:54:02.854 [main] 等待4秒后的状态（新对象）:biasable
15:54:02.858 [thread1] thread1开始执行准备获取锁:biasable
15:54:02.858 [thread1] thread1获取锁执行中:biased
15:54:02.859 [thread1] thread1释放锁:biased
15:54:03.367 [thread2] thread2开始执行准备获取锁:biased
15:54:03.368 [thread2] thread2获取锁执行中:thin lock
15:54:03.869 [thread3] thread3开始执行准备获取锁:thin lock
15:54:04.872 [thread3] thread3获取锁执行中:fat lock
15:54:04.872 [thread2] thread2释放锁:fat lock
15:54:04.873 [thread3] thread3释放锁:fat lock
15:54:07.868 [main] main结束状态:non-biasable
```
The following conclusions can be drawn from the analysis:

- For the HotSpot Virtual Machine, objects created immediately after startup are non-biasable.
- Objects created after 4 seconds are biasable.
- When thread1 acquires the lock, as there is only one thread holding the synchronized lock for this object, it transitions to the biased lock state.
- When thread1 releases the lock, the lock object remains in the biased lock state and does not revert to biasable.
- After 500ms, when thread2 acquires the lock, the lock object's state is upgraded to a lightweight lock.
- Another 500ms later, when thread3 also attempts to acquire the lock, its state is lightweight lock (thin lock) until it executes the synchronized block. It then blocks until thread2 releases the lock, at which point it immediately acquires the lock (the timestamps in the third and fourth lines are exactly the same, both 15:54:04.872).
- Thread3 releases the lock immediately, at this moment, it is still a heavyweight lock.
- After the main thread waits for 5 seconds, the lock state is restored but becomes non-biasable.
- You can try commenting out the earlier 4-second delay; in this case, the object would immediately acquire a lightweight lock.

## Locking with ReentrantLock
ReentrantLock is a lightweight, reentrant lock. It can be specified as a fair lock during creation. ReentrantLock can be used in conjunction with Condition. It provides several functions related to concurrent programming, offering higher flexibility compared to synchronized.

- ReentrantLock supports whether the lock is fair.
- In addition to the regular lock() function, ReentrantLock also provides tryLock() for polling and lockInterruptibly() for interruptible locking.
- After acquiring a lock with ReentrantLock, you can wait on different conditions based on your business logic.

## ReentrantLock can be used as a fair lock.
```java
package com.windypath.lockcondition;

import java.util.concurrent.locks.ReentrantLock;

public class Reen {
    int count = 0;
    final ReentrantLock lock = new ReentrantLock(true);
    void play() {
        int loopTimes = 1000;
        ReenThread t1 = new ReenThread(loopTimes, "t1");
        ReenThread t2 = new ReenThread(loopTimes, "t2");
        ReenThread t3 = new ReenThread(loopTimes, "t3");
        ReenThread t4 = new ReenThread(loopTimes, "t4");
        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }

    public static void main(String[] args) {
        Reen reen = new Reen();
        reen.play();
    }
    class ReenThread extends Thread {
        int loopTimes;

        public ReenThread(int loopTimes, String threadName) {
            super(threadName);
            this.loopTimes = loopTimes;
        }
        @Override
        public void run() {
            int times = 0;

            while (count <= 200000) {
                try {
                    lock.lock();
                    count++;
//                    System.out.println(getName() + " 输出 " + count);
                    times++;
                } finally {
                    lock.unlock();
                }
            }
            System.out.println(getName() + "一共输出了 " + times + " 次");
        }
    }
}
```
Output:
```
t3一共输出了 49953 次
t4一共输出了 49988 次
t1一共输出了 50077 次
t2一共输出了 49986 次
```
It can be observed that the output of the four threads is generally around 50,000.

## ReentrantLock Simulating the Dining Philosophers Problem Without Using Condition
The dining philosophers problem involves five philosophers sitting around a circular table with only five chopsticks. After a philosopher finishes contemplating, they need to simultaneously acquire the chopsticks on their left and right sides to eat.

Here, we use threads to simulate philosophers and use ReentrantLock to simulate chopsticks.

```java
package com.windypath;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class DiningPhilosopher {
    public static void main(String[] args) {
        int numPhilosophers = 5;
        Philosopher[] philosophers = new Philosopher[numPhilosophers];
        Chopstick[] chopsticks = new Chopstick[numPhilosophers];

        for (int i = 0; i < numPhilosophers; i++) {
            chopsticks[i] = new Chopstick();
        }

        for (int i = 0; i < numPhilosophers; i++) {
            Chopstick leftChopstick = chopsticks[i];
            Chopstick rightChopstick = chopsticks[(i + 1) % numPhilosophers];
//            philosophers[i] = new Philosopher(i, leftChopstick, rightChopstick);
            if (i % 2 == 0) {
                philosophers[i] = new Philosopher(i, leftChopstick, rightChopstick);
            } else {
                philosophers[i] = new Philosopher(i, rightChopstick, leftChopstick);
            }
            Thread thread = new Thread(philosophers[i]);
            thread.start();
        }
    }

    static class Philosopher implements Runnable {
        private final int id;
        private final Chopstick leftChopstick;
        private final Chopstick rightChopstick;

        private int eatTimes = 0;

        public Philosopher(int id, Chopstick leftChopstick, Chopstick rightChopstick) {
            this.id = id;
            this.leftChopstick = leftChopstick;
            this.rightChopstick = rightChopstick;
        }

        private void think() throws InterruptedException {
            System.out.println("Philosopher " + id + " is thinking.");
            Thread.sleep((long) ( 1000));
        }

        private void eat() throws InterruptedException {
            leftChopstick.pickUp();
            rightChopstick.pickUp();

            System.out.println("Philosopher " + id + " picks up both chopsticks and eats.");
            Thread.sleep((long) ( 1000));
            System.out.println("Philosopher " + id + " puts down both chopsticks.");

            rightChopstick.putDown();
            leftChopstick.putDown();
            eatTimes++;
            if (eatTimes % 10 == 0) {
                System.out.println("Philosopher " + id + " 目前吃了" + eatTimes + "次");
            }
        }

        @Override
        public void run() {
            try {
                while (true) {
                    think();
                    eat();
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    static class Chopstick {
        private final Lock lock = new ReentrantLock();

        public void pickUp() {
            lock.lock();

        }

        public void putDown() {
            lock.unlock();
        }
    }
}
```

In the above code, the chopsticks only need to be locked by philosophers when picked up using the lock() function and unlocked when put down using the unlock() function to achieve the goal of "simultaneously having the chopsticks on the left and right sides."
### Code Using Condition:
```java
package com.windypath;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class DiningPhilosophers {
    public static void main(String[] args) {
        int numPhilosophers = 5;
        Philosopher[] philosophers = new Philosopher[numPhilosophers];
        Chopstick[] chopsticks = new Chopstick[numPhilosophers];

        for (int i = 0; i < numPhilosophers; i++) {
            chopsticks[i] = new Chopstick();
        }

        for (int i = 0; i < numPhilosophers; i++) {
            Chopstick leftChopstick = chopsticks[i];
            Chopstick rightChopstick = chopsticks[(i + 1) % numPhilosophers];
//            philosophers[i] = new Philosopher(i, leftChopstick, rightChopstick);
            if (i % 2 == 0) {
                philosophers[i] = new Philosopher(i, leftChopstick, rightChopstick);
            } else {
                philosophers[i] = new Philosopher(i, rightChopstick, leftChopstick);
            }
            Thread thread = new Thread(philosophers[i]);
            thread.start();
        }
    }

    static class Philosopher implements Runnable {
        private final int id;
        private final Chopstick leftChopstick;
        private final Chopstick rightChopstick;

        private int eatTimes = 0;

        public Philosopher(int id, Chopstick leftChopstick, Chopstick rightChopstick) {
            this.id = id;
            this.leftChopstick = leftChopstick;
            this.rightChopstick = rightChopstick;
        }

        private void think() throws InterruptedException {
            System.out.println("Philosopher " + id + " is thinking.");
            Thread.sleep((long) ( 1000));
        }

        private void eat() throws InterruptedException {
            leftChopstick.pickUp();
            rightChopstick.pickUp();
            
            System.out.println("Philosopher " + id + " picks up both chopsticks and eats.");
            Thread.sleep((long) ( 1000));
            System.out.println("Philosopher " + id + " puts down both chopsticks.");
            
            rightChopstick.putDown();
            leftChopstick.putDown();
            eatTimes++;
            if (eatTimes % 10 == 0) {
                System.out.println("Philosopher " + id + " 目前吃了" + eatTimes + "次");
            }
        }

        @Override
        public void run() {
            try {
                while (true) {
                    think();
                    eat();
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    static class Chopstick {
        private final Lock lock = new ReentrantLock(true);
        private final Condition condition = lock.newCondition();
        private boolean taken = false;

        public void pickUp() throws InterruptedException {
            lock.lock();
            try {
                while (taken) {
                    condition.await();
                }
                taken = true;
            } finally {
                lock.unlock();
            }
        }

        public void putDown() {
            lock.lock();
            try {
                taken = false;
                condition.signal();
            } finally {
                lock.unlock();
            }
        }
    }
}
```
You can see that the Chopstick class has an added taken status to determine whether the chopstick is currently possessed by a philosopher. When the first philosopher picks up a chopstick, taken becomes true, and the lock is released. When the second philosopher picks up the same chopstick, they still acquire the same lock but enter a waiting state with condition.await() due to taken being true. At this point, the lock is also released, allowing other philosophers to pick up this chopstick.

When the chopstick is put down, the signal() method is called, and the thread waiting at the previous await() function is awakened, allowing it to execute its subsequent logic.

> If a fair lock is not used, you might see that two philosophers eat very late in their 10 cycles. With a fair lock, almost all five philosophers synchronize and complete their 10 cycles.

> Note that when initializing philosophers, chopsticks for philosophers with odd numbers are swapped. This is because in the subsequent logic for picking up chopsticks, we always pick up the left chopstick first and then the right one. If we don't reverse the hands for some philosophers, there would be a situation where all five philosophers pick up the left chopstick simultaneously and then wait for the right one, leading to a deadlock. (Of course, we could also use random numbers to let philosophers decide whether to pick up the left or right chopstick first.)