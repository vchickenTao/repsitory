# 🌆 Redis - Stream结构详解

---

# 为什么会设计Stream

> Redis5.0 中还增加了一个数据结构Stream，从字面上看是流类型，但其实从功能上看，应该是Redis对消息队列（MQ，Message Queue）的完善实现。

用过Redis做消息队列的都了解，基于Reids的消息队列实现有很多种，例如：

- PUB/SUB，订阅/发布模式
  - 但是发布订阅模式是无法持久化的，如果出现网络断开、Redis 宕机等，消息就会被丢弃；
- 基于`List LPUSH+BRPOP`或者 基于`Sorted-Set`的实现
  - 支持了持久化，但是不支持多播，分组消费等

为什么上面的结构无法满足广泛的MQ场景？ 这里便引出一个核心的问题：如果我们期望设计一种数据结构来实现消息队列，最重要的就是要理解**设计一个消息队列需要考虑什么**？初步的我们很容易想到

- 消息的生产
- 消息的消费
  - 单播和多播（多对多）
  - 阻塞和非阻塞读取
- 消息有序性
- 消息的持久化

其它还要考虑啥嗯？借助美团技术团队的一篇文章，[消息队列设计精要](https://tech.meituan.com/2016/07/01/mq-design.html) 中的图

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-16/8a1b4093-cbcc-4cf8-8820-a29563c906e6_db-redis-stream-1.png)

**我们不妨看看Redis考虑了哪些设计**？

- 消息ID的序列化生成
- 消息遍历
- 消息的阻塞和非阻塞读取
- 消息的分组消费
- 未完成消息的处理
- 消息队列监控
- ...

> 这也是我们需要理解Stream的点，但是结合上面的图，我们也应该理解Redis Stream也是一种超轻量MQ并没有完全实现消息队列所有设计要点，这决定着它适用的场景。

# Stream详解

> 经过梳理总结，我认为从以下几个大的方面去理解Stream是比较合适的，总结如下：@pdai

- Stream的结构设计
- 生产和消费
  - 基本的增删查改
  - 单一消费者的消费
  - 消费组的消费
- 监控状态

## Stream的结构

每个 Stream 都有唯一的名称，它就是 Redis 的 key，在我们首次使用 xadd 指令追加消息时自动创建。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-16/98e42327-3089-481c-84d4-76dcaad31606_db-redis-stream-2.png)

上图解析：

