# 认识Redis

Redis诞生于2009年，全称是Remote Dictionary Server，远程词典服务器，是一个基于内存的键值型NoSQL数据库。

特征：

- key-value型，value支持多种不同数据结构，功能丰富
- 单线程，每个命令具备原子性
- 低延迟、速度快（基于内存、IO多路复用、良好的编码）
- 支持数据持久化
- 支持主从集群、分片集群
- 支持多语言客户端

# Redis数据结构介绍

Redis是一个key-value的数据库，key一般是String类型，不过value的类型分为两大类，**基本类型**和**特殊类型**。

基本类型：

- String
- Hash
- List
- Set
- SortedSet

特殊类型：

- GEO
- BitMap
- HyperLog

## Redis通用命令

- KEYS：查看符合模板的所有key**(不要用在生产环境上)**
- DEL：删除一个指定的KEY
- EXISTS：判断KEY是否存在
- EXPIRE：给一个KEY设置有效期，有效期到期该KEY会被自动删除
- TTL：查看一个KEY的剩余有效期

## String类型

字符串类型，是Redis中最简单的存储类型。其value是字符串，不过根据字符串的不同，又可以分为3类：

- string：普通字符串
- int：整形字符串，可以做自增、自减操作
- float：浮点类型，可以做自增、自减操作

不管是哪种格式，底层都是字节数组形式存储，只不过是编码方式不同。**字符串类型的最大空间不能超过512m。**

### String的常见命令有

- SET：添加或者修改已经存在的一个String类型的键值对
- GET：根据key获取String类型的value
- MSET：批量添加多个String类型的键值对
- MGET：根据多个key获取多个String类型的value
- INCR：让一个整型的key自增1
- INCRBY:让一个整型的key自增并指定步长，例如：incrby num 2 让num值自增2
- INCRBYFLOAT：让一个浮点类型的数字自增并指定步长
- SETNX：添加一个String类型的键值对，前提是这个key不存在，否则不执行
- SETEX：添加一个String类型的键值对，并且指定有效期

### key的结构

Redis的key允许有多个单词形成层级结构，多个单词之间用`:`隔开，格式如下：

> 项目名:业务名:类型:id

**这个格式并非固定，可以根据自己的需求来删除或添加词条。**

