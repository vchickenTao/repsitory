# Spring

## 简介

**优点**

- 方便解耦，简化开发；

   通过 Spring 提供的 IOC容器，可以将对象间的依赖关系交由 Spring 进行控制，避免硬编码所造成的过度耦合。

   用户也不必再为单例模式类、属性文件解析等这些很底层的需求编写代码，可以更专注于上层的应用。

- AOP 编程的支持；

   不改变源代而增强功能。

- 声明式事务的支持；

   可以将我们从单调烦闷的事务管理代码中解脱出来，通过声明式方式灵活的进行事务管理，提高开发效率和质量。

- 方便程序的测试；

   可以用非容器依赖的编程方式进行几乎所有的测试工作，测试不再是昂贵的操作，而是随手可做的事情。

- 方便集成各种优秀框架；

   Spring 对各种优秀框架（Mybatis、Struts、Hibernate、Hessian、Quartz等）的支持。

- 降低 JavaEE API 的使用难度；

   Spring 对 JavaEE API（如 JDBC、JavaMail、远程调用等）进行了薄薄的封装层，使这些 API 的使用难度大为降低。

- Java 源码是经典学习范例。

   Spring的源代码设计精妙、结构清晰、匠心独用，处处体现着大师对Java 设计模式灵活运用以及对 Java技术的高深造诣。它的源代码无意是 Java 技术的最佳实践的范例。

**体系结构**

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/image-20210817171018121.png" alt="image-20210817171018121" style="zoom:70%;" />



## Spring简单入门

**开发步骤**

1. 导入 Spring 开发的基本包

```xml
<properties>
  <spring.version>5.0.5.RELEASE</spring.version>
</properties>

<dependencies>
  <!-- 导入spring的context坐标，context依赖core、beans、expression -->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring.version}</version>
  </dependency>
</dependencies>
```

2. 创建实体类

3. 创建 Spring 核心配置文件

   约定 **<u>applicatonContext.xml</u>**

4.  在 Spring 配置文件中配置实体类

   ```xml
   <bean id="user" class="com.tsuiraku.spring.User"></bean>
   </beans>
   ```

5. 使用 Spring 的 API 获得 Bean 实例

   ```java
   public void testAdd() {
     // 加载 spring 配置文件
     ApplicationContext context = new 
       ClassPathXmlApplicationContext("applicationContext.xml");
     // 获取配置创建的对象
     User user = context.getBean("user", User.class);
     System.out.println(user);
   }
   ```

   

# IOC

- 控制反转，把对象创建和对象之间的调用过程，交给 Spring 进行管理；
- 使用 IOC 目的，是为了耦合度降低。



## 相关API

### ApplicationContext的继承体系	

**applicationContext**：接口类型，代表应用上下文，可以通过其实例获得 Spring 容器中的 Bean 对象

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/image-20210818162141884.png" alt="image-20210818162141884" style="zoom:90%;" />



### ApplicationContext的实现类

- `ClassPathXmlApplicationContext`

  它是从类的根路径下加载配置文件，推荐使用这种

- `FileSystemXmlApplicationContex`

  它是从磁盘路径上加载配置文件，配置文件可以在磁盘的任意位置。

- `AnnotationConfigApplicationContext`

  当使用注解配置容器对象时，需要使用此类来创建 spring 容器。它用来读取注解。3



### getBean方法使用

- 传入id
- 传入字节码类型

```java
ApplicationContext app = new ClasspathXmlApplicationContext("xxx.xml")app.getBean("id")app.getBean(Class)
```



## 底层原理

> XML 解析；
>
> 工厂模型；
>
> 反射。



**原始模式**

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-21%2009.44.42.png" style="zoom:50%;" />

- 耦合的太高！

- 如何解决？- 工厂模型。



### 工厂模型

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-21%2009.46.46.png" style="zoom:50%;" />

- 降低 UserService 和 UserDao 的耦合的。



### IOC解耦过程

1. 配置创建的对象

```xml
<bean id="user" class="com.tsuiraku.User"></bean>
```

2. 创建 service、dao 类，创建工厂类

```java
class UserFactory{
  public static UserDao getDao(){
		// XML解析
    String classValue = class属性值;
    // 通过反射创建对象
    Class clazz = Class.forName(classValue);
    // 创建对象
    return (UserDao)clazz.newInstance();
  }
}
```



**<u>Spring 提供 IOC 容器实现的两种方式（两个接口）</u>**

- ***BeanFactory***
- ***ApplicationContext***



## 接口BeanFactory

IOC 容器基本实现 ，是 Spring 内部使用的接口。（开发人员一般不使用）

加载配置文件不创建对象，获取对象时创建对象。



### ApplicationContext

***BeanFactory*** 的子接口，提供更多更强大的功能。

加载配置文件时创建对象。



**两个主要实现类**（读取 ***XML*** 配置）

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-21%2010.28.31.png" style="zoom:50%;" />

> ***FileSystemXmlApplicationContext***
>
> ***ClassPathXmlApplicationContext***



## Bean管理

- Spring 创建对象；
- Spring 注入属性。



