## 数据结构

- String :其内部是动态字符创，可以修改该可以扩容，每次扩容1M，最大512M
  	 get set mget exists del setnx incr incrby
- List: 类似于linkedlist，是个双端对列，可以左右弹出(插入删除时间复杂度O(1)，查找O(n))
  	rpush LPOP rpop lrange lindex
- Hash:类似于java的hashmap
  	hset hget hgetall hincrby
- Set:类似于java hashSet
  	add smembers sismember scard spop
- Zset:它类似于 Java 的 SortedSet 和 HashMap 的结合体，一方面它是一个 set，保证了内部 value 的唯一性，另一方面它可以给每个 value 赋予score，代表这个 value 的排序权重。它的内部实现用的是一种叫做「跳跃列表」的数据结构。
  	zadd zrange zrevrange zrank

## 分布式锁

1. set 指令的扩展参数，使得 setnx 和 expire 指令可以一起执行，彻底解决了分布式锁的乱象
2. set lock:codehole true ex 10 nx

## 持久化方式

AOF:保存Redis服务器锁执行的写状态来记录数据库的
RDB:在内存中的的状态保存到硬盘中，可以手动触发和定期执行
			SAVE：阻塞redis的服务器进程，直到RDB文件被创建完毕
			BGSAVE：派生(fork)一个子进程来创建新的RDB文件

## 过期策略

- 定时过期：到过期时间就会立即清除
- 懒过期：访问一个key时，过期则清除
- 定期过期：每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key

## 内存淘汰

- noeviction：不淘汰，满了报错
- volatile-lru：设置过期时间使用最少的KEY
- volatile-ttl: 淘汰TTL 即将过期的KEY
- volatile-random: 随机TTL KEY
- allkeys-lru ：全体KEY LRU
- allkeys-random : 全体KEY 随机

## 事物

MULTI 队列开始 EXEC 执行 形成一个批量操作，没有回滚操作，错误的命令后的操作继续执行
WATCH （事物基于乐观锁实现，如果事物执行期间，指定Watch 的key的数据被改动，事务会执行失败）

## 缓存

缓存穿透：由于缓存是不命中后查询DB
缓存雪崩：缓存时间同一时间过期导致大量查询直接打到DB
缓存击穿：热点数据缓存失效