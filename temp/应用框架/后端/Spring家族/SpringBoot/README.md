# SpringBoot

## 简介

SpringBoot提供了一种快速使用Spring的方式，基于约定优于配置的思想，可以让开发人员不必在配置与逻辑业务之间进行思维的切换，全身心的投入到逻辑业务的代码编写中，从而大大提高了开发的效率，一定程度上缩短了项目周期。2014 年 4 月，Spring Boot 1.0.0 发布。Spring的顶级项目之一(https://spring.io)。

### SpringBoot的功能

![截屏2021-08-25 10.56.05](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-08-25%2010.56.05.png)

-  **自动配置**

Spring Boot的自动配置是一个运行时(更准确地说，是应用程序启动时)的过程，考虑了众多因素，才决定 Spring配置应该用哪个，不该用哪个。该过程是SpringBoot自动完成的。

- **起步依赖**

起步依赖本质上是一个Maven项目对象模型(Project Object Model，POM)，定义了对其他库的传递依赖 ，这些东西加在一起即支持某项功能。 简单的说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的功能。

- **辅助功能**

提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等。



**Spring Boot 并不是对 Spring 功能上的增强，而是提供了一种快速使用 Spring 的方式。**



### Spring的生态

web开发

数据访问

安全控制

分布式

消息服务

移动开发

批处理

$...$



### 与Spring对比

#### Spring

-  **Spring 配置繁琐**

虽然Spring的组件代码是轻量级的，但它的配置却是重量级的。一开始，Spring 用 XML 配置，而且是很多 XML配置。Spring 2.5 引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。 Spring 3.0 引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。 所有这些配置都代表了开发时的损耗。因为在思考 Spring 特性配置和解决业务问题之间需要进行思维切换，所 以编写配置挤占了编写应用程序逻辑的时间。和所有框架一样，Spring 实用，但它要求的回报也不少。

- **Spring 依赖繁琐**

项目的依赖管理也是一件耗时耗力的事情。在环境搭建时，需要分析要导入哪些库的坐标，而且还需要分析导 入与之有依赖关系的其他库的坐标，一旦选错了依赖的版本，随之而来的不兼容问题就会严重阻碍项目的开发 进度。

#### SpringBoot

- Create stand-alone Spring applications
  - 创建独立Spring应用

- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)
  - 内嵌web服务器

- Provide opinionated 'starter' dependencies to simplify your build configuration
  - 自动starter依赖，简化构建配置

- Automatically configure Spring and 3rd party libraries whenever possible
  - 自动配置Spring以及第三方功能

- Provide production-ready features such as metrics, health checks, and externalized configuration
  - 提供生产级别的监控、健康检查及外部化配置

- Absolutely no code generation and no requirement for XML configuration
  - 无代码生成、无需编写XML



*<u>SpringBoot是整合Spring技术栈的一站式框架</u>*

<u>*SpringBoot是简化Spring技术栈的快速开发脚手架*</u>



## 快速入门

**开发步骤**

① 创建Maven项目

② 导入SpringBoot起步依赖 

```xml

<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.5.0</version>
		<relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
</dependencies>
```

③ 定义Controller

④ 编写引导类（启动类，SpingBoot项目的入口）

⑤ 启动测试





# 配置文件

## 配置文件分类

SpringBoot是基于约定的，所以很多配置都有默认值，但如果想使用自己的配置替换默认配置的话，就可以使用 `application.properties` 或者 `application.yml(application.yaml)` 进行配置。

- `properties`

```properties
server.port=8080
```

- `yml`

```yml
server:
	port: 8080
```

优先级：`application.proerties > application.yml > application.yaml `



### yaml

YML文件是以数据为核心的，比传统的 `xml` 方式更加简洁。YAML文件的扩展名可以使用 `.yml` 或者 `.yaml` 。

#### 基本语法

- 大小写敏感
- **数据值前边必须有空格，作为分隔符**
- 使用缩进表示层级关系
- 缩进时不允许使用Tab键，只允许使用空格(各个系统 Tab对应的 空格数目可能不同，导致层次混乱)。 
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
- 表示注释，从这个字符一直到行尾，都会被解析器忽略

#### 数据格式

- 对象(map)：键值对的集合

```yml
name: tsuiraku
age: 21

person:
	name: zhangsan
	age: 30

# 行内写法
person: {name: zhangsan}
```

- 数组：一组按次序排列的值

```yml
address:
  - beijing
  - shanghai
# 行内写法
address: [beijing,shanghai]
```

- 纯量：单个的、不可再分的值

```yml
msg1: 'hello \n world' # 单引忽略转义字符 
输出：
	hello \n world
	
msg2: "hello \n world" # 双引识别转义字符
输出：
	hello
	 world
```

