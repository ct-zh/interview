# 补充
### 内核态和用户态的划分与切换
### 进程虚拟空间分布，全局变量放哪里？
### 操作系统内存管理？进程通讯，为什么共享存储区效率最高?
### ipc方式，共享存储区原理
### 虚拟地址怎么映射到物理地址
### 线程的栈在哪里分配
### 多个线程读，一个线程写一个int32会不会有问题，int64呢
>（这里面试官后来说了要看数据总线的位数，32位的话写int32没问题，int64就有问题）
### 简述 IO 多路复用

## goroutine
### 对goroutine的理解, 讲一下goroutine的实现方式
### 什么是协程泄露(Goroutine Leak)？
### Go可以限制运行时操作系统线程的数量吗
### goroutine的调度是怎样的？
### goroutine 和 kernel thread 之间是什么关系？
### goroutine超时如何处理?

## channel
### channel的死锁与解决方法; 
### 如何用channel实现一个令牌桶？
### 退出程序时怎么防止channel没有消费完
### 用channel实现定时器？（实际上是两个协程同步）
### 简述channel, buffer channel, unbuffer channel; 单向channel与双向channel; select多路监听
### channel的实现方式;为什么channel可以做到线程安全?; mutex和channel作并发控制你喜欢用哪个，哪个快，为什么? 自己设计一个mutex. 

## lock
### 了解读写锁吗，原理是什么样的，为什么可以做到？
### go的锁如何实现，用了什么cpu指令