- `Consumer Group` ：消费组，使用 XGROUP CREATE 命令创建，一个消费组有多个消费者(Consumer), 这些消费者之间是竞争关系。
- `last_delivered_id` ：游标，每个消费组会有个游标 last_delivered_id，任意一个消费者读取了消息都会使游标 last_delivered_id 往前移动。
- `pending_ids` ：消费者(Consumer)的状态变量，作用是维护消费者的未确认的 id。 pending_ids 记录了当前已经被客户端读取的消息，但是还没有 `ack` (Acknowledge character：确认字符）。如果客户端没有ack，这个变量里面的消息ID会越来越多，一旦某个消息被ack，它就开始减少。这个pending_ids变量在Redis官方被称之为PEL，也就是Pending Entries List，这是一个很核心的数据结构，它用来确保客户端至少消费了消息一次，而不会在网络传输的中途丢失了没处理。

此外我们还需要理解两点：

- `消息ID`: 消息ID的形式是timestampInMillis-sequence，例如1527846880572-5，它表示当前的消息在毫米时间戳1527846880572时产生，并且是该毫秒内产生的第5条消息。消息ID可以由服务器自动生成，也可以由客户端自己指定，但是形式必须是整数-整数，而且必须是后面加入的消息的ID要大于前面的消息ID。
- `消息内容`: 消息内容就是键值对，形如hash结构的键值对，这没什么特别之处。

## 增删改查

消息队列相关命令：

- XADD - 添加消息到末尾
- XTRIM - 对流进行修剪，限制长度
- XDEL - 删除消息
- XLEN - 获取流包含的元素数量，即消息长度
- XRANGE - 获取消息列表，会自动过滤已经删除的消息
- XREVRANGE - 反向获取消息列表，ID 从大到小
- XREAD - 以阻塞或非阻塞方式获取消息列表

```bash
# *号表示服务器自动生成ID，后面顺序跟着一堆key/value
127.0.0.1:6379> xadd codehole * name laoqian age 30  #  名字叫laoqian，年龄30岁
1527849609889-0  # 生成的消息ID
127.0.0.1:6379> xadd codehole * name xiaoyu age 29
1527849629172-0
127.0.0.1:6379> xadd codehole * name xiaoqian age 1
1527849637634-0
127.0.0.1:6379> xlen codehole
(integer) 3
127.0.0.1:6379> xrange codehole - +  # -表示最小值, +表示最大值
127.0.0.1:6379> xrange codehole - +
1) 1) 1527849609889-0
   1) 1) "name"
      1) "laoqian"
      2) "age"
      3) "30"
2) 1) 1527849629172-0
   1) 1) "name"
      1) "xiaoyu"
      2) "age"
      3) "29"
3) 1) 1527849637634-0
   1) 1) "name"
      1) "xiaoqian"
      2) "age"
      3) "1"
127.0.0.1:6379> xrange codehole 1527849629172-0 +  # 指定最小消息ID的列表
1) 1) 1527849629172-0
   2) 1) "name"
      2) "xiaoyu"
      3) "age"
      4) "29"
2) 1) 1527849637634-0
   2) 1) "name"
      2) "xiaoqian"
      3) "age"
      4) "1"
127.0.0.1:6379> xrange codehole - 1527849629172-0  # 指定最大消息ID的列表
1) 1) 1527849609889-0
   2) 1) "name"
      2) "laoqian"
      3) "age"
      4) "30"
2) 1) 1527849629172-0
   2) 1) "name"
      2) "xiaoyu"
      3) "age"
      4) "29"
127.0.0.1:6379> xdel codehole 1527849609889-0
(integer) 1
127.0.0.1:6379> xlen codehole  # 长度不受影响
(integer) 3
127.0.0.1:6379> xrange codehole - +  # 被删除的消息没了
1) 1) 1527849629172-0
   2) 1) "name"
      2) "xiaoyu"
      3) "age"
      4) "29"
2) 1) 1527849637634-0
   2) 1) "name"
      2) "xiaoqian"
      3) "age"
      4) "1"
127.0.0.1:6379> del codehole  # 删除整个Stream
(integer) 1  
```

## 独立消费

我们可以在不定义消费组的情况下进行Stream消息的独立消费，当Stream没有新消息时，甚至可以阻塞等待。Redis设计了一个单独的消费指令xread，可以将Stream当成普通的消息队列(list)来使用。使用xread时，我们可以完全忽略消费组(Consumer Group)的存在，就好比Stream就是一个普通的列表(list)。

```bash
# 从Stream头部读取两条消息
127.0.0.1:6379> xread count 2 streams codehole 0-0
1) 1) "codehole"
   2) 1) 1) 1527851486781-0
         2) 1) "name"
            2) "laoqian"
            3) "age"
            4) "30"
      2) 1) 1527851493405-0
         2) 1) "name"
            2) "yurui"
            3) "age"
            4) "29"
# 从Stream尾部读取一条消息，毫无疑问，这里不会返回任何消息
127.0.0.1:6379> xread count 1 streams codehole $
(nil)
# 从尾部阻塞等待新消息到来，下面的指令会堵住，直到新消息到来
127.0.0.1:6379> xread block 0 count 1 streams codehole $
# 我们从新打开一个窗口，在这个窗口往Stream里塞消息
127.0.0.1:6379> xadd codehole * name youming age 60
1527852774092-0
# 再切换到前面的窗口，我们可以看到阻塞解除了，返回了新的消息内容
# 而且还显示了一个等待时间，这里我们等待了93s
127.0.0.1:6379> xread block 0 count 1 streams codehole $
1) 1) "codehole"
   2) 1) 1) 1527852774092-0
         2) 1) "name"
            2) "youming"
            3) "age"
            4) "60"
(93.11s) 
```

客户端如果想要使用xread进行顺序消费，一定要记住当前消费到哪里了，也就是返回的消息ID。下次继续调用xread时，将上次返回的最后一个消息ID作为参数传递进去，就可以继续消费后续的消息。

block 0表示永远阻塞，直到消息到来，block 1000表示阻塞1s，如果1s内没有任何消息到来，就返回nil

```bash
127.0.0.1:6379> xread block 1000 count 1 streams codehole $
(nil)
(1.07s)
```

## 消费组消费

- **消费组消费图**

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-16/9c06eb4d-6e3b-426a-8b08-4056af566613_db-redis-stream-3.png)

- 相关命令：
  - XGROUP CREATE - 创建消费者组
  - XREADGROUP GROUP - 读取消费者组中的消息
  - XACK - 将消息标记为"已处理"
  - XGROUP SETID - 为消费者组设置新的最后递送消息ID
  - XGROUP DELCONSUMER - 删除消费者
  - XGROUP DESTROY - 删除消费者组
  - XPENDING - 显示待处理消息的相关信息
  - XCLAIM - 转移消息的归属权
  - XINFO - 查看流和消费者组的相关信息；
  - XINFO GROUPS - 打印消费者组的信息；
  - XINFO STREAM - 打印流信息
- **创建消费组**

Stream通过xgroup create指令创建消费组(Consumer Group)，需要传递起始消息ID参数用来初始化last_delivered_id变量。

```bash
127.0.0.1:6379> xgroup create codehole cg1 0-0  #  表示从头开始消费
OK
# $表示从尾部开始消费，只接受新消息，当前Stream消息会全部忽略
127.0.0.1:6379> xgroup create codehole cg2 $
OK
127.0.0.1:6379> xinfo stream codehole  # 获取Stream信息
 1) length
 2) (integer) 3  # 共3个消息
 3) radix-tree-keys
 4) (integer) 1
 5) radix-tree-nodes
 6) (integer) 2
 7) groups
 8) (integer) 2  # 两个消费组
 9) first-entry  # 第一个消息
10) 1) 1527851486781-0
    2) 1) "name"
       2) "laoqian"
       3) "age"
       4) "30"
11) last-entry  # 最后一个消息
12) 1) 1527851498956-0
    2) 1) "name"
       2) "xiaoqian"
       3) "age"
       4) "1"
127.0.0.1:6379> xinfo groups codehole  # 获取Stream的消费组信息
1) 1) name
   2) "cg1"
   3) consumers
   4) (integer) 0  # 该消费组还没有消费者
   5) pending
   6) (integer) 0  # 该消费组没有正在处理的消息
2) 1) name
   2) "cg2"
   3) consumers  # 该消费组还没有消费者
   4) (integer) 0
   5) pending
   6) (integer) 0  # 该消费组没有正在处理的消息 
```

