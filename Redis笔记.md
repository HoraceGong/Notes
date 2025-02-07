---
title: Redis笔记
date: 2022-11-06 20:09:40
tags: [Redis, Java Web, NoSQL, note]
---

Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。

Redis 与其他 key - value 缓存产品有以下三个特点：

- Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list（列表），set（集合），zset（有序集合），hash（哈希表）等数据结构的存储。
- Redis支持数据的备份，即master-slave（主从模式）模式的数据备份。
- 

<!-- more -->

# Redis数据类型

Redis支持五种数据类型：`string`、`hash`、`list`、`set`、`zset`

- String：Redis的最基本类型，一个key对应一个value。string类型是二进制安全的，也就是说string可以包含任何数据，比如jpg图片或者序列化的对象。**string一个键最大能存储512MB**。
  ```shell
  redis 127.0.0.1:6379> SET name "w3cschool.cn"
  OK
  redis 127.0.0.1:6379> GET name
  "w3cschool.cn"
  ```

- Hash：一个键值对集合。是一个string类型的field和value的映射表，hash特别适合存储对象。
  ```shell
  redis 127.0.0.1:6379> HMSET user:1 username w3cschool.cn password w3cschool.cn points 200
  OK
  redis 127.0.0.1:6379> HGETALL user:1
  1) "username"
  2) "w3cschool.cn"
  3) "password"
  4) "w3cschool.cn"
  5) "points"
  6) "200"
  redis 127.0.0.1:6379>
  ```

  每个hash可以存储 2^32 - 1键值对。

- List：列表是简单的字符串列表，按照插入顺序排序。可以在列表的头部或尾部添加元素。

  ```shell
  redis 127.0.0.1:6379> lpush w3cschool.cn redis
  (integer) 1
  redis 127.0.0.1:6379> lpush w3cschool.cn mongodb
  (integer) 2
  redis 127.0.0.1:6379> lpush w3cschool.cn rabitmq
  (integer) 3
  redis 127.0.0.1:6379> lrange w3cschool.cn 0 10
  1) "rabitmq"
  2) "mongodb"
  3) "redis"
  redis 127.0.0.1:6379>
  ```

  列表最多可存储 2^32 -1元素。

- Set：是string类型的无序集合，因为是通过哈希表实现的，所以添加、删除、查找的复杂度都是O(1)。
  ```shell
  redis 127.0.0.1:6379> sadd w3cschool.cn redis
  (integer) 1
  redis 127.0.0.1:6379> sadd w3cschool.cn mongodb
  (integer) 1
  redis 127.0.0.1:6379> sadd w3cschool.cn rabitmq
  (integer) 1
  redis 127.0.0.1:6379> sadd w3cschool.cn rabitmq
  (integer) 0
  redis 127.0.0.1:6379> smembers w3cschool.cn
  1) "rabitmq"
  2) "mongodb"
  3) "redis"
  ```

  **注意：**以上实例中 `rabitmq` 添加了两次，但根据集合内元素的唯一性，第二次插入的元素将被忽略。

  集合中最大的成员数为 232-1.

- zset：`Redis zset` 和` set` 一样也是 `string` 类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个 `double` 类型的分数。`redis` 正是通过分数来为集合中的成员进行从小到大的排序。`zset` 的成员是唯一的,但分数`(score)`却可以重复。
  ```shell
  redis 127.0.0.1:6379> zadd w3cschool.cn 0 redis
  (integer) 1
  redis 127.0.0.1:6379> zadd w3cschool.cn 0 mongodb
  (integer) 1
  redis 127.0.0.1:6379> zadd w3cschool.cn 0 rabitmq
  (integer) 1
  redis 127.0.0.1:6379> zadd w3cschool.cn 0 rabitmq
  (integer) 0
  redis 127.0.0.1:6379> ZRANGEBYSCORE w3cschool.cn 0 1000
  1) "redis"
  2) "mongodb"
  3) "rabitmq"
  ```




# Redis命令

## Redis键（key）

| 命令                                | 描述                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| DEL key                             | 在key存在时删除key                                           |
| DUMP key                            | 序列化给定key，并返回序列化的值                              |
| EXISIT key                          | 检查给定的key是否存在                                        |
| EXPIRE key seconds                  | 为给定的key设置过期时间                                      |
| EXPIREAT key timestamp              | 用于为给定的key设置过期事件，但是此命令接收的参数是UNXI时间戳 |
| PEXPIRE key milliseconds            | 设置key的过期时间，以毫秒计                                  |
| PEXPIREAT key millisecond-timestamp | 设置key过期时间的时间戳，以毫秒计                            |
| MOVE key db                         | 将当前数据库的key移动到给定的数据库db当中                    |
| PERSIST key                         | 移除key的过期时间，key将持久保持                             |
| PTTL key                            | 以毫秒为单位返回key的剩余过期时间                            |
| TTL key                             | 以秒为单位，返回给定key的剩余生存时间                        |
| RANDOMKEY                           | 从当前数据库中随机返回一个key                                |
| RENAME key newkey                   | 修改key的名称                                                |
| RENAMENX key newkey                 | 仅当newkey不存在时，将key改名为newkey                        |
| TYPE key                            | 返回key所存储的值的类型                                      |



## Redis字符串（String）

| 命令                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| SET key value                  | 设置指定的key的值                                            |
| GET key                        | 获取指定的key的值                                            |
| GETRANGE key start end         | 返回指定范围的字符串子串                                     |
| GETSET key offest              | 将给定的key的值设为value，并返回key的旧值                    |
| GETBIT key offset              | 对key所存的字符串值，获取指定偏移量上的位                    |
| MGET key1[key2..]              | 获取一个或多个key的值                                        |
| SETBIT key offset value        | 对 key 所储存的字符串值，设置或清除指定偏移量上的位          |
| SETTEX key seconds value       | 将value关联到key，并将key的过期时间设为seconds               |
| SETNX key value                | 只有key不存在的时候才设置key的值                             |
| SETRANGE key offset value      | 用value参数覆写给定key所存储的字符串值，从偏移量offset开始   |
| STRLEN key                     | 返回key所存储的字符串值的长度                                |
| MSET key value [key value..]   | 同时设置多个key-value对                                      |
| MSETNX key value [key value..] | 同时设置多个key-value对，仅当key不存在时才执行               |
| PSETEX key milliseconds value  | 以毫秒为单位设置key的过期时间                                |
| INCR key                       | 将key存储的值加一                                            |
| INCRBY key increment           | 将key所存的值加上给定的增量值                                |
| INCRBYFLOAT key increment      | 将key所存的值加上给定的浮点型增量值                          |
| DECR key decrement             | 将key所存的值减一                                            |
| DECRBY key decrement           | 将key所存的值减去给定的增量值                                |
| APPEND key value               | 如果key已经存在并且是一个字符串，将vale追加到key原来值的末尾 |

