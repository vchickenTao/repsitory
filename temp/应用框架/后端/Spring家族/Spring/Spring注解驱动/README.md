# Spring的注解开发

Spring原始注解主要是替代\<Bean>的配置。

**原始注解**

| **注解**       | **说明**                                       |
| -------------- | ---------------------------------------------- |
| @Component     | 使用在类上用于实例化Bean                       |
| @Controller    | 使用在web层类上用于实例化Bean                  |
| @Service       | 使用在service层类上用于实例化Bean              |
| @Repository    | 使用在dao层类上用于实例化Bean                  |
| @Autowired     | 使用在字段上用于根据类型依赖注入               |
| @Qualifier     | 结合@Autowired一起使用用于根据名称进行依赖注入 |
| @Resource      | 相当于@Autowired+@Qualifier，按照名称进行注入  |
| @Value         | 注入普通属性                                   |
| @Scope         | 标注Bean的作用范围                             |
| @PostConstruct | 使用在方法上标注该方法是Bean的初始化方法       |
| @PreDestroy    | 使用在方法上标注该方法是Bean的销毁方法         |

**新注解**

| **注解**        | **说明**                                                     |
| --------------- | ------------------------------------------------------------ |
| @Configuration  | 用于指定当前类是一个 Spring 配置类，当创建容器时会从该类上加载注解 |
| @ComponentScan  | 用于指定 Spring 在初始化容器时要扫描的包。  作用和在 Spring 的 xml 配置文件中的  <context:component-scan base-package="com.tsuiraku"/>一样 |
| @Bean           | 使用在方法上，标注将该方法的返回值存储到 Spring 容器中       |
| @PropertySource | 用于加载xxx.properties 文件中的配置                          |
| @Import         | 用于导入其他配置类                                           |



使用注解进行开发时，需要在`applicationContext.xml`中配置组件扫描，作用是指定哪个包及其子包下的Bean需要进行扫描以便识别使用注解配置的类、字段和方法。

```xml
<!-- 注解的组件扫描 -->
<!-- 基于XML -->
<!-- 基于注解：@ComponentScan -->
<context:component-scan base-package="com.tsuiraku"></context:component-scan
```



# IOC容器

## 组件注册

### @Configuration

给容器中注册组件。

```java
@Configurationpublic class MyConfig{  
  @Bean("person01") 
   // 容器中注册一个bean；类型为返回值的类型；id默认是用方法名作为id；  
   public Person person(){		
     return new Person("tsuiraku", 21);	  
   }
}
```

```xml
<!-- 基于XML -->
<bean id='person01' class="com.tsuiraku.person">	
  <property name="tsuiraku" value="21"></property>
</bean>
```



### @ComponentScan

自动扫描组件。

```java
@Configuration
@ComponentScan("com.tsuiraku") 
// 指定需要扫描的包
public class MyConfig{}
```

```xml
<!-- 包扫描：@Controller @Service @Repository @Component -->
<!-- 基于XML -->
<context:component-scan base-package="com.tsuiraku"></context:component-scan>
```



#### TypeFilter

通过 TypeFilter 来指定过滤规则（`includeFilters` or `excludeFilters`）。

- <u>**FilterType.ANNOTATION**</u>

  按照注解进行包含或者排除。

  ```java
  @ComponentScan(value="com.tsuiraku", includeFilters={		
    /*		 
    * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等		 
    * classes：我们需要Spring在扫描时，只包含@Controller注解标注的类		 
    */		
    @Filter(type=FilterType.ANNOTATION, classes={Controller.class})}, useDefaultFilters=false) // value指定要扫描的包
  ```

- **<u>FilterType.ASSIGNABLE_TYPE</u>**

  按照给定的类型进行包含或者排除。

  ```java
  @ComponentScan(value="com.tsuiraku", includeFilters={		
    /*		 
    * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等		 
    */		
    // 只要是 BookService 这种类型的组件都会被加载到容器中，不管是它的子类还是什么它的实现类。
    @Filter(type=FilterType.ASSIGNABLE_TYPE, classes={BookService.class})}, useDefaultFilters=false) // value指定要扫描的包
  ```

- **<u>FilterType.ASPECTJ</u>**

  按照ASPECTJ表达式进行包含或者排除。

  ```java
  @ComponentScan(value="com.tsuiraku", includeFilters={		
    /*		 
    * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等		 
    */		@Filter(type=FilterType.ASPECTJ, classes={AspectJTypeFilter.class})}, useDefaultFilters=false) // value指定要扫描的包
  ```

- **<u>FilterType.REGEX</u>**

  按照正则表达式进行包含或者排除。

  ```java
  @ComponentScan(value="com.tsuiraku", includeFilters={		
    /*		 
    * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等		 
    */		@Filter(type=FilterType.REGEX, classes={RegexPatternTypeFilter.class})}, useDefaultFilters=false) // value指定要扫描的包
  ```