- **消费组消费**

Stream提供了xreadgroup指令可以进行消费组的组内消费，需要提供消费组名称、消费者名称和起始消息ID。它同xread一样，也可以阻塞等待新消息。读到新消息后，对应的消息ID就会进入消费者的PEL(正在处理的消息)结构里，客户端处理完毕后使用xack指令通知服务器，本条消息已经处理完毕，该消息ID就会从PEL中移除。

```bash
# >号表示从当前消费组的last_delivered_id后面开始读
# 每当消费者读取一条消息，last_delivered_id变量就会前进
127.0.0.1:6379> xreadgroup GROUP cg1 c1 count 1 streams codehole >
1) 1) "codehole"
   2) 1) 1) 1527851486781-0
         2) 1) "name"
            2) "laoqian"
            3) "age"
            4) "30"
127.0.0.1:6379> xreadgroup GROUP cg1 c1 count 1 streams codehole >
1) 1) "codehole"
   2) 1) 1) 1527851493405-0
         2) 1) "name"
            2) "yurui"
            3) "age"
            4) "29"
127.0.0.1:6379> xreadgroup GROUP cg1 c1 count 2 streams codehole >
1) 1) "codehole"
   2) 1) 1) 1527851498956-0
         2) 1) "name"
            2) "xiaoqian"
            3) "age"
            4) "1"
      2) 1) 1527852774092-0
         2) 1) "name"
            2) "youming"
            3) "age"
            4) "60"
# 再继续读取，就没有新消息了
127.0.0.1:6379> xreadgroup GROUP cg1 c1 count 1 streams codehole >
(nil)
# 那就阻塞等待吧
127.0.0.1:6379> xreadgroup GROUP cg1 c1 block 0 count 1 streams codehole >
# 开启另一个窗口，往里塞消息
127.0.0.1:6379> xadd codehole * name lanying age 61
1527854062442-0
# 回到前一个窗口，发现阻塞解除，收到新消息了
127.0.0.1:6379> xreadgroup GROUP cg1 c1 block 0 count 1 streams codehole >
1) 1) "codehole"
   2) 1) 1) 1527854062442-0
         2) 1) "name"
            2) "lanying"
            3) "age"
            4) "61"
(36.54s)
127.0.0.1:6379> xinfo groups codehole  # 观察消费组信息
1) 1) name
   2) "cg1"
   3) consumers
   4) (integer) 1  # 一个消费者
   5) pending
   6) (integer) 5  # 共5条正在处理的信息还有没有ack
2) 1) name
   2) "cg2"
   3) consumers
   4) (integer) 0  # 消费组cg2没有任何变化，因为前面我们一直在操纵cg1
   5) pending
   6) (integer) 0
# 如果同一个消费组有多个消费者，我们可以通过xinfo consumers指令观察每个消费者的状态
127.0.0.1:6379> xinfo consumers codehole cg1  # 目前还有1个消费者
1) 1) name
   2) "c1"
   3) pending
   4) (integer) 5  # 共5条待处理消息
   5) idle
   6) (integer) 418715  # 空闲了多长时间ms没有读取消息了
# 接下来我们ack一条消息
127.0.0.1:6379> xack codehole cg1 1527851486781-0
(integer) 1
127.0.0.1:6379> xinfo consumers codehole cg1
1) 1) name
   2) "c1"
   3) pending
   4) (integer) 4  # 变成了5条
   5) idle
   6) (integer) 668504
# 下面ack所有消息
127.0.0.1:6379> xack codehole cg1 1527851493405-0 1527851498956-0 1527852774092-0 1527854062442-0
(integer) 4
127.0.0.1:6379> xinfo consumers codehole cg1
1) 1) name
   2) "c1"
   3) pending
   4) (integer) 0  # pel空了
   5) idle
   6) (integer) 745505
```

## 信息监控

Stream提供了XINFO来实现对服务器信息的监控，可以查询：

- 查看队列信息

```bash
127.0.0.1:6379> Xinfo stream codehole
 1) "length"
 2) (integer) 6
 3) "radix-tree-keys"
 4) (integer) 1
 5) "radix-tree-nodes"
 6) (integer) 2
 7) "last-generated-id"
 8) "1665888202093-0"
 9) "max-deleted-entry-id"
10) "0-0"
11) "entries-added"
12) (integer) 6
13) "recorded-first-entry-id"
14) "1665886670676-0"
15) "groups"
16) (integer) 2
17) "first-entry"
18) 1) "1665886670676-0"
    2) 1) "name"
       2) "laoqian"
       3) "age"
       4) "30"
19) "last-entry"
20) 1) "1665888202093-0"
    2) 1) "name"
       2) "lanying"
       3) "age"
       4) "61"
```

- 消费组信息

```bash
127.0.0.1:6379> Xinfo groups codehole
1)  1) "name"
    2) "cg1"
    3) "consumers"
    4) (integer) 1
    5) "pending"
    6) (integer) 4
    7) "last-delivered-id"
    8) "1665888202093-0"
    9) "entries-read"
   10) (integer) 6
   11) "lag"
   12) (integer) 0
2)  1) "name"
    2) "cg2"
    3) "consumers"
    4) (integer) 0
    5) "pending"
    6) (integer) 0
    7) "last-delivered-id"
    8) "1665886927948-0"
    9) "entries-read"
   10) (nil)
   11) "lag"
   12) (nil)
```

- 消费者组成员信息