### 基于XML的Bean管理

#### **对象创建**

**使用bean标签注册容器**

```xml
<bean id="user" class="com.tsuiraku.User"></bean>
```

**bean标签属性**

> id：唯一标识；
>
> class：类全路径。

<u>**创建对象时，默认执行无参数构造方法完成对象创建。**</u>



#### 依赖注入

**使用 Set 方法注入属性**

> property：set方法注入属性。

```java
public class User { 
  // 创建属性
  private String name;
  private String age; 
  // 创建属性对应的 set 方法
  public void setName(String name) {
        this.name = name;
    }
  public void setAge(String age) { 
  this.age = age;
  }
}
```

```xml
<bean id="user" class="com.tsuiraku.User">
	<property name="name" value="tsuiraku"></property>
  <property name="age" value="21"></property>
</bean>
```



**使用有参构造进行注入属性**

> constructor-arg：构造器注入属性。

```java
public class User { 
  // 创建属性
  private String name;
  private String age; 
  // 创建有参构造器
  
  public User(String name, String age){
    this.name = name;
    this.age = age;
  }
}
```

```xml
<bean id="user" class="com.tsuiraku.User">
  <constructor-arg name="name" value="tsuiraku"></constructor-arg>
  <constructor-arg name="age" value="21"></constructor-arg>
</bean>
```



**注入其他类型属性**

- 字面量

  - ***null*** 值

  ```xml
  <bean id="user" class="com.tsuiraku.dao.impl.UserDaoImpl">
    <property name="address">
    	<null/> <!-- 注入 null 值 -->
    </property>
  </bean>
  ```

  - 特殊符号

  ```xml
  <bean id="user" class="com.tsuiraku.dao.impl.UserDaoImpl">
    <property name="address">
    	<value><![CDATA[<<我的地址>>]]></value> <!-- 注入特殊符号值 -->
    </property>
  </bean>
  ```

- 外部 ***bean***

  （创建 Service、Dao 类，Service 调用 Dao）

  ```xml
  <bean id="userDaoImpl" class="com.tsuiraku.dao.impl.UserDaoImpl"></bean>
  <bean id="userService" class="com.tsuiraku.service.impl.UserService">
  		<!-- 注入userDao对象 -->
    	<property name="userDao" ref="userDaoImpl"></property>
  </bean>
  ```

  ```java
  public class UserService {
    // 创建 UserDao 类型属性并生成 set 方法 
    // 类中属性名userDao与注册bean中的name一致
    private UserDao userDao;
    public void setUserDao(UserDao userDao) {
          this.userDao = userDao;
    }
    public void add() {
      System.out.println("service add..............."); 
      userDao.update();
    }
  }
  ```

- 内部 ***bean***

  （员工和部门，多对一关系，在员工中定义所属的部门）

  ```xml
  <bean id="emp" class="com.tsuiraku.entity.Emp">
  	<property name="name" value="西野七濑"></property>
    <property name="gender" value="女"></property>
    <!-- 内部bean设置对象属性 -->
    <property name="dept">
    	<bean id="dept" class="com.tsuiraku.entity.Dept">
      	<property name="dname" value="财务部"></property>
      </bean>
    </property>
  </bean>
  ```

- 级联赋值

  （员工和部门，多对一关系，在员工中定义所属的部门）

  ```xml
  <bean id="emp" class="com.tsuiraku.entity.Emp">
  	<property name="name" value="西野七濑"></property>
    <property name="gender" value="女"></property>
    <!-- 第一种级联赋值 -->
    <property name="dept" ref="dept"></property>
    <!-- 第二种级联赋值 -->
    <!-- 需要生存get方法 -->
    <property name="dept.dname" value="技术部"></property>
  </bean>
  
  <bean id="dept" class="com.tsuiraku.entity.Dept">
    <property name="dname" value="财务部"></property>
  </bean>
  ```

  

**依赖注入的数据类型**

- 普通数据类型

  ```xml
  <bean id="user" class="com.tsuiraku.User">
    <property name="name" value="tsuiraku"></property>
    <property name="age" value="21"></property>
  </bean>
  ```

- 集合类型

  ```java
  public class User{
    private String[] arr;
    private List<String> list;
    private Map<String, String> map;
    private Set<String> set;
    
    // get
    // set
  }
  ```

  - 数组

  ```xml
  <bean id="user" class="com.tsuiraku.User">   
    <property name="arr">
      <array>
        <value>a</value>
        <value>b</value>
        <value>c</value>
      </array>
    </property>
  </bean>
  ```

  - 列表

  ```xml
  <bean id="user" class="com.tsuiraku.User">   
    <property name="arr">
      <list>
        <value>a</value>
        <value>b</value>
        <value>c</value>
      </list>
    </property>
  </bean> 
  ```

  - ***Map***

  ```xml
  <bean id="user" class="com.tsuiraku.User">   
    <property name="arr">
      <map>
        <entry key="java" value="java"></entry>
      </map>
    </property>
  </bean> 
  ```

  - ***Set***

  ```xml
  <bean id="user" class="com.tsuiraku.User">   
    <property name="arr">
      <set>
        <value>a</value>
        <value>b</value>
        <value>c</value>
      </set>
    </property>
  </bean> 
  ```

