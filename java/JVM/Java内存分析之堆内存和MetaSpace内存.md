**<span style="font-size: 35px;">🍦 调试排错  Java内存分析之堆内存和MetaSpace内存</span>**

---

## 常见的内存溢出问题(内存和MetaSpace内存)

> 常见的内存溢出问题(内存和MetaSpace内存)。

### Java 堆内存溢出

Java 堆内存（Heap Memory)主要有两种形式的错误：

1. OutOfMemoryError: Java heap space
2. OutOfMemoryError: GC overhead limit exceeded

#### OutOfMemoryError: Java heap space

在 Java 堆中只要不断的创建对象，并且 `GC-Roots` 到对象之间存在引用链，这样 `JVM` 就不会回收对象。

只要将`-Xms(最小堆)`,`-Xmx(最大堆)` 设置为一样禁止自动扩展堆内存。

当使用一个 `while(true)` 循环来不断创建对象就会发生 `OutOfMemory`，还可以使用 `-XX:+HeapDumpOutofMemoryErorr` 当发生 OOM 时会自动 dump 堆栈到文件中。

伪代码:

```java
public static void main(String[] args) {
	List<String> list = new ArrayList<>(10) ;
	while (true){
		list.add("1") ;
	}
}
```

当出现 OOM 时可以通过工具来分析 `GC-Roots` [引用链  (opens new window)](https://github.com/crossoverJie/Java-Interview/blob/master/MD/GarbageCollection.md#可达性分析算法) ，查看对象和 `GC-Roots` 是如何进行关联的，是否存在对象的生命周期过长，或者是这些对象确实改存在的，那就要考虑将堆内存调大了。

```text
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:261)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:235)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:227)
	at java.util.ArrayList.add(ArrayList.java:458)
	at com.crossoverjie.oom.HeapOOM.main(HeapOOM.java:18)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:147)

Process finished with exit code 1
```

`java.lang.OutOfMemoryError: Java heap space`表示堆内存溢出。

#### OutOfMemoryError: GC overhead limit exceeded

GC overhead limt exceed检查是Hotspot VM 1.6定义的一个策略，通过统计GC时间来预测是否要OOM了，提前抛出异常，防止OOM发生。Sun 官方对此的定义是：“并行/并发回收器在GC回收时间过长时会抛出OutOfMemroyError。过长的定义是，超过98%的时间用来做GC并且回收了不到2%的堆内存。用来避免内存过小造成应用不能正常工作。“

PS：-Xmx最大内存配置2GB

```java
public void testOom1() {
	List<Map<String, Object>> mapList = new ArrayList<>();
	for (int i = 0; i < 1000000; i++) {
		Map<String, Object> map = new HashMap<>();
		for (int j = 0; j < i; j++) {
				map.put(String.valueOf(j), j);
		}
		mapList.add(map);
	}
}
```

上述的代码执行会：old区占用过多导致频繁Full GC，最终导致GC overhead limit exceed。

```java
java.lang.OutOfMemoryError: GC overhead limit exceeded
	at java.util.HashMap.newNode(HashMap.java:1747) ~[na:1.8.0_181]
	at java.util.HashMap.putVal(HashMap.java:642) ~[na:1.8.0_181]
	at java.util.HashMap.put(HashMap.java:612) ~[na:1.8.0_181]
	at tech.pdai.test.oom.controller.TestOomController.testOom1(TestOomController.java:33) ~[classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_181]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_181]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_181]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_181]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:197) ~[spring-web-5.3.9.jar:5.3.9]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:141) ~[spring-web-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:106) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1064) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:963) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:655) ~[tomcat-embed-core-9.0.50.jar:4.0.FR]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:764) ~[tomcat-embed-core-9.0.50.jar:4.0.FR]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:228) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:163) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) ~[tomcat-embed-websocket-9.0.50.jar:9.0.50]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:190) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:163) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100) ~[spring-web-5.3.9.jar:5.3.9]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) ~[spring-web-5.3.9.jar:5.3.9]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:190) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:163) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93) ~[spring-web-5.3.9.jar:5.3.9]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) ~[spring-web-5.3.9.jar:5.3.9]
```

还可以使用 `-XX:+HeapDumpOutofMemoryErorr` 当发生 OOM 时会自动 dump 堆栈到文件中。

JVM还有这样一个参数：`-XX:-UseGCOverheadLimit` 设置为false可以禁用这个检查。其实这个参数解决不了内存问题，只是把错误的信息延后，替换成 java.lang.OutOfMemoryError: Java heap space。

### MetaSpace (元数据) 内存溢出

> `JDK8` 中将永久代移除，使用 `MetaSpace` 来保存类加载之后的类信息，字符串常量池也被移动到 Java 堆。

`PermSize` 和 `MaxPermSize` 已经不能使用了，在 JDK8 中配置这两个参数将会发出警告。

JDK 8 中将类信息移到到了本地堆内存(Native Heap)中，将原有的永久代移动到了本地堆中成为 `MetaSpace` ,如果不指定该区域的大小，JVM 将会动态的调整。

可以使用 `-XX:MaxMetaspaceSize=10M` 来限制最大元数据。这样当不停的创建类时将会占满该区域并出现 `OOM`。

```java
public static void main(String[] args) {
	while (true){
		Enhancer  enhancer = new Enhancer() ;
		enhancer.setSuperclass(HeapOOM.class);
		enhancer.setUseCache(false) ;
		enhancer.setCallback(new MethodInterceptor() {
			@Override
			public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
				return methodProxy.invoke(o,objects) ;
			}
		});
		enhancer.create() ;

	}
}
```

使用 `cglib` 不停的创建新类，最终会抛出:

```text
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at net.sf.cglib.core.ReflectUtils.defineClass(ReflectUtils.java:459)
	at net.sf.cglib.core.AbstractClassGenerator.generate(AbstractClassGenerator.java:336)
	... 11 more
Caused by: java.lang.OutOfMemoryError: Metaspace
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
	... 16 more
```

注意: 这里的 OOM 伴随的是 `java.lang.OutOfMemoryError: Metaspace` 也就是元数据溢出。

## 分析案例

> 在实际工作中，如何去定位内存泄漏问题呢？

### 堆内存dump

- **通过OOM获取**

即在OutOfMemoryError后获取一份HPROF二进制Heap Dump文件，在jvm中添加参数：

```bash
-XX:+HeapDumpOnOutOfMemoryError
```

- **主动获取**

在虚拟机添加参数如下，然后在Ctrl+Break组合键即可获取一份Heap Dump

```bash
-XX:+HeapDumpOnCtrlBreak
```

- **使用HPROF agent**

使用Agent可以在程序执行结束时或受到SIGOUT信号时生成Dump文件

配置在虚拟机的参数如下：

```bash
-agentlib:hprof=heap=dump,format=b
```

- **jmap获取** (常用)

jmap可以在cmd里执行，命令如下：

```bash
jmap -dump:format=b file=<文件名XX.hprof> <pid>
```

- **使用JConsole**

Acquire Heap Dump

- **使用JProfile**

Acquire Heap Dump

### 使用MAT分析内存

MAT 等工具可以看：[Java 问题排查之JVM可视化工具 - MAT](/java/JVM/Java问题排查之JVM可视化工具)



## 参考文章

- 转载自[Java 全栈知识体系](https://www.pdai.tech/md/java/jvm/java-jvm-oom.html)