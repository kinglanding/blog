---
title: java基础系列2：反射
date: 2016-06-23 11:48:51
tags:
- java
- 反射
- reflect
---

反射功能强大，运用运行时获取任意类的任何属性和方法。功能虽强大，单不要滥用哦。反射的概念是由Smith在1982年首次提出的，
主要是指程序可以访问、检测和修改它本身状态或行为的一种能力。这一概念的提出很快引发了计算机科学领域关于应用反射性的研究。
它首先被程序语言的设计领域所采用,并在Lisp和面向对象方面取得了成绩。当然反射本身并不是一个新概念，
它可能会使我们联想到光学中的反射概念，尽管计算机科学赋予了反射概念新的含义，但是，从现象上来说，它们确实有某些相通之处，这些有助于我们的理解。

## 反射的应用场景

反射通常用来在运行期检测和修改应用的行为，是一项相对高级的特性。
* 可扩展<br>
  　　应用可以使用外部或者用户自定义的类，来创建实例。
* 类浏览、IDE<br>
  　　类浏览器需要读取类的成员。嘿，瞧瞧你现在使用的IDE，是不是有个地方可以浏览类的成员对象？这种类元信息可视化的功能就是借助于反射实现的。
  各种开发工具就是凭借这个特性读取的类型信息，帮助开发及时发现错误。
* 调试、测试工具<br>
  　　调试器需要获取类的私有成员。测试工具可以用来获取类的所有方法，从而编写对应的测试用例，保证覆盖到所有方法。

通过上面的描述，是不是有了感性的认识？原来好多地方都使用到了反射，好厉害，我也要多多用用他们！且慢，往下看。

## 反射的缺点

尽管反射功能强大，但是不能肆意使用。如果可以不通过反射就能达到目的，最好就是避免使用它。在使用反射访问代码时，以下几点需要注意：

* 性能开销<br>
　　因为反射涉及到动态类型解析，所以一些java优化指令不会被执行。因此，翻身操作比起一般操作来说，性能低多了。性能敏感类应用应该避免
  在调用频繁的地方使用反射。
* 安全限制<br>
  　　反射需要运行时权限。但是当运行在安全管理器下，这个权限有可能不存在。例如Applet类应用需要运行在严格的安全环境中，此时就必须好好考虑下如何
  权衡反射和安全的问题了。
* 内部暴漏<br>
  　　因为反射执行一些‘非法’操作，相对*非反射*而言,例如访问*private*成员（变量和方法），有可能会导致一些副作用，例如代码功能失调或者丧失移植性。

# 应用
## Classes
　　每个对象要不然是一引用类型，要不然就是基本类型。所有的引用类型都继承自`java.lang.Object`，类，枚举，队列和接口都是引用类型。而节本类型包含
`boolean`, `byte`, `short`, `int`, `long`, `char`, `float`,和`double`。而引用类型，比如`String`、基本类型的包装类，接口`java.io.Serializable`,
以及枚举类型`SortOrder`。


对于每个类型，虚拟机都实例化了一个不可变的的`java.lang.Class`实例，可以在运行时提供方法，用于获取类型信息和成员。Class还可以创建新的类型和对象。
最重要的是，踏射所有反射API的入口。

### 获取类对象

`java.lang.Class`提供了所有的反射操作方法。除了`java.lang.reflect.ReflectPermission`,所有`java.lang.reflect`包内的类都没有公共
构造函数。要想获取这些类，在`Class`中提供一些方法是必要的。提供了几种方式：1）使用对象；2）类名；3）现有的类。

#### Object.getClass()
最简单的方式就是调用`Object.getClass()`.

```java
Class c = "foo".getClass();
```
返回结果是String。

```java
enum E { A, B }
Class c = A.getClass();
```
因为A是枚举E的实例，返回值E
```java
byte[] bytes = new byte[1024];
Class c = bytes.getClass();
```
数组是Objects类型，当然可以调用`getClass()`，返回结果是数组类型和字节的组合`[B`.

#### .class语法
如果类型已知，但是对象并没有初始化，调用getClass方法会出错，此时可以使用.class的方式的得到类信息。我觉得这是最最简单的方式了。。。
```java
boolean b;
Class c = b.getClass();   // compile-time error

Class c = boolean.class;  // correct
```
注意，boolean.getClass()是产生编译时错误，因为boolean是基本类型，是没有引用的。
#### Class.forName()