- 集合注入对象类型

  - 第一种

  ```java
  public class Course{
  	private string cname;
    private List<String> list;
    // get
    // set
  }
  
  public class User{
  	private List<Course> list;
    // get
    // set
  }
  ```

  ```xml
  <bean id="user" class="com.tsuiraku.User">   
    <property name="list">
      <list>
        <ref bean="course1"></ref>
        <ref bean="course2"></ref>
      </list>
    </property>
  </bean> 
  
  <bean id="course1" class="com.tsuiraku.User">
  	<property name="cname" value="Java"></property>
  </bean>
  <bean id="course2" class="com.tsuiraku.User">
  	<property name="cname" value="Python"></property>
  </bean>
  ```

  - 第二种

  ```xml
  <!-- 引入util命名空间 -->
  <util:list id="courseList">
  	<value>java</value>
    <value>python</value>
  </util:list>
  
  <bean id="user" class="com.tsuiraku.User">   
    <property name="list" ref="courseList"></property>
  </bean> 
  ```



#### 工厂bean(FactoryBean)

- 普通 **bean**：在配置文件中定义 **bean** 类型即返回类型；

- 工厂 **bean**：在配置文件中定义 **bean** 类型可以与返回类型不一致。

  创建类，让类作为工厂 **bean**，实现接口 **<u>FactoryBean</u>**；

  实现接口中的方法，在实现方法中定义需要返回的 **bean** 类型。

  ```java
  public class MyBean implements FactoryBean<Course> { 
    // 定义返回 bean
    @Override
    public Course getObject() throws Exception {
          Course course = new Course();
          course.setCname("abc");
          return course;
  }
  
    @Override
    public Class<?> getObjectType() {
          return null;
  }
  @Override
  
    // 单实例/多实例
    public boolean isSingleton() { 
      return false;
  
    }
  }
  ```

  ```xml
  <bean id="myBean" class="com.tsuiraku.factorybean.MyBean">
  </bean>
  ```



#### Bean的作用域

默认情况下创建 bean 为单实例。

通过属性 **scope** 设置。

> singleton：表示单实例对象，在加载 ***spring*** 配置文件时创建单实例对象；
>
> prototyoe：表示多实例对象，在调用方法 ***getBean*** 时创建多实例对象。

```xml
<bean id="" class="" scope="prototyoe">
</bean>
```



#### Bean的生命周期

bean 对象创建、销毁的过程。

1. 创建 bean 实例（无参构造）；
2. 为 bean 的属性设置值和对其他 bean 进行引用（调用 set 方法）；
3. 调用 bean 的初始化方法（需要配置）；
4. bean 已经可以使用（已经获取对象）；
5. 当容器关闭，调用 bean 的销毁方法（需要配置）。



```java
public class User{
  
  public User(){
    System.out.println("第一步 无参数构造创建 bean 实例");
  }
  private String name;
  public void setName(String name) {
    this.name = name;
    System.out.println("第二步 为 bean 的属性赋值");
  }
  
  // 执行初始化方法
  public void init(){
    System.out.println("第三步 执行初始化方法");
  }
}
```

```xml
<bean id="user" class="com.tsuiraku.User" init-method="init" destory-method="destory">
	<property name="name" value="西野七濑"></property>
</bean>
```

```java
@Test
public void test() {
  ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
  User user = context.getbean("user", User.class);
  System.out.println("第四步 获取创建 bean 实例对象");
  
  ((ClassPathXmlApplication)context).close();
  System.out.println("第五步 执行销毁方法");
}
```



#### Bean的后置处理器

将会为容器中所有 bean 添加后置处理器。

1. 创建 bean 实例（无参构造）；
2. 为 bean 的属性设置值和对其他 bean 进行引用（调用 set 方法）；
3. **<u>把 bean 实例传递 bean 后置处理器的方法（postProcessBeforeInitialization）</u>**；
4. 调用 bean 的初始化方法（需要配置）；
5. **<u>把 bean 实例传递 bean 后置处理器的方法（postProcessAfterInitialization）</u>**；
6. bean 已经可以使用（已经获取对象）；
7. 当容器关闭，调用 bean 的销毁方法（需要配置）。



新建类需要实现接口 **<u>BeanPostProcessor</u>**。

```java
public class MyBeanPost implements BeanPostProcessor { 
  @Override
  public Object postProcessBeforeInitialization(Object bean, String beanName) throws 
    BeansException {

    System.out.println("在初始化之前执行的方法");
        
    return bean;
    }

  @Override
  public Object postProcessAfterInitialization(Object bean, String beanName) throws 
    BeansException {

    System.out.println("在初始化之后执行的方法");
      
    return bean;
    }
}
```

```xml
<!-- 配置后置处理器 -->
<bean id="myBeanPost" class="com.tsuiraku.bean.MyBeanPost"></bean>
```



#### 自动装配

