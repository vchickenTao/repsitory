### 1.Jenkins简介

> 待完善

### 2.安装GitLab

> 非必要，待更新，在服务器安装GitLab需要占用较大的内存资源，暂时先不学习，待日后继续学习完善，这里先用GitHub/gitee替代，起到学习的目的作用即可。

### 3.安装Jenkins

1. <a href="https://www.jenkins.io/">官网</a>上下载Jenkins安装包，得到一个jenkins.war的压缩包
2. 安装JDK环境，由于Jenkins是用Java编写的，所以他需要Java环境，如果是没有安装过JDK，则需要进行以下步骤，否则跳过：

```shell
#查找可以安装的JDK版本，选择需要的版本安装
yum search java|grep jdk
#这里选择JDK1.8
yum install -y java-1.8.0-openjdk
#查看JDK是否安装成功
[root@vchicken ~]# java -version
openjdk version "1.8.0_342"
OpenJDK Runtime Environment (build 1.8.0_342-b07)
OpenJDK 64-Bit Server VM (build 25.342-b07, mixed mode)
#说明已经安装成功

```
  3. 安装Jenkins
  - 使用XFTP工具将jenkins.war上传到服务器你自己指定的路径下
    ```shell
[root@vchicken ~]# cd /vchicken
[root@vchicken vchicken]# ls
app  copy  jenkins.war  nginx-1.17.10.tar.gz
    ```
  - 运行Jenkins
     ```shell
    [root@vchicken vchicken]# java -jar jenkins.war
    Aug 16, 2022 8:08:51 PM Main verifyJavaVersion
    WARNING: You are running Jenkins on Java 1.8, support for which will end on or after September 1, 2022. Please refer to the documentation for details on upgrading to Java 11: https://www.jenkins.io/redirect/upgrading-jenkins-java-version-8-to-11
    Running from: /vchicken/jenkins.war
    
    ......
    
    Jenkins initial setup is required. An admin user has been created and a password generated.
    Please use the following password to proceed to installation:
    
    4926df534ec94abaa2f68931c6405616
    ......
    #这里说明我们的Jenkins运行成功了，已经为我们初始化了一个管理员账户和密码，我们需要区初始化下账户
    ```
  - 访问Jenkins,默认端口是8080，通过ip+端口访问即可，首次访问会需要初始化，耐心等待，成功后会出现如下页面，。输入密码
    ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-16/08aec6ef-76de-43eb-8847-d3aaf506ae1c.png)
  - 选择安装推荐的插件即可
    ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-16/3803db18-196d-48bf-b454-3e4e48310053.png)
  - 安装maven环境
    ```shell
    #下载maven安装包，上传到/usr/local/maven路径下，解压即可
    [root@vchicken ~]# tar xcvf apache-maven-3.8.6-bin.tar.gz 
    #测试是否安装成功
    [root@vchicken ~]# /usr/local/maven/apache-maven-3.8.6/bin/mvn

    [root@vchicken ~]# /usr/local/maven/apache-maven-3.8.6/bin/mvn -v
    Apache Maven 3.8.6 (84538c9988a25aec085021c365c560670ad80f63)
    Maven home: /usr/local/maven/apache-maven-3.8.6
    Java version: 1.8.0_342, vendor: Red Hat, Inc., runtime: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.342.b07-1.el7_9.x86_64/jre
    Default locale: en_US, platform encoding: UTF-8
    OS name: "linux", version: "3.10.0-1160.62.1.el7.x86_64", arch: "amd64", family: "unix"
    ```
  4. 使用docker安装Jenkins：当然，我们也可以选择使用docker安装Jenkins，但是不推荐，网上也有很多教程，这里便不赘述。

### 4.Jenkins配置Maven+Git自动构建jar包

