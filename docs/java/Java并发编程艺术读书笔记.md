# Java并发编程的艺术阅读笔记
# 第2章 Java并发机制的底层实现原理
## 1、volatile的应用
### 1.1、volatile的定义与实现原理
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622086505121-73e64d31-1676-49e3-add2-a8c87f14bd14.png#align=left&display=inline&height=392&margin=%5Bobject%20Object%5D&name=image.png&originHeight=392&originWidth=825&size=245984&status=done&style=shadow&width=825)

1. 实现原则
   - Lock前缀指令会引起处理器缓存回写到内存
   - 一个处理器的缓存回写到内存会导致其他处理器的缓存无效
2. 使用优化
   - 队列集合类`LinkedTransferQueue`
> 追加字节

   - 追加到64字节
> 如果队列的头节点和尾节点都不足64字节的话，**处理器会将它们都读到同一个高速缓存行中**，在多处理器下每个处理器都会缓存同样的头、尾节点，当一个处理器试图修改头节点时，会将整个缓存行锁定，那么在缓存一致性机制的作用下，会导致其他处理器不能访问自己高速缓存中的尾节点，而队列的入队和出队操作则需要不停修改头节点和尾节点，所以在多处理器的情况下将会严重影响到队列的入队和出队效率。Doug lea使 用追加到64字节的方式来填满高速缓冲区的缓存行，避免头节点和尾节点加载到同一个缓存行，使头、尾节点在修改时不会互相锁定。

   - 不被追加的情况
      - 缓存行非64字节宽
      - 共享变量不被频繁地写



## 2、synchronized的实现原理与应用

1. 实现同步的基础
   - 普通方法，锁对象
   - 静态方法，锁类
   - 方法块，锁括号内配置的对象
2. JVM级别的实现原理
   - 进入和退出`monitor`
      - 代码块同步
         - `monitor enter`（开始位置）
         - `monitor exit`（结束处和异常处）
> 任何对象有一个monitor与之关联，当且一个monitor被持有后，它将处于锁定状态。
> **线程执行到monitorenter指令时，会尝试获取对象所对应的monitor的所有权，即将尝试获得对象的锁**

      - 方法同步（另一种方法，细节在JVm规范未详细说明



### 2.1、Java对象头
`synchronized `用的锁存在Java对象头
对象是

- 数组类型 3个子宽存储对象头
- 非数组类型 2字宽存储

在31位虚拟机中， 1 字宽 = 4 字节 = 32bit
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622097384229-2cd60474-9b38-4b8c-88cc-7554cb47f481.png#align=left&display=inline&height=155&margin=%5Bobject%20Object%5D&name=image.png&originHeight=155&originWidth=708&size=45809&status=done&style=shadow&width=708)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622097482324-ab8d2e64-a6d3-46c2-b31c-30d4497d85b5.png#align=left&display=inline&height=114&margin=%5Bobject%20Object%5D&name=image.png&originHeight=114&originWidth=759&size=38315&status=done&style=shadow&width=759)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622097590732-b82b6180-f1ca-4181-bea9-ca738cd402dc.png#align=left&display=inline&height=186&margin=%5Bobject%20Object%5D&name=image.png&originHeight=186&originWidth=755&size=56443&status=done&style=shadow&width=755)
### 2.2、锁的升级与对比
锁的状态

- 无锁
- 偏向锁
- 轻量级锁
- 重量级锁

锁可以升级但不能降级

1. 偏向锁
   - 提出背景：锁总是由同一线程多次获得
> 当一个线程访问同步块并 获取锁时，会在对象头和栈帧中的锁记录里**存储锁偏向的线程ID**，以后该线程在进入和退出 同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下**对象头的Mark Word里是否 存储着指向当前线程的偏向锁**。如果测试成功，表示线程已经获得了锁。如果测试失败，则需 要再测试一下Mark Word中偏向锁的标识是否设置成1（表示当前是偏向锁）：如果没有设置，则 使用CAS竞争锁；如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程。

   - 偏向锁的撤销
> 使用的机制：**等到出现竞争才释放锁**
> 需要等待全局安全点（在这个时间点上没有正在执行的字节码）

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622098344302-384bf7e2-9c9f-4177-9d75-b21791fb47af.png#align=left&display=inline&height=853&margin=%5Bobject%20Object%5D&name=image.png&originHeight=853&originWidth=760&size=260508&status=done&style=shadow&width=760)

   - 关闭偏向锁
> 偏向锁在Java 6和Java 7里是默认启用的，但是它在应用程序启动几秒钟之后才激活，如 有必要可以使用JVM参数来关闭延迟：-XX:BiasedLockingStartupDelay=0。如果你确定应用程 序里所有的锁通常情况下处于竞争状态，**可以通过JVM参数关闭偏向锁：-XX:- UseBiasedLocking=false，那么程序默认会进入轻量级锁状态。**

2. 轻量级锁
   - 加锁