如果已知一个类的全名，可以使用类的静态方法`Class.forName()`.记住一点，这个不适用于基本类型。`Class.getName()`对引用类型和基本类型都适用。
例如：
```java
Class cDoubleArray = Class.forName("[D");
Class cStringArray = Class.forName("[[Ljava.lang.String;");
```

#### 基本类型获取类型的方法
使用.class可以很方便的获取基本类型的类型，除此之外，还可以使用`包装类.TYPE`的方式获取...
例如:
```
Class c = Double.TYPE;
Class c = Void.TYPE;
```
`Double.TYPE`等同于`double.class`.

#### 一些返回类型的方法
有一些反射API可以返回类型

**Class.getSuperclass()**<br>
　　返回所给类的父类

**Class.getClasses()**<br>
　　返回类中所有*公共*的类，接口和枚举，包含继承类成员。当然也是public属性的。

**Class.getDeclaredClasses()**<br>
　　返回类中所有的类，接口，枚举。包括*private*修饰的类型

**Class.getDeclaringClass()<br>
java.lang.reflect.Field.getDeclaringClass()<br>
java.lang.reflect.Method.getDeclaringClass()<br>
java.lang.reflect.Constructor.getDeclaringClass()**<br>
　　返回声明这个类、域、方法或者构造函数的类。例如类ReflectTest中声明了一个B，那么B的声明类就是A。
```java
package jls.basic.reflect;
public class ReflectTest {
    public static void main(String[] args) throws ClassNotFoundException {
        System.out.println(B.class.getDeclaringClass());
    }
    class B {}
}
```

数据结果是class jls.basic.reflect.ReflectTest

匿名类声明不会有声明它的类，但是会拥有**包含（Enclosing）**它的类。
```java
    // 匿名类声明
    public static Object anonymous  = new Object() {
        public void m() {
        }
    };
    public static Class claz = anonymous .getClass().getDeclaringClass();
```

匿名类的声明类是null。

**Class.getEnclosingClass()**<br>
　　返回包含该类的类，例如上述例子中的B被ReflectTest包含。
```java
    // 匿名类声明
    public static Object anonymous  = new Object() {
        public void m() {
        }
    };
    public static Class claz = anonymous .getClass().getEnclosingClass();
```
匿名类此时的被包含类仍然是null...


### 检查类的修饰符合类型
类可以拥有一到多个修饰符，从而影响类的运行时行为。

- 访问修饰符： public, protected 和 private
- 需要重载的修饰符： abstract
- 保证只有一个实例的修饰符： static
- 防止值修改的修饰符： final
- 强制执行IEEE 754浮点数标准修饰符： strictfp
- 注解

注意，不是所有的修饰符，都可以修饰所有的类型，接口就不能使用`final`来修饰，枚举不能使用`abstract`修饰。`java.lang.reflect.Modifier`包含了各种
可能修饰符的声明。

下面样例介绍了如何获取一个类的各个声明部分：修饰符，泛型参数，接口和继承链。1.5版本的 Class实现了 `java.lang.reflect.AnnotatedElement`的接口。
所以也可以获取类的运行时注解。

```java

import java.lang.annotation.Annotation;
import java.lang.reflect.Modifier;
import java.lang.reflect.Type;
import java.lang.reflect.TypeVariable;
import java.util.ArrayList;
import java.util.List;

import static java.lang.System.out;

public class ClassDeclarationSpy {
    public static void main(String... args) {
        try {
            Class<?> c = Class.forName(args[0]);
            // 获取类的全名
            out.format("Class:%n  %s%n%n", c.getCanonicalName());
            // 获取类的修饰符
            out.format("Modifiers:%n  %s%n%n",
                    Modifier.toString(c.getModifiers()));

            // 获取泛型参数 K V
            out.format("Type Parameters:%n");
            TypeVariable[] tv = c.getTypeParameters();
            if (tv.length != 0) {
                out.format("  ");
                for (TypeVariable t : tv)
                    out.format("%s ", t.getName());
                out.format("%n%n");
            } else {
                out.format("  -- No Type Parameters --%n%n");
            }

            // 获取是实现的接口
            out.format("Implemented Interfaces:%n");
            Type[] intfs = c.getGenericInterfaces();
            if (intfs.length != 0) {
                for (Type intf : intfs)
                    out.format("  %s%n", intf.toString());
                out.format("%n");
            } else {
                out.format("  -- No Implemented Interfaces --%n%n");
            }

            // 获取继承链
            out.format("Inheritance Path:%n");
            List<Class> l = new ArrayList<Class>();
            printAncestor(c, l);
            if (l.size() != 0) {
                for (Class<?> cl : l)
                    out.format("  %s%n", cl.getCanonicalName());
                out.format("%n");
            } else {
                out.format("  -- No Super Classes --%n%n");
            }

            // 获取注解
            out.format("Annotations:%n");
            Annotation[] ann = c.getAnnotations();
            if (ann.length != 0) {
                for (Annotation a : ann)
                    out.format("  %s%n", a.toString());
                out.format("%n");
            } else {
                out.format("  -- No Annotations --%n%n");
            }

            // production code should handle this exception more gracefully
        } catch (ClassNotFoundException x) {
            x.printStackTrace();
        }
    }

    private static void printAncestor(Class<?> c, List<Class> l) {
        Class<?> ancestor = c.getSuperclass();
        if (ancestor != null) {
            l.add(ancestor);
            printAncestor(ancestor, l);
        }
    }
}
```