- **<u>FilterType.CUSTOM</u>**

  **<u>自定义规则</u>**进行包含或者排除。

  ```java
  public class MyTypeFilter implements TypeFilter {
  
  	/**
  	 * 参数：
  	 * metadataReader：读取到的当前正在扫描的类的信息
  	 * metadataReaderFactory：可以获取到其他任何类的信息的（工厂）
  	 */
  	@Override
  	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
  		// 获取当前类注解的信息
  		AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
  		// 获取当前正在扫描的类的类信息，比如说它的类型是什么啊，它实现了什么接口啊之类的
  		ClassMetadata classMetadata = metadataReader.getClassMetadata();
  		// 获取当前类的资源信息，比如说类的路径等信息
  		Resource resource = metadataReader.getResource();
  		// 获取当前正在扫描的类的类名
  		String className = classMetadata.getClassName();
  		System.out.println("--->" + className);
  		
  		// 现在来指定一个规则
  		if (className.contains("er")) {
  			return true; // 匹配成功，就会被包含在容器中
  		}
  		
  		return false; // 匹配不成功，所有的bean都会被排除
  	}
  
  }
  ```
  
  ```java
  @ComponentScans(value={
  		@ComponentScan(value="com.tsuiraku", includeFilters={
  				/*
  				 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
  				 */
  				// 指定新的过滤规则，这个过滤规则是我们自个自定义的，过滤规则就是由我们这个自定义的MyTypeFilter类返回true或者false来代表匹配还是没匹配
  				@Filter(type=FilterType.CUSTOM, classes={MyTypeFilter.class})
  		}, useDefaultFilters=false) // value指定要扫描的包
  })
  @Configuration
  public class MainConfig {
  	@Bean("person")
  	public Person person01() {
  		return new Person("tsuiraku", 20);
  	}
  }
  ```
  
  

### @Scope

设置组件的作用域。

```java
@Configurationpublic class MyConfig{  
  @Bean   
  @Scope("prototype") 
  // 设置Person对象的作用域修改成prototype  
  public Person person(){		
    return new Person("tsuiraku", 21);	  
  }
}
```



#### 单实例 bean 注意的事项

单实例 bean 是整个应用所共享的，所以需要考虑到线程安全问题。SpringMVC 中的 Controller 默认是单例的，有些开发者在 Controller中创建了一些变量，那么这些变量实际上就变成共享的了，Controller 又可能会被很多线程同时访问，这些线程并发去修改 Controller 中的共享变量，此时很有可能会出现数据错乱的问题，所以使用的时候需要特别注意。

```java
@Configuration
public class MainConfig {		
  @Bean("person")	
  public Person person() {		
    return new Person("tsuiraku", 21);
  }
}

##########################################################################################@S	
@SuppressWarnings("resource")
@Test
public void test() {    
  AnnotationConfigApplicationContext applicationContext = new 
    AnnotationConfigApplicationContext(MainConfig.class);        
  // 获取到的这个Person对象默认是单实例的，因为在IOC容器中给我们加的这些组件默认都是单实例的 
  // 所以说在这儿我们无论多少次获取，获取到的都是我们之前new的那个实例对象    
  Person person = (Person) applicationContext.getBean("person");    
  Person person2 = (Person) applicationContext.getBean("person");    System.out.println(person == person2); // true
}
```

**单实例 bean 创建的时间**

Spring 容器在启动时，将单实例组件实例化之后，会即刻加载到 Spring 容器中，以后每次从容器中获取组件实例对象时，都是直接返回相应的对象，而不必再创建新的对象了。



#### 多实例 bean 注意的事项

多实例 bean 每次获取的时候都会重新创建，如果这个 bean 比较复杂，创建时间比较长，那么就会影响系统的性能。

```java
@Configuration
public class MainConfig {		
  @Scope("prototype")	
  @Bean("person")	
  public Person person() {		
    return new Person("tsuiraku", 21);	
  }
}

##########################################################################################

@SuppressWarnings("resource")
@Test
public void test() {    
  AnnotationConfigApplicationContext applicationContext = new 
    AnnotationConfigApplicationContext(MainConfig.class);        
  Person person = (Person) applicationContext.getBean("person");    
  Person person2 = (Person) applicationContext.getBean("person");    System.out.println(person == person2); // false
}
```

**多实例 bean 创建的时间**

当向 Spring 容器中获取 Person 实例对象时，Spring 容器才会实例化 Person 对象，再将其加载到 Spring 容器中。



### @Lazy

懒加载。懒加载指在 Spring 容器启动的时候，先不创建对象，在第一次使用（获取）bean 的时候再来创建对象，并进行一些初始化。（<u>**懒加载，也称延时加载，仅针对单实例 bean 生效**</u>）

```java
@Configuration
public class MainConfig {		
  @Lazy	
  @Bean("person")	
  public Person person() {    
    System.out.println("获取person对象");		
    return new Person("tsuiraku", 21);	
  }
}

##########################################################################################
  @SuppressWarnings("resource")
  @Test
  public void test() {    
  AnnotationConfigApplicationContext applicationContext = new 
    AnnotationConfigApplicationContext(MainConfig.class);        
  Person person = (Person) 
    applicationContext.getBean("person"); // 此时创建对象 bean
}
```



### @Conditional

按照一定的条件进行判断，满足条件向容器中注册 bean，不满足条件就不向容器中注册 bean。

**<u>需要实现 Condition 接口</u>**

```java
@Configurationpublic class MainConfig {		
  @Bean("person")	
  public Person person() {		
    return new Person("tsuiraku", 21);	
  }	
  @Conditional({WindowsCondition.class})	
  @Bean("bill")	public Person person01() {		
    return new Person("Bill Gates", 62);	
  }		
  @Conditional({LinuxCondition.class})	
  @Bean("linus")	public Person person02() {		
    return new Person("linus", 48);	
  }
}
```

