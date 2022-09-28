# Mybatis-Plus

***MyBatis-Plus*** 是一个 ***MyBatis*** 的增强工具，在 ***MyBatis*** 的基础上只做增强不做改变，为简化开发、提高 效率而生。

***官网：https://mp.baomidou.com/***。

**特性**

> **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑；
>
> **损耗小**：启动即会自动注入基本 ***CURD***，性能基本无损耗，直接面向对象操作；
>
> **强大的 *CRUD* 操作**：内置通用 ***Mapper***、通用 ***Service***，仅仅通过少量配置即可实现单表大部分 CRUD 操作， 更有强大的条件构造器，满足各类使用需求；
>
> **支持 *Lambda* 形式调用**：通过 ***Lambda*** 表达式，方便的编写各类查询条件，无需再担心字段写错；
>
> **支持多种数据库**：支持 ***MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer2005、SQLServer*** 等多种数据库；
>
> **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - ***Sequence***），可自由配置，完美解决主键问题；
>
> **支持 *XML* 热加载**：***Mapper*** 对应的 ***XML*** 支持热加载，对于简单的 ***CRUD*** 操作，甚至可以无 ***XML*** 启动；
>
> **支持 *ActiveRecord* 模式**：支持 ***ActiveRecord*** 形式调用，实体类只需继承 ***Model*** 类即可进行强大的 ***CRUD*** 操作；
>
> **支持自定义全局通用操作**：支持全局通用方法注入（ ***Write once，use anywhere*** ）；
>
> **支持关键词自动转义**：支持数据库关键词（***order、key......***）自动转义，还可自定义关键词；
>
> **内置代码生成器**：采用代码或者 ***Maven*** 插件可快速生成 ***Mapper 、 Model 、 Service 、 Controller*** 层代码， 支持模板引擎，更有超多自定义配置等您来使用；
>
> **内置分页插件**：基于 ***MyBatis*** 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 ***List*** 查询；
>
> **内置性能分析插件**：可输出 ***Sql*** 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询；
>
> **内置全局拦截插件**：提供全表 ***delete 、 update*** 操作智能分析阻断，也可自定义拦截规则，预防误操作；
>
> **内置 *Sql* 注入剥离器**：支持 ***Sql*** 注入剥离，有效预防 ***Sql*** 注入攻击。

**架构**

![](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-20%2016.38.01.png)



## 快速入门

1. 开发环境

（ ***SpingBoot + mybatisplus*** ）

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-20%2013.29.28.png" alt="" style="zoom:50%;" />

依赖引入

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.1.tmp</version>
</dependency>
```

2. ***application.yml*** 配置数据库源

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db?useUnicode=true&characterEncoding=UTF-8
    username: root
    password: root
    
mybatis-plus:
  configuration:
  	# 日志打印sql语句
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

3. 创建实体类

```java
@Data
@NoArgsConstructor
public class User {
    private Integer id;
    private String name;
    private Integer age;

