---
title: synchronized & ReentrantLock 区别

tags: [Java]
authors: Flankx
description: 锁的区别

---

## `synchronized` 与 `ReentrantLock` 区别

| 类别 | synchronized | ReentrantLock |
|--|--|--|
| 存在层次 | Java的关键字，JVM层面 | 是一个类 |
| 锁的释放 | 1.以获取锁的线程执行同步代码，释放锁 2.线程执行发生一次，JVM会让线程释放锁|在finally中必须释放锁，不然容易造成线程死锁|
| 锁的获取 | 加锁A线程获得锁，B线程等待，如果A阻塞，B线程会一直等待|分情况而定，Lock有多种锁的获取方式 condition |
| 锁状态 |无法判断 | 可以判断|
| 锁类型 | 可重入，不可判断，非公平| 可重入，可判断，可公平|
| 性能 | 少量同步 | 大量同步 |

## 锁的状态

### 锁状态：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态，这几个状态会随着竞争情况逐渐升级。锁可以升级但不能降级

![Alt text](../_media/lock-1.webp)

### 各种锁的比较

![Alt text](../_media/lock-2.webp)