#### 参数引用

```yml
name: tsuiraku
person:
	name: ${name} # 引用上边定义的name值
```



## 如何读取配置文件内容

### @Value

```java
public class HelloController{
  @Value("${name}") // tsuiraku
  private String name;
  
  @Value("${person.name}") // zhangsan
  private String name;
  
  @Value("${address[0]}") // beijing
  private String name;
}
```

### Environment

```java
public class HelloController{
	@Autowired
  private Enviroment env;
  
  System.out.print(env.getProperty("person.name")) // zhangsan
}
```

###  @ConfigurationProperties

作用：配置文件中将会有提示

- 依赖的引入（非必需）

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>


################一般打包时不引入################


 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-configuration-processor</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
</build>
```

即在 `.yml` 或者 `.properties` 中将会有相应的代码段提示

```yml
name: tsuiraku
age: 21

person:
	name: zhangsan
	age: 30
```

```java
@Component
@ConfigurationProperties
public class Person{
  // 属性和application.properties/application.yml中需要对应
  private String name; // tsuiraku
  private int age; // 21
  
  // get/set方法
}
```

```java
@Component
@ConfigurationProperties(perfix = "person")
public class Person{
  private String name; // zhangsan
  private int age; // 30
  
  // get/set方法
}
```



## profile

我们在开发Spring Boot应用时，通常同一套程序会被安装到不同环境，比如:开发、测试、生产等。其中数据库地址、服务 器端口等等配置都不同，如果每次打包时，都要修改配置文件，那么非常麻烦。profile功能就是来进行动态配置切换的。

### profile配置方式

- 配置文件方式

```
application-dev.properties
application-pro.properties
application-test.properties
```

```yml
spring.profiles.active: dev # 启用application-dev.properties
```

- `yml` 的多文档方式 

```yml
---
server:
	port: 8001
spring:
	profiles: dev
---
server:
	port: 8002
spring:
	profiles: pro
---
server:
	port: 8003
spring:
	profiles: test
---
spring:
	profiles:
		active: dev
```



### profile激活方式

- 配置文件 
- 虚拟机参数 

```
VM option:

-Dspring.profiles.actice=dev
```

- 命令行参数

```
Program arguments:

--spring.profiles.actice=dev

java -jar xxx.jar --spring.profiles.actice=pro
```



## 内部配置加载顺序

Springboot程序启动时，会从以下位置加载配置文件。

```
优先级
① ./config/ :当前项目下的/config目录下
② ./ :当前项目的根目录
③ classpath:/config/ :classpath的/config
④ classpath:/ :classpath的根目录
```

加载顺序为上文的排列顺序，高优先级配置的属性会生效。



## 外部配置加载顺序 

通过 [官网](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html) 查看外部属性加载顺序

```
java -jar xxx.jar --server.port=8082
```



# 自动配置

① **依赖管理**

```xml
<!-- 依赖管理 -->   
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
</parent>

<!-- spring-boot-starter-parent的父项目 -->
 <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.4.RELEASE</version>
  </parent>

<!-- 几乎声明了所有开发中常用的依赖的版本号，自动版本仲裁机制 -->
```

- 在 `spring-boot-starter-parent` 中定义了各种技术的版本信息，组合了一套最优搭配的技术版本。
- 在各种 starter 中，定义了完成该功能需要的坐标合集，其中大部分版本信息来自于父工程。
- 我们的工程继承 parent，引入 starter 后，通过依赖传递，就可以简单方便获得需要的jar包，并且不会存在 版本冲突等问题，可以不需要写版本号。

② **修改版本号**

```xml
<!-- 查看spring-boot-dependencies里面规定当前依赖的版本所用的key -->
<!-- 在当前项目里面重写配置 -->
<properties>
  <mysql.version>5.1.43</mysql.version>
</properties>
```

③ **开发导入starter场景启动器**

```xml
<!-- 官方 spring-boot-starter-* ：*代表某种场景 -->
<!-- 只要引入starter，场景的所有常规需要的依赖都自动引入 -->

<!-- *-spring-boot-starter：第三方为提供的简化开发的场景启动器 -->

<!-- 所有场景启动器最底层的依赖 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>2.3.4.RELEASE</version>
  <scope>compile</scope>
</dependency>
```

- SpringBoot所有支持的场景：[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter)



## 自动配置

- Tomcat
- SpringMVC
- 自动配置web开发常见场景
- $...$

### 默认包扫描规则

- 默认的包结构
  - 主程序所在包及其下面的所有子包里面的组件都会被默认扫描

```
com
 +- example
     +- myapplication
         +- MyApplication.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

- 想要改变扫描路径，`@SpringBootApplication(scanBasePackages="com.tsuiarku")`

