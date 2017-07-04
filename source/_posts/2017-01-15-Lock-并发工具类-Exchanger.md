---
title: 'Lock: 并发工具类-Exchanger'
date: 2017-01-15 17:11:35
categories:
- Java锁
tags:
- 线程
- 锁
- Lock
- Exchanger
---

Exchanger是一个用于线程间协作的工具类，可以进行**线程间的数据交换**。
它提供一个**同步点**，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过`exchange(V x)`方法交换数据，如果第一个线程先执行`exchange(V x)`方法，它会一直等待第二个线程也执行`exchange(V x)`方法，当两个线程同时到达同步点时，这两个线程就可以交换数据。
