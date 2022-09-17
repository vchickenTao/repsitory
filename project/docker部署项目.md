### **在linux服务器上使用docker部署springboot项目**
---
#### 1.准备工作
 - 一个正常运行的spring boot项目
 - 一台安装了docker的Linux服务器


#### 2.准备一个Dockerfile文件（记得删除注释）
```java
FROM java:8

MAINTAINER vchicken #创作者名字

VOLUME /tmp   #容器的/tmp/目录挂载到本地var/lib/docker

ADD vue-admin-0.0.1-SNAPSHOT.jar app.jar  #添加jar包并重命名，此处为jar，并不会解压

#运行jar包
RUN bash -c 'touch /app.jar'

ENTRYPOINT ["nohup","java","-jar","/app.jar","&"] #项目启动命令

EXPOSE 9001  #对外暴露端口，通过该端口访问项目

```

#### 3.打包项目，并上传至服务器指定文件夹
 - 如果是maven项目则直接clean 然后package，可以选择排除test
 > 也可以打开本地的cmd，进入到项目源文件使用命令 mvn clean install -Dskiptests=true

   ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-08/bf612c27-d50a-4b8b-a0b3-4512d4e352ba.png)

 - 如果是gradle项目则先clean，然后build

 - 将jar包和Dockfile上传到服务器你所存放jar包的文件上

#### 4.使用docker构建项目镜像，并运行项目
 - 构建项目镜像
   
   > 使用命令 docker build -t vueadmin .  (这里注意镜像名称要小写，不要漏掉最后面的.号)
   
 - 如果出现以下情况，那说明镜像已经构建成功，若有步骤失败则对应修改步骤重新运行即可，如果是镜像创建异常，则需要删除镜像（docker rm -f 镜像名/镜像ID），重新生成。
   


```shell
[root@vchicken app]# docker build -t vueadmin .
Sending build context to Docker daemon  40.65MB
Step 1/7 : FROM java:8
 ---> d23bdf5b1b1b
Step 2/7 : MAINTAINER vchicken #创作者名字
 ---> Using cache
 ---> ca08849a8009
Step 3/7 : VOLUME /tmp
 ---> Using cache
 ---> 33fd4470674b
Step 4/7 : ADD vue-admin-0.0.1-SNAPSHOT.jar app.jar
 ---> a14a43bddc8c
Step 5/7 : RUN bash -c 'touch /app.jar'
 ---> Running in f0588d2e62bc
Removing intermediate container f0588d2e62bc
 ---> 68d98ce2d4f8
Step 6/7 : ENTRYPOINT ["nohup","java","-jar","/app.jar","&"]
 ---> Running in dd945cf8d83d
Removing intermediate container dd945cf8d83d
 ---> b8b1c8d742ac
Step 7/7 : EXPOSE 9001
 ---> Running in 34cf30bdd220
Removing intermediate container 34cf30bdd220
    ---> 704d6c8f96a1
   Successfully built 704d6c8f96a1 #构架成功，704d6c8f96a1为镜像ID
   Successfully tagged vueadmin:latest
   ```
   
   
   
- 使用docker运行项目
  > docker run -d -p 9001:9999 vueadmin
    -d 以分离模式，也就是在后台中，运行容器
    -p 9000:9999 把宿主机的9999端口映射到容器的9001端口
    vueadmin 使用的镜像
  ```shell
[root@vchicken app]# docker run -d -p 9001:9999 vueadmin
2a4fbe717a6cf3586ccf7c0c42d4b88b00b9f03021803920ac3ba27d4d49db95
  ```

#### 5.测试
到这里使用docker部署springboot项目就完成了，可以使用服务器的外网ip/域名，加上对应的暴露端口以及上下文，去访问自己需要的接口，快去试试吧！ 