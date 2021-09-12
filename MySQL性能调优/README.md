# 简介
> 数据库锁定机制简单来说就是数据库为了保证数据的一致性而使各种共享资源在被并发访问访问变
> 得有序所设计的一种规则。MySQL 数据库由于其自身架构的特点，存在多种数据存储引擎，每种存储引擎所针对的应用场景特点都不太一样，为了满足各自特定应用场景的需求，每种存储引擎的锁定机制都是为各自所面对的特定场景而优化设计，所以各存储引擎的锁定机制也有较大区别。


MySQL 各存储引擎使用了三种类型（级别）的锁定机制：行级锁定，页级锁定和表级锁定。

# 行级锁定
> 优点：行级锁定最大的特点就是锁定对象的颗粒度很小，也是目前各大数据库管理软件所实现的锁定颗粒
> 度最小的。由于锁定颗粒度很小，所以发生锁定资源争用的概率也最小，能够给予应用程序尽可能大的
> 并发处理能力而提高一些需要高并发应用系统的整体性能。
> 缺点：虽然能够在并发处理能力上面有较大的优势，但是行级锁定也因此带来了不少弊端。由于锁定资源
> 的颗粒度很小，所以每次获取锁和释放锁需要做的事情也更多，带来的消耗自然也就更大了。此外，行
> 级锁定也最容易发生死锁

## 锁定类型

> Innodb的行级锁定同样分为两种类型，共享锁和排他锁，而在锁定机制的实现过程中为了让行级锁定和表级锁定共存，Innodb 也同样使用了意向锁（表级锁定）的概念，也就有了意向共享锁和意向排他锁这两种。

- 共享锁
- 排他锁
- 意向共享锁
- 意向排他锁

四种锁的共存逻辑关系

|                      | 共享锁（S） | 排他锁（X） | 意 向 共 享 锁（IS） | 意 向 排 他 锁（IX） |
| -------------------- | ----------- | ----------- | -------------------- | -------------------- |
| 共享锁（S）          | 兼容        | 冲突        | 兼容                 | 冲突                 |
| 排他锁（X）          | 冲突        | 冲突        | 冲突                 | 冲突                 |
| 意 向 共 享 锁（IS） | 兼容        | 冲突        | 兼容                 | 兼容                 |
| 意 向 排 他 锁（IX） | 冲突        | 冲突        | 兼容                 | 兼容                 |



## 行锁优化

1. 尽可能让所有的数据检索通过索引来完成，避免Innodb因为无法通过索引键加锁而升级为表级锁定
2. 尽量控制事务的大小，减少锁定的资源量和锁定时间长度



## 防死锁建议

1. 类似业务模块中，尽可能按照相同的访问顺序来访问，防止发生死锁
2. 对于非常容易产生死锁的业务部分，可以尝试升级锁定颗粒度，通过表级锁定来减少死锁发生的概率



## 锁定情况查询

```
mysql>show status like 'innodb_row_lock%';
```

| Variable_name                 | Value  | Comment                                          |
| ----------------------------- | ------ | ------------------------------------------------ |
| Innodb_row_lock_current_waits | 0      | 当前正在等待锁定的数量                           |
| \| Innodb_row_lock_time       | 490578 | 从系统启动到现在锁定总时间长度                   |
| \| Innodb_row_lock_time_avg   | 37736  | 每次等待所花平均时间（重要）                     |
| \| Innodb_row_lock_time_max   | 121411 | 从系统启动到现在等待最常的一次所花的时间（重要） |
| \| Innodb_row_lock_waits      | 13     | 系统启动后到现在总共等待的次数（重要）           |



## Innodb 锁定机制演示

有索引行锁演示

| 时刻 | Session a                                                    | Session b                                                    |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | mysql> set autocommit=0; <br>Query OK, 0 rows affected (0.00sec) | mysql> set autocommit=0 <br/>Query OK, 0 rows affected (0.00 sec) |
| 2    | mysql> update test_innodb_lock set b='100' where a=1;<br/>Query OK, 1 row affected (0.00sec)<br/>Rows matched: 1 Changed: 1Warnings: 0 |                                                              |
| 3    |                                                              | mysql> update test_innodb_lock set b = '100' where a = 1;<br/>被阻塞，等待释放 |
| 4    | mysql> commit;<br/>Query OK, 1 row affected (0.00sec)        |                                                              |
| 5    |                                                              | mysql> update test_innodb_lock set b = '100' where a = 1;<br/>Query OK, 0 row affected (36.14sec)<br/>Rows matched: 1 Changed: 0 Warnings: 0<br/>阻塞解除，正常运行 |

无索引表锁演示

