# Mybatis

> MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架；
>
> MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集；
>
> MyBatis可以使用简单的 XML 或注解用于配置和原始映射，将接口和 Java 的 POJO（Plain Old Java Objects，普通的 Java 对象）映射成数据库中的记录。



## Mybatis简单入门

**Mybatis操作数据库**

1. 创建 ***MyBatis*** **全局配置文件** ***mybatis-config.xml***，配置如数据库连接池信息等。

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
   	<environments default="development">
   		<environment id="development">
   			<transactionManager type="JDBC" />
   			<dataSource type="POOLED">
   				<property name="driver" value="com.mysql.jdbc.Driver" />
   				<property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
   				<property name="username" value="root" />
   				<property name="password" value="root" />
   			</dataSource>
   		</environment>
   	</environments>
   	<!-- 将 sql 映射文件（EmployeeMapper.xml）注册到全局配置文件（mybatis-config.xml）中 -->
   	<mappers>
   		<mapper resource="EmployeeMapper.xml" />
   	</mappers>
   </configuration>
   ```

2. 创建 ***SQL*** 映射文件 ***EmployeeMapper.xml*** 。

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.tsuiraku.mybatis.dao.EmployeeMapper">
   <!-- 
     namespace：名称空间，指定为接口的全类名
     id：唯一标识，推荐使用 namespace + id
     resultType：返回值类型
     #{id}：从传递过来的参数中取出id值
     public Employee getEmpById(Integer id);
    -->
   	<select id="getEmpById" resultType="com.tsuiraku.mybatis.bean.Employee">
   		select id,last_name lastName,email,gender from tbl_employee where id = #{id}
   	</select>
   </mapper>
   ```

   

**测试**

1. 根据全局配置文件，利用 ***SqlSessionFactoryBuilder*** 创建 ***SqlSessionFactory***。

   ```java
   public SqlSessionFactory getSqlSessionFactory() throws IOException {
   		String resource = "mybatis-config.xml";
   		InputStream inputStream = Resources.getResourceAsStream(resource);
   		return new SqlSessionFactoryBuilder().build(inputStream);
   	}
   ```

2. 使用 ***SqlSessionFactory*** 获取 ***sqlSession*** 对象。一个 ***SqlSession*** 对象代表和数据库的一次会话。

   ```java
   // SqlSession session = SqlSessionFactory.openSession();
   
   
   // 老版本
   
   /**
   	 * 1. 根据xml配置文件（全局配置文件）配置如数据源一些运行环境信息，创建一个SqlSessionFactory对象 
   	 * 2. sql映射文件，配置sql语句，以及sql的封装规则等
   	 * 3. 将sql映射文件注册在全局配置文件中
   	 * 4. 使用
   	 * 		1) 根据全局配置文件得到SqlSessionFactory
   	 * 		2) 使用SqlSessionFactory，获取到sqlSession对象使用他来执行增删改查
   	 * 			 一个sqlSession就是代表和数据库的一次会话，用完需要关闭
   	 * 		3) 使用sql的唯一标志来告诉MyBatis执行哪个sql，sql都是保存在sql映射文件中的。
   	 * 
   	 */
    @Test
    public void test() throws IOException {
     	// 使用 SqlSessionFactory 获取 sqlSession 对象
     	SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
   	  SqlSession session = sqlSessionFactory.openSession();
       try {
         // 执行 sql 映射文件中的 sql 语句
         Employee employee = openSession.selectOne(
           "com.tsuiraku.mybatis.dao.EmployeeMapper.getEmpById", 1);
         System.out.println(employee);
       } finally {
         // 关闭 session
         session.close();
       }
   }
   
   // #########################################################################################
   
   // 推荐使用接口式编程
   
   /**
    * 1. 接口式编程
    * 	原生：		Dao		====>  DaoImpl
    * 	mybatis：	Mapper	====>  xxxMapper.xml
    * 
    * 2. SqlSession代表和数据库的一次会话；用完必须关闭；
    * 3. SqlSession和connection一样都是非线程安全。每次使用都应该去获取新的对象。
    * 4. mapper接口没有实现类，但是mybatis会为这个接口生成一个代理对象。
    * 		（将接口和xml进行绑定）
    * 		EmployeeMapper empMapper =	sqlSession.getMapper(EmployeeMapper.class);
    * 5. 两个重要的配置文件：
    * 		mybatis的全局配置文件：包含数据库连接池信息，事务管理器信息等...系统运行环境信息
    * 		sql映射文件：保存了每一个sql语句的映射信息：
    * 					将sql抽取出来。	
    *
    */
   @Test
   public void test01() throws IOException {
   		// 使用 SqlSessionFactory 获取 sqlSession 对象
   		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
   		SqlSession session = sqlSessionFactory.openSession();
   		try {
   			// 会为接口自动的创建一个代理对象，代理对象去执行增删改查方法
   			EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
   			Employee employee = mapper.getEmpById(1);
   			System.out.println(mapper.getClass());
   			System.out.println(employee);
   		} finally {
   			session.close();
   		}
   }
   ```

3. 接口式编程

   ```java
   public interface EmployeeMapper {
   	
   	public Employee getEmpById(Integer id);
   }
   ```



# 全局配置文件

***MyBatis*** 的配置文件包含了影响 ***MyBatis*** 行为的设置和属性信息。

```
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```



## properties（属性）

引入外部 ***properties*** 配置文件。

> resource：引入类路径下的资源；
>
> url：引入网络路径或者磁盘路径下的资源。



