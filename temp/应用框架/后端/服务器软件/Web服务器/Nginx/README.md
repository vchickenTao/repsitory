# Nginx

***Nginx*** 是高性能的 ***HTTP*** 和反向代理的服务器，处理高并发能力是十分强大的，能经受高负载的考验，有报告表明能支持高达 50,000 个并发连接数。



## 反向代理

**正向代理**

***Nginx*** 不仅可以做反向代理，实现负载均衡。还能用作正向代理来进行上网等功能。 正向代理：如果把局域网外的 ***Internet*** 想象成一个巨大的资源库，则局域网中的客户端要访问 ***Internet***，则需要通过代理服务器来访问，这种代理服务就称为正向代理。

<u>需要在客户端配置代理服务器进行指定网站访问</u>。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2016.29.54.png" style="zoom:50%;" />

**反向代理**

反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，<u>暴露的是代理服务器地址，隐藏了真实服务器 ***IP*** 地址</u>。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2016.35.09.png" style="zoom:50%;" />



## 负载均衡

**单一模式**

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2016.40.11.png" style="zoom:50%;" />

**负载均衡**

增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，即负载均衡。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2016.45.55.png" style="zoom:50%;" />

## 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2016.49.18.png" style="zoom:50%;" />

# 安装和启动

官网：http://nginx.org/



> 1. 需要安装 ***gcc-c++*** 编译器
>
> ```
> yum install gcc-c++
> yum install -y openssl openssl-devel
> ```
>
> 2. 安装 ***pcre*** 包
>
> ```
> yum install -y pcre pcre-devel
> ```
>
> 3. 安装 ***zlib*** 包
>
> ```
> yum install -y zlib zlib-devel
> ```
>
> 4. 在 ***/opt/***  下载 ***nginx***
>
> ```bash
> wget https://nginx.org/download/nginx-1.19.9.tar.gz
> ```
>
> 5. 解压并进入 ***nginx*** 目录
>
> ```bash
> tar -zxvf nginx-1.19.9.tar.gz
> cd nginx-1.19.9
> ```
>
> 6. 使用 ***nginx*** 默认配置
>
> ```bash
> ./configure
> ```
>
> 7. 编译安装
>
> ```bash
> make && make install
> ```
>
> 
>
> 默认安装路径：`/usr/local/nginx`
>
> 
>
> 启动：`cd /usr/local/nginx/sbin`
>
> 默认启动脚本：`./nginx`
>
> <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2017.02.59.png" style="zoom:50%;" />
>
> 
>
> 成功启动后可以进行访问：`172.16.88.168:80`（需要关闭防火墙）
>
> 默认端口：80

 	

## 常用命令

```bash
cd /usr/local/nginx/sbin # 默认安装路径

./nginx -v # 查看 nginx 版本号 

./nginx # 启动 nginx

./nginx -s stop # 停止 nginx 

./nginx -s reload # 重新加载 nginx 
```






# 配置

## 配置文件

***nginx.conf***

默认路径：***/usr/local/nginx/conf/nginx.conf***



> ***全局块***
>
> 主要包括配置运行 ***Nginx*** 服务器的用户（组）、允许生成的 ***worker process*** 数，进程 ***PID*** 存放路径、日志存放路径和类型以及配置文件的引入等。
>
> <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2021.47.03.png" style="zoom:50%;" />
>
> 
>
> ***events块***
>
> ***events*** 块涉及的指令主要影响 ***Nginx*** 服务器与用户的网络连接，常用的设置包括是否开启对多 ***work process*** 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 ***word process*** 可以同时支持的最大连接数等。
>
> <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2021.48.11.png" style="zoom:50%;" />
>
> 
>
> ***http块***
>
> 这算是 ***Nginx*** 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。 需要注意的是：***http*** 块也可以包括 ***http*** 全局块、***server*** 块。
>
> 1. ***http 全局块***
>
>    ***http*** 全局块配置的指令包括文件引入、***MIME-TYPE*** 定义、日志自定义、连接超时时间、单链接请求数上限等。
>
> 2. ***server块***
>
>    这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了 节省互联网服务器硬件成本。
>
>    每个 ***http*** 块可以包括多个 ***server*** 块，而每个 ***server*** 块就相当于一个虚拟主机。 而每个 ***server*** 块也分为全局 ***server*** 块，以及可以同时包含多个 ***locaton*** 块。
>
>    1. ***全局 server 块***
>
>       最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。
>
>    2. ***location 块***
>
>       一个 ***server*** 块可以配置多个 ***location*** 块。
>
>       这块的主要作用是基于 ***Nginx*** 服务器接收到的请求字符串（例如 ***server_name/uri-string***），对虚拟主机名称（也可以是 ***IP*** 别名）之外的字符串（例如 前面的 ***/uri-string***）进行匹配，对特定的请求进行处理。地址定向、数据缓 存和应答控制等功能，还有许多第三方模块的配置也在这里进行。
>
> <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2021.48.47.png" style="zoom:50%;" />



## 反向代理

#### 案例 1

```
实现效果：使用 nginx 进行反向代理，访问 www.test.com 直接跳转到 127.0.0.1:8080
```