- 或者 `@ComponentScan` 指定扫描路径

```
@SpringBootApplication
等价于
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.tsuiraku.boot") // 修改包扫描
```

- 各种配置拥有默认值
  - 默认配置最终都是映射到某个类上
  - 配置文件的值最终会绑定每个类上，这个类会在容器中创建对象

- 按需加载所有自动配置项
  - 非常多的 starter
  - 引入了哪些场景这个场景的自动配置才会开启
  - SpringBoot 所有的自动配置功能都在 `spring-boot-autoconfigure` 包里面

## <u>自动配置的原理</u>

### @SpringBootApplication

相当于

```
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.tsuiraku.boot")
```

由以下注解合成

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{}    
```

### @SpringBootConfiguration

- 底层是 @Configuration

### @ComponentScan

- 指定包的扫描

### @EnableAutoConfiguration

由以下注解合成

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

### @AutoConfigurationPackage

```java
@Import(AutoConfigurationPackages.Registrar.class)  // 给容器中导入一个组件
public @interface AutoConfigurationPackage {}

// 利用Registrar给容器中导入一系列组件
// 将指定的一个包下的所有组件导入进来，即 MainApplication 所在包下
```

### @Import(AutoConfigurationImportSelector.class)

```
1.利用getAutoConfigurationEntry(annotationMetadata)向容器中批量导入组件
2.调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取所有需要导入到容器中的配置类
3.利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)获得所有的组件
4.从META-INF/spring.factories加载文件
	默认扫描当前系统里面所有META-INF/spring.factories位置的文件
  spring-boot-autoconfigure-2.3.4.RELEASE.jar包中也存在META-INF/spring.factories
```



### 自动配置流程

- SpringBoot先加载所有的自动配置类  类如：xxxAutoConfiguration。
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件所指定的值。指定配置的值从xxxProperties获取拿。xxxProperties和配置文件进行了绑定。

- 生效的配置类将会给容器中装配组件。
- 当容器中存在这些组件，这些组件提供的功能就存在。

- SpringBoot默认会在底层配好所有的组件。但是如果用户自己配置了以用户的优先。
- 定制化配置
  - 用户使用自己写的@Bean替换原生的底层的组件。
  - 用户查看组件获取的配置文件（xxxProperties）的值进行修改。（推荐使用）

**xxxAutoConfiguration ----> 组件  ---->** **xxcaProperties中获取相应配置的值  ----> application.properties**



### 最佳实践

- 引入场景依赖

  - [官方文档-starter](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter)

- 查看自动配置（选做）

  - 分析，引入场景对应的自动配置一般都生效

  - 配置文件 `application.yml` 中 `debug=true` 开启自动配置报告
    - Negative（不生效）
    - Positive（生效）

- 是否需要修改

  - 参照文档修改配置项
    - [官方配置文档](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties)
    - 分析。xxxProperties 绑定配置文件
  - 自定义加入或者替换组件
  - 自定义器  **XXXCustomizer**

  - $...$




## 容器功能

### 组件添加

#### @Configuration

- 基本使用：告诉 SpringBoot 这是一个配置类
- **Full 模式与 Lite 模式**
  - 配置类组件之间无依赖关系用 Lite 模式加速容器启动过程，减少判断
  - 配置类组件之间有依赖关系，方法会被调用得到之前单实例组件，用 Full 模式

```java
======================Configuration示例======================

/**
 * 1.配置类里面使用@Bean标注在方法上给容器注册组件，默认单实例的
 * 2.配置类本身也是组件
 * 3.proxyBeanMethods：代理bean的方法
 *      Full(proxyBeanMethods = true) [保证每个@Bean方法被调用多少次返回的组件都是单实例的]
 *      Lite(proxyBeanMethods = false) [每个@Bean方法被调用多少次返回的组件都是新创建的]
 *      组件依赖必须使用Full模式默认。其他默认是否Lite模式
 *
 *
 *
 */
@Configuration(proxyBeanMethods = false) // 告诉SpringBoot这是一个配置类等同于配置文件
public class MyConfig {

    /**
     * Full：外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
     * @return
     */
    @Bean 
  	// 给容器中添加组件
  	// 以方法名作为组件的id=user01
  	// 返回类型=User就是组件类型
  	// 返回的值=user，就是组件在容器中的实例
    public User user(){
        User user = new User("tsuiraku", 21);
      	// 当proxyBeanMethods=true
        // user组件依赖了Pet组件
      	// user.getPet() == context.getBean("tomcatPer") -> true
      
        // 当proxyBeanMethods=false
      	// user.getPet() == context.getBean("tomcatPer") -> false
        user.setPet(tomcatPet());
        return user;
    }

