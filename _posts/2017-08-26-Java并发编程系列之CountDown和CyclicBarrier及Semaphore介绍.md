---
layout: post
title:  "java并发编程系列之CountDownLatch,CyclicBarrier,Semaphore介绍"
keywords: "java,并发编程,CountDownLatch,CyclicBarrier,Semaphore"
description: "java.util.corrurnent包里面的CountDownLatch,CyclicBarrier,Semaphore使用介绍"
category: java并发
tags: [java]
---

### 吐槽一下自己
发现从2017年1月1号至现在自己有些懒散，虽然工作比较忙，但是不能成为不写博客的理由，自己得好好反省一下，写博客得坚持下去，写博客的目的不仅让自己的知识能沉淀下来，还能帮助别人，做一名乐于分享的技术人，好了，吐槽到此结束，下面进入正题。
### CountDownLatch,CyclicBarrier,Semaphore介绍
#### 1.CountDownLatch
利用它可以实现类似计数器的功能，初始值为线程的数量，每当一个线程完成自己的任务后，计数器的值就会减1，当计数器的值到达0时，表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复继续执行任务。

```
主要方法:
1.countDown方法，当前线程调用此放方法计数减一。
2.await方法，调用该方法会一直阻塞当前线程，直到计时器的值为0。
使用场景:
比如有一个任务，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。
```
#### 2.CyclicBarrier
它允许一组线程互相等待，直到达到某个公共屏障点。

```
主要方法:
1.await方法,调用该方法的线程被阻塞，直至所有线程都到达barrier状态再同时执行后续任务。
使用场景:
比如有一个大型的任务，需要分配好多子任务去执行，只有当所有子任务都执行完成时候，才能执行主任务。
```
#### 3.Semaphore
控制同时访问的线程个数。

```
主要方法:
1.acquire方法，获取许可，若没有许可获得，则一直等待，直到获得许可。
2.release方法，是否许可，但是需要注意在释放许可之前必须先获取许可。
3.tryAcquire方法，获取一个许可，若获取成功，则立即返回true,否则返回false。
使用场景:
一般用于控制对某组资源的访问权限。
```
#### 4.三者的异同
1.CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行。

2.CountDownLatch是不能够重用的，而CyclicBarrier被叫做回环栅栏，当所有等待线程都被释放以后，可以被重用。

3.锁有点类似，它一般用于控制对某组资源的访问权限

