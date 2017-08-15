---
uuid: 4eaa11c0-3bd9-11e7-b0d0-2307cdf5da87
title: redis和memcached
date: 2017-05-18 22:50:17
tags: nosql
---

memcached是多线程，非阻塞IO复用的网络模型，分为监听主线程和worker子线程，监听线程监听网络连接，接受请求后，将连接描述字pipe传递给worker线程，进行读写IO，网络层使用libevent封装的事件库，多线程模型可以发挥多核作用，但是引入了cache coherency和锁的问题，比如：memcached最常用的stats命令，实际memcached所有操作都要对这个全局变量加锁，进行技术等工作，带来了性能损耗。

redis使用单线程的IO复用模型，自己封装了一个简单的AeEvent事件处理框架，主要实现了epoll, kqueue和select，对于单存只有IO操作来说，单线程可以将速度优势发挥到最大，但是redis也提供了一些简单的计算功能，比如排序、聚合等，对于这些操作，单线程模型施加会严重影响整体吞吐量，CPU计算过程中，整个IO调度都是被阻塞的。

*参考*[Redis 和 Memcached 的区别](http://blog.jobbole.com/101496/)