访问的流程：

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2023.55.12.png" alt="" style="zoom:50%;" />

如何启动 ***tomcat***

```
// tomcat安装路径bin目录
// 默认端口8080
./startup.sh
```



1. 本地 ***hosts*** 文件进行域名和 ***ip*** 对应配置

```
# 最后一行加上
# 虚拟机对应的ip
172.16.88.168 www.test.com
```

2. 在 ***nginx*** 进行请求转发的配置（反向代理配置）

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2023.54.20.png" alt="" style="zoom:50%;" />

3. 启动 ***nginx***

```
// 默认路径 /usr/local/nginx/sbin/
./nginx -c /opt/etc/nginx.conf # 配置文件的路径
```



最终：访问 `wwww.test.com` 跳转到 ***tomcat*** 成功启动页面。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-22%2023.56.21.png" style="zoom:50%;" />



---

#### 案例 2

```
实现效果：
使用 nginx 反向代理，根据访问的路径跳转到不同端口的服务中 nginx 监听端口为 9001
访问 http://172.16.88.168:9001/edu/ 直接跳转到 127.0.0.1:8080
访问 http://172.16.88.168:9001/vod/ 直接跳转到 127.0.0.1:8081
```

1. 准备两个 ***tomcat*** 服务器（略）

2. 配置 ***nginx.conf***

```
   server {
        listen       9001;
        server_name  172.16.88.168;

        location ~ /edu/ {
            proxy_pass http://127.0.0.1:8080;
        }
        location ~ /vod/ {
            proxy_pass http://127.0.0.1:8081;
        }
 				...
 		}
```



### *proxy_pass*

> 格式： ***proxy_pass URL***
>
> URL包含：传输协议、主机名、***uri***



***url*** 的 / 问题

> 在 ***nginx*** 中配置 ***proxy_pass*** 时，当在后面的 ***url*** 加上了/，相当于是绝对根路径，则 ***nginx*** 不会把 ***location*** 中匹配的路径部分代理；
>
> 如果没有 /，则会把匹配的路径部分也给代理。

```bash
假设 server_name:192.168.1.10
		请求:http://192.168.1.10/a/a.html

示例1：
location /a/
{
    proxy_pass http://192.168.1.10;
    ...
}结果1：http://192.168.1.10/a/a.html

示例2：
location /a/
{
    proxy_pass http://192.168.1.10/;
    ...
}结果2：http://192.168.1.10/a.html

示例3：
location /a/
{
    proxy_pass http://192.168.1.10/nanase;
    ...
}
结果3：http://192.168.1.10/nanasea.html

示例4：
location /a/
{
    proxy_pass http://192.168.1.10/nanase/;
    ...
}
结果4：http://192.168.1.10/nanase/a.html
```



**<u>为了方便记忆和规范配置，建议所有的 *proxy_pass* 后的 *url* 都以 / 结尾。</u>**



### *location*

该指令用于匹配 ***URL***。

```
location [ = | ~ | ~* | ^~ ] uri {

}
```

> `=` 
>
> 用于不含正则表达式的 ***uri*** 前，要求请求字符串与 ***uri*** 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。
>
> `~`
>
> 用于表示 ***uri*** 包含正则表达式，并且区分大小写。
>
> `~*`
>
> 用于表示 ***uri*** 包含正则表达式，并且不区分大小写。
>
> `^~`
>
> 用于不含正则表达式的 ***uri*** ，要求 ***Nginx*** 服务器找到标识 ***uri*** 和请求 符串匹配度最高的 ***location*** 后，立即使用此 ***location*** 处理请求，而不再使用 ***location*** 块中的正则 ***uri*** 和请求字符串做匹配。



**<u>*注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~\* 标识。*</u>**		 		

​			 					 					 				

## 负载均衡

增加服务器的数量，然后将请求分发到各个服务器上面。即将原先请求集中到单个服务器上转化为将请求分发到多个服务器上，将负载分发到不同的服务器，即负载均衡。



#### 案例 1

```
实现效果：浏览器地址栏输入地址 http://172.16.88.168/edu/a.html，负载均衡效果，平均分担 8080 和 8081 端口中。
```

1. 准备两个 ***tomcat*** 服务器（略）
2. 配置 ***nginx.conf*** 文件

```
http {
	...
	upsteam myserver {
		ip_hash;
		server 172.16.88.168:8080 weight=1;
		server 172.16.88.168:8081 weight=1;
	}
	server {
        listen       80;
        server_name  172.16.88.168;

        location / {
            proxy_pass http://myserver;
            root html;
            index index.html index.htm;
        }
 				...
 		}
}
```



### 分配策略

- ***轮询***（默认）
   每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 ***down*** 掉，能自动剔除。

- ***weight***
   ***weight*** 代表权重默认为 1，权重越高被分配的客户端越多。

  ```
  upstream server_pool {
  	server 192.168.5.21 weight=10; 
  	server 192.168.5.22 weight=10; 
  }
  ```

