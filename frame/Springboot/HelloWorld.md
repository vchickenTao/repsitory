# **🦡 Springboot HelloWorld项目**

------

> [!TIP]
>
> 老规矩，先来写个HelloWorld感受以下Springboot！@vchicken

## 1、系统要求

- [Java 8](https://www.java.com/) & 兼容java14 .
- Maven 3.3+
- idea 2019.1.2



### 1.1、maven设置

```xml
 <mirrors>
      <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
      </mirror>
  </mirrors>
 
  <profiles>
         <profile>
              <id>jdk-1.8</id>
              <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.8</jdk>
              </activation>
              <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
              </properties>
         </profile>
  </profiles>
```

## 2、HelloWorld

需求：浏览发送/hello请求，响应 Hello，Spring Boot 2 

### 2.1、创建maven工程



### 2.2、引入依赖

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

    </dependencies>
```

### 2.3、创建主程序

```java
/**
 * 主程序类
 * @SpringBootApplication：这是一个SpringBoot应用
 */
@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
```

### 2.4、编写业务

```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String handle01(){
        return "Hello, Spring Boot 2!";
    }
}
```

### 2.5、测试

直接运行main方法

### 2.6、简化配置

application.properties

```xml
# 设置端口号
server.port=8888
```

### 2.7、简化部署

```xml
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

在IDEA的Maven插件上点击运行 clean 、package，把helloworld工程项目的打包成jar包，

打包好的jar包被生成在helloworld工程项目的target文件夹内。

用cmd运行java -jar boot-01-helloworld-1.0-SNAPSHOT.jar，既可以运行helloworld工程项目。

将jar包直接在目标服务器执行即可。

### 2.8、测试

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-16/ebec081b-9eee-4eda-8f86-21b05c269a65_helloworld.png)

注意点：

- 取消掉cmd的快速编辑模式