根据指定装配规则（属性名称或者属性类型），Spring 自动将匹配的属性值进行注入。

> 属性 ***autowire***
>
> - byName：根据属性名进行自动注入，此时 bean 中 id 与 属性值名称需要保存一致；
> - byType：根据属性类型进行自动注入。

```xml
<bean id="emp" class="com.tsuiraku.Emp" autowire="byName"></bean>
<bean id="dept" class="com.tsuiraku.Dept"></bean>


<bean id="emp" class="com.tsuiraku.Emp" autowire="byType"></bean>
<bean id="dept" class="com.tsuiraku.Dept"></bean>
```



#### 引入外部属性文件

创建外部属性文件，***jdbc.properties***；

```properties
prop.driverClass=com.mysql.jdbc.Driver
prop.url=jdbc:mysql//localhost:3306/test
prop.userName=root
prop.password=root
```

外部 ***properties*** 属性文件引入到 ***spring*** 配置文件中，需要引入 ***context*** 命名空间；

```xml
<!-- 引入外部属性文件 -->
<context:property-placeholder location="classpath:jdbc.properties"/>
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
	<propery name="driverClassName" value="#{prop.driverClass}"></propery>
  <propery name="url" value="#{prop.url}"></propery>
  <propery name="username" value="#{prop.userName}"></propery>
  <propery name="password" value="#{prop.password}"></propery>
</bean>
```



### 基于注解方式

***AnnotationConfigApplication* **加载配置类。