```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.username=root
jdbc.password=root
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!--
	 mybatis可以使用properties来引入外部properties配置文件的内容
			resource：引入类路径下的资源
			url：引入网络路径或者磁盘路径下的资源
	  -->
	<properties resource="dbconfig.properties"></properties>
	
	 <environments default="dev_mysql">
		<environment id="dev_mysql">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
    </environment>
</configuration>
```



若属性在不只一个地方进行了配置，那么 ***MyBatis*** 将按照下面的顺序来加载

- 在 ***properties*** 元素体内指定的属性首先被读取；
- 然后根据 ***properties*** 元素中的 ***resource*** 属性读取类路径下属性文件或，根据 ***url*** 属性指定的路径读取属性文件，并覆盖已读取的同名属性；

- 最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。



## settings（设置）

包含很多重要的设置项，***MyBatis*** 中极为重要的调整设置，会改变 ***MyBatis*** 的运行时行为。

| 设置名                         | 描述                                                         | 有效值                                                       | 默认值                                       |
| :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | -------------------------------------------- |
| ***cacheEnabled***             | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true                                         |
| ***lazyLoadingEnabled***       | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false                                        |
| ***aggressiveLazyLoading***    | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true \| false                                                | false （在 3.4.1 及之前的版本中默认为 true） |
| ***mapUnderscoreToCamelCase*** | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false                                                | false                                        |
| ***logImpl***                  | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置                                       |



例如，开启驼峰命名法。

```xml
	
	<!-- 
	 settings包含很多重要的设置项
	 	setting：用来设置每一个设置项
			name：设置项名
			value：设置项取值
	 -->
	<settings>
		<setting name="mapUnderscoreToCamelCase" value="true"/>
	</settings>
```



## typeAliases（类型命名）

类型别名是为 ***Java*** 类型设置一个别名，可以方便引用某个类。存在的意义仅在于用来降低冗余的全限定类名书写。



```xml
	<!-- typeAliases：别名处理器，可以为java类型起别名 
				别名不区分大小写
	-->
	<typeAliases>
		<!-- 1. typeAlias:为某个java类型起别名
				 	type：指定要起别名的类型全类名；默认别名就是类名小写；employee；别名不区分大小写
				 	alias：指定新的别名
		 -->
		<typeAlias type="com.tsuiraku.mybatis.bean.Employee" alias="emp"/>
		
		<!-- 2. package：为某个包下的所有类批量起别名 
				 	name：指定包名（为当前包以及下面所有的后代包的每一个类都起一个默认别名（类名小写））
		-->
		<package name="com.tsuiraku.mybatis.bean"/>
		
		<!-- 3. 批量起别名的情况下，使用@Alias注解为某个类型指定新的别名 -->
	</typeAliases>
```



下面是一些为常见的 Java 类型内建的类型别名。它们都是不区分大小写的，

注意，为了应对原始类型的命名重复，采取了特殊的命名风格。

| 别名         | 映射的类型  |
| ------------ | ----------- |
| _byte        | byte        |
| _long        | long        |
| _short       | short       |
| **_int**     | **int**     |
| **_integer** | **int**     |
| _double      | double      |
| _float       | float       |
| _boolean     | boolean     |
| string       | String      |
| byte         | Byte        |
| long         | Long        |
| short        | Short       |
| **int**      | **Integer** |
| **integer**  | **Integer** |
| double       | Double      |
| float        | Float       |
| boolean      | Boolean     |
| date         | Date        |
| decimal      | BigDecimal  |
| bigdecimal   | BigDecimal  |
| object       | Object      |
| map          | Map         |
| hashmap      | HashMap     |
| list         | List        |
| arraylist    | ArrayList   |
| collection   | Collection  |
| iterator     | Iterator    |



### @Alias

该注解在批量起别名的情况下，为某个类型指定别名。



## typeHandlers（类型处理器）

在预处理语句（***PreparedStatement***）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 ***Java*** 类型。



## objectFactory（对象工厂）



## plugins（插件）

***<u>插件开发</u>***。



## environments（环境）

可以配置多种环境，开发、测试和生产模式。

每种环境使用一个 ***environment*** 标签进行配置并指定唯一标识符。

可以使用 ***environment*** 标签中的 ***default*** 属性指定一个环境的标识符来快速切换环境。



<u>**尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**</u>

```xml
<!--  environments：环境，mybatis可以配置多种环境，default指定使用某种环境。可以达到快速切换环境
				environment：配置一个具体的环境信息；必须有两个标签；id代表当前环境的唯一标识
				transactionManager：事务管理器
				type：事务管理器的类型；JDBC(JdbcTransactionFactory)|MANAGED(ManagedTransactionFactory)
						  自定义事务管理器：实现TransactionFactory接口，type指定为全类名
				dataSource：数据源
					type:数据源类型
							  ｜UNPOOLED(UnpooledDataSourceFactory)
								 |POOLED(PooledDataSourceFactory)
							  	|JNDI(JndiDataSourceFactory)
					自定义数据源：实现DataSourceFactory接口，type是全类名
		 -->
		 		 
	<environments default="dev_mysql">
		<environment id="dev_mysql">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	
		<environment id="dev_oracle">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${orcl.driver}" />
				<property name="url" value="${orcl.url}" />
				<property name="username" value="${orcl.username}" />
				<property name="password" value="${orcl.password}" />
			</dataSource>
		</environment>
	</environments>
	
```



## databaseldProvider(数据库厂商标识)

可以根据不同的数据库厂商执行不同的语句。



