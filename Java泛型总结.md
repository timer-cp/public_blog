---
title: Java泛型总结
date: 2017-02-28 10:51:45
categories:  
- 技术总结 
- Java泛型 
tags:  
- Java 
- 泛型
---

## 1.什么是java泛型?

* 泛型是Java SE1.5的新特性，泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。
* Java语言引入泛型的好处是安全简单。可以将运行时错误提前到编译时错误。

## 2.泛型接口、泛型类

* 对于常见的泛型模式，推荐的名称是：

```java
K ——键，比如映射的键。
V ——值，比如 List 和 Set 的内容，或者 Map 中的值。
E ——异常类。
T ——泛型。
```

* 泛型不是协变的：List<Object> 不是 List<String> 的父类型

```java
ist<Integer> intList = newArrayList<Integer>();
List<Number> numberList = intList; //invalid
```

* 泛型接口示例：

```java
public interface Generator<T> {
    public T next();
}
```

* 泛型类示例：

```java
public class Container {
    private String key;
    private String value;

    public Container(String k, String v) {
        key = k;
        value = v;
    }
    
    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}
```

* **注意：**
  * 泛型方法必须在访问修饰符后面声明泛型

    `public <T> void out(T t)`

  * 泛型类必须在类名后面声明泛型

    `public class ClassTest<T>` 

## 3.泛型方法

* 泛型方法使得该方法能独立于类而产生变化。以下是一个基本的指导原则：无论何时，只要你能做到，你就应该尽量使用泛型方法。也就是说，如果使用泛型方法可以取代将整个类泛型化，那么就应该只使用泛型方法，因为它可以使事情更清楚明白。
* **对于一个static的方法而言，无法访问泛型类的类型参数**。所以，如果static方法需要使用泛型能力，就必须使其成为泛型方法
* 要定义泛型方法，只需**将泛型参数列表置于返回值之前**：

```java
public class Main {
    public static <T> void out(T t) {
        System.out.println(t);
    }
    public static void main(String[] args) {
        out("findingsea");
        out(123);
        out(true);
    }
}
// 可变参数例子
public class Main {
    public static <T> void out(T... args) {
        for (T t : args) {
            System.out.println(t);
        }
    }
    public static void main(String[] args) {
        out("findingsea", 123, 11.11, true);
    }
}
```

## 4.泛型的擦除：

### 4.1 代码实例：

```java
Class c1 = new ArrayList<String>().getClass();
Class c2 = new ArrayList<Integer>().getClass();
System.out.println(c1 == c2); //true
```

* 在泛型内部，无法获得任何有关泛型参数类型的信息。
* **所有反射的操作都是在运行时的，既然为true，就证明了编译之后，程序会采取去泛型化的措施，也就是说Java中的泛型，只在编译阶段有效。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦出，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，成功编译过后的class文件中是不包含任何泛型信息的。泛型信息不会进入到运行时阶段。**

```java
ArrayList<String> lists = new ArrayList<String>();  
lists.add("测试");  
Class c = lists.getClass();  
try{  
    Method method = c.getMethod("add",Object.class);  
    method.invoke(lists,100);  
    System.out.println(lists);  
}catch(Exception e){  
    e.printStackTrace();  
} 
// 输出结果：[测试, 100]
```

* 通过反射绕过了泛型的编译期检查。

### 4.2 擦除的补偿

```java
class Parent{}
class Child extends Parent{}
class Sunzi extends Child{};

public class ClassTest<T> {
	Class<T> kind;
	public ClassTest(Class<T> kind) {
		this.kind = kind;
	}
	public boolean test1(T t){
		return kind.isInstance(t);
	}
	public boolean test2(Object obj){
		return kind.isInstance(obj);
	}
	public static void main(String[] args) {
		ClassTest<Child> ct1 = new ClassTest<>(Child.class);
		System.out.println(ct1.test1(new Sunzi())); // true
		System.out.println(ct1.test1(new Child())); // true
//		System.out.println(ct1.test1(new Parent())); // 编译错误：参数不匹配
		System.out.println(ct1.test2(new Sunzi())); // true
		System.out.println(ct1.test2(new Child())); // true
		System.out.println(ct1.test2(new Parent())); // false
	}
}
```

## 5.泛型数组

