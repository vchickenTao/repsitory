# RabbitMQ

## MQ

***MQ***（***message queue***），从字面意思上看，本质是个队列，***FIFO*** 先入先出，只不过队列中存放的内容是 ***message*** 而已，还是一种跨进程的通信机制，用于上下游传递消息。在互联网架构中，***MQ*** 是一种非常常 见的上下游 ***逻辑解耦+物理解耦*** 的消息通信服务。使用了 ***MQ*** 之后，消息发送上游只需要依赖 ***MQ***，不 用依赖其他服务。



**MQ的3大功能**

> ***流量消峰***
>
> 高峰期，服务器最多能处理的请求达到上限。而使用消息队列做缓存，可以将请求分散成一段时间来处理。
>
> <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2012.23.59.png" alt="" style="zoom:50%;" />
>
> 
>
> ***应用解耦***
>
> 任何一个子模块出现故障，都会造成总系统操作异常。当转变成基于消息队列的方式后，系统间调用的问题会减少很多。当模块 3 出现故障时，需要几分钟来修复。在这几分钟的时间里，模块 3 需要处理的内存被缓存在消息队列中，而模块 1 可以正常完成。当模块 3 恢复后，继续处理队列中的信息即可，而用户感受不到模块 3 的故障，提升系统的可用性。
>
> <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2012.23.50.png" alt="" style="zoom:50%;" />
>
> 
>
> ***异步处理***
>
> 有些服务间调用是异步的，例如 ***A*** 调用 ***B***，***B*** 需要花费很长时间执行，但是 ***A*** 需要知道 ***B*** 什么时候可以执行完成。
>
> 以前一般有两种方式，***A*** 过一段时间去调用 ***B*** 的查询 ***api*** 查询。或者 A 提供一个 ***callback api***， B 执行完之后调用 ***api*** 通知 A 服务。
>
> 这两种方式都不是很优雅，使用消息总线，可以很方便解决这个问题， ***A*** 调用 ***B*** 服务后，只需要监听 ***B*** 处理完成的消息，当 ***B*** 处理完成后，会发送一条消息给 ***MQ***，***MQ*** 会将此消息转发给 A 服务。这样 ***A*** 服务既不用循环调用 ***B*** 的查询 ***api***，也不用提供 ***callback api***。同样 ***B*** 服务也不用做这些操作。***A*** 服务还能及时的得到异步处理成功的消息。
>
> <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2012.23.32.png" style="zoom:50%;" />



**分类**

> ***ActiveMQ***
>
> ***Kafka***
>
> 日志采集。
>
> ***RocketMQ***
>
> 高并发场景。
>
> ***RabbitMQ***
>
> 数据量没有那么大，中小型公司。



## RabbitMQ简介

***RabbitMQ*** 是一个消息中间件：它接受并转发消息。可以将它当做一个快递站点，当需要发送一个包裹时，把包裹放到快递站，快递员最终会把快递送到收件人那里，按照这种逻辑 ***RabbitMQ*** 是一个快递站，一个快递员帮你传递快件。

***RabbitMQ*** 与快递站的主要区别在于，它不处理快件而是接收、存储和转发消息数据。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2012.43.48.png" alt="" style="zoom:50%;" />

**四大核心概念**

> ***生产者***
>
> 产生数据发送消息的程序是生产者。
>
> ***交换机***
>
> 交换机是 ***RabbitMQ*** 非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定。
>
> ***队列***
>
> 队列是 ***RabbitMQ*** 内部使用的一种数据结构，尽管消息流经 ***RabbitMQ*** 和应用程序，但它们只能存储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。
>
> ***消费者***
>
> 消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。



***RabbitMQ*** **核心部分**

> ***Hello World***
>
> ***Work Queues***
>
> ***Publish/Subscribe***
>
> ***Routing***
>
> ***Topics***
>
> ***Publisher Confirms***



***RabbitMQ*** **工作原理**

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2013.34.27.png" style="zoom:50%;" />

