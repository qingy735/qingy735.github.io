---
title: Java八股-Redis篇
tags:
  - Redis
  - Java
categories: 八股
abbrlink: 45296
date: 2024-05-17 20:32:11
---



# Java八股-Redis篇

## 使用场景

### 缓存
#### 缓存穿透
> 查询一个不存在的数据，mysql查询不到数据也不会直接写入缓存，每次请求都会查数据库

解决方案：
- 缓存空数据，查询返回的数据为空，仍然把空结果进行缓存（简单、消耗内存，存在不一致问题）
- 布隆过滤器（缓存预热时初始化）
通过多个hash函数获取hash，将hash结果对应数组位置改为1

#### 缓存击穿
> 当某一个key设置了过期时间，当key过期时，恰好有大量请求访问这个key，并发请求压垮数据库

解决方案：
- 互斥锁
保证数据强一致性，性能差
- 逻辑过期
热点key不设置过期时间。发现逻辑过期，获取互斥锁，重开线程进行缓存重建并更新过期时间。高可用、性能优，不保证数据绝对一致

#### 缓存雪崩
> 同一个时间段内缓存key同时失效或者redis宕机，导致大量请求到达数据库

解决方案：
- 给不同的key的过期时间添加随机值（同时过期）
- 利用redis集群提高服务可用性
- 给缓存业务添加降级限流策略（保底策略；防止缓存穿透、击穿、雪崩）
- 给业务添加多级缓存

#### mysql数据如何和redis进行同步（双写一致性）

##### 保证强一致性，先删除缓存还是数据库

> 延迟双删（数据库主从同步时间，延时时间不好把控且延时过程中会出现脏数据）、redis读写锁

- 先删除缓存再删除数据库

  ![image-20240517205825138](https://gitee.com/qingy735/blogimg/raw/master/img/image-20240517205825138.png)


- 先删除数据库再删除缓存

  ![image-20240517205950034](https://gitee.com/qingy735/blogimg/raw/master/img/image-20240517205950034.png)

##### 保证弱一致性

异步通知，保证最终一致性（消息队列，Canal中间件）



#### redis持久化
- RDB（文件小，恢复速度快，数据丢失多）
> 把内存中的所有数据都记录到磁盘中，redis故障重启后，从磁盘读取快照文件并恢复数据

bgsave执行原理：
通过**fork**创建子进程复制页表信息，可以访问到磁盘数据。同时采用**copy-on-write** 技术，对磁盘数据进行**read-only** 设置，主进程写操作时拷贝数据副本进行改写，同时读取时也读取副本内容。

- AOF（追加文件，文件大小大，恢复速度慢，数据丢失少）
> 记录每次执行命令


结合两者使用：首先使用RDB进行数据恢复，在使用AOF的增量数据进行数据恢复

#### redis数据过期策略
- 惰性删除（CPU友好，内存占用大）
设置key过期时间后不去管他，需要使用时检查是否过期，过期立马删除，否则返回

- 定期删除
每隔一段时间对一些key进行检查，删除里面过期的key

redis删除策略：**惰性删除**+**定期删除**

#### redis数据淘汰策略
> 内存不够时，继续添加新的key，redis会按照某种规则将内存中的数据删掉

- noeviction（默认策略，不淘汰，禁止写入）
- volatile-ttl（ttl越小越先淘汰）
- allkeys-random（随机淘汰）
- volatile-random（对设置了ttl的的key，随机淘汰）
- allkeys-lru（所有key LRU）
- volatile-lru（设置了ttl的key LRU）
- allkeys-lfu（所有key LFU）
- volatile-lfu（设置了ttl的key LFU）


### 分布式锁
> 分布式环境下synchronized锁无法实现加锁功能
#### setnx

#### redisson（lua脚本 原子性）
加锁（成功），操作redis，另起一个线程（watch dog）每隔一段时间(默认10秒)做一次过期时间的续期
加锁（失败），watch dog执行while循环不断获取锁直到达到等待时间
> redisson加锁可重入，根据线程id，利用hash结构记录线程id和重入次数key-(<threadId, count>)；不能解决主从数据一致性问题

## 其他面试题
### redis主从同步
#### 全量同步
- 从节点发送数据同步请求并附带replid和offset
- 主节点判断replid是否一致，不一致表示第一次同步，返回自身replid和offset至从节点；一致则直接将repl_backlog发送给从节点，从节点根据对比offset进行数据同步
- replid不一致，从节点保存主节点回传的replid和offset，同时主节点执行bgsave生成RDB文件并发送，从节点清空本地数据加载RDB，最后执行主节点发送的repl_backlog文件中的命令

#### 增量同步（slave重启或后期数据变化）
> 从节点发送请求，主节点判断是否第一次请求，不是则从repl_backlog中获取offset后的数据发送给从节点，从节点执行命令

### 哨兵模式

![image-20240517222059017](https://gitee.com/qingy735/blogimg/raw/master/img/image-20240517222059017.png)





![image-20240517222128710](https://gitee.com/qingy735/blogimg/raw/master/img/image-20240517222128710.png)



![image-20240517222145269](https://gitee.com/qingy735/blogimg/raw/master/img/image-20240517222145269.png)



### 分片集群

![image-20240517222522412](https://gitee.com/qingy735/blogimg/raw/master/img/image-20240517222522412.png)



### Redis单线程为什么这么快

- Redis是纯内存操作，执行速度快
- 采用单线程，避免了不必要的上下文切换，多线程还要考虑线程安全问题
- 使用I/O多路复用模型，非阻塞IO

​	Redis性能瓶颈是**网络延迟**
