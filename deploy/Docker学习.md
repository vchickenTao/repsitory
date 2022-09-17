# Docker学习

## 1.Docker为什么出现？

一款产品的上线，环境的配置十分繁琐，每一台服务器都要部署集群等，费时费力！

 Java  --- jar（环境）---- 打包项目带上环境（镜像） ---- Docker商店 ---- 运维下载镜像  ---- 发布即可！

Docker针对以上问题，提出了解决方案！

Docker的思想就是来自于集装箱！

**隔离：**Docker核心思想！打包装箱！每个箱子都是隔离的!



## 2.Docker 的历史

2010年，几个搞IT的年轻人，在美国成立了一家公司`dotCloud` ,做一些pass的云计算服务！LXC 有关的容器化技术！

他们将自己的技术（容器化技术）命名就是Docker。

Docker刚成立的时候，没有引起行业的注意，无法生存！因此他们选择`开源`！

2013年 Docker开源！

2014年4月9日，Docker 1.0发布。

Docker火的原因，非常的轻巧。

在容器化技术出现前，我们使用的都是虚拟化技术。

**虚拟机**: 在window中装一个Vmware，我们可以虚拟出来一台或者多台电脑，非常笨重！

**容器技术**：Docker也是一种虚拟化技术

```shell
vm: linux 原生镜像（一台电脑） 隔离，需要开启多个虚拟机！
docker： 隔离，镜像（最核心和环境 4m + jdk + mysql）非常小巧，运行镜像即可！
```

> 聊一聊Docker

Docker是基于Go语言开发的开源项目！



## 3.Docker能做什么？

> 虚拟机技术

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/73eb9385-fd7b-4f16-8f3c-b6842b93ffab_123456789.jpg)

缺点：

1.资源占用很大

2.冗余步骤很多

3.启动很慢

> 容器化技术

![image-20220724125538270](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/b8a2cf4a-2a0f-47cc-8203-8974dc0a2d30_image-20220724125538270.png)

比较Docker和虚拟化技术的不同之处：

- 传统虚拟机，虚拟出一条硬件，运行一个完成的操作系统，然后在这个系统上安装和运行软件！
- 容器内的应用直接运行在宿主机的内容，容器是没有自己的内核的，也没有虚拟我们的硬件，这样就轻便了
- 每个容器间是互相隔离的，每个容器内都有一个属于自己的文件系统，互不影响。

> DevOps(开发/运维)

**应用更快的交付和部署**

传统：一堆帮助文件，安装程序

Docker：打包镜像发布测试，一键运行

**更便捷的升级和扩容**

使用Docker后，我们部署应用就跟搭积木一样

项目打包为一个镜像，拓展 服务器A！服务器B！......

**更简单的系统运维**

在容器化后，我们的开发，测试环境都是高度一致的。

**更高效的计算资源利用**

Docker是内核级别的虚拟化，可以在一个物理机上运行很多的容器实例！服务器的性能可以发挥到极致！



## 4.Docker安装

### Docker的基本组成

![image-20220724130755162](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/f50a0cfb-087d-4e86-8d94-583423ca9b6c_image-20220724130755162.png)

**镜像（images）：**

docker镜像就好比是一个模板，可以通过这个模板来参加容器服务，tomcat镜像===>run===>tomcat01容器（提供服务器），

通过这个镜像可以创建多个容器（最终服务运行或者项目运行就是在容器中的）

**容器（container）：**

Docker利用容器技术，独立运行一个或者一组应用，通过镜像来创建的，

启动/通知/删除，基本命令！

目前可以把这个容器简单的理解为就是一个简易的linux系统。

**仓库（repository）：**

仓库就是放镜像的地方！

仓库分为一个公有仓库和私有仓库！

Docker Hub（默认是国外的）/阿里云



### 安装Docker

> 环境准备

1.linux基础

2.CentOS 7

3.使用工具连接服务器

> 环境查看

```shell
#系统环境是3.10以上的
# uname -r
3.10.0-1160.62.1.el7.x86_64
```

```shell
#系统版本
[root@instance-ruybnuk8 ~]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

> 安装

官方文档：

```shell
 #1. 卸载旧版本
 yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
                  
 #2.安装需要的安装包
 yum install -y yum-utils

 #3.设置镜像的仓库
 yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo #默认是国外的
 
 #使用阿里云
 yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
 
 #更新yum的索引
 yum makecache fast
 
 #4.安装docker相关的内容 cocker-ce 社区版 ee企业版
 yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
 
 #5.启动 docker
 systemctl start docker
 
 #6. docker version 查看是否安装成功
 