- 选择构建maven项目
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-17/74963fce-2846-4d59-8cf7-3db702593521.png)
- 添加git仓库地址
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-17/629ac2bd-be4a-4692-bbc3-21b946b7c0d1.png)
如果出现以上无法连接上远程仓库的原因，大概率是服务器上没有安装git，所以我们使用以下命令安装下git，再次尝试添加git仓库地址则正常。
> yum install -y git
- 指定分支
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-17/4254c222-ed2c-4936-835e-177c2ce46ad5.png)
- 配置maven地址/环境
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-17/c6149974-defc-44a0-804c-283e80e67025.png)
出现了红色提示则说明，没有识别到maven环境，那么点击`the tool configuration`，如下图：
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-17/7559c714-9c3b-410e-aea6-8dbe2a84b6ad.png)
我们取消勾选自动安装，写入之前我们maven安装包解压的文件路径，然后保存。
- 完成构建后，保存，我们点击绿色箭头开始Build scheduled，通过控制台我们可以看到如下操作，首次构建由于没有导入过依赖，所以会很慢，需要耐心等待
```shell
Started by user vchickenTao
Running as SYSTEM
Building in workspace /root/.jenkins/workspace/first
The recommended git tool is: NONE
No credentials specified
Cloning the remote Git repository
Cloning repository https://github.com/vchickenTao/jenkins-demo.git
 git init /root/.jenkins/workspace/first # timeout=10
Fetching upstream changes from https://github.com/vchickenTao/jenkins-demo.git
 git --version # timeout=10
 git --version # 'git version 1.8.3.1'
 git fetch --tags --progress https://github.com/vchickenTao/jenkins-demo.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 git config remote.origin.url https://github.com/vchickenTao/jenkins-demo.git # timeout=10
 git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
 git rev-parse refs/remotes/origin/master^{commit} # timeout=10
Checking out Revision 553e938344e7d1504d008d9455052f4fc813c2db (refs/remotes/origin/master)
 git config core.sparsecheckout # timeout=10
 git checkout -f 553e938344e7d1504d008d9455052f4fc813c2db # timeout=10
Commit message: "提交代码"
First time build. Skipping changelog.
Parsing POMs
Downloaded artifact https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-starter-parent/2.7.2/spring-boot-starter-parent-2.7.2.pom
Downloaded artifact https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-dependencies/2.7.2/spring-boot-dependencies-2.7.2.pom
Downloaded artifact https://repo.maven.apache.org/maven2/com/datastax/oss/java-driver-bom/4.14.1/java-driver-bom-4.14.1.pom
Downloaded artifact https://repo.maven.apache.org/maven2/io/dropwizard/metrics/metrics-bom/4.2.10/metrics-bom-4.2.10.pom
Downloaded artifact https://repo.maven.apache.org/maven2/io/dropwizard/metrics/metrics-parent/4.2.10/metrics-parent-4.2.10.pom
Downloaded artifact https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy-bom/3.0.11/groovy-bom-3.0.11.pom
Downloaded artifact https://repo.maven.apache.org/maven2/org/infinispan/infinispan-bom/13.0.10.Final/infinispan-bom-13.0.10.Final.pom
Downloaded artifact https://repo.maven.apache.org/maven2/org/infinispan/infinispan-build-configuration-parent/13.0.10.Final/infinispan-build-
configuration-parent-13.0.10.Final.pom
    ...... #此处省略大约一万个字
Waiting for Jenkins to finish collecting data
[JENKINS] Archiving /root/.jenkins/workspace/first/pom.xml to com.vchicken/jenkins/0.0.1-SNAPSHOT/jenkins-0.0.1-SNAPSHOT.pom
[JENKINS] Archiving /root/.jenkins/workspace/first/target/jenkins-0.0.1-SNAPSHOT.jar to com.vchicken/jenkins/0.0.1-SNAPSHOT/jenkins-0.0.1-SNAPSHOT.jar
channel stopped
Finished: SUCCESS
```

- 修改阿里云镜像，默认是从国外的镜像，下载速度很慢
找到maven的目录，conf文件下，setting.xml文件添加下列镜像，并注释原有的镜像：
```java
<mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>https://maven.aliyun.com/repository/central</url>
</mirror>
<mirror> 
    <id>aliyun-maven</id>
    <mirrorOf>*</mirrorOf> 
    <name>aliyun maven</name> 
<url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```
### 5.自动化发布到测试服务器并自动运行

- 安装publish over SSH插件，用来传输jar包到测试服务器
- 配置publish over SSH
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-17/2c7a3cf2-9af8-4f9e-8434-ecc81b7e2fd9.png)

**5.Steps执行构建前执行目标服务器脚本**
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-20/df49106e-348c-468a-8c25-2dde107797ce.png)