然后这么使用

```java
java ClassDeclarationSpy java.util.concurrent.ConcurrentNavigableMap
输出：
Class:
  java.util.concurrent.ConcurrentNavigableMap

Modifiers:
  public abstract interface

Type Parameters:
  K V

Implemented Interfaces:
  java.util.concurrent.ConcurrentMap<K, V>
  java.util.NavigableMap<K, V>

Inheritance Path:
  -- No Super Classes --

Annotations:
  -- No Annotations --
```

还可以这么使用
```java
ava ClassDeclarationSpy "[Ljava.lang.String;"
Class:
  java.lang.String[]

Modifiers:
  public abstract final

Type Parameters:
  -- No Type Parameters --

Implemented Interfaces:
  interface java.lang.Cloneable
  interface java.io.Serializable

Inheritance Path:
  java.lang.Object

Annotations:
  -- No Annotations --
```
`[Ljava.lang.String;`代表String[], 因为String类的修饰符就是final，并且实现了Cloneable和Serializable接口，而且数据类型是继承自Object。
解析是正确的。

注意哈，不是所有的注解都会被探测到的，只有那些Retention是RUNTIME的注解才会被加载到虚拟机中，所以一些预定义的注解如`@Override`，`@Deprecated`和
`@SuppressWarnings`,只有`@Deprecated`可以在运行时获取。

### 发现类成员

`Class`类提供了两类方法，用于获取fields，methods和constructors：一类是列举这些成员；一类是通过名字搜索成员。一些方法可以直接获取类本身声明的
成员，相对应的是，一些方法可以搜索父接口和父类的继承成员。下表是`成员定位方法`的一些特性总结。

`Class`类中`域定位`方法

Class API | List of members? | Inherited members? | 	Private members?
-----|------|-----|------
getDeclaredField(name)   | no | no | yes
getField(name)  | no | yes | no
getDeclaredFields()   | yes | no | yes
getFields()   | yes | yes | no

`Class`类中`方法定位`方法

Class API | List of members? | Inherited members? | 	Private members?
-----|------|-----|------
getDeclaredMethod(String name,Class<?>... parameterTypes)   | no | no | yes
getMethod(String name,Class<?>... parameterTypes)  | no | yes | no
getDeclaredMethods()   | yes | no | yes
getMethods()   | yes | yes | no

`Class`类中`构造函数定位`方法

Class API | List of members? | Inherited members? | 	Private members?
-----|------|-----|------
getDeclaredConstructor(Class<?>... parameterTypes)   | no | N/A | yes
getConstructor(Class<?>... parameterTypes)  | no | N/A | no
getDeclaredConstructors()   | yes | N/A | yes
getConstructors()   | yes | N/A | no

注意，构造函数是不会继承的。

下面这个ClassSpy函数，是一个使用get*s()方法获取所有public和继承的元素的例子。

