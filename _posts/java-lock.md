---
title: Java AQS疑问
date: 2019-04-24 21:39:03
tags: Java
---
Java AQS源码阅读疑问
<!-- more -->
学习Java锁（Lock），阅读AQS源码一直有个疑问，如下：
```java
final boolean acquireQueued(final Node node, int arg) {
	boolean failed = true;
	try {
		boolean interrupted = false;
		for (;;) {
			final Node p = node.predecessor();
			if (p == head && tryAcquire(arg)) {
				setHead(node);
				p.next = null; // help GC
				failed = false;
				return interrupted;
			}
			if (shouldParkAfterFailedAcquire(p, node) &&
				parkAndCheckInterrupt())
				interrupted = true;
		}
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}
// release方法中unparkSuccessor方法代码
private void unparkSuccessor(Node node) {
	int ws = node.waitStatus;
	if (ws < 0)
		compareAndSetWaitStatus(node, ws, 0);

	Node s = node.next;
	if (s == null || s.waitStatus > 0) {
		s = null;
		for (Node t = tail; t != null && t != node; t = t.prev)
			if (t.waitStatus <= 0)
				s = t;
	}
	if (s != null)
		LockSupport.unpark(s.thread);
}
```
如果第一个线程获得了锁，第二个线程执行以上方法添加到队列里阻塞自己，但是当第二个线程执行到`shouldParkAfterFailedAcquire()`该方法时，返回true，在执行`parkAndCheckInterrupt()`阻塞自己之前，cpu时间片切换，第一个线程执行`release()`方法，以上代码中`Node s = node.next`为第二个线程的节点，执行`LockSupport.unpark(s.thread)`操作第二个线程，然后cpu时间片再次切换，第二个线程调用`parkAndCheckInterrupt()`阻塞自己，这样第二个线程会一直阻塞下去吗？

后来试了下发现，如果一个线程先调用`LockSupport.unpark`方法，然后调用`LockSupport.park`是阻塞不了的。
```java
// main方法测试
public static void main(String[] args) {
	LockSupport.unpark(Thread.currentThread());
	LockSupport.park();
	System.out.println(1); // 不阻塞，输出1
	LockSupport.park();
	System.out.println(2); // 阻塞，不输出
}
```

由于水平有限，本文有错误请在[Github Issue](https://github.com/wuare/wuare.github.io/issues)区留言指正。