# 🎄 MybatisPlus入门 - 概念与基础

---

> [!TIP]
>
> 在之前的文章中，我们学习了最常用的ORM框架——Mybatis，但是我们知道Mybatis仍然存在一些缺点，例如：
>
> - 编写SQL语句时工作量很大，尤其是字段多、关联表多时，更是如此。
> - SQL语句依赖于数据库，导致数据库移植性差，不能更换数据库。
> - 框架还是比较简陋，功能尚有缺失，虽然简化了数据绑定代码，但是整个底层数据库查询实际还是要自己写的，工作量也比较大，而且不太容易适应快速数据库修改。
> - 二级缓存机制不佳
>
> 为了尽量解决以上问题，我们开始学习MybatisPlus，进一步简化数据库操作！@vchicken

## 什么是MybatisPlus？

### 概念和特性

`MyBatis-Plus` 是一个 **MyBatis** 的增强工具，在 **MyBatis** 的基础上只做增强不做改变，为简化开发、提高 效率而生。

> **官网地址：https://mp.baomidou.com**。

**特性**

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑；

- **损耗小**：启动即会自动注入基本 `CURD`，性能基本无损耗，直接面向对象操作；

- **强大的 `CRUD` 操作**：内置通用 `Mapper`、通用 `Service`，仅仅通过少量配置即可实现单表大部分 CRUD 操作， 更有强大的条件构造器，满足各类使用需求；

- **支持`Lambda`形式调用**：通过 `Lambda` 表达式，方便的编写各类查询条件，无需再担心字段写错；

- **支持多种数据库**：支持 `MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer2005、SQLServer` 等多种数据库；

- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - `Sequence`），可自由配置，完美解决主键问题；

- **支持 `XML` 热加载**：**Mapper** 对应的 `XML` 支持热加载，对于简单的 `CRUD` 操作，甚至可以无 `XML` 启动；

- **支持 `ActiveRecord` 模式**：支持 `ActiveRecord` 形式调用，实体类只需继承 `Model` 类即可进行强大的 `CRUD` 操作；

- **支持自定义全局通用操作**：支持全局通用方法注入（ `Write once，use anywhere`）；

- **支持关键词自动转义**：支持数据库关键词（`order、key......`）自动转义，还可自定义关键词；

- **内置代码生成器**：采用代码或者 `Maven` 插件可快速生成 `Mapper 、 Model 、 Service 、 Controller` 层代码， 支持模板引擎，更有超多自定义配置等您来使用；

- **内置分页插件**：基于 `MyBatis` 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 `List` 查询；

- **内置性能分析插件**：可输出 `Sql` 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询；

- **内置全局拦截插件**：提供全表 `delete 、 update` 操作智能分析阻断，也可自定义拦截规则，预防误操作；

- **内置 `Sql` 注入剥离器**：支持 `Sql` 注入剥离，有效预防 `Sql` 注入攻击。



### 支持数据库

> 任何能使用 `MyBatis` 进行 CRUD, 并且支持标准 SQL 的数据库，具体支持情况如下，如果不在下列表查看分页部分教程 PR 您的支持。

- MySQL，Oracle，DB2，H2，HSQL，SQLite，PostgreSQL，SQLServer，Phoenix，Gauss ，ClickHouse，Sybase，OceanBase，Firebird，Cubrid，Goldilocks，csiidb
- 达梦数据库，虚谷数据库，人大金仓数据库，南大通用(华库)数据库，南大通用数据库，神通数据库，瀚高数据库



### 框架结构

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-29/453d9719-6b7a-4e4c-8d51-1dec14c8ab2e_mybatisplus架构.png)



## 参考文章

[MyBatis-Plus](https://baomidou.com/)