    public User(Integer id, String name, Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
}
```

4. 创建 ***mapper*** 接口

```java
public interface UserMapper extends BaseMapper<User> {
}
```

5. 测试

```java
@SpringBootTest
class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    void test() {
        userMapper.selectList(null).forEach(System.out::println);
    }
}
```



# 常用注解

## @TableName

> 映射数据库的表名



```java
@Data
@TableName(value = "user")
public class Account {
    private Integer id;
    private String name;
    private Integer age;
}
```



## @TableId

> 设置主键映射。
>
> - ***value*** 映射主键字段名
>
> - ***type*** 设置主键类型，即主键的生成策略
>
>   ***IdType***
>
>   <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-20%2014.30.14.png" alt="" style="zoom:50%;" />
>
>   ```
>   IdType.
>   	AUTO(0)：数据库自增，开发者无需给主键赋值
>   	NONE(1)：MP设置主键，雪花算法
>   	INPUT(2)：需要开发者手动赋值，即传入主键的值
>   	ASSIGN_ID(3)：MP分配ID（Long、Integer、String），雪花算法
>   	ASSIGN_UUID(4),MP分配UUID（String）,主键必须是String，自动生成UUID进行赋值
>   ```



## @TableField

> 映射非主键字段。
>
> - ***value*** 映射字段名
>
> - ***exist*** 表示是否为数据库字段 
>
>   如果实体类中的成员变量在数据库中没有对应的字段，则可以使用 ***@TableField(exist=false)***
>
> - ***select*** 表示是否查询该字段
>
> - ***fill*** 表示是否自动填充
>
>   将对象存入数据库的时候，由 ***MP*** 自动给某些字段赋值
>
>   1. 数据库表中添加 ***create_time、update_time***
>
>   2. 实体类添加 ***create_time、update_time***
>
>   ```java
>   @Data
>   public class User {
>       @TableId
>       private String id;
>       @TableField(value = "name",select = false) // 不查询该字段
>       private String title;
>       private Integer age;
>       @TableField(exist = false) // 该字段在数据库中不存在
>       private String gender;
>       
>     	// create_time
>       @TableField(fill = FieldFill.INSERT)
>       private Date createTime;
>     	// update_time
>       @TableField(fill = FieldFill.INSERT_UPDATE)
>       private Date updateTime;
>   }
>   ```
>
>   3. 创建自动填充处理器
>
>   ***MetaObjectHandler***
>
>   ```java
>   // com.tsuirka.handler.MyMetaObjectHandler
>   @Component
>   public class MyMetaObjectHandler implements MetaObjectHandler {
>     	// 插入字段时
>       @Override
>       public void insertFill(MetaObject metaObject) {
>           this.setFieldValByName("createTime", new Date(), metaObject);
>           this.setFieldValByName("updateTime", new Date(), metaObject);
>       } 
>     
>     	// 修改字段时
>       @Override
>       public void updateFill(MetaObject metaObject) {
>           this.setFieldValByName("updateTime", new Date(), metaObject);
>       }
>   }
>   ```





## @Version

> 标记乐观锁，通过 ***version*** 字段来保证数据的安全性，当修改数据的时候，会以 ***version*** 作为条件，当条件成立的时候才会修改成功。
>
> 
>
> 1. 数据库表添加 ***version*** 字段，默认值为 1
> 2. 实体类添加 ***version*** 成员变量，并且添加注解 ***@Version*** 
>
> ```java
> @Data
> @TableName(value = "user")
> public class User {
>     @TableId
>     private String id;
>     @TableField(value = "name",select = false)
>     private String title;
>     private Integer age;
>     @TableField(exist = false)
>     private String gender;
>     @TableField(fill = FieldFill.INSERT)
>     private Date createTime;
>     @TableField(fill = FieldFill.INSERT_UPDATE)
>     private Date updateTime;
>     @Version // 乐观锁
>     private Integer version;
> }
> ```
>
> 3. 注册配置类
>
> ```java
> // com.tsuiraku.config.MyBatisPlusConfig
> @Configuration
> public class MyBatisPlusConfig {
>     
>     @Bean
>     public OptimisticLockerInterceptor optimisticLockerInterceptor(){
>         return new OptimisticLockerInterceptor();
>     }
> }
> ```



## @EnumValue

> 通用枚举类注解，将数据库字段映射成实体类的枚举类型成员变量。
>
> 1. 注解方式实现枚举
>
> ```java
> // com.tsuiraku.mybatisplus.enums
> 
> public enum StatusEnum {
>     WORK(1,"上班"),
>     REST(0,"休息");
> 
>     StatusEnum(Integer code, String msg) {
>         this.code = code;
>         this.msg = msg;
>     }
> 
>     @EnumValue
>     private Integer code;
>     private String msg;
> }
> ```
>
> ```java
> @Data
> @TableName(value = "user")
> public class User {
>     @TableId
>     private String id;
>     @TableField(value = "name",select = false)
>     private String title;
>     private Integer age;
>     @TableField(exist = false)
>     private String gender;
>     @TableField(fill = FieldFill.INSERT)
>     private Date createTime;
>     @TableField(fill = FieldFill.INSERT_UPDATE)
>     private Date updateTime;
>     @Version
>     private Integer version;
>   	// 枚举类型
>     private StatusEnum status;
> }
> ```
>
> ```yml
> # application.yml
> mybatis-plus:
> 	type-enums-package: 
>   	com.tsuiraku.mybatisplus.enums
> ```
>
> 2. 接口方式实现枚举
>
> ```java
> public enum AgeEnum implements IEnum<Integer> {
>     ONE(1, "一岁"),
>     TWO(2, "两岁"),
>     THREE(3, "三岁");
> 
>     private Integer code;
>     private String msg;
> 
>     AgeEnum(Integer code, String msg) {
>         this.code = code;
>         this.msg = msg;
>     }
> 
>     @Override
>     public Integer getValue() {
>         return this.code;
>     }
> }
> ```



## @TableLogic

> 映射逻辑删除。
>
> 1. 数据表添加 ***deleted*** 字段
> 2. 实体类添加注解
>
> ```java
> @Data
> @TableName(value = "user")
> public class User {
>     @TableId
>     private String id;
>     @TableField(value = "name",select = false)
>     private String title;
>     private AgeEnum age;
>     @TableField(exist = false)
>     private String gender;
>     @TableField(fill = FieldFill.INSERT)
>     private Date createTime;
>     @TableField(fill = FieldFill.INSERT_UPDATE)
>     private Date updateTime;
>     @Version
>     private Integer version;
>     @TableField(value = "status")
>     private StatusEnum statusEnum;
>   
>   	// 逻辑删除
>     @TableLogic
>     private Integer deleted;
> }
> ```
>
> 3. ***application.yml*** 配置
>
> ```yml
> mybatis-plus:
> 	global-config:
>   	db-config:
>     	logic-not-delete-value: 0
>     	logic-delete-value: 1
> ```



# CRUD接口

通过继承 ***BaseMapper*** 就可以获取到各种各样的单表操作。***（ <u>Mapper CRUD 接口</u> ）***

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-20%2014.19.47.png" style="zoom:50%;" />



## Insert

> 插入。



```java
// 插入一条记录
int insert(T entity);
```



**参数说明**

| 类型 | 参数名 |   描述   |
| :--: | :----: | :------: |
|  T   | entity | 实体对象 |



## Delete

> 删除。



```java
// 根据 entity 条件，删除记录
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
// 删除（根据ID 批量删除）
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 ID 删除
int deleteById(Serializable id);
// 根据 columnMap 条件，删除记录
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
```



**参数说明**

|                类型                |  参数名   |                描述                |
| :--------------------------------: | :-------: | :--------------------------------: |
|            Wrapper\<T>             |  wrapper  | 实体对象封装操作类（可以为 null）  |
| Collection<? extends Serializable> |  idList   | 主键ID列表(不能为 null 以及 empty) |
|            Serializable            |    id     |               主键ID               |
|        Map<String, Object>         | columnMap |          表字段 map 对象           |

### 

## Update

> 更新。



```java
// 根据 whereWrapper 条件，更新记录
int update(@Param(Constants.ENTITY) T updateEntity, @Param(Constants.WRAPPER) Wrapper<T> whereWrapper);
// 根据 ID 修改
int updateById(@Param(Constants.ENTITY) T entity);
```



**参数说明**

|    类型     |    参数名     |                             描述                             |
| :---------: | :-----------: | :----------------------------------------------------------: |
|      T      |    entity     |               实体对象 (set 条件值,可为 null)                |
| Wrapper\<T> | updateWrapper | 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句） |



## Select

> 查询。



```java
// 根据 ID 查询
T selectById(Serializable id);
// 根据 entity 条件，查询一条记录
T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 查询（根据ID 批量查询）
List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 entity 条件，查询全部记录
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 查询（根据 columnMap 条件）
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
// 根据 Wrapper 条件，查询全部记录
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录。注意： 只返回第一个字段的值
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 根据 entity 条件，查询全部记录（并翻页）
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录（并翻页）
IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询总记录数
Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```



 **参数说明**

|                类型                |    参数名    |                   描述                   |
| :--------------------------------: | :----------: | :--------------------------------------: |
|            Serializable            |      id      |                  主键ID                  |
|            Wrapper\<T>             | queryWrapper |    实体对象封装操作类（可以为 null）     |
| Collection<? extends Serializable> |    idList    |    主键ID列表(不能为 null 以及 empty)    |
|        Map<String, Object>         |  columnMap   |             表字段 map 对象              |
|             IPage\<T>              |     page     | 分页查询条件（可以为 RowBounds.DEFAULT） |



### 分页查询

1. 注册配置类拦截器 ***PaginationInterceptor***

```java
// com.tsuiraku.config.MyBatisPlusConfig
@Configuration
public class MyBatisPlusConfig {
    