* 不能创建一个确切的泛型类型的数组。下面使用[Sun](http://docs.oracle.com/javase/tutorial/extra/generics/fineprint.html)[的一篇文档](http://docs.oracle.com/javase/tutorial/extra/generics/fineprint.html)的一个例子来说明这个问题:

```java
List<String>[] lsa = new List<String>[10]; // Not really allowed.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Unsound, but passes run time store check    
String s = lsa[1].get(0); // Run-time error: ClassCastException. 
```

* 这种情况下，由于JVM泛型的擦除机制，在运行时JVM是不知道泛型信息的，所以可以给oa[1]赋上一个ArrayList<Integer>而不会出现异常，但是在取出数据的时候却要做一次类型转换，所以就会出现ClassCastException，如果可以进行泛型数组的声明，上面说的这种情况在编译期将不会出现任何的警告和错误，只有在运行时才会出错。

```java
List<?>[] lsa = new List<?>[10]; // OK, array of unbounded wildcard type.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Correct.    
Integer i = (Integer) lsa[1].get(0); // OK  
```

## 6.泛型通配符`?`

* 可以解决当具体类型不确定的时候，这个通配符就是 **?**  ；当操作类型时，不需要使用类型的具体功能时，只使用Object类中的功能。那么可以用 ? 通配符来表未知类型。

```java
Class<?>classType = Class.forName("java.lang.String");
```

* 如下代码：

```java
List<Integer> intList= new ArrayList<Integer>();    
List<Number> numList = ex_int; //非法的 
```

* 上述第2行会出现编译错误，因为Integer虽然是Number的子类，但List<Integer>不是List<Number>的子类。
* 假定第2行代码没有问题，那么我们可以使用语句numList.add(newDouble())在一个List中装入了各种不同类型的子类，这显然是不可以的，因为我们在取出List中的对象时，就分不清楚到底该转型为Integer还是Double了。因此，我们需要一个在逻辑上可以用来同时表示为List<Integer>和List<Number>的父类的一个引用类型，类型通配符应运而生。在本例中表示为List<?>即可。

## 7.泛型限定(上限和下限)

### 7.1 上限：

* <? extends E>：可以接收E类型或者E的子类型对象。

```java
class Parent{}
class Child extends Parent{}
class Sunzi extends Child{};

public class ClassTest {
	public static void main(String[] args) {
//		mTest(new ArrayList<Parent>()); // 编译检查不通过
		mTest(new ArrayList<Child>());
		mTest(new ArrayList<Sunzi>());
	}
  	public static void mTest(ArrayList<? extends Child> temp){
		System.out.println(temp);
	}
}
```

* 使用<? extends E>可以从List里面get元素，但不可以add元素：

```java
ArrayList<? extends Parent> al = new ArrayList<>();
al.add(new Child()); // 编译错误
al.add(new Sunzi()); // 编译错误
al.add(new Parent()); // 编译错误
al.add(null); // 编译正确，但没意思
```

* 不能往<? extends E>里添加元素主要还是因为<? extends E>类型不确定的原因，而我们要往里面添加一个确定的类型，所以会编译时出错。所以对于实现了<? extends E>的集合类只能将它视为Producer向外提供(get)元素，而不能作为Consumer来对外获取(add)元素。
* 使用时机：当从集合中获取元素进行操作的时候，可以用当前元素的类型接收，也可以用当前元素的父类型接收。

### 7.2 下限：

* <? super E>：可以接收E类型或者E的父类型对象。

```java
class Parent{}
class Child extends Parent{}
class Sunzi extends Child{};

public class ClassTest {
	public static void main(String[] args) {
		mTest(new ArrayList<Parent>());
		mTest(new ArrayList<Child>());
//		mTest(new ArrayList<Sunzi>()); // 编译检查不通过
	}
  	public static void mTest(ArrayList<? super Child> temp){
		System.out.println(temp);
	}
}
```

* 使用<? super E>可以往List里add数据，但不能get数据：

```java
ArrayList<? super Child> al = new ArrayList<>();
Child child = al.get(0); // 编译器提示需要强制转换，但这就可爱出现强制转换异常。
```

* 原因和上面一样，还是因为在编译时无法确定<? super Child>的具体类型，如果将get的对象赋给一个具体类型可能会出现类型转换错误。
* 使用时机：往集合中添加元素时，既可以添加E类型对象，又可以添加E的子类型对象。为什么? 因为取的时候，E类型既可以接收E类对象，又可以接收E的子类型对象。

### 7.3 总结：

* 根据上面的例子，我们可以总结出一条规律，”Producer Extends, Consumer Super”：
  * “Producer Extends” - 如果你需要一个只读List，用它来produce T，那么使用`? extends T`。
  * “Consumer Super” - 如果你需要一个只写List，用它来consume T，那么使用`? super T`。
  * 如果需要同时读取以及写入，那么我们就不能使用通配符了。
* 如果阅读过一些Java集合类的源码，可以发现通常我们会将两者结合起来一起用，比如像下面这样：

```java
public class Collections {
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        for (int i=0; i<src.size(); i++)
            dest.set(i, src.get(i));
    }
}
```

## 8.泛型使用时机：

* 当接口、类及方法中的操作的引用数据类型不确定的时候，以前用的Object来进行扩展的，现在可以用泛型来表示。这样可以避免强转的麻烦，而且将运行问题转移到的编译时期。

## 9.注意点：

* 泛型到底代表什么类型取决于调用者传入的类型，如果没传，默认是Object类型
* 泛型的类型参数只能是类类型（包括自定义类），不能是简单类型。
* 使用带泛型的类创建对象时，等式两边指定的泛型必须一致
  * 原因：编译器检查对象调用方法时只看变量，然而程序运行期间调用方法时就要考虑对象具体类型了；
* 等式两边可以在任意一边使用泛型，在另一边不使用(考虑向后兼容)



# 参考：

1.[Java泛型详解](http://www.ziwenxie.site/2017/03/01/java-generic/)