**x.sh文件配置**
```shell
#! /bin/bash

#获取传入的参数
echo "arg$1"

if [ -z $1 ];
        then
                echo "param is empty!"
        else
                #获取正在运行的jar包pid
                pid=`ps -ef |grep $1 | grep java | grep -v war | awk '{printf $2}'`
                echo $pid

                #如果pid为空，提示一下，否则执行kill命令
                if [ -z $pid ];
                #使用-z 做空值判断
                        then
                                echo "$1 not started"
                        else
                        #删除进程
                        kill -9 $pid
                        echo "$1 stopping..."

                check=`ps -ef | grep -W $pid | grep java`
                if [ -z $check ];
                        then
                                echo "$1 pid:$pid is stop"
                        else
                                echo "$1 stop failed"
                fi

                fi

fi

```

### 6.几种常用的构建触发器

- **快照按照依赖构建**/Build whenever a SNAPSHOT dependency is built
  - 当依赖的快照被构建时执行本job
- **触发远程构建**（例如使用脚本）
  - 远程调用本job的restap时执行本job
- **job依赖构建**/Build after other projects are bulid
  - 当依赖的job被构建时执行本job 
- **定时构建**/Build periodically
  - 使用cron表达式定时构建本job
- **向GitHub提交代码时触发的Jenkins自动构建**/GitHub hook trigger for GITScm polling
  - GitHub-WebHook出发时构建本job
- **轮训SCM**/Poll SCM
  - 使用cron表达式定时检查代码变更，变更后构建本job 

**1.定时执行任务，Jenkins cron表达式**

> 在线cron生成网站：https://crontab.guru, 可以帮助我们理解cron表达式

**注意点**：jenkins的cron表达式和普通表达式有所不同，它多了H，H值得是哈希散列。以下距举例说明：
比如当前我们的cron表达式是：H/10 * * * *，按照正常的cron表达式*/10 * * * *的意思是每隔10分钟执行一次，但是在jenkins的cron表达式中，实际是将去10分钟和当前`项目名（jobname）`的哈希值相加的绝对值，即`|hash值+10|`，才是真正的执行时间。
这样做的目的是为了解决并发下有多个定时任务同时启动的问题，使用H则可以使多定时任务错峰执行，又称为`分散负载`。

**常用例子：**
- 每30分钟构建一次：H/30 * * * *
- 每2个小时构建一次：H H/2 * * *
- 每天早上8点构建一次：0 8 * * *
- 多个时间点，中间用逗号隔开：每天的8点，12点，22点，一天构建3次：0 8,12,22 * * *
- 每天中午12点定时构建一次：H 12 * * *
- 每天下午18点定时构建一次：H 18 * * *
- 在每个小时的前半个小时内的每10分钟：H(0-29)/10 * * * *
- 周一至周五，每两小时45分钟，从上午9:45开始，每天下午15:45结束：45 9-16/2 * * 1-5
- 每两小时一次，每个工作日上午9点到下午5点(也许是上午10:38，下午12:38，下午2:38，下午4:38)：H H(9-16)/2 * * 1-5
- 每天凌晨2:00跑一次：H 2 * * *
- 每隔5分钟构建一次：H/5 * * * *
- 每两小时构建一次：H H/2 * * *
- 每天中午12点定时构建一次：H 12 * * * 或0 12 * * *（0这种写法也被H替代了）
- 每天下午18点前定时构建一次：H 18 * * *
- 每15分钟构建一次：H/15 * * * * 或*/5 * * * *(这种方式已经被第一种替代了，jenkins也不推荐这种写法了)
- 周六到周日，18点-23点，三小时构建一次：H 18-23/3 * * 6-7

**其中：**
- *：表示任意合理的数
- a-b：表示一个范围，
- a-b/c OR */c：表示一个范围内每个c构建一次
- a,b,c：表示为a、b、c时构建一次

**2.Poll SCM触发构建**
根据cron表达式去定时轮询代码的远程仓库，进行对比，如果有代码更新则重新部署，否则不执行，等待下一次查询。



### 7.容器化构建

**1.Docker jar文件打包到镜像中**

> vi或vim编辑器中，命令模式下，Ctrl+Z将任务转到后台，fg切回前台，多个后台任务使用fg+数字切换

1.把jar包打包到容器内

**dockerfile**

```dockerfile
FROM openjdk:8
EXPOSE 8080

WORKDIR /root

ADD jarfile/jenkins*.jar /root/app.jar

ENTRYPOINT ["nohup" "java","-jar","/root/app.jar","&"]
```