    @Bean
    public PaginationInterceptor paginationInterceptor(){
        return new PaginationInterceptor();
    }
}
```

2. 使用

```java
/**
* 
* Page<>(param1, parma2)
* 参数1：页数
* 参数2：每页记录数
*/
Page<User> page = new Page<>(1, 2);
Page<User> res = mapper.selectPage(page, null);
res.getSize();	// 每页记录数
res.getTotal();	// 总记录数
res.getRecords(); // 结果

Page<Map<String, Object>> page = new Page<>(1, 2);
Page<User> res = mapper.selectMapsPage(page, null);
```



## 自定义SQL（多表关联查询）

***User***

| id   | name     | age  |
| ---- | -------- | ---- |
| 1    | tsuiraku | 21   |
| 2    | nanase   | 26   |

***Grade***

| id   | u_id | score |
| ---- | ---- | ----- |
| 10   | 1    | 100   |
| 20   | 2    | 99    |

***Vo***

```java
@Data
public class Vo {
  private String name;
  private String score;
}
```

***Mapper***

```java
public interface UserMapper extends	BaseMapper<User> {
  @Select("select u.name,g.score from user u,grade g where u.id = g.u_id and u.id = 1")
  List<Vo> listVo(Integer id);
}
```



# 代码生成器

> ***AutoGenerator*** 是 ***MyBatis-Plus*** 的代码生成器，通过 ***AutoGenerator*** 可以快速生成 ***Entity、Mapper、Mapper、XML、Service、Controller*** 等各个模块的代码，极大的提升了开发效率。



1. 依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.3.1.tmp</version>
</dependency>

<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity</artifactId>
    <version>1.7</version>
</dependency>
```

