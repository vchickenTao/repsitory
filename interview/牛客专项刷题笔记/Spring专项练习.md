**<span style="font-size: 35px;">🍭 Spring专项练习</span>**

---

<!-- tabs:start -->

### **Spring**

#### 关于Spring AOP的术语，下列说法错误的是?

![关于Spring AOP的术语](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-27/bd87236e-c960-4a81-8c41-b0814329d244.png)
`切面`（aspect），是一个可以定义切点、各类通知和引入的内容，SpringAOP将通过它的信息来增强Bean的功能或者将对应的方法织入流程。

#### Spring创建Bean的方式有哪几种方式

Spring容器创建Bean对象的方法有三种方式，分别是：**用构造器来实例化，使用静态工厂方法实例化和使用实例工厂方法实例化**。

#### 关于IoC注解，下面说法错误的是

![关于IoC注解](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-27/ff646987-2b6f-40d8-95fb-eb7ec44bb42e.png)
@AutoWired注解还可以写在set方法、构造器上；@Qualifier注解也可以引用默认名称；@Bean注解可以用于装配任何Bean。

#### ＠Autowired注解的规则

```java
有如下接口：
public interface Student{
    public void introduce();
}

该接口有两个实现类：
@Component
public class StudentImplXH implements Student {
    @Override
    public void introduce() {
        System.out.println("我叫小华");
    }
}
@Component
public class StudentImplXM implements Student{
    @Override
    public void introduce() {
        System.out.println("我叫小明");
    }
}

测试类中代码如下：
@Autowired
private Student student;
@Test
void StudentTest(){
    student.introduce();
运行测试代码，控制台会输出什么结果？
```

**解答：**

＠Autowired注解提供这样的规则，首先根据**类型**找到对应的Bean，如果对应类型的Bean不是唯一的，那么就根据**属性名称**和**Bean的名称**进行匹配。如果匹配得上，就会使用该Bean。如果还无法匹配，就会抛出异常。即先byType再byName。故本题程序发生异常。

#### 属于Spring容器的类有

Spring提供了众多容器类，最常用的有**BeanFactory**和**ApplicationContext**。

#### 下列关于Spring事务管理的描述中，错误的是

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-02/f24844ee-0136-4d3a-855f-84f883c0eaaf_Spring事务1.png)

**解答：**

在有些场景下，我们需要获取事务的状态，是执行成功了还是失败回滚了，那么使用声明式事务就不够用了，需要编程式事务。

#### Spring IoC注入方式

- 基于属性注入
- 基于构造方法注入
- 基于setter方法注入

#### 下列关于@Transactional注解的说法中，错误的是

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-02/c0bed074-b4e6-4bfc-8ad3-98693951fb12_@Transactional注解1.png)

**解答：**

>  @Transactional可以作用在类上，代表这个类的所有公共非静态方法都将启用事务。

#### Spring在TransactionDefinition接口中规定了7种类型的事务传播行为

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-02/920334fc-cac8-4780-8c12-3d41294ac838_Spring在TransactionDefinition接口中规定了7种类型的事务传播行为.jpg)

#### Spring AOP支持的通知类型

> 前置通知、后置通知、环绕通知、返回通知、异常通知

#### Spring中Bean作用域

bean的作用域由@scope注解来修改，该注解有五个不同的取值，分别是：singleton、prototype、request、session、global-session。

1. singleton，在每一个Spring容器中，一个Bean定义只有一个对象实例（默认为singleton）
2. prototype，允许Bean的定义可以被实例化任意次（每次调用都创建一个一个实例）
3. request，在一次HTTP请求中，每个Bean定义对应一个实例。该作用域仅在基于Web的Spring上下文（例如SpringMVC）中才有效
4. session，在一个HTTP Session中，每个Bean定义对应一个实例。该作用域仅在基于Web的Spring上下文（例如SpringMVC）中才有效
5. global-session，在一个全局HTTP Session中，每个Bean定义对应一个实例。该作用域仅在基于Web的Spring上下文（例如SpringMVC）中才有效

#### 可以管理Spring Bean的生命周期的注解有

- @PostContruct Bean初始化
- @PreDestroy Bean销毁

#### 解决使用＠Autowired注解时产生的”歧义性“

当发现有多种类型的Bean时，**@Primary**注解会通知IoC容器优先使用它所标注的Bean进行注入；**@Quelifier**注解可以与@AutoWired注解组合使用，达到通过类型和名称一起筛选Bean的效果。



### **SpringMVC**

#### Spring MVC的数据模型

在Spring MVC中，**Model、ModelMap、ModelAndView**都可以作为数据模型对象，以上述类型作为控制器的方法参数时，Spring MVC会自动实例化这些类型。**ModelAttribute是注解**，用于定义控制器方法执行之前，对数据模型的操作。

#### 下列选项中，哪一个不是Spring MVC的核心组件

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-02/34b3b18f-006a-47ed-8ca0-0deadd21ff0d_SpringMVC核心组件.png)

**解答：**

```html
SpringFactoriesLoader是Spring Boot的组件，不是Spring MVC的组件。
```

#### SpringMVC的核心组件：

1:前端控制器（DispatcherServlet）
本质上是一个Servlet，相当于一个中转站，所有的访问都会走到这个Servlet中，再根据配置进行中转到相应的Handler中进行处理,获取到数据和视图后，在使用相应视图做出响应。

2:处理器映射器(HandlerMapping)
请求派发，本质上就是一段映射关系，将访问路径和对应的Handler存储为映射关系，在需要时供前端控制器查阅。

