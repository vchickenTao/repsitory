# 🌄 Redis - Redis数据类型

---

## Redis数据结构简介

> Redis基础文章非常多，关于基础数据结构类型，我推荐你先看下[官方网站内容](https://redis.io/topics/data-types)，然后再看下面的小结

首先对redis来说，所有的key（键）都是字符串。我们在谈基础数据结构时，讨论的是存储值的数据类型，主要包括常见的5种数据类型，分别是：String、List、Set、Zset、Hash。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/d1ad6a21-6f31-4d6f-84c9-5dcbbdd956ce_db-redis-ds-1.jpeg)

| 结构类型         | 结构存储的值                               | 结构的读写能力                                               |
| ---------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **String字符串** | 可以是字符串、整数或浮点数                 | 对整个字符串或字符串的一部分进行操作；对整数或浮点数进行自增或自减操作； |
| **List列表**     | 一个链表，链表上的每个节点都包含一个字符串 | 对链表的两端进行push和pop操作，读取单个或多个元素；根据值查找或删除元素； |
| **Set集合**      | 包含字符串的无序集合                       | 字符串的集合，包含基础的方法有看是否存在添加、获取、删除；还包含计算交集、并集、差集等 |
| **Hash散列**     | 包含键值对的无序散列表                     | 包含方法有添加、获取、删除单个元素                           |
| **Zset有序集合** | 和散列一样，用于存储键值对                 | 字符串成员与浮点数分数之间的有序映射；元素的排列顺序由分数的大小决定；包含方法有添加、获取、删除单个元素以及根据分值范围或成员来获取元素 |



## 基础数据结构详解

> 内容其实比较简单，我觉得理解的重点在于这个结构怎么用，能够用来做什么？所以我在梳理时，围绕**图例**，**命令**，**执行**和**场景**来阐述。

### String字符串

> String是redis中最基本的数据类型，一个key对应一个value。

- String 类型是二进制安全的。意味着 Redis 的 string 可以包含任何数据。比如 jpg 图片或者序列化的对象。
- String 类型是 Redis 最基本的数据类型，一个 Redis 中字符串 value 最多可以是 512M。

#### 数据结构

String 的数据结构为简单动态字符串 (Simple Dynamic String, 缩写 SDS)，是可以修改的字符串，内部结构实现上类似于 Java 的 ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配.

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/be121104-cff5-47a8-8e94-598424eba7f2_redis-base-03.png)

如图中所示，内部为当前字符串实际分配的空间 capacity 一般要高于实际字符串长度 len。当字符串长度小于 1M 时，扩容都是加倍现有的空间，如果超过 1M，扩容时一次只会多扩 1M 的空间。需要注意的是字符串最大长度为 512M。

下图是一个String类型的实例，其中键为hello，值为world

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/08264f3b-4991-4d86-844d-8da9b03846fd_db-redis-ds-3.png)

- **命令使用**

| 命令   | 简述                   | 使用              |
| ------ | ---------------------- | ----------------- |
| GET    | 获取存储在给定键中的值 | GET name          |
| SET    | 设置存储在给定键中的值 | SET name value    |
| DEL    | 删除存储在给定键中的值 | DEL name          |
| INCR   | 将键存储的值加1        | INCR key          |
| DECR   | 将键存储的值减1        | DECR key          |
| INCRBY | 将键存储的值加上整数   | INCRBY key amount |
| DECRBY | 将键存储的值减去整数   | DECRBY key amount |

- **命令执行**

```bash
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> del hello
(integer) 1
127.0.0.1:6379> get hello
(nil)
127.0.0.1:6379> set counter 2
OK
127.0.0.1:6379> get counter 
"2"
127.0.0.1:6379> incr counter
(integer) 3
127.0.0.1:6379> get counter 
"3"
127.0.0.1:6379> incrby counter 100
(integer) 103
127.0.0.1:6379> get counter 
"103"
127.0.0.1:6379> decr counter
(integer) 102
127.0.0.1:6379> decrby counter 100
(integer) 2
127.0.0.1:6379> get counter 
"2"
```

- 实战场景
  - **缓存**： 经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis作为缓存层，mysql做持久化层，降低mysql的读写压力。
  - **计数器**：redis是单线程模型，一个命令执行完才会执行下一个，同时数据可以一步落地到其他的数据源。
  - **session**：常见方案spring session + redis实现session共享，

### List列表

> Redis中的List其实就是链表（Redis用双端链表实现List）。

使用List结构，我们可以轻松地实现最新消息排队功能（比如新浪微博的TimeLine）。List的另一个应用就是消息队列，可以利用List的 PUSH 操作，将任务存放在List中，然后工作线程再用 POP 操作将任务取出进行执行。

- **图例**

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/09ae130d-c5ea-4ee7-8473-033decad1623_db-redis-ds-5.png)