- ***ip_hash***
   每个请求按访问 ***ip*** 的 ***hash*** 结果分配，这样每个访客固定访问一个后端服务器。

  ```
  upstream server_pool {
  	ip_hash;
  	server 192.168.5.21:80; 
  	server 192.168.5.22:80; 
  }
  ```

- ***fair***（第三方） 按后端服务器的响应时间来分配请求，响应时间短的优先分配。

  ```
  upstream server_pool { 
  	server 192.168.5.21:80; 
  	server 192.168.5.22:80; 
  	fair;
  }
  ```

  

## 动静分离

***Nginx*** 动静分离简单来说就是把动态跟静态请求分开，不能理解成只是单纯的把动态页面和 静态页面物理分离。

一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案。

另外一种方法就是动态跟静态文件混合在一起发布，通过 ***nginx*** 来分开。



<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-27%2014.18.44.png" style="zoom:50%;" />



#### 案例 1

1. 在 ***linux*** 系统中准备静态资源，用于进行访问
2. 配置 ***nginx*** 文件

```
   server {
        listen       80;
        server_name  172.16.88.168;

        location /html/ {
        	root /data/;
        	index index.html index.htm;
        	# 访问 /data/html/.*html
        }
        location /image/ {
        	root /data/;
        	autoindex on; # 列出文件目录
        }
 				...
 		}
```



## 高可用集群

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-27%2014.21.39.png" alt="截屏2021-10-27 14.21.39" style="zoom:50%;" />

即主服务器宕机，启动备用服务器。



#### 案例 1

***keepalive + nginx***

**高可用集群（主从模式）**

1. 准备两台服务器

   ```
   服务器ip：
   	192.168.17.128
   	192.168.17.131
   ```

2. 两台服务器安装 ***keepalive*** 和 ***nginx*** （略）

   ```
   yum install keepalive-y
   ```

3. 完成高可用配置（主从模式）

   在 `/usr/local/src/` 添加检测脚本

   ```bash
   
   #!/bin/bash
   A=`ps -C nginx –no-header |wc -l` if [ $A -eq 0 ];then
   /usr/local/nginx/sbin/nginx
   sleep 2
   if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
           killall keepalived
       fi
   fi
   ```

   配置主服务器

   ```bash
   # 主要修改 keepalive 配置文件
   # /etc/keepalived/keepalived.conf
   # 主和备服务器修改不同地方
   
   # 全局配置
   global_defs {
   	notification_email { 
   		acassen@firewall.loc 
   		failover@firewall.loc 
   		sysadmin@firewall.loc
   }
   	notification_email_from 
   	Alexandre.Cassen@firewall.loc smtp_server 192.168.17.129
   	smtp_connect_timeout 30
   	router_id LVS_DEVEL # 访问主机服务器名字
   }
   
   # 脚本配置
   vrrp_script chk_http_port { 
   	script "/usr/local/src/nginx_check.sh" # 脚本
   	interval 2 # 检测脚本执行的间隔
   	weight 2 # 设置当前服务器的权重
   }
   
   # 虚拟ip配置
   vrrp_instance VI_1 { 
   	state MASTER # 主服务器：MASTER 备份服务器：BACKUP 
   	interface ens33 # 网卡
   	virtual_router_id 51 # 主备机的id必须相同
   	priority 100 # 优先级；主机值较大，备份机值较小 
   	advert_int 1 # 时间间隔检测
   	
   	# 权限
   	authentication { 
   		auth_type PASS
   		auth_pass 1111 
   	}
   	
   	# 虚拟ip
   	virtual_ipaddress {
   		192.168.17.50 # VRRP H 虚拟地址
   	}
   }
   ```

4. 启动 ***nginx*** 和 ***keepalived*** 服务



# 原理

## master&worker

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-27%2014.49.04.png" style="zoom:50%;" />



## worker如何工作

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-27%2014.59.12.png" style="zoom:50%;" />



## 一个master多个worker

- 可以使用 ***nginx –s reload*** 热部署，利用 ***nginx*** 进行热部署操作；
- 每个 ***woker*** 是独立的进程，如果有其中的一个 ***woker*** 出现问题，其他 ***woker*** 独立的， 继续进行争抢，实现请求过程，不会造成服务中断。



## worker的数量

***nginx*** 类似 ***redis*** 都采用 ***io*** 多路复用机制，每个 ***worker*** 都是一个独立的进程。

即 ***worker*** 数量和服务器的 ***cpu*** 数量相等最合适。



## 连接数worker_connection

```
第一个：发送请求，占用了 woker 的几个连接数? 

答案: 2 或者 4 
```

```
第二个：nginx 有一个 master，有 4 个 woker，每个 woker 支持最大的连接数 1024，支持的最大并发数是多少?

worker支持的最大连接数：4 * 1024

普通的静态访问最大并发数是：
	worker_connections * worker_processes / 2
	1024 * 4 / 2

而如果是 HTTP 作 为反向代理来说，最大并发数量应该是：
	worker_connections * worker_processes / 4
	1024 * 4 / 4
```



# 感谢

- [尚硅谷-王老师](https://www.bilibili.com/video/BV1zJ411w7SV?from=search&seid=5693495616515059218&spm_id_from=333.337.0.0)