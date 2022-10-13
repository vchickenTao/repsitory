# ⛲ Redis - 单机Redis的安装

---

> [!NOTE]
>
> 在经过前面基础概念的学习，我们大致了解了Redis是什么，用来解决什么问题。为了方便后面的学习，我们需要先学习安装下单机版的Redis，至于集群版的Redis，例如，`主从复制`、`一主多从`的集群模式等，我们会在后面的学习中再阐述。Redis的官方建议在Linux环境下安装Redis，而且官网也只提供了基于Linux的安装包，但是在GitHub上提供Windows版本，接下来我们会从**Windows下安装Redis**、**Linux下安装Redis**以及**使用Docker安装Redis**讲解三种最常见的Redis安装方式。初学者不建议使用Docker安装，可以选择前两种方式安装，更有利于入门学习。@vchicken

<!-- tabs:start -->

## **Linux下安装Redis**

### 下载安装包

> 从[官网](https://redis.io/download/)下载指定版本的安装包,我这里下载的是当前最新的Redis7.0.5.

![下载安装包](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/01ccfc82-9de2-46fe-887d-483a889ed15f_download.png)

> 下载完我们会得到一个这样的安装包

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/458abba5-edf3-4481-8b2a-cdd8ece3beed_download-package.png)



### 上传文件到Linux服务器

> 我这里将.gz文件上传到usr/local/，创建一个redis文件夹，将文件存放到这里

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/bc9625ed-fa38-4820-8f77-a6e5f9f65e73_upload-package.png)



### 安装环境

> 要运行Redis，需要有C语言的编译环境，因此我们先检查自己的服务器是否已经安装了环境，输入gcc --version命令

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/c60af7c7-182d-4d3c-8c31-241d3666097e_检查gcc.png)

> 可以看到我这里已经有该环境了，其实很多云服务器出厂默认已经为我们安装这个基础环境，如果已经安装则忽略，如果没有安装则继续以下操作。
>
> **输入命令：yum intsall gcc ，gcc开始安装，接下来的[y/d/n]都输入y即可**

这里我们使用网络上的图来展示

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/fa7611d4-be60-4bd4-876a-d50bea408c82_安装gcc1.png)

安装成功

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/e17c8cce-9e4c-4f5e-8d17-9196b652e370_gcc安装成功.png)



### 解压安装包

> **输入命令：  tar zxvf redis-7.0.5.tar.gz -C ./**

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/90f669f0-de5b-465b-83f2-d5a1cef85db7_解压成功.png)

> 可以看到redis目录下，多了redis-7.0.5目录

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/48644ff7-72f4-4af6-837e-a388d5fa0c79_redis安装目录.png)

### 编译并安装

> **输入命令：**
>
> **cd redis-7.0.5/** 
>
> **make && make install**

安装成功！

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/cbe05514-aac0-4cef-8fd2-260ba2e86b52_编译安装成功.png)

在编译过程中，可能也会有人遇到下图的问题

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/744aa61f-06c5-4a72-89bb-ab4ab0f30713_gcc错误.png)

这是由于c语言编译环境没有安装好，会报错-Jemalloc/jemalloc.h: 没有那个文件

> 解决办法
>
> 1.输入命令：gcc --version 检查环境是否安装成功
>
> 2.输入命令：make distclean



### 安装目录文件介绍

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/8714f5a8-02a6-43a2-8e4b-cff1b820b072_Redis安装目录介绍.png)



### Redis启动

#### 前台启动（不推荐）

> 进入usr/local/sbin目录，执行redis-server命令

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/d1084b56-a4de-498d-8b22-90e6eb156d85_Redis前台启动.png)

这样启动的弊端就是这个命令窗口就不能再做其他操作了，如果命令行窗口关闭，则Redis服务停止。

#### 后台启动（推荐）

> 按Ctrl+C停掉前台启动的Redis

- 进入 /usr/local/redis/redis-7.0.5目录

- 复制redis.conf配置文件，输入命令 cp redis.conf /etc/redis.conf

- 进入etc目录下，修改redis.conf配置文件，将daemonize no 改为yes

  > vi redis.conf
  >
  > 英文状态下输入：i
  >
  > 修改daemonize no 为daemonize yes
  >
  > 按ESC，输入`: wq!` 保存并退出

- 启动redis，进入/usr/local/bin，输入命令：redis-server /etc/redis.conf

- 查看启动状态， 输入命令：ps -ef | grep redis

  ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/dd0689a1-f935-4dbd-86cc-a56befdaa809_后台启动redis.png)

- 关闭redis，可以输入命令：shutdown或者直接将上图中pid为5786的进程杀掉：kill -9 5786



### 使用redis-cli连接

> 进入usr/local/sbin目录，输入命令： redis-cli



### 修改Redis访问密码

> 默认redis没有密码，这样明显不安全，我们可以手动为它设置访问密码

- vi /etc/redis.conf
- 找到#requirepass foobared去掉注释，foobared改为自己的密码，我在这里改为requirepass 123456
- 关闭原先的服务并重启服务 redis-server /etc/redis.conf
- 再次使用redis-cli访问，输入命令：redis-cli -h 127.0.0.1 -p 6379 -a 123456



### 设置允许远程连接

> 以上步骤我们已经成功的在服务器上安装了Redis，那么未来我们的项目需要集成Redis并远程连接Redis，现在我们通过RedisDesktopManager可视化工具来尝试远程连接Redis。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/58ff9735-00f0-4a0a-8e92-716a8b1499c8_远程连接失败.png)

> 事实上我们发现并不能远程连接Redis，这是由于Redis的conf配置文件默认设置127.0.0.1才可以访问，这就意味着只能本机访问，因此我们需要修改配置文件，让Redis支持远程连接。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/b3f8fd21-57a1-40eb-8254-07b4ae8a98bb_默认本机访问.png)

- 进入/etc目录,输入命令L: vi redis.conf,找到bind 127.0.0.1,将该行注释掉

  ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/58efed6d-ea28-4a80-8efe-f00943066450_远程连接.png)

- 保存退出

- 重启Redis服务，redis-server /etc/redis.conf

  ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/778f46b9-1b17-4e4b-85cf-35962540fecd_注释Redis安全组.png)

- 修改成功

> 现在我们再重新远程连接Redis试试，OK，完美！

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-27/08e31974-8a57-41d7-8cbd-7317ed2eb669_远程连接成功.png)



## **Windows下安装Redis**

> [!TIP]
>
> To be continue~



## **使用Docker安装Redis**

> [!TIP]
>
> To be continue~

<!-- tabs:end -->