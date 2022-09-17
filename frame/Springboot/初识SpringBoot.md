# **🐾 初识Springboot**

------

# 1.Spring能做什么

## 1.1、Spring的能力

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-16/e294c8cd-6db1-4956-8935-29d2cc3b60c0_image1.png)



## 1.2、Spring的生态

https://spring.io/projects/spring-boot

覆盖了：

- web开发

- 数据访问

- 安全控制

- 分布式

- 消息服务

- 移动开发

- 批处理

- ......

## 1.3、Spring5重大升级

### 1.3.1、响应式编程

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-16/043fcc17-12fc-4cc5-89b9-e6ee4c59633e_image2.png)

### 1.3.2、内部源码设计

基于Java8的一些新特性，如：接口默认实现。重新设计源码架构。



# 2.Spring的缺点

既然spring已经足够强大，那么为什么又会诞生Springboot呢？我们要回答这个问题，首先要先思考下面的问题：

## Spring解决了什么问题，还有什么问题没解决？

- **Spring解决了什么问题？**

Spring是Java企业版（Java Enterprise Edition，JEE，也称J2EE）的轻量级代替品。无需开发重量级的EnterpriseJavaBean（EJB），Spring为企业级Java开发提供了一种相对简单的方法，通过依赖注入和面向切面编程，用简单的Java对象（Plain Old Java Object，POJO）实现了EJB的功能。

1. 使用Spring的IOC容器,将对象之间的依赖关系交给Spring,降低组件之间的耦合性,让我们更专注于应用逻辑 

2. 可以提供众多服务,事务管理,WS等。 

3. AOP的很好支持,方便面向切面编程。 

4. 对主流的框架提供了很好的集成支持,如Hibernate,Struts2,JPA等

5. Spring DI机制降低了业务对象替换的复杂性。 

6. Spring属于低侵入,代码污染极低。 

7. Spring的高度可开放性,并不强制依赖于Spring,开发者可以自由选择Spring部分或全部

- **Spring还有什么问题没解决？**

1. 虽然Spring的组件代码是**轻量级**的，但它的配置却是**重量级**的。一开始，Spring用XML配置，而且是很多XML配置。Spring 2.5引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。Spring 3.0引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。所有这些配置都代表了开发时的损耗。因为在思考Spring特性配置和解决业务问题之间需要进行思维切换，所以编写配置挤占了编写应用程序逻辑的时间。和所有框架一样，Spring实用，但与此同时它要求的回报也不少。

2. 除此之外，项目的依赖管理也是一件耗时耗力的事情。在环境搭建时，需要分析要导入哪些库的坐标，而且还需要分析导入与之有依赖关系的其他库的坐标，一旦选错了依赖的版本，随之而来的不兼容问题就会严重阻碍项目的开发进度。
3. jsp中要写很多代码、控制器过于灵活,缺少一个公用控制器 
4. Spring不支持分布式,这也是EJB仍然在用的原因之一。



# 3.SringBoot的概述

## SpringBoot解决上述Spring的缺点

SpringBoot对上述Spring的缺点进行的改善和优化，基于**约定优于配置**的思想，可以让开发人员不必在配置与逻辑业务之间进行思维的切换，全身心的投入到逻辑业务的代码编写中，从而大大提高了开发的效率，一定程度上缩短了项目周期。



## SpringBoot的特点

1. 为基于Spring的开发提供更快的入门体验
2. 开箱即用，没有代码生成，也无需XML配置。同时也可以修改默认值来满足特定的需求
3. 提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等

SpringBoot不是对Spring功能上的增强，而是提供了一种快速使用Spring的方式



## SpringBoot的核心功能

- **起步依赖** 起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。简单的说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的功能。

- **自动配置** Spring Boot的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程，考虑了众多因素，才决定Spring配置应该用哪个，不该用哪个。该过程是Spring自动完成的。



## SpringBoot优点

- Create stand-alone Spring applications 创建独立Spring应用
- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files) 内嵌web服务器
- Provide opinionated ‘starter’ dependencies to simplify your build configuration 自动starter依赖，简化构建配置
- Automatically configure Spring and 3rd party libraries whenever possible 自动配置Spring以及第三方功能
- Provide production-ready features such as metrics, health checks, and externalized configuration 提供生产级别的监控、健康检查及外部化配置
- Absolutely no code generation and no requirement for XML configuration 无代码生成、无需编写XML
- SpringBoot是整合Spring技术栈的一站式框架
- SpringBoot是简化Spring技术栈的快速开发脚手架



## SpringBoot缺点

- 人称版本帝，迭代快，需要时刻关注变化
- 封装太深，内部原理复杂，不容易精通