```bash
127.0.0.1:6379> XINFO CONSUMERS mq mqGroup
1) 1) "name"
   2) "consumerA"
   3) "pending"
   4) (integer) 1
   5) "idle"
   6) (integer) 18949894
2) 1) "name"
   2) "consumerB"
   3) "pending"
   4) (integer) 1
   5) "idle"
   6) (integer) 3092719
3) 1) "name"
   2) "consumerC"
   3) "pending"
   4) (integer) 1
   5) "idle"
   6) (integer) 23683256
```

至此，消息队列的操作说明大体结束！



# SpringBoot集成Stream示例

## 一、背景

`Stream`类型是 `redis5`之后新增的类型，在这篇文章中，我们实现使用`Spring boot data redis`来消费`Redis Stream`中的数据。实现独立消费和消费组消费。

## 二 、配置步骤

### 1、引入jar包

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.11.1</version>
  </dependency>
</dependencies>
```

主要是上方的这个包，其他的不相关的包此处省略导入。

### 2、配置RedisTemplate依赖

```java
@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        // 这个地方不可使用 json 序列化，如果使用的是ObjectRecord传输对象时，可能会有问题，会出现一个 java.lang.IllegalArgumentException: Value must not be null! 错误
        redisTemplate.setHashValueSerializer(RedisSerializer.string());
        return redisTemplate;
    }
}
```

**注意：**

此处需要注意 `setHashValueSerializer` 的序列化的方式，具体注意事项后期再说。

### 3、准备一个实体对象

这个实体对象是需要发送到`Stream`中的对象。

```java
@Getter
@Setter
@ToString
public class Book {
    private String title;
    private String author;
    
    public static Book create() {
        com.github.javafaker.Book fakerBook = Faker.instance().book();
        Book book = new Book();
        book.setTitle(fakerBook.title());
        book.setAuthor(fakerBook.author());
        return book;
    }
}
```

每次调用`create`方法时，会自动产生一个`Book`的对象，对象模拟数据是使用`javafaker`来模拟生成的。

### 4、编写一个常量类，配置Stream的名称

```java
/**
 * 常量
 *
 */
public class Cosntants {
    
    public static final String STREAM_KEY_001 = "stream-001";
    
}
```

### 5、编写一个生产者，向Stream中生产数据

#### 1、编写一个生产者，向Stream中产生ObjectRecord类型的数据

```java
/**
 * 消息生产者
 
 */
@Component
@RequiredArgsConstructor
@Slf4j
public class StreamProducer {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    public void sendRecord(String streamKey) {
        Book book = Book.create();
        log.info("产生一本书的信息:[{}]", book);
        
        ObjectRecord<String, Book> record = StreamRecords.newRecord()
                .in(streamKey)
                .ofObject(book)
                .withId(RecordId.autoGenerate());
        
        RecordId recordId = redisTemplate.opsForStream()
                .add(record);
        
        log.info("返回的record-id:[{}]", recordId);
    }
}
```

#### 2、每隔5s就生产一个数据到Stream中

```java
/**
 * 周期性的向流中产生消息
 */
@Component
@AllArgsConstructor
public class CycleGeneratorStreamMessageRunner implements ApplicationRunner {
    
    private final StreamProducer streamProducer;
    
    @Override
    public void run(ApplicationArguments args) {
        Executors.newSingleThreadScheduledExecutor()
                .scheduleAtFixedRate(() -> streamProducer.sendRecord(STREAM_KEY_001),
                        0, 5, TimeUnit.SECONDS);
    }
}
```

## 三、独立消费

独立消费指的是脱离消费组的直接消费`Stream`中的消息，是使用 `xread`方法读取流中的数据，流中的数据在读取后并不会被删除，还是存在的。如果多个程序同时使用`xread`读取，都是可以读取到消息的。

### 1、实现从头开始消费-xread实现

此处实现的是从Stream的第一个消息开始消费

```java
package com.huan.study.redis.stream.consumer.xread;

import com.huan.study.redis.constan.Cosntants;
import com.huan.study.redis.entity.Book;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.data.redis.connection.stream.ObjectRecord;
import org.springframework.data.redis.connection.stream.ReadOffset;
import org.springframework.data.redis.connection.stream.StreamOffset;
import org.springframework.data.redis.connection.stream.StreamReadOptions;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import javax.annotation.Resource;
import java.time.Duration;
import java.util.List;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * 脱离消费组-直接消费Stream中的数据，可以获取到Stream中所有的消息
 */
@Component
@Slf4j
public class XreadNonBlockConsumer01 implements InitializingBean, DisposableBean {
    
    private ThreadPoolExecutor threadPoolExecutor;
    
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    
    private volatile boolean stop = false;
    
    @Override
    public void afterPropertiesSet() {
        
        // 初始化线程池
        threadPoolExecutor = new ThreadPoolExecutor(1, 1, 0, TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(), r -> {
            Thread thread = new Thread(r);
            thread.setDaemon(true);
            thread.setName("xread-nonblock-01");
            return thread;
        });
        
        StreamReadOptions streamReadOptions = StreamReadOptions.empty()
                // 如果没有数据，则阻塞1s 阻塞时间需要小于`spring.redis.timeout`配置的时间
                .block(Duration.ofMillis(1000))
                // 一直阻塞直到获取数据，可能会报超时异常
                // .block(Duration.ofMillis(0))
                // 1次获取10个数据
                .count(10);
        
        StringBuilder readOffset = new StringBuilder("0-0");
        threadPoolExecutor.execute(() -> {
            while (!stop) {
                // 使用xread读取数据时，需要记录下最后一次读取到offset，然后当作下次读取的offset，否则读取出来的数据会有问题
                List<ObjectRecord<String, Book>> objectRecords = redisTemplate.opsForStream()
                        .read(Book.class, streamReadOptions, StreamOffset.create(Cosntants.STREAM_KEY_001, ReadOffset.from(readOffset.toString())));
                if (CollectionUtils.isEmpty(objectRecords)) {
                    log.warn("没有获取到数据");
                    continue;
                }
                for (ObjectRecord<String, Book> objectRecord : objectRecords) {
                    log.info("获取到的数据信息 id:[{}] book:[{}]", objectRecord.getId(), objectRecord.getValue());
                    readOffset.setLength(0);
                    readOffset.append(objectRecord.getId());
                }
            }
        });
    }
    
