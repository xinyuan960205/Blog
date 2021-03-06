# MYSQL锁

## 锁类型

### 共享锁（S，读锁）

简写为 S 锁，又称读锁。

一个事务对数据对象 A 加了 S 锁，可以对 A 进行读取操作，但是不能进行更新操作。加锁期间其它事务能对 A 加 S 锁，但是不能加 X 锁。

### 排它锁（X，写锁）

简写为 X 锁，又称写锁。

一个事务对数据对象 A 加了 X 锁，就可以对 A 进行读取和更新。加锁期间其它事务不能对 A 加任何锁。

- 互斥锁（Exclusive），简写为 X 锁，又称写锁。
- 共享锁（Shared），简写为 S 锁，又称读锁。

有以下两个规定：

- 一个事务对数据对象 A 加了 X 锁，就可以对 A 进行读取和更新。加锁期间其它事务不能对 A 加任何锁。
- 一个事务对数据对象 A 加了 S 锁，可以对 A 进行读取操作，但是不能进行更新操作。加锁期间其它事务能对 A 加 S 锁，但是不能加 X 锁。

## 锁粒度

### 介绍

加锁需要消耗资源，锁的各种操作（包括获取锁、释放锁、以及检查锁状态）都会增加系统开销。

不同的存储引擎支持的锁粒度是不一样的：

- **InnoDB行锁和表锁都支持**！
- **MyISAM只支持表锁**！

InnoDB只有通过**索引条件**检索数据**才使用行级锁**，否则，InnoDB将使用**表锁**

- 也就是说，**InnoDB的行锁是基于索引的**！

### 行级锁

开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高

### 表级锁

开销小，加锁快；不会出现死锁；锁定力度大，发生锁冲突概率高，并发度最低

### 页级锁（MySQL特有）

开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

页级锁定是MySQL中比较独特的一种锁定级别，在其他数据库管理软件中也并不是太常见。页级锁定的特点是锁定颗粒度介于行级锁定与表级锁之间，所以获取锁定所需要的资源开销，以及所能提供的并发处理能力也同样是介于上面二者之间。

## 加锁协议

1.一级封锁协议：事务T中如果对数据R有写操作，必须在这个事务中对R的第一次读操作前对它加X锁，直到事务结束才释放。事务结束包括正常结束（COMMIT）和非正常结束（ROLLBACK）。

2.二级封锁协议：一级封锁协议加上事务T在读取数据R之前必须先对其加S锁，读完后方可释放S锁。

3.三级封锁协议 ：一级封锁协议加上事务T在读取数据R之前必须先对其加S锁，直到事务结束才释放。

三级锁操作一个比一个厉害（满足高级锁则一定满足低级锁）。但有个非常致命的地方，一级锁协议就要在第一次读加x锁，直到事务结束。几乎就要在整个事务加写锁了，效率非常低。三级封锁协议只是一个理论上的东西，实际数据库常用另一套方法来解决事务并发问题。

## InnoDB

### 意向锁

使用意向锁（Intention Locks）可以更容易地支持多粒度封锁。

在存在行级锁和表级锁的情况下，事务 T 想要对表 A 加 X 锁，就需要先检测是否有其它事务对表 A 或者表 A 中的任意一行加了锁，那么就需要对表 A 的每一行都检测一次，这是非常耗时的。

意向锁在原来的 X/S 锁之上引入了 IX/IS，IX/IS 都是表锁，用来表示一个事务想要在表中的某个数据行上加 X 锁或 S 锁。有以下两个规定：

- 一个事务在获得某个数据行对象的 S 锁之前，必须先获得表的 IS 锁或者更强的锁；
- 一个事务在获得某个数据行对象的 X 锁之前，必须先获得表的 IX 锁。

通过引入意向锁，事务 T 想要对表 A 加 X 锁，只需要先检测是否有其它事务对表 A 加了 X/IX/S/IS 锁，如果加了就表示有其它事务正在使用这个表或者表中某一行的锁，因此事务 T 加 X 锁失败。

各种锁的兼容关系如下：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/%E6%84%8F%E5%90%91%E9%94%81.png)

解释如下：

- 任意 IS/IX 锁之间都是兼容的，因为它们只表示想要对表加锁，而不是真正加锁；
- 这里兼容关系针对的是表级锁，而表级的 IX 锁和行级的 X 锁兼容，两个事务可以对两个数据行加 X 锁。（事务 T1 想要对数据行 R1 加 X 锁，事务 T2 想要对同一个表的数据行 R2 加 X 锁，两个事务都需要对该表加 IX 锁，但是 IX 锁是兼容的，并且 IX 锁与行级的 X 锁也是兼容的，因此两个事务都能加锁成功，对同一个表中的两个数据行做修改。

### 行锁的算法

#### Record Lock

锁定一个记录上的索引，而不是记录本身。

如果表没有设置索引，InnoDB 会自动在主键上创建隐藏的聚簇索引，因此 Record Locks 依然可以使用。



#### Gap Lock