```xml
<!-- databaseIdProvider：支持多数据库厂商的
		 		type="DB_VENDOR"：VendorDatabaseIdProvider
		 		作用是得到数据库厂商的标识(驱动getDatabaseProductName())，就能根据数据库厂商标识来执					行不同的sql；MySQL，Oracle，SQL Server,xxxx
	  -->
	<databaseIdProvider type="DB_VENDOR">
		<!-- 为不同的数据库厂商起别名 -->
		<property name="MySQL" value="mysql"/>
		<property name="Oracle" value="oracle"/>
		<property name="SQL Server" value="sqlserver"/>
	</databaseIdProvider>
```





## mappers（映射器）

***sql*** 映射文件，将 ***sql*** 映射注册到全局配置中。



```xml
	<!-- sql映射文件（EmployeeMapper.xml）一定要注册到全局配置文件（mybatis-config.xml）中 -->
	<!-- mappers：将sql映射注册到全局配置中 -->
	<mappers>
		<!-- 
			mapper：注册一个sql映射 
				注册配置文件
				resource：引用类路径下的sql映射文件
					mybatis/mapper/UserMapper.xml
				url：引用网路路径或者磁盘路径下的sql映射文件
					file:///var/mappers/UserMapper.xml
					
				注册接口
				class：引用（注册）接口，
					1. 有sql映射文件，映射文件名必须和接口同名，并且放在与接口同一目录下；
					2. 没有sql映射文件，所有的sql都是利用注解写在接口上;
					推荐：
						比较重要的，复杂的Dao接口我们来写sql映射文件
						不重要，简单的Dao接口为了开发快速可以使用注解
		-->
		<mapper resource="mybatis/mapper/UserMapper.xml"/> 
		<mapper class="com.tsuiraku.mybatis.dao.UserMapperAnnotation"/> 
		
		<!-- 批量注册 -->
		<package name="com.tsuiraku.mybatis.dao"/>
	</mappers>
```



### class

基于注解版本的。

```java
public interface EmployeeMapper {
	@Select(...)
	public Employee getEmpById(Integer id);
}
```



# 映射文件

## Insert

### 主键生成方式

若数据库支持自动生成主键的字段，则可以设置 ***useGeneratedKeys="true"***，然后再把 ***keyProperty*** 设置到目标属性上。

```xml
	<!-- public void addUser(User user); -->
	<!-- parameterType：参数类型，可以省略， 
	获取自增主键的值：
		mysql支持自增主键，自增主键值的获取，mybatis也是利用statement.getGenreatedKeys()；
		useGeneratedKeys="true"；使用自增主键获取主键值策略
		keyProperty；指定对应的主键属性，也就是mybatis获取到主键值以后，将这个值封装给javaBean的哪个属性
	-->
	<insert id="addUser" parameterType="com.tsuiraku.mybatis.bean.User"
		useGeneratedKeys="true" keyProperty="id" databaseId="mysql">
		insert into user(name,email,gender) 
		values(#{name},#{email},#{gender})
	</insert>
```



### 非自增主键

对于不支持自增的数据库。

使用 ***\<selectKey>*** 标签。

```xml
	<!-- 
	获取非自增主键的值：
		Oracle不支持自增；Oracle使用序列来模拟自增；
		每次插入的数据的主键是从序列中拿到的值；如何获取到这个值；
	 -->
	<insert id="addUser" databaseId="oracle">
		<!-- 
		keyProperty：查出的主键值封装给javaBean的哪个属性
		order="BEFORE"：当前sql在插入sql之前运行
			    "AFTER"：当前sql在插入sql之后运行
		resultType:查出的数据的返回值类型
		
		BEFORE运行顺序：
			先运行selectKey查询id的sql；查出id值封装给javaBean的id属性
			在运行插入的sql；就可以取出id属性对应的值
		AFTER运行顺序：
			先运行插入的sql（从序列中取出新值作为id）；
			再运行selectKey查询id的sql；
		 -->
		<selectKey keyProperty="id" order="BEFORE" resultType="Integer">
			<!-- 编写查询主键的sql语句 -->
			<!-- BEFORE-->
			select EMPLOYEES_SEQ.nextval from dual 
			<!-- AFTER：
			 select EMPLOYEES_SEQ.currval from dual -->
		</selectKey>
		
		<!-- 插入时的主键是从序列中拿到的 -->
		<!-- BEFORE:-->
		insert into employees(EMPLOYEE_ID,NAME,EMAIL) 
		values(#{id},#{name},#{email<!-- ,jdbcType=NULL -->}) 
		<!-- AFTER：
		insert into employees(EMPLOYEE_ID,LAST_NAME,EMAIL) 
		values(employees_seq.nextval,#{lastName},#{email}) -->
	</insert>
```



## 参数处理

### 单个参数&多个参数&命名参数

> - 单个参数：***mybatis*** 不会做特殊处理，
>
>   ​	#{参数名/任意名}：取出参数值。
>
> - 多个参数：***mybatis*** 会做特殊处理。
>
>   ​	多个参数会被封装成 一个***map***，
>   ​    	***key***：***param1...paramN***，或者参数的索引也可以
>   ​    	***value***：传入的参数值
>      	**#{} ** 就是从 ***map*** 中获取指定的 ***key*** 的值；
>
> - 命名参数：明确指定封装参数时 ***map*** 的 ***key*** 。
>
>   ​	多个参数会被封装成 一个 ***map***
>      	 ***key***：使用 ***<u>@Param</u>*** 注解指定的值
>   ​    	***value***：参数值
>      #{指定的key}取出对应的参数值



### POJO&Map&To

