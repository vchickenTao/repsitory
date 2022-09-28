# SpringMVC

## 简介

### MVC框架

MVC是一种软件架构的思想，将软件按照模型、视图、控制器来划分。

- M：Model，模型层

  指工程中的 JavaBean，作用是处理数据。

  JavaBean 分为两类：

  - 一类称为实体类 Bean：专门存储业务数据的，如 Student、User 等。
  - 一类称为业务处理  Bean：指 Service 或 Dao 对象，专门用于处理业务逻辑和数据访问。

- V：View，视图层

  指工程中的 html 或 jsp 等页面，作用是与用户进行交互，展示数据。

- C：Controller，控制层

  指工程中的 servlet，作用是接收请求和响应浏览器。



> **工作流程**
> 用户通过视图层发送请求到服务器，在服务器中请求被 Controller 接收，Controller 调用相应的 Model 层处理请求，处理完毕将结果返回到 Controller，Controller 再根据请求处理的结果找到相应的View视图，渲染数据后最终响应给浏览器。



### SpringMVC

SpringMVC 是 Spring 为表述层开发提供的一整套完备的解决方案。

**特点**

- **Spring 家族原生产品**，与 IOC 容器等基础设施无缝对接；
- **基于原生的 Servlet**，通过了功能强大的**前端控制器 DispatcherServlet**，对请求和响应进行统一处理；
- 表述层各细分领域需要解决的问题**全方位覆盖**，提供**全面解决方案**；
- **代码清新简洁**，大幅度提升开发效率；
- 内部组件化程度高，可插拔式组件**即插即用**，想要什么功能配置相应组件即可；
- **性能卓著**，尤其适合现代大型、超大型互联网项目要求。



## 快速入门

### 开发步骤

需求：客户端发起请求，服务器端接收请求，执行逻辑并进行视图跳转。

开发步骤：

1. 导入 SpringMVC 依赖

```xml
 <!-- Spring -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>5.0.5.RELEASE</version>
</dependency>
<!-- SpringMVC -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>5.0.5.RELEASE</version>
</dependency>
<!-- Servlet -->
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>servlet-api</artifactId>
  <version>2.5</version>
</dependency>
<!-- Jsp -->
<dependency>
  <groupId>javax.servlet.jsp</groupId>
  <artifactId>jsp-api</artifactId>
  <version>2.0</version>
</dependency>
```

2. `web.xml` 中配置 SpringMVC 核心控制器 **<u>DispathcerServlet</u>** ，SpringMVC 的配置文件默认位于 WEB-INF 下，默认名称为 \<servlet-name>-servlet.xml。

```xml
<!-- SpringMVC前端控制器 -->
<servlet>
  <servlet-name>DispatcherServlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
  <init-param>
    <!-- contextConfigLocation为固定值 -->
    <param-name>contextConfigLocation</param-name>
     <!-- 使用classpath:表示从类路径查找配置文件，例如maven工程中的src/main/resources -->
    <param-value>classpath:spring-mvc.xml</param-value>
  </init-param>
  <!-- 
 		作为框架的核心组件，在启动过程中有大量的初始化操作要做
		而这些操作放在第一次请求时才执行会严重影响访问速度
		因此需要通过此标签将启动控制DispatcherServlet的初始化时间提前到服务器启动时
		-->
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>DispatcherServlet</servlet-name>
  <!--
        设置springMVC的核心控制器所能处理的请求的请求路径
        /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
        但是/不能匹配.jsp请求路径的请求
    -->
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

> 关于 \<url-pattern>/\</url-pattern>
>
> - / 所匹配的请求可以是 /login 或 .html 或 .js 或 .css 方式的请求路径，但是 / 不能匹配 .jsp 请求路径的请求。因此就可以避免在访问 jsp 页面时，该请求被 **<u>DispatcherServlet</u>** 处理，从而找不到相应的页面。
> - /* 则能够匹配所有请求，例如在使用过滤器时，若需要对所有请求进行过滤，就需要使用 /\* 的写法。

3. 创建 Controller 类和视图页面

```java
public class HelloController {
  public String Test(){
    System.out.println("quick method running...");        
    return "Hello";
  }
}
```

4. 使用注解配置 Controller 类中业务方法的映射地址

```java
@Controller
public class HelloController {
  @RequestMapping("/")
  public String index(){
    return "index";
  }
}
```

5. 配置SpringMVC核心文件 `spring-mvc.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 自动扫描包 -->
    <context:component-scan base-package="com.tsuiraku.controller"></context:component-scan>
  
  	<!-- 配置Thymeleaf视图解析器 -->
    <bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <property name="order" value="1"/>
        <property name="characterEncoding" value="UTF-8"/>
        <property name="templateEngine">
            <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                <property name="templateResolver">
                    <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">

                        <!-- 视图前缀 -->
                        <property name="prefix" value="/WEB-INF/templates/"/>

                        <!-- 视图后缀 -->
                        <property name="suffix" value=".html"/>
                        <property name="templateMode" value="HTML5"/>
                        <property name="characterEncoding" value="UTF-8" />
                    </bean>
                </property>
            </bean>
        </property>
    </bean>
  
    <!-- 
     处理静态资源，例如html、js、css、jpg
     若只设置该标签，则只能访问静态资源，其他请求则无法访问
     此时必须设置<mvc:annotation-driven/>解决问题
     -->
    <mvc:default-servlet-handler/>
  
  	<!-- 开启mvc注解驱动 -->
    <mvc:annotation-driven>
        <mvc:message-converters>
            <!-- 处理响应中文内容乱码 -->
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="defaultCharset" value="UTF-8" />
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html</value>
                        <value>application/json</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