> ***Broker***
>
> 接收和分发消息的应用，***RabbitMQ Server*** 就是 ***Message Broker***。
>
> 
>
> ***Virtual host***
>
> 出于多租户和安全因素设计的，把 ***AMQP*** 的基本组件划分到一个虚拟的分组中，类似于网络中的 ***namespace*** 概念。当多个不同的用户使用同一个 ***RabbitMQ server*** 提供的服务时，可以划分出多个 vhost，每个用户在自己的 ***vhost*** 创建 ***exchange/queue*** 等。
>
> 
>
> ***Connection***
>
> ***publisher/consumer*** 和 ***broker*** 之间的 ***TCP*** 连接。
>
> 
>
> **Channel**
>
> 如果每一次访问 ***RabbitMQ*** 都建立一个 ***Connection***，在消息量大的时候建立 ***TCP Connection*** 的开销将是巨大的，效率也较低。***Channel*** 是在 ***connection*** 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 ***thread*** 创建单独的 ***Channel*** 进行通讯，***AMQP method*** 包含了 ***channel id*** 帮助客户端和 ***message broker*** 识别 ***Channel***，所以 ***Channel*** 之间是完全隔离的。***Channel*** 作为轻量级的 ***Connection*** 极大减少了操作系统建立 TCP connection 的开销。
>
> 
>
> ***Exchange***
>
> ***message*** 到达 ***broker*** 的第一站，根据分发规则，匹配查询表中的 ***routing key***，分发消息到 ***queue*** 中去。常用的类型有：***direct (point-to-point)、topic (publish-subscribe) 、fanout(multicast)***
>
> 
>
> ***Queue***
>
> 消息最终被送到这里等待 ***consumer*** 取走。
>
> 
>
> ***Binding***
>
> ***exchange*** 和 ***queue*** 之间的虚拟连接，***binding*** 中可以包含 ***routing key***，***Binding*** 信息被保存到 ***exchange*** 中的查询表中，用于 ***message*** 的分发依据。



# 安装和启动

官网：https://www.rabbitmq.com/download.html



1. ***Erlang*** 语言环境安装

***Erlang*** 语言版本和 ***RabbitMQ*** 版本对应： https://www.rabbitmq.com/which-erlang.html

2. ***FTP*** 上传 ***Erlang*** 和 ***RabbitMQ*** 安装包

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2014.01.41.png" style="zoom:50%;" />



3. 安装

```
rpm -ivh erlang-21.3-1.el7.x86_64.rpm
yum install socat -y
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm
```

4. 开启 ***web*** 图形化界面

```
rabbitmq-plugins enable rabbitmq_management
```

5. 访问 ***ip*** + 端口（默认***15672***）

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2014.36.13.png" style="zoom:50%;" />

6. 默认账号密码

```
username:guest
password:guest

# 显示没有权限登陆
```

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2014.37.37.png" style="zoom:50%;" />

7. 创建新的用户以及授权

> 创建账号
>
> ```
> rabbitmqctl add_user username password
> # rabbitmqctl add_user admin tokyo302
> ```
>
> 
>
> 设置用户角色
>
> ```
> rabbitmqctl set_user_tags username role
> # rabbitmqctl set_user_tags admin administrator
> ```
>
> 
>
> 设置用户权限
>
> ```
> rabbitmqctl set_permissions [-p <vhostpath>] <user> <conf> <write> <read> rabbitmqctl 
> 
> # rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
> ```
>
> 
>
> 查看当前用户和角色
>
> ```
> rabbitmqctl list_users 
> ```

8. 使用新创建的用户成功登陆

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2014.44.06.png" style="zoom:50%;" />





**常用的命令**

> 添加开机启动 ***RabbitMQ*** 服务
>
> ```
> chkconfig rabbitmq-server on
> ```
>
> 
>
> 启动服务
>
> ```
> /sbin/service rabbitmq-server start
> ```
>
> 
>
> 查看服务状态
>
> ```
> /sbin/service rabbitmq-server status
> ```
>
> 
>
> 停止服务（选择执行）
>
> ```
>  /sbin/service rabbitmq-server stop
> ```
>
> 
>
> 开启 ***web*** 管理插件
>
> ```
>  rabbitmq-plugins enable rabbitmq_management
> ```



# *Hello World*

***P***：生产者；

***C***：消费者；

***RabbitMQ***：使用者保留的消息缓冲区。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-23%2014.51.48.png" style="zoom:50%;" />



```
使用 Java 编写两个程序。发送单个消息的生产者和接收消息并打印出来的消费者。将介绍 Java API 中的一些细节。
```



1. 依赖

```xml
    <!-- 指定jdk版本 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <!-- rabbitmq  -->
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.8.0</version>
        </dependency>
        <!-- 操作文件流   -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
    </dependencies>
```

2. 创建消息生产者

```java
public class Producer {
    private final static String QUEUE_NAME = "Hello";
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();

        factory.setHost("172.16.88.168");
        factory.setUsername("admin");
        factory.setPassword("tokyo302");

        // 创建连接
        Connection connection = factory.newConnection();

        // 信道
        Channel channel = connection.createChannel();

        /**
         *  @Author: tsuiraku
         *  @Date: 2021/10/24 下午1:41
         *  @Description: 队列
         *  param1: 队列名称
         *  param2: 队列消息是否持久化（持久化代表存储在磁盘，默认存储在内存）
         *  param3: 队列消息是否共享（true，允许多个消费者访问）
         *  param4: 是否自动删除（当最后一个消费者断开连接后）
         *  param5: 其他参数
         *
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        String msg = "Hello World";

        /**
         *  @Author: tsuiraku
         *  @Date: 2021/10/24 下午1:45
         *  @Description: 发送消息
         *  param1: 交换机（使用默认）
         *  param2: 队列名称
         *  param3: 其他参数
         *  param4: 消息体
         */
        channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());

        System.out.println("消息发送完毕");
    }
}
```

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2013.50.42.png" alt="" style="zoom:50%;" />