简化 ***XML*** 配置开发。**<u>关于纯注解驱动开发，[Spring注解驱动](https://gitee.com/tsuiraku/java/tree/master/%E5%BA%94%E7%94%A8%E6%A1%86%E6%9E%B6/%E5%90%8E%E7%AB%AF/Spring%E5%AE%B6%E6%97%8F/Spring/%E5%9F%BA%E4%BA%8E%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8)。</u>**

#### 创建对象

**对象组件注解**

> @Component；
>
> @Service；
>
> @Controller；
>
> @Repository。
>
> 当不写属性 value 值指定 bean 的 id 时，id 默认位类名首字母小写。

#### 开启组件扫描

```xml
<context:component-scan base-package="com.tsuiraku"></context>
```

**TypeFilter**

```xml
<!--
	use-default-filters="false"，表示现在不使用默认 filter
	filter context:include-filter，设置扫描哪些内容
-->
<context:component-scan base-package="com.tsuiraku" use-default-filters="false">
 <context:include-filter type="annotation"
                         expression="org.springframework.stereotype.Controller"/>
</context:component-scan>

<!--
	配置扫描包所有内容 
	context:exclude-filter，设置哪些内容不进行扫描
-->
<context:component-scan base-package="com.tsuiraku">
 <context:exclude-filter type="annotation"
                         expression="org.springframework.stereotype.Controller"/>
</context:component-scan
```

#### 属性注入

> @Autowied：根据属性类型自动装配；
>
> @Qualifier：根据属性名称自动装配，需要和 @Autowied 一起使用；
>
> @Resource：根据属性类型或者属性名称自动装配，根据名称注入 `@Resource(name = "")`；
>
> @Value：注入普通类型属性。



**<u>关于纯注解驱动开发，[Spring注解驱动](https://gitee.com/tsuiraku/java/tree/master/应用框架/后端/Spring家族/Spring/基于注解驱动)。</u>**



# AOP

> 面向切面编程（方面），利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
>
> 通俗描述：***<u>不通过修改源代码方式，在主干功能里面添加新功能</u>***。



## 底层原理

### 动态代理



#### JDK动态代理

1. 有接口情况，使用 ***JDK*** 动态代理

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/截屏2021-10-10 10.07.29.png" style="zoom:50%;" />



**接口**

```java
interface UserDao {
  public void login();
}
```



**实现类**

```java
class UserDaoImpl implements UserDao {
  public void login(){
    // 登录实现
  }
}
```



**JDK动态代理**

```java
// 创建UserDao接口实现代理对象
// 代理对象增强方法
```



#### CGLB动态代理

2. 动态代理没有接口情况，使用 ***CGLIB*** 动态代理

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-10%2010.11.41.png" alt="" style="zoom:50%;" />



**类**

```java
class User {
  public void login(){
    // 登录实现
  }
}
```



原始做法：创建 ***User*** 的子类，重写 ***login*** 方法。

动态代理：***CGLB***，创建当前类子类的代理对象。



#### JDK动态代理实现

> 使用 ***Proxy*** 类里面的方法创建代理对象；
>
> 调用 ***newProxyInstance*** 方法；
>
> 方法有三个参数：
>
> - 第一参数，类加载器；
> - 第二参数，增强方法所在的类，这个类实现的接口，支持多个接口；
> - 第三参数，实现这个接口 ***InvocationHandler***，创建代理对象，写增强的部分。
>
> 编写 ***JDK*** 动态代理。
>
> - 创建接口，定义方法；
>
>   ```java
>   public interface UserDao {
>     public int add(int a, int b);
>     public void update(String id);
>   }
>   ```
>
> - 创建接口实现类，实现方法；
>
>   ```java
>   public class UserDaoImpl implements UserDao { 
>     	@Override
>       public int add(int a, int b) {
>           return a+b;
>   
>       }
>   		@Override
>       public String update(String id) {
>           return id;
>       } 
>   }
>   ```
>
> - 使用 ***Proxy*** 类创建接口代理对象。
>
>   ```java
>   public class JDKProxy {
>     
>     public static void main(String[] args) {
>     
>       // 创建接口实现类代理对象
>       Class[] interfaces = {UserDao.class};
>       // Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new InvocationHandler() {
>         // @Override
>         // public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
>         // return null; // }
>         // });
>       UserDaoImpl userDao = new UserDaoImpl();
>       UserDao dao = (UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new UserDaoProxy(userDao));
>       int result = dao.add(1, 2);
>       System.out.println("result:" + result);
>       }
>   }
>   ```
>
>   ```java
>     // 创建代理对象代码
>   class UserDaoProxy implements InvocationHandler { 
>     // 把创建的是谁的代理对象，就把谁传递过来 
>     // 有参数构造传递
>     private Object obj;
>     public UserDaoProxy(Object obj) {
>       this.obj = obj;
>     }
>     
>     // 增强的逻辑
>     @Override
>     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable 
>     {
>       // 方法之前
>       System.out.println("方法之前执行...." + method.getName() + " :传递的参 数..."+ 
>                          Arrays.toString(args));
>       // 被增强的方法执行
>       Object res = method.invoke(obj, args); 
>         
>       //方法之后 
>       System.out.println("方法之后执行...." + obj); 
>         
>       return res;
>     }
>   }
>   ```



## 操作术语

**连接点（JoinPoint）**

类里面可以被增强的方法，即称为连接点。

**切入点（Pointcut）**

实际被真正增强的方法，即称为切入点。

**通知/增强（Advice）**

实际真正增强的逻辑部分，即称为通知（增强）。通知有多种类型。

- 前置通知；
- 后置通知；
- 环绕通知；
- 异常通知；
- 最终通知。

**切面（Aspect）**

把通知应用到切入点过程。

**引入（introduction）**

**目标（target）**

**代理（proxy）**

**织入（weaving）**



## AOP操作

> ***Spring*** 框架一般都是基于 ***AspectJ*** 实现 ***AOP*** 操作，***AspectJ*** 不是 ***Spring*** 组成部分，独立 ***AOP*** 框架；
>
> 基于 ***AspectJ*** 实现 ***AOP*** 操作：
>
> - 基于 xml 配置文件实现；
> - 基于注解方式实现（推荐）。
>
> 在项目工程里面引入 **AOP** 相关依赖：
>
> <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-15%2010.52.07.png" style="zoom:50%;" />
>
> 切入点表达式：
>
> - 切入点表达式作用：知道对哪个类里面的哪个方法进行增强；
> - 语法结构：***execution***( [权限修饰符] [返回类型] [类全路径] [方法名称]\([参数列表]) )。



### 基于注解

1. 创建类

```java
public class User { 
  public void add() {
    System.out.println("add......."); }
}
```

2. 创建增强方法

```java
// 在增强类里面，创建方法，让不同方法代表不同通知类型
public class UserProxy {
  public void before() {
    // 前置通知
    System.out.println("before......"); }
}
```

3. 进行通知的配置

   1. 需要开启注解扫描

   ```xml
   <?
   xml version="1.0" encoding="UTF-8"
   <beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   ?>
   xmlns:context="http://www.springframework.org/schema/context"
   xmlns:aop="http://www.springframework.org/schema/aop"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context/spring-context.xsd>
   <!-- 开启注解扫描 -->
   <context:component-sacn base-package="com.tsuiraku.spring5.aopanno"></context:component-scan>
   ```

   2. 使用注解标注 ***User*** 和 ***UserProxy*** 对象

   ```java
   // 被增强的类
   @Component
   public class User {
   }
   
   // 增强的类
   @Component
   // 生成代理对象
   @Aspect
   public class UserProxy {
   }
   ```

   3. 开启生成代理对象

   ```xml
   <!-- 开启 Aspect 生成代理对象-->
   <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
   ```

4. 配置不同类型的通知

```java
@Component
@Aspect // 生成代理对象
public class UserProxy { 

  // 前置通知
  // @Before 注解表示作为前置通知
  @Before(value = "execution(* com.tsuiraku.spring5.aopanno.User.add(..))") 
  public void before() {
    System.out.println("before........."); 
  }
  // 后置通知(返回通知)

  @AfterReturning(value = "execution(* com.tsuiraku.spring5.aopanno.User.add(..))")
  public void afterReturning() { 
    System.out.println("afterReturning.........");
  }

  // 最终通知
  @After(value = "execution(* com.tsuiraku.spring5.aopanno.User.add(..))") 
  public void after() {
    System.out.println("after........."); 
  }


  @AfterThrowing(value = "execution(* com.tsuiraku.spring5.aopanno.User.add(..))")
  public void afterThrowing() { 
    System.out.println("afterThrowing.........");
  }

  // 环绕通知
  @Around(value = "execution(* com.tsuiraku.spring5.aopanno.User.add(..))")
  public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
    System.out.println("环绕之前........."); //被增强的方法执行 
    proceedingJoinPoint.proceed(); 
    System.out.println("环绕之后.........");
  } 
}
```

5. 相同的切入点抽取

```java
// 相同切入点抽取
@Pointcut(value = "execution(* com.tsuiraku.spring5.aopanno.User.add(..))")
public void pointdemo() {
}
// 前置通知
// @Before 注解表示作为前置通知 
@Before(value = "pointdemo()") 
public void before() {
  System.out.println("before........."); 
}
```

6. 有多个增强类多同一个方法进行增强，设置增强类优先级

```java
@Component
@Aspect
@Order(1) // 在增强类上面添加注解 @Order(数字类型值)，数字类型值越小优先级越高
public class PersonProxy
```

7. 完全使用注解开发

```java
@Configuration
@ComponentScan(basePackages = {"com.tsuiraku"}) 
@EnableAspectJAutoProxy(proxyTargetClass = true) 
public class ConfigAop {}
```



### 基于配置

1. 创建两个类，增强类和被增强类，创建方法 
2. 在 ***spring*** 配置文件中创建两个类对象

```xml
<!--创建对象-->

<bean id="book" class="com.tsuiraku.spring5.aopxml.Book"></bean>

<bean id="bookProxy" class="com.tsuiraku.spring5.aopxml.BookProxy"></bean>
```

3. 在 ***spring*** 配置文件中配置切入点

```xml
<!--配置 aop 增强-->

<aop:config> 
  <!--切入点-->
  <aop:pointcut id="p" expression="execution(*com.tsuiraku.spring5.aopxml.Book.buy(..))"/>
  <!--配置切面-->
  <aop:aspect ref="bookProxy"> <!--增强作用在具体的方法上-->
    <aop:before method="before" pointcut-ref="p"/>
  </aop:aspect>
</aop:config>
```



# JdbcTemplate

## 简介

***Spring*** 框架对 ***JDBC*** 进行封装，使用 ***JdbcTemplate*** 方便实现对数据库操作。



## 快速入门

1. 引入相关依赖

```
druid
mysql-connector-java
spring-jdbc
spring-orm
spring-tx
```

2. 配置文件配置数据库连接池

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
destroy-method="close">
 <property name="url" value="jdbc:mysql:///user_db" />
 <property name="username" value="root" />
<property name="password" value="root" />
 <property name="driverClassName" value="com.mysql.jdbc.Driver" />
</bean>
```

3. 配置 ***JdbcTemplate*** 对象，注入 ***DataSource***

```xml
<!-- JdbcTemplate 对象 -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate"> <!--注入 dataSource-->
 <property name="dataSource" ref="dataSource"></property>
</bean>
```

4. 创建 ***service*** 类，创建 ***dao*** 类，在 ***dao*** 注入 ***jdbcTemplate*** 对象

```xml
<!-- 组件扫描 -->
<context:component-scan base-package="com.atguigu"></context:component-scan>
```



## 操作数据库

1. 创建实体类（***entity***）

```java
public class User{}
```

2. 创建 ***service*** 和 ***dao*** 层

### 添加

```java
(1)在 dao 进行数据库添加操作
(2)调用 JdbcTemplate 对象里面 update 方法实现添加操作
```

***update(sql，可变参数即sql的value)*** 

```java
@Repository
public class BookDaoImpl implements BookDao { 
  //注入 JdbcTemplate

  @Autowired
  private JdbcTemplate jdbcTemplate; 
  
  //添加的方法
  @Override
  public void add(Book book) { //1 创建 sql 语句
    String sql = "insert into t_book values(?,?,?)";

    //2 调用方法实现
    Object[] args = {book.getUserId(), book.getUsername(),book.getUstatus()};
    int update = jdbcTemplate.update(sql,args);
    System.out.println(update); 
  }
}
```

**测试**

```java
@Test
public void testJdbcTemplate() { 
  ApplicationContext context =
    new ClassPathXmlApplicationContext("bean1.xml"); 
  BookService bookService = context.getBean("bookService",BookService.class);
  Book book = new Book();
  book.setUserId("1");
  book.setUsername("java"); 
  book.setUstatus("a"); 
  bookService.addBook(book);
}
```



### 修改

```java
@Override
public void updateBook(Book book) {

  String sql = "update t_book set username=?,ustatus=? where user_id=?"; 
  Object[] args = {book.getUsername(), book.getUstatus(),book.getUserId()}; 
  int update = jdbcTemplate.update(sql, args);
  System.out.println(update);
}
```



### 删除

```java
@Override
public void delete(String id) {
  String sql = "delete from t_book where user_id=?"; 
  int update = jdbcTemplate.update(sql, id); 
  System.out.println(update);
}
```



### 查询

#### 返回某个值

***queryForObject(sql，Class\<T>，requiredType)***

```java
@Override
public int selectCount() {
  String sql = "select count(*) from t_book";
  Integer count = jdbcTemplate.queryForObject(sql, Integer.class); 
  return count;
}
```



#### 返回对象

***queryForObject(sql，RowMapper\<T>，Object...  args)***

```java
@Override
public Book findBookInfo(String id) {

  String sql = "select * from t_book where user_id=?"; //调用方法
  Book book = jdbcTemplate.queryForObject(sql, new
                                          BeanPropertyRowMapper<Book>(Book.class), id); 
  return book;
}
```



### 批量操作

#### 添加

***batchUpdate(sql，List\<Object[]> batchArgs)***

```java
@Override
public void batchAddBook(List<Object[]> batchArgs) { 
  String sql = "insert into t_book values(?,?,?)"; 
  int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs); 
  System.out.println(Arrays.toString(ints));
}
```

**测试**

```java
//批量添加测试
List<Object[]> batchArgs = new ArrayList<>(); 
Object[] o1 = {"3","java","a"};
Object[] o2 = {"4","c++","b"};
Object[] o3 = {"5","MySQL","c"}; 
batchArgs.add(o1);
batchArgs.add(o2); 
batchArgs.add(o3); 
// 调用批量添加 
bookService.batchAdd(batchArgs);
```



#### 修改

```java
@Override
public void batchUpdateBook(List<Object[]> batchArgs) {
  String sql = "update t_book set username=?,ustatus=? where user_id=?"; 
  int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs); 
  System.out.println(Arrays.toString(ints));
}
```

**测试**

```java
// 批量修改
List<Object[]> batchArgs = new ArrayList<>(); 
Object[] o1 = {"java0909","a3","3"}; 
Object[] o2 = {"c++1010","b4","4"};
Object[] o3 = {"MySQL1111","c5","5"}; 
batchArgs.add(o1);
batchArgs.add(o2);
batchArgs.add(o3);
// 调用方法实现批量修改
bookService.batchUpdate(batchArgs);
```



#### 删除

```java
@Override
public void batchDeleteBook(List<Object[]> batchArgs) { 
  String sql = "delete from t_book where user_id=?"; 
  int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs); 
}
```

**测试**

```java
//批量删除
List<Object[]> batchArgs = new ArrayList<>();
Object[] o1 = {"3"};
Object[] o2 = {"4"}; 
batchArgs.add(o1); 
batchArgs.add(o2); 
// 调用方法实现批量删除 
bookService.batchDelete(batchArgs);
```



# 事务

## 简介

事务是数据库操作最基本单元，逻辑上一组操作，要么都成功，如果有一个失败所有操作都失败。
典型场景：银行转账。

事务四大特性（ACID）：

- 原子性
- 一致性
- 隔离性
- 持久性

## 快速入门

（银行转账）

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-17%2023.10.09.png" style="zoom:50%;" />

1. 创建数据库（略）
2. 创建 ***Service*** 和 ***Dao*** 层

```java
@Service
public class UserService { 
  // 注入 dao
  @Autowired
  private UserDao userDao;
}