</beans>
```

6. 客户端发起请求测试



**流程演示**

![截屏2021-08-19 19.16.34](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-08-19%2019.16.34.png)



# SpringMVC的注解解析

## @RequestMapping

作用：用于建立请求 URL 和处理请求方法之间的对应关系。

位置

- 类上，请求URL 的第一级访问目录。此处不写的话，就相当于应用的根目录；

- 方法上，请求 URL 的第二级访问目录，与类上的使用 @ReqquestMapping 标注的一级目录一起组成访问虚拟路径。

属性

- **<u>value</u>**：用于指定请求的 URL。它和 path 属性的作用是一样的；
- **<u>method</u>**：用于指定请求的方式，`method = {RequestMethod.GET, RequestMethod.POST}`；（匹配不成功 ---> 405）
- params：用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的 key 和 value 必须和配置的一模一样。（匹配不成功 ---> 400）
- headers：属性通过请求的请求头信息匹配请求映射。（匹配不成功 ---> 404）



> <u>**value**</u>
>
> 支持 ant 风格的路径
>
> - ？：表示任意的单个字符
> - \*：表示任意的0个或多个字符*
> - **：表示任意的一层或多层目录
>
> 
>
> 支持路径中的占位符
>
> - 原始方式：/deleteUser?id=1
>
> - Rest方式：/deleteUser/1
>
> 
>
> **<u>method</u>**
>
> 1. 对于处理指定请求方式的控制器方法，SpringMVC 中提供了 @RequestMapping 的派生注解
>
>    - 处理 get 请求的映射-->@GetMapping
>
>    - 处理 post 请求的映射-->@PostMapping
>
>    - 处理 put 请求的映射-->@PutMapping
>
>    - 处理 delete 请求的映射-->@DeleteMapping
>
> 2. 常用的请求方式有 **<u>get、post、put、delete</u>**
>
>    - 但是目前浏览器只支持 get 和 post ，若在 form 表单提交时，为 method 设置了其他请求方式的字符串（put或delete），则按照默认的请求方式 get 处理；
>
>    - 若要发送 put 和 delete 请求，则需要通过 Spring 提供的过滤器 **<u>HiddenHttpMethodFilter</u>**，在RESTful部分会讲到。
>
> 
>
> **param**
>
> "param"：要求请求映射所匹配的请求必须携带param请求参数；
>
> "!param"：要求请求映射所匹配的请求必须不能携带param请求参数；
>
> "param=value"：要求请求映射所匹配的请求必须携带param请求参数且param=value；
>
> "param!=value"：要求请求映射所匹配的请求必须携带param请求参数但是param!=value。
>
> 
>
> **headers**
>
> "header"：要求请求映射所匹配的请求必须携带header请求头信息；
>
> "!header"：要求请求映射所匹配的请求必须不能携带header请求头信息；
>
> "header=value"：要求请求映射所匹配的请求必须携带header请求头信息且header=value；
>
> "header!=value"：要求请求映射所匹配的请求必须携带header请求头信息且header!=value。



## [@RequestParam](#@RequestParam注解)

<p class="note note-info">点击标题跳转到详情</p>

## [@RequestHeader](#@RequestHeader注解)

<p class="note note-info">点击标题跳转到详情</p>

## [@CookieValue](#@CookieValue注解)

<p class="note note-info">点击标题跳转到详情</p>

## [@PathVariable](#PathVariable注解)

<p class="note note-info">点击标题跳转到详情</p>

## [@RequestBody](#@RequestBody注解)

<p class="note note-info">点击标题跳转到详情</p>

## [@ResponseBody](#@RequestBody注解)

<p class="note note-info">点击标题跳转到详情</p>

## [@RestController](#@RestController注解)

<p class="note note-info">点击标题跳转到详情</p>

## [@ControllerAdvice](#@ControllerAdvice注解)

<p class="note note-info">点击标题跳转到详情</p>

## [@ExceptionHandler](#@ExceptionHandler注解)

<p class="note note-info">点击标题跳转到详情</p>



# SpringMVC.xml

## @Controller注解扫描

**<u>\<context:componet-scan base-pacakage="com.xxx.xxx"> \</context:componet-scan></u>**



## <u>注解驱动</u>

**<u>\<mvc:annotation-driven/></u>**

开启 mvc 注解驱动。如果不开启注解驱动，那么所有的请求将会被默认 Servlet 处理。

搭配：**<u>\<mvc:default-servlet-handler></u>**（开启默认 Servlet，开启对静态资源的访问）。



## [视图解析器](#<u>视图解析器</u>（ViewResolver）)

<p class="note note-info">点击标题跳转到详情</p>



## 文件解析器

**<u>multipartResolver</u>**

```xml
<!-- 必须通过文件解析器的解析才能将文件转换为MultipartFile对象 -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
```



## [拦截器](#SpringMVC的拦截器)

<p class="note note-info">点击标题跳转到详情</p>

## [异常处理器](#SpringMVC的异常处理器)

<p class="note note-info">点击标题跳转到详情</p>



# SpringMVC获取请求参数

> 客户端请求参数的格式是：`name=value&name=value…`
>
> 服务器端要获得请求的参数，有时还需要进行数据的封装，SpringMVC 可以接收如下类型的参数
>
> - 基本类型参数
> - POJO类型参数
> - 数组类型参数
> - 集合类型参数



## ServletAPI

<p class="note note-warning">不建议使用</p>

将 HttpServletRequest 作为控制器方法的形参，此时 HttpServletRequest 类型的参数表示封装了当前请求的请求报文的对象。

```java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request){
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```



## 控制器方法的形参

在控制器方法的形参位置，**<u>设置和请求参数同名的形参</u>**，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet 中就会将请求参数赋值给相应的形参。

```java
@RequestMapping("/testParam")
public String testParam(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```

- 若请求所传输的请求参数中有多个同名的请求参数，此时可以在控制器方法的形参中设置字符串数组或者字符串类型的形参接收此请求参数；
- 若使用字符串数组类型的形参，此参数的数组中包含了每一个数据；
- 若使用字符串类型的形参，此参数的值为每个数据中间使用逗号拼接的结果。



## @RequestParam注解

当请求的参数名称与 Controller 的业务方法参数名称不一致时，就需要通过 @RequestParam 注解显示的绑定。

@RequestParam 是将请求参数和控制器方法的形参创建映射关系。

> **value**：指定为形参赋值的请求参数的参数名；
>
> **required**：设置是否必须传输此请求参数，默认值为 true；
>
> - 若 `required=true` 时，则当前请求必须传输 value 所指定的请求参数，若没有传输该请求参数，且没有设置 defaultValue 属性，则页面报错400：`Required String parameter 'xxx' is not present`；
> - 若 `required=false`，则当前请求不是必须传输 value 所指定的请求参数，若没有传输，则注解所标识的形参的值为 null。
>
> **defaultValue**：不管 required 属性值为 true 或 false ，当 value 所指定的请求参数没有传输或传输的值为空字符串时，则使用默认值为形参赋值。



```jsp
<form action="${pageContext.request.contextPath}/test" method="post">  
  <input type="text" name="name"><br>  
  <input type="submit" value="提交"><br>
</form>
```

```java
@RequestMapping("/test")
@ResponseBody
public void getTest(@RequestParam("name")String username) throws IOException {
  System.out.println(username);
}
```



## @RequestHeader注解

@RequestHeader 是将请求头信息和控制器方法的形参创建映射关系。	

> - **value**：请求头的名称；
> - **required**：是否必须携带此请求头。
> - **defaultValue**。
>

```java
@RequestMapping("/test")
@ResponseBody
public void testParam(
  // 获取请求头 User-Agent 
  @RequestHeader(value = "User-Agent", required = false) String headerValue) {}
```



## @CookieValue注解

@CookieValue 是将 cookie 数据和控制器方法的形参创建映射关系。使用 @CookieValue 可以获得指定Cookie的值。

> - **value**：指定 cookie 的名称；
> - **required**：是否必须携带此 cookie；
> - **defaultValue**。
>

```java
@RequestMapping("/test")
@ResponseBody
public void testParam(@CookieValue(value = "JSESSIONID", required = false) String headerValue) {}
```



## POJO类型

可以在控制器方法的形参位置设置一个实体类类型的形参。Controller 中的业务方法的 POJO 参数的属性名与请求参数的 name 一致，参数值会自动映射匹配。

```xml
<form th:action="@{/testpojo}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    性别：<input type="radio" name="sex" value="男">男<input type="radio" name="sex" value="女">女<br>
    年龄：<input type="text" name="age"><br>
    邮箱：<input type="text" name="email"><br>
    <input type="submit">
</form>
```

```java
public class User(){  
  private Integer id;  
  private String username;  
  private String password;  
  private Integer age;  
  private String sex; 
  private String email; 
  
  // 生成 set/get 方法
}

@RequestMapping("/test")
@ResponseBody
public void testParam(User user) {  
  System.out.println(user);
}
```





## 集合类型参数

获得集合参数时，要将集合参数包装到一个 POJO 中才可以。

```html
<form action="${pageContext.request.contextPath}/test" method="post">  
  <input type="text" name="userList[0].username"><br>  
  <input type="text" name="userList[0].age"><br>  
  <input type="text" name="userList[1].username"><br>  
  <input type="text" name="userList[1].age"><br>  
  <input type="submit" value="提交"><br>
</form>
```

```java
public Class Vo{  
  private List<User> userList;  
  // set/get
}
```

```java
@RequestMapping("/test")
@ResponseBody
public void getTest(Vo vo) {
  System.out.println(vo.getUserList());
}
```



## 请求参数的乱码问题

### <u>**CharacterEncodingFilter**</u>

当 <u>**get**</u> 请求，可以在 tomcat 中 server.xml 中修改配置。

当 **<u>post</u>** 请求时，数据会出现乱码，我们可以设置一个过滤器来进行编码的过滤。使用 SpringMVC 提供的编码过滤器 CharacterEncodingFilter。

```xml
<!-- 配置CharacterEncodingFilter -->
<!-- web.xml -->
<filter>  
  <filter-name>CharacterEncodingFilter</filter-name>  
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  <init-param>  
  <param-name>encoding</param-name>    
  <param-value>UTF-8</param-value>  
  </init-param></filter>
  <init-param>
    <param-name>forceResponseEncoding</param-name>
    <param-value>true</param-value>
  </init-param>
<filter-mapping>  
    <filter-name>CharacterEncodingFilter</filter-name>  
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

- SpringMVC 中处理编码的过滤器一定要配置到其他过滤器之前，否则无效。



# SpringMVC的域对象共享数据

## 向request域对象中共享数据

根据共享数据的键获取数据。

### ServletAPI

<p class="note note-warning">不建议使用</p>

```java
@RequestMapping("/testServletAPI")
public String testServletAPI(HttpServletRequest request){
    request.setAttribute("testScope", "hello servletAPI"); // 设置域共享数据
    return "success";
}
```



### ModelAndView

<p class="note note-primary">重要</p>

```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
    /**
     * Model：主要用于向请求域共享数据
     * View：主要用于设置视图，实现页面跳转
     */
    ModelAndView mav = new ModelAndView();
    // 向请求域共享数据
    mav.addObject("testScope", "hello ModelAndView");
    // 设置视图，实现页面跳转
    mav.setViewName("success");
    return mav;
}
```



### Model

```java
@RequestMapping("/testModel")
public String testModel(Model model){
    model.addAttribute("testScope", "hello Model");
    return "success";
}
```



### Map

```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    map.put("testScope", "hello Map");
    return "success";
}
```



### ModelMap

```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
    modelMap.addAttribute("testScope", "hello ModelMap");
    return "success";
}
```



## Model & ModelMap & Map的关系

Model、ModelMap、Map 类型的参数其实本质上都是 **<u>BindingAwareModelMap</u>** 类型的。

```
// 继承关系
public interface Model{}
public class ModelMap extends LinkedHashMap<String, Object> {}
public class ExtendedModelMap extends ModelMap implements Model {}
public class BindingAwareModelMap extends ExtendedModelMap {}
```



## 向session域共享数据

根据 session.key 获取数据。

### ServletAPI

<p class="note note-info">建议使用</p>

```java
@RequestMapping("/testSession")
public String testSession(HttpSession session){
    session.setAttribute("testSessionScope", "hello session");
    return "success";
}
```



## 向application域共享数据

对应整个工程域范围。

```java
@RequestMapping("/testApplication")
public String testApplication(HttpSession session){
  	// ServletContext
	  ServletContext application = session.getServletContext();
    application.setAttribute("testApplicationScope", "hello application");
    return "success";
}
```



# SpringMVC的视图

SpringMVC 中的视图是 View 接口，视图的作用渲染数据，将模型Model中的数据展示给用户。

SpringMVC 视图的种类很多，默认有转发视图（InternalResourceView）和重定向视图（RedirectView）。

若使用的视图技术为 Thymeleaf，在 SpringMVC 的配置文件中配置了 Thymeleaf 的视图解析器，由此视图解析器解析之后所得到的是 ThymeleafView。

## ThymeleafView

当控制器方法中所设置的视图名称没有任何前缀时，此时的视图名称会被 SpringMVC 配置文件中所配置的视图解析器解析，视图名称拼接视图前缀和视图后缀所得到的最终路径，会通过转发的方式实现跳转。

```xml
  	<!-- 配置Thymeleaf视图解析器 -->
		<!-- ThymeleafViewResolver -->
    <bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <property name="order" value="1"/>
        <property name="characterEncoding" value="UTF-8"/>
        <property name="templateEngine">
            <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                <property name="templateResolver">
                    <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">

                        <!-- 视图前缀 -->
                        <property name="prefix" value="/WEB-INF/templates/"/>

                        <!-- 视图后缀 -->
                        <property name="suffix" value=".html"/>
                        <property name="templateMode" value="HTML5"/>
                        <property name="characterEncoding" value="UTF-8" />
                    </bean>
                </property>
            </bean>
        </property>
    </bean>
```

```java
@RequestMapping("/testHello")
public String testHello(){
    return "hello";
}
```



## 转发视图

**<u>InternalResourceView</u>**

当 Controller 方法中所设置的视图名称以 `forward:` 为前缀时，创建InternalResourceView 视图，此时的视图名称不会被 SpringMVC 配置文件中所配置的视图解析器解析，而是会将前缀 `forward:` 去掉，剩余部分作为最终路径通过转发的方式实现跳转。

```java
@RequestMapping("/testHello")
public String testHello(){
    return "hello";
}

@RequestMapping("/testForward")
public String testForward(){
    return "forward:/testHello";
}
```



## 重定向视图

**<u>RedirectView</u>**

当 Controller 方法中所设置的视图名称以 `redirect:` 为前缀时，创建 RedirectView 视图，此时的视图名称不会被 SpringMVC 配置文件中所配置的视图解析器解析，而是会将前缀 `redirect:` 去掉，剩余部分作为最终路径通过重定向的方式实现跳转。

```java
@RequestMapping("/testHello")
public String testHello(){
    return "hello";
}

@RequestMapping("/testRedirect")
public String testRedirect(){
    return "redirect:/testHello";
}
```



## 视图控制器

**<u>view-controller</u>**

当控制器方法中，仅仅用来实现页面跳转，即只需要设置视图名称时，可以将处理器方法使用 view-controller 标签进行表示。

当 SpringMVC 中设置任何一个 view-controller 时，其他控制器中的请求映射将全部失效，此时需要在SpringMVC 的核心配置文件中设置开启 mvc 注解驱动的标签：**<u><mvc:annotation-driven /></u>**。

```xml
<!--
	path：设置处理的请求地址
	view-name：设置请求地址所对应的视图名称
-->
<!-- springMVC.xml -->
<mvc:view-controller path="/testView" view-name="success"></mvc:view-controller>

<mvc:annotation-driven />	
```



## <u>视图解析器</u>（ViewResolver）

**<u>ViewResolver</u>**

**<u>InternalResourceViewResolver</u>**

该解析器的默认设置，如下：

```java
REDIRECT_URL_PREFIX = "redirect:"  --重定向前缀
FORWARD_URL_PREFIX = "forward:"    --转发前缀（默认值）
prefix = "";     --视图名称前缀
suffix = "";     --视图名称后缀
```

`springMVC.xml` 配置中通过属性注入的方式修改视图的的前后缀

```xml
<!-- 配置内部资源视图解析器 -->
<!-- 解析JSP视图 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
  <property name="prefix" value="/WEB-INF/views/"></property>  
  <property name="suffix" value=".jsp"></property>
</bean>
```



# RESTful

REST：**Re**presentational **S**tate **T**ransfer，表现层资源状态转移。

**资源**

资源是一种看待服务器的方式，即，将服务器看作是由很多离散的资源组成。每个资源是服务器上一个可命名的抽象概念。因为资源是一个抽象的概念，所以它不仅仅能代表服务器文件系统中的一个文件、数据库中的一张表等等具体的东西，可以将资源设计的要多抽象有多抽象，只要想象力允许而且客户端应用开发者能够理解。与面向对象设计类似，资源是以名词为核心来组织的，首先关注的是名词。一个资源可以由一个或多个 URI 来标识。URI 既是资源的名称，也是资源在 Web 上的地址。对某个资源感兴趣的客户端应用，可以通过资源的 URI 与其进行交互。

**资源的表述**

资源的表述是一段对于资源在某个特定时刻的状态的描述。可以在客户端-服务器端之间转移（交换）。资源的表述可以有多种格式，例如HTML/XML/JSON/纯文本/图片/视频/音频等等。资源的表述格式可以通过协商机制来确定。请求-响应方向的表述通常使用不同的格式。

**状态转移**

状态转移说的是：在客户端和服务器端之间转移（transfer）代表资源状态的表述。通过转移和操作资源的表述，来间接实现操作资源的目的。

Restful 是一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件。主要用于客户端和服务器交互类的软件，基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存机制等。

Restful风格的请求是使用 **"<u>url+请求方式</u>”** 表示一次请求目的的，HTTP 协议里面四个表示操作方式的动词如下：

- GET：用于获取资源

- POST：用于新建资源

- PUT：用于更新资源

- DELETE：用于删除资源 

REST 风格提倡 URL 地址使用统一的风格设计，从前到后各个单词使用斜杠分开，不使用问号键值对方式携带请求参数，而是将要发送给服务器的数据作为 URL 地址的一部分，以保证整体风格的一致性。

| 操作     | 传统方式           | REST风格               |
| -------- | ------------------ | ---------------------- |
| 查询操作 | `getUserById?id=1` | `user/1` get请求方式   |
| 保存操作 | `saveUser`         | `user` post请求方式    |
| 删除操作 | `deleteUser?id=1`  | `user/1` elete请求方式 |
| 更新操作 | `updateUser`       | `user` put请求方式     |

上述 url 地址 `/user/1` 中的 1 就是要获得的请求参数，在 SpringMVC 中可以使用占位符进行参数绑定。地址 `/user/1` 可以写成 `/user/{id}`，占位符 {id} 对应的就是 1 的值。在业务方法中我们可以使用 **<u>@PathVariable</u>** 注解进行占位符的匹配获取工作。



```java
public class UserController{
  @RequestMapping(value = "/user" method = RequestMethod.GET)
  public getUserAll(){
    return "success";
	}
  
  @RequestMapping(value = "/user/{id}" method = RequestMethod.GET)
  public getUserById(){
    return "success";
  }
  
  @RequestMapping(value = "/user" method = RequestMethod.POST)
  public insertUser(string username, string password){
    return "success";
	}
  
  
  /**
  * PUT 和 DELETE 的条件
  * POST
  * _method
  */
  @RequestMapping(value = "/user" method = RequestMethod.PUT)
  public updateUser(string username, string password){
    return "success";
	}
  
  @RequestMapping(value = "/user" method = RequestMethod.DELETE)
  public deleteUser(){
    return "success";
	}
}
```



## <u>HiddenHttpMethodFilter</u>

由于浏览器只支持发送 get 和 post 方式的请求，那么该如何发送 put 和 delete 请求。

SpringMVC 提供了 **<u>HiddenHttpMethodFilter</u>** 帮助我们**<u>将 POST 请求转换为 DELETE 或 PUT 请求</u>**。

**<u>HiddenHttpMethodFilter</u>** 处理put和delete请求的条件：

- 当前请求的请求方式必须为 post；

- 当前请求必须传输请求参数 _method。

注册 **HiddenHttpMethodFilter** 。

```xml
<!-- web.xml -->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 注意！必须先注册 **<u>CharacterEncodingFilter</u>**，再注册 <u>**HiddenHttpMethodFilter**</u>。
>
> 原因
>
> - 在 **<u>CharacterEncodingFilter</u>** 中通过 `request.setCharacterEncoding(encoding)` 方法设置字符集的；
>
> - `request.setCharacterEncoding(encoding)` 方法要求前面不能有任何获取请求参数的操作；
>
> - 而 <u>**HiddenHttpMethodFilter**</u> 恰恰有一个获取请求方式的操作 `String paramValue = request.getParameter(this.methodParam);`。



## @PathVariable注解

@PathVariable 可以用来映射 URL 中的占位符到目标方法的参数中。

```java
@RequestMapping("/testPathVariable/{id}")
    public String testPathVariable(@PathVariable("id") Integer id)
    {
        System.out.println("testPathVariable:"+id);
        return SUCCESS;
    }
```





# SpringMVC的数据响应

## <u>HttpMessageConverter</u>

**<u>HttpMessageConverter</u>**，报文信息转换器，将请求报文转换为 Java 对象，或将 Java 对象转换为响应报文。

<u>HttpMessageConverter</u> 提供了两个注解和两个类型。

@RequestBody，@ResponseBody，RequestEntity，ResponseEntity。



## @RequestBody注解

@RequestBody 可以获取**请求体**，需要在 Controller 方法设置一个形参，使用 @RequestBody 进行标识，当前请求的请求体就会为当前注解所标识的形参赋值。

```html
<form th:action="@{/testRequestBody}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    <input type="submit">
</form>
```

```java
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody){
    System.out.println("requestBody:"+requestBody);
    return "success";
}

// 输出
// requestBody:username=admin&password=123456
```



## RequestEntity

RequestEntity 是封装**请求报文**的一种类型，需要在 Controller 方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过 `getHeaders()` 获取请求头信息，通过 `getBody()` 获取请求体信息。

```java
@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity){
    System.out.println("requestHeader:"+requestEntity.getHeaders());
    System.out.println("requestBody:"+requestEntity.getBody());
    return "success";
}
```



## @ResponseBody注解

@ResponseBody 用于标识一个 Controller 方法，可以将该方法的返回值直接作为**响应报文**的响应体响应到浏览器。

```java
@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";
}
```



## 响应Json数据

当 Java 对象作为返回值进行响应，会出现 HttpMessageNotWritableException 异常。需要将数据转化成 json 格式返回。



> - 导入 **<u>jackson</u>** 依赖；
>
>   ```xml
>   <dependency>
>       <groupId>com.fasterxml.jackson.core</groupId>
>       <artifactId>jackson-databind</artifactId>
>       <version>2.12.1</version>
>   </dependency>
>   ```
>
> - 开启 mvc 注解驱动；
>
>   （在 **<u>HandlerAdaptor</u>** 中会自动装配一个消息转换器：**<u>MappingJackson2HttpMessageConverter</u>**，可以将响应到浏览器的 Java 对象转换为 json 格式的字符串）
>
>   ```xml
>   <mvc:annotation-driven />
>   ```
>
> - 在 Controller 方法上使用 @ResponseBody 注解进行标识；
>
> - 将 Java 对象直接作为 Controller 方法返回，自动转换为 json 格式字符串。



```java
@RequestMapping("/testResponseUser")
@ResponseBody
public User testResponseUser(){
    return new User(1001,"admin","123456",23,"男");
}