    @Override
    public void destroy() throws Exception {
        stop = true;
        threadPoolExecutor.shutdown();
        threadPoolExecutor.awaitTermination(3, TimeUnit.SECONDS);
    }
}
```

**注意：**

下一次读取数据时，offset 是上一次最后获取到的id的值，否则可能会出现漏数据。

### 2、StreamMessageListenerContainer实现独立消费

见下方的消费组消费的代码

## 四、消费组消费

### 1、实现StreamListener接口

实现这个接口的目的是为了，消费`Stream`中的数据。需要注意在注册时使用的是`streamMessageListenerContainer.receiveAutoAck()`还是`streamMessageListenerContainer.receive()`方法，如果是第二个，则需要`手动ack`，手动ack的代码：`redisTemplate.opsForStream().acknowledge("key","group","recordId");`

```java
/**
 * 通过监听器异步消费
 *
 * @author vchicken
 */
@Slf4j
@Getter
@Setter
public class AsyncConsumeStreamListener implements StreamListener<String, ObjectRecord<String, Book>> {
    /**
     * 消费者类型：独立消费、消费组消费
     */
    private String consumerType;
    /**
     * 消费组
     */
    private String group;
    /**
     * 消费组中的某个消费者
     */
    private String consumerName;
    
    public AsyncConsumeStreamListener(String consumerType, String group, String consumerName) {
        this.consumerType = consumerType;
        this.group = group;
        this.consumerName = consumerName;
    }
    
    private RedisTemplate<String, Object> redisTemplate;
    
    @Override
    public void onMessage(ObjectRecord<String, Book> message) {
        String stream = message.getStream();
        RecordId id = message.getId();
        Book value = message.getValue();
        if (StringUtils.isBlank(group)) {
            log.info("[{}]: 接收到一个消息 stream:[{}],id:[{}],value:[{}]", consumerType, stream, id, value);
        } else {
            log.info("[{}] group:[{}] consumerName:[{}] 接收到一个消息 stream:[{}],id:[{}],value:[{}]", consumerType,
                    group, consumerName, stream, id, value);
        }
        
        // 当是消费组消费时，如果不是自动ack，则需要在这个地方手动ack
        // redisTemplate.opsForStream()
        //         .acknowledge("key","group","recordId");
    }
}
```

### 2、获取消费或消费消息过程中错误的处理

```java
/**
 * StreamPollTask 获取消息或对应的listener消费消息过程中发生了异常
 *
 * @author vchicken
 */
@Slf4j
public class CustomErrorHandler implements ErrorHandler {
    @Override
    public void handleError(Throwable t) {
        log.error("发生了异常", t);
    }
}
```

### 3、消费组配置

```java
/**
 * redis stream 消费组配置
 *
 * @author vchicken
 */
@Configuration
public class RedisStreamConfiguration {
    
    @Resource
    private RedisConnectionFactory redisConnectionFactory;
    