```java
/**
* 判断操作系统是否是Linux系统
*
*/
public class LinuxCondition implements Condition {

    /**
    * ConditionContext：判断条件能使用的上下文（环境）
    * AnnotatedTypeMetadata：当前标注了@Conditional注解的注释信息
    */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 判断操作系统是否是Linux系统
        
        // 1. 获取到bean的创建工厂（能获取到IOC容器使用到的BeanFactory，它就是创建对象以及进行装配的工厂）
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        // 2. 获取到类加载器
        ClassLoader classLoader = context.getClassLoader();
        // 3. 获取当前环境信息，它里面就封装了我们这个当前运行时的一些信息，包括环境变量，以及包括虚拟机的一些变量
        Environment environment = context.getEnvironment();
        // 4. 获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();
        
        boolean definition = registry.containsBeanDefinition("person");

        String property = environment.getProperty("os.name");
        if (property.contains("linux")) {
            return true;
        }
     
        return false;
    }

}

```

```java
@Test
public void test06() {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
    // 查看IOC容器中Person这种类型的bean都有哪些
    String[] namesForType = applicationContext.getBeanNamesForType(Person.class);
    ConfigurableEnvironment environment = applicationContext.getEnvironment(); // 拿到IOC运行环境
    // 动态获取坏境变量的值，例如操作系统的名字
    String property = environment.getProperty("os.name"); // 获取当前操作系统的名字
    System.out.println(property);
    
    for (String name : namesForType) {
        System.out.println(name);
    }
    
    Map<String, Person> persons = applicationContext.getBeansOfType(Person.class); // 找到这个Person类型的所有bean
    System.out.println(persons);
}
```



### @Import

> 给容器中注册组件的方法
>
> - 包扫描 + 组件标注注解（@Controller/@Service/@Repository/@Component）；
> - @Bean（导入第三方包里面的组件）；
> - @Import（快速导入一个组件）；
> - Spring 提供的 FactoryBean。



#### @Import

利用 `@Import({})` 快速导入组件。

```java
public class Color {}
```

```java
@Configuration
@Import(Color.class) // @Import快速地导入组件，id默认是组件的全类名
public class MainConfig2 {}
```



#### ImportSelector

自定义需要导入容器的组件。

<u>**需要实现 ImportSelector 接口。**</u>

**<u>重写 selectImports 方法。</u>**

```java
@Configuration
@Import(MyImportSelector.class)
public class MainConfig {}
```

```java
/** 
* 自定义逻辑，返回需要导入的组件 
* 
*/
public class MyImportSelector implements ImportSelector {	
  // 返回值：就是要导入到容器中的组件的全类名	
  // AnnotationMetadata：当前已经标注@Import注解的类的所有注解信息，并且不仅能获取到@Import注解里面的信息，还能获取到其他注解的信息	
  @Override	public String[] selectImports(AnnotationMetadata importingClassMetadata) { 				// 方法不要返回null值，否则会报空指针异常		
    return new String[]{}; // 可以返回一个空数组	
  }
}
```



#### ImportBeanDefinition

需要配合 @Configuration 和 @Import 注解进行使用。@Configuration 注解定义 Java 格式的 Spring 配置文件，@Import 注解导入实现了 ImportBeanDefinitionRegistrar 接口的类。

**<u>需要实现 ImportBeanDefinitionRegistrar 接口。</u>**