    @Bean("tom")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}


======================@Configuration测试代码======================

@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.tsuiraku.boot")
public class MainApplication {

    public static void main(String[] args) {
        // 返回我们IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);

        // 查看容器里面的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }

        // 从容器中获取组件

        Pet tom01 = run.getBean("tom", Pet.class);

        Pet tom02 = run.getBean("tom", Pet.class);

        System.out.println("组件："+ (tom01 == tom02)); // true

        // 验证配置类本身也是组件
        MyConfig bean = run.getBean(MyConfig.class);
        System.out.println(bean);

        // @Configuration(proxyBeanMethods = true)代理对象调用方法
        // SpringBoot总会检查这个组件是否在容器中存在
        // 始终保持组件单实例
      	// 默认：proxyBeanMethods = true
      
      	// proxyBeanMethods = false
      
        User user1 = bean.user();
        User user2 = bean.user();
        System.out.println(user1 == user2); 
        // proxyBeanMethods = true -> true
        // proxyBeanMethods = false -> false
 
        User user01 = run.getBean("user", User.class);
        Pet tom = run.getBean("tom", Pet.class);

        System.out.println("用户的宠物："+ (user01.getPet() == tom));

    }
}
```

#### @Import

```java
/*
 * @Import({User.class, DBHelper.class})
 *  容器中自动创建出这两个类型的组件、默认组件的名字=name就是全类名
 *
 *
 *
 */

@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods = false) // 告诉SpringBoot这是一个配置类等同于配置文件
public class MyConfig {
}
```

#### @Conditional

条件装配：满足 Conditional 指定的条件，则进行组件注入

```java
======================测试条件装配======================
public class MyConfig {

 		@ConditionalOnBean(name = "tom") // 当容器中存在name=tom的bean -> 注入user01
  	// @ConditionalOnMissingBean(name = "tom")  当容器中不存在name=tom的bean -> 注入user01
    @Bean 
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }

    @Bean("tom")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}

public static void main(String[] args) {
        // 返回我们IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);

        // 查看容器里面的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }

        boolean tom = run.containsBean("tom");
        System.out.println("容器中Tom组件："+ tom);

        boolean user01 = run.containsBean("user01");
        System.out.println("容器中user01组件："+ user01);


    }
```

### 原生配置文件的引入

#### @ImportResource

```xml
======================beans.xml=========================
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="haha" class="com.tsuiraku.boot.bean.User">
        <property name="name" value="zhangsan"></property>
        <property name="age" value="18"></property>
    </bean>

    <bean id="hehe" class="com.tsuiraku.boot.bean.Pet">
        <property name="name" value="tomcat"></property>
    </bean>
</beans>
```

```java
@ImportResource("classpath:beans.xml")
public class MyConfig {}

======================测试======================
        boolean haha = context.containsBean("haha");
        boolean hehe = context.containsBean("hehe");
        System.out.println("haha："+ haha);//true
        System.out.println("hehe："+ hehe);//true
```

### 配置绑定

#### @ConfigurationProperties + @Component

```properties
mycar.brand=BYD
mycar.prive=1000
```

```java
/**
 * 只有在容器中的组件，才会拥有SpringBoot提供的强大功能
 * 将Car注入容器
 */
@Component
@ConfigurationProperties(prefix = "mycar")
public class Car {

    private String brand;
    private Integer price;

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public Integer getPrice() {
        return price;
    }

    public void setPrice(Integer price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", price=" + price +
                '}';
    }
}
```

#### @EnableConfigurationProperties + @ConfigurationProperties

一般用于第三方包中没有加 @Component 的情况

```java
/**
 * 只有在容器中的组件，才会拥有SpringBoot提供的强大功能
 */
@ConfigurationProperties(prefix = "mycar")
public class Car {
	...
}

@Configuration
@EnableConfigurationProperties(Car.class)
// 一般写在config配置类
// 开启Car的属性配置
// 将组件Car自动注册导入容器
public class MyConfig {

    @Bean 
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }

    @Bean("tom")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}
```





# Web开发

> If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.2.9.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.
>
> **不用@EnableWebMvc注解。使用** `@Configuration` **+** `WebMvcConfigurer` **自定义规则**
>
> 
>
> If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.
>
> **声明** `WebMvcRegistrations` **改变默认底层组件**
>
> 
>
> If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.
>
> **使用** `@EnableWebMvc+@Configuration+DelegatingWebMvcConfiguration 全面接管SpringMVC`

## 简单功能的分析

### 静态资源访问

#### 静态资源的目录

> 类路径下（`classpath:`）
>
> - `/static` 
> -  `/public` 
> -  `/resources` 
> -   `/META-INF/resources`
>

访问 ： 当前项目根路径 + 静态资源名 

`http://localhost:8080/` + 静态资源名



