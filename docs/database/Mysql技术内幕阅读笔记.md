# MySQL
# 第1章  MySQL体系结构和存储引擎

## 1、MySQL体系结构
<img src="https://cdn.nlark.com/yuque/0/2021/png/2674809/1621840698147-6b8b1ba2-d7cc-4b7c-b791-d816a71e837b.png#align=left&display=inline&height=512&originHeight=512&originWidth=795&size=241231&status=done&style=shadow&width=795" alt="image.png" style="zoom:80%;" />
组成部分

- 连接池组件
- 管理服务和工具组件
- SQL接口组件
- 查询分析器组件
- 优化器组件
- 缓冲组件
- **插件式存储引擎**
> 存储引擎是基于表的，而不是数据库

- 物理文件
## 2、MySQL存储引擎
> MySQL体系结构的核心

1. InnoDB存储引擎
1. MyISAM存储引擎（5.5.8版本之前默认）
> 缓冲池只缓冲索引文件，而不缓冲数据文件

3. NDB存储引擎
> 集群存储引擎，数据放在内存中

等等


## 3、链接MySQL

1. TCP/IP
1. 命名管道和共享内存
1. UNIX域套接字





# 第2章  InnoDB存储引擎
## 1、概述
从MySQL5.5版本开始时默认的表存储引擎（之前的版本仅在Windows默认）
## 2、版本
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621842624872-76176f85-87c1-40a0-8989-09784c5cbbcd.png#align=left&display=inline&height=201&originHeight=201&originWidth=808&size=55335&status=done&style=shadow&width=808)
## 3、InnoDB体系结构