> ***POJO***
>
> 如果多个参数正好是业务逻辑的数据模型，我们就可以直接传入 ***pojo***。
>    #{属性名}：取出传入的 ***pojo*** 的属性值   
>
> ***Map***
> 如果多个参数不是业务模型中的数据，没有对应的 ***pojo***，不经常使用，为了方便，我们也可以传入 ***map***。
>    #{key}：取出 ***map*** 中对应的值
>
> ***TO***
> 如果多个参数不是业务模型中的数据，但是经常要使用，推荐来编写一个TO（Transfer Object）数据传输对象。
>
> ex：	Page{
>    			int index;
>    			int size;
> 			}



### 参数封装扩展

> ```
> public User getUser(@Param("id")Integer id, String name);
>    取值：id==>#{id/param1}   name==>#{param2}
> 
> public User getUser(Integer id, @Param("e")User user);
>    取值：id==>#{param1}    name===>#{param2.name/e.name}
> 
> ##特别注意：如果是Collection（List、Set）类型或者是数组，
> 					也会特殊处理。也是把传入的list或者数组封装在map中。
>          key：Collection（collection)，如果是List还可以使用这个key(list)数组(array)
> public Employee getEmpById(List<Integer> ids);
>    取值：取出第一个id的值：   #{list[0]}
> ```



### 参数封装Map的过程

**源码分析**

################ $to\space do$ ################



### #与$的区别

> #{}：可以获取 ***map*** 中的值或者 ***pojo*** 对象属性的值；
> ${}：可以获取 ***map*** 中的值或者 ***pojo*** 对象属性的值；
>
> 区别：
>    #{}，是以预编译的形式，将参数设置到sql语句中；***PreparedStatement***；防止 ***sql*** 注入。
>    ${}，取出的值直接拼装在 ***sql*** 语句中；会有安全问题。
>
> 大多情况下，我们去参数的值都应该去使用#{}；
>
> 原生 ***jdbc*** 不支持占位符的地方我们就可以使用${}。



### #取值时指定参数相关规则

> 规定参数的一些规则
>    javaType、 jdbcType、 mode（存储过程）、 numericScale、
>    resultMap、 typeHandler、 jdbcTypeName、 expression（未来准备支持的功能）。



## Select

>  ***Select*** 元素来定义查询操作
>
> - ***id*** 
>
>   唯一标识符。用来引用这条语句，需要和接口的方法名一致 。
>
> - ***parameterType***
>
>   参数类型。可以不传，***MyBatis*** 会根据 ***TypeHandler*** 自动推断。
>
> - ***resultType***
>
>   返回值类型。别名或者全类名，如果返回的是集合，定义集合中元素的类型。不能和 ***resultMap*** 同时使用。



### resultType

#### 封装List

```java
public List<User> getUserById(String name);
/**
* 返回list
* User [id=1,lastName=tsuiraku]
* User [id=2,lastName=nanase]
*/
```

```xml
<select id="getUserById" resultType="com.tsuiraku.User">
	select * from xxxTable where name like #{name}
</select>
```



#### 封装Map

```java
public Map<String, Object> getSingleUserById(Integer id);
/**
* 返回一条记录
*	key：列名
* value：值
* {name=tsuiraku, age=21}
*/
```

```xml
<select id="getSingleUserById" resultType="map">
	select * from xxxTable where id=#{id}
</select>
```



```java
@MapKey("id") // 封装map使用id作为key
public Map<Integer, User> getMultipleUserById(Integer id);
/**
*	key：主键
* value：某个实体类对象
*/
```

```xml
<select id="getMultipleUserById" resultType="com.tsuiraku.User">
	select * from xxxTable where id=#{id}
</select>
```



### resultMap

外部 ***resultMap*** 的命名引用。和 ***resultType*** 属性不能同时使用。实现高级结果集映射。



```xml
<!-- 自定义JavaBean的封装规则
	type：自动规则的Java类型
	id：方便引用
-->
<resultMap type="com.tsuiraku.User" id="myUser">
  <!-- 指定主键列的封装规则 
		column：指定哪一列
		property：指定对应的JavaBean属性
	-->
	<id column="id" property="id"/>
  <!-- 指定普通列的封装规则 
		column：指定哪一列
		property：指定对应的JavaBean属性
	-->
  <result column="last_name" property="lastName"/>
</resultMap>

<select id="getUserById" resultMap="myUser">
  <!-- 
			数据库中字段为 last_name
			实体类中字段为 lastName
	-->
	select * from xxxTable where last_name like #{lastName}
</select>
```



#### 关联查询

使用 ***\<association>*** 定义封装对象。

```java
public class User{
  private Integer id;
  private String lastName;
  private Dept dept;
  
  //get/set
}

public class Dept{
  private Integer id;
  private String departmentName;
  
   //get/set
}
```

```xml
<!-- 第1种 -->
<resultMap type="com.tsuiraku.User" id="myUser">
	<id column="id" property="id"></id>
  <result column="last_name" property="lastName"/>
  <result column="did" property="dept.id"/>
  <result column="dept_name" property="dept.departmentName"/>
</resultMap>

<select id="getUserAndDept" resultMap="myUser">
	select e.id id,e.last_name last_name,d.id did,d.dept_name dept_name 
  from User e,Dept d
  where e.d_id = d.id And e.id=#{id}
</select>

<!-- 第2种 -->
<resultMap type="com.tsuiraku.User" id="myUser">
	<id column="id" property="id"/>
  <result column="last_name" property="lastName"/>
  <!-- 
		property：指定联合对象 
		javaType：指定属性对象的类型
	-->
  <association property="dept" javaType="com.tsuiraku.Dept">
  	<id column="did" property="id"/>
    <result column="dept_name" property="departmentName"/>
  </association>
</resultMap>

<select id="getUserAndDept" resultMap="myUser">
	select e.id id,e.last_name last_name,d.id did,d.dept_name dept_name 
  from User e,Dept d
  where e.d_id = d.id And e.id=#{id}
</select>
```