**原理**

当请求进来，----> Controller是否能够处理。不能处理的所有请求都交给静态资源处理器。静态资源也找不到的资源则响应 404 页面。



#### 静态资源访问前缀

```yml
spring:
  mvc:
    static-path-pattern: /resources/**
    # 默认：/**
    # 访问静态资源：当前项目根路径 + /resources/ + 静态资源名
```



**改变静态资源目录路径**

```yml
spring:
	mvc:
	  resources:
    	static-locations: [classpath:/haha/]
    	# 当前访问的静态资源文件在/haha文件夹下
    	# 访问静态资源：当前项目根路径 +（是否添加了前缀）+ 静态资源名
    	
```



#### webjars

路径

- /webjars/**
- classpath:/META-INF/resources/webjars/

自动映射：`http:localhost:8080/webjars/jquery/3.5.1/jquery/**   `

文档：[官方文档-webjars](https://www.webjars.org/)

```xml
<dependency>
    <groupId>org.webjars</groupId>
	  <artifactId>jquery</artifactId>
  	<version>3.5.1</version>
</dependency>
```



### 欢迎页面

- 静态资源路径  `index.html`

- - 可以配置静态资源路径
  - 但不可以配置静态资源的访问前缀。否则导致 `index.html` 不能被默认访问

```yml
spring:
#  mvc:
#    static-path-pattern: /resources/**   
# 会导致welcome page功能失效
# 当让也可访问/resources/index.html

  resources:
    static-locations: [classpath:/haha/]
```

- Controller 能处理够的 /index



### 自定义Favicon

favicon.ico 放在静态资源目录下即可



### <u>静态资源配置原理</u>

- SpringBoot 启动默认加载 `xxxAutoConfiguration` 

  web 包下的自动配置类：

  <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-08-30%2012.03.50.png" alt="截屏2021-08-30 12.03.50" style="float:left;zoom:50%" />

- **SpringMVC** 自动配置类：`WebMvcAutoConfiguration`

  ```java
  @Configuration(proxyBeanMethods = false)
  @ConditionalOnWebApplication(type = Type.SERVLET)
  @ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
  @ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
  @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
  @AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
  		ValidationAutoConfiguration.class })
  public class WebMvcAutoConfiguration {}
  ```

- 容器中的配置

  ```java
  public class WebMvcAutoConfiguration {	
    ...
  	@Configuration(proxyBeanMethods = false)
  	@Import(EnableWebMvcConfiguration.class)
  	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
  	@Order(0)
  	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {}
  	...
  }
  ```

  - `WebMvcProperties` 

    ```java
    @ConfigurationProperties(
        prefix = "spring.mvc"
    )
    ```

  - `ResourceProperties`

    ```java
    @ConfigurationProperties(
        prefix = "spring.resources",
        ignoreUnknownFields = false
    )
    ```



#### 配置类只有一个有参构造器

有参构造器所有参数的值都会从容器中确定

- `ResourceProperties resourceProperties`：获取和 spring.resources 绑定的所有的值的对象；
- `WebMvcProperties mvcProperties` ：获取和 spring.mvc 绑定的所有的值的对象；
- `ListableBeanFactory beanFactor`：Spring 的 beanFactory；
- `HttpMessageConverters` ：找到所有的HttpMessageConverters；
- `ResourceHandlerRegistrationCustomizer` ：找到资源处理器的自定义器；
- `DispatcherServletPathServletRegistrationBean`   应用注册Servlet、Filter等。

```java
	public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
				ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
				ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
				ObjectProvider<DispatcherServletPath> dispatcherServletPath,
				ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
			this.resourceProperties = resourceProperties;
			this.mvcProperties = mvcProperties;
			this.beanFactory = beanFactory;
			this.messageConvertersProvider = messageConvertersProvider;
			this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
			this.dispatcherServletPath = dispatcherServletPath;
			this.servletRegistrations = servletRegistrations;
		}
```

#### <u>资源处理默认规则</u>

```java
@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
            if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
            
            //
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}
```

- `this.resourceProperties.isAddMappings`：静态资源的路径配置

  ```yml
  spring:
  #  mvc:
  #    static-path-pattern: /res/**
  
    resources:
      add-mappings: false   # 禁用所有静态资源规则
  ```

- `this.resourceProperties.getStaticLocations()`：默认位置

  ```java
  @ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
  public class ResourceProperties {
  
  	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
  			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };
  
  	/**
  	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
  	 * /resources/, /static/, /public/].
  	 */
  	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
  ```

  

#### <u>欢迎页的处理规则</u>

```java
// HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。	

	@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
			return welcomePageHandlerMapping;
		}

	WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
			ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
		if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
            // 要用欢迎页功能，必须是/**
			logger.info("Adding welcome page: " + welcomePage.get());
			setRootViewName("forward:index.html");
		}
		else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
            // 调用Controller  /index
			logger.info("Adding welcome page template: index");
			setRootViewName("index");
		}
	}
