- [前提概要](#前提概要)
  - [什么是MVCC](#什么是mvcc)
  - [什么是当前读和快照读](#什么是当前读和快照读)
  - [MVCC实现原理](#mvcc实现原理)
    - [隐式字段](#隐式字段)
    - [undo日志](#undo日志)
# 前提概要

MySQL 中InnoDB中实现了事务（多版本并发控制MVCC+锁）， 其中通过MVCC解决隔离性问题。具体而言，MVCC就是为了实现读-写冲突不加锁，而这个读指的就是快照读, 而非当前读，当前读实际上是一种加锁的操作，是悲观锁的实现。

## 什么是MVCC

    MVCC，全称Multi-Version Concurrency Control，即多版本并发控制。MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。

    MVCC在MySQL InnoDB中的实现主要是为了提高数据库并发性能，用更好的方式去处理读-写冲突，做到即使有读写冲突时，也能做到不加锁，非阻塞并发读
## 什么是当前读和快照读
    - ### 当前读

        像select lock in share mode(共享锁), select for update ; update, insert ,delete(排他锁)这些操作都是一种当前读，为什么叫当前读？就是它读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁
    - ### 快照读

        像不加锁的select操作就是快照读，即不加锁的非阻塞读；快照读的前提是隔离级别不是串行级别，串行级别下的快照读会退化成当前读；之所以出现快照读的情况，是基于提高并发性能的考虑，快照读的实现是基于多版本并发控制，即MVCC,可以认为MVCC是行锁的一个变种，但它在很多情况下，避免了加锁操作，降低了开销；既然是基于多版本，即快照读可能读到的并不一定是数据的最新版本，而有可能是之前的历史版本
## MVCC实现原理

    MVCC的目的就是多版本并发控制，在数据库中的实现，就是为了解决读写冲突，它的实现原理主要是依赖记录中的 3个隐式字段，undo日志 ，Read View 来实现的。
### 隐式字段
      - DB_ROW_ID 6byte, 隐含的自增ID（隐藏主键），如果数据表没有主键，InnoDB会自动以DB_ROW_ID产生一个聚簇索引
      - DB_TRX_ID 6byte, 最近修改(修改/插入)事务ID：记录创建这条记录/最后一次修改该记录的事务ID
      - DB_ROLL_PTR 7byte, 回滚指针，指向这条记录的上一个版本（存储于rollback segment里）
      - DELETED_BIT 1byte, 记录被更新或删除并不代表真的删除，而是删除flag变了
### undo日志
- Insert undo log

    插入一条记录时，至少要把这条记录的主键值记下来，之后回滚的时候只需要把这个主键值对应的记录删掉就好了。
- Update undo log

    修改一条记录时，至少要把修改这条记录前的旧值都记录下来，这样之后回滚时再把这条记录更新为旧值就好了。

- Delete undo log

    删除一条记录时，至少要把这条记录中的内容都记下来，这样之后回滚时再把由这些内容组成的记录插入到表中就好了。