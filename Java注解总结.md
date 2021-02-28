---
title: Java注解总结
date: 2017-01-23 16:48:03
categories:  
- 技术总结 
- Java注解
tags:  
- Java 
- 注解
---

## 1.注解说明：

* 注解（Annotation），也叫元数据。一种代码级别的说明。它是JDK 1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。

## 2.注解的作用：

* 标记作用，用于告诉编译器一些信息让编译器能够实现基本的编译检查，如@Override、Deprecated，看下它俩的源码

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}

@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Deprecated {
}
```

* 编译时动态处理，动态生成代码
* 运行时动态处理，获得注解信息

## 3.注解的分类：

* 注解的分类有两种分法

### 3.1 第一种分法

* 基本内置注解，是指Java自带的几个Annotation，如@Override、Deprecated、@SuppressWarnings等
* 元注解（meta-annotation），是指负责注解其他注解的注解，JDK 1.5及以后版本定义了4个标准的元注解类型，如下:

```
@Target
@Retention
@Documented
@Inherited
```

* 自定义注解，根据需要可以自定义注解，自定义注解需要用到上面的meta-annotation

### 3.2 第二种分法

* 源码时注解（RetentionPolicy.SOURCE）
* 编译时注解（RetentionPolicy.CLASS）
* 运行时注解（RetentionPolicy.RUNTIME）

## 4.注解相关知识点

### 4.1 元注解：

* @Target：指Annotation所修饰的对象范围，通过ElementType取值有8种，如下

```java
TYPE：类、接口（包括注解类型）或枚举
FIELD：属性
METHOD：方法
PARAMETER：参数
CONSTRUCTOR：构造函数
LOCAL_VARIABLE：局部变量
ANNOTATION_TYPE：注解类型
PACKAGE：包
```

* @Retention：指Annotation被保留的时间长短，通过RetentionPolicy取值有3种，如下：

```java
SOURCE：在源文件中有效（即源文件保留）  
CLASS：在class文件中有效（即class保留）  
RUNTIME：在运行时有效（即运行时保留）
```

* @Documented：是一个标记注解，用于描述其它类型的注解应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。
* @Inherited：也是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。注意：@Inherited annotation类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation，方法并不从它所重载的方法继承annotation。

### 4.2 内建注解：

* **@Override**——当我们想要复写父类中的方法时，我们需要使用该注解去告知编译器我们想要复写这个方法。这样一来当父类中的方法移除或者发生更改时编译器将提示错误信息。
* **@Deprecated**——当我们希望编译器知道某一方法不建议使用时，我们应该使用这个注解。Java在javadoc 中推荐使用该注解，我们应该提供为什么该方法不推荐使用以及替代的方法。
* **@SuppressWarnings**——这个仅仅是告诉编译器忽略特定的警告信息，例如在泛型中使用原生数据类型。它的保留策略是SOURCE（译者注：在源文件中有效）并且被编译器丢弃。

### 4.3 注解的定义格式：

```java
public @interface 注解名 { 定义体 }
```

### 4.4 注解参数可支持的数据类型：

```java
8种基本数据类型 int、float、boolean、byte、double、char、long、short  
String、Class、enum、Annotation  
以上所有类型的数组
```

### 4.5 注意：

* 自定义注解如果只有一个参数成员，最好把定义体参数名称设为"value"，如@Target

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

## 5.自定义注解:

### 5.1 介绍：

* 使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。@interface用来声明一个注解，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、Annotation、enum及以上类型的数组）。可以通过default来声明参数的默认值。
* 自定义注解格式：public @interface 注解名 {定义体}

### 5.2 注解参数设置：

* 只能用public或默认(default)这两个访问权修饰.例如,String value();这里把方法设为defaul默认类型；
* 参数成员只能用基本类型byte,short,char,int,long,float,double,boolean八种基本数据类型和 String,Enum,Class,annotations等数据类型,以及这一些类型的数组.例如,String value();这里的参数成员就为String;
* 如果只有一个参数成员,最好把参数名称设为"value",后加小括号，这样可以省去注解的参数名：

```java
@FruitName("Apple")
private String appleName;
```

### 5.3 注解元素的默认值：

* 注解元素必须有确定的值，要么在定义注解的默认值中指定，要么在使用注解时指定，非基本类型的注解元素的值不可为null。因此, 使用空字符串或0作为默认值是一种常用的做法。这个约束使得处理器很难表现一个元素的存在或缺失的状态，因为每个注解的声明中，所有元素都存在，并且都具有相应的值，为了绕开这个约束，我们只能定义一些特殊的值，例如空字符串或者负数，一次表示某个元素不存在，在定义注解时，这已经成为一个习惯用法。

## 6.运行时框架：

### 6.1 相关类介绍

* Java使用Annotation接口来代表程序元素前面的注解，Annotation接口是所有Annotation类型的父接口。除此之外，Java在java.lang.reflect 包下新增了AnnotatedElement接口，该接口代表程序中可以接受注解的程序元素，该接口主要有如下几个实现类：

```java
Class：类定义
Constructor：构造器定义
Field：累的成员变量定义
Method：类的方法定义
Package：类的包定义
```

* java.lang.reflect 包下主要包含一些实现反射功能的工具类，实际上，java.lang.reflect 包所有提供的反射API扩充了读取运行时Annotation信息的能力。当一个Annotation类型被定义为运行时的Annotation后，该注解才能是运行时可见，当class文件被装载时被保存在class文件中的Annotation才会被虚拟机读取。
* AnnotatedElement 接口是所有程序元素（Class、Method和Constructor）的父接口，所以程序通过反射获取了某个类的AnnotatedElement对象之后，程序就可以调用该对象的如下四个个方法来访问Annotation信息：
  * 方法1：<T extends Annotation> T getAnnotation(Class<T> annotationClass): 返回改程序元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。
  * 方法2：Annotation[] getAnnotations():返回该程序元素上存在的所有注解。
  * 方法3：boolean isAnnotationPresent(Class<?extends Annotation> annotationClass):判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false。
  * 方法4：Annotation[] getDeclaredAnnotations()：返回直接存在于此元素上的所有注释。与此接口中的其他方法不同，该方法将忽略继承的注释。（如果没有注释直接存在于此元素上，则返回长度为零的一个数组。）该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。

### 6.2 简单示例：

```java
// 1. 自定义注解类
// PersonName
@Inherited
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PersonName {
	String value() default "无名氏";
}
// PersonAge
@Inherited
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PersonAge {
	int age() default -1;
}
// PersonEat
@Inherited
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PersonEat {
	String eat() default "";
}
// 2. 注解的使用
public class Person {
	@PersonName("测试员")
	private String name;