#### 分步查询

使用 ***\<association>*** 进行分步查询。

```xml
<resultMap type="com.tsuiraku.User" id="myUser">
	<id column="id" property="id"/>
  <result column="last_name" property="lastName"/>
  <!-- association定义关联对象的封装规则
 		select：表面当前属性是调用select查询返回的结果
		column：指定将哪一列的值传递
	-->
  <association property="dept" select="com.tsuiraku.mapper.Dept.getDeptById" 		
               column="d_id">
  </association>
</resultMap>

<select id="getUserAndDept" resultMap="myUser">
  select * from User where id=#{id}
</select>
```

```xml
<select id="getDeptById" resultType="com.tsuiraku.Dept">
  select * from Dept where id=#{id}
</select>
```



#### 延迟加载

全局配置 ***mybatis-conf.xml*** 中配置。

```xml
<settings>
  <setting name="LazyLoadingEnabled" value="true"></setting>
  <setting name="aggressiveLazyLoading" value="false"></setting>
</settings>
```



#### 关联集合封装规则

```java
public class User{
  private Integer id;
  private String lastName;
  private Dept dept;
  
  //get/set
}

public class Dept{
  private Integer id;
  private String departmentName;
  private List<User> list;
  
   //get/set
}
```

```xml
<!-- 查询部门时，将部门下所有员工也查询出 -->

<resultMap type="com.tsuiraku.Dept" id="myDept">
	<id column="id" property="id"/>
  <result column="dept_name" property="departmentName"/>
  <collection property="list" ofTypes="com.tsuiraku.User">
  		<id column="eid" property="id"/>
 		 <result column="last_name" property="lastName"/>
  </collection>
</resultMap>
<select id="getDeptById" resultMap="myDept">
  ...
</select>
```



# 动态SQL

> **动态SQL就是根据不同的条件生成不同的SQL语句**。



## if

判断标签。



**接口**

```java
public List<User> getUserByConditionIf(User user);
```

**映射文件**

```xml
<!-- if -->
<select id="getUserByConditionIf" resultType="com.tsuiraku.User">
    select * from user where
    <!-- test: 判断的表达式 (OGNL)-->
    <if test="name != null">
        name like #{name}
    </if>
    <if test="gender != null">
        and gender=#{gender}
    </if>
</select>
```



**问题**

若不携带 ***name***，那么 ***sql*** 将会变为 `select * from user where and age = #{age}`。

**解决**

1. 在 ***where*** 后加上 ***1=1***，如下

   ```xml
   <!-- if -->
   <select id="getUserByConditionIf" resultType="com.tsuiraku.User">
       select * from user where 1=1
       <!-- test: 判断的表达式 (OGNL)-->
       <if test="name != null">
           and name like #{name}
       </if>
       <if test="age != null">
           and age=#{age}
       </if>
   </select>
   ```

2. <u>使用 ***where*** 标签</u>

   ```xml
   <!-- if -->
   <select id="getUserByConditionIf" resultType="com.tsuiraku.User">
       select * from user
     <where>
       <!-- test: 判断的表达式 (OGNL)-->
       <if test="name != null">
           name like #{name}
       </if>
       <if test="gender != null">
           and gender=#{gender}
       </if>
     </where>
   </select>
   ```



## Trim

自定义字符串截取。

> ***prefix***：前缀，整个字符串加一个前缀。
>
> ***prefixOverrides***：前缀覆盖，去掉整个字符串前面多余的字符。
>
> ***suffix***：后缀，整个字符串加一个后缀。
>
> ***suffixOverrides***：后缀覆盖，去掉整个字符串后面多余的字符。



**接口**

```java
List<User> getUserByConditionTrim(User user);
```

**映射文件**

```xml
<!-- if -->
<select id="getUserByConditionTrim" resultType="com.tsuiraku.User">
    select * from user 
   <trim prefix="where" suffixOverrides="and">
    <!-- test: 判断的表达式 (OGNL)-->
    <if test="name != null">
        name like #{name} and 
    </if>
    <if test="gender != null">
        gender=#{gender}
    </if>
   </trim>
</select
```



## choose

分支选择标签。



**接口**

```java
List<User> getConditionByChoose(User user);
```

**映射文件**

```xml
<select id="getUserByConditionChoose" resultType="com.tsuiraku.User">
    select * from user
    <where>
        <choose>
            <when test="id != null and id > 0">
                and id = #{id}
            </when>
            <when test="lastName != null">
                and lastName like #{lastName}
            </when>
            <otherwise>
                1=1
            </otherwise>
        </choose>
    </where>
</select>
```



## Set

***set*** 元素会动态地在行首插入  ***SET*** 关键字，并会删掉额外的逗号。



 **接口**

```java
void update(User user);
```

**映射文件**

```xml
<!-- set -->
<update id="update">
    update user
    <set>
        <if test="lastName != null and lastName.trim() != ''">
            title = #{title},
        </if>
        <if test="gender != null and gender.trim() != ''">
            gender = #{gender},
        </if>
    </set>
    where id = #{id}
</update>

<!-- trim -->
<update id="update">
    update user
    <trim prefix="set" suffixOverrides=",">
        <if test="lastName != null and lastName.trim() != ''">
            title = #{title},
        </if>
        <if test="gender != null and gender.trim() != ''">
            gender = #{gender},
        </if>
    </trim>
    where id = #{id}
</update>
```

