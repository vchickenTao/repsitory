# Utils 和 Tools

## Lombok

- 依赖

```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
</dependency>
```

- 自动生成 get/set 方法
  - @Data
- 自动生成无参/有参构造器
  - @NoArgsConstructor
  - @AllArgsConstructor
- 自动生成 toString 方法
  - @ToString
- 日志
  - @Slf4j



## Dev-tools

热更新工具

- 依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
</dependency>
```



## EasyExcel

- 依赖

```xml
<!-- 需要POI的依赖 -->

```

### 写操作



### 读操作







# SpringBoot整合框架

## Junit

- 依赖 

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

- 测试注解
  - `@RunWith(SpringRunner.class)`
  - `@SpringBootTest(classes = 启动类.class)` 



## Redis

- 依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
  <artifactID>spring-boot-starter-data-redis</artifactID>
</dependency>
```

③ 配置redis相关属性

- 本机的redis可以不需要配置
- `application.yml`

```yml
spring:
	redis:
		host: 127.0.0.1 
		port: 6379
```

④ 注入RedisTemplate模板

```java
private RedisTemplate redisTemplate
  
redisTemplate.boundValueOps("name").set("tsuiraku")
redisTemplate.boundValueOps("name").get()
```





创建redis配置类

1. 引入依赖
2. 创建redis缓存配置类，配置插件

3. SpringBoot缓存注解





## Mybatis

- 依赖

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <!-- <scope>runtime</scope> -->
</dependency>
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>2.2.0</version>
</dependency>
```

③ 配置 DataSource 和 MyBatis 相关配置

```yml
# datasource
spring:
	datasource:
	url: jdbc:mysql:///springboot?serverTimezone-UTC
	username: root
	password: root
	driver-class-name: com.mysql.cj.jdbc.Driver
	