2. 代码生成器模版解析（历史版本）

```java
public class Main {
  public static void main(String[] args) {
    // 创建generator对象
    AutoGenerator autoGenerator = new AutoGenerator();
    // 设置数据源
    DataSourceConfig dataSourceConfig = new DataSourceConfig();
    dataSourceConfig.setDbType(DbType.MYSQL);
    dataSourceConfig.setUrl("jdbc:mysql://localhost:3306/database");
    dataSourceConfig.setUsername("root");
    dataSourceConfig.setPassword("root");
    dataSourceConfig.setDriverName("com.mysql.cj.jdbc.Driver");
    autoGenerator.setDataSource(dataSourceConfig);
    // 设置全局配置
    GlobalConfig globalConfig = new GlobalConfig();
    globalConfig.setOutputDir(System.getProperty("user.dir")+"/src/main/java"); // 输出路径
    globalConfig.setOpen(false); // 创建完毕后是否自动打开
    globalConfig.setAuthor("tsuiraku"); // 作者
    globalConfig.setServiceName("%sService") // 生成Service而不是iService
    autoGenerator.setGlobalConfig(globalConfig);
    // 设置包信息
    PackageConfig packageConfig = new PackageConfig();
    packageConfig.setParent("com.tsuiraku"); // 父包
    packageConfig.setModuleName("generator"); // 设置生成包的名字
    packageConfig.setController("controller");
    packageConfig.setService("service"); 
    packageConfig.setServiceImpl("service.impl");
    packageConfig.setMapper("mapper");
    packageConfig.setEntity("entity");
    autoGenerator.setPackageInfo(packageConfig);
    // 设置配置策略
    StrategyConfig strategyConfig = new StrategyConfig();
    strategyConfig.setEntityLombokModel(true); // lombok
    strategyConfig.setNaming(NamingStrategy.underline_to_camel); // 驼峰命名
    strategyConfig.setColumnNaming(NamingStrategy.underline_to_camel); // 驼峰命名
    autoGenerator.setStrategy(strategyConfig);
    autoGenerator.execute();
  }
}
```



# 感谢

- [Java-楠哥](https://www.bilibili.com/video/BV1yA411t782?p=1)

