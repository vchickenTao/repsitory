### 1.Spring是如何创建一个Bean对象的？

比如有以下两个service实例：

```java
@Component("orderService")
public class OrderService(){
}

@Component("userService")
public class UserService(){
	@Autowired
	private OrderService orderService;
	
	public void test(){
        System.out.println(orderService);
    }
}

```

- Spring也是java写的，根据我们对java的基本认知，要创建一个对象，需要基于对应的类里面的**构造方法**。
  在spring中并没有显示声明特殊的构造方法，那么就使用的是无参的构造方法。
  **userService -----> 无参构造方法 -----> 对象**
  此时此刻，得到的userService对象中的属性orderService是没有值的。(相当于 new UserService())，但是相信大家用过都知道，使用getBean返回的userService对象中orderService属性是有值的。

- 在Spring内部通过构造方法生成的对象并不是我们上面理解的那样，二者之间还是有一区别的中间还有一些其他步骤。
  **userService -----> 无参构造方法 -----> 对象-----> 依赖注入-----> spring内部一些其他步骤----->Bean 对象**
  `依赖注入`，这是非常重要的一个步骤。笼统的说，依赖注入就是给对象的属性赋值（@Autowired/@Resource/ @Inject 注解修饰的属性）

```java
UserService userSerice1 = new UserService();
//伪代码，获取所有的声明字段
for(Field fields : userService1.getDeclaredFields()){
    //判断当前的字段的属性是否包含Autowired注解
    if(field.isAnnotationPresent(Autowired.class)){
       field.set(userService1,??);
    }
}
//............................
```

​	**1、推断构造方法**

这里先插一个点，比如在userService中，手动添加了一个构造方法，

```java
@Component
public class UserService{
    
    private OrderService orderService;
    
    public UserService(OrderService orderService){
        this.orderService = orderService;
        System.out.println("1");
    }
}
```

此时userService中有几个构造方法呢？

这个时候只有一个构造方法了。再调用无参构造方法的时候就会报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/3b9416172606445ab1f408c7ea5783a7.png)
**如果类没有构造方法，java会把我们默认一个无参的构造方法，如果自己手动加了构造方法，那么java就不会默认有无参构造方法了，如果需要的话，需要自己手动声明。**

```java
@Component
public class UserService{

    private OrderService orderService;

    public UserService(){
        System.out.println("0");
    }
    
    public UserService(OrderService orderService){
        this.orderService = orderService;
        System.out.println("1");
    }
}
```

现在共有两个构造方法，那么 spring会用哪个呢？可以跑下程序看下效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/c464dfc3a0814ad7a850b7b29e9807c7.png)
打印出0，走的是无参的构造方法，那是不是第二个有参数的方法不能用呢？我们把无参构造方法注释掉，发现是可以正常使用的，打印了1，那既然Spring两个构造方法都可以用，为什么用无参的构造方法呢？怎么选择的呢？

这个稍后分析，我们现在再加个构造方法

```java
@Component
public class UserService{

    private OrderService orderService;

    public UserService(OrderService orderService){
        this.orderService = orderService;
        System.out.println("1");
    }

    public UserService(OrderService orderService,OrderService orderService1){
        this.orderService = orderService;
        System.out.println("2");
    }
}
```

现在依然是2个构造方法，现在spring会用哪个构造方法呢？
运行下程序，发现在实例化的时候报错了

为什么有两个构造方法的时候报错呢？都用不了么？在上面已经测试了，一个参数的构造方法Spring是可以使用的，那么两个参数的构造方法Spring不能用么？？

很简单，可以把一个参数的构造方法注释了，只留下2个参数的构造方法，发现是可以正常执行的。

- **那么既然这三个构造方法（无参、一个参数、两个参数）都可以单独使用，为什么一起声明出来的时候会报错呢？**
  - 对于Spring来说，发现一个类里面有两个/多个构造方法，spring该怎么选择呢？？其实是没有办法进行决策的，所以就直接报错了
- **那为什么无参和一个参数的构造方法两个一起声明可以正常执行呢？**
  - Spring发现一个类里面有多个构造方法且没有明确指定使用哪个构造方法的情况下，会优先找是否有无参的构造方法，无参的构造方法是java默认提供的，没有太多的业务逻辑，更方便去创建对象，spring也是类似于这样一个逻辑。
  - 当声明了无参和一个参数的构造方法，spring默认找无参的构造方法，找到了，直接使用，正常初始化
  - 当声明了一个参数和两个参数的构造方法，spring默认找无参的构造方法，没有找到，所以就报错了。其实在报错信息里面也可以看出来，错误是：xxx.UserService.< init>() 这个<init>()就代表的是无参的构造方法      
  - 当只声明一个构造方法，spring不需要选择，直接使用，也可以正常执行