    /**
     * 可以同时支持 独立消费 和 消费者组 消费
     * <p>
     * 可以支持动态的 增加和删除 消费者
     * <p>
     * 消费组需要预先创建出来
     *
     * @return StreamMessageListenerContainer
     */
    @Bean(initMethod = "start", destroyMethod = "stop")
    public StreamMessageListenerContainer<String, ObjectRecord<String, Book>> streamMessageListenerContainer() {
        AtomicInteger index = new AtomicInteger(1);
        int processors = Runtime.getRuntime().availableProcessors();
        ThreadPoolExecutor executor = new ThreadPoolExecutor(processors, processors, 0, TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(), r -> {
            Thread thread = new Thread(r);
            thread.setName("async-stream-consumer-" + index.getAndIncrement());
            thread.setDaemon(true);
            return thread;
        });
        
        StreamMessageListenerContainer.StreamMessageListenerContainerOptions<String, ObjectRecord<String, Book>> options =
                StreamMessageListenerContainer.StreamMessageListenerContainerOptions
                        .builder()
                        // 一次最多获取多少条消息
                        .batchSize(10)
                        // 运行 Stream 的 poll task
                        .executor(executor)
                        // 可以理解为 Stream Key 的序列化方式
                        .keySerializer(RedisSerializer.string())
                        // 可以理解为 Stream 后方的字段的 key 的序列化方式
                        .hashKeySerializer(RedisSerializer.string())
                        // 可以理解为 Stream 后方的字段的 value 的序列化方式
                        .hashValueSerializer(RedisSerializer.string())
                        // Stream 中没有消息时，阻塞多长时间，需要比 `spring.redis.timeout` 的时间小
                        .pollTimeout(Duration.ofSeconds(1))
                        // ObjectRecord 时，将 对象的 filed 和 value 转换成一个 Map 比如：将Book对象转换成map
                        .objectMapper(new ObjectHashMapper())
                        // 获取消息的过程或获取到消息给具体的消息者处理的过程中，发生了异常的处理
                        .errorHandler(new CustomErrorHandler())
                        // 将发送到Stream中的Record转换成ObjectRecord，转换成具体的类型是这个地方指定的类型
                        .targetType(Book.class)
                        .build();
        
        StreamMessageListenerContainer<String, ObjectRecord<String, Book>> streamMessageListenerContainer =
                StreamMessageListenerContainer.create(redisConnectionFactory, options);
        
        // 独立消费
        String streamKey = Cosntants.STREAM_KEY_001;
        streamMessageListenerContainer.receive(StreamOffset.fromStart(streamKey),
                new AsyncConsumeStreamListener("独立消费", null, null));
        
        // 消费组A,不自动ack
        // 从消费组中没有分配给消费者的消息开始消费
        streamMessageListenerContainer.receive(Consumer.from("group-a", "consumer-a"),
                StreamOffset.create(streamKey, ReadOffset.lastConsumed()), new AsyncConsumeStreamListener("消费组消费", "group-a", "consumer-a"));
        // 从消费组中没有分配给消费者的消息开始消费
        streamMessageListenerContainer.receive(Consumer.from("group-a", "consumer-b"),
                StreamOffset.create(streamKey, ReadOffset.lastConsumed()), new AsyncConsumeStreamListener("消费组消费A", "group-a", "consumer-b"));
        
        // 消费组B,自动ack
        streamMessageListenerContainer.receiveAutoAck(Consumer.from("group-b", "consumer-a"),
                StreamOffset.create(streamKey, ReadOffset.lastConsumed()), new AsyncConsumeStreamListener("消费组消费B", "group-b", "consumer-bb"));
        
        // 如果需要对某个消费者进行个性化配置在调用register方法的时候传递`StreamReadRequest`对象
        
        return streamMessageListenerContainer;
    }
}
```

**注意：**

提前建立好消费组

```accesslog
127.0.0.1:6379> xgroup create stream-001 group-a $
OK
127.0.0.1:6379> xgroup create stream-001 group-b $
OK
```

#### 1、独有消费配置

```reasonml
 streamMessageListenerContainer.receive(StreamOffset.fromStart(streamKey), new AsyncConsumeStreamListener("独立消费", null, null));
```

不传递`Consumer`即可。

#### 2、配置消费组-不自动ack消息

```reasonml
streamMessageListenerContainer.receive(Consumer.from("group-a", "consumer-b"),
                StreamOffset.create(streamKey, ReadOffset.lastConsumed()), new AsyncConsumeStreamListener("消费组消费A", "group-a", "consumer-b"));
```

1、需要注意`ReadOffset`的取值。

2、需要注意`group`需要提前创建好。

#### 3、配置消费组-自动ack消息

```erlang
streamMessageListenerContainer.receiveAutoAck()
```

## 五、序列化策略

| Stream Property | Serializer          | Description                            |
| --------------- | ------------------- | -------------------------------------- |
| key             | keySerializer       | used for `Record#getStream()`          |
| field           | hashKeySerializer   | used for each map key in the payload   |
| value           | hashValueSerializer | used for each map value in the payload |

## 六、`ReadOffset`策略

消费消息时的Read Offset 策略