Client: Docker Engine - Community
 Version:           20.10.17
 API version:       1.41
 Go version:        go1.17.11
 Git commit:        100c701
 Built:             Mon Jun  6 23:05:12 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.17
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.17.11
  Git commit:       a89b842
  Built:            Mon Jun  6 23:03:33 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.6
  GitCommit:        10c12954828e7c7c9b6e0ea9b0c02b01407d3ae1
 runc:
  Version:          1.1.2
  GitCommit:        v1.1.2-0-ga916309
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
  
 #7. docker run hello-world 尝试运行docker
 [root@instance-ruybnuk8 ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally 
latest: Pulling from library/hello-world #远程拉取镜像
2db29710123e: Pull complete 
Digest: sha256:53f1bbee2f52c39e41682ee1d388285290c5c8a76cc92b42687eecf38e0af3f0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 
 # 8. 查看一下下载的这个 hello-world镜像
  docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   10 months ago   13.3kB
```

> 卸载docker

```shell
#1. 卸载依赖
yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin

#2. 删除资源
rm -rf /var/lib/docker #docker的默认工作路径
rm -rf /var/lib/containerd
```

### 阿里云镜像加速（仅限阿里云服务器）

1.登录阿里云找到容器服务

2.镜像加速器（最新已收费）

![image-20220724135343348](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/da5b2001-6c34-4f0c-80ab-b56053376e88_image-20220724135343348.png)

3.配置使用



### 回顾HelloWold流程

![image-20220724135749332](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/062cf5f0-1a59-4d33-8fee-2761117bac12_image-20220724135749332.png)

 ![image-20220724140830257](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/04ef19f7-94e2-46ac-8c46-4ca274427731_image-20220724140830257.png)



### 底层原理

**Docker是怎么工作的？**

Docker是一个Client + server 结构的系统，Docker的守护进程运行在主机上。通过Socket从客户端访问！

DockerServer接收到Docker-Client的指令，就会执行这个命令！

![image-20220724141511326](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/bc0bb240-a6df-4ea1-877a-9ddd80836fd2_image-20220724141511326.png)

**Docker为什么比虚拟机快？**

1. Docker有比虚拟机更少抽象层；

2. docker利用的是宿主机的内核，vm需要的是Guest OS

   ![image-20220724141709900](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/a48c533a-960a-408b-846b-7c0d07fb1cc7_image-20220724141709900.png)

所以，新建一个容器的时候，docker不需要像虚拟机一样重新加载一个操作系统内核，避免引导。虚拟机是加载 Guest OS，

分钟级别的，而docker是利用宿主机的操作系统，省略了复杂的过程，从而实现了秒级。

![image-20220724142312045](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-06/fc8f3b6e-b5b9-4f1b-81bc-b637a3fcb7ab_image-20220724142312045.png)



## 5.Docker的常用命令

### 帮助命令：

```shell
docker version  #显示docker的版本信息
docker info     #显示docker的系统信息，包括镜像和容器的数量
docker --help   #万能命令

```

### 镜像命令

**docker images 查看所有本地的主机上的镜像**

```shell
[root@vchicken ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   10 months ago   13.3kB

#解释
REPOSITORY 镜像的仓库源
TAG        镜像的标签
IMAGE ID   镜像的id
CREATED    镜像的创建时间
SIZE       镜像的大小

#可选项
  -a, --all             Show all images (default hides intermediate images) 列出所有镜像
      --digests         Show digests 
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs  只显示镜像的ID
```

**docker search 搜索镜像**

```shell
[root@vchicken ~]# docker search mysql
NAME                           DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                          MySQL is a widely used, open-source relation…   12956     [OK]       
mariadb                        MariaDB Server is a high performing open sou…   4963      [OK]   

# 可选项，通过搜索来过滤
-f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
 
[root@vchicken ~]# docker search mysql --filter=STARS=5000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   12956     [OK]   
```

**docker pull 下载镜像**

```shell
# 下载镜像 docker pull 镜像名[:tag]版本
[root@vchicken ~]# docker pull mysql
Using default tag: latest #如果不写tag，默认就是 latest
latest: Pulling from library/mysql
e54b73e95ef3: Pull complete  # 分层下载，docker image的核心 联合文件系统
327840d38cb2: Pull complete 
642077275f5f: Pull complete 
e077469d560d: Pull complete 
cbf214d981a6: Pull complete 
f4fda5f8b9a8: Pull complete 
a41c2763043b: Pull complete 
f86b3df6abb1: Pull complete 
95b1c2ed2ecf: Pull complete 
b0edcd52771b: Pull complete 
a3d312b5c835: Pull complete 
Digest: sha256:657d78ee56e09101902673adcdd7d2bf03012e759c1aa525eeca28cb0fe1aa7d  #签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest  #真实地址

# 等价于
docker pull mysql
docker pull 

# 指定版本下载
```

**docker rmi 删除镜像**

```shell
[root@vchicken ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
mysql         latest    38643ad93215   5 days ago      446MB
hello-world   latest    feb5d9fea6a5   10 months ago   13.3kB
[root@vchicken ~]# docker rmi -f 38643ad93215[容器id] #删除指定的镜像
Untagged: mysql:latest
Untagged: mysql@sha256:657d78ee56e09101902673adcdd7d2bf03012e759c1aa525eeca28cb0fe1aa7d
Deleted: sha256:38643ad93215bedea00fedd3d6f2a1c8e1bff3b9a172aa2547fd8b4bac9cfee3
Deleted: sha256:47e742a69a0555678b9e812fa9ecb57da6f0edeff769f9deaa565d4f1a206eaf
Deleted: sha256:451dabedc195bcb12548ec2097c6c37d79763bb9fb9c5d0ab611988858247c38
Deleted: sha256:d6518568924a06ced7708dc381c4ea25d38c31bbc5a2ce7827a9e8432460bf25
Deleted: sha256:65e56d8d80e42df6836a6ab9d3bc93b9a8c1e1ea92592c3447e828ac2e23e5e9
Deleted: sha256:3eab7fb11398a95c1f8dc960b5d8afac7ca6b5d66a831321683de7670fe28ab6
Deleted: sha256:4b9af9119abc61e3fc814ee35dcfa678bfab88e3d7880b1a01c66066c43afae3
Deleted: sha256:ce92b5bb86bd1d23ba0db641ea1a4009a76c59b6f1fb69e917c904407fda060c
Deleted: sha256:5b4e33f9109ff778969630f905ff31336c187cace319b73a3f1106bce02dcad9
Deleted: sha256:ef40bcf2fa76e9055e74eba2580488b98e17d795c96b96670f56b238b685a14d
Deleted: sha256:c294df63d04d9c99ad34fd2279f9c2b021486088d33f4d5de335c6712657bec9
Deleted: sha256:c6c89a36c214d7ecf7a684bf0fc21692dd60e9f204f48545bcb4085185166031

[root@vchicken ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   10 months ago   13.3kB

[root@vchicken ~]# docker rmi -f 容器id 容器id 容器id  #删除多个镜像
[root@vchicken ~]# docker rmi -f $(docker images -aq)  #删除全部的镜像
```

### 容器命令

**说明：我们有了镜像才可以创建容器，Linux，下载一个centos镜像来学习** 

```shell
docker pull centos
```

**新建容器并启动**

```shell
docker run [可选参数] image

# 参数说明
--name="Name"  容器名字 tomcat01....原来区分容器
-d             后台方式运行
-it            使用交互方式运行，进入容器查看内容
-P             指定容器的端口 -P 8080:8080
	-P ip:主机端口:容器端口
	-P 主机端口:容器端口（常用）
	-P 容器端口
	容器端口
-p             指定随机的端口

# 测试，启动并进入容器
[root@vchicken ~]# docker images     
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   10 months ago   13.3kB
centos        latest    5d0da3dc9764   10 months ago   231MB
[root@vchicken ~]# docker run -it centos /bin/bash
[root@b12872e9a1a7 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@b12872e9a1a7 /]# 

```

**列出所有运行的容器**

```shell
# docker ps 命令
    # 列出当前正在运行的容器
 -a # 列出当前正在运行的容器+历史运行过的容器
 -n=? #显示最近创建的容器  docker ps -a -n=1
 -q # 只显示容器的编号
[root@vchicken ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@vchicken ~]# docker ps -a
CONTAINER ID   IMAGE         COMMAND       CREATED         STATUS                        PORTS     NAMES
b12872e9a1a7   centos        "/bin/bash"   4 minutes ago   Exited (127) 11 seconds ago             modest_feistel
a76ea4ab94fd   hello-world   "/hello"      8 days ago      Exited (0) 8 days ago                   eloquent_goldberg
```

**推出容器**

```shell
# exit 容器停止并退出
# CTRL + P + Q 容器不停止退出
```

**删除容器**

```shell
docker rm 容器id #删除指定容器，不能删除正在运行的容器，强制删除 rm -f
docker rm -f $(docker ps -aq) # 删除全部容器
docker ps -a -q|xargs docker rm #删除全部容器
```

**启动和停止容器的操作**

```shell
docker start 容器id    #启动容器
docker restart 容器id #重启容器
docker stop 容器id    #停止容器
docker kill 容器id    #强制停止容器
```