#### 

## foreach

批量查询。



**接口**

```java
List<User> getUserByIds(List<String> ids);
```

**映射文件**

```xml
<select id="getUserByIds" resultType="com.tsuiraku.User">
    select * from user
    <where>
        <if test="list != null and list.size() > 0">
            <!--
                collection: 指定遍历的集合；只能写与集合类型对应的名字，如果想使用其他名称则必须使用@param注解指定名称
                item: 将当前遍历的元素赋值给指定的变量
                separator: 元素之间的分隔符
                open: 指定开始符号
                close: 指定结束符号
                index: 遍历List的时候是index索引，item是当前值
                       遍历Map的时候index是map的key，item是map的值
            -->
            <foreach collection="ids" item="item_id" open="id in(" separator="," 
                     close=")">
                #{id}
            </foreach>
        </if>
    </where>
</select>
```

#### 

## 两个内置参数

> 不只是方法传递的参数可以用于判断、取值。
>
> ***mybatis***，默认还有两个参数；
>
> - ***_parameter***：代表整个参数。
> - ***_databaseId***：若配置 ***databaseIdProvider*** 标签，***_databaseId*** 就代表当前数据库的别名。



**接口**

```java
public List<User>	getUserTestInnerParameter(User user);
```

**映射文件**

```xml
<select id="getUserTestInnerParameter" resultType="com.tsuiraku.User"> 
  <if test="_databaseId=='mysql'">
  	select * from mysql_user
  </if>
  <if test="_databaseId=='oracle'">
  	select * from oracle_user
  </if>
</select>
```



## bind

元素允许你在 ***OGNL*** 表达式以外创建一个变量，并将其绑定到当前的上下文。**通常用来拼接模糊查询**。



**接口**

```java
public List<User>	getUserTestInnerParameter(User user);
```

**映射文件**

```xml
<select id="getUserTestInnerParameter" resultType="com.tsuiraku.User"> 
  <bind name="_lastName" value="'%'+ lastName+'%'"/>
  <if test="_databaseId=='mysql'">
  	select * from mysql_user
    <if test="_parameter!=null">
    	where last_name like like #{_lastName}
    </if>
  </if>
  <if test="_databaseId=='oracle'">
  	select * from oracle_user
  </if>
</select>
```



## Sql

抽取可重用的 ***sql*** 片段。

> 在真实开发中不能写这样的 ***SQL*** 语句 `select * from xxx`，但是每写一个查询语句都写上全部的列名，就造成了代码的冗余，而且也不易于维护。
>
>  ***MyBatis*** 提供了解决方案。如果表中字段发生了更改，我们只需要修改 ***sql*** 片段。



**接口方法**

```java
List<User> getAll();
```

**方法映射**

```xml
<!--
		抽取可重用的sql片段
    sql：抽取片段
    id：唯一标识
 -->
<sql id="column">
  <if test="_databasedId=mysql">
    id, title, author, create_time, views
  </if>
</sql>
<select id="getAll" resultType="org.tsuiraku.User">
    <!--
				引用抽取的片段
        include：引入sql节点定义的sql片段
        refid：引用指定id的sql片段
     -->
    select <include refid="column"/> from user
</select>
```



# 缓存机制

> 1. 什么是缓存？
>
>    存在内存中的临时数据；
>
>    将用户经常查询的数据放在缓存（内存）中，用户查询数据就不用从数据库中查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题。
>
> 2. 为什么使用缓存？
>
>    减少和数据库的交互次数，减少系统开销，提高系统效率。
>
> 3. 什么样的数据能使用缓存？
>
>    经常查询并且不经常改变的数据。
>
> 
>
> ***MyBatis*** 包含一个非常强大的查询缓存特性，它可以非常方便地定制和配置缓存。缓存可以极大的提升查询效率。
>
> ***MyBatis*** 系统中默认定义了两个缓存：
>
> 一级缓存和二级缓存；
>
> - 默认情况下，只有一级缓存开启。（***SqlSession*** 级别的缓存，也称为本地缓存）
> - 二级缓存需要手动开启和配置，他是基于 ***namespace*** 级别的缓存。
> - 为了提高拓展性，***MyBatis*** 定义了缓存接口 ***Cache***。可以通过实现 ***Cache*** 接口来定义二级缓存。



## 一级缓存

> 也叫本地缓存；
>
> 与数据库同一次会话期间查询到的数据会放在本地缓存中；
>
> 以后如果需要获取相同的数据，直接从缓存中拿，没必要再取查询数据库。



```java
public void testFirstLevelCache() throws IOException{
  SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
  SqlSession openSession = sqlSessionFactory.openSession();
  
  try{
    UserMapper mapper = openSession.getMapper(UserMapper.class);
    User user01 = mapper.getUserById(1);
    User user02 = mapper.getUserById(1);
  } finally {
    openSession.close();
  }
}
```



###  缓存失效的情况

- 不同的 ***SqlSession*** 对应不同的一级缓存；
- 同一个 ***SqlSession*** 但是查询条件不同；
- 同一个 ***SqlSession*** 两次查询期间执行了任何一次增删改操作；
- 同一个 ***SqlSession*** 两次查询期间手动清空了缓存。



## 二级缓存