	@PersonAge(age = 18)
	private int age;

//	@PersonEat
	@PersonEat(eat="面条")
	public void haveLunch() {

	}
}
// 3. 注解分析
public class PersonTest {
	public static void main(String[] args) {
		String personInfo = getPersonInfo(Person.class);
		System.out.println(personInfo);
	}
	
	public static String getPersonInfo(Class<?> clazz){
		Field[] fields = clazz.getDeclaredFields();
		Method[] methods = clazz.getDeclaredMethods();
		StringBuilder str = new StringBuilder();
		str.append("我是");
		for(Field field : fields){
			if(field.isAnnotationPresent(PersonName.class)){
				PersonName name = field.getAnnotation(PersonName.class);
				str.append(name.value());
			}else if(field.isAnnotationPresent(PersonAge.class)){
				PersonAge age = field.getAnnotation(PersonAge.class);
				if(age.age()!=-1){
					str.append("我今年"+age.age()+"岁,");
				}
			}
		}
		for(Method method : methods){
			if(method.isAnnotationPresent(PersonEat.class)){
				PersonEat eat = method.getAnnotation(PersonEat.class);
				String eatStr = eat.eat();
				if(eatStr.equals("")){
					str.append("我中午没吃饭！");
				}else{
					str.append("我中午吃的是"+eatStr+"!");
				}
			}
		}
		return str.toString();
	}
}
// 输出结果：我是测试员我今年18岁,我中午吃的是面条!
```

## 7.源码级框架：

### 7.1 概念

* 运行时框架是在虚拟机运行程序时使用反射技术搭建的框架；而源码级框架是在javac编译源码时,生成框架代码或文件。因为源码级别框架发生过程是在编译期间，所以并不会过多影响到运行效率。因此，搭建框架时候应该优先考虑使用源码级别框架。
* 注解处理器能够在编译源码期间扫描[Java](http://lib.csdn.net/base/javase)代码中的注解，并且根据相关注解动态生成相关的文件。之后在程序运行时就可以使用这些动态生成的代码。值得注意的是，注解处理器运行在跟最终程序不同的虚拟机，也就是说，编译器为注解处理器开启了另外一台虚拟机来运行注解处理器。

7.2 实例：==todo==,从第5个参考开始

# 参考：

1.[反射、注解与依赖注入总结](http://www.open-open.com/lib/view/open1461126839190.html)

2.[深入理解Java：注解](http://www.cnblogs.com/peida/archive/2013/04/24/3036689.html)

3.[Java注解](http://blog.csdn.net/duo2005duo/article/details/50505884)

4.[Android 打造编译时注解解析框架 这只是一个开始](http://blog.csdn.net/lmj623565791/article/details/43452969)

5.[Java注解与自定义注解处理器](http://blog.csdn.net/haveferrair/article/details/52182927)

6.[Java注解处理器](http://www.importnew.com/15246.html)

7.[Java语言使用注解处理器生成代码——第二部分：注解处理器](http://blog.csdn.net/qinxiandiqi/article/details/49182735)

8.[Java中实现自定义的注解处理器（Annotation Processor）](http://blog.csdn.net/ucxiii/article/details/52025005)

9.[【译】从java注解分析ButterKnife工作流程](http://www.jianshu.com/p/89c07ce0c99c)

10.[深入浅出 ButterKnife，听说你还在 findViewById ?](http://www.println.net/post/Deep-in-ButterKnife-3)