@Repository
public class UserDaoImpl implements UserDao { 
  @Autowired
  private JdbcTemplate jdbcTemplate;
}
```

3. ***Dao*** 创建两个方法和 ***Service*** 创建转账方法

```java
@Repository
public class UserDaoImpl implements UserDao { @Autowired
  private JdbcTemplate jdbcTemplate;
  @Override
  public void reduceMoney() {
    String sql = "update t_account set money=money-? where username=?"; 
    jdbcTemplate.update(sql,100,"lucy");
}

  @Override
  public void addMoney() {
    String sql = "update t_account set money=money+? where username=?"; 
    jdbcTemplate.update(sql,100,"mary");
  } 
}

@Service
public class UserService { 
  // 注入 dao
  @Autowired
  private UserDao userDao;
  // 转账的方法
  public void accountMoney() { 
    userDao.reduceMoney();
    userDao.addMoney();
    }
}
```

4. 当上述代码异常执行时会出现问题

```java
// 模拟一个异常
public void accountMoney() { 
  userDao.reduceMoney();
  int i = 10 / 0;
  userDao.addMoney();
}
```

**问题**：此时，***lucy*** 的金钱减少 ***100***，但是 ***mary*** 并没有增加 ***100***。

**解决**：

```java
public void accountMoney() { 
  try {
    // 第一步，开启事务
    // 第二步，进行业务操作
    userDao.reduceMoney();
  	int i = 10 / 0;
  	userDao.addMoney();
    // 第三步，没有发生异常，提交事务	
  } catch(Exception e) {
    // 第四步，若出现异常，回滚
  }
}
```



## 事务操作

事务添加到 ***JavaEE*** 三层结构里面 ***Service*** 层（业务逻辑层）。

***Spring*** 事务管理 ***API***。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-18%2014.15.38.png" style="zoom:50%;" />



### 基于注解

> 推荐使用。



#### 如何使用

1. 配置文件配置事务管理器

```xml
<!-- 创建事务管理器 --> 
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <!-- 注入数据源 -->
  <property name="dataSource" ref="dataSource"></property> 
