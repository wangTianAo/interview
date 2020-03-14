# 操作系统
## 进程和线程的区别
* 进程是对运行时程序的封装，是系统进行资源调度和分配的的基本单位，实现了操作系统的并发
* 线程是进程的子任务，是CPU调度和分派的基本单位，用于保证程序的 实时性，实现进程内部的并发
* 一个程序至少有一个进程，一个进程至少有一个线程，线程依赖于进程而存在
* 进程在执行过程中拥有独立的内存单元，而多个线程共享进程的内存

## 进程间的通信的几种方式
* 管道（pipe）及命名管道（named pipe）：管道可用于具有亲缘关系的父子进程间的通信，有名管道除了具有管道所具有的功能外，它还允许无亲缘关系进程间的通信
* 信号（signal）：信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生
* 消息队列：消息队列是消息的链接表，它克服了上两种通信方式中信号量有限的缺点，具有写权限得进程可以按照一定得规则向消息队列中添加新信息；对消息队列有读权限得进程则可以从消息队列中读取信息
* 共享内存：可以说这是最有用的进程间通信方式。它使得多个进程可以访问同一块内存空间，不同进程可以及时看到对方进程中对共享内存中数据得更新。这种方式需要依靠某种同步操作，如互斥锁和信号量等
* 信号量：主要作为进程之间及同一种进程的不同线程之间得同步和互斥手段
* 套接字：这是一种更为一般得进程间通信机制，它可用于网络中不同机器之间的进程间通信，应用非常广泛

## 线程同步的方式
* 互斥量 Synchronized/Lock：采用互斥对象机制，只有拥有互斥对象的线程才有访问公共资源的权限。因为互斥对象只有一个，所以可以保证公共资源不会被多个线程同时访问
* 信号量 Semphare：它允许同一时刻多个线程访问同一资源，但是需要控制同一时刻访问此资源的最大线程数量
* 事件(信号)，Wait/Notify：通过通知操作的方式来保持多线程同步，还可以方便的实现多线程优先级的比较操作

## 什么是死锁？
* 在两个或者多个并发进程中，如果每个进程持有某种资源而又等待其它进程释放它或它们现在保持着的资源，在未改变这种状态之前都不能向前推进，称这一组进程产生了死锁。通俗的讲，就是两个或多个进程无限期的阻塞、相互等待的一种状态。

## 死锁产生的条件？
* 互斥：至少有一个资源必须属于非共享模式，即一次只能被一个进程使用；若其他申请使用该资源，那么申请进程必须等到该资源被释放为止
* 占有并等待：一个进程必须占有至少一个资源，并等待另一个资源，而该资源为其他进程所占有
* 非抢占：进程不能被抢占，即资源只能被进程在完成任务后自愿释放
* 循环等待：若干进程之间形成一种头尾相接的环形等待资源关系