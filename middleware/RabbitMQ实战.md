# RabbitMQ实战
---

- [RabbitMQ消息队列--1 -谷粒商城](https://blog.csdn.net/qq_43284469/article/details/120914682?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167677395816782429735695%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=167677395816782429735695&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-120914682-null-null.blog_rank_default&utm_term=rabbitmq&spm=1018.2226.3001.4450)
- [RabbitMQ消息队列--2 -谷粒商城](https://blog.csdn.net/qq_43284469/article/details/120928040?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167677395816782429735695%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=167677395816782429735695&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-120928040-null-null.blog_rank_default&utm_term=rabbitmq&spm=1018.2226.3001.4450)

### 1.什么是MQ？

`Message Queue` 消息队列

> MQ是常见的消息中间件，一般java代码中的queue是存储在机器（电脑/服务器）的内存中，无法满足分布式项目的需求，因此需要可持久化的消息中间件。

### 2.MQ的常用场景/解决了说明问题？

- **异步处理**：提升了系统的异步处理能力，加快了响应速度。

  ![image-20220802193453652](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-05/20220805214332-kTAaA6GsPSMd6sdrtT7i.png)

- **应用解耦**：服务端和消费端解耦

  ![image-20220802194128931](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/29434203-cdba-46a6-89dc-5a1af5ac076d_image-20220802194128931.png)

- **流量控制**：流量削峰，防止瞬时的大流量使系统宕机

  ![image-20220802194343655](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/02061234-0f71-4682-82f7-e95ef6aaf9bf_image-20220802194343655.png)

### 3.MQ概述

1. 大多应用中，可通过消息服务中间件来提升系统异步通信,拓展解耦能力

2. 消息服务中的两个重要概念：`消息代理(message broker)`和`目的地（destination）`,当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。

3. 消息队列主要有两种形式的目的地：

   1. **队列（queue）**:点对点消息通信（point-to-point）
   2. **主题（topic）**:发布（pulish）/订阅（subcribe）消息通信

4. 点对点式：

   - 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移出队列
   - 消息只有唯一的发送者和接收者，但并不是说只能有一个接收者

5. 发布订阅式：

   - 发送者（发布者）发送消息到主题，多个接收者（监听者）监听（订阅）这个主题，那么就会在消息到达的同时收到消息

6. JMS（Java Message Service）JAVA消息服务：

   - 基于JVM消息代理的规范。ActiveMQ/HornetMQ是JMS实现的

7. AMQP（Advanced Message Queuing Protocol）

   - 高级信息队列协议，也是以一个消息代理的规范，兼容JMS

   - RabbitMQ是AMQP的实现

     ![image-20220803201145767](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/f7e62a1e-547d-4e98-82a0-75ff50718ae1_image-20220803201145767.png)

8. Spring支持

   - spring-jms提供了对JMS的支持
   - spring-rabbit提供了对AMQP的支持
   - 需要ConnectionFactory的实现来连接消息代理
   - 提供JmsTemplate/RabbitTemplate来发送消息
   - @JmsListener（JMS）/@RabbitListener(AMQP)注解来在方法上监听消息代理发布的消息
   - @EnableJms/@EnableRabbit注解开启支持

9. SpringBoot自动配置

   - JmsAutoConfiguration
   - RabbitAutoConfiguration

10. 市面上的MQ产品

    ActiveMQ/RabbitMQ/RocketMQ/Kafka

### 4.RabbitMQ概念

 **1.简介：**

RabbitMQ是使用erlang语言开发的AMQP的开源实现的消息队列中间件。

**2.核心概念：**

![image-20220803202446583](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/78e6db5c-cbfd-4dc7-81de-e075799586a2_image-20220803202446583.png)

![image-20220803204401753](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/698062d9-b4b3-4c0a-8634-ee5c000a0c0a_image-20220803204401753.png)

![image-20220803204413689](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/8611cf9e-ff0b-41a8-845f-b997aae062d3_image-20220803204413689.png)

![image-20220803204129027](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/c57b25ff-7db0-48ac-8558-529c62ac39b1_image-20220803204129027.png)



### 5.Docker安装RabbitMQ

> docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management
>
> docker update rabbitmq --restart=always

![image-20220804185529036](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/68065841-ba79-4d4e-8583-2e47ddfaacf3_image-20220804185529036.png)

> 访问管理页面 ip+15672

![image-20220804192035610](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/e53e035f-5ee3-42a5-8ba6-652e8bbe7ca4_image-20220804192035610.png)

### 6.RabbitMQ运行机制

- AMQP中的消息路由

![image-20220804192418053](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/2c9bae36-8e58-4864-8687-06946012ba79_image-20220804192418053.png)

- Exchange类型

  - **直接（direct）**

    ![image-20220804193108865](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/711dfd46-93de-45ce-8a69-43d282fe9bcf_image-20220804193108865.png)

  - **扇出（fanout）【广播模式】**![image-20220804193443849](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/772d6af7-0490-4a81-860e-efb24f70c55d_image-20220804193432908.png)

  - **主题（topic）【发布订阅模式】**

    ![image-20220804193601319](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/ad59357d-792d-4d15-82cc-bb4bcf5e1b38_image-20220804193601319.png)

​	       

### 7.SpringBoot整合RabbitMQ

1. 引入spring-boot-starter-amqp
2. yml配置
3. 测试MQ
   1. AmqpAdmin：管理组件
   2. RabbitTemplate：消息发送处理组件

### 8.RabbitMQ消息确认机制-可靠到达
- 保证消息不丢失，可靠到达们可以使用事务消息，性能下降250倍，为此引入确认机制
- publish confirmCallback 确认模式
- publish returnCallback 未投递到queue退回模式
- consumer ack机制
 ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-07/20220807123812-DrK8sdPxkFS56hhdWsN3.png)
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-07/9177023b-f6af-4fc0-a06f-cc18a506d9ad.png)
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-07/1853f267-ab43-4bc3-a044-846f5298163f.png)
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-07/82ad57de-e31a-4830-83d3-07961bc8b1ce.png)

### 9.RabbitMQ延时队列（实现定时任务）
**1.场景：**
比如未付款订单，超过一定时间后，系统自动取消订单并释放占有物品。

- **常用的解决方案：**
 spring的schedule定时轮询数据库
- **缺点：**消耗系统内存，增加了数据库的压力，存在较大的时间误差
- **更好的解方案：**rabbitmq的消息TTL和死信Exchange结合

**2.什么是TTL和死信（Exchange）**
- **消息的TTL（Time To Live）**
  ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-07/f7040ec5-cfa2-47bd-8620-cdad26cddf59.png)
- **Dead Letter Exchanges（DLX）死信Exchange**
  ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-07/0ce0025c-ada1-4d04-8857-e078dff32284.png)

**3.延时队列的设计思路**
- 实现1：给队列设置过期时间（推荐）
  ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-07/fe19a636-6eaf-4c76-8809-c2ad44efaf90.png)
- 实现2：给消息设置过期时间
  ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-07/cfc71b82-c0bc-49c1-baf5-8c10bd57c2ec.png)