// 浏览器的页面中展示的结果
// {"id":1001,"username":"admin","password":"123456","age":23,"sex":"男"}
```



## @RestController注解

@RestController 注解是 SpringMVC 提供的一个复合注解，标识在控制器的类上，就相当于为类添加了@Controller 注解，并且为其中的每个方法添加了@ResponseBody 注解。



## ResponseEntity

ResponseEntity 用于 Controller 方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文。

（用于实现文件下载）



# 文件的上传和下载

## 文件下载

使用 ResponseEntity 实现下载文件的功能。

```java
@RequestMapping("/testDown")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException {
    // 获取 ServletContext 对象
    ServletContext servletContext = session.getServletContext();
    // 获取服务器中文件的真实路径
    String realPath = servletContext.getRealPath("/static/img/1.jpg");
    // 创建输入流
    InputStream is = new FileInputStream(realPath);
    // 创建字节数组
    byte[] bytes = new byte[is.available()];
    // 将流读到字节数组中
    is.read(bytes);
    // 创建HttpHeaders对象设置响应头信息
    MultiValueMap<String, String> headers = new HttpHeaders();
    // 设置要下载方式以及下载文件的名字
    headers.add("Content-Disposition", "attachment;filename=1.jpg");
    // 设置响应状态码
    HttpStatus statusCode = HttpStatus.OK;
    // 创建 ResponseEntity 对象
    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers, statusCode);
    // 关闭输入流
    is.close();
    return responseEntity;
}
```



## 文件上传

文件上传要求 form 表单的请求方式必须为 **post**，并且添加属性 `enctype="multipart/form-data"` ，SpringMVC 中将上传的文件封装到 **<u>MultipartFile</u>** 对象中，通过此对象可以获取文件相关信息。



```xml
<!-- thymeleay视图解析器 -->
<form th:action="@{/testUp}" method="post" enctype="multipart/form-data">
  头像：<input type="file" name="photo"></input>
  <input type="submit" value="上传"></input>