3:处理器适配器(HandlerAdapter)
本质上是一个适配器，可以根据要求找到对应的Handler来运行。前端控制器通过处理器映射器找到对应的Handler信息之后，将请求响应和对应的Handler信息交由处理器适配器处理，处理器适配器找到真正handler执行后，将结果即model和view返回给前端控制器

4:视图解析器(ViewResolver)
本质上也是一种映射关系，可以将视图名称映射到真正的视图地址。前端控制器调用处理器适配完成后得到model和view，将view信息传给视图解析器得到真正的view。

5:视图渲染(View)
本质上就是将handler处理器中返回的model数据嵌入到视图解析器解析后得到的jsp页面中，向客户端做出响应

**核心组件工作流程图：**

![**核心组件工作流程图](https://img-blog.csdnimg.cn/4f7001f338484ba3946748a55e83754f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Yuk6Ieq55yB,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 关于Spring MVC拦截器，下列说法错误的是

```html
A.开发Spring MVC拦截器，需实现WebMvcConfigurer接口。
B.preHandle方法在Controller之前执行，若返回false，则终止执行后续的请求。
C.postHandle方法在Controller之后、模板之前执行。
D.afterCompletion方法在模板之后执行。
```

**解答：**

```html
选A  
拦截器是SpringMVC中的一个核心应用组件,主要用于处理多个 Controller的共性问题.当我们的请求由DispatcherServlet派发 到具体Controller之前首先要执行拦截器中一些相关方法,在这些 方法中可以对请求进行相应预处理(例如权限检测,参数验证),这些方法可以决定对这个请求进行拦截还是放行。
服务器一启动,就会创建拦截器对象,对拦截器配置延迟加载是没有用的，拦截器是单例的,整个过程,拦截器只有一个实例对象,拦截器需要实现HandleInterceptor接口,或者继承HandlerInterceptorAdaptor抽象类; 
HandlerInterceptor接口的三个方法: 
1.preHandle() 是拦截器最先执行的方法,是在请求到达Controller之前执行的,其实就是拦截器用于拦截请求的，三个参数,分别是request,response,handelr就是这个请求要去找的后端处理器Controller.方法的返回值是bloolean类型,如果返回为false,就说明请求在此终结,不能执行后面的代码了.如果返回值为true,那么这个拦截器就要放行,将请求交给后端处理器Controller. 
2.postHandle() 这个方法,是在后端控制器controller处理完请求之后,就执行的,这个方法,多了一个参数,ModelAndView，后端控制器controller处理请求可能需要返回页面和数据,所以会多一个ModelAndView,但是这个方法,是在渲染页面之前执行的,渲染热面是交个前端控制器来完成的. 
3.afterCompletion() 拦截器最后执行的方法
```

#### 下列关于Spring MVC注解的说法中，正确的是

```html
@ControllerAdvice用于定义控制器的通知类，它可以对指定的控制器进行增强。
@InitBinder用于定义控制器参数绑定规则，如转换规则，它会在参数转换之前执行。
@ExceptionHandler用于定义控制器发生异常后的操作。
@ModelAttribute用于定义控制器方法执行之前，对数据模型的操作。
```

#### SpringMVC的处理过程

首先控制器接受用户的请求，并决定应该调用哪个模型来进行处理，然后模型用业务逻辑来处理用户的请求并返回数据，最后控制器用相应的视图格式化模型返回的数据，并通过表示层呈现给用户。

#### 属于Spring MVC的注解的有

```html
@RequestMapping
@RequestParam
@RequestBody
@PathVariable
```

#### 在Spring MVC中，若要实现上传功能，则需要使用的核心组件是

在Spring MVC中实现上传功能，主要依赖**MultipartHttpServletRequest**从读取请求中的文件，然后对读取到的**MultipartFile**类型进行处理。



### **SpringBoot**

#### 属于Spring Bean的作用域的是

Spring容器中Bean包含五种作用域：singleton(默认)、prototype、request、session、globalSession。

#### SpringApplication调用的run方法作用包括

```html
SpringApplication调用的run方法执行流程如下：

1. 初始化监听器，以及添加到SpringApplication的自定义监听器。
2. 发布ApplicationStartedEvent事件，如果想监听ApplicationStartedEvent事件，你可以这样定义：public class ApplicationStartedListener implements ApplicationListener，然后通过SpringApplication.addListener(..)添加进去即可。

3. 装配参数和环境，确定是web环境还是非web环境。

4. 装配完环境后，就触发ApplicationEnvironmentPreparedEvent事件。

5. 如果SpringApplication的showBanner属性被设置为true，则打印启动的Banner。

6. 创建ApplicationContext，会根据是否是web环境，来决定创建什么类型的ApplicationContext。

7. 装配Context的环境变量，注册Initializers、beanNameGenerator等。

8. 发布ApplicationPreparedEvent事件。

9. 注册springApplicationArguments、springBootBanner，加载资源等

10. 遍历调用所有SpringApplicationRunListener的contextLoaded()方法。

11. 调用ApplicationContext的refresh()方法,装配context beanfactory等非常重要的核心组件。

12. 查找当前ApplicationContext中是否注册有CommandLineRunner，如果有，则遍历执行它们。

13. 发布ApplicationReadyEvent事件，启动完毕，表示服务已经可以开始正常提供服务了。通常我们这里会监听这个事件来打印一些监控性质的日志，表示应用正常启动了。
```

#### 关于@EnableAutoConfiguration注解说法正确的是

@EnableAutoConfiguration由@SpringBootApplication引入，它的主要功能是启动Spring应用程序上下文时进行自动配置，它会尝试猜测并配置项目可能需要的Bean。从源代码得知@Import是@EnableAutoConfiguration注解的组成部分，也是自动配置功能的核心实现者。

<!-- tabs:end -->