```



## 请求参数处理

### 请求映射

#### Rest使用与原理

Rest风格支持（*使用**HTTP**请求方式动词来表示对资源的操作*）

- - *以前：/getUser*  *获取用户*    */deleteUser* *删除用户*   */editUser*  *修改用户*      */saveUser* *保存用户*
  - *现在： /user*    *GET-获取用户*    *DELETE-删除用户*     *PUT-修改用户*      *POST-保存用户*

- - 核心Filter：`HiddenHttpMethodFilter` 

- - - 用法，表单中添加 `<input>` 

      ```html
      <form actioo="/user" method="post">
        <input name="_method" type="hidden" value="delete"></input>
      </form>
      ```

    - 需要在 SpringBoot 中手动开启表单的 Rest 功能

      ```yml
      spring:
        mvc:
          hiddenmethod:
            filter:
              enabled: true   #开启页面表单的Rest功能
      ```

- - 扩展：如何把 _method 这个名字换成我们自己喜欢的？

    - 重写 `hiddenHttpMethodFilter`

    ```java
    @Configuration
    public class WebConfig{
      @Bean
      publice HiddenHttpMethodFilter hiddenHttpMethodFilter(){
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter(;)
        methodFilter.setMethodParam("_m");
        return methodFilter;
      }
    }
    ```

    

**原理**

- 表单提交会带上 `_method=PUT/DELETE/PATCH`
- 当请求发送过来，会被 `HiddenHttpMethodFilter` 拦截

- - `HiddenHttpMethodFilter` 的工作：检查请求是否正常，并且是 POST 请求方式

- - - 获取到 ` _method` 的值
      - 兼容以下请求；`PUT.DELETE.PATCH`

- - - 原生 `request（post）`，*包装模式* `requesWrapper` 重写了 `getMethod` 方法，返回的是传入的值
    - 过滤器链放行的时候用 `wrapper`。以后的方法调用 `getMethod` 是调用 `requesWrapper` 的



#### 请求映射原理

![img](https://gitee.com/tsuiraku/typora/raw/master/img/1603181171918-b8acfb93-4914-4208-9943-b37610e93864.png)

SpringMVC功能分析都从 `org.springframework.web.servlet.DispatcherServlet ---> doDispatch()`



```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// 找到当前请求使用哪个Handler（Controller的方法）处理
				mappedHandler = getHandler(processedRequest);
                
                // HandlerMapping：处理器映射。/xxx --->xxxx
```

![img](https://cdn.nlark.com/yuque/0/2020/png/1354552/1603181460034-ba25f3c0-9cfd-4432-8949-3d1dd88d8b12.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_14%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

**RequestMappingHandlerMapping**：保存了所有 `@RequestMapping` 和 `handler` 的映射规则。

![img](https://gitee.com/tsuiraku/typora/raw/master/img/1603181662070-9e526de8-fd78-4a02-9410-728f059d6aef.png)

所有的请求映射都在HandlerMapping中。



- SpringBoot 自动配置欢迎页的 `WelcomePageHandlerMapping` 。能访问到 `index.html` ；
- SpringBoot 自动配置了默认 的 `RequestMappingHandlerMapping`

- 请求进来，依次尝试所有的 `HandlerMapping` 查看是否有请求信息。

- - 如果有就找到这个请求对应的 `handler`
  - 如果没有就是下一个 `HandlerMapping`

- 自定义一些自定义的映射处理，也可以手动给容器中放 `HandlerMapping` 。自定义 **HandlerMapping**

```java
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
```



### 普通参数和基本注解

#### 注解方式

##### @PathVariable

- 路径变量

```java
@RestController
public class ParameterTestController {

    @GetMapping("/car/{id}/owner/{username}")
    public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                     @PathVariable("username") String name,
                                     @PathVariable Map<String,String> pv{


        Map<String,Object> map = new HashMap<>();

        map.put("id", id);
        map.put("name", name);
        map.put("pv", pv);

        return map;
    }

}
```

##### @RequestHeader

- 获取请求头

```java
@RestController
public class ParameterTestController {

    @GetMapping("/car/3/owner/lisi?age=18%%inters=basketball&inters=game")
    public Map<String,Object> getCar(@RequestHeader("User-Agent") String userAgent,
                                     @PathVariable Map<String,String> header{


        Map<String,Object> map = new HashMap<>();

        map.put("userAgent", userAgent);
        map.put("headers",header);

        return map;
    }

}
```

##### @RequestParam

- 获取请求参数

```java
@RestController
public class ParameterTestController {