> 二级缓存也叫全局缓存，一级缓存作用域太小。
>
> 基于 ***namespace*** 级别的缓存，一个名称空间对应一个二级缓存。
>
> 
>
> ***工作机制***
>
> 一个会话查询一条数据，当前这个数据会被放到当前会话的一级缓存中；
>
> 如果当前会话关闭，则会话对应的一级缓存就失效；若开启了二级缓存，即使会话关闭，一级缓存中的数据会被保存到二级缓存中；
>
> 新的会话查询信息，就可以从二级缓存中获取内容；
>
> 不同的 ***mapper*** 查出的数据会存放在对应的缓存中。



### 缓存失效的情况

- 不同的SqlSession对应不同的一级缓存
- 同一个SqlSession但是查询条件不同
- 一个SqlSession两次查询期间执行了任何一次增删改操作
- 同一个SqlSession两次查询期间手动清空了缓存



### 二级缓存的使用



**开启全局缓存**

```xml
<!-- 开启二级缓存总开关 --> 
<setting name="cacheEnabled" value="true"/>
```



**开启 Mapper.xml 的二级缓存**

```xml
<!-- 开启mapper下的namespace的二级缓存，cache标签中的所有属性都是可选的 --> 
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```



**cache标签的属性**

```
eviction：缓存的回收策略，默认的是 LRU
	LRU – 最近最少使用的：移除最长时间不被使用的对象。
	FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
	SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
	WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。

flushInterval: 缓存刷新间隔，默认不清空
	缓存多长时间清空一次，设置一个毫秒值

readOnly：是否只读，默认false
	true：只读，mybatis认为所有从缓存中获取数据的操作都是只读操作，不会修改数据。mybatis为了加快获取速度，直接就会将数据在缓存中的引用交给用户。不安全，速度快。
	false：非只读，mybatis觉得获取的数据可能会被修改。mybatis会利用序列化&反序列的技术克隆一份新的数据给你。安全，速度慢。

size：缓存存放多少元素；

type：指定自定义缓存的全类名；
```



**POJO类需要实现序列化接口**

```java
public class User implements Serializable{
  ...
}
```



```java
public void testSecondLevelCache() throws IOException{
  SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
  SqlSession openSession01 = sqlSessionFactory.openSession();
  SqlSession openSession02 = sqlSessionFactory.openSession();
  
  try{
    UserMapper mapper01 = openSession01.getMapper(UserMapper.class);
    UserMapper mapper02 = openSession02.getMapper(UserMapper.class);
    User user01 = mapper01.getUserById(1);
    openSession01.close();
    User user02 = mapper02.getUserById(1);
    openSession02.close();
  } finally {
  }
}
```



## 与缓存相关的设置/属性

> 1. 全局配置中，***cacheEnabled=true/false***，表示开启/关闭二级缓存。
> 2. ***\<select>***标签中，***useCache=true***，表示开启/关闭二级缓存。
> 3. 增删改标签中，默认 ***flushCache=true***，增删改后一级和二级缓存就清空。
> 4. 查询标签中，默认 ***flushCache=false***。
> 5. ***sqlSession.clearCache***，清除一级缓存。
> 6. ***LocalCacheScope***：本地缓存作用域。



## 缓存工作原理

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-08%2017.00.13.png" alt="截屏2021-10-08 17.00.13" style="zoom:50%;" />

缓存的顺序：二级缓存 $\gt$ 一级缓存 $\gt$ 数据库



# 逆向工程

> ***MyBatis Generator***，简称，***MBG***，，是一个专门为 ***mybatis*** 框架使用者定制的**代码生成器**，可以快速的根据表生成对应的映射文件，接口，以及 ***bean*** 类。支持基本的增删改查，以及 ***QBC*** 风格的条件查询。但是表连接、存储过程等这些复杂 ***sql*** 的定义需要我们手工编写。



**Maven**

```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.4.0</version>
</dependency>
```



**mbg.xml**

```xml
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--
        targetRuntime: 生成策略
            MyBatis3Simple: 简单版的CRUD
            MyBatis3: 豪华版的CRUD, 支持QBC风格
     -->
    <context id="mybatisGenerator" targetRuntime="MyBatis3">
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
        <!-- 数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis?userSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"
                        userId="root"
                        password="1234">
        </jdbcConnection>

        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和
            NUMERIC 类型解析为java.math.BigDecimal -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- targetProject:生成POJO类的位置 -->
        <javaModelGenerator targetPackage="com.tsuiraku.pojo"
                            targetProject=".\src\main\java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- targetProject:mapper映射文件生成的位置 -->
        <sqlMapGenerator targetPackage="org.hong.mapper"
                         targetProject=".\src\main\resources">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>
        <!-- targetPackage：mapper接口生成的位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.tsuiraku.mapper"
                             targetProject=".\src\main\java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>
        <!-- 指定数据库表 -->
        <table tableName="user" domainObjectName="User"></table>
    </context>
</generatorConfiguration>
```



**Test**

```java
public class MyBatisTest {
    // 运行这个单元测试, 自动生成
    @Test
    public void testMbg() throws IOException, XMLParserException, InvalidConfigurationException, SQLException, InterruptedException {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        File configFile = new File("mgb.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }
}
```

运行 `Test` 会自动生成 `mapper` 和 `pojo`，注意：移动文件后记得改配置文件的 `parameterType`、`type` 等属性。



# 插件开发