```java
package jls.basic.reflect;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Member;
import java.lang.reflect.Method;

import static java.lang.System.out;

enum ClassMember {CONSTRUCTOR, FIELD, METHOD, CLASS, ALL}

public class ClassSpy {
    public static void main(String[] rag) {
        // 直接在这里修改就可以了。
        String[] args = {"jls.basic.reflect.ClassSpy","ALL"}; 
        try {
            Class<?> c = Class.forName(args[0]);
            out.format("Class:%n  %s%n%n", c.getCanonicalName());

            Package p = c.getPackage();
            out.format("Package:%n  %s%n%n",
                    (p != null ? p.getName() : "-- No Package --"));

            for (int i = 1; i < args.length; i++) {
                switch (ClassMember.valueOf(args[i])) {
                    case CONSTRUCTOR:
                        printMembers(c.getConstructors(), "Constructor");
                        break;
                    case FIELD:
                        printMembers(c.getFields(), "Fields");
                        break;
                    case METHOD:
                        printMembers(c.getMethods(), "Methods");
                        break;
                    case CLASS:
                        printClasses(c);
                        break;
                    case ALL:
                        printMembers(c.getConstructors(), "Constuctors");
                        printMembers(c.getFields(), "Fields");
                        printMembers(c.getMethods(), "Methods");
                        printClasses(c);
                        break;
                    default:
                        assert false;
                }
            }

            // production code should handle these exceptions more gracefully
        } catch (ClassNotFoundException x) {
            x.printStackTrace();
        }
    }

    private static void printMembers(Member[] mbrs, String s) {
        out.format("%s:%n", s);
        for (Member mbr : mbrs) {
            if (mbr instanceof Field)
                out.format("  %s%n", ((Field) mbr).toGenericString());
            else if (mbr instanceof Constructor)
                out.format("  %s%n", ((Constructor) mbr).toGenericString());
            else if (mbr instanceof Method)
                out.format("  %s%n", ((Method) mbr).toGenericString());
        }
        if (mbrs.length == 0)
            out.format("  -- No %s --%n", s);
        out.format("%n");
    }

    private static void printClasses(Class<?> c) {
        out.format("Classes:%n");
        Class<?>[] clss = c.getClasses();
        for (Class<?> cls : clss)
            out.format("  %s%n", cls.getCanonicalName());
        if (clss.length == 0)
            out.format("  -- No member interfaces, classes, or enums --%n");
        out.format("%n");
    }
}
```
例子很简单，注意Field，Method，Constructor都实现了Member接口，函数printMembers在根据具体类型进行强制类型转换，打印。

使用也很简单，直接运行，就好。运行结果如下:

```
Class:
  jls.basic.reflect.ClassSpy

Package:
  jls.basic.reflect

Constuctors:
  public jls.basic.reflect.ClassSpy()

Fields:
  -- No Fields --

Methods:
  public static void jls.basic.reflect.ClassSpy.main(java.lang.String[])
  public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
  public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
  public final void java.lang.Object.wait() throws java.lang.InterruptedException
  public boolean java.lang.Object.equals(java.lang.Object)
  public java.lang.String java.lang.Object.toString()
  public native int java.lang.Object.hashCode()
  public final native java.lang.Class<?> java.lang.Object.getClass()
  public final native void java.lang.Object.notify()
  public final native void java.lang.Object.notifyAll()

Classes:
  -- No member interfaces, classes, or enums --
```

### 成员

反射定义了`Member`接口，`Feild`,`Method`和`Constructor`都实现了该接口。下面分别讲述下每类成员的使用方法。
注意java语言规范，java SE 7 版本规定了，一个类的成员包括field，method，嵌套类，接口和枚举类型，已经继承自父类的
这类成员。因为构造函数是不能被继承的，他们不属于成员。这和Member接口的实现类（有Constructor）迥然不同..

####  域
域都有类型和值，`java.lang.reflect.Field`类提供了获取类型信息和值的get和set方法。

##### 获取字段类型

域可能是基本类型，也可能是引用类型。基本类型包括8类：boolean，byte，short，int，long，char，float和double。引用类型就
直接或间接继承Object类型的类和接口，数组以及枚举类型。

下面这个程序可以打印类的字段信息。

