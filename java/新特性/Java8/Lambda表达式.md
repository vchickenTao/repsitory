# 🍼 Java8 新特性 - Lambda表达式

---

## 概念

### 什么是Lambda表达式？

`入`希腊字母表中排序第十一位的字母，英语名称为Lambda

`Lambda 表达式`，也称为`闭包`，是 Java 8 中最大和最令人期待的语言改变。它允许我们将函数当成参数传递给某个方法，或者把代码本身当作数据处理，函数式开发者非常熟悉这些概念。

很多JVM平台上的语言(Groovy、Scala等)从诞生之日就支持 Lambda 表达式，但是 Java 开发者没有选择，只能使用**匿名内部类**代替Lambda表达式。

例如：

```java
//匿名内部类方式排序 
List<String> names = Arrays.asList( "a", "b", "d" ); 
 
Collections.sort(names, new Comparator<String>() { 
    @Override 
    public int compare(String s1, String s2) { 
        return s1.compareTo(s2); 
    } 
}); 
```

为了避免内部类定义过多，Java的开发者开始引入Lambda表达式，Lambda 的设计可谓耗费了很多时间和很大的社区力量，最终找到一种折中的实现方案，可以实现简洁而紧凑的语言结构,其实质属于函数式编程的概念。

**Lambda 表达式的语法格式为**：

```java
（params） -> expression  [表达式]
（params） -> statement [语句]
（params） -> {statement }
```



### 为什么要使用lambda表达式

  1.避免匿名内部类定义过多

  2.可以让你的代码看起来很简洁

  3.去掉了一堆没有意义的代码，只留下核心的逻辑



## Lambda的编程风格

Lambda 编程风格，可以总结为四类：

- **可选类型声明**：不需要声明参数类型，编译器可以统一识别参数值
- **可选的参数圆括号**：一个参数无需定义圆括号，但多个参数需要定义圆括号
- **可选的大括号**：如果主体包含了一个语句，就不需要使用大括号
- **可选的返回关键字**：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值

### 2.1、可选类型声明

在使用过程中，我们可以不用显示声明参数类型，编译器可以统一识别参数类型，例如：

```java
Collections.sort(names, (s1, s2) -> s1.compareTo(s2)); 
```

上面代码中的参数s1、s2的类型是由编译器推理得出的，你也可以显式指定该参数的类型，例如：

```java
Collections.sort(names, (String s1, String s2) -> s1.compareTo(s2)); 
```

运行之后，两者结果一致!

### 2.2、可选的参数圆括号

当方法那只有一个参数时，无需定义圆括号，例如：

```java
Arrays.asList( "a", "b", "d" ).forEach( e -> System.out.println( e ) ); 
```

但多个参数时，需要定义圆括号，例如：

```java
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> e1.compareTo( e2 ) ); 
```

### 2.3、可选的大括号

当主体只包含了一行时，无需使用大括号，例如：

```java
Arrays.asList( "a", "b", "c" ).forEach( e -> System.out.println( e ) ); 
```

当主体包含多行时，需要使用大括号，例如：

```java
Arrays.asList( "a", "b", "c" ).forEach( e -> { 
    System.out.println( e ); 
    System.out.println( e ); 
} ); 
```

### 2.4、可选的返回关键字

如果表达式中的语句块只有一行，则可以不用使用return语句，返回值的类型也由编译器推理得出，例如：

```java
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> e1.compareTo( e2 ) ); 
```

如果语句块有多行，可以在大括号中指明表达式返回值，例如：

```java
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> { 
    int result = e1.compareTo( e2 ); 
    return result; 
} ); 
```

### 2.5、变量作用域

还有一点需要了解的是，Lambda 表达式可以引用类成员和局部变量，但是会将这些变量隐式得转换成final，例如：

```java
String separator = ","; 
Arrays.asList( "a", "b", "c" ).forEach( 
    ( String e ) -> System.out.print( e + separator ) ); 
```

和

```java
final String separator = ","; 
Arrays.asList( "a", "b", "c" ).forEach( 
    ( String e ) -> System.out.print( e + separator ) ); 
```

两者等价!

同时，Lambda 表达式的局部变量可以不用声明为final，但是必须不可被后面的代码修改(即隐性的具有 final 的语义)，例如：

```java
int num = 1; 
Arrays.asList(1,2,3,4).forEach(e -> System.out.println(num + e)); 
num =2; 
//报错信息：Local variable num defined in an enclosing scope must be final or effectively final 
```

在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量，例如：

```java
int num = 1; 
Arrays.asList(1,2,3,4).forEach(num -> System.out.println(num)); 
//报错信息：Variable 'num' is already defined in the scope 
```



## 函数式接口的定义

```java
public interface Runnable{
    public abstract void run();
}
```

> 详细介绍请查看[Java8 新特性 - 函数式接口](java/新特性/Java8/函数式接口)一文。



## 如何使用lambda 表达式

### 案例1

```java
/**
 * @Description Testlambda
 * @Author vchicken
 * @Date 2022/8/30 19:31
 */
public class TestLambda {

    //3.静态内部类
    static class Like2 implements ILike {
        @Override
        public void lambda() {
            System.out.println(" I like lambda2");
        }
    }

    public static void main(String[] args) {
        ILike like = new Like();
        like.lambda();

        like = new Like2();
        like.lambda();

        //4.局部内部类
        class Like3 implements ILike {
            @Override
            public void lambda() {
                System.out.println(" I like lambda3");
            }
        }

        like = new Like3();
        like.lambda();

        //5.匿名内部类,没有类的名称，必须借助接口或者父类
        like = new ILike() {
            @Override
            public void lambda() {
                System.out.println(" I like lambda4");
            }
        };

        like.lambda();

        //6.用lambda表达式
        like = ()->{
            System.out.println(" i like lambda5");
        };
        like.lambda();
    }
}

//1.定义一个函数式接口
interface ILike {
    void lambda();
}

//2.实现类
class Like implements ILike {

    @Override
    public void lambda() {
        System.out.println(" I like lambda");
    }
}
```

### 案例2

```java
/**
 * @Description TestLambda2
 * @Author vchicken
 * @Date 2022/8/30 19:42
 */
public class TestLambda2 {

    public static void main(String[] args) {
        class Love implements ILove {

            @Override
            public void love(int a) {
                System.out.println("I love a" + a);
            }
        }

        ILove love = new Love();
        love.love(1);

        love = new ILove() {

            @Override
            public void love(int a) {
                System.out.println("I love b" + a);
            }
        };

        love.love(2);

        love = (a) -> {
            System.out.println("I love c" + a);
        };

        love.love(3);

        love = a -> {
            System.out.println("I love d" + a);
        };

        love.love(4);

        love = a -> System.out.println("I love e" + a);
        
        love.love(5);
    }
}

interface ILove {
    void love(int a);
}
```



## 参考资料

- [遇见狂神说](https://www.bilibili.com/video/BV1V4411p7EF?p=10&vd_source=fbca5d67749f4a9991aefdc97868f643)
- [Java8 新特性全面介绍，强烈建议收藏](https://www.51cto.com/article/647804.html)