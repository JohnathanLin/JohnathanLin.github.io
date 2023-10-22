---
title: "Java并发编程中上锁的几种方式"
date: 2023-09-05T20:52:44+08:00
draft: true
categories: ["技术文章", "笔记"]
---
> 前言：本文想要介绍Synchronized，ReentrantLock和ReentrantLock的Condition的相关用法。

# Synchronized上锁
Synchronized可以修饰实例方法、静态方法和代码块。修饰代码块时，可以对具体的对象上锁，也可以对某个类(.class)上锁。

以下代码是通过给一个多线程能访问到的变量使用synchronized进行上锁，实现有序打印数字的功能。并且在最后会统计不同线程打印数字的次数：
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
输出结果如下：
```
t2一共输出了 103061 次
t1一共输出了 37174 次
t4一共输出了 33751 次
t3一共输出了 26018 次
```
可以看到线程t2输出的次数比其他三个线程加起来还要多。因为synchronized是非公平锁。
## synchronized的等待队列
使用synchronized上锁的对象的等待队列位于ObjectMonitor中的_waitSet。这个ObjectMonitor是底层native(也就是C/C++)的内容。

## synchronized锁升级
TODO 偏向锁 轻量级锁 重量级锁 
https://blog.csdn.net/F_Hello_World/article/details/104666734
https://www.cnblogs.com/paddix/p/5405678.html
https://blog.csdn.net/weixin_43899792/article/details/124634419
但并不是一开始就上重量级锁，而是先优化成偏向锁，如有竞争才会升级为轻量级锁，大量的线程参与锁的竞争时，才会从轻量级锁升级到重量级锁。

上锁的对象使用其对象头中的MarkWord来存储锁的信息。

一个Java对象在内存中的存储结构包括三个部分：
- 对象头
- 实例变量
- 填充字节

其中对象头中主要存储一些运行时的数据：
- MarkWord 
- Class Metadata Address (指向对象类型数据的指针)
- Array Length (是数组的话，记录长度)

锁的信息记录在对象头的MarkWord中。下图是不同的锁的MarkWord的不同位的信息：
![](/images/java_lock_bit_detail.jpg)


- 偏向锁（biased lock）
偏向锁是为了避免在非多线程环境下，执行synchronized上锁时使用轻量级锁等更高等级的锁消耗资源。

偏向的意思是，被上锁的对象偏向于某个线程。其对象头会存储偏向的线程id。


- 轻量级锁（lightweight lock）
https://zhuanlan.zhihu.com/p/141554048 
TODO 轻量级锁加锁过程
