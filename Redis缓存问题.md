在高并发的业务场景下，数据库大多数情况都是用户并发访问最薄弱的环节。所以，就需要使用Redis做一个缓冲操作，让请求先访问到Redis，而不是直接访问Mysql数据库。这样可以大大缓解数据库的压力。

# 缓存穿透

缓存穿透是指**缓存和数据库中没有的数据**，而用户不断发起请求。由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，对存储层造成较大压力，失去了缓存的意义。

比如发起id为"-1"的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

## 解决方案

1. 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截。
2. 从缓存取不到的数据，在数据库中也没有取到，这时可以将key-value写为`key-null`，并**设置较短缓存有效时间**，如30秒。这样可以防止用户反复用同一个id暴力攻击。
3. **布隆过滤器**：BloomFilter就类似于一个hash set， 用于快速判断某个元素是否存在于集合中，其典型的应用场景就是快速判断某个key是否存在于某容器，不存在就直接返回。布隆过滤器的关键就在于**hash算法和容器大小**。



# 缓存击穿

缓存击穿是指**缓存中没有但是数据库中有的数据**（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库取数据，引起数据库压力瞬间增大。

## 解决方案

1. 设置**热点数据永不过期**。
2. **接口限流与熔断、降级**：重要的接口一定要做好限流策略，防止用户恶意刷接口，同时要降级准备，当接口中的某些服务不可用时，进行熔断，失败快速返回机制。
3. 加互斥锁



# 缓存雪崩

缓存雪崩是指缓存中**数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大**。和缓存击穿不同的是，**缓存击穿指并发查一条数据，缓存雪崩是不同数据都过期了**，很多数据都查不到从而查数据库。

## 解决方案

1. 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
2. 如果缓存数据库是分布式部署，将热点数据均匀分布在不同的缓存数据库中。
3. 设置热点数据永不过期。

# 缓存污染

缓存污染问题说的是缓存中一些只会被访问一次或者几次的的数据，被访问完后，再也不会被访问到，但这部分数据依然留存在缓存中，消耗缓存空间。

缓存污染会随着数据的持续增加而逐渐显露，随着服务的不断运行，缓存中会存在大量的永远不会再次被访问的数据。缓存空间是有限的，如果缓存空间满了，再往缓存里写数据时就会有额外开销，影响Redis性能。这部分额外开销主要是指写的时候判断淘汰策略，根据淘汰策略去选择要淘汰的数据，然后进行删除操作。

# 最大缓存设置

系统的设计选择是一个权衡的过程：大容量缓存是能带来性能加速的收益，但是成本也会更高，而小容量缓存不一定就起不到加速访问的效果。一般来说，**我会建议把缓存容量设置为总数据量的 15% 到 30%，兼顾访问性能和内存空间开销**。

对于 Redis 来说，一旦确定了缓存最大容量，比如 4GB，你就可以使用下面这个命令来设定缓存的大小了：

```bash
CONFIG SET maxmemory 4gb
```

不过，缓存被写满是不可避免的, 所以需要数据淘汰策略。

# 缓存淘汰策略

Redis共支持八种淘汰策略，分别是noeviction、volatile-random、volatile-ttl、volatile-lru、volatile-lfu、allkeys-lru、allkeys-random 和 allkeys-lfu 策略。

主要分为三类：

- 不淘汰
  - noeviction（4.0后默认）
- 对设置了过期时间的数据进行淘汰
  - 随机：volatile-random
  - ttl：volatile-ttl
  - lru: volatile-lru
  - lfu: volatile-lfu
- 全部数据进行淘汰
  - 随机：allkeys-random
  - lru: allkeys-lru
  - lfu: allkeys-lfu

## noeviction

该策略是Redis的默认策略。在这种策略下，一旦缓存被写满了，再有写请求来时，Redis 不再提供服务，而是直接返回错误。这种策略不会淘汰数据，所以无法解决缓存污染问题。一般生产环境不建议使用。

## volatile-random

**在设置了过期时间的键值对中，进行随机删除**。因为是随机删除，无法把不再访问的数据筛选出来，所以可能依然会存在缓存污染现象。

## volatile-ttl

这种算法判断淘汰数据是参考的指标比随机删除时多进行一步**过期时间的排序**。Redis在筛选需要删除的数据时，越早过期的数据越优先被选中。

## volatile-lru

# 数据库和缓存一致性

读取缓存步骤一般没有什么问题，但是一旦涉及到数据更新：数据库和缓存更新，就容易出现缓存(Redis)和数据库（MySQL）间的数据一致性问题。

**不管是先写MySQL数据库，再删除Redis缓存；还是先删除缓存，再写库，都有可能出现数据不一致的情况**。举一个例子：

1.如果删除了缓存Redis，还没有来得及写库MySQL，另一个线程就来读取，发现缓存为空，则去数据库中读取数据写入缓存，此时缓存中为脏数据。

2.如果先写了库，在删除缓存前，写库的线程宕机了，没有删除掉缓存，则也会出现数据不一致情况。

因为写和读是并发的，没法保证顺序,就会出现缓存和数据库的数据不一致的问题。

## Cache Aside Pattern

- **读的时候**：先读缓存，缓存没有的话，就读数据库。然后取出数据库后放入缓存，同时返回响应。
- **更新的时候**：先更新数据库，然后再删除缓存。

其具体逻辑如下：

- **失效**：应用程序先从cache取数据，没有得到，则从数据库中取数据，成功后，放入缓存中。
- **命中**：应用程序从cache中取数据，取到后返回
- **更新**：先把数据存到数据库中，成功后，再让缓存失效

### 原来先删除Redis数据的问题：（待更新）

先删除Redis - 同时存在读和写 - 写删除了Redis - 此时读没有读到Redis数据，去数据库中取出 - 此时写操作将新数据写入数据库 - 读操作将旧数据写入Redis中 - 二者数据不一致

### 现在先修改数据库中的数据的情况：

数据库没有修改成功之前，读操作会继续读旧数据；数据库修改成功之后，哪怕是读操作找不到Redis中的数据，也可以去数据库中将最新的数据取出，不存在数据不一致的情况。

### Cache Aside Pattern存在的问题



## 队列+重试机制



## 异步更新缓存（基于订阅binlog的同步机制）