</form>
```



> - 导入 **<u>commons-fileupload</u>** 依赖；
>
>   ```xml
>   <!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
>   <dependency>
>       <groupId>commons-fileupload</groupId>
>       <artifactId>commons-fileupload</artifactId>
>       <version>1.3.1</version>
>   </dependency>
>   ```
>
> - 在 SpringMVC 的配置文件中添加配置；
>
>   ```xml
>   <!-- 必须通过文件解析器的解析才能将文件转换为MultipartFile对象 -->
>   <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
>   ```
>
> - Controller 方法（接受形参名与提交的名一致）。
>
>   ```java
>   @RequestMapping("/testUp")
>   public String testUp(MultipartFile photo, HttpSession session) throws IOException {
>       // 获取上传的文件的文件名
>       String fileName = photo.getOriginalFilename();
>       // 处理文件重名问题
>     	// 获取上传文件后缀名
>       String hzName = fileName.substring(fileName.lastIndexOf("."));
>       fileName = UUID.randomUUID().toString() + hzName;
>       // 获取服务器中photo目录的路径
>       ServletContext servletContext = session.getServletContext();
>       String photoPath = servletContext.getRealPath("photo");
>       File file = new File(photoPath);
>       if(!file.exists()){
>           file.mkdir();
>       }
>       String finalPath = photoPath + File.separator + fileName;
>       // 实现上传功能
>       photo.transferTo(new File(finalPath));
>       return "success";
>   }
>   ```







# SpringMVC的拦截器

SpringMVC 的拦截器类似于 Servlet 开发中的过滤器 Filter，用于对处理器进行 **预处理（preHandler）** 和**后处理（postHandler）**。

将拦截器按一定的顺序联结成一条链，这条链称为**拦截器链（Interceptor Chain）**。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按其之前定义的顺序被调用。拦截器也是 AOP 思想的具体实现。

**拦截器与过略器的区别**

| **区别** | **过滤器**                                                | **拦截器**                                                   |
| -------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| 使用范围 | 是 servlet 规范中的一部分，任何 Java Web 工程都可以使用   | 是 SpringMVC 框架自己的，只有使用了 SpringMVC 框架的工程才能用 |
| 拦截范围 | 在 url-pattern 中配置了/*之后，可以对所有要访问的资源拦截 | 只会拦截访问的控制器方法，如果访问的是 jsp、html、css、image、js 是不会进行拦截的 |

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-18%2015.53.35.png" alt="截屏2021-09-18 15.53.35" style="zoom:50%;" />



***即对 DispatchServlet 所处理的所有请求进行拦截。***



**配置使用拦截器。**

- 创建拦截器类实现 `HandlerInterceptor` 接口
  - `preHandle`
    - 目标方法执行前执行；
  - `postHandle`
    - 目标方法执行后，视图返回前返回前执行；
  - `afterCompletion`
    - 整个流程执行后执行。

**<u>HandlerInterceptor</u>**

SpringMVC 中的拦截器用于拦截控制器方法的执行，SpringMVC 中的拦截器需要实现 **<u>HandlerInterceptor</u>**。

```java
@Component
public class MyHandlerInterceptor implements HandlerInterceptor {
  // 返回 true：放行 
  // 返回 false：拦截
  public boolean preHandle(HttpServletRequest request, HttpServletResponse 
        response, Object handler) {
    System.out.println("preHandle running...");
    String param = request.getParametr("param");
    if(parma.equals("yes")) {
      return true;
    } else {
      request.getRequestDisaptcher("/error.jsp").forward(request, response);
      return false;
		}
    return true; 
  }
  // 可以在此方法修改视图 modelAndView
  public void postHandle(HttpServletRequest request, HttpServletResponse 
        response, Object handler, ModelAndView modelAndView) {
    System.out.println("postHandle running...");
  }
  public void afterCompletion(HttpServletRequest request, HttpServletResponse 
        response, Object handler, Exception ex) {
    System.out.println("afterCompletion running...");
  }
}
```

- 配置注册拦截器

```xml
<!-- springMVC.xml -->