3. 创建消息消费者

```java
public class Consumer {
    private final static String QUEUE_NAME = "Hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("172.16.88.168");
        factory.setUsername("admin");
        factory.setPassword("tokyo302");

        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();


        // 声明接受消息
        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("消息：" + new String(message.getBody()));
        };

        // 声明取消消息
        CancelCallback cancelCallback = consumerTag -> {
            System.out.println("被中断！" );
        };

        /**
         *  @Author: tsuiraku
         *  @Date: 2021/10/24 下午1:54
         *  @Description: 消费者消费队列
         *  param1: 消费哪个队列
         *  param2: 消费成功后是否自动应答
         *  param3: 消费者未成功消费的回调
         *  param4: 消费者取消消费的回调
         */
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2014.00.24.png" style="zoom:50%;" />

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2014.00.15.png" style="zoom:50%;" />



# Work Queue

工作队列（又称任务队列）的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。 相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2014.02.55.png" style="zoom:50%;" />



注意：一个消息只能被处理一次，而不能被多次处理。即轮询处理模式。消费者（工作线程）之间的关系为竞争关系。



1. 抽取连接工厂工具类

```java
public class RabbitMqUtils {
    public static Channel getChannel() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("172.16.88.168");
        factory.setUsername("admin");
        factory.setPassword("tokyo302");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        return channel;
    }
}
```

2. 创建两个消费者（工作线程）

```java
public class Work01 {
    private final static String QUEUE_NAME = "Hello";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("接受到的消息：" + new String(message.getBody()));
        };

        CancelCallback cancelCallback = consumerTag -> {
            System.out.println("消息被取消，消息：" + consumerTag);
        };

      	// System.out.println("C1等待接受消息...");
        System.out.println("C2等待接受消息...");
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```

3. 创建生产者

```java
public class Task01 {
    private final static String QUEUE_NAME = "Hello";
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false , null);

        Scanner input = new Scanner(System.in);
        while(input.hasNext()) {
            String msg = input.next();
            channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
            System.out.println("发送" + msg + "成功！" );
        }
    }
}
```

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2014.36.37.png" style="zoom:50%;" />

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2014.37.11.png" style="zoom:50%;" />

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2014.38.01.png" style="zoom:50%;" />

（采用轮询的方式，工作线程依次接受消息队列中的消息）



## 消息应答

消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成 了部分突然它挂掉了，会发生什么情况。***RabbitMQ*** 一旦向消费者传递了一条消息，便立即将该消息标记为删除。在这种情况下，突然有个消费者挂掉了，将丢失正在处理的消息。以及后续发送给该消费这的消息，因为它无法接收到。

为了保证消息在发送过程中不丢失，***Rabbitmq*** 引入消息应答机制，消息应答就是：消费者在接收到消息并且处理该消息之后，告诉 ***Rabbitmq*** 它已经处理了，此时，***Rabbitmq*** 就可以把该消息删除了。



### 自动应答

消息发送后立即被认为已经传送成功，这种模式需要在高吞吐量和数据传输安全性方面做权衡，因为这种模式如果消息在接收到之前，消费者那边出现连接或者 ***Channel*** 关闭，那么消息就丢失了。

当然另一方面这种模式消费者那边可以传递过载的消息，没有对传递的消息数量进行限制，这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使得内存耗尽，最终这些消费者线程被操作系统杀死，所以**<u>这种模式仅适用在消费者可以高效并以某种速率能够处理这些消息的情况下使用</u>**。



### 手动消息应答

#### 消息应答的方法

> `Channel.basicAck`
>
> （用于肯定确认） ***RabbitMQ*** 已知道该消息并且成功的处理消息，可以将其丢弃。
>
> `Channel.basicNack`
>
> （用于否定确认）
>
> `Channel.basicReject`
>
> （用于否定确认）与 ***Channel.basicNack*** 相比少一个参数 ***Multiple***，表示不处理该消息直接拒绝，可以将其丢弃。



#### *Multiple*

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2014.52.55.png" style="zoom:30%;" />

> ***Multiple*** 的 ***true*** 和 ***false*** 代表不同意思
>
> ***true*** 代表批量应答 ***Channel*** 上未应答的消息。
>
> 例如 ***Channel*** 上有传送 ***tag*** 的消息 `5 6 7 8` 当前 ***tag=8*** ，那么此时` 5 6 7 8` 这些还未应答的消息都会被确认收到消息应答 。***false*** 只会应答 ***tag=8*** 的消息 `5 6 7` 这三个消息依然不会被确认收到消息应答。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2015.00.58.png" style="zoom:30%;" />



### 消息自动重新入队

如果消费者由于某些原因失去连接（其通道已关闭，连接已关闭或 ***TCP*** 连接丢失)，导致消息未发送 ***ACK*** 确认，***RabbitMQ*** 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-24%2015.00.24.png" style="zoom:40%;" />



### 消息手动应答代码

应对 **<u>消息自动重新入队</u>** 的实例。

1. 创建生产者

```java
public class Task02 {
    private final static String QUEUE_NAME = "ACK_QUEUE";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        Scanner input = new Scanner(System.in);
        while(input.hasNext()) {
            String msg = input.next();
            channel.basicPublish("", QUEUE_NAME, null, msg.getBytes("UTF-8"));
            System.out.println("生产者生产消息：" + msg + "。");
        }
    }
}
```

2. 创建两个消费者

```java
public class FastWork02 {
    private final static String QUEUE_NAME = "ACK_QUEUE";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C1等待消息处理。");

