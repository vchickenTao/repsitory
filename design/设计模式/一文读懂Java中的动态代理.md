**<span style="font-size: 35px;">🧑‍🌾 一文读懂Java中的动态代理</span>**

---

## 从代理模式说起

要读懂`动态代理`，应从`代理模式`说起。而实现代理模式，常见有下面两种实现：

(1) 代理类`关联`目标对象，实现目标对象实现的接口

```java
public class Proxy implements Subject {
    // 维持一个对真实主题对象的引用
    private RealSubject realSubject;

    public Proxy(RealSubject realSubject) {
        this.realSubject = realSubject;
    }

    public void preRequest() {
        // ...
    }

    public void postRequest() {
        // ...
    }

    @Override
    public void request() {
        preRequest();
        // 调用真实主题对象的方法
        realSubject.request();
        postRequest();
    }
}
```

(2) 代理类`继承`目标类，重写需要代理的方法

```java
public class Proxy extends RealSubject {

    public void preRequest() {
        // ...
    }

    public void postRequest() {
        // ...
    }

    @Override
    public void request() {
        preRequest();
        super.request();
        postRequest();
    }
}
```

> 如果程序`运行前`就在Java代码中定义好代理类(`Proxy`)，那么这种代理方式就叫做`静态代理`；若代理类在程序`运行时`创建就叫做`动态代理`

- 如果为特定类的特定方法生成固定的代理，当然使用`静态代理`就能很好满足需求。
- 如果要为大量不同类的不同方法生成代理，使用静态代理的话就需要编写大量的代理类，且大量代码冗余，此时`动态代理`就应该闪亮登场了。

Java中实现动态代理常用的技术包括`JDK的动态代理`、`CGLib`等。

## JDK的动态代理

### 快速入门

假设我们的业务系统中有对用户(`UserService`)和商品(`ProductService`)的查询(`query`)和删除(`delete`)业务逻辑，代码如下：

```java
public interface CommonService {
    Object query(Long id);

    void delete(Long id);
}

public class UserService implements CommonService {
    @Override
    public Object query(Long id) {
        String s = "查询到用户：" + id;
        System.out.println(s);
        return s;
    }

    @Override
    public void delete(Long id) {
        System.out.println("已删除用户：" + id);
    }
}

public class ProductService implements CommonService {
    @Override
    public Object query(Long id) {
        String s = "查询到商品：" + id;
        System.out.println(s);
        return s;
    }

    @Override
    public void delete(Long id) {
        System.out.println("已删除商品：" + id);
    }
}
```

现在想用`代理模式`给这些`Service`统一加上业务处理时间的日志(`log`)，如果使用`静态代理`，那么拿上面的例子来说就要再手动写代理类，但实际的业务系统肯定远不止这2个类，那么就需要写大量的相似冗余的代码。

那么使用JDK提供的`动态代理`，应该如何实现呢？

(1) 编写日志处理器`LogHandler`，该处理器需要实现java中的`InvocationHandler`接口中的`invoke`方法

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class LogHandler implements InvocationHandler {
    // 目标对象
    private Object target;

    public LogHandler(Object target) {
        this.target = target;
    }

    private void preHandle() {
        System.out.println("开始处理请求时间: " + System.currentTimeMillis());
    }

    private void postHandle() {
        System.out.println("结束处理请求时间: " + System.currentTimeMillis());
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 请求处理前 记录日志
        preHandle();
        // 目标对象的业务处理逻辑
        Object result = method.invoke(target, args);
        // 请求处理完成 记录日志
        postHandle();
        return result;
    }
}
```

(2) 生成代理对象，并测试代理是否生效

```java
public static void main(String[] args) {
    /**
     * @see sun.misc.ProxyGenerator#saveGeneratedFiles
     * jdk1.8加上这样的配置(其他版本应当取找sun.misc.ProxyGenerator#saveGeneratedFiles用的是什么)
     * 会将运行时生成的代理Class落磁盘，方便我们查看动态代理生成的class文件。jdk1.8应该是在当前项目根目录的com/sun/proxy目录
     * 注意：在main方法中加该配置
     */
    System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");

    // 原对象
    CommonService userService = new UserService();
    CommonService productService = new ProductService();

    // 代理对象
    CommonService proxyUserService = (CommonService) Proxy.newProxyInstance(
            CommonService.class.getClassLoader(),
            new Class[]{CommonService.class},
            new LogHandler(userService));

    CommonService proxyProductService = (CommonService) Proxy.newProxyInstance(
            CommonService.class.getClassLoader(),
            new Class[]{CommonService.class},
            new LogHandler(productService));

    // 测试代理是否生效
    proxyUserService.query(1L);
    System.out.println("----------");
    proxyUserService.delete(1L);

    System.out.println("\n");

    proxyProductService.query(1L);
    System.out.println("----------");
    proxyProductService.delete(1L);
}
```

(3) 运行结果

```sh
开始处理请求时间: 1594528163163
查询到用户：1
结束处理请求时间: 1594528163163
----------
开始处理请求时间: 1594528163163
已删除用户：1
结束处理请求时间: 1594528163163