<!-- 使用bean -->
<mvc:interceptor>
    <mvc:mapping path="/**"/>
    <mvc:exclude-mapping path="/testRequestEntity"/>
  	<bean class="com.tsuiraku.interceptor.MyHandlerInterceptor"></bean>
</mvc:interceptor>

<!-- 使用ref -->
<mvc:interceptor>
    <mvc:mapping path="/**"/>
    <mvc:exclude-mapping path="/testRequestEntity"/>
    <ref bean="MyHandlerInterceptor"></ref>
</mvc:interceptor>

<!-- 
	以上配置方式可以通过ref或bean标签设置拦截器，通过mvc:mapping设置需要拦截的请求，通过mvc:exclude-mapping设置需要排除的请求，即不需要拦截的请求
-->
```

使用 ***\<bean>*** 配置拦截器，此时所有的请求都会经过 ***MyHandlerInterceptor*** 拦截器。

使用 ***\<ref>*** 配置拦截器，此时所有的请求都会经过 ***MyHandlerInterceptor*** 拦截器。

当只使用以上两种方式之一，此时会对所有请求路径进行拦截。

当使用 ***\<mvc:mapping*>** ，指定需要拦截的请求路径。

当使用 ***\<mvc:exclude-mapping>***，指定放行的请求路径。



**拦截器的三个方法。**

|      **方法名**       | **说明**                                                     |
| :-------------------: | :----------------------------------------------------------- |
|    **preHandle()**    | 方法将在请求处理之前进行调用，该方法的返回值是布尔值 Boolean 类型的。当它返回为 false 时，表示请求结束，后续的 Interceptor 和 Controller 都不会再执行；当返回值为 true 时就会继续调用下一个Interceptor 的 preHandle 方法。 |
|   **postHandle()**    | 该方法是在当前请求进行处理之后被调用，前提是 preHandle 方法的返回值为true 时才能被调用，且它会在 DispatcherServlet 进行视图返回渲染之前被调用，所以我们可以在这个方法中对 Controller 处理之后的 ModelAndView 对象进行操作。 |
| **afterCompletion()** | 该方法将在整个请求结束之后，也就是在 DispatcherServlet 渲染了对应的视图之后执行，前提是 preHandle 方法的返回值为true 时才能被调用。 |



***当有多个拦截器时，执行顺序。***

- **preHandler** 按照顺序执行；
-  ***postHandler*** 和 ***afterCompletion*** 按照反序执行。

```
OneHandlerInterceptor -> preHandler
TwoHandlerInterceptor -> preHandler
TwoHandlerInterceptor -> postHandler
OneHandlerInterceptor -> postHandler
TwoHandlerInterceptor -> afterCompletion
OneHandlerInterceptor -> afterCompletion
```



# SpringMVC的异常处理器

**<u>HandlerExceptionResolver</u>**

系统中异常包括两类：**预期异常**和**运行时异常（RuntimeException）**，前者通过捕获异常从而获取异常信息，后者主要通过规范代码开发、测试等手段减少运行时异常的发生。

系统的 **Dao**、**Service**、**Controller** 出现都通过 throws Exception 向上抛出，最后由 SpringMVC 前端控制器交由异常处理器进行异常处理，如下图：

![image-20210821171506546](https://gitee.com/tsuiraku/typora/raw/master/img/image-20210821171506546.png)



##### 

**自定义异常处理的方式**

- 使用 Spring MVC 提供的简单异常处理器 **<u>SimpleMappingExceptionResolver</u>**

- 实现 Spring 的异常处理接口 **<u>HandlerExceptionResolver</u>** 自定义自己的异常处理器



## **基于配置的异常处理**

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
        	<!--
        		properties的键表示处理器方法执行过程中出现的异常
        		properties的值表示若出现指定异常时，设置一个新的视图名称，跳转到指定页面
        	-->
            <prop key="java.lang.ArithmeticException">error</prop>
        </props>
    </property>
    <!--
    	exceptionAttribute属性设置一个属性名，将出现的异常信息在请求域中进行共享
    -->
    <property name="exceptionAttribute" value="ex"></property>
</bean>
```