- **数据结构**

  List 的数据结构为快速链表 quickList。

  首先在列表元素较少的情况下会使用一块**连续的内存存储**，这个结构是 ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。

  当数据量比较多的时候才会改成 quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是 int 类型的数据，结构上还需要两个额外的指针 prev 和 next。

  ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/17ccd036-79b2-479c-852a-ca00f34f126a_redis-base-04.png)

  Redis 将链表和 ziplist 结合起来组成了 quicklist。也就是将多个 ziplist 使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

  

- **命令使用**

| 命令   | 简述                                                         | 使用             |
| ------ | ------------------------------------------------------------ | ---------------- |
| RPUSH  | 将给定值推入到列表右端                                       | RPUSH key value  |
| LPUSH  | 将给定值推入到列表左端                                       | LPUSH key value  |
| RPOP   | 从列表的右端弹出一个值，并返回被弹出的值                     | RPOP key         |
| LPOP   | 从列表的左端弹出一个值，并返回被弹出的值                     | LPOP key         |
| LRANGE | 获取列表在给定范围上的所有值                                 | LRANGE key 0 -1  |
| LINDEX | 通过索引获取列表中的元素。你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。 | LINDEX key index |

- 使用列表的技巧
  - lpush+lpop=Stack(栈)
  - lpush+rpop=Queue（队列）
  - lpush+ltrim=Capped Collection（有限集合）
  - lpush+brpop=Message Queue（消息队列）
- **命令执行**

```bash
127.0.0.1:6379> lpush mylist 1 2 ll ls mem
(integer) 5
127.0.0.1:6379> lrange mylist 0 -1
1) "mem"
2) "ls"
3) "ll"
4) "2"
5) "1"
127.0.0.1:6379> lindex mylist -1
"1"
127.0.0.1:6379> lindex mylist 10        # index不在 mylist 的区间范围内
(nil)
```

- 实战场景
  - **微博TimeLine**: 有人发布微博，用lpush加入时间轴，展示新的列表信息。
  - **消息队列**

### Set集合

> Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

- Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

- Redis 的 Set 是 string 类型的无序集合。它底层其实是一个 value 为 null 的 hash 表，所以添加，删除，查找的复杂度都是 O (1)。

  

- **图例**

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/b0ca27e7-867a-4aec-81ee-849ae5fbbf8b_db-redis-ds-7.png)

- **数据结构**
  - Set 数据结构是 dict 字典，字典是用哈希表实现的。
  - Java 中 HashSet 的内部实现使用的是 HashMap，只不过所有的 value 都指向同一个对象。Redis 的 set 结构也是一样，它的内部也使用 hash 结构，所有的 value 都指向同一个内部值。
- **命令使用**

| 命令      | 简述                                  | 使用                 |
| --------- | ------------------------------------- | -------------------- |
| SADD      | 向集合添加一个或多个成员              | SADD key value       |
| SCARD     | 获取集合的成员数                      | SCARD key            |
| SMEMBERS  | 返回集合中的所有成员                  | SMEMBERS key member  |
| SISMEMBER | 判断 member 元素是否是集合 key 的成员 | SISMEMBER key member |

其它一些集合操作，请参考这里https://www.runoob.com/redis/redis-sets.html

- **命令执行**

```bash
127.0.0.1:6379> sadd myset hao hao1 xiaohao hao
(integer) 3
127.0.0.1:6379> smembers myset
1) "xiaohao"
2) "hao1"
3) "hao"
127.0.0.1:6379> sismember myset hao
(integer) 1
```

- 实战场景
  - **标签**（tag）,给用户添加标签，或者用户给消息添加标签，这样有同一标签或者类似标签的可以给推荐关注的事或者关注的人。
  - **点赞，或点踩，收藏等**，可以放到set中实现

### Hash散列

> Redis hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。

- 类似 Java 里面的 Map<String,Object>。
- 用户 ID 为查找的 key，存储的 value 用户对象包含姓名，年龄，生日等信息，如果用普通的 key/value 结构来存储，主要有以下 2 种存储方式：


![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/5e695128-5ecd-49f8-8c7a-560a486cf258_redis-base-05.png)

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/11e08088-74d9-41bb-8db4-b63b5b1f189b_redis-base-07.png)

方法一：每次修改用户的某个属性需要，先反序列化改好后再序列化回去。开销较大。

方法二：用户 ID 数据冗余。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/dce9c1c1-a51c-40c6-802b-c55aa0290f4a_redis-base-06.png)

通过 key (用户 ID) + field (属性标签) 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题。



- **图例**

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/6400c556-6f2c-46c3-8001-df23496185fe_db-redis-ds-4.png)

- **数据结构**

  Hash 类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当 field-value 长度较短且个数较少时，使用 ziplist，否则使用 hashtable。

  

- **命令使用**

| 命令    | 简述                                     | 使用                          |
| ------- | ---------------------------------------- | ----------------------------- |
| HSET    | 添加键值对                               | HSET hash-key sub-key1 value1 |
| HGET    | 获取指定散列键的值                       | HGET hash-key key1            |
| HGETALL | 获取散列中包含的所有键值对               | HGETALL hash-key              |
| HDEL    | 如果给定键存在于散列中，那么就移除这个键 | HDEL hash-key sub-key1        |