```java
package jls.basic.reflect;
import java.lang.reflect.Field;
import java.util.List;

public class FieldSpy<T> {
    public boolean[][] b = {{false, false}, {true, true}};
    public String name = "Alice";
    public List<Integer> list;
    public T val;

    public static void main(String[] arg) {
        try {
            // class name
            String[] args = {"jls.basic.reflect.FieldSpy", "b"};

            Class<?> c = Class.forName(args[0]);
            Field[] fields = c.getFields();
            for (Field field : fields) {
                System.out.format("Name: %s%n", field.getName());
                System.out.format("Type: %s%n", field.getType());
                System.out.format("GenericType: %s%n", field.getGenericType());
                System.out.println();
            }
            // production code should handle these exceptions more gracefully
        } catch (ClassNotFoundException x) {
            x.printStackTrace();
        }
    }
}
```
输出结果如下：
```java
Name: b
Type: class [[Z
GenericType: class [[Z

Name: name
Type: class java.lang.String
GenericType: class java.lang.String

Name: list
Type: interface java.util.List
GenericType: java.util.List<java.lang.Integer>

Name: val
Type: class java.lang.Object
GenericType: T
```

Field.getType()返回的是Class<?>类型，需要注意的是，val的类型是Object，这是因为泛型在编译期经过类型擦除之后，所有的信息都不见了。所以T的类型就被顶层类型替换掉了，也就是成了Object类型。

Field.getGenericType返回的是Type类型，这个函数将会查询class文件中的签名属性，如果不能获取该属性，那么它就会调用Field.getType().

#### 获取域的修饰符

#### 设置域的值

程序写起来很简单，不在列举了。只是有个地方需要注意下：通过反射设置域的值会带来一定的性能损耗，因为会涉及一些权限验证等操作。所以使用反射会使运行时的优化下降。
例如如下的代码很有可能被java虚拟机优化：
```java
int x=1;
x=2;
x=3;
```
如果使用Field.set*(),那么优化就不会发生了，每行都会执行一遍。。

### 方法

方法都拥有返回值，形参和异常。Method类提供了获取形参和返回值的类型信息方法。也可以调用给定对象的方法。

* 获取方法的类型信息
* 获取方法形参的名字
* 获取解析方法修饰符
* 方法调用

反射提供了一种方法调用的方式，一般很少使用，除非强制类型转化不起作用，才有必要使用这种方式。结合jdk中的源码和下面的例子，会有更好的感性认识。
直接看代码和注释就好，可以直接运行这个代码增加感性认识。

```java
package jls.basic.reflect;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

import static java.lang.System.out;

public class MethodInvoker<T> {
    // 私有函数，不可能被访问。使用反射修改修饰符，然后在调用

    // 无形参
    private void helloWorld() {
        out.println("Hello world");
    }

    // 有形参，有返回值
    private String hello(String name) {
        String sentence = "Hello " + name;
        return sentence;
    }

    public static void main(String... args) {

        try {
            // 加载本类
            Class<?> c = Class.forName("jls.basic.reflect.MethodInvoker");
            // 实例化1个本类的对象
            Object t = c.newInstance();
            // getDeclaredMethod 用来获取私有函数

            // 调用helloWorld 这个私有方法,所以方法名是helloWorld
            String privateMethodName = "helloWorld";
            // 因为helloWorld 的无形参，所以构造一个大小为零的数组
            Class[] helloWorldMethodParameterTypes = new Class[0];
            // 获取私有函数
            Method helloWorldMethod = c.getDeclaredMethod(privateMethodName, helloWorldMethodParameterTypes);

            // 调用私有方法... 不用设置访问属性..
            //helloWorldMethod.setAccessible(true);
            out.format("invoking %s()%n", helloWorldMethod.getName());
            Object result = helloWorldMethod.invoke(t, null);
            // 返回值是null
            out.format("%s() returned: %s%n", helloWorldMethod.getName(), result);

            // 调用 hello 这个私有方法,所以方法名是helloWorld
            privateMethodName = "hello";
            // hello 方法有一个String类型的参数，所以构造一个大小为1的数组
            Class[] helloMethodParameterTypes = new Class[]{String.class};
            String parameter = "aluen";
            // 获取私有函数
            Method helloMethod = c.getDeclaredMethod(privateMethodName, helloMethodParameterTypes);

            // 调用私有方法
            //helloMethod.setAccessible(true);
            out.format("invoking %s()%n", helloMethod.getName());
            result = helloMethod.invoke(t, parameter);
            // 返回值是String
            out.format("%s() returned: %s%n", helloMethod.getName(), result);

            // production code should handle these exceptions more gracefully
        } catch (ClassNotFoundException x) {
            x.printStackTrace();
        } catch (InstantiationException x) {
            x.printStackTrace();
        } catch (IllegalAccessException x) {
            x.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```

### 构造函数


