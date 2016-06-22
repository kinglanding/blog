---
title: dive into annotation
date: 2016-06-22 15:21:00
tags:
- 注解
- java
- Annotation
- 来自陈年佳酿的笔记
---

初识注解，还是源自懵懂年代查看开源代码中的`@Override`,`@Deprecated`,`@SuppressWarnings`,这种新特性引进与jdk5.0, 借助于编辑器的功劳，
能为平常的代码编写提供很多的帮助。

注解 | 功能
-----|------
Override    | 表明当前方法是覆盖了父类方法，好的编程习惯，编译器会检查代码的某些错误 
Deprecated    |  表明当前的元素已经不推荐使用    
SuppressWarnings    | 关闭不当的编译器警告信息

# 自定义注解

在编写自定义注解之前，有必要了解下`java.lang.annotation`这个包

<!-- more -->

## 元注解(meta-annotation)

元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解,
UML图如下所示:

![java.lang.annotation UML图](/img/post/dive-into-annotation.png)

其中最为重要的就是红色框内的类。包含4个元注解，2个枚举类。

* Retention
* Documented
* Target
* Inherited

以上4个注解即为jdk提供的元注解，因为他们的Target的适应类型都是`ElementType.ANNOTATION_TYPE`,只能用于修饰其他注解。

元注解 | 作用
-----|------
Retention    | 描述注解的生命周期，详见RetentionPoicy解释
Documented    | 若一个注解A使用@Document修饰，那么被A修饰的类Class，可以使用javadoc编译Class，A将会显示
Target    | 用于描述注解的使用范围
Inherited    | 若一个注解A使用@Inherited修饰，那么被A修饰的类Parent的子类Child会继承注解A的修饰


另外两个枚举类是`ElementType`和`RetentionPolicy`,见如下表格:

ElementType | 作用
-----|------
TYPE    | 描述类、接口(包括注解类型)或enum声明
FIELD    | 域声明（包含枚举常量） 
METHOD    | 方法声明
PARAMETER    | 参数声明
CONSTRUCTOR    |  构造函数声明  
LOCAL_VARIABLE    | 局部变量声明
ANNOTATION_TYPE    | 注解类型声明，和TYPE相比，这个类型只能用于注解！（元注解的Target都是这个声明） 
PACKAGE    | 包声明

ElementType | 作用
-----|------
SOURCE    | 被该字段修饰的注解，会被编译器丢弃
CLASS    | 编译器会将该注解记录到class文件中，不会被加载到虚拟机中，这是Target的默认值
RUNTIME    | 编译器会将该注解记录到class文件中，也会被加载到虚拟机中。所以有可能会通过反射读取。

## 如何编写自定义的注解

定义注解，格式如下：
```java
@meta-annotation A
@meta-annotation B
@meta-annotation ...
public @interface AnnotationName {
    // 方法实际上是声明了一个配置参数, 方法的名称就是参数的名称
    // 可以通过default来声明参数的默认值。(option)
    public TYPE function-name() [ default {vlaue} ]
}
```

`@meta-annotation A`是一些元注解；
@interface自定义注解，自动继承了java.lang.annotation.Annotation接口，由编译器完成内部细节。
`TYPE`是返回值类型，只能是基本类型，Class，String，enum和Annotation以及他们的数据类型。
方法名即参数名，使用时类型这样 `@AnnotationName(function-name = {value})`

## 例子

Doc注解
```java
package jls.basic.annotation;

import java.lang.annotation.*;

/**
 * 使用Documented （meta-annotation 元注释），用来
 * 生成javadoc
 */
@Documented
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
public @interface Doc {
}
```
Document注解
```java
package jls.basic.annotation;

import java.lang.annotation.*;

/**
 * 使用Documented （meta-annotation 元注释），用来
 * 生成javadoc,这个保留到运行期
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Document {
    String value() default "";
}
```
Color注解
```java
package jls.basic.annotation.use;

import java.lang.annotation.*;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Color {
    ColorPolicy value() default ColorPolicy.RED;
}
```
Name注解
```java
package jls.basic.annotation.use;

import java.lang.annotation.*;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Name {
    String value() default "";
}
```
ColorPolicy 枚举
```java
package jls.basic.annotation.use;
public enum ColorPolicy {
    RED,
    GREEN,
    BLACK,
    PUPLE,
    YELLOW
}
```

测试用例，用来说明如何使用注解。
```java
package jls.basic.annotation.use;

import jls.basic.annotation.Doc;
import jls.basic.annotation.Document;

import javax.annotation.PostConstruct;
import java.lang.annotation.Annotation;
import java.lang.reflect.Field;

@Doc// 这是一个SOURCE级别的注解
@Document("Document是RUNTIME级的")
public class Banana {
    @Name(value = "banana")
    protected String name;
    @Color(value = ColorPolicy.BLACK)
    protected ColorPolicy color;

    public Banana() {
    }


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public ColorPolicy getColor() {
        return color;
    }

    public void setColor(ColorPolicy color) {
        this.color = color;
    }

    @PostConstruct
    public void display() {
        System.out.println(toString());
    }

    @Override
    public String toString() {
        return "Banana{" +
                "name='" + name + '\'' +
                ", color=" + color +
                '}';
    }

    public static void main(String[] args) {
        Banana banana = new Banana();
        System.out.println(banana);
     
        // 要想使用注解，还是得自己动手丰衣足食啊~
        
        Class clz = Banana.class;
        // 看是否有Doc类型的注解。无，因为Doc注解的Retention是SOURCE
        System.out.println(clz.getAnnotation(Doc.class));
        // 看是否有 Document 类型的注解。有，因为Document注解的Retention是 RUNTIME
        System.out.println(clz.getAnnotation(Document.class));
        // 获取class level的注解值
        Document document = (Document) clz.getAnnotation(Document.class);
        System.out.println(document.value());

        // 获取 类 这一level上的所有注解
        Annotation[] annotations = clz.getDeclaredAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }

        // 分析字段，像Spring初始化bean的方式那样设置字段的值
        Field[] fields = clz.getDeclaredFields();
        for (Field field : fields) {
            if (field.isAnnotationPresent(Color.class)) {
                banana.setColor(field.getAnnotation(Color.class).value());
            } else if (field.isAnnotationPresent(Name.class)) {
                banana.setName(field.getAnnotation(Name.class).value());
            }
        }
        System.out.println(banana);
    }
}
```

`Banana`使用注解修饰了name和color，是不是给人一种感觉:"只要我初始化一个对象，它就应该有默认值啦~"，上述例子main函数的前两行说明这个现象，new了一个对象，
打印下，诶？ 怎么没有值？注解并不像在无参构造函数中给某些字段赋初值。注解如果没有额外的操作，是不起作用的。

所以有了下面的那些操作，通过反射获取注解，拿到他们的值，在给对象设置值，ok，其实这就是Spring底层初始化bean的逻辑，通过bean工厂类，拿到bean中的注解值，
返回一个有值的对象，托管到Spring容器中.

所以呢，注解要和反射结合起来，才能起作用。后面再总结啦。 

## 总结

有个思维导图不错，侵权立删
![注解思维导图](http://images.cnitblog.com/blog/34483/201304/25200814-475cf2f3a8d24e0bb3b4c442a4b44734.jpg)


> Reference
1. [图片来源](http://www.cnblogs.com/peida/archive/2013/04/26/3038503.html)
2. [Java注解（1）-基础](http://blog.csdn.net/duo2005duo/article/details/50505884)