## **基于注解的异常处理**

### @ControllerAdvice注解

增强的 Controller 注解。

> 1. 全局异常处理；
> 2. 全局数据绑定；
> 3. 全局数据预处理。



### @ExceptionHandler注解

统一处理某一类异常，从而能够减少代码重复率和复杂度。



```java
// 将当前类标识为异常处理的组件
@ControllerAdvice
public class ExceptionController {

    // 用于设置所标识方法处理的异常
    @ExceptionHandler(ArithmeticException.class)
    // ex表示当前请求处理中出现的异常对象
    public String handleArithmeticException(Exception ex, Model model){
        model.addAttribute("ex", ex);
        return "error";
    }
}
```



# 基于注解配置SpringMVC

***使用配置类和注解代替 web.xml 和 SpringMVC 配置文件的功能***



## WebInit

**<u>AbstractAnnotationConfigDispatcherServletInitializer</u>**

在Servlet3.0 环境中，容器会在类路径中查找实现 ***javax.servlet.ServletContainerInitializer*** 接口的类，如果能够找到就使用它来配置 Servlet 容器。

Spring 提供了这个接口的实现，名为 ***SpringServletContainerInitializer***，这个类反过来又会查找实现***WebApplicationInitializer*** 的类并将配置的任务交给它来完成。Spring3.2 引入了一个便利的***WebApplicationInitializer*** 基础实现，即 ***AbstractAnnotationConfigDispatcherServletInitializer***。