![ReadOffset策略](https://segmentfault.com/img/remote/1460000040946714)

| Read offset         | Standalone                                                   | Consumer Group                                               |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Latest              | Read latest message（读取最新的消息）                        | Read latest message（读取最新的消息）                        |
| Specific Message Id | Use last seen message as the next MessageId<br/>（读取大于指定的消息id的消息） | Use last seen message as the next MessageId<br/>（读取大于指定的消息id的消息） |
| Last Consumed       | Use last seen message as the next MessageId<br/>（读取大于指定的消息id的消息） | Last consumed message as per consumer group<br/>（读取还未分配给消费组中的消费组的消息） |

## 七、注意事项

### 1、读取消息的超时时间

当我们使用 `StreamReadOptions.empty().block(Duration.ofMillis(1000))` 配置阻塞时间时，这个配置的阻塞时间必须要比 `spring.redis.timeout`配置的时间短，否则可能会报超时异常。

### 2、ObjectRecord反序列化错误

如果我们在读取消息时发生如下异常，那么排查思路如下：

```stylus
java.lang.IllegalArgumentException: Value must not be null!
    at org.springframework.util.Assert.notNull(Assert.java:201)
    at org.springframework.data.redis.connection.stream.Record.of(Record.java:81)
    at org.springframework.data.redis.connection.stream.MapRecord.toObjectRecord(MapRecord.java:147)
    at org.springframework.data.redis.core.StreamObjectMapper.toObjectRecord(StreamObjectMapper.java:138)
    at org.springframework.data.redis.core.StreamObjectMapper.toObjectRecords(StreamObjectMapper.java:164)
    at org.springframework.data.redis.core.StreamOperations.map(StreamOperations.java:594)
    at org.springframework.data.redis.core.StreamOperations.read(StreamOperations.java:413)
    at com.huan.study.redis.stream.consumer.xread.XreadNonBlockConsumer02.lambda$afterPropertiesSet$1(XreadNonBlockConsumer02.java:61)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:748)
```

1、检测 `RedisTemplate`的`HashValueSerializer`的序列化方式，最好不要使用`json`可以使用`RedisSerializer.string()`。

2、检查`redisTemplate.opsForStream()`中配置的`HashMapper`，默认是`ObjectHashMapper`这个是把对象字段和值序列化成`byte[]`格式。

**提供一个可用的配置**

```reasonml
# RedisTemplate的hash value 使用string类型的序列化方式
redisTemplate.setHashValueSerializer(RedisSerializer.string());
# 这个方法opsForStream()里面使用默认的ObjectHashMapper
redisTemplate.opsForStream()
```

关于上面的这个错误，我在`Spring Data Redis`的官方仓库提了一个 [issue](https://link.segmentfault.com/?enc=Adja53lQnwi2uD7qHs23nA%3D%3D.GKSAwS6%2B1y2o5sQ0%2F6vgjQ4%2B6SgLqKn%2FC%2BADKHWmTvzCUbanEwOEANTv0NlFt%2FTYd7h0ODBtbPyV4Qnl9ra1Eo4Nnb8UDY0h31r9zBtFiUw%3D)，得到官方的回复是，这是一个bug，后期会修复的。

![官方回答](https://segmentfault.com/img/remote/1460000040954005)

### 3、使用xread顺序读取数据漏数据

如果我们使用`xread`读取数据发现有写数据漏掉了，这个时候我们需要检查第二次读取时配置的`StreamOffset`是否合法，这个值需要是上一次读取的最后一个值。

**举例说明：**

1、`SteamOffset`传递的是 `$` 表示读取最新的一个数据。

2、处理上一步读取到的数据，此时另外的生产者又向`Stream`中插入了几个数据，这个时候读取到的数据还没有处理完。

3、再次读取`Stream`中的数据，还是传递的`$`，那么表示还是读取最新的数据。那么在上一步流入到Stream中的数据，这个消费者就读取不到了，因为它读取的是最新的数据。

### 4、`StreamMessageListenerContainer`的使用

1、可以动态的添加和删除消费者

2、可以进行消费组消费

3、可以直接独立消费

4、如果传输ObjectRecord的时候，需要注意一下序列化方式。参考上面的代码。



# 更深入理解

> 我们结合MQ中常见问题，看Redis是如何解决的，来进一步理解Redis。

## Stream用在什么样场景

可用作时通信等，大数据分析，异地数据备份等

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-16/1325c2eb-6d21-46c5-8b18-01162311cd9a_db-redis-stream-4.png)

客户端可以平滑扩展，提高处理能力

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-16/a44e4198-e160-4d3b-8f5b-c5a7e356b472_db-redis-stream-5.png)

## 消息ID的设计是否考虑了时间回拨的问题？

> 在 [分布式算法 - ID算法](https://www.pdai.tech/md/algorithm/alg-domain-id-snowflake.html)设计中, 一个常见的问题就是时间回拨问题，那么Redis的消息ID设计中是否考虑到这个问题呢？

XADD生成的1553439850328-0，就是Redis生成的消息ID，由两部分组成:**时间戳-序号**。时间戳是毫秒级单位，是生成消息的Redis服务器时间，它是个64位整型（int64）。序号是在这个毫秒时间点内的消息序号，它也是个64位整型。

可以通过multi批处理，来验证序号的递增：

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> XADD memberMessage * msg one
QUEUED
127.0.0.1:6379> XADD memberMessage * msg two
QUEUED
127.0.0.1:6379> XADD memberMessage * msg three
QUEUED
127.0.0.1:6379> XADD memberMessage * msg four
QUEUED
127.0.0.1:6379> XADD memberMessage * msg five
QUEUED
127.0.0.1:6379> EXEC
1) "1553441006884-0"
2) "1553441006884-1"
3) "1553441006884-2"
4) "1553441006884-3"
5) "1553441006884-4"
```

由于一个redis命令的执行很快，所以可以看到在同一时间戳内，是通过序号递增来表示消息的。

为了保证消息是有序的，因此Redis生成的ID是单调递增有序的。由于ID中包含时间戳部分，为了避免服务器时间错误而带来的问题（例如服务器时间延后了），Redis的每个Stream类型数据都维护一个latest_generated_id属性，用于记录最后一个消息的ID。**若发现当前时间戳退后（小于latest_generated_id所记录的），则采用时间戳不变而序号递增的方案来作为新消息ID**（这也是序号为什么使用int64的原因，保证有足够多的的序号），从而保证ID的单调递增性质。

强烈建议使用Redis的方案生成消息ID，因为这种时间戳+序号的单调递增的ID方案，几乎可以满足你全部的需求。但同时，记住ID是支持自定义的，别忘了！

## 消费者崩溃带来的会不会消息丢失问题?

为了解决组内消息读取但处理期间消费者崩溃带来的消息丢失问题，STREAM 设计了 Pending 列表，用于记录读取但并未处理完毕的消息。命令XPENDIING 用来获消费组或消费内消费者的未处理完毕的消息。演示如下：

```bash
127.0.0.1:6379> XPENDING mq mqGroup # mpGroup的Pending情况
1) (integer) 5 # 5个已读取但未处理的消息
2) "1553585533795-0" # 起始ID
3) "1553585533795-4" # 结束ID
4) 1) 1) "consumerA" # 消费者A有3个
      2) "3"
   2) 1) "consumerB" # 消费者B有1个
      2) "1"
   3) 1) "consumerC" # 消费者C有1个
      2) "1"