![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621842738551-f6b33b9b-124f-4631-8747-008b37cf8359.png#align=left&display=inline&height=418&originHeight=418&originWidth=735&size=55164&status=done&style=shadow&width=735)
### 3.1、后台线程
后台线程：负责刷新内存池中的数据，保证缓存的是最近的数据
InnoDB是**多线程的模型**，因此后台有多个不同的后台线程，负责处理不同的任务

1. Master Thread
> **负责将缓冲池中的数据异步刷新到磁盘**，保证数据的一致性，包括脏页的刷新、合并插入缓冲、UNDO页的回收

2. IO Thread
> 大量使用了AIO来**处理写IO请求**，极大提高数据库的性能
> IO Thread的工作主要是负责这些IO请求的回调处理

3. Purge Thread
> 事务提交之后，所使用的undolog可能不再需要，因此需要Purge Thread**回收已经使用并分配的undo页**

4. Page Cleaner Thread
> 将之前版本中**脏页的刷新操作都放入到单独的线程**中来完成，目的是减轻原Master Thread的工作及对用户查询线程的阻塞

### 3.2、内存

1. 缓冲池
> InnoDB是**基于磁盘存储**，**按页的方式进行管理**；
> 缓冲池来处理CPU速度和磁盘速度之间的鸿沟，提高性能
> 就是一块内存区域，通过内存的速度弥补
> 页从缓冲池刷新回磁盘的操作不是在每次页发生更新时触发，而是用CheckPoint机制

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621843942475-5f011ed9-efbe-482c-9fa1-3e539d2c9076.png#align=left&display=inline&height=288&originHeight=288&originWidth=709&size=55800&status=done&style=shadow&width=709)

2. LRU List、Free List和Flush List

**数据库中的缓冲池通过LRU算法进行管理**，InnoDB的大小默认是16KB，稍微不同的是对传统的LRU算法做了一些优化，在InnoDB的存储引擎中，LRU列表中加入了midpoint位置，新读到的页放入midpoint位置（默认在列表长度的5/8处）
**midpoint insertion strategy**
midpoint之前的为new列表，之后的为old列表
**防止热点数据被刷出**
脏页：缓冲池的页和磁盘的页的数据产生了不一致——> Flush List


3. 重做日志缓冲
> Master Thread每1秒将 **重做日志缓冲刷新到重做日志文件;**
> 每个事务提交时会将重做日志缓冲刷新到重做日志文件;
> **当重做日志缓冲池剩余空间小于1/2时，重做日志缓冲刷新到重做日志文件**。

3. 额外的内存池
> InnoDB对内存的管理是通过**内存堆**的方式进行的



## 4、Checkpoint技术
解决问题：

- 缩短数据库的恢复时间;
- 缓冲池不够用时，将脏页刷新到磁盘;
- 重做日志不可用时，刷新脏页。



分类：

- Sharp Checkpoint
> 在数据库关闭时将所有的脏页都刷新回磁盘

- Fuzzy Checkpoint
> 只刷新一部分



## 5、Master Thread工作方式

1. 1.0.x版本之前
> 内部多个循环：
> - 主循环
> - 后台循环
> - 刷新循环
> - 暂停循环

。。。
## 6、InnoDB关键特性
### 6.1、插入缓冲

1. Insert buffer
> 数据结构是一颗B+树，非叶子节点存放的是search key（键值）
> ![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621847670768-80085252-e706-45cb-9968-6b9029d726f8.png#align=left&display=inline&height=99&originHeight=99&originWidth=497&size=12627&status=done&style=none&width=497)

对于非聚集索引的插入或者更新操作，不是每次直接插入到索引页中，而是先判断插入的非聚集索引页是否在缓冲池中，若在则直接插入；若不在，则先放入一个Insert Buffer对象中。然后再以一定的频率和情况进行Insert buffer 和辅助索引页子节点的merge操作。

- 索引是辅助索引
- 索引不是唯一的
2. Change buffer

非唯一的辅助索引，1的升级

3. Delete buffer
3. Merge Insert Buffer
辅助索引页被读取到缓冲池时;> Insert Buffer Bitmap页追踪到该辅助索引页已无可用空间时;
> Master Thread.

### 6.2、两次写
数据页的可靠性
组成部分

- 内存中的doublewrite buffer，大小为2MB
- 物理磁盘中共享表空间中连续的128个页，即2个区，大小2MB

在对**缓冲池的脏页进行刷新时，不直接写入磁盘，而是先将脏页复制到内存中的doublewrite buffer**，之后分两次，每次1MB顺序写入共享表空间的物理磁盘上，然后马上**调用fsync函数，同步磁盘**，避免缓冲写带来的问题
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621848074948-5e9cb7a9-ed6d-474d-8f74-d2cb0a9a0384.png#align=left&display=inline&height=383&originHeight=383&originWidth=564&size=51606&status=done&style=shadow&width=564)
### 6.3、自适应哈希索引AHI
时间复杂度为O(1)，B+树的查找次数取决于B+树的高度，一般3到4层
### 6.4、异步IO
异步io
优势：可以进行IO Merge操作，可以提高IOPS的性能
### 6.5、刷新邻接页
工作原理：
刷新一个脏页时，InnoDB存储引擎会检测该页所在区的所有页，如果是脏页，则一起刷新
# 第3章 文件
## 1、参数文件
mysql--help| grep my.cnf

1. 参数——键值对
1. 参数类型
   1. 动态参数
   1. 静态参数（不得进行更改）
## 2、日志文件
记录了影响MySQL数据库的各种类型活动
### 2.1、错误日志
记录启动运行和关闭过程，记录错误信息和警告信息以及正确的信息
> SHOW VARIABLES LIKE 'log_ error'

### 2.2、慢查询日志
设置阈值`long_query_time`，运行时间超过（不等于）阈值的所有SQL语句加入到慢查询日志文件中
走索引
磁盘读取次数等
### 2.3、查询日志
记录了所有对MySQL数据库请求的信息；
默认文件名：主机名.log
### 2.4、二进制日志
binary log记录了对MySQL数据库执行更改的所有操作，但是不包括SELECT和SHOW这类操作，因为这类操作对数据本身并没有修改。
作用

- 恢复
- 复制
- 审计：判断是否有对数据库进行注入的攻击

配置log-bin[=name]可以启动二进制日志
开启只会是性能下降1%


当使用InnoDB时，所有未提交的二进制日志会记录到一个缓存中去，等待该事务提交时直接将缓冲中的二进制日志写入二进制日志文件，缓冲大小为32K


## 3、套接字文件
## 4、pid文件
MySQL实例启动时，将自己的进程ID写入一个文件中，该文件为pid文件，主机名.pid
## 5、表结构定义文件
.frm的文件记录了该表的表结构定义
## 6、InnoDB存储引擎文件
之前的文件都是数据库本身的文件，与存储引擎无关
### 6.1、表空间文件
初始大小10M
名字;ibdata1
将存储的数据按表空间进行存放的设计
### 6.2、重做日志文件
**redo log file**

- 记录了对于InnoDB存储引擎的事务日志
- 当实例或介质失败时派上用场
- 每个存储引擎**至少有一个重做日志文件组，每组下至少两个重做日志文件。**
- 为了提高可靠性，设置多个镜像日志组，放在不同的磁盘上
- 先写文件1，再写2，循环切换
- 不能设置过大，不然在恢复时需要很长时间
- InnoDB的**重做日志文件记录的是关于每个页的更改的物理情况**。
- **写入重做日志文件的操作不是直接写，而是先写入一个重做日志缓冲中，再按照一定条件顺序地写入日志文件**
> redo log是InnoDB存储引擎层的日志，又称重做日志文件，用于记录事务操作的变化，记录的是数据修改之后的值，不管事务是否提交都会记录下来。在实例和介质失败（media failure）时，redo log文件就能派上用场，如数据库掉电，InnoDB存储引擎会使用redo log恢复到掉电前的时刻，以此来保证数据的完整性。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621868726917-47a7d1c9-d562-4029-82af-c6d7383c5e2d.png#align=left&display=inline&height=255&originHeight=255&originWidth=409&size=47604&status=done&style=shadow&width=409)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621869057734-32e4da73-53c7-4b7c-a64c-fa7e1cc86244.png#align=left&display=inline&height=232&originHeight=232&originWidth=534&size=47149&status=done&style=shadow&width=534)
> 补充

### 6.3、redolog和binlog的区别

- redo log是属于innoDB层面，binlog属于MySQL Server层面的，这样在数据库用别的存储引擎时可以达到一致性的要求。
- redo log是物理日志，记录该数据页更新的内容；binlog是逻辑日志，记录的是这个更新语句的原始逻辑
- redo log是循环写，日志空间大小固定；binlog是追加写，是指一份写到一定大小的时候会更换下一个文件，不会覆盖。
- binlog可以作为恢复数据使用，主从复制搭建，redo log作为异常宕机或者介质故障后的数据恢复使用。
> undo log
> 保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读



# 第4章 表
## 1、索引组织表
在InnoDB存储引擎中，**表都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表**，每个表都有个主键
当表中有多个非空唯一索引时，**InnoDB 存储引擎将选择建表时第一个定 义的非空唯一索引为主键。**这里需要非常注意的是，主键的选择根据的是定义索引的顺序。


## 2、InnoDB逻辑存储结构
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621869618293-17a693c9-9a98-4114-a217-c1452494825c.png#align=left&display=inline&height=424&originHeight=424&originWidth=633&size=141319&status=done&style=shadow&width=633)
所有数据都被逻辑地存放在一个空间中，即表空间
表空间

- 段
   - 数据段 B+树的叶子节点
   - 索引段 B+树的非叶子节点
   - 回滚段
- 区 由连续页组成的空间
- 页（块 每个页大小为16KB
   - 数据页
   - undo页
   - 系统页
   - 事务数据页
   - 插入缓冲位图页
   - 插入缓冲空闲列表页
   - 未压缩的二进制大对象页
   - 压缩的二进制大对象页
- 行



## 3、InnoDB行记录格式

1. Compact 5.0之后

行数据越多，性能就越高

2. Redundant 5.0之前
2. 行溢出数据





## 4、InnoDB数据页结构
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621870751259-256921e7-e367-4b7d-bc7d-2846ed8fdf14.png#align=left&display=inline&height=433&originHeight=433&originWidth=477&size=142748&status=done&style=shadow&width=477)
## 5、Named File Formats 机制


## 6、约束
### 6.1、数据完整性

1. 实体完整性：保证表中有一个主键
1. 域完整性：保证数据每列的值满足特定的条件
1. 参照完整性：保证两个表之间的关系
### 6.2、约束的创建和查找
### 6.3、约束和索引的区别

- 约束是一个逻辑的概念，保证数据的完整性
- 索引是一个数据结构，既有逻辑上的概念，在数据库中还代表着物理存储的方式
### 6.4、触发器和约束
### 6.5、外键约束
外键用来保证参照完整性


## 7、视图
## 8、分区表


# 第5章 索引与算法
## 1、InnoDB存储引擎索引概述
常见索引

- B+树索引
> 并不能找到一个给定键值的具体行，找到只是被查找数据行所在的页，然后数据库把页读入到内存，再在内存中进行查找，最后得到要查找的数据

- 全文索引
- 哈希索引（自适应）
> B+树的B是Balance

## 2、数据结构与算法
### 2.1、二分查找法
### 2.2、二叉查找树和平衡二叉树
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621929420537-f1d278fe-309b-47ee-aa84-bb60ba2e0c6e.png#align=left&display=inline&height=158&originHeight=158&originWidth=169&size=8153&status=done&style=shadow&width=169)
**二叉查找树**的中序遍历为递增
二叉查找树的弊端就是容易退化成线性查找


**平衡二叉树：**
首先符合二叉查找树的定义，其次满足任何节点的两个子树的高度最大差为1
查询速度很快，维护的代价很大（左旋，右旋）


## 3、B+树
B+树是为磁盘或其他直接存取辅助设备设计的一种平衡查找树
所有**记录节点**都是按键值的大小顺序存放在同一层的叶子节点上
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621929761040-f5af08d1-ce96-4962-b5c7-ddb12185680a.png#align=left&display=inline&height=281&originHeight=281&originWidth=658&size=52065&status=done&style=shadow&width=658)
### 3.1、插入操作
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621929847012-410c2689-c1d9-41ae-b752-b1bcb6db6653.png#align=left&display=inline&height=267&originHeight=267&originWidth=595&size=52611&status=done&style=shadow&width=595)
**如果叶子节点满了 叶节点的中间节点放到上一次Index page**
**
### 3.2、删除操作
填充因子最小值设置50%
**![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621930046769-6452f8b4-97c3-460e-8040-993f64231a94.png#align=left&display=inline&height=169&originHeight=169&originWidth=624&size=40341&status=done&style=shadow&width=624)**
## 4、B+树索引
在数据库中， B+树的高度一般都在2~4层
B+树索引分为：

- 聚集索引
- 辅助索引
> 区别在于叶子节点存放的是否是一整行的信息

### 4.1、聚集索引
就是按照每张表的主键构造一棵B+树，叶子节点存放的就是整张表的行记录数据，也将叶子节点称为数据页。这个特性决定了索引组织表中数据也是索引的一部分。每个数据页都通过一个双向链表进行链接。
因为实际的数据页只能按照一棵B+树进行排序，所以**一张表只能拥有一个聚集索引。**
查询优化器采用聚集索引
**数据页上存放的是完整的每行的记录,而在非数据页的索引页中，存放的仅仅是键值及指向数据页的偏移量，而不是一个完整的行记录。**
好处：
> 1. 对于主键的排序查找和范围查找速度非常快
> 1. 另一个是范围查询(range query),即如果要查找主键某一范围内的数据，通过叶子节点的上层中间节点就可以得到页的范围，之后直接读取数据页即可

### 4.2、辅助索引
叶子节点包含键值和书签bookmark，这个书签就是告诉存储引擎哪里可以找到与索引相对应的行数据，由于InnoDB是索引组织表，所以书签就是相应行数据的聚集索引键


### 4.3、B+树索引的分裂
 /？？


### 4.4、B+树索引的管理
**![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621931346381-fcc28ca6-1b89-4c65-801a-665edbf2066a.png#align=left&display=inline&height=330&originHeight=330&originWidth=636&size=57072&status=done&style=shadow&width=636)**
## 5、Cardinality值
### 5.1、什么是
> 选择性
> 低：性别
> 高：取值范围很广

show index结果中的列Cardinality，是一个预估值
### 5.2、如何统计
通过采样的方法来完成
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621931675275-65f2d72e-9a76-493a-b017-883929dfca6d.png#align=left&display=inline&height=163&originHeight=163&originWidth=604&size=54757&status=done&style=shadow&width=604)


## 6、B+树索引的使用
### 6.1、不同应用中
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621931841478-cbb941ae-0efe-472c-8bad-2cde2576ee52.png#align=left&display=inline&height=435&originHeight=435&originWidth=610&size=166773&status=done&style=none&width=610)
### 6.2、联合索引
建立在多个列上的索引
### 6.3、覆盖索引
从辅助索引中就可以得到查询的记录，不需要查询聚集索引中的记录。
好处是辅助索引不包含整行记录的信息，所以大小远小于聚集索引，可以减少大量IO

1. 包含主键信息
1. 对某些统计问题

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621932343090-f337b249-170f-4a75-a30b-b04ace32ece2.png#align=left&display=inline&height=224&originHeight=224&originWidth=758&size=57398&status=done&style=shadow&width=758)
### 6.4、优化器选择不使用索引的情况
### 6.5、索引提示
显示地告诉优化器使用哪个索引
### 6.6、Multi-Range Read优化
为了减少磁盘的随机访问，将随机访问转化为较为顺序的数据访问


## 7、哈希算法
### 7.1、哈希表
链接法处理哈希冲突
### 7.2、InnoDB存储引擎中的哈希算法
InnoDB存储引擎哈希函数采用除法散列方式，冲突机制采用链表方式
### 7.3、自适应哈希索引
数据库自身创建并使用的
对于字典型查找很快


## 8、全文检索

1. 概述
> 全文检索是将存储于数据库中的整本书或整篇文章中的任意内容信息查找出来的技术

2. 倒排索引

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621933293955-ba42b319-28b5-43f1-aa83-574d6eb1f3bc.png#align=left&display=inline&height=344&originHeight=344&originWidth=720&size=114046&status=done&style=shadow&width=720)
/？？
# 第6章 锁
## 1、什么是锁
锁机制用于管理对共享资源的并发访问
InnoDB是行级锁
MyISAM是表级锁
## 2、lock 与 latch
latch一般称为闩锁（轻量级的锁），分为互斥量和读写锁，目的是保证并发线程操作临界资源的正确性，并且没有死锁检测的机制。
lock的对象是事务，锁定是数据库中的对象，如表、页、行
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621934495801-53868cff-24af-4700-93f5-c87e5144da37.png#align=left&display=inline&height=230&originHeight=230&originWidth=847&size=79159&status=done&style=shadow&width=847)
## 3、InnoDB存储引擎中的锁
### 3.1、锁的类型
行级锁

- 共享锁（S Lock） 允许事务读一行数据
- 排他锁（X Lock） 允许事务删除或更新一行数据

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621934634394-2ce9cd2d-1311-43b5-9fd8-2125aa3bb559.png#align=left&display=inline&height=117&originHeight=117&originWidth=828&size=24337&status=done&style=shadow&width=828)
意向锁：将锁定的对象分为多个层次，意味着事务希望在更细粒度上进行加锁
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621934849206-ca02fcf9-74d2-4030-a63e-e832581a3ab1.png#align=left&display=inline&height=662&originHeight=662&originWidth=811&size=259217&status=done&style=shadow&width=811)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1621934869537-8d1ff0e1-dba0-490f-993b-bf7635cd94d2.png#align=left&display=inline&height=199&originHeight=199&originWidth=817&size=44672&status=done&style=shadow&width=817)
### 3.2、一致性非锁定读
指InnoDB存储引擎通过行**多版本控制的方式读取当前执行时间数据库中行的数据**

- 如果读取的行正在执行DELETE或UPDATE操作，这时读取操作不会因此去等待行的释放
- 相反，会去读取行的一个快照数据
- 不需要等待访问的行上X锁的释放

MVCC
在事务隔离级别 读提交和可重复读下，InnoDB存储引擎使用非锁定的一致性读

- 读提交    非一致性读总是读取被锁定行的最新一份快照数据
- 可重复读    总是读取事务开始时的行数据版本



### 3.3、一致性锁定读
在可重复读模式下，InnoDB存储引擎的SELECT操作使用一致性锁定读
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622192188375-540f52de-d595-4ab2-b3c7-59debc21dfc5.png#align=left&display=inline&height=97&originHeight=97&originWidth=603&size=26334&status=done&style=shadow&width=603)
### 3.4、自增长与锁
### 3.5、外键和锁
外键用于引用完整性的约束检查
在InnoDB中，对于外键列，若未显式加索引，则自动对其加一索引，这样可以避免表锁




## 4、锁的算法
### 4.1、行锁的三种算法

1. `Record Lock` ： 单个行记录上锁
1. `Gap Lock`：间隙锁，锁定一个范围，但不包含记录本身
1. `Next-Key Lock`：`Gap Lock` +` Record Lock`，锁定一个范围，并且锁定记录本身（技术：Next-Key Locking）

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622193243963-c3c29d60-4d10-41f4-a325-1d1ae1303b87.png#align=left&display=inline&height=101&originHeight=101&originWidth=625&size=36937&status=done&style=shadow&width=625)


### 4.2、解决幻读问题
指在同一事务下，连续执行两次同样的SQL语句可能导致不同的结果，第二次的SQL语句可能会返回之前不存在的行。
InnoDB存储引擎采取Next-Key Locking技术避免幻读

## 5、锁问题
### 5.1、脏读
脏页：在缓冲池中已经被修改的页，但是还没刷新到磁盘中
脏数据：事务对缓冲池中行记录的修改，并且还没有被提交（未提交的数据，违反了数据库的隔离性）
脏读：在不同的事务下，当前事务可以读到其他事务未提交的数据
需要的事务级别是：读未提交


### 5.2、不可重复读
指一个事务内多次读取同一数据集合，在这个事务还没结束时，另外一个事务访问到该数据并操作，因此在第一个事务中的两次读取数据不同。

- 脏读是读到未提交的数据
- 不可重复读读到的是已提交的数据

解决不可重复读：Next-Key Lock算法
官方文档将不可重复读的问题定义为幻读。


### 5.3、丢失更新
一个事务的更新操作会被另一个事务的更新操作所覆盖，从而导致数据的不一致
数据库能阻止丢失更新问题的产生（阻塞）
解决：可串行化
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622299176336-964ae91a-41ba-42b2-b4ab-1944f9429fba.png#align=left&display=inline&height=749&originHeight=749&originWidth=715&size=274693&status=done&style=shadow&width=715)
## 6、阻塞
因为不同锁之间的兼容性关系，有些时刻一个事务中的锁需要等待另一个事务中的锁释放它所占用耳朵资源，这就是阻塞
## 7、死锁
### 7.1、死锁定义
死锁是指两个或两个以上的事务在执行过程中，因争夺锁资源而造成的一种互相等待的现象
解决：

- 超时
- 数据库普遍采用**等待图**的方式进行死锁检测
   - 保存信息
      - 锁的信息链表
      - 事务等待链表



### 7.2、死锁概率
非常少发生，死锁次数小于等待


## 8、锁升级
是指将当前锁的粒度降低
例如：可以将一个表的1000个行锁升级为一个页锁，或页锁升级为表锁
# 第7章 事务
事务是数据库区别于文件系统的重要特性之一
ACID

- 原子性
- 一致性
- 隔离性
- 持久性



## 1、认识事务
### 1.1、概述

1. 原子性
> 要么都做， 要么不做

2. 一致性
> 指事务将数据库从一种状态转变为下一种一致的状态
> 在事务开始之前与结束之后，数据库的完整性约束没被破坏

3. 隔离性
> 其他称呼：并发控制、可串行化、锁
> 事务的隔离性要求每个读写事务的对象对其他事务的操作对象相互分离，即事务提交前对其他事务都不可见

4. 持久性
> 事务一旦提交，其结果就是永久性的。

### 1.2、分类
事务分为：

- 扁平事务
> 最简单的一种，所有操作都处于同一层次

- 带有保存点的扁平事务
- 链事务
- 嵌套事务
- 分布式事务



## 2、事务的实现
### 2.1、redo

1. 基本概念

重做日志用来实现事务的持久性，两个部分

- 重做日志缓冲
- 重做日志文件

当事务提交时，必须先将事务的所有日志写入重做日志文件进行持久化，待事务的COMMIT操作完成才算完成
在InnoDB中，

- redo log 保证事务的持久性
- undo log 帮助事务回滚及MVCC的功能
2. log block

重做日志缓存和文件都是以块的方式进行保存的

3. log group

重做日志组

4. 重做日志格式



### 2.2、undo

1. 基本概念

进行回滚操作需要undo
redo存在重做日志文件中，
undo存在数据库内部的一个特殊段中，称为undo段，undo段位于共享表空间内
undo另一个作用：MVCC
undo的产生会伴随着redo log的产生

2. undo存储管理
2. undo log格式
2. 查看undo信息



### 2.3、purge


总结：
[https://www.jianshu.com/p/57c510f4ec28](https://www.jianshu.com/p/57c510f4ec28)
[https://blog.nowcoder.net/n/b9057d9341e9475db6d187067353e64f](https://blog.nowcoder.net/n/b9057d9341e9475db6d187067353e64f)


# 第8章 备份与恢复
## 1、概述
根据备份的方法不同可以将备份分为：

- 热备
> 在数据库运行中直接备份，对正在运行的数据库操作没有任何的影响，即在线备份

- **冷备**
> 指备份操作是在数据库停止的情况下，最为简单，一般只需要复制相关的数据库物理文件即可，即离线备份

- 温备
> 也是在数据库运行中进行，但是对当前数据库的操作有所影响，如加一个全局读锁以保证备份数据的一致性



按照备份后文件的内容，分为：

- **逻辑备份**
> 文件内容可读，一般是文本文件，缺点是时间较长

- 裸文件备份
> 复制数据库的物理文件，既可以是在数据库运行中的复制，也可以是数据库停止运行时直接的数据文件复制，时间较短

按备份数据库的内容来分，分为

- 完全备份
- 增量备份（MySQL未提供
- 日志备份
> 对数据库二进制日志的备份

MySQL数据库复制的原理就是异步实时的将二进制日志重做传送并应用到从数据库


## 2、冷备
对于InnoDB存储引擎的冷备，只需要备份MySQL数据库的frm文件，共享表空间文件，独立表空间文件和重做日志文件，建议备份my.conf
同一台机器冷备不够，需要远程服务器存储
优点

- 备份简单
- 易于恢复
- 恢复迅速

缺点

- 冷备的文件币逻辑文件大很多
- 不总是可以轻易跨平台



## 3、逻辑备份
### 3.1、mysqldump
用来完成转存数据库的备份以及不同数据库之间的移植
### 3.2、SELECT..INTO OUTFILE
导出一张表中的数据
### 3.3、逻辑备份的恢复
备份的软件是导出的SQL语句，只需要执行这个文件就可以。
### 3.4、LOAD DATA INFILE
导入
### 3.4、mysqlimport
是一个命令行程序，是LOAD DATA INFILE的命令接口
不同的是可以用来导入多张表
## 4、二进制日志备份与恢复
二进制日志默认不开启，开启需要设置：
> log-bin = mysql-bin

恢复：
> mysqlbinlog

## 5、热备
### 5.1、ibbackup
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622468277693-ac2789be-bca2-4305-a04c-4a4b4b105e65.png#align=left&display=inline&height=213&originHeight=213&originWidth=794&size=86832&status=done&style=shadow&width=794)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622468303620-67f6ea2c-91ad-4e0f-b031-5e63a06f3275.png#align=left&display=inline&height=159&originHeight=317&originWidth=805&size=110527&status=done&style=shadow&width=402.5)
### 5.2、XtraBackUp
## 6、快照备份
是指通过文件系统支持的快照功能对数据库进行备份
## 7、复制
### 7.1、复制的工作原理
3个步骤：

1. 主服务器master把数据更改记录到二进制日志（binlog）中
1. 从服务器slave把主服务器的二进制日志复制到自己的中继日志（relaylog）中
1. 从服务器重做中继日志中的日志，把更改应用到自己的数据库上，以达到数据库的最终一致性。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622468555242-65459f5c-c1be-4bf2-8bb9-9c728ed27832.png#align=left&display=inline&height=430&originHeight=430&originWidth=764&size=110680&status=done&style=shadow&width=764)
**从服务器两个线程**

- IO线程，读取二进制日志并保存为中继日志
- SQL线程，复制执行中继日志
### 7.2、快照+复制的备份架构
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2674809/1622468663314-db60dc92-5262-4936-895e-348dbd409231.png#align=left&display=inline&height=244&originHeight=390&originWidth=701&size=103473&status=done&style=shadow&width=439)
[https://www.cnblogs.com/wade-luffy/p/6307470.html#_label1](https://www.cnblogs.com/wade-luffy/p/6307470.html#_label1)
还有延时复制