    @GetMapping("/car/3/owner/lisi?age=18%%inters=basketball&inters=game")
    public Map<String,Object> getCar(@RequestParam("age") Integer age,
                                     @RequestParam("inters") List<String> inters,
                                     @RequestParam Map<String, String> params{


        Map<String,Object> map = new HashMap<>();

        map.put("age", age);
        map.put("inters",inters);
        map.put("params",params);           

        return map;
    }
}
```

##### @CookieValue

- 获取cookie值

```java
@RestController
public class ParameterTestController {

    @GetMapping("/cookie")
    public Map<String,Object> getCar(@CookieValue("_ga") String _ga,
                                     @CookieValue("_ga") Cookie cookie{


        Map<String,Object> map = new HashMap<>();

        map.put("_ga", _ga);
        map.put("cookie",cookie);           

        return map;
    }
}
```

##### @RequestBody

- 获取请求体
- POST请求才有请求体

```java
@RestController
public class ParameterTestController {

    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }
}
```

##### @RequestAttribute

- 获取request域内数据

```JAVA
@Controller
public class ParameterTestController {

    @GetMapping("/goto")
    public String goToPage(HttpServletRequest request){
      	request.setAttribute("msg", "success");
        request.setAttribute("code", 200);
        return "forward:/success";
    }
  
  	@ResponseBody
    @GetMapping("/success")
    public String success(@RequestAttribute("msg") String msg,
                          @RequestAttribute("code" Integer code,
                           HttpServletRequest request){
        request.getAttribute("msg");
  			request.getAttribute("code"); 
        return null;
    }
}
```

##### @MatrixVariable

- 矩阵变量
- SpringBoot 默认是禁用矩阵变量的功能
  - 需要手动开启，对于路径的处理：**<u>UrlPathHelper</u>** 进行解析
    - <u>**removeSemicolonContent=false**</u>（是否移除分号内容）
  - 矩阵变量必须有 url 路径变量才能被解析

```java
@Controller
public class ParameterTestController {  
    @GetMapping("/cars/{path}") // 请求路径：/cars/sell;low=34;brand=byd,audi,yd
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();

        map.put("low",low);
        map.put("brand",brand);
        map.put("path",path);
        return map;
    }

    // /boss/1;age=20/2;age=10

    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
        Map<String,Object> map = new HashMap<>();

        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        return map;
}
```

- **注意**，需要在 SpringBoot 中开启矩阵变量

  - 写法一

  ```java
  @Configuration
  public WebConfig implements WebMvcConfigurer{
    @Override
    public void configurePathMatch(PathMatchConfigure configurer){
  			UrlPathHelper urlPathHelper = new UrlPathHelper();
      	urlPathHelper.setRemoveSemicolonContent(false);
     		configurer.setUrlPathHelper(urlPathHelper);
    }
  }
  ```

  - 写法二

  ```java
  @Configuration
  public WebConfig{
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
      	return new WebMvcConfigurer{
          @Override
           public void configurePathMatch(PathMatchConfigure configurer){
           		UrlPathHelper urlPathHelper = new UrlPathHelper();
      				urlPathHelper.setRemoveSemicolonContent(false);
     					configurer.setUrlPathHelper(urlPathHelper);
           }
  			};
    }
  }
  ```



### <u>参数处理原理</u>





p32 ～ 42 源码 to do









## 视图解析与模版引擎

即处理完请求，需要跳转到视图的过程。***SpringBoot*** 默认不支持 ***JSP***，需要引入第三方模板引擎技术实现页面渲染。



<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-21%2013.19.16.png" alt="" style="zoom:40%;" />



### <u>视图解析原理</u>

to do p47



### Thymeleaf

> Thymeleaf is a modern server-side Java template engine for both web and standalone environments, capable of processing HTML, XML, JavaScript, CSS and even plain text.

***Thymeleaf*** 是现代化、服务端 ***Java*** 模板引擎，一般由后端人员使用。（我选择前后端分离开发）

**官网：https://www.thymeleaf.org/**



#### 快速使用

**依赖引入**

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymleaf</artifactId>
</dependency>
```

***ThymeleafAutoConfiguration* 自动装配**

```java
// 依赖于springboot的自动装配
Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ThymeleafProperties.class)
@ConditionalOnClass({ TemplateMode.class, SpringTemplateEngine.class })
@AutoConfigureAfter({ WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class })
public class ThymeleafAutoConfiguration { }
```

**页面位置**

```java
public static final String DEFAULT_PREFIX = "classpath:/templates/"; // 页面位置

public static final String DEFAULT_SUFFIX = ".html";  // 后缀默认
```

**页面开发**

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text="${msg}">Hello Thymeleaf</h1>
<h2>
    <a href="www.baidu.com" th:href="${link}">百度</a> 
</h2>
</body>
</html>
```

**Controller**

```java
@Controller
public class ThmeleafTest {
  @GetMapping("/hello")
  public String hello(Model model) {
    model.addAttribute("msg", "你好tsuiraku");
    model.addAttribute("link", "www.tsuiraku.com");
		return "success";
  }
}
```





## 拦截器

### HandlerInterceptor

> ***preHandler***
>
> ***postHandler***
>
> ***afterCompletion***



<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-21%2014.21.43.png" style="zoom:50%;" />

1. 编写拦截器，实现接口 ***HandlerInterceptor***

```java
/**
 * 1.配置好拦截器需要拦截哪些请求
 * 2.将这些配置放在容器中
 */
public class myInterceptor implements HandlerInterceptor {