当然，也可以明确告诉spring使用哪个构造方法生成对象（多个构造方法的情况下）
在对应的构造方法上面增加 @Autowired 注解

运行程序，发现可以正常执行了。
那如果，在多个构造方法上面加上了 @Autowired注解，spring不知道如何选择，仍然是会报错的。

​	**2.什么是先bytype再byName**

比如以上代码，指定了使用一个参数的构造方法，这个构造方法是需要参数的，spring在调用的时候会给参数赋值么？
可以打断点看下，运行代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a6cbd5caa6e423b9a027ce90a7afadb.png)

 可以看到执行到这里的时候，参数是有值的。那spring是从哪里拿到orderService对象来赋值给参数的呢？
   - 因为orderService也是一个bean，spring在执行到这里的时候，发现需要一个orderService对象，会先从单例池中去找，是否存在orderService的bean对象。如果有，就直接拿来使用。如果没有，不会传一个null进去，而是会创建一个orderBean对象，（这里呢就可能出现循环依赖的情况，具体可前往循环依赖介绍及场景查看）创建完成之后，会放入单例池，并将创建的bean对象传给userService的构造方法参数使用。

        

        单例池是一个map，map的值是bean的名字，value是bean对象，那spring具体该怎么去map中去找呢？现在只需要一个orderService的对象，是根据入参的名字把它当做map的key去找，这样可不可以呢？

        - 首先，根据入参名称当做key肯定是不可以的，一般情况下参数命名不会很规范，如就存在以下情况：

        ![在这里插入图片描述](https://img-blog.csdnimg.cn/83dc2d24dd6f4ebe97b9ad495b8b1348.png)

        其实希望获取的是orderService的对象，如果根据入参名称取，取得就是一个memberService对象，就会有问题。

        -  所以Spring会先根据类型去找

          判断单例池的map中是否有bean对象的类型是orderService，如果有就直接拿来使用。
                当然这样直接根据类型去找也会有问题，单例池map的key是唯一的，并不代表value是唯一的，可能会存在<orderService,OrderService>，<orderService1,OrderService>,<orderService2,OrderService>,这三个都是OrderService类型，但是是不同的对象，只根据类型寻找，会找到3个数据，那怎么区分到底使用哪个呢？

          **这里穿插一个问题。如果说orderService是一个单例bean，那么就表示在Spring容器中只能有一个类型为OrderService的对象么？答案是否定的，可以有多个，需要保证多个bean的名称不同即可。单例bean的效果是传入同一个名多次获取，结果是同一个对象。需要和单例模式区分开**

          需要从3个对象中找到一个，可以思考下这三个对象有什么是唯一的，很明显就是bean的名字。所以spring接着会根据入参的名字从找到的数据中进行匹配（当然，如果根据类型只找到了一条数据就不需要根据名称去找了）
           根据名称去匹配，要么匹配到一个对象直接使用，要么没有匹配到 直接报错，

          ![在这里插入图片描述](https://img-blog.csdnimg.cn/6b98fceb543541faac3f41d2a44f4363.png)

          需要一个orderService对象，找到3个bean，名称分别为：orderService，orderService1，orderService2

          这个就是**所说的先bytype，后byname**
          **userService -----> 推断构造方法 -----> 对象-----> 依赖注入-----> spring内部一些其他步骤----->Bean 对象**



### 2.什么是单例池？作用是什么？

对于userService而言，默认情况下是一个单例bean，那么单例bean体现的效果是什么样子呢？
比如有以下代码：

```java
UserService userService = (UserService)applicationContext.getBean("userService");
UserService userService1 = (UserService)applicationContext.getBean("userService");
UserService userService2 = (UserService)applicationContext.getBean("userService");
```

通过getBean获取3次bean，最终返回的结果是同一个对象（指向同一地址的）

底层使用map实现，保证key唯一。Map<名字，对象>，getBean时先从map中找是否存在需要的对象。有的话就直接返回了，所以一个名字对应一个对象，相同名称会返回相同对象。
如果在map中根据名字没有找到对象，就需要去创建一个对象了，然后会把创建对象放入map中,第二次/第三次获取的时候就可以直接从map中获取了。
**userService -----> 推断构造方法 -----> 对象-----> 依赖注入----->放入单例池map ----->spring内部一些其他步骤----->Bean 对象**

这个map呢就是spring中的单例池（在Spring源码底层，单例池singletonObjects ConcurrentHashMsp<beanName,bean对象>）

也就是说
因为map里面存储的就是单例bean对象，所以叫`单例池`。
反过来想一下，如果三次获取的userService都是原型bean（也就是多例bean，每次getBean，返回的都是不同的bean），那在创建Springbean的时候就不需要单例池了
**userService -----> 推断构造方法 -----> 对象-----> 依赖注入----->放入单例池map （若是多例bean，则不需要该步骤）----->spring内部一些其他步骤----->Bean 对象**

### 3.Spring的bean对象和普通对象之间的区别是什么？

到这里，相比大家都理解了，通常说的Spring中bean对象到底是什么样子了。
其实通过无参构造方法创建出的对象和最终生成的bean对象，大部分情况下二者是同一个对象，只不过是处于不同时期而已。

这里先引入初始化的概念，接着会详细介绍
**userService -----> 推断构造方法 -----> 对象-----> 依赖注入----->初始化前----->初始化----->初始化后----->放入单例池map （若是多例bean，则不需要该步骤）----->spring内部一些其他步骤----->Bean 对象**

### 4.初始化前

比如有以下业务场景
新增User类

```java
public class User{
}
```

在userSerice中增加user属性

```java
@Component
public class UserService{

    @Autowired
    private OrderService orderService;

    private User admin;

    public void test(){
        System.out.println(orderService);
    }
}
```

类中的user是管理员用户，一般情况下，管理员只有一个，如果希望在getBean获取userService之后，其中的admin属性就是有值的。
现在获取到的是空值，希望可以获取到数据中存储的admin的信息。

这里呢，有小伙伴说，可以在userService中的admin上面加上Autowired注解，然后在User类上加上Component注解当做一个bean对象不就可以自动注入了么？
这样做，admin属性是会有值，但是spring相当于只做了个 new User()的动作，并不是我们希望的数据库中的值。

可以再深入思考下，对于spring来说，怎么知道要查哪个表？查哪个字段？等这些信息
这些其实spring都是不知道的，只有程序员自己知道，所以这个时候就需要提供一个方法，在方法中自己去执行SQL查询管理员信息。

```java
public class UserService{
    @Autowired
    private OrderService orderService;

    private User admin;

    public void a(){
        //mysql--管理员信息---user对象--admin
    }
    
    public void test(){
        System.out.println(orderService);
    }
}
```

有了方法之后，spring就可以去调用该方法去执行了（也就是在创建bean的过程中，自动调用service中的a方法）。
**userService ----->推断构造方法 -----> 对象-----> 依赖注入----->初始化前（a()）----->初始化----->初始化后----->放入单例池map （若是多例bean，则不需要该步骤）----->spring内部一些其他步骤----->Bean 对象**

现在最主要的就是，spring怎么知道需要在初始化前调用a方法呢？service中可能有很多方法， spring怎么确定呢？敲代码的小伙伴就要想办法让a方法变得更特殊一点，那么实现呢？其实也很简单，Spring中有提供注解PostConstruct

```java
    @PostConstruct
    public void a(){
        //mysql--管理员信息---user对象--admin
    }

UserService userSerice1 = new UserService();
//伪代码，获取所有的声明字段
for(Field fields : userService1.getDeclaredFields()){
    //判断当前的字段的属性是否包含PostConstruct注解
    if(field.isAnnotationPresent(PostConstruct.class)){
       field.invoke(userService1,null);
    }
}
//............................	
```

spirng在执行到初始化前时会去找前面生成对象里面有没有存在用PostConstruct注解修饰的方法，然后用反射的方式执行这些方法
**userService -----> 推断构造方法 -----> 对象-----> 依赖注入----->初始化前（@PostConstruct）----->初始化----->初始化后----->放入单例池map （若是多例bean，则不需要该步骤）----->spring内部一些其他步骤----->Bean 对象**

### 5.初始化

对于上述的需求，还可以使用另一种方法实现。
主要目的是为了使a方法更特殊一点，那么可以通过什么方式让userService中的a方法特殊一点呢？
比如说，实现接口：InitializingBean
**userService -----> 推断构造方法 -----> 对象-----> 依赖注入----->初始化前（@PostConstruct）----->初始化（InitializingBean）----->初始化后----->放入单例池map （若是多例bean，则不需要该步骤）----->spring内部一些其他步骤----->Bean 对象**
这个是spring框架提供的一个接口，类里面只有一个接口：afterPropertiesSet

![在这里插入图片描述](https://img-blog.csdnimg.cn/d3ae8dcc551e420d940b52a3e278521c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwqDml7blhYnmuIXmtYXjgbTorrjkvaDlronnhLY=,size_15,color_FFFFFF,t_70,g_se,x_16)

需要在service中实现对应的方法

```java
public class UserService implements InitializingBean {
    @Autowired
    private OrderService orderService;
    private User admin;

    @Override
    public void afterPropertiesSet() throws Exception {
        //mysql--管理员信息---user对象--admin
    }
    public void test(){
        System.out.println(orderService);
    }
}
```

其实在初始化的过程中，就相当于在处理InitializingBean接口，在初始化这一步，spring先判断当前bean对象是否有afterPropertiesSet方法，如果存在，要去调用。
那spring怎么判断当前对象有没有afterPropertiesSet方法呢？这里可以简单来看下源码

- 在doCreateBean方法中，若实例对象为空，则会创建一个对象，相当于new了一个对象![在这里插入图片描述](https://img-blog.csdnimg.cn/7d1a6292f86d462db38b163f1826fd03.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwqDml7blhYnmuIXmtYXjgbTorrjkvaDlronnhLY=,size_20,color_FFFFFF,t_70,g_se,x_16)

- 继续往下走，会填充bean的属性（被@Autowired @Resource @Inject 注解修饰的属性）
  然后会初始化其他信息

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/1a36e90672444ef09452b8c8717243dd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwqDml7blhYnmuIXmtYXjgbTorrjkvaDlronnhLY=,size_20,color_FFFFFF,t_70,g_se,x_16)

- 在初始化的方法里面，我们可以看到调用了方法applyBeanPostProcessorsBeforeInitialization，这里是处理 第四点里面介绍的被注解@PostConstruct修饰的方法

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/d8df6818d34343279c46e63fadd25320.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwqDml7blhYnmuIXmtYXjgbTorrjkvaDlronnhLY=,size_20,color_FFFFFF,t_70,g_se,x_16)

- 接下来就会执行初始化方法
  会先判断是否实现接口，实现会强制转换类型，直接调用方法

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/aee87d938b7a4163b7a3b2823f0328fa.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwqDml7blhYnmuIXmtYXjgbTorrjkvaDlronnhLY=,size_20,color_FFFFFF,t_70,g_se,x_16)

### 6.Bean的初始化和Bean实例化区别是什么？

- Bean的实例化：
  userService -----> 推断构造方法 -----> 对象-----
  简单说就是通过构造方法得到一个对象

- Bean的初始化：
  userService -----> -----初始化（InitializingBean）-----
  简单说就是调用InitializingBean接口的初始化方法 或者 通过@PostConstruct去指定一个初始化的方法。

  总而言之，可以理解为Bean的初始化其实就是去执行实例化出来的对象中的某个方法，至于在初始化的方法里面实现什么逻辑，spring是不管的。如果在初始化方法中抛了异常出来，spring也会抛异常出来，在创建bean的过程中出异常了，那么会导致整个bean对象创建失败

### 7.初始化后

spring中主要创建bean的一个流程：

**userService -----> 推断构造方法 -----> 对象-----> 依赖注入----->初始化前（@PostConstruct）----->初始化（InitializingBean）----->初始化后（AOP）----->放入单例池map ----->spring内部一些其他步骤----->Bean 对象**

假设：userService这个bean需要进行AOP，AOP完成会通过动态代理产生一个代理对象，我们把之前产生的对象称为普通对象。

那最终放入单例池中的是普通对象还是代理对象呢？
如果没有AOP，不会产生代理对象，那么很明显，放入单例池的就是普通对象。
如果有AOP，最终是将代理对象放入单例map。既然userService有AOP的逻辑，那么通过getBean从单例池中拿bean对象的时候，就需要拿到有AOP逻辑的对象，因为会用到，，所以最后放入单例池的是代理对象
**userService -----> 推断构造方法 -----> 普通对象-----> 依赖注入----->初始化前（@PostConstruct）----->初始化（InitializingBean）----->初始化后（AOP）----->代理对象----->放入单例池map ----->Bean 对象**