当我们的类扩展了 ***AbstractAnnotationConfigDispatcherServletInitializer*** 并将其部署到 Servlet3.0 容器的时候，容器会自动发现它，并用它来配置 Servlet 上下文。



初始化类 ***WebInit***，代替 ***web.xml***。

```java
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * 指定spring的配置类
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    /**
     * 指定SpringMVC的配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 指定DispatcherServlet的映射规则，即url-pattern
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 添加过滤器
     * @return
     */
    @Override
    protected Filter[] getServletFilters() {
      	// 设置字符串过略器
        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8");
        encodingFilter.setForceRequestEncoding(true);
      	// 设置HiddenHttpMethodFilter
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
        return new Filter[]{encodingFilter, hiddenHttpMethodFilter};
    }
}
```

 

## WebConfig

**<u>WebMvcConfigurer</u>**

初始化类 ***WebConfig***，代替 ***springMVC.xml***。

```java
/**
* 代替springMVC.xml
* 1. 扫描组件
* 2. 视图解析器
* 3. view-controller
* 4. default-servlet-handler
* 5. mvc注解驱动
* 6. 文件解析器
* 7. 异常处理
* 8. 拦截器
*/
@Configuration // 配置类
@ComponentScan("com.tsuiraku") // 扫描组件
@EnableWebMvc // 开启MVC注解驱动
public class WebConfig implements WebMvcConfigurer {

    // 使用default-servlet-handler处理静态资源
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer){
        configurer.enable(); // 开启默认default-servlet-handler
    }
  
  	// 配置view-controller
    @Override
    public void addViewController(ViewControllerRegistry registry){
        registry.addViewController("/hello").setViewName("hello");
    }

    // 配置文件上传解析器
  	// （导入依赖）
    @Bean
    public CommonsMultipartResolver multipartResolver(){
        return new CommonsMultipartResolver();
    }

    // 配置拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        FirstInterceptor firstInterceptor = new FirstInterceptor();
        registry.addInterceptor(firstInterceptor).addPathPatterns("/**");
    }
    
    // 配置视图控制
    
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
    }
    
    // 配置异常处理器
    @Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers)
    {
        SimpleMappingExceptionResolver exceptionResolver = new
          SimpleMappingExceptionResolver();
        Properties prop = new Properties();
        prop.setProperty("java.lang.ArithmeticException", "error");
        // 设置异常映射
        exceptionResolver.setExceptionMappings(prop);
        // 设置共享异常信息的键
        exceptionResolver.setExceptionAttribute("ex");
        resolvers.add(exceptionResolver);
    }

  	// thymeleaf
    // 配置生成模板解析器
    @Bean
    public ITemplateResolver templateResolver() {
        WebApplicationContext webApplicationContext = 
          ContextLoader.getCurrentWebApplicationContext();
        // ServletContextTemplateResolver需要一个ServletContext作为构造参数
      	// 可通过WebApplicationContext 的方法获得
        ServletContextTemplateResolver templateResolver = new 
          ServletContextTemplateResolver(webApplicationContext.getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        return templateResolver;
    }

  	// thymeleaf
    // 生成模板引擎并为模板引擎注入模板解析器
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }
  
		// thyleaf
    // 生成视图解析器并未解析器注入模板引擎
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }

}
```