</bean>

```

2. 配置文件开启事务注解

```xml
<!-- 需要先引入tx空间 -->

<!--开启事务注解-->
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
```

3. 在 ***service*** 类上面（或者 ***service*** 类里面方法上面）添加事务注解

```java
/**
* (1)@Transactional，这个注解添加到类上面，也可以添加方法上面 
* (2)如果把这个注解添加类上面，这个类里面所有的方法都添加事务 
* (3)如果把这个注解添加方法上面，为这个方法添加事务
*/
@Service
@Transactional
public class UserService {}
```



#### 相关属性配置

**@Transactional**

> ***propagation***
>
> - `@Transactional(propagation = Propagation.REQUIRED)`
>
> - 事务传播行为。
> - 多事务方法直接进行调用，这个过程中事务是如何进行管理的。
>
> <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-10-18%2015.14.07.png" style="zoom:50%;" />
>
> ***isolation***
>
> - `@Transactional(isolation = Isolation.READ_UNCOMMITTED)`
>
> - 事务隔离级别。
> - 事务有特性成为隔离性，多事务操作之间不会产生影响。不考虑隔离性产生很多问题。
> - 有三个读问题：脏读、不可重复读、虚(幻)读。
> - 脏读：一个未提交事务读取到另一个未提交事务的数据。
> - 不可重复读：一个未提交事务读取到另一提交事务修改数据。
> - 虚读：一个未提交事务读取到另一提交事务添加数据。
>
> 隔离级别
>
> |                              | 脏读 | 不可重复度 | 幻读 |
> | ---------------------------- | ---- | ---------- | ---- |
> | READ UNCOMMITTED（读未提交） | ✔️    | ✔️          | ✔️    |
> | READ COMMITTED（读已提交）   |      | ✔️          | ✔️    |
> | REPEATABLE READ（可重复读）  |      |            | ✔️    |
> | SERIALIZABLE（串行化）       |      |            |      |
>
> 
>
> ***timeout***
>
> - `@Transactional(timeout = -1)`
>
> - 超时时间。
> - 事务需要在一定时间内进行提交，如果不提交进行回滚。
> - 默认值是 -1 ，设置时间以秒单位进行计算。
>
> 
>
> ***readOnly***
>
> - `@Transactional(readOnly = false)`
>
> - 是否只读。
> - ***readOnly*** 默认值 ***false***，表示可以查询，可以添加修改删除操作。
> - 设置 ***readOnly*** 值是 ***true***，设置成 ***true*** 之后，只能查询。
>
> 
>
> ***rollbackFor***
>
> - 设置哪些异常进行回滚。
>
> 
>
> ***norollbackFor***
>
> - 设置哪些异常不进行回滚。



#### 完全注解声明式事务开发

```java
@Configuration // 配置类
@ComponentScan(basePackages = "com.tsuiraku") // 组件扫描 
@EnableTransactionManagement // 开启事务
public class TxConfig {
  // 创建数据库连接池
  @Bean
  public DruidDataSource getDruidDataSource() {
    DruidDataSource dataSource = new DruidDataSource(); 
    dataSource.setDriverClassName("com.mysql.jdbc.Driver"); 
    dataSource.setUrl("jdbc:mysql:///user_db"); dataSource.setUsername("root"); 
    dataSource.setPassword("root");
    return dataSource;
  }
  