**<u> 重写registerBeanDefinitions 方法。</u>**

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {	
  /**	
  * AnnotationMetadata：当前类的注解信息	 
  * BeanDefinitionRegistry：BeanDefinition注册类	 
  * 我们可以通过调用 BeanDefinitionRegistry 接口中的 registerBeanDefinition 方法，手动注册所有需要添加到容器中的bean	 
  */	
  @Override	
  public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {    
    boolean definition = registry.containsBeanDefinition("com.tsuiraku.bean.Red");		
    boolean definition2 = registry.containsBeanDefinition("com.tsuiraku.bean.Bule");    
    // 如果当前两种bean都存在		
    if (definition && definition2) {			
      // 指定bean的定义信息，包括bean的类型、作用域等等			
      // RootBeanDefinition 是 BeanDefinition 接口的一个实现类			
      RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class); 
      // bean的定义信息			
      // 注册一个新的bean，并且指定bean的名称			
      registry.registerBeanDefinition("rainBow", beanDefinition);		
    }	
  }
}
```

```java
@Configuration
@Import(MyImportBeanDefinitionRegistrar.class) // @Import快速地导入组件，id默认是组件的全类名
public class MainConfig {}
```



### FactoryBean

FactoryBean 向 Spring 容器中注册bean。

**<u>需要实现 FactoryBean 接口。</u>**

<u>**重写 getObject 和 getObjectType 方法。**</u>

```java
/** 
* 创建一个 Spring 定义的 FactoryBean 
* T（泛型）：指定我们要创建什么类型的对象 
*  
*/
public class ColorFactoryBean implements FactoryBean<Color> {	
  // 返回一个Color对象，这个对象会添加到容器中	
  @Override	
  public Color getObject() throws Exception {		
    // TODO Auto-generated method stub		
    System.out.println("ColorFactoryBean...getObject...");		
    return new Color();	
  }	
  @Override	
  public Class<?> getObjectType() {		
    // TODO Auto-generated method stub		
    return Color.class; // 返回这个对象的类型	
  }	
  // 控制实例在容器中类型	
  // 如果返回true，那么代表这个bean是单实例，在容器中只会保存一份；	
  // 如果返回false，那么代表这个bean是多实例，每次获取都会创建一个新的bean	
  @Override	public boolean isSingleton() {		
    // TODO Auto-generated method stub		
    return false;	
  }
}
```

```java
@Configuration
@Import(MyImportBeanDefinitionRegistrar.class)
public class MainConfig {  
  ...	
  @Bean	public ColorFactoryBean colorFactoryBean() {		
    return new ColorFactoryBean();	
  }
}
```

虽然在 MainConfig 中使用 @Bean 注解向 Spring 容器中注册的是 ColorFactoryBean 对象。但是通过调用 ` context.getBean("colorFactoryBean")` 获取得到 Bean 发现返回类型为：`com.tsuiraku.bean.Color` 。

也可以获取 ColorFactoryBean 对象本身，通过调用 ` context.getBean("&colorFactoryBean")` 。



## 生命周期

> **<u>生命周期：创建 - - - 初始化 - - - 销毁</u>**
>
> 指定初始化和销毁
>
> - 通过 @Bean 指定 init-method 和 destroy-method；
> - bean 通过实现 InitiallizingBean 接口 和 DisposableBean 接口；
> - @PostConstruct 和 @PreDestory；
> - @BeanPostProcessor。



**<u>bean 的实例化</u>**

调用 bean 的构造方法，我们可以在 bean 的无参构造方法中执行相应的逻辑。



**<u>bean 的初始化</u>**

在初始化时，可以通过 **<u>BeanPostProcessor</u>** 的 `postProcessBeforeInitialization()` 方法和`postProcessAfterInitialization()` 方法进行拦截，执行自定义的逻辑；

通过**<u>@PostConstruct</u>** 注解、**<u>InitializingBean</u>** 和 <u>**init-method**</u> 来指定 bean 初始化前后执行的方法，在该方法中可以执行自定义的逻辑。



**<u>bean 的销毁</u>**：可以通过 **<u>@PreDestroy</u>** 注解、**<u>DisposableBean</u> **和 <u>**destroy-method**</u> 来指定 bean 在销毁前执行的方法，在该方法中可以执行自定义的逻辑。



### init-method && destroy-method 

```java
@Component
public class Car {	
  public Car() {		
    System.out.println("car constructor...");	
  }		
  public void init() {		
    System.out.println("car ... init...");	
  }		
  public void destroy() {		
    System.out.println("car ... destroy...");	
  }
}

##########################################################################################  
  
  
public class MainConfigOfLifeCycle {	
  @Bean(initMethod = "init", destroyMethod = "destroy")	
  public Car car() {		
    return new Car();	
  }
}
```

**<u>bean 对象的初始化方法调用的时机</u>**

对象创建完成，如果对象中存在一些属性，并且这些属性也都赋好值之后，那么就会调用 bean 的初始化方法。

- 对于**单实例 bean** 来说，在 Spring 容器创建完成后，Spring容器会自动调用 bean 的初始化方法；
- 对于**多实例 bean **来说，在每次获取 bean 对象的时候，调用 bean 的初始化方法。

**<u>bean 对象的销毁方法调用的时机</u>**

- 对于**单实例 bean **来说，在容器关闭的时候，会调用bean的销毁方法；
- 对于**多实例 bean** 来说，Spring 容器不会管理这个bean，也就不会自动调用这个 bean 的销毁方法了。



### InitiallizingBean && DisposableBean

- **<u>实现 afterPropertiesSet 方法</u>**
- <u>**实现 InitiallizingBean 和 DisposableBean**</u>

```java
@Component
public class Cat implements InitializingBean, DisposableBean {		
  public Cat() {		
    System.out.println("cat constructor...");	
  }	
  /**	 
  * 会在容器关闭的时候进行调用	 
  */	
  @Override	public void destroy() throws Exception {		
    // TODO Auto-generated method stub		
    System.out.println("cat destroy...");	}	
  /**	 
  * 会在bean创建完成，并且属性都赋好值以后进行调用	 
  */	
  @Override	public void afterPropertiesSet() throws Exception {		
    // TODO Auto-generated method stub		
    System.out.println("cat afterPropertiesSet...");	
  }
}
```

```java
@ComponentScan("com.tsuiraku.bean")
@Configuration
public class MainConfigOfLifeCycle {	
  @Bean(initMethod="init", destroyMethod="destroy")	
  public Car car() {		
    return new Car();	
  }
}
```

- Spring 为 bean 提供了两种初始化的方式，实现 InitializingBean 接口（也就是要实现该接口中的`afterPropertiesSet()` 方法），或者在配置文件或 @Bean 注解中通过 `init-method` 来 指定，两种方式可以同时使用。
- 实现 InitializingBean 接口是直接调用 `afterPropertiesSet()` 方法，与通过反射调用 `init-method` 指定的方法相比，效率相对来说要高点。但是 `init-method` 方式消除了对 Spring 的依赖。
- 如果调用 `afterPropertiesSet()` 方法时出错，那么就不会调用 `init-method` 指定的方法了。



### @PostConstruct && @PreDestory

- @PostConstruct 在 bean 创建完成并且完成属性赋值，来执行初始化方法；
- @PreDestory 在容器销毁 bean 之前通知进行清理工作。

```java
@Component
public class Dog {	
  public Dog() {		
  System.out.println("dog constructor...");	
}		
  // 在对象创建完成并且属性赋值完成之后调用	
  @PostConstruct	
  public void init() {		
    System.out.println("dog...@PostConstruct...");	
  }		
  // 在容器销毁（移除）对象之前调用	
  @PreDestroy	
  public void destory() {		
    System.out.println("dog...@PreDestroy...");	
  }
}
```

```java
@ComponentScan("com.tsuiraku.bean")
@Configuration
public class MainConfigOfLifeCycle {}
```



### @BeanPostProcessor

bean 的后置处理器，在 bean 初始化之前进行一些处理工作。

**<u>实现 BeanPostProcessor 接口</u>**

> - postProcessBeforeInitialization：在 Spring 容器中的 bean 初始化后执行
> - postProcessAfterInitialization：在 Spring 容器中的 bean 初始化后执行

```java
/** 
* 后置处理器，在初始化前后进行处理工作 
* 
*/
@Component 
// 将后置处理器加入到容器中
public class MyBeanPostProcessor implements BeanPostProcessor {	
  @Override	
  public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {		
    // TODO Auto-generated method stub		
    System.out.println("postProcessBeforeInitialization..." + beanName + "=>" + bean);		
    return bean;	
  }	
  @Override	
  public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {		
    // TODO Auto-generated method stub		
    System.out.println("postProcessAfterInitialization..." + beanName + "=>" + bean);		
    return bean;	
  }
}
```



## 组件赋值

### @Value

- 注入基本数值类型

```java
@Value("tsuiraku")
private String name; // 注入普通字符串
```

- 注入操作系统属性

```java
@Value("#{systemProperties['os.name']}")
private String systemPropertiesName; // 注入操作系统属性
```

- 注入 SpEL 表达式或者 #{}

```java
@Value("#{ T(java.lang.Math).random() * 100.0 }")
private double randomNumber; //注入SpEL表达式结果
```

- 注入配置文件中的值

```java
@Value("${person.name}")
private String username; // 注入perproties中的值
```

- 注入文件资源

```java
@Value("classpath:/config.properties")
private Resource resourceFile; // 注入文件资源
```



### @PropertySource

可以将 `properties` 配置文件中的 `{key:value}` 存储到 Spring 的 Environment 中，也可以使用 @Value 注解用 `${}` 占位符为 bean 的属性注入值。



```java
// 配置文件
person.nickName=tsuiraku