- **命令执行**

```bash
127.0.0.1:6379> hset user name vchicken
(integer) 1
127.0.0.1:6379> hset user email vchicken@163.com
(integer) 1
127.0.0.1:6379> hget user name
"vchicken"
127.0.0.1:6379> hgetall user
1) "name"
2) "vchicken"
3) "email"
4) "vchicken@163.com"
127.0.0.1:6379> hset use name1 bill
(integer) 1
127.0.0.1:6379> hset user email1 bill@163.com
(integer) 1
127.0.0.1:6379> hgetall user
1) "name"
2) "vchicken"
3) "email"
4) "vchicken@163.com"
5) "email1"
6) "bill@163.com"
```

- 实战场景
  - **缓存**： 能直观，相比string更节省空间，适合用来缓存信息，如用户信息，视频信息等。



### Zset有序集合

> Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。不同之处是有序集合的每个成员都关联了一个评分（score），这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复。
>

有序集合是通过两种数据结构实现：

1. **压缩列表(ziplist)**: ziplist是为了提高存储效率而设计的一种特殊编码的双向链表。它可以存储字符串或者整数，存储整数时是采用整数的二进制而不是字符串形式存储。它能在O(1)的时间复杂度下完成list两端的push和pop操作。但是因为每次操作都需要重新分配ziplist的内存，所以实际复杂度和ziplist的内存使用量相关
2. **跳跃表（zSkiplist)**: 跳跃表的性能可以保证在查找，删除，添加等操作的时候在对数期望时间内完成，这个性能是可以和平衡树来相比较的，而且在实现方面比平衡树要优雅，这是采用跳跃表的主要原因。跳跃表的复杂度是O(log(n))。

- **图例**

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/15c2de71-a126-4323-8f2b-b77624ea2c0b_db-redis-ds-8.png)

- **数据结构**

  SortedSet (zset) 是 Redis 提供的一个非常特别的数据结构，一方面它等价于 Java 的数据结构 Map<String, Double>，可以给每一个元素 value 赋予一个权重 score，另一方面它又类似于 TreeSet，内部的元素会按照权重 score 进行排序，可以得到每个元素的名次，还可以通过 score 的范围来获取元素的列表。

  zset 底层使用了两个数据结构：

  - hash，hash 的作用就是关联元素 value 和权重 score，保障元素 value 的唯一性，可以通过元素 value 找到相应的 score 值。
  - 跳跃表，跳跃表的目的在于给元素 value 排序，根据 score 的范围获取元素列表。

  

- **命令使用**

| 命令   | 简述                                                     | 使用                           |
| ------ | -------------------------------------------------------- | ------------------------------ |
| ZADD   | 将一个带有给定分值的成员添加到有序集合里面               | ZADD zset-key 178 member1      |
| ZRANGE | 根据元素在有序集合中所处的位置，从有序集合中获取多个元素 | ZRANGE zset-key 0-1 withccores |
| ZREM   | 如果给定元素成员存在于有序集合中，那么就移除这个元素     | ZREM zset-key member1          |

更多命令请参考这里 https://www.runoob.com/redis/redis-sorted-sets.html

- **命令执行**

```bash
127.0.0.1:6379> zadd myscoreset 100 hao 90 xiaohao
(integer) 2
127.0.0.1:6379> ZRANGE myscoreset 0 -1
1) "xiaohao"
2) "hao"
127.0.0.1:6379> ZSCORE myscoreset hao
"100"
```

- 实战场景
  - **排行榜**：有序集合经典使用场景。例如小说视频等网站需要对用户上传的小说视频做排行榜，榜单可以按照用户关注数，更新时间，字数等打分，做排行。



### 跳跃表

#### 简介

有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis 采用的是跳跃表，跳跃表效率堪比红黑树，实现远比红黑树简单。

#### 实例

对比有序链表和跳跃表，从链表中查询出 51：

**有序链表**

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/cbbd77c2-05be-4bb1-8417-7ac0f597cced_redis-base-08.png)

要查找值为 51 的元素，需要从第一个元素开始依次查找、比较才能找到。共需要 6 次比较。

**跳跃表**

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-13/d4fcc74a-84da-41c1-8f5f-449bdfe4e1bb_redis-base-09.png)

- 从第 2 层开始，1 节点比 51 节点小，向后比较；
- 21 节点比 51 节点小，继续向后比较，后面就是 NULL 了，所以从 21 节点向下到第 1 层；
- 在第 1 层，41 节点比 51 节点小，继续向后，61 节点比 51 节点大，所以从 41 向下；
- 在第 0 层，51 节点为要查找的节点，节点被找到，共查找 4 次。

从此可以看出跳跃表比有序链表效率要高。



## 参考文章

- http://ddrv.cn/a/260579
- https://www.cnblogs.com/haoprogrammer/p/11065461.html
- https://www.pianshen.com/article/6479421770/
- https://www.runoob.com/redis/redis-sorted-sets.html
- [Java 全栈知识体系](https://www.pdai.tech/md/db/nosql-redis/db-redis-data-types.html)