| 时刻 | Session a                                                    | Session b                                                    |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | mysql> set autocommit=0; <br>Query OK, 0 rows affected (0.00sec) | mysql> set autocommit=0 <br/>Query OK, 0 rows affected (0.00 sec) |
| 2    | mysql> update test_innodb_lock set b='100' where b='200';<br/>Query OK, 1 row affected (0.00sec)<br/>Rows matched: 1 Changed: 1Warnings: 0 |                                                              |
| 3    |                                                              | mysql> update test_innodb_lock set b = '100' where b = '300';<br/>被阻塞，等待释放 |
| 4    | mysql> commit;<br/>Query OK, 1 row affected (0.00sec)        |                                                              |
| 5    |                                                              | mysql> update test_innodb_lock set b = '100' where b = '300'; <br/> Query OK, 1 row affected (36.14sec)<br/>Rows matched: 1 Changed: 1 Warnings: 0<br/>阻塞解除，正常运行 |

死锁演示


| 时刻 | Session a                                                    | Session b                                                    |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | mysql> set autocommit=0; <br>Query OK, 0 rows affected (0.00sec) | mysql> set autocommit=0 <br/>Query OK, 0 rows affected (0.00 sec) |
| 2    | mysql> update test_innodb_lock_t1 set b='100' where a=1;<br/>Query OK, 1 row affected (0.00sec)<br/>Rows matched: 1 Changed: 1Warnings: 0 |                                                              |
| 3    |                                                              | mysql> update test_innodb_lock_t2 set b = '100' where a = 1;<br/>Query OK, 1 row affected (0.00sec)<br/>Rows matched: 1 Changed: 1Warnings: 0 |
| 4    | mysql> update test_innodb_lock_t2 set b = '100' where a = 1;<br/>被阻塞，等待释放 |                                                              |
| 5    |                                                              | mysql> update test_innodb_lock_t1 set b='100' where a=1;<br/>被阻塞，等待释放 |


# 表级锁定
> 优点：和行级锁定相反，表级别的锁定是 MySQL 各存储引擎中最大颗粒度的锁定机制。该锁定机制最大的特点是实现逻辑非常简单，带来的系统负面影响最小。所以获取锁和释放锁的速度很快。由于表级锁一次会将整个表锁定，所以可以很好的避免困扰我们的死锁问题。
> 缺点：当然，锁定颗粒度大所带来最大的负面影响就是出现锁定资源争用的概率也会最高，致使并大度大
> 打折扣。

## 锁定类型
- 读锁定
- 写锁定

>MySQL主要通过四个队列来维护这两种锁定：两个存放当前正在锁定中的读和写锁定信息，另外两个存放等待中的读写锁定信息，如下：
>- Current read-lock queue  (lock->read)
>- Pending read-lock queue  (lock->read_wait)
>- Current write-lock queue  (lock->write)
 - Pndinrit-lk uu (lock->writewait)

### 读锁定
一个新的客户端请求在申请获取读锁定资源的时候，需要满足两个条件：
1、 请求锁定的资源当前没有被写锁定；
2、 写锁定等待队列（Pending write-lock queue）中没有更高优先级的写锁定等待；
如果满足了上面两个条件之后，该请求会被立即通过，并将相关的信息存入 Current read-lock
queue 中，而如果上面两个条件中任何一个没有满足，都会被迫进入等待队列 Pending read-lock queue
中等待资源的释放。

### 写锁定
当客户端请求写锁定的时候，MySQL 首先检查在 Current write-lock queue 是否已经有锁定相同资
源的信息存在。

如果 Current write-lock queue 没有，则再检查 Pending write-lock queue，如果在 Pending
write-lock queue 中找到了，自己也需要进入等待队列并暂停自身线程等待锁定资源。反之，如果
Pending write-lock queue 为空，则再检测 Current read-lock queue，如果有锁定存在，则同样需要
进入 Pending write-lock queue 等待。当然，也可能遇到以下这两种特殊情况：
1. 请求锁定的类型为 WRITE_DELAYED;
2. 请求 锁定 的 类型 为 WRITE_CONCURRENT_INSERT 或者 是 TL_WRITE_ALLOW_WRITE，同 时
Current read lock 是 READ_NO_INSERT 的锁定类型。
当遇到这两种特殊情况的时候，写锁定会立即获得而进入 Current write-lock queue 中
如果刚开始第一次检测就 Current write-lock queue 中已经存在了锁定相同资源的写锁定存在，那
么就只能进入等待队列等待相应资源锁定的释放了

## 表锁优化

1. 缩短锁定时间

2. 分离能并行的操作
3. 合理利用读写优先级



## 锁定情况查询

```
mysql>show status like 'table%';
```

如果这里的Table_locks_waited 状态值比较高，那么说明系统中表级锁定争用现象比较严重，就需要进一步分析为什么会有较多的锁定资源争用了。

| Variable_name         | Value | Comment                          |
| --------------------- | ----- | -------------------------------- |
| Table_locks_immediate | 100   | 产生表级锁定的次数               |
| Table_locks_waited    | 0     | 出现表级锁定争用而发生等待的次数 |



# 页级锁定
> 页级锁定是 MySQL 中比较独特的一种锁定级别，在其他数据库管理软件中也并不是太常见。页级锁
> 定的特点是锁定颗粒度介于行级锁定与表级锁之间，所以获取锁定所需要的资源开销，以及所能提供的
> 并发处理能力也同样是介于上面二者之间。另外，页级锁定和行级锁定一样，会发生死锁。