########################################################################################## 

// 类
public class Person {	
    private String nickName; 
  }
```



**使用XML配置文件方式获取值**

```xml
<context:property-placeholder location="classpath:person.properties" />
<!-- 注册组件 -->
<bean id="person" class="com.tsuiraku.bean.Person">    
  <property name="nickName" value="${person.nickName}"> </property>
</bean>
```



**使用注解方式获取值**

```java
@PropertySource(value={"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValues {
	@Bean
	public Person person() {
		return new Person();
	}
}
```

```java
@Value("${person.nickName}")
private String nickName; // nickName=tsuiraku
```



**使用 Environment 获取值**

```java
@Test
public void test01() {  	
  ...	  	
  // environment    
  ConfigurableEnvironment environment = applicationContext.getEnvironment();   
  String property = environment.getProperty("person.nickName");    
  System.out.println(property);        
  // 关闭容器    
  applicationContext.close();
}
```



## 自动装配

### @Autowired

> @Autowired 注解可以对类成员变量、方法和构造函数进行标注，完成自动装配的工作；
>
> @Autowired 注解可以放在构造器、方法、参数等。
>
> ```java
> @Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
> ```



在使用 @Autowired 注解之前，我们对一个 bean 配置属性时，是用如下 XML 配置文件的形式进行配置的。

```xml
<!-- 基于XML -->
<property name="属性名" value=" 属性值"/>
```



- @Autowired 注解默认是优先按照类型去容器中找对应的组件，相当于是调用了如下这个方法

  ```java
  applicationContext.getBean(类名.class);
  ```

- 如果找到多个相同类型的组件，那么是将属性名称作为组件的 id，到 IOC 容器中进行查找，这时就相当于是调用了如下这个方法

  ```java
  applicationContext.getBean("组件的id");
  ```



#### 当存在自定义组件需要使用 Spring 底层的组件

此时无法通过自动注入将 Spring 底层的组件进行注入。

**<u>自定义组件实现 XXXAware 接口</u>**

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/截屏2021-09-14 15.10.52.png" alt="截屏2021-09-14 15.10.52" style="zoom:30%;" />



### @Qualifier

@Autowired 是根据类型进行自动装配的，如果需要**按名称**进行装配，那么就需要配合 @Qualifier 注解来使用了。



### @Primary

Spring 进行自动装配的时候，默认选择首选的 bean。



### @Resource

类似 @Autowired 自动装配。<u>属于 Java 规范中的注解</u>。该注解默认按照名称进行装配，名称可以通过 name 属性进行指定，如果没有指定 name 属性，当注解写在字段上时，那么默认取字段名将其作为组件的名称在 IOC 容器中进行查找，如果注解写在setter方法上，那么默认取属性名进行装配。当找不到与名称匹配的 bean 时才按照类型进行装配。但是需要注意的一点是，如果 name 属性一旦指定，那么就只会按照名称进行装配。



### @Inject

<u>属于 Java 规范中的注解</u>。

需要引入相关依赖。

```xml
<dependency>   
  <groupId>javax.inject</groupId>   
  <artifactId>javax.inject</artifactId>   
  <version>1</version>
</dependency>
```



### @Profile

动态的激活和切换一系列组件的功能。

开发环境、测试环境、生产环境。

指定组件在哪个环境下才会被注册到容器中。

```properties
# 配置数据库
# dbconfig.properties
db.user=root
db.password=tsuiraku
db.driverClass=com.mysql.jdbc.Driver
```

```java
@PropertySource("classpath:/dbconfig.properties") // 加载外部的配置文件
@Configuration
public class MainConfigOfProfile implements EmbeddedValueResolverAware {
	
	@Value("${db.user}")
	private String user;
	
	private StringValueResolver valueResolver;
	
	private String dirverClass;
	
	@Profile("test")
	@Bean("testDataSource")
	public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws Exception {
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser(user);
		dataSource.setPassword(pwd);
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
		dataSource.setDriverClass(dirverClass);
		return dataSource;
	}
	 
	@Profile("dev") // 定义了一个环境标识，只有当dev环境被激活以后，我们这个bean才能被注册进来
	@Bean("devDataSource")
	public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws Exception {
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser(user);
		dataSource.setPassword(pwd);
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/ssm_crud");
		dataSource.setDriverClass(dirverClass);
		return dataSource;
	}
	
	@Profile("prod")
	@Bean("prodDataSource")
	public DataSource dataSourceProd(@Value("${db.password}") String pwd) throws Exception {
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser(user);
		dataSource.setPassword(pwd);
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/scw_0515");
		dataSource.setDriverClass(dirverClass);
		return dataSource;
	}

	@Override
	public void setEmbeddedValueResolver(StringValueResolver resolver) {
		this.valueResolver = resolver;
		dirverClass = valueResolver.resolveStringValue("${db.driverClass}");
	}

}

```



**如何启动不同的环境**

- 命令行参数启动环境

  ```
  -Dspring.profiles.active=test
  ```

- 代码方式启动环境

  ```java
  @Test
  public void test() {
  	// 1. 使用无参构造器创建一个IOC容器
  	AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
  	// 2. 在我们容器还没启动创建其他bean之前，先设置需要激活的环境（可以设置激活多个环境哟）
  	applicationContext.getEnvironment().setActiveProfiles("test");
  	// 3. 注册主配置类
  	applicationContext.register(MainConfigOfProfile.class);
  	// 4. 启动刷新容器
  	applicationContext.refresh();
  	
  	String[] namesForType = applicationContext.getBeanNamesForType(DataSource.class);
  	for (String name : namesForType) {
  		System.out.println(name);
  	}
  	
  	// 关闭容器
  	applicationContext.close();
  }
  ```

  

- @Profile 注解不仅可以标注在方法上，也可以标注在配置类上。

- 如果 @Profile 注解标注在配置类上，那么只有是在指定的环境的时候，整个配置类里面的所有配置才会生效。

- 如果一个 bean 上没有使用 @Profile 注解进行标注，那么这个 bean 在任何环境下都会被注册到 IOC 容器中。



# AOP

> <u>**AOP是指在程序的运行期间动态地将某段代码切入到指定方法、指定位置进行运行的编程方式。AOP的底层是使用动态代理实现的。**</u>



**简单示例**

> 定义一个业务逻辑类（MathCalculator），在业务逻辑运行的时候将日志进行打印（方法运行之前、方法运行结束后、方法出现异常）



```java
/** 
* 目标类 
* 
*/
public class MathCalculator {	
  public int div(int i, int j) {		
    System.out.println("MathCalculator...div...");		
    return i / j;	
  }	
}
```

**切面类**

> @Aspect ：告诉 Spring 容器当前是切面类。

AOP中的通知方法及其对应的注解与含义如下：

- 前置通知（对应的注解是 @Before）：在目标方法运行之前运行
- 后置通知（对应的注解是 @After）：在目标方法运行结束之后运行，无论目标方法是正常结束还是异常结束都会执行
- 返回通知（对应的注解是 @AfterReturning）：在目标方法正常返回之后运行
- 异常通知（对应的注解是 @AfterThrowing）：在目标方法运行出现异常之后运行
- 环绕通知（对应的注解是 @Around）：动态代理，我们可以直接手动推进目标方法运行

```java
/**
 * 切面类
 *
 */
public class LogAspects {
	
	// @Before：在目标方法（即div方法）运行之前切入，public int com.meimeixia.aop.MathCalculator.div(int, int)这一串就是切入点表达式，指定在哪个方法切入
	@Before("public int com.meimeixia.aop.MathCalculator.*(..)")
	public void logStart() {
		System.out.println("除法运行......@Before，参数列表是：{}");
	}
	
	// 在目标方法（即div方法）结束时被调用
	@After("public int com.meimeixia.aop.MathCalculator.*(..)")
	public void logEnd() {
		System.out.println("除法结束......@After");
	}
	
	// 在目标方法（即div方法）正常返回了，有返回值，被调用
	@AfterReturning("public int com.meimeixia.aop.MathCalculator.*(..)")
	public void logReturn() {
		System.out.println("除法正常返回......@AfterReturning，运行结果是：{}");
	}
	
	// 在目标方法（即div方法）出现异常，被调用
	@AfterThrowing("public int com.meimeixia.aop.MathCalculator.*(..)")
	public void logException() {
		System.out.println("除法出现异常......异常信息：{}");
	}

}

```

```java
/**
 * AOP：面向切面编程，其底层就是动态代理
 *      指在程序运行期间动态地将某段代码切入到指定方法指定位置进行运行的编程方式。
 *        
 */
@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAOP {
	
	// 将业务逻辑类（目标方法所在类）加入到容器中
	@Bean
	public MathCalculator calculator() {
		return new MathCalculator();
	}
	
	// 将切面类加入到容器中
	@Bean
	public LogAspects logAspects() {
		return new LogAspects();
	}
}
```

- **<u>@EnableAspectJAutoProxy：开启基于注解的 AOP 代理模式。</u>**



> [切点表达式](https://www.cnblogs.com/zhangxufeng/p/9160869.html)



## @EnableAspectJAutoProxy

在配置类上添加 @EnableAspectJAutoProxy 注解，便能够开启注解版的AOP功能。



@EnableAspectJAutoProxy 注解使用 @Import 注解给容器中引入了 AspectJAutoProxyRegister 组件。

![截屏2021-09-15 08.39.37](https://gitee.com/tsuiraku/typora/raw/master/img/截屏2021-09-15 08.39.37.png)



AspectJAutoProxyRegistrar 类实现了 ImportBeanDefinitionRegistrar 接口。

![截屏2021-09-15 08.40.03](https://gitee.com/tsuiraku/typora/raw/master/img/截屏2021-09-15 08.40.03.png)



> ImportBeanDefinitionRegistrar 接口实现将自定义的组件添加到IOC容器中。

**@EnableAspectJAutoProxy 注解使用 AspectJAutoProxyRegistrar 对象自定义组件，并将相应的组件添加到了IOC容器中。**

![截屏2021-09-15 08.40.20](https://gitee.com/tsuiraku/typora/raw/master/img/截屏2021-09-15 08.40.20.png)



**向 Spring 的配置类上添加 @EnableAspectJAutoProxy 注解之后，会向 IOC 容器中注册 AnnotationAwareAspectJAutoProxyCreator（后置处理器），翻译过来就叫注解装配模式的 AspectJ 切面自动代理创建器。**



组件的继承关系

```
AnnotationAwareAspectJAutoProxyCreator    ->AspectJAwareAdvisorAutoProxyCreator（父类）        ->AbstractAdvisorAutoProxyCreator（父类）            ->AbstractAutoProxyCreator（父类）                implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware（两个接口）
```



## AnnotationAwareAspectJAutoProxyCreator

############################# $to\space do$ #############################



# 声明式事务



**简单示例**

```java
@Repository
public class UserDao {		
  @Autowired	private JdbcTemplate jdbcTemplate;		
  public void insert() {		
    String sql = "insert into `tbl_user`(username, age) values(?, ?)";		
    String username = UUID.randomUUID().toString().substring(0, 5);		
    jdbcTemplate.update(sql, username, 19); 
    // 增删改都来调用这个方法	
  }
}

########################################################
  
@Service
public class UserService {	
    @Autowired	private UserDao userDao;		
    public void insertUser() {		
      userDao.insert();		
      System.out.println("插入完成...");	}	}


########################################################  
  
public class Test {		
    @Test	public void test() {	
      AnnotationConfigApplicationContext applicationContext = new 
        AnnotationConfigApplicationContext(TxConfig.class);				
      UserService userService = applicationContext.getBean(UserService.class);				
      userService.insertUser();				
      applicationContext.close();	
    }
  }
```

此时插入方法并没有事务特性。



**<u>如何开启事务管理</u>**

> 1. 在方法上配置 @Transactional；
> 2. 在配置类中开启事务管理 @EnableTransactionManagement；
> 3. 配置事务管理器，实现相应事务的接口。



## @Transactional

表明当前方法是一个事物方法。

```java
@Transactional
public void insertUser() {
	userDao.insert();
	// otherDao.other(); // 该方法中的业务逻辑势必不会像现在这么简单，肯定还会调用其他dao的方法
	System.out.println("插入完成...");
	
	int i = 10 / 0;
}

########################################################  

@EnableTransactionManagement // 它是来开启基于注解的事务管理功能的 
@ComponentScan("com.tsuiraku.tx")
@Configuration
public class TxConfig {

	// 注册c3p0数据源
	@Bean
	public DataSource dataSource() throws Exception {
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser("root");
		dataSource.setPassword("root");
		dataSource.setDriverClass("com.mysql.jdbc.Driver");
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
		return dataSource;
	}
	
	@Bean
	public JdbcTemplate jdbcTemplate() throws Exception {
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
		return jdbcTemplate;
	}
	
}
```



## @EnableTransactionManagement

开启基于注解的事物管理功能



@EnableTransactionManagement 注解使用 @Import 注解给容器中引入了 TransactionManagementConfigurationSelector 组件。

![截屏2021-09-15 09.37.10](https://gitee.com/tsuiraku/typora/raw/master/img/截屏2021-09-15 09.37.10.png)

TransactionManagementConfigurationSelector 类中继承 AdviceModeImportSelector。

![截屏2021-09-15 09.37.30](https://gitee.com/tsuiraku/typora/raw/master/img/截屏2021-09-15 09.37.30.png)

AdviceModeImportSelector 类实现了接口 ImportSelector。

![截屏2021-09-15 09.37.40](https://gitee.com/tsuiraku/typora/raw/master/img/截屏2021-09-15 09.37.40.png)

############################# $to\space do$ #############################



# 扩展原理

## BeanFactoryPostProcessor

BeanFactoryPostProcessor 的调用时机是在 BeanFactory 标准初始化之后。此时，所有的 bean 已经被定义保存到 BeanFactory，但是 bean 的实例还未创建。



编写自定义类 MyBeanFactoryPostProcessor 实现 BeanFactoryPostProcessor。测试 BeanFactoryPostProcessor。

```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {	
  @Override	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {		
    System.out.println("MyBeanFactoryPostProcessor...postProcessBeanFactory..."); 
    // 这个时候我们所有的bean还没被创建		                                                                              
    // 但是我们可以看一下通过Spring给我们传过来的这个beanFactory，我们能拿到什么		
    int count = beanFactory.getBeanDefinitionCount(); // 我们能拿到有几个bean定义		
    String[] names = beanFactory.getBeanDefinitionNames(); // 除此之外，我们还能拿到每一个bean定义的名字		
    System.out.println("当前BeanFactory中有" + count + "个Bean");		System.out.println(Arrays.asList(names));	
  }
}

########################################################
  
@ComponentScan("com.tsuiraku.ext")
@Configuration
public class Config {}
```



**原理**

1. IOC 容器创建对象；

2. invokeBeanFactoryPostProcessors(beanFactory)。

   如果找到所有的 BeanFactoryPostProcessor 并且执行方法

   1. 直接在 BeanFactory 中找到所有类型是 BeanFactoryPostProcessor 的组件，并且执行它们的方法；
   2. 在初始化创建其他组件前执行。



## BeanDefinationRegistryPostProcessor

BeanFactoryPostProcessor 的子接口。

调用时间在所有 bean 定义信息将要被加载，但是 bean 实例还未创建。



编写自定义类 MyBeanDefinitionRegistryPostProcessor，实现 BeanDefinitionRegistryPostProcessor 接口。

测试 BeanDefinitionRegistryPostProcessor。

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {

	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("MyBeanDefinitionRegistryPostProcessor...bean的数量：" + beanFactory.getBeanDefinitionCount());
	}

	/**
	 * 这个BeanDefinitionRegistry就是Bean定义信息的保存中心，这个注册中心里面存储了所有的bean定义信息，
	 * 以后，BeanFactory就是按照BeanDefinitionRegistry里面保存的每一个bean定义信息来创建bean实例的。
	 * 
	 * bean定义信息包括有哪些呢？有这些，这个bean是单例的还是多例的、bean的类型是什么以及bean的id是什么。
	 * 也就是说，这些信息都是存在BeanDefinitionRegistry里面的。
	 */
	@Override
	public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("postProcessBeanDefinitionRegistry...bean的数量：" + registry.getBeanDefinitionCount());
		// 除了查看bean的数量之外，我们还可以给容器里面注册一些bean，我们以前也简单地用过
		/*
		 * 第一个参数：我们将要给容器中注册的bean的名字
		 * 第二个参数：BeanDefinition对象
		 */
		// RootBeanDefinition beanDefinition = new RootBeanDefinition(Blue.class); // 现在我准备给容器中添加一个Blue对象
		// 咱们也可以用另外一种办法，即使用BeanDefinitionBuilder这个构建器生成一个BeanDefinition对象，很显然，这两种方法的效果都是一样的
		AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(Blue.class).getBeanDefinition();
		registry.registerBeanDefinition("hello", beanDefinition);
	}
}
```



**BeanDefinitionRegistryPostProcessor 是优先于 BeanFactoryPostProcessor 执行的，可以利用它给容器中再额外添加一些组件**。



**原理**

############################# $to\space do$ #############################

1. IOC 创建对象；
2. refresh() ————> invokeBeanFactoryPostProcessors (beanFactory)；
3. 从容器中获取所有的 BeanDefinitionRegistryPostProcessor 组件；
   1. 依次触发 postProcessBeanDefinitionRegistry()方法；
   2. 再触发 postProcessBeanFactory()方法。
4. 再从容器中找到 BeanFactoryPostProcessor 组件，然后依次触发 postProcessBeanFactory() 方法。



## ApplicationListener

Spring 里面的应用监听器，也就是 Spring 为我们提供的基于事件驱动开发的功能。



编写自定义监听器 MyApplicationListener，实现接口 ApplicationListener。

```java
@Component
public class MyApplicationListener implements ApplicationListener<ApplicationEvent> {

	// 当容器中发布此事件以后，下面这个方法就会被触发
	@Override
	public void onApplicationEvent(ApplicationEvent event) {
		// TODO Auto-generated method stub
		System.out.println("收到事件：" + event);
	}
}
```



编写监听器监听某个事件。

1. 监听的这个事件必须是 ApplicationEvent 及其子类；

2. 把监听器加入到容器中；

3. 只要容器中有相关事件发布，那么就能监听到这个事件。

4. 如何发布一个事件？

   ```java
   @Test
   public void Test(){
     
     AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
     context.publishEvent(new ApplicationEvent(new String("发布的事件")));
     context.close;
   }
   ```



**原理**

############################# $to\space do$ #############################



### @EventListener

让任意方法实现事件监听。

```java

@Service
public class UserService {
	
  ...

	@EventListener(classes=ApplicationEvent.class)
	public void listen() {
		System.out.println("UserService...");
	}
	
}
```

**原理**

############################# $to\space do$ #############################



## Spring容器创建过程







# Web相关





感谢：

- [尚硅谷-雷神]()
- [黑马程序员](https://www.bilibili.com/video/BV1WZ4y1P7Bp?spm_id_from=333.999.0.0)
- [CSDN- 李阿昀](https://liayun.blog.csdn.net/category_10613855_2.html)