当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙(GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁

锁定索引之间的间隙，但是不包含索引本身。例如当一个事务执行以下语句，其它事务就不能在 t.c 中插入 15。

```
SELECT c FROM t WHERE c BETWEEN 10 and 20 FOR UPDATE;
```

- 所以可以看出，在进行数据查询的时候应该尽量优化成=而不是范围查询。

目的：

1. 防止幻读，以满足相关隔离级别的要求；
2. 满足恢复和复制的需要；

#### Next-Key Lock

它是 Record Locks 和 Gap Locks 的结合，不仅锁定一个记录上的索引，也锁定索引之间的间隙。它锁定一个前开后闭区间，例如一个索引包含以下值：10, 11, 13, and 20，那么就需要锁定以下区间：

```
(-∞, 10]
(10, 11]
(11, 13]
(13, 20]
(20, +∞)
```

**但是**我们要想到LBCC这样的加锁访问，其实并不算是真正的并发，或者说它只能实现并发的读，因为它最终实现的是读写串行化，这样就大大降低了数据库的读写性能。
基于这样，MySQL在后续设计中引入MVCC这一基于多版本的并发控制协议。主要是解决读写的冲突！！！
MVCC只工作在RC与RR级别下，当然最主要还是为RR级别设计。
然后我们要思考一个问题，MVCC解决读写冲突，那写写冲突如何解决？那就是一个乐观锁的设计了。

### InnoDB加锁方法

```sql
//显示共享锁（S） ：
 SELECT * FROM table_name WHERE .... LOCK IN SHARE MODE
 //显示排他锁（X）：
 SELECT * FROM table_name WHERE .... FOR UPDATE.
```

使用select … in share mode获取共享锁，主要用在需要数据依存关系时，确认某行记录是否存在，并确保没有人对这个记录进行update或者delete。



## 锁的使用方式

### 乐观锁

乐观锁是对于数据冲突保持一种乐观态度，操作数据时不会对操作的数据进行加锁（这使得多个任务可以并行的对数据进行操作），只有到数据提交的时候才通过一种机制来验证数据是否存在冲突(一般实现方式是通过加版本号然后进行版本号的对比方式实现);

特点：乐观锁是一种并发类型的锁，其本身不对数据进行加锁通而是通过业务实现锁的功能，不对数据进行加锁就意味着允许多个请求同时访问数据，同时也省掉了对数据加锁和解锁的过程，这种方式大大的提高了数据操作的性能;

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E4%B9%90%E8%A7%82%E9%94%81.jpg)



### 悲观锁

顾名思义，悲观锁是基于一种悲观的态度类来防止一切数据冲突，它是以一种预防的姿态在修改数据之前把数据锁住，然后再对数据进行读写，在它释放锁之前任何人都不能对其数据进行操作，直到前面一个人把锁释放后下一个人数据加锁才可对数据进行加锁，然后才可以对数据进行操作，一般数据库本身锁的机制都是基于悲观锁的机制实现的;

特点：可以完全保证数据的独占性和正确性，因为每次请求都会先对数据进行加锁， 然后进行数据操作，最后再解锁，而加锁释放锁的过程会造成消耗，所以性能不高;

手动加悲观锁：读锁LOCK tables test_db read释放锁UNLOCK TABLES;

写锁：LOCK tables test_db WRITE释放锁UNLOCK TABLES;



## 多版本并发控制（MVCC）

多版本并发控制（Multi-Version Concurrency Control, MVCC）是 MySQL 的 InnoDB 存储引擎实现隔离级别的一种具体方式，用于实现提交读和可重复读这两种隔离级别。而未提交读隔离级别总是读取最新的数据行，要求很低，无需使用 MVCC。可串行化隔离级别需要对所有读取的行都加锁，单纯使用 MVCC 无法实现。

### 基本思想

在封锁一节中提到，加锁能解决多个事务同时执行时出现的并发一致性问题。在实际场景中读操作往往多于写操作，因此又引入了读写锁来避免不必要的加锁操作，例如读和读没有互斥关系。读写锁中读和写操作仍然是互斥的，而 MVCC 利用了多版本的思想，写操作更新最新的版本快照，而读操作去读旧版本快照，没有互斥关系，这一点和 CopyOnWrite 类似。

在 MVCC 中事务的修改操作（DELETE、INSERT、UPDATE）会为数据行新增一个版本快照。

脏读和不可重复读最根本的原因是事务读取到其它事务未提交的修改。在事务进行读取操作时，为了解决脏读和不可重复读问题，MVCC 规定只能读取已经提交的快照。当然一个事务可以读取自身未提交的快照，这不算是脏读。

### 版本号

- 系统版本号 SYS_ID：是一个递增的数字，每开始一个新的事务，系统版本号就会自动递增。
- 事务版本号 TRX_ID ：事务开始时的系统版本号。

### Undo 日志

MVCC 的多版本指的是多个版本的快照，快照存储在 Undo 日志中，该日志通过回滚指针 ROLL_PTR 把一个数据行的所有快照连接起来。

例如在 MySQL 创建一个表 t，包含主键 id 和一个字段 x。我们先插入一个数据行，然后对该数据行执行两次更新操作。