> ***插件原理***
>
> 在4大对象创建的时候，
>
> 1. 每个创建的对象是由 ***interceptorChain.pluginAll(parameterHandler)*** 返回；
>
>    ```java
>    public Object pluginAll(object target) {
>      for(Interceptor interceptor : interceptors) {
>        target = interceptor.plugin(target);
>      }
>      return target;
>    }
>    ```
>
> 2. 获取到所有的 ***Interceptor***（拦截器，插件需要实现的接口）；
>
>    调用 ***interceptor.plugin(target)***；
>
>    返回 ***target*** 包装后的对象。
>
> 3. 插件机制，可以使用插件为目标对象创建一个代理对象；
>
>    插件可以为4大对象创建出代理对象；
>
>    代理对象可以拦截到4大对象的每一个执行。
>
> 
>
> 
>
> **<u>插件编写</u>**
>
> 1. 编写 ***Interceptor*** 的实现类；
> 2. 使用 ***@Intercepts*** 注解完成插件签名；
> 3. 将插件注册到全局配置文件。
>
> ```java
> /**
>  * 完成插件签名：
>  *		告诉MyBatis当前插件用来拦截哪个对象的哪个方法
>  */
> @Intercepts(
> 		{
> 			@Signature(type=StatementHandler.class,method="parameterize",args=java.sql.Statement.class)
> 		})
> public class MyFirstPlugin implements Interceptor{
> 
> 	/**
> 	 * intercept：拦截；
> 	 * 		拦截目标对象的目标方法的执行；
> 	 */
> 	@Override
> 	public Object intercept(Invocation invocation) throws Throwable {
> 		// TODO Auto-generated method stub
> 		System.out.println("MyFirstPlugin...intercept:"+ invocation.getMethod());
> 		// 执行目标方法
> 		Object proceed = invocation.proceed();
> 		// 返回执行后的返回值
> 		return proceed;
> 	}
> 
> 	/**
> 	 * plugin：插件包装；
> 	 * 		包装目标对象的：包装：为目标对象创建一个代理对象
> 	 */
> 	@Override
> 	public Object plugin(Object target) {
> 		// TODO Auto-generated method stub
> 		// 我们可以借助Plugin的wrap方法来使用当前Interceptor包装我们目标对象
> 		System.out.println("MyFirstPlugin...plugin:mybatis将要包装的对象"+ target);
> 		Object wrap = Plugin.wrap(target, this);
> 		// 返回为当前target创建的动态代理
> 		return wrap;
> 	}
> 
> 	/**
> 	 * setProperties：
> 	 * 		设置插件注册时的property属性
> 	 */
> 	@Override
> 	public void setProperties(Properties properties) {
> 		// TODO Auto-generated method stub
> 		System.out.println("插件配置的信息："+ properties);
> 	}
> 
> }
> ```
>
> ```xml
> <configuration>
>   <!-- 注册插件 -->
> 	<plugins>
> 		<plugin interceptor="com.tsuiraku.mybatis.dao.MyFirstPlugin">
> 			<property name="username" value="root"/>
> 			<property name="password" value="root"/>
> 		</plugin>
> 	</plugins>
> </configuration>
> ```



***<u>当多个插件，会产生多个代理。</u>***



# 扩展

## 分页

***PageHelper*** 插件进行分页。



**Maven**

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.2.0</version>
</dependency>
```



**Pojo**

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String id;
    private String name;
    private Date create_time;
}
```



**Mapper**

```java
public interface UserMapper{
  
  public List<User> getUsers();
}
```



**Mapper.xml**

```xml
<mapper namespace="com.tsuiraku.mapper.UserMapper">
	<select id="getUsers" resultType="com.tsuiraku.entity.User">
  	select ...
  </select>
</mapper>
```



**全局配置**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!-- configuration: 核心配置文件 -->
<configuration>
		
  	<!-- 插件注册 -->
  	<plugins>
      <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
  	</plugins>
  
    <!-- 导入外部配置文件, 放在最前面 -->
    <properties resource="jdbc.properties"/>

     <settings>
        <!-- 设置日志输出, 方便观察sql语句和参数 -->
        <setting name="logImpl" value="LOG4J"/>
        <!-- 开启驼峰命名法 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

    <!--
        environments配置项目的运行环境, 可以配置多个
        default: 启用的环境
     -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!-- 数据库连接信息 -->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 每一个Mapper.xml都需要在MyBatis核心配置文件中注册 -->
    <mappers>
        <package name="com.tsuiraku.mapper"/>
    </mappers>

</configuration>
```



**Test**

```java
@Test
public void test() throws IOException {
  SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
  Sqlsession openSession = sqlSessionFactory.openSession();
  try {
    UserMapper mapper = openSession.getMapper(UserMapper.class);
    // 使用分页插件PageHelper-page
    Page<Object> page = PageHelper.startPage(1, 5);
    List<User> list = mapper.getUsers();
    System.out.println("当前页码:" + page.getPageNum());
    System.out.println("总记录数:" + page.getTotal());
    System.out.println("每页记录数:" + page.getPageSize());
    System.out.println("总页数:" + page.getPages());
    
    // 使用分页插件PageHelper-pageInfo
    PageInfo<User> info = new PageInfo<>(list);
    System.out.println("当前页码:" + info.getPageNum());
    System.out.println("总记录数:" + info.getTotal());
    System.out.println("每页记录数:" + info.getPageSize());
    System.out.println("总页数:" + info.getPages());
    System.out.println("是否第一页:" + info.isIsFirstPage());
  } finally {
    openSession.close();
  }
}
```



## 批量操作

################################### to do ###################################





## 存储过程

################################### to do ###################################





## 自定义类型处理器

################################### to do ###################################



# 工作原理

################################### to do ###################################



# SSM整合

<u>*建议直接使用 **springboot**。*</u>



# 感谢

- [尚硅谷-雷神](https://www.bilibili.com/video/BV1mW411M737?p=1)