开始处理请求时间: 1594528163163
查询到商品：1
结束处理请求时间: 1594528163163
----------
开始处理请求时间: 1594528163163
已删除商品：1
结束处理请求时间: 1594528163163
```

可见，通过代理模式增加统一日志处理生效了，而且即便是给多个不同类的对象添加统一日志处理，写一个`LogHandler`就够了，不用为每个类额外写一个对应的代理类。

### 实现原理

```java
System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
```

上面代码中有这样一行，加上这个之后就能把运行时生成的代理class文件写到文件中(在项目根目录的com/sun/proxy下），关键奥秘就在于生成的这个class文件。

运行之后，在当前项目的根目录的com/sun/proxy下，会多出一个`$Proxy0.class`文件，反编译查看源代码(这里去除了`equals()`、`toString()`、`hashCode()`方法)，如下：

```java
package com.sun.proxy;
import com.github.itwild.proxy.CommonService;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements CommonService {
    private static Method m4;
    private static Method m3;

    public $Proxy0(InvocationHandler var1) {
        super(var1);
    }

    public final Object query(Long var1) {
        try {
            return (Object)super.h.invoke(this, m4, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final void delete(Long var1) {
        try {
            super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }
    
    static {
        try {
            m4 = Class.forName("com.github.itwild.proxy.CommonService").getMethod("query", Class.forName("java.lang.Long"));
            m3 = Class.forName("com.github.itwild.proxy.CommonService").getMethod("delete", Class.forName("java.lang.Long"));
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

看到上面的代码，你有没有似曾相识的感觉，这不正是博客一开篇介绍的实现代理模式的第一种方式吗(`代理类关联目标对象，实现目标对象实现的接口`)。

我们再理一下生成的代理类的代码逻辑，`$Proxy0`继承了`java.lang.reflect.Proxy`，并实现了`CommonService`接口，对代理类的方法调用(比如说`query()`)实际上都会转发到`super.h`对象的`invoke()`方法调用，再看下`super.h`到底是啥，追踪一下父类`java.lang.reflect.Proxy`可知

```java
/**
 * the invocation handler for this proxy instance.
 */
protected InvocationHandler h;
```

这正是`快速入门`中我们编写的`LogHandler`所实现的`InvocationHandler`接口。这样整个过程就理清了，这里通过`super.h`调用了我们前面编写的`LogHandler`中的处理逻辑。

那么，新的问题又来了，代理类是怎么生成的，我们没有写任何相关的代码，它是怎么知道我需要代理的方法以及方法参数等等。我们在创建代理对象的时候调用`Proxy.newProxyInstance`传入了代理类需要实现的接口

```java
/**
 * Returns an instance of a proxy class for the specified interfaces
 * that dispatches method invocations to the specified invocation
 * handler.
 *
 * @param   loader the class loader to define the proxy class
 * @param   interfaces the list of interfaces for the proxy class
 *          to implement
 * @param   h the invocation handler to dispatch method invocations to
 */
@CallerSensitive
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
```

至于一步步如何生成class的byte[]可先追踪`java.lang.reflect.Proxy`中的`ProxyClassFactory`相关代码

```java
/**
 * A factory function that generates, defines and returns the proxy class given
 * the ClassLoader and array of interfaces.
 */
private static final class ProxyClassFactory
    implements BiFunction<ClassLoader, Class<?>[], Class<?>>
{
    // prefix for all proxy class names
    private static final String proxyClassNamePrefix = "$Proxy";

    // next number to use for generation of unique proxy class names
    private static final AtomicLong nextUniqueNumber = new AtomicLong();

    @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
        for (Class<?> intf : interfaces) {
            /*
             * Verify that the class loader resolves the name of this
             * interface to the same Class object.
             */
            Class<?> interfaceClass = null;
            try {
                interfaceClass = Class.forName(intf.getName(), false, loader);
            } catch (ClassNotFoundException e) {
            }
            if (interfaceClass != intf) {
                throw new IllegalArgumentException(
                    intf + " is not visible from class loader");
            }
            /*
             * Verify that the Class object actually represents an
             * interface.
             */
            if (!interfaceClass.isInterface()) {
                throw new IllegalArgumentException(
                    interfaceClass.getName() + " is not an interface");
            }
            /*
             * Verify that this interface is not a duplicate.
             */
            if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                throw new IllegalArgumentException(
                    "repeated interface: " + interfaceClass.getName());
            }
        }

        String proxyPkg = null;     // package to define proxy class in
        int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

        /*
         * Record the package of a non-public proxy interface so that the
         * proxy class will be defined in the same package.  Verify that
         * all non-public proxy interfaces are in the same package.
         */
        for (Class<?> intf : interfaces) {
            int flags = intf.getModifiers();
            if (!Modifier.isPublic(flags)) {
                accessFlags = Modifier.FINAL;
                String name = intf.getName();
                int n = name.lastIndexOf('.');
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {
            // if no non-public proxy interfaces, use com.sun.proxy package
            proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
        }

        /*
         * Choose a name for the proxy class to generate.
         */
        long num = nextUniqueNumber.getAndIncrement();
        String proxyName = proxyPkg + proxyClassNamePrefix + num;

        /*
         * Generate the specified proxy class.
         */
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
            proxyName, interfaces, accessFlags);
        try {
            return defineClass0(loader, proxyName,
                                proxyClassFile, 0, proxyClassFile.length);
        } catch (ClassFormatError e) {
            throw new IllegalArgumentException(e.toString());
        }
    }
}
```

通过上面一段代码不知道你有没有明白生成的第一个代理类的`ClassName`为什么是`$Proxy0`。通过观察生成的`class $Proxy0 extends Proxy implements CommonService`，我们知道JDK的动态代理必须要针对接口，而上面一段代码也做了合法性检查

```java
if (!interfaceClass.isInterface()) {
    throw new IllegalArgumentException(
        interfaceClass.getName() + " is not an interface");
}
```

然后就要往`sun.misc.ProxyGenerator#generateProxyClass()`方法里看了

```java
private static final boolean saveGeneratedFiles = (Boolean)AccessController.doPrivileged(new GetBooleanAction("sun.misc.ProxyGenerator.saveGeneratedFiles"));

public static byte[] generateProxyClass(final String var0, Class<?>[] var1, int var2) {
    ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
    final byte[] var4 = var3.generateClassFile();
    if (saveGeneratedFiles) {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                try {
                    // 省略...
                    Files.write(var2, var4, new OpenOption[0]);
                    return null;
                } catch (IOException var4x) {
                    throw new InternalError("I/O exception saving generated file: " + var4x);
                }
            }
        });
    }

    return var4;
}
```

这里看到了为什么我们要在main方法一开始加上`sun.misc.ProxyGenerator.saveGeneratedFiles`配置就是为了让生成的代理class字节码落盘生成文件。

继续就是`ProxyGenerator#generateClassFile()`如何根据`className`、`interfaces`生成classfile的byte[]以及如何得到`class`对象`java.lang.reflect.Proxy#defineClass0`，有兴趣可以深入探究。

```java
private static native Class<?> defineClass0(ClassLoader loader, String name,
                                                byte[] b, int off, int len); 
```

### 注意事项

JDK的动态代理是不需要第三方库支持的，被代理的对象必须要实现接口。

## CGLib

CGLib(`Code Generation Library`)是一个功能较为强大、性能也较好的代码生成包，在许多AOP框架中得到广泛应用。

### 快速入门

除了`UserService`、`ProductService`，还有订单业务(`OrderService`)也需要用代理模式添加统一日志处理，但是注意，`OrderService`并没有实现任何接口，且`delete()`方法用`final`修饰。

```java
public class OrderService {
    public Object query(Long id) {
        String s = "查询到订单：" + id;
        System.out.println(s);
        return s;
    }

    public final void delete(Long id) {
        System.out.println("已删除订单：" + id);
    }
}
```

我们知道，JDK的动态代理必须要求实现了接口，而`cglib`没有这个限制。具体操作如下：

(1) 引入cglib的maven依赖

```sh
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

(2) 编写方法拦截器

```java
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

public class LogInterceptor implements MethodInterceptor {

    private void preHandle() {
        System.out.println("开始处理请求时间: " + System.currentTimeMillis());
    }

    private void postHandle() {
        System.out.println("结束处理请求时间: " + System.currentTimeMillis());
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        // pre handle
        preHandle();
        // Invoke the original (super) method on the specified object
        Object object = proxy.invokeSuper(obj, args);
        // post handle
        postHandle();
        return object;
    }
}
```

(3) 生成代理对象，并测试代理是否生效

```java
public static void main(String[] args) {
    // 指定目录生成动态代理类class文件
    System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "/tmp/cglib");

    Enhancer enhancer = new Enhancer();
    // set the class which the generated class will extend
    enhancer.setSuperclass(OrderService.class);
    // set the single Callback to use
    enhancer.setCallback(new LogInterceptor());

    // generate a new class
    OrderService proxy = (OrderService) enhancer.create();

    proxy.query(1L);
    System.out.println();
    proxy.delete(1L);
}
```

(4) 运行结果

```sh
开始处理请求时间: 1594653500162
查询到订单：1
结束处理请求时间: 1594653500183

已删除订单：1
```

可见，对`OrderService`的`query()`方法实现了代理，而被`final`修饰的`delete()`方法没有被代理。

### 实现原理

非常类似学习JDK的动态代理，这里我们同样反编译生成的代理class文件，去除其他暂时这里不关注的信息，代码如下：

```java
import java.lang.reflect.Method;
import net.sf.cglib.core.ReflectUtils;
import net.sf.cglib.core.Signature;
import net.sf.cglib.proxy.Callback;
import net.sf.cglib.proxy.Factory;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class OrderService$$EnhancerByCGLIB$$ba8463fa extends OrderService implements Factory {
    private static final Method CGLIB$query$0$Method;
    private static final MethodProxy CGLIB$query$0$Proxy;

    static void CGLIB$STATICHOOK1() {
        CGLIB$query$0$Method = ReflectUtils.findMethods(new String[]{"query", "(Ljava/lang/Long;)Ljava/lang/Object;"}, (var1 = Class.forName("com.github.itwild.proxy.OrderService")).getDeclaredMethods())[0];
        CGLIB$query$0$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/Long;)Ljava/lang/Object;", "query", "CGLIB$query$0");
    }

    final Object CGLIB$query$0(Long var1) {
        return super.query(var1);
    }

    public final Object query(Long var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        return var10000 != null ? var10000.intercept(this, CGLIB$query$0$Method, new Object[]{var1}, CGLIB$query$0$Proxy) : super.query(var1);
    }

    public static MethodProxy CGLIB$findMethodProxy(Signature var0) {
        String var10000 = var0.toString();
        switch(var10000.hashCode()) {
        case -508378822:
            if (var10000.equals("clone()Ljava/lang/Object;")) {
                return CGLIB$clone$4$Proxy;
            }
            break;
        case 842547398:
            if (var10000.equals("query(Ljava/lang/Long;)Ljava/lang/Object;")) {
                return CGLIB$query$0$Proxy;
            }
            break;
        case 1826985398:
            if (var10000.equals("equals(Ljava/lang/Object;)Z")) {
                return CGLIB$equals$1$Proxy;
            }
            break;
        case 1913648695:
            if (var10000.equals("toString()Ljava/lang/String;")) {
                return CGLIB$toString$2$Proxy;
            }
            break;
        case 1984935277:
            if (var10000.equals("hashCode()I")) {
                return CGLIB$hashCode$3$Proxy;
            }
        }

        return null;
    }
    
    static {
        CGLIB$STATICHOOK1();
    }
}
```

观察`OrderService$$EnhancerByCGLIB$$ba8463fa`得知该类继承了`OrderService`，并且override了`query(Long id)`方法，而`delete`方法被`final`修饰不能被重写。

到了这里，不知道你有没有想起开篇讲到的实现代理模式的第二种方式(`代理类继承目标类，重写需要代理的方法`)。这里应用的正是这种。

关于cglib更详细的介绍并不是这里的重点，后面我会抽时间细致学习学习做个笔记出来。不过这里还是要多提几句。

当调用代理类的`query()`方法时，会寻找该`query()`方法上有没有被绑定拦截器(比如说编写代码时实现的`MethodInterceptor`接口)，没有的话则不需要代理。JDK动态代理的拦截对象是通过反射的机制来调用被拦截方法的，反射的效率较低，cglib采用了[FastClass](https://www.jianshu.com/p/0604d79435f1) 的机制来实现对被拦截方法的调用。FastClass机制会对一个类的方法建立索引，通过索引来直接调用相应的方法，提高了效率。

# 参考文章

- 本文转载自[一文读懂Java中的动态代理](https://www.cnblogs.com/bytesfly/p/dynamic-proxy-in-java.html)