        DeliverCallback deliverCallback =  (consumerTag, message) -> {
            // 沉睡1s
            SleepUtils.sleep(1);
            System.out.println("C1接受的消息：" + new String(message.getBody(), "UTF-8") + "。");
            /**
             *  @Author: tsuiraku
             *  @Date: 2021/10/24 下午3:39
             *  @Description: 手动应答
             *  param1: 消息的标记 tag
             *  param2: 是否批量应答 multiple
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
        };

        CancelCallback cancelCallback = consumer -> {
            System.out.println("消费者取消消费。");
        };

        channel.basicConsume(QUEUE_NAME, false, deliverCallback, cancelCallback);

    }
}

public class SlowWork02 {
    private final static String QUEUE_NAME = "ACK_QUEUE";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C2等待消息处理。");

        DeliverCallback deliverCallback =  (consumerTag, message) -> {
            // 沉睡10s
            SleepUtils.sleep(10);
            System.out.println("C1接受的消息：" + new String(message.getBody(), "UTF-8") + "。");
            /**
             *  @Author: tsuiraku
             *  @Date: 2021/10/24 下午3:39
             *  @Description: 手动应答
             *  param1: 消息的标记 tag
             *  param2: 是否批量应答 multiple
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
        };

        CancelCallback cancelCallback = consumer -> {
            System.out.println("消费者取消消费。");
        };

        channel.basicConsume(QUEUE_NAME, false, deliverCallback, cancelCallback);

    }
}
```



## RabbitMQ持久化

如何保障当 ***RabbitMQ*** 服务停掉以后消息生产者发送过来的消息不丢失。默认情况下 ***RabbitMQ*** 退出或由于某种原因崩溃时，它忽视队列和消息，除非告知它不要这样做。

确保消息不会丢失需要做两件事：我们需要将队列和消息都标记为持久化。



### 队列实现持久化

如果要将队列实现持久化，那么需要在**<u>声明队列</u>**的时候把 ***durable*** 参数设置为持久化。

```java
boolean durable = true;
channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
```

注意：就是如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新创建一个持久化的队列，不然就会出现错误。



### 消息实现持久化

要想让消息实现持久化需要修改生产者代码，***MessageProperties.PERSISTENT_TEXT_PLAIN*** 。

```java
channel.basicPublish("", QUEUE_Name, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes())
```

将消息标记为持久化并不能完全保证不会丢失消息。尽管 ***RabbitMQ*** 会将消息保存到磁盘，但依然存在当消息刚准备存储在磁盘的时候，但是还没有存储完，消息还在缓存的一个间隔点。此时并没有真正写入磁盘。持久性保证并不强。



### 不公平分发

默认采用**<u>轮训</u>**分发。

但是在某种场景下这种策略并不是很好，例如有两个消费者在处理任务，其中有个消费者 1 处理任务的速度非常快，而另外一个消费者 2 处理速度却很慢，这时如果还是采用轮训分发，就会使得处理速度快消费者 1 产生很大一部分时间处于空闲状态，而处理慢的消费者 2 则一直在干活。

设置参数 ***channel.basicQos(1)***

```java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```











# 感谢

- [尚硅谷-旭哥](https://www.bilibili.com/video/BV1cb4y1o7zz?spm_id_from=333.999.0.0)