# mybatis
mybatis:	
	mapper-location: classpath:mapper/*Mapper.xml # mapper映射文件路径
	#config-location: # 指定mybatis的核心配置文件
	type-aliases-package: # 返回自定义的类
```



## Mybatis-plus

- 依赖

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>
```

### 代码生成器

- 引入依赖

```xml
<!-- velocity -->
<dependency>
  <groupId>org.apache.velocity</groupId>
  <artifactId>velocity-engine-core</artifactId>
</dependency>
```

- 代码生成器一般不参与编译，放在test下

```java
public class CodeGenerator {

    @Test
    public void run() {

        // 创建代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();

        gc.setOutputDir("/Users/nanase/Documents/Gitee/Java-Web/edu/service/service_edu" + "/src/main/java");

        gc.setAuthor("tsuiraku");
        gc.setOpen(false); // 生成后是否打开资源管理器
        gc.setFileOverride(false); // 重新生成时文件是否覆盖

        // UserServie
        gc.setServiceName("%sService");	// 去掉Service接口的首字母I

        gc.setIdType(IdType.ID_WORKER_STR); // 主键策略与数据库中主键对应
        gc.setDateType(DateType.ONLY_DATE); // 定义生成的实体类中日期类型
        gc.setSwagger2(true); // 开启Swagger2

        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/guli_project?serverTimezone=GMT%2B8");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("tokyo302");
        dsc.setDbType(DbType.MYSQL);
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName("eduservice"); //模块名
        // com.tsuiraku.eduservice
        pc.setParent("com.tsuiraku");
        // com.atguigu.tsuiraku.controller
        pc.setController("controller");
        pc.setEntity("entity");
        pc.setService("service");
        pc.setMapper("mapper");
        mpg.setPackageInfo(pc);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();

        // 数据库对应表名
        strategy.setInclude("edu_teacher");

        strategy.setNaming(NamingStrategy.underline_to_camel); // 数据库表映射到实体的命名策略
        strategy.setTablePrefix(pc.getModuleName() + "_");  // 生成实体时去掉表前缀

        strategy.setColumnNaming(NamingStrategy.underline_to_camel);// 数据库表字段映射到实体的命名策略
        strategy.setEntityLombokModel(true); // lombok 模型 @Accessors(chain = true) setter链式操作

        strategy.setRestControllerStyle(true); // restful api风格控制器
        strategy.setControllerMappingHyphenStyle(true); // url中驼峰转连字符

        mpg.setStrategy(strategy);


        // 执行
        mpg.execute();
    }
}

```



### 分页

- Config配置分页插件

```java
@Configuration
@MapperScan("com.tsuiraku.eduservice.mapper")
public class EduServiceConfig {
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

- Controller调用Service



### 自动填充

- 自动填充的类

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        // 属性为实体类的字段名称
        this.setFieldValByName("gmtCreate", new Date(), metaObject);
        this.setFieldValByName("gmtModified", new Date(), metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("gmtModified", new Date(), metaObject);
    }
}
```

- 实体类中对应字段添加注解
  - @TableField(fill = FieldFill.INSERT)

```java
@ApiModelProperty(value = "创建时间")
@TableField(fill = FieldFill.INSERT)
private Date gmtCreate;

@ApiModelProperty(value = "更新时间")
@TableField(fill = FieldFill.INSERT)
private Date gmtModified;
```



### 逻辑删除

- Config配置逻辑删除插件

```java
@Configuration
@MapperScan("com.tsuiraku.eduservice.mapper")
public class EduServiceConfig {
    @Bean
    public ISqlInjector sqlInjector() {
        return new LogicSqlInjector();
    }
}
```

- 实体类中对应字段添加注解
  - @TableLogic

```java
@ApiModelProperty(value = "逻辑删除 1（true）已删除， 0（false）未删除")
@TableLogic
private Integer isDeleted;
```

- Controller调用Service

## Swagger

- 依赖

```xml
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
</dependency>
```

```java
package com.tsuiraku.servicebase;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket webApiConfig(){
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("webApi")
                .apiInfo(webApiInfo())
                .select()
                .paths(Predicates.not(PathSelectors.regex("/admin/.*")))
                .paths(Predicates.not(PathSelectors.regex("/error.*")))
                .build();

    }

    private ApiInfo webApiInfo(){

        return new ApiInfoBuilder()
                .title("在线教育系统API文档")
                .description("本文档描述了在线教育系统微服务接口定义")
                .version("1.0")
                .contact(new Contact("java", "https://gitee.com/tsuiraku", "tsuiraku@126.com"))
                .build();
    }
}
```



# 自定义Utils

## 返回统一的数据格式

- 链式

```java
public interface ResultCode {
    public static Integer SUCCESS = 200; //成功

    public static Integer ERROR = 201; //失败
}

@Data
public class R {
    @ApiModelProperty(value = "返回码")
    private Integer code;

    @ApiModelProperty(value = "返回消息")
    private String msg;

    @ApiModelProperty(value = "返回数据")
    private Map<String, Object> data = new HashMap<String, Object>();

    public R () {};

    public static R success() {
        R r = new R();
        r.setCode(ResultCode.SUCCESS);
        return r;
    }

    public static R error() {
        R r = new R();
        r.setCode(ResultCode.ERROR);
        return r;
    }

    public R msg(String message){
        this.setMsg(message);
        return this;
    }

    public R code(Integer code){
        this.setCode(code);
        return this;
    }

    public R data(String key, Object value){
        this.data.put(key, value);
        return this;
    }

    public R data(Map<String, Object> map){
        this.setData(map);
        return this;
    }
}
```

## 统一异常处理

- 全局异常

```java
@ControllerAdvice
public class GlobalExceptionHandler {
		// 全局异常处理
    @ExceptionHandler(Exception.class)  // 指定出现什么异常执行这个方法
    @ResponseBody
    public R error(Exception e) {
        e.printStackTrace();
        return R.error().msg("执行了全局异常处理");
    }
}
```

- 特定异常

```java
@ControllerAdvice
public class GlobalExceptionHandler {
  	// 特定异常
    @ExceptionHandler(ArithmeticException.class) 
    @ResponseBody 
    public R error(ArithmeticException e) {
        e.printStackTrace();
        return R.error().msg("执行了ArithmeticException异常处理");
    }
}
```

- 自定义异常

  - 创建自定义异常类继承 RuntimeException

  ```java
  @Data
  @AllArgsConstructor // 有参构造
  @NoArgsConstructor // 无参构造
  public class MyException extends RuntimeException {
      private Integer code; // 状态码
      private String msg; // 异常信息
  }
  ```

  - 统一异常类

  ```java
  // 自定义异常
  @ExceptionHandler(MyException.class)
  @ResponseBody //为了返回数据
  public R error(MyException e) {
    log.error(e.getMessage());
    e.printStackTrace();
    return R.error().code(e.getCode()).msg(e.getMsg());
  }
  ```

  - 执行自定义异常

  ```java
  try {
    int i = 10 / 0;
  } catch(Exception e) {
    throw new MyException(20001,"执行自定义异常");
  }
  ```

## Properties注入常量Utils

- 需要实现接口 `InitializaingBean`

```java
@Component
public class ConstantPropertiesUtils implements InitializingBean {
    @Value("${aliyun.oss.file.endpoint}")
    private String endpoint;
    @Value("${aliyun.oss.file.keyid}")
    private String keyId;
    @Value("${aliyun.oss.file.keysecret}")
    private String keySecret;
    @Value("${aliyun.oss.file.bucketname}")
    private String bucketName;
    public static String END_POINT;
    public static String ACCESS_KEY_ID;
    public static String ACCESS_KEY_SECRET;
    public static String BUCKET_NAME;

    @Override
    public void afterPropertiesSet() throws Exception {
        END_POINT = endpoint;
        ACCESS_KEY_ID = keyId;
        ACCESS_KEY_SECRET = keySecret;
        BUCKET_NAME = bucketName;
    }
}

```



# 日志

日志记录器(Logger)的行为是分等级的。**OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL**
 默认情况下，spring boot从控制台打印出来的日志级别只有INFO及以上级别，可以手动配置日志级别。

```properties
logging.level.root=WARN
```

## LogBack

- 配置Logback日志

  - 删除 `application.yml` 中的日志配置
  - 安装idea彩色日志插件：grep-console（默认已经安装）
  - resources 中创建 logback-spring.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <configuration  scan="true" scanPeriod="10 seconds">
      <!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
      <!-- scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true -->
      <!-- scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。 -->
      <!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 -->
  
      <contextName>logback</contextName>
      <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。 -->
      <property name="log.path" value="/Users/nanase/Documents/Gitee/Java-Web/edu/edu_parent/service/service_edu/src/main/resources/log" />
  
      <!-- 彩色日志 -->
      <!-- 配置格式变量：CONSOLE_LOG_PATTERN 彩色日志格式 -->
      <!-- magenta:洋红 -->
      <!-- boldMagenta:粗红-->
      <!-- cyan:青色 -->
      <!-- white:白色 -->
      <!-- magenta:洋红 -->
      <property name="CONSOLE_LOG_PATTERN"
                value="%yellow(%date{yyyy-MM-dd HH:mm:ss}) |%highlight(%-5level) |%blue(%thread) |%blue(%file:%line) |%green(%logger) |%cyan(%msg%n)"/>
  
  
      <!--输出到控制台-->
      <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
          <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
          <!-- 例如：如果此处配置了INFO级别，则后面其他位置即使配置了DEBUG级别的日志，也不会被输出 -->
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>INFO</level>
          </filter>
          <encoder>
              <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
              <!-- 设置字符集 -->
              <charset>UTF-8</charset>
          </encoder>
      </appender>
  
  
      <!--输出到文件-->
  
      <!-- 时间滚动输出 level为 INFO 日志 -->
      <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <!-- 正在记录的日志文件的路径及文件名 -->
          <file>${log.path}/log_info.log</file>
          <!--日志文件输出格式-->
          <encoder>
              <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
              <charset>UTF-8</charset>
          </encoder>
          <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <!-- 每天日志归档路径以及格式 -->
              <fileNamePattern>${log.path}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
              <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                  <maxFileSize>100MB</maxFileSize>
              </timeBasedFileNamingAndTriggeringPolicy>
              <!--日志文件保留天数-->
              <maxHistory>15</maxHistory>
          </rollingPolicy>
          <!-- 此日志文件只记录info级别的 -->
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>INFO</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
      </appender>
  
      <!-- 时间滚动输出 level为 WARN 日志 -->
      <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <!-- 正在记录的日志文件的路径及文件名 -->
          <file>${log.path}/log_warn.log</file>
          <!--日志文件输出格式-->
          <encoder>
              <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
              <charset>UTF-8</charset> <!-- 此处设置字符集 -->
          </encoder>
          <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <fileNamePattern>${log.path}/warn/log-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
              <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                  <maxFileSize>100MB</maxFileSize>
              </timeBasedFileNamingAndTriggeringPolicy>
              <!--日志文件保留天数-->
              <maxHistory>15</maxHistory>
          </rollingPolicy>
          <!-- 此日志文件只记录warn级别的 -->
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>warn</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
      </appender>
  
  
      <!-- 时间滚动输出 level为 ERROR 日志 -->
      <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <!-- 正在记录的日志文件的路径及文件名 -->
          <file>${log.path}/log_error.log</file>
          <!--日志文件输出格式-->
          <encoder>
              <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
              <charset>UTF-8</charset> <!-- 此处设置字符集 -->
          </encoder>
          <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <fileNamePattern>${log.path}/error/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
              <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                  <maxFileSize>100MB</maxFileSize>
              </timeBasedFileNamingAndTriggeringPolicy>
              <!--日志文件保留天数-->
              <maxHistory>15</maxHistory>
          </rollingPolicy>
          <!-- 此日志文件只记录ERROR级别的 -->
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>ERROR</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
      </appender>
  
      <!--
          <logger>用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender>。
          <logger>仅有一个name属性，
          一个可选的level和一个可选的addtivity属性。
          name:用来指定受此logger约束的某一个包或者具体的某一个类。
          level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
                如果未设置此属性，那么当前logger将会继承上级的级别。
      -->
      <!--
          使用mybatis的时候，sql语句是debug下才会打印，而这里我们只配置了info，所以想要查看sql语句的话，有以下两种操作：
          第一种把<root level="INFO">改成<root level="DEBUG">这样就会打印sql，不过这样日志那边会出现很多其他消息
          第二种就是单独给mapper下目录配置DEBUG模式，代码如下，这样配置sql语句会打印，其他还是正常DEBUG级别：
       -->
      <!--开发环境:打印控制台-->
      <springProfile name="dev">
          <!--可以输出项目中的debug日志，包括mybatis的sql日志-->
          <logger name="com.guli" level="INFO" />
  
          <!--
              root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性
              level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，默认是DEBUG
              可以包含零个或多个appender元素。
          -->
          <root level="INFO">
              <appender-ref ref="CONSOLE" />
              <appender-ref ref="INFO_FILE" />
              <appender-ref ref="WARN_FILE" />
              <appender-ref ref="ERROR_FILE" />
          </root>
      </springProfile>
  
  
      <!--生产环境:输出到文件-->
      <springProfile name="pro">
  
          <root level="INFO">
              <appender-ref ref="CONSOLE" />
              <appender-ref ref="DEBUG_FILE" />
              <appender-ref ref="INFO_FILE" />
              <appender-ref ref="ERROR_FILE" />
              <appender-ref ref="WARN_FILE" />
          </root>
      </springProfile>
  
  </configuration>
  ```

- 将异常信息输出到文件

  - 全局异常类 + @Slf4j
  - 异常输出语句

  ```java
  log.error(e.getMessage());
  ```

  



# 云服务

## 阿里云

### OSS

- 依赖

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.10.2</version>
</dependency>
```

- 阅读：[官方文档-OSS](https://help.aliyun.com/document_detail/32007.html)

### VOD

- 依赖

```xml
<dependency>
  <groupId>com.aliyun</groupId>
  <artifactId>aliyun-java-sdk-core</artifactId>
  <version>4.5.1</version>
</dependency>
<dependency>
  <groupId>com.aliyun</groupId>
  <artifactId>aliyun-java-sdk-vod</artifactId>
  <version>2.15.11</version>
</dependency>
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.62</version>
</dependency>
<dependency>
  <groupId>com.aliyun</groupId>
  <artifactId>aliyun-java-sdk-kms</artifactId>
  <version>2.10.1</version>
</dependency>
```

- 阅读：[官方文档-JavaSDK](https://help.aliyun.com/document_detail/57723.html)





# Application.yml

## 统一返回的json时间格式

```yml
spring:
  # 时区配置
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
```



# 后台模版框架

## vue-admin-template

- vue + element-ui







# JWT

1. 依赖
2. 工具类





# 阿里云短信



