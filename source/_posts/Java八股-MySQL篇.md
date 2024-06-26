---
title: Java八股-MySQL篇
tags:
  - MySQL
  - Java
categories: 八股
abbrlink: 43973
date: 2024-05-20 18:24:37
---

# Java八股-MySQL篇

## MySQL优化

### 如何定位慢查询

> 聚合查询、多表查询、表数据量过大查询、深度分页查询

- 开源工具
  - 调试工具：Arthas
  - 运维工具：Prometheus、Skywalking
- MySQL自带慢查询日志（slow_query_log、long_query_time）

### 如何分析慢查询

**explain**或者**desc**获取MySQL如何执行select语句的信息

- possible_keys：当前sql可能用到的索引
- key：当前sql实际命中的索引
- key_len：索引占用的大小
- type：sql连接的类型，NULL、system、const、eq_ref、ref、range、index、all

### 什么是索引

>  索引是帮助MySQL高效获取数据的数据结构（有序）

采用B+树作为数据结构存储索引

- 阶数更多。路径更短
- 磁盘读写代价B+树更低，非叶子结点只存储指针，叶子结点存储数据
- B+树便于扫库和区间查询，叶子结点是一个双向链表

### 聚簇索引？非聚簇索引？会回表查询？覆盖索引？

- 聚簇索引：数据和索引放一块，B+树的叶子结点保存整行数据，有且只有一个
- 非聚簇索引（二级索引）：数据和索引分开存储，B+树的叶子结点保存对应的主键，可以有多个
- 回表查询：通过二级索引找到对应的主键值，到聚簇索引中查找整行数据
- 覆盖索引：查询使用了索引，并且需要返回的列在该索引中已经全部能够找到

### MySQL超大分页处理

>  在数据量比较大时，如果进行limit分页查询，在查询时越往后分页查询效率越低

优化思路：一般分页查询时，通过创建覆盖索引能够比较好地提升性能，可以通过覆盖索引加子查询形式优化。

超大分页一般都是在数据量比较大时，我们使用limit分页查询，并且需要对数据进行排序，这个时候效率就很低，我们可以采用覆盖索引+子查询来解决。

先分页查询数据id字段，确定了id之后又，再用子查询来过滤，只查询这个id列表中的数据就可以了，因为查询id的时候，走的覆盖索引，所以效率可以提升很多。

```mysql
select * 
from User_table a,
	(select id from User_table order by id limit 1000000,10) b 
where a.id = b.id
```

### 索引创建原则

- **针对数据量较大，且查询比较频繁的表建立索引（10w）**
- **针对常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引**
- 选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高
- 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引
- **尽量使用联合索引，减少单列索引，查询时联合索引很多时候可以使用覆盖索引，节省存储空间，避免回表，提高查询效率**
- **控制索引数量，影响增删改效率**
- 如果索引列不能存储NULL，创建表时使用NOT NULL约束，优化器知道每列是否包含NULL值时，可以更好地确定哪个索引更有效地用于查询

### 索引失效

> 联合索引：（a，b，c）

- 违反最左前缀法则

  a=A，c=C时会只使用a索引

- 范围查询右边的列不能使用索引

  a=A，b>B时用不到c索引

- 索引列上进行运算操作

  类似substring(a, 0, 2)，索引失效

- 字符串不加单引号

  查询时，没有对字符串加单引号，MySQL的查询优化器会自动进行类型转换造成索引失效

- 模糊查询

  %like会失效，like%不会

### sql优化