```
INSERT INTO t(id, x) VALUES(1, "a");
UPDATE t SET x="b" WHERE id=1;
UPDATE t SET x="c" WHERE id=1;
```

因为没有使用 `START TRANSACTION` 将上面的操作当成一个事务来执行，根据 MySQL 的 AUTOCOMMIT 机制，每个操作都会被当成一个事务来执行，所以上面的操作总共涉及到三个事务。快照中除了记录事务版本号 TRX_ID 和操作之外，还记录了一个 bit 的 DEL 字段，用于标记是否被删除。

[![img](https://camo.githubusercontent.com/09832513cf5a63e15816143a204052677c46fc0c/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383136343830383231372e706e67)](https://camo.githubusercontent.com/09832513cf5a63e15816143a204052677c46fc0c/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383136343830383231372e706e67)



INSERT、UPDATE、DELETE 操作会创建一个日志，并将事务版本号 TRX_ID 写入。DELETE 可以看成是一个特殊的 UPDATE，还会额外将 DEL 字段设置为 1。

### ReadView

MVCC 维护了一个 ReadView 结构，主要包含了当前系统未提交的事务列表 TRX_IDs {TRX_ID_1, TRX_ID_2, ...}，还有该列表的最小值 TRX_ID_MIN 和 TRX_ID_MAX。

[![img](https://camo.githubusercontent.com/4d58315fa3b98e4b09cc51b9debaf9ce27b1c312/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383137313434353637342e706e67)](https://camo.githubusercontent.com/4d58315fa3b98e4b09cc51b9debaf9ce27b1c312/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383137313434353637342e706e67)



在进行 SELECT 操作时，根据数据行快照的 TRX_ID 与 TRX_ID_MIN 和 TRX_ID_MAX 之间的关系，从而判断数据行快照是否可以使用：

- TRX_ID < TRX_ID_MIN，表示该数据行快照时在当前所有未提交事务之前进行更改的，因此可以使用。
- TRX_ID > TRX_ID_MAX，表示该数据行快照是在事务启动之后被更改的，因此不可使用。
- TRX_ID_MIN <= TRX_ID <= TRX_ID_MAX，需要根据隔离级别再进行判断：
  - 提交读：如果 TRX_ID 在 TRX_IDs 列表中，表示该数据行快照对应的事务还未提交，则该快照不可使用。否则表示已经提交，可以使用。
  - 可重复读：都不可以使用。因为如果可以使用的话，那么其它事务也可以读到这个数据行快照并进行修改，那么当前事务再去读这个数据行得到的值就会发生改变，也就是出现了不可重复读问题。

在数据行快照不可使用的情况下，需要沿着 Undo Log 的回滚指针 ROLL_PTR 找到下一个快照，再进行上面的判断。

### 快照读与当前读

#### 1. 快照读

MVCC 的 SELECT 操作是快照中的数据，不需要进行加锁操作。

```
SELECT * FROM table ...;
```

#### 2. 当前读

MVCC 其它会对数据库进行修改的操作（INSERT、UPDATE、DELETE）需要进行加锁操作，从而读取最新的数据。可以看到 MVCC 并不是完全不用加锁，而只是避免了 SELECT 的加锁操作。

```
INSERT;
UPDATE;
DELETE;
```

在进行 SELECT 操作时，可以强制指定进行加锁操作。以下第一个语句需要加 S 锁，第二个需要加 X 锁。

```
SELECT * FROM table WHERE ? lock in share mode;
SELECT * FROM table WHERE ? for update;
```



## 死锁

并发的问题就少不了死锁，在MySQL中同样会存在死锁的问题。

但一般来说MySQL通过回滚帮我们解决了不少死锁的问题了，但死锁是无法完全避免的，可以通过以下的经验参考，来尽可能少遇到死锁：

- 1）以**固定的顺序**访问表和行。比如对两个job批量更新的情形，简单方法是对id列表先排序，后执行，这样就避免了交叉等待锁的情形；将两个事务的sql顺序调整为一致，也能避免死锁。
- 2）**大事务拆小**。大事务更倾向于死锁，如果业务允许，将大事务拆小。
- 3）在同一个事务中，尽可能做到**一次锁定**所需要的所有资源，减少死锁概率。
- 4）**降低隔离级别**。如果业务允许，将隔离级别调低也是较好的选择，比如将隔离级别从RR调整为RC，可以避免掉很多因为gap锁造成的死锁。
- 5）**为表添加合理的索引**。可以看到如果不走索引将会为表的每一行记录添加上锁，死锁的概率大大增大。


## 参考

- <https://blog.csdn.net/Jack__Frost/article/details/73347688>
- <https://zhuanlan.zhihu.com/p/31537871>
- <https://zhuanlan.zhihu.com/p/29150809>
- <https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484721&idx=1&sn=410dea1863ba823bec802769e1e6fe8a&chksm=ebd74430dca0cd265a9a91dcb2059e368f43a25f3de578c9dbb105e1fba0947e1fd0b9c2f4ef&token=1676899695&lang=zh_CN###rd>