## SpringConfig

初始化类 ***SpringConfig***，代替 ***spring.xml***。

```java
@Configuration
public class SpringConfig {
	// ssm整合之后，spring的配置信息写在此类中
}
```



# SpringMVC的组件解析

## 常用组件

> 1. **前端控制器：DispatcherServlet**
>
>    用户请求到达前端控制器，它就相当于 ***MVC*** 模式中的 ***C***，***DispatcherServlet*** 是整个流程控制的中心，由，它调用其它组件处理用户的请求，***DispatcherServlet*** 的存在降低了组件之间的耦合性。
>
>    统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求。
>
> 2. **处理器映射器：HandlerMapping**
>
>    ***HandlerMapping*** 负责根据用户请求找到 ***Handler*** 即处理器 Controller，SpringMVC 提供了不同的映射器实现不同的。映射方式，例如：配置文件方式，实现接口方式，注解方式等。
>
>    根据请求的 ***url、method*** 等信息查找 ***Handler***，即控制器方法。
>
> 3. **处理器适配器：HandlerAdapter**
>
>    通过 ***HandlerAdapter*** 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理。
>
> 4. **处理器：Handler**
>
>    在 ***DispatcherServlet*** 的控制下 ***Handler*** 对具体的用户请求进行处理。
>
>    它就是我们开发中要编写的具体业务控制器。由 ***DispatcherServlet*** 把用户请求转发到 ***Handler***。由
>
>    ***Handler*** 对具体的用户请求进行处理。
>
>    即控制器 ***Controller***。
>
> 5. **视图解析器：View Resolver**
>
>    ***View Resolver*** 负责将处理结果生成 ***View*** 视图，***View Resolver*** 首先根据逻辑视图名解析成物理视图名，即具体的页面地址，再生成 ***View*** 视图对象，最后对 ***View*** 进行渲染将处理结果通过页面展示给用户。（***ThymeleafView、InternalResourceView、RedirectView***）
>
>    即进行视图解析，得到相应的视图。
>
> 6. **视图：View**
>
>    ***SpringMVC*** 框架提供了很多的 ***View*** 视图类型的支持，包括：***jstlView、freemarkerView、pdfView*** 等。最常用的视图就是 ***jsp***。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。
>
>    即将模型数据通过页面展示给用户。



## DispatchServlet

***：中央控制器***



<img src="https://gitee.com/tsuiraku/typora/raw/master/img/img005.png" alt="img005" style="zoom:70%;" />



### 初始化过程

***<u>源码 to do</u>***

### 服务过程

***<u>源码 to do</u>***

### 调用组件处理请求

***<u>源码 to do</u>***



## 执行流程

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-19%2010.23.18.png" alt="截屏2021-09-19 10.23.18" style="zoom:50%;" />

**<u>总结流程</u>**

> 1. 用户向服务器发送请求，请求被前端控制器 ***DispatcherServlet*** 捕获；
>
> 2. ***DispatcherServlet*** 收到请求，对请求 ***URL*** 进行解析，得到请求资源标志符（***URL***）， 调用 ***HandlerMapping*** 处理器映射器；
>
>    1. 若当前映射不存在（***HandlerMapping***）；
>
>       1. 判断是否配置默认 ***Servlet*** 处理资源资源： ***\<default-servlet-handler>***。
>
>       2. 若未配置，控制台显示映射无法找到，客户端展示 ***404*** 页面错误；
>
>          <img src="https://gitee.com/tsuiraku/typora/raw/master/img/img007.png" alt="img007" style="zoom:50%;" />
>
>       3. 若已经配置，但是无法找到访问资源，客户端展示 ***404*** 页面错误。
>
>    2. 若当前映射存在。
>
>       1. 根据该 ***URI***，调用 ***HandlerMapping*** 获得该 ***Handler*** 配置的所有相关的对象（包括 ***Handler*** 对象以及 ***Handler*** 对象对应的拦截器），最后以 ***HandlerExecutionChain*** 执行链对象的形式返回；
>
>       2. ***DispatcherServlet*** 根据获得的 ***Handler***，选择一个合适的 ***HandlerAdapter***；
>
>       3. 如果成功获得 ***HandlerAdapter***，此时将开始执行拦截器的 ***preHandler***方法；
>
>       4. 提取 ***Request*** 中的模型数据，填充 ***Handler*** 入参，开始执行 ***Handler*** 方法，即 ***Controller*** 方法，处理请求。在填充 ***Handler*** 的入参过程中，根据你的配置，***Spring*** 将帮你做一些额外的工作：
>
>          1. ***HttpMessageConveter***
>
>             将请求消息（如 ***Json、xml*** 等数据）转换成一个对象，将对象转换为指定的响应信息。
>
>          2. 数据转换
>
>             对请求消息进行数据转换。
>
>          3. 数据格式化
>
>             对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等
>
>          4. 数据验证
>
>             验证数据的有效性（长度、格式等）。
>
>       5. ***Handler*** 执行完成后，向 ***DispatcherServlet*** 返回一个 ***ModelAndView*** 对象；
>
>       6. 此时将开始执行拦截器的 ***postHandle*** 方法；
>
>       7. 根据返回的 ***ModelAndView*** 
>
>          1. 若存在异常，则执行 ***HandlerExceptionResolver*** 进行异常处理；
>
>          2. 否则，选择一个适合的 ***ViewResolver*** 进行视图解析，根据 ***Model*** 和 ***View***，来渲染视图。
>
>       8. 渲染视图完毕执行拦截器的 ***afterCompletion*** 方法；
>
>       9. 将渲染结果返回给客户端。



# 感谢

- [尚硅谷-杨博超](https://www.bilibili.com/video/BV1Ry4y1574R?p=1)
- [黑马程序员](https://www.bilibili.com/video/BV1WZ4y1P7Bp?spm_id_from=333.999.0.0)