![image-20240521161345554](https://gitee.com/qingy735/blogimg/raw/master/img/image-20240521161345554.png)

## 事务

### 事务特性

> ACID；原子性、一致性、隔离性、持久性

### 并发事务带来了哪些问题

> 脏读：一个事务读到另一个事务还没提交的数据
>
> 不可重复度：一个事务先后读取同一条数据，但是两次读取数据不同
>
> 幻读：一个事务按照条件查询数据时，没有对应的数据行，但是插入数据时又发现这行数据已经存在

#### 隔离级别：

|         隔离级别          | 脏读 | 不可重复读 | 幻读 |
| :-----------------------: | :--: | :--------: | :--: |
| Read uncommitted 读未提交 |  √   |     √      |  √   |
|  Read committed 读已提交  |  ×   |     √      |  √   |
| Repeatable Read 可重复度  |  ×   |     ×      |  √   |
|    Serializable 串行化    |  ×   |     ×      |  ×   |

### undo log和redo log

- 缓冲池（buffer pool）：主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，增删改查时先操作缓冲池中的数据（没有就从磁盘上加载并缓存），以一定频率刷新到磁盘，从而减少磁盘IO，加快处理速度
- 数据页（page）：是InnoDB存储引擎磁盘管理的最小单元，每个页大小默认为16KB，页中存储的是行数据

#### redo log

重做日志，记录的是事务提交时数据页的物理修改，是**用来实现事务的持久性**

该文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log file），前者在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都存在该日志文件中，用于在刷新脏页到磁盘发生错误时进行数据恢复使用。

#### undo log

回滚日志，用于记录数据被修改前的信息，作用包含两个：**提供回滚**和**MVCC**。undo log和redo log记录物理日志不一样，它是**逻辑日志**。

- delete一条记录时，undo log中会记录一条对应的insert记录，反之亦然
- update一条数据时，记录一条相反的update记录

**undo log可以实现事务的一致性和原子性**

#### redo log和undo log区别

- redo log记录的是数据页的物理变化，服务器宕机可以用来同步数据
- undo log记录的是逻辑日志，当事务回滚时通过逆操作恢复原来数据
- redo log保证事务持久性，undo log保证事务原子性和一致性

### MVCC（多版本并发控制）

#### 实现原理

- 记录中的隐藏字段
  |  隐藏字段   |                             含义                             |
  | :---------: | :----------------------------------------------------------: |
  |  DB_TRX_ID  | 最近一次修改事务ID，记录插入这条记录或最后一次修改该记录的事务ID |
  | DB_ROLL_PTR | 回滚指针，指向这条记录的上一个版本，用于配合undo log，指向上个版本 |
  |  DB_ROW_ID  |     隐藏主键，如果表结构没有指定主键，将会生成该隐藏字段     |

- undo log

  当insert时产生的undo log日志只在回滚时需要，在事务提交后可被立即删除

  update、delete的时候产生的undo log日志不仅回滚时需要，mvcc版本访问时也需要，不会立即删除

- undo log版本链

  不同事务或相同事务对同一条记录进行修改，会导致该记录的undo log生成一条记录版本链表，链表的头部是最新的旧记录，链表尾部是最早的记录

- readview

  ReadView（视图读）是**快照读**SQL执行时MVCC提取数据的依赖，记录并维护系统当前活跃的事务（未提交）id

  |      字段      |              含义              |
  | :------------: | :----------------------------: |
  |     m_ids      |      当前活跃的事务ID集合      |
  |   min_trx_id   |         最小活跃事务ID         |
  |   max_trx_id   | 预分配事务ID，当前最大事务ID+1 |
  | creator_trx_id |     ReadView创建者的事务ID     |

  **RC隔离级别下，在事务中每一次执行快照读时生成ReadView**

  **RR隔离级别下，仅在事务第一次执行快照读时生成ReadView，后续复用该ReadView**

![image-20240521174453239](https://gitee.com/qingy735/blogimg/raw/master/img/image-20240521174453239.png)



### MySQL主从同步

> 二进制日志（binlog）记录了所有的DDL（数据定义语言 create drop等）语句和DML（数据操纵语言 update insert等），但不包括数据查询（select show）语句

- Master主库在事务提交时会把数据变更记录在二进制日志文件Binlog中
- 从库读取主库的binlog文件，写入到从库的中继日志relay log
- 从库重做中继日志中的事件，实现主从同步

### 分库分表

### 垂直分库

> 以表为依据，根据业务将不同表拆分到不同库中

特点：

- 按业务对数据分级管理、维护、监控、扩展
- 在高并发下，提高磁盘IO和数据量连接数

### 垂直分表

> 以字段为依据，根据字段属性将不同字段拆分到不同表中；不常用的字段单独放一张表，把text、blob等大字段拆分出来放在附表中

特点：

- 冷热数据分离
- 减少IO过度争抢，两表互不影响

### 水平分库

> 将一个库的数据拆分到多个库中

路由规则：

- 根据id取模
- 按照id范围路由（1-100w）（100w-200w

特点：

- 解决了单库大数量、高并发的性能瓶颈问题
- 提高系统稳定性和可用性

### 水平分表

> 将一个表的数据拆分到多个表中（可以在同一个库中）

特点：

- 优化单表数据过大产生的性能问题
- 避免IO争抢并减少锁表的几率