  // 创建 JdbcTemplate 对象
  @Bean
  public JdbcTemplate getJdbcTemplate(DataSource dataSource) { // IOC容器中已经注入dataSource
    JdbcTemplate jdbcTemplate = new JdbcTemplate();
    // 注入 dataSource 
    jdbcTemplate.setDataSource(dataSource);
    return jdbcTemplate;
  }

  // 创建事务管理器
  @Bean
  public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource) 
  { 
    DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
    transactionManager.setDataSource(dataSource);
    return transactionManager;
    }
}
```



### 基于配置

> 基于 ***XML*** 配置。



1. 配置事务管理器
2. 配置通知
3. 配置切入点和切面

```xml
<!-- 1.创建事务管理器 --> 
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <!-- 注入数据源 -->
 <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 2.配置通知-->
<tx:advice id="txadvice"> 
  <!-- 配置事务参数 --> 
  <tx:attributes>
		<!-- 指定哪种规则的方法上面添加事务 -->
    <tx:method name="accountMoney" propagation="REQUIRED"/>
  </tx:attributes>
</tx:advice>

<!-- 3.配置切入点和切面 -->
<aop:config> 
  <!-- 配置切入点 -->
   <aop:pointcut id="pt" expression="execution(*com.tsuiraku.spring5.service.UserService.*(..))"/>
  <!-- 配置切面 -->
 <aop:advisor advice-ref="txadvice" pointcut-ref="pt"/>  
</aop:config>
```



# 感谢

- [尚硅谷-王老师](https://www.bilibili.com/video/BV1Vf4y127N5?p=1)
- [黑马程序员](https://www.bilibili.com/video/BV1WZ4y1P7Bp?spm_id_from=333.999.0.0)