**打包镜像**

```shell
 docker build -t jenkins-demo .   
```

```shell
Sending build context to Docker daemon  1.322GB
Step 1/5 : FROM openjdk:8
8: Pulling from library/openjdk
001c52e26ad5: Pull complete 
d9d4b9b6e964: Pull complete 
2068746827ec: Pull complete 
9daef329d350: Pull complete 
d85151f15b66: Pull complete 
52a8c426d30b: Pull complete 
8754a66e0050: Pull complete 
Digest: sha256:86e863cc57215cfb181bd319736d0baf625fe8f150577f9eb58bd937f5452cb8
Status: Downloaded newer image for openjdk:8
 ---> b273004037cc
Step 2/5 : EXPOSE 8080
 ---> Running in 965d50f632ae
Removing intermediate container 965d50f632ae
 ---> 2943ff4138ea
Step 3/5 : WORKDIR /root
 ---> Running in 3d3b12aacdb2
Removing intermediate container 3d3b12aacdb2
 ---> 13a2acefd555
Step 4/5 : ADD jarfile/jenkins*.jar /root/app.jar
 ---> a7903570399d
Step 5/5 : ENTRYPOINT ["nohup" "java","-jar","/root/app.jar","&"]
 ---> Running in 731a01ed8d20
Removing intermediate container 731a01ed8d20
 ---> d6cbb585fcd6
Successfully built d6cbb585fcd6
Successfully tagged jenkins-demo:latest
```

**配置国内镜像**

修改 `/etc/docker/daemon.json`文件，如果没有就创建一个

写入：

```shell
{
"registry-mirrors": [
	"https://ustc-edu-cn.mirror.aliyuncs.com",
	"http://hub-mirror.c.163.com",
	"https://registry.aliyun.com"
	]
}
```

重启服务:

```shell
systemctl daemon-reload
systemctl restart docker
```

**运行**

```shell
docker run -d --name jenkins-demo -p 8080:8080 jenkins-demo #执行

1607c632f2184a5f85585d4c2898cb8be481cf10cec33af5af0d406ededccb72
docker: Error response from daemon: driver failed programming external connectivity on endpoint jenkins-demo (e41374a790bf865fd2f9c7d89be66ef84c8334510943f61bdb8a90629109bde2): Error starting userland proxy: listen tcp4 0.0.0.0:8080: bind: address already in use.
```

发现端口被占用，查看下端口的情况

```shell
netstat -tanlp

tcp6       0      0 :::5671                 :::*                    LISTEN      25856/docker-proxy  
tcp6       0      0 :::5672                 :::*                    LISTEN      25838/docker-proxy  
tcp6       0      0 :::25672                :::*                    LISTEN      25783/docker-proxy  
tcp6       0      0 :::9001                 :::*                    LISTEN      26194/docker-proxy  
tcp6       0      0 :::781                  :::*                    LISTEN      1214/./bcm-agent    
tcp6       0      0 :::8080                 :::*                    LISTEN      7231/java           
tcp6       0      0 :::4369                 :::*                    LISTEN      25873/docker-proxy  
tcp6       0      0 :::22                   :::*                    LISTEN      1154/sshd           
tcp6       0      0 :::15671                :::*                    LISTEN      25819/docker-proxy  
tcp6       0      0 :::15672                :::*                    LISTEN      25800/docker-proxy  
```

可以看到8080被占用，我们可以将占用8080的进程杀掉

```shell
kill 7231
```

然后在继续执行命令

```shell
docker run -d --name jenkins-demo -p 8080:8080 jenkins-demo

docker: Error response from daemon: Conflict. The container name "/jenkins-demo" is already in use by container "1607c632f2184a5f85585d4c2898cb8be481cf10cec33af5af0d406ededccb72". You have to remove (or rename) that container to be able to reuse that name.
```

发现名字已经被使用了，我们改下运行的镜像名称

```shell
docker run -d --name demo -p 8080:8080 jenkins-demo
1858ac14f3faa385bea780178cc7f702a2e5d73cddba94ff58530f86f3f031cc
```

查看下正在运行的镜像

```shell
docker ps
```



**Docker外挂目录**

```shell
docker run -d -p 8100:8100 --name demo-out -v /root/jarfile/jenkins-0.0.1-SNAPSHOT.jar:/app.jar openjdk:8 java -jar app.jar
```
