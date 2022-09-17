# Linux命令整理

## 1.简单系统命令

```shell
# 查看ip地址
	ip a
	ip addr
# ping网络(测试网络连通)
	ip 目标机器的ip
# 查看系统时间
	date
# 注销
	logout
# 关机
	shutdown now
# 重启
	reboot
# 清屏
	clear
	ctrl + L
```

## 2.Linux文件系统

- **核心**

  > 1.Linux一切皆文件
  >
  > 2.只有一个顶级目录，不像windows分C盘、D盘、E盘

- **目录结构**
  ![img](https://img-blog.csdnimg.cn/6de096f199db4780b03af8757826cf18.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA56ys5LqM6IyD5byP,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

- 文件含义

  ![image-20220727100543730](C:\Users\97762\AppData\Roaming\Typora\typora-user-images\image-20220727100543730.png)

## 3. 文件管理命令

> 注意事项：命令区分大小写

```shell
# 1. 查看文件列表
	ls [-参数1参数2] [目标文件夹]

#  举栗子：
 # 查看当前目录下的文件列表
	ls
 # 查看指定目录下的文件
	ls /
 	[root@instance-ruybnuk8 ~]# ls /
 	bin  boot  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
 # 查看详细信息，元数据信息(用户、组、大小、创建时间、权限信息、文件类型)
	ls -l
	[root@instance-ruybnuk8 ~]# ls -l
	total 0
 # 查看隐藏文件
	ls -a 
	[root@instance-ruybnuk8 ~]# ls -a
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .cache  .config  .cshrc  .pki  .ssh  .tcshrc
 # 参数并用
	ls -la
	
	[root@instance-ruybnuk8 ~]# ls -la
    total 48
    dr-xr-x---.  6 root root 4096 Jul 24 13:37 .
    dr-xr-xr-x. 19 root root 4096 Jul 24 18:15 ..
    -rw-------   1 root root 2840 Jul 27 10:14 .bash_history
    -rw-r--r--.  1 root root   18 Dec 29  2013 .bash_logout
    -rw-r--r--.  1 root root  176 Dec 29  2013 .bash_profile
    -rw-r--r--.  1 root root  176 Dec 29  2013 .bashrc
    drwxr-xr-x   3 root root 4096 Jul 24 12:06 .cache
    drwxr-xr-x   3 root root 4096 Jul 24 12:06 .config
    -rw-r--r--.  1 root root  100 Dec 29  2013 .cshrc
    drwxr-----   3 root root 4096 Jul 24 13:37 .pki
    drwx------   2 root root 4096 Jul 24 11:49 .ssh
    -rw-r--r--.  1 root root  129 Dec 29  2013 .tcshrc	
    
    ls / -l
    
    [root@instance-ruybnuk8 ~]# ls / -l
    total 64
    lrwxrwxrwx.   1 root root     7 Jan 26  2021 bin -> usr/bin
    dr-xr-xr-x.   5 root root  4096 May  7 11:13 boot
    drwxr-xr-x   20 root root  3140 Jul 24 11:53 dev
    drwxr-xr-x.  87 root root  4096 Jul 24 18:23 etc
    drwxr-xr-x.   2 root root  4096 Apr 11  2018 home
    lrwxrwxrwx.   1 root root     7 Jan 26  2021 lib -> usr/lib
    lrwxrwxrwx.   1 root root     9 Jan 26  2021 lib64 -> usr/lib64
    drwx------.   2 root root 16384 Jan 26  2021 lost+found
    drwxr-xr-x.   2 root root  4096 Apr 11  2018 media
    drwxr-xr-x.   2 root root  4096 Apr 11  2018 mnt
    drwxr-xr-x.   8 root root  4096 Jul 24 13:41 opt
    dr-xr-xr-x  156 root root     0 Jul 24 11:53 proc
    dr-xr-x---.   6 root root  4096 Jul 24 13:37 root
    drwxr-xr-x   29 root root   900 Jul 24 18:23 run
    lrwxrwxrwx.   1 root root     8 Jan 26  2021 sbin -> usr/sbin
    drwxr-xr-x.   2 root root  4096 Apr 11  2018 srv
    dr-xr-xr-x   13 root root     0 Jul 27 10:08 sys
    drwxrwxrwt.   9 root root  4096 Jul 27 09:20 tmp
    drwxr-xr-x.  13 root root  4096 Jan 26  2021 usr
    drwxr-xr-x.  20 root root  4096 Jan 26  2021 var
```

```shell
# 2. 切换目录
	cd 目标文件夹
```

```shell
# 3. 查看当前命令所在的目录

# 举栗子：
 # 查看当前目录路径
     pwd
 # 特殊目录符号
	~ 当前用户的home目录
	. 当前目录
	.. 上一级目录
```

```shell
# 4. 新建文件夹及文件

# 举栗子：
 # 在当前位置新建文件夹
	mkdir 文件夹名
 # 在指定目录位置，创建文件夹，并创建父文件夹
	mkdir -p /a/b/文件夹名
 # 在当前目录下新建文件
	touch 文件名

```

```shell
# 5. 删除文件

# 删除文件
	rm 文件
# 删除文件夹
	rm -r 文件夹
# 强制删除不询问
	rm -rf 文件
```

```shell
# 6. 拷贝文件

# 拷贝文件
	cp 原文件  新文件
# 拷贝文件夹
	cp -r 源文件夹 新文件夹
```

```shell
# 7. 移动文件或修改文件名

# 移动源文件到目标文件夹中
	mv 文件  文件夹
# 修改文件A的名字为文件B
	mv 文件A 文件B
```

```shell
# 8. 获取文件的md5指纹(数字签名)

md5sum 文件名
# 简介
1. 数字签名，又称数字指纹
2. 可以验证文件是否被修改
3. 一个文件通过计算得到的一串字符串,文件内容的唯一标记(文件内容不变,指纹不会变)
```

## 4. 文本内容查看命令

```shell
# cat命令

# 查看文件中的全部信息(适合查看小文档)
	cat 文件名

# less命令
 # 以分页的方式浏览文件信息(适合查看大文档)，进入浏览模式
	less 文件名
 # 浏览模式快捷键
	↑ #上一行
    ↓ #下一行
    G #第一页
    g #最后一页
    空格 #下一页
    /关键词 #搜索关键词
 # 退出浏览模式，回到Linux命令行模式
    q #退出
    
# tail 命令

# 实时滚动显示文件的最后10行信息(默认10行)
tail -f 文件名
# 显示文件的最后20行信息
tail -n 20 文件名
tail -n -20 文件名
# 显示文件信息从第20行至文件末尾
tail -n +20 文件名
```

## 5. 文件查找

```shell
#1. 文件名查找

    # 语法
        find 搜索路径 -name "文件名关键词"
    # 例子
        find / -name "passwd"
        find / -name "ifcfg-*"

# 2. 文件内容查找

	# 语法
	grep -参数 要查找的目录范围
	# 参数
	-n 显示查找结果所在行号
	-R 递归查找目录下的所有文件
# 例子
	grep aries /etc
	grep aries /etc/passwd
```

## 6.系统管理

```shell
# 静态查看系统进程
	ps -aux
	
# 实时查看系统进程
	top
	# 快捷键
		↑ 下翻
		↓ 上翻
		q 退出

# 关闭进程
	kill 进程id 
# 强制关闭进程(谨慎使用)
	kill -9 进程id
```

## 7.输出

```shell
# 覆盖输出

# 将命令1的执行结果，输出到后面的文件中。
`覆盖写入`
	命令1 > 文件
# 例子
	date > date.log

# 追加输出

# 将命令1的执行结果，输出到后面的文件中。
`追加写入`
	命令1 >> 文件
# 例子
	date >> date.log

```

## 8.管道

```shell
# 语法，将命令1的输出结果，作为命令2的输入
命令1 | 命令2

# 例子
查找aries用户：cat /etc/passwd | grep -n “baizhi”
查找aries组：cat /etc/group | grep -n “baizhi”
查找sshd进程：ps -aux | grep sshd
```

## 9.系统权限

- ### 用户组

  ```shell
  1. 创建组
    `groupadd 组名`
  2. 删除组
    `groupdel 组名`
  3. 查找系统中的组
    `cat /etc/group | grep -n “组名”`
    说明：系统每个组信息都会被存放在/etc/group的文件中
  ```

- 用户

  ```shell
  1. 创建用户
    `useradd -g 组名 用户名`
  2. 设置密码
    `passwd 用户名`
  3. 查找系统账户
    说明：系统每个用户信息保存在`/etc/passwd`文件中
  4. 切换用户
    `su 用户名`
  5. 删除用户
    `userdel -r 用户名`
  6. 修改用户密码（root登录）
    `passwd 用户名`
  ```

- 权限

  - 权限含义

    ![image-20220727115812180](C:\Users\97762\AppData\Roaming\Typora\typora-user-images\image-20220727115812180.png)

  - 权限访问控制列表(ACL access controll list)

    ![image-20220727120015069](C:\Users\97762\AppData\Roaming\Typora\typora-user-images\image-20220727120015069.png)

  - 命令

    ```shell
    # 查看权限
    
    ls -la 文件
    ll 文件
    
    # 设置文件所有者
    
    语法：chown [-R] user名:group名 文件名
    参数：-R 如果是文件夹，需要使用这个参数，可以将文件夹及其内部所有文件的所有者和组全部修改
    注意：命令权限需要root
    ## 修改文件所有者
    	chown 用户名 文件名
    ## 修改文件所属组
    	chown :组名 文件名
    ## 修改文件所有者和所属组
    	chown 用户名:组名 文件名
    ## 修改文件夹的所有者和所属组
    	chown [-R] 用户名:组名 文件夹
    	
    # 权限设置1
    
    语法：chmod u±rwx,g±rwx,o±rwx 文件名
    运算符：
    	- 删除权限
    	+ 添加权限
    	= 赋值权限
    ## 给文件的所有者添加执行权限
    chmod u+x 文件名
    ## 给文件的其他人删除所有权限
    chmod o-rwx 文件名
    ## 给文件的所属组设置读写权限
    chmod g=wx 文件名
    
    # 权限设置2
    
    # 文件的每个归属方的权限的值使用rwx之和计算出来的。
    # 语法
    	`chmod [-R] nnn 文件` 
    	-R 递归设置文件夹内所有文件
    # 设置文件的权限为(所有者可读可写可执行，所属组可读可写，其他人可读)
    	chmod 764 文件名
    
    ```

    ![image-20220727141720171](C:\Users\97762\AppData\Roaming\Typora\typora-user-images\image-20220727141720171.png)



## 10.系统软件管理

- ### 压缩解压缩

  ```shell
  #压缩语法：
  tar -zcvf 压缩后文件名 被压缩文件
  
  #解压缩语法:
  tar -zxvf 压缩文件名 -C 解压后文件所在目录
  ```

  ![image-20220727142827575](C:\Users\97762\AppData\Roaming\Typora\typora-user-images\image-20220727142827575.png)

- ### rpm软件

  ```shell
  1. 安装rpm软件
    语法：`rpm -ivh xxx.rpm`
  2. 查看系统中是否已安装的过该rpm软件
    语法：`rpm -qa 软件名`
  3. 卸载rpm软件
    语法：`rpm -e 软件名`
  4. 例子：安装tree工具
    作用：查看某个目录下的文件信息
    # 以树状结构查看2层文件信息
    tree -L 2 要查看的路径
  ```

- ### yum

  > yum是基于rpm实现的，提供了除了rpm的安装软件、卸载软件等功能以外还有，自动查找、下载软件并自动处理软件的彼此之间的依赖关系，下载并安装依赖包。

  ```shell
  ## 列出所有可以安装的软件包
  	yum list
  ## 安装软件
  	yum install -y 软件名
  ## 卸载软件
  	yum remove 软件名
  ## 查找软件包
  	yum search all 软件名
  ```

## 11.Linux服务

```shell
# 例如：sshd network firewalld 等

# 服务器管理命令
	systemctl status 服务名
# 启动服务
	systemctl start 服务名
# 重启服务
	systemctl restart 服务名
# 停止服务
	systemctl stop 服务名
# 禁止服务随linux启动。
	systemctl disable 服务名
# 设置服务随linux启动。
	systemctl enable 服务名
```

## 12.ip设置(`network`)

```shell
[root@centos7 dirnew]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
----------------网卡对应的文件内容---------------------
    TYPE="Ethernet"
    PROXY_METHOD="none"
    BROWSER_ONLY="no"
    BOOTPROTO="none"
    DEFROUTE="yes"
    IPV4_FAILURE_FATAL="no"
    IPV6INIT="yes"
    IPV6_AUTOCONF="yes"
    IPV6_DEFROUTE="yes"
    IPV6_FAILURE_FATAL="no"
    IPV6_ADDR_GEN_MODE="stable-privacy"
    NAME="ens33"
    UUID="0bd5d8a5-fe1b-42de-82bd-bfa7d2984b95"
    DEVICE="ens33"
    ONBOOT="yes"
    IPADDR="192.168.199.8" # 修改这里的ip地址即可
    PREFIX="24"
    GATEWAY="192.168.199.2"
    DNS1="192.168.199.2"
    DNS2="8.8.8.8"
    IPV6_PRIVACY="no"
[root@centos7 dirnew]# systemctl restart network #重启网卡服务
```

## 13.防火墙(`firewalld`)

```shell
# 关闭防火墙
systemctl stop 服务名
# 临时关闭防火墙
systemctl stop firewalld
# 直接停止防火墙开机启动
systemctl disable firewalld
```

## 14.主机名

```shell
# 查看主机名
hostname
# 设置主机名
hostnamectl set-hostname 主机名
```

## 15.ip映射

- 域名解析
- 本地hosts编辑

```shell
[root@centos7 ~]# vim /etc/hosts
--------------下面是文件------------------
	192.168.199.8 centos7
```

## 16.SSH

```shell
# 远程登录linux
ssh 远程linux的ip或者映射域名
```

## 17.远程拷贝

```shell
scp 本地的文件 root@远程linuxip:/远程linux的文件路径
scp -r 本地的目录 root@远程linuxip:/远程linux的文件路径
```