    /**
     * 目标方法执行之前
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
				// 编写逻辑
      	if() {
          return true;
				}
        return false;
    }

    /**
     * 目标方法执行完成以后
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    }

    /**
     * 页面渲染以后
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    }
}
```

2. 实现接口 ***WebMvcConfigurer***，将拦截器注册

```java
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {
  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new myInterceptor()) // 注册拦截器
    				.addPathPatterns("/**") // /**：所有请求都被拦截包括静态资源
            .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**"); 																	// 放行的请求
    }
}
```



### <u>拦截器原理</u>

p49



## 文件上传

> ***MutipartAutoConfiguration***：文件自动配置
>
> ```java
> @Configuration(
>     proxyBeanMethods = false
> )
> @ConditionalOnClass({Servlet.class, StandardServletMultipartResolver.class, MultipartConfigElement.class})
> @ConditionalOnProperty(
>     prefix = "spring.servlet.multipart",
>     name = {"enabled"},
>     matchIfMissing = true
> )
> @ConditionalOnWebApplication(
>     type = Type.SERVLET
> )
> @EnableConfigurationProperties({MultipartProperties.class})
> public class MultipartAutoConfiguration {}
> ```
>
> ***@ConditionalOnProperty***：规定前缀 ***spring.servlet.multipart***
>
> ***@EnableConfigurationProperties***：开启 ***MultipartProperties*** 配置
>
> ```java
> public class MultipartProperties {    
> 		private boolean enabled = true;
>     private String location;
>     private DataSize maxFileSize = DataSize.ofMegabytes(1L);
>     private DataSize maxRequestSize = DataSize.ofMegabytes(10L);
>     private DataSize fileSizeThreshold = DataSize.ofBytes(0L);
>     private boolean resolveLazily = false;
> }
> ```





### MultipartFile

使用 ***MultipartFile*** 自动封住上传的文件，当文件有多组，则使用 ***MultipartFile[]***。

1. 上传的表单

```html
<form method="post" action="/upload" enctype="multipart/form-data">
    <input type="file" name="file"><br>
    <input type="submit" value="提交">
</form>
```

2. 文件上传 ***Controller***

```java
    /**
     * MultipartFile 自动封装上传过来的文件
     * @param email
     * @param username
     * @param headerImg
     * @param photos
     * @return
     */
    @PostMapping("/upload")
    public String upload(@RequestParam("email") String email,
                         @RequestParam("username") String username,
                         @RequestPart("headerImg") MultipartFile headerImg,
                         @RequestPart("photos") MultipartFile[] photos) throws IOException {

        log.info("上传的信息：email={}，username={}，headerImg={}，photos={}",
                email,username,headerImg.getSize(),photos.length);

        if(!headerImg.isEmpty()){
            // 1.保存到文件服务器 - OSS服务器
          	// ...
          	// 2.存储到本地
            String originalFilename = headerImg.getOriginalFilename(); // 获取原始文件名
            headerImg.transferTo(new File("存储的路径" + originalFilename));
        }

        if(photos.length > 0){
            for (MultipartFile photo : photos) {
                if(!photo.isEmpty()){
                    String originalFilename = photo.getOriginalFilename();
                    photo.transferTo(new File("存储的路径" + originalFilename));
                }
            }
        }

        return "main";
    }
```

3. 配置文件中以前缀 ***spring.servlet.multipart*** 修改上传文件相关参数

```properties
spring.servlet.multipart.max-file-size=10MB #上传单个文件不超过
spring.servlet.multipart.max-request-size=100MB #上传整体上传不超过
```



### <u>文件上传原理</u>

p51





## 错误处理









# 数据访问



# 感谢

- [springboot2-雷神](https://www.bilibili.com/video/BV19K4y1L7MT?p=1)

