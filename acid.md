### 数据库隔离级别

#### ACID 事务的特性

- A：atomicity 原子性

  事务进行多个更改时，要么都提交更改成功，要么都回滚撤销更改

- C：consistency 一致性

  数据在任何时刻都是一致的，在事务提交后或者回滚后，或者进行中。当跨表更新数据时，要么是看到都是提交后的新数据，要么看到的都是回滚后的旧数据，不会既出现新数据又同时出现旧数据

- I: isolation 隔离性

  事务在进行中时，是互相隔离的，不会被其他事务干扰，也不会看到其他事务未提交的数据。隔离性是通过锁机制实现的。

- D：durability 持久性

  一旦事务提交后，数据就会被持久化，不会受外力影响，如系统崩溃，电源中断等

#### mysql的隔离级别

 The default isolation level for `InnoDB` is [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/5.6/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read).innodb使用锁策略来实现不同的隔离级别，隔离级别越高，并发量和性能越差

-  [`READ UNCOMMITTED`](https://dev.mysql.com/doc/refman/5.6/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted)

  非锁定方式进行读取，会造成脏读现象。即读取到其他事务未提交的数据

-  [`READ COMMITTED`](https://dev.mysql.com/doc/refman/5.6/en/innodb-transaction-isolation-levels.html#isolevel_read-committed) 
  1. 普通的非加锁读通过快照来实现一致性读取
  2. 加锁的读取或者update，delete操作，会锁住索引记录，不会锁住间隙，所以会导致中间插入数据的情况，造成幻读现象
  3. 这种隔离模式下，仅支持基于行操作的二进制binlog格式
  4. 执行更新或者删除时，仅锁定要操作的行，对where没有命中的行，会释放掉行锁，大大减少死锁
  5. 对于更新操作，如果该行已经被锁定，会执行半一致性读，返回该行的最新的提交记录，mysql来判断是否匹配where条件，若匹配，会再次读取该行，这次会锁定该行或者等待该行的锁被释放
  6. 如果没有命中索引，对每行逐一更新，可重复读会锁住所有的行，即使不需要更新该行，直到整个事务结束；如果是已提交读则会判断是否需要更新，不更新则立刻释放。
- [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/5.6/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)

    1.  同一事务的一致性读是由第一次读取建立的快照实现的，同一个事务里，多个不加锁的select读，这些select之间都是一致性的
       2.  对于加锁的读操作，或者update，delete操作，取决于是否使用了唯一索引的唯一查找条件或者范围查找。如果是唯一查找，则会锁住找到的索引记录，不会锁住前面的间隙；如果是范围查找，会锁住扫描的索引范围，使用gap锁或者next-key锁防止其他事务在中间插入数据。（如果没有命中索引呢？锁全表吗？）

-  [`SERIALIZABLE`](https://dev.mysql.com/doc/refman/5.6/en/innodb-transaction-isolation-levels.html#isolevel_serializable)

  会将select语句加锁进行读取