> 线程在执行同步块之前，**JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并 将对象头中的Mark Word复制到锁记录中**，官方称为Displaced Mark Word。**然后线程尝试使用 CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。**

   - 加锁
> 轻量级解锁时，会使用原子的CAS操作将Displaced Mark Word替换回到对象头，如果成 功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。图2-2是 两个线程同时争夺锁，导致锁膨胀的流程图

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622098761862-412b9cbc-666f-497c-8a9f-7ed381f9c31b.png#align=left&display=inline&height=784&margin=%5Bobject%20Object%5D&name=image.png&originHeight=784&originWidth=747&size=287077&status=done&style=shadow&width=747)

3. 锁的优缺点对比

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622098893042-7b76c73a-e4e5-4ea5-a8a5-f4ed07afc35c.png#align=left&display=inline&height=224&margin=%5Bobject%20Object%5D&name=image.png&originHeight=224&originWidth=758&size=111601&status=done&style=shadow&width=758)




## 2.3、原子操作的实现原理

- 总线锁
> 所谓总线锁就是使用处理器提供的一个 LOCK＃信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住，那么该 处理器可以独占共享内存。

- 缓存锁（对总线锁的优化）



## 3、Java如何实现原子操作

1. 使用循环CAS实现原子操作
1. CAS实现原子操作的三大问题
   - ABA问题
      - 解决思路：使用版本号
         - AtomicStampedReference
   - 循环时间长开销大
   - 只能保证一个共享变量的原子操作
      - 把多个变量放在一个对象里进行CAS操作
3. 使用锁机制实现原子操作
> 锁机制保证了只有获得锁的线程才能够操作锁定的内存区域。JVM内部实现了很多种锁 机制，有偏向锁、轻量级锁和互斥锁**。有意思的是除了偏向锁，JVM实现锁的方式都用了循环 CAS**，即当一个线程想进入同步块的时候使用循环CAS的方式来获取锁，当它退出同步块的时 候使用循环CAS释放锁。



# 第3章 Java内存模型
## 1、Java内存模型的基础
### 1.1、并发编程模型的两个关键问题

1. 线程之间如何通信
   - 共享内存
   - 消息传递
2. 线程之间如何同步
> **同步是指程序中用于控制不同线程间操作发生相对顺序的机制。**在共享内存并发模型 里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。 在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。

### 1.2、Java内存模型的抽象结构
Java线程之间的通信由**Java内存模型（JMM）**控制
从抽象的角度来看，JMM定义了线程和主内存之间的抽 象关系：线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地 内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的 一个抽象概念，并不真实存在。它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优 化。Java内存模型的抽象示意如图3-1所示。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622100723301-123376a5-174e-46b0-96a3-7f3b25e2c461.png#align=left&display=inline&height=679&margin=%5Bobject%20Object%5D&name=image.png&originHeight=679&originWidth=742&size=136776&status=done&style=shadow&width=742)
### 1.3、从源代码到指令序列的重排序
为了提高性能，**编译器和处理器常常会对指令做重排序**

- 编译器优化的重排序
- 指令级并行的重排序
- 内存系统的重排序

从Java源代码到最终实际执行的指令序列，会分别经历下面3种重排序，如图3-3所示
1属于编译器重排序，2和3属于处理器重排序
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622100914932-169b7f55-4de6-46f3-be1b-f1a446c44971.png#align=left&display=inline&height=133&margin=%5Bobject%20Object%5D&name=image.png&originHeight=133&originWidth=738&size=45662&status=done&style=shadow&width=738)
> 对于编译器，**JMM的编译器重排序规则会禁止特定类型的编译器重排序**（不是所有的编译器重排序都要禁止）。对于处理器重排序，**JMM的处理器重排序规则会要 求Java编译器在生成指令序列时，插入特定类型的****内存屏障****（Memory Barriers，Intel称之为 Memory Fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序。**

### 1.4、并发编程模型的分类
为了保证内存可见性，Java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。
### 1.5、happens-before
两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个 操作之前执行！happens-before**仅仅要求前一个操作（执行的结果）对后一个操作可见**，且前一 个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）。 happens-before的定义很微妙，后文会具体说明happens-before为什么要这么定义。
## 2、重排序
重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段。
### 2.1、数据依赖性
如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间 就存在数据依赖性。这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作， 不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。
### 2.2、as-if-serial语义
为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因 为这种重排序会改变执行结果
## 3、顺序一致性
## 4、volatile的内存语义
### 4.1、volatile的特性
把对volatile变量的单个读/写，看成是使用同一个锁对这 些单个读/写操作做了同步
锁的`happens-before`规则**保证释放锁和获取锁的两个线程之间的内存可见性**，这意味着对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入

- 可见性：对一个volatile变量的读，总是能看到任意线程对这个volatile变量最后的写入
- 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不 具有原子性。
### 4.2、volatile写-读建立的happens-before关系
volatile的写-读与锁的释放-获取有相同的内存效果：**volatile写和 锁的释放**有相同的内存语义；**volatile读与锁的获取**有相同的内存语义。