127.0.0.1:6379> XPENDING mq mqGroup - + 10 # 使用 start end count 选项可以获取详细信息
1) 1) "1553585533795-0" # 消息ID
   2) "consumerA" # 消费者
   3) (integer) 1654355 # 从读取到现在经历了1654355ms，IDLE
   4) (integer) 5 # 消息被读取了5次，delivery counter
2) 1) "1553585533795-1"
   2) "consumerA"
   3) (integer) 1654355
   4) (integer) 4
# 共5个，余下3个省略 ...

127.0.0.1:6379> XPENDING mq mqGroup - + 10 consumerA # 在加上消费者参数，获取具体某个消费者的Pending列表
1) 1) "1553585533795-0"
   2) "consumerA"
   3) (integer) 1641083
   4) (integer) 5
# 共3个，余下2个省略 ...
```

每个Pending的消息有4个属性：

- 消息ID
- 所属消费者
- IDLE，已读取时长
- delivery counter，消息被读取次数

上面的结果我们可以看到，我们之前读取的消息，都被记录在Pending列表中，说明全部读到的消息都没有处理，仅仅是读取了。那如何表示消费者处理完毕了消息呢？使用命令 XACK 完成告知消息处理完成，演示如下：

```bash
127.0.0.1:6379> XACK mq mqGroup 1553585533795-0 # 通知消息处理结束，用消息ID标识
(integer) 1

127.0.0.1:6379> XPENDING mq mqGroup # 再次查看Pending列表
1) (integer) 4 # 已读取但未处理的消息已经变为4个
2) "1553585533795-1"
3) "1553585533795-4"
4) 1) 1) "consumerA" # 消费者A，还有2个消息处理
      2) "2"
   2) 1) "consumerB"
      2) "1"
   3) 1) "consumerC"
      2) "1"
127.0.0.1:6379>
```

有了这样一个Pending机制，就意味着在某个消费者读取消息但未处理后，消息是不会丢失的。等待消费者再次上线后，可以读取该Pending列表，就可以继续处理该消息了，保证消息的有序和不丢失。

## 消费者彻底宕机后如何转移给其它消费者处理？

> 还有一个问题，就是若某个消费者宕机之后，没有办法再上线了，那么就需要将该消费者Pending的消息，转义给其他的消费者处理，就是消息转移。

消息转移的操作时将某个消息转移到自己的Pending列表中。使用语法XCLAIM来实现，需要设置组、转移的目标消费者和消息ID，同时需要提供IDLE（已被读取时长），只有超过这个时长，才能被转移。演示如下：

```bash
# 当前属于消费者A的消息1553585533795-1，已经15907,787ms未处理了
127.0.0.1:6379> XPENDING mq mqGroup - + 10
1) 1) "1553585533795-1"
   2) "consumerA"
   3) (integer) 15907787
   4) (integer) 4

# 转移超过3600s的消息1553585533795-1到消费者B的Pending列表
127.0.0.1:6379> XCLAIM mq mqGroup consumerB 3600000 1553585533795-1
1) 1) "1553585533795-1"
   2) 1) "msg"
      2) "2"

# 消息1553585533795-1已经转移到消费者B的Pending中。
127.0.0.1:6379> XPENDING mq mqGroup - + 10
1) 1) "1553585533795-1"
   2) "consumerB"
   3) (integer) 84404 # 注意IDLE，被重置了
   4) (integer) 5 # 注意，读取次数也累加了1次
```

以上代码，完成了一次消息转移。转移除了要指定ID外，还需要指定IDLE，保证是长时间未处理的才被转移。被转移的消息的IDLE会被重置，用以保证不会被重复转移，以为可能会出现将过期的消息同时转移给多个消费者的并发操作，设置了IDLE，则可以避免后面的转移不会成功，因为IDLE不满足条件。例如下面的连续两条转移，第二条不会成功。

```bash
127.0.0.1:6379> XCLAIM mq mqGroup consumerB 3600000 1553585533795-1
127.0.0.1:6379> XCLAIM mq mqGroup consumerC 3600000 1553585533795-1   
```

这就是消息转移。至此我们使用了一个Pending消息的ID，所属消费者和IDLE的属性，还有一个属性就是消息被读取次数，delivery counter，该属性的作用由于统计消息被读取的次数，包括被转移也算。这个属性主要用在判定是否为错误数据上。

## 坏消息问题，Dead Letter，死信问题

正如上面所说，如果某个消息，不能被消费者处理，也就是不能被XACK，这是要长时间处于Pending列表中，即使被反复的转移给各个消费者也是如此。此时该消息的delivery counter就会累加（上一节的例子可以看到），当累加到某个我们预设的临界值时，我们就认为是坏消息（也叫死信，DeadLetter，无法投递的消息），由于有了判定条件，我们将坏消息处理掉即可，删除即可。删除一个消息，使用XDEL语法，演示如下：

```bash
# 删除队列中的消息
127.0.0.1:6379> XDEL mq 1553585533795-1
(integer) 1
# 查看队列中再无此消息
127.0.0.1:6379> XRANGE mq - +
1) 1) "1553585533795-0"
   2) 1) "msg"
      2) "1"
2) 1) "1553585533795-2"
   2) 1) "msg"
      2) "3"
```

注意本例中，并没有删除Pending中的消息因此你查看Pending，消息还会在。可以执行XACK标识其处理完毕！



# 参考文章

- [Java 全栈知识体系](https://www.pdai.tech/md/db/nosql-redis/db-redis-data-type-stream.html)
- https://segmentfault.com/a/1190000040946712

