---
title: Gson使用总结
date: 2016-10-16 12:57:26
categories:  
- 技术总结 
- Gson
tags:  
- 技术总结
- Gson
- 转载
---

## 转载出处：

- [你真的会用Gson吗?Gson使用指南（一）](http://www.jianshu.com/p/e740196225a4)
- [你真的会用Gson吗?Gson使用指南（二）](http://www.jianshu.com/p/c88260adaf5e)
- [你真的会用Gson吗?Gson使用指南（三）](http://www.jianshu.com/p/0e40a52c0063)
- [你真的会用Gson吗?Gson使用指南（四）](http://www.jianshu.com/p/3108f1e44155)

**注：此系列基于Gson 2.4。**

# 你真的会用Gson吗?Gson使用指南（一）

本篇文章的主要内容：

- Gson的基本用法
- 属性重命名 `@SerializedName` 注解的使用
- Gson中使用泛型

## 一、Gson的基本用法

Gson提供了`fromJson()` 和`toJson()` 两个直接用于解析和生成的方法，前者实现反序列化，后者实现了序列化。同时每个方法都提供了重载方法，我常用的总共有5个。

**基本数据类型的解析**

```java
Gson gson = new Gson();
int i = gson.fromJson("100", int.class);              //100
double d = gson.fromJson("\"99.99\"", double.class);  //99.99
boolean b = gson.fromJson("true", boolean.class);     // true
String str = gson.fromJson("String", String.class);   // String
```

注：不知道你是否注意到了第2、3行有什么不一样没

**基本数据类型的生成**

```java
Gson gson = new Gson();
String jsonNumber = gson.toJson(100);       // 100
String jsonBoolean = gson.toJson(false);    // false
String jsonString = gson.toJson("String"); //"String"
```

**POJO类的生成与解析**

```java
public class User {
    //省略其它
    public String name;
    public int age;
    public String emailAddress;
}
```

生成JSON：

```java
Gson gson = new Gson();
User user = new User("怪盗kidou",24);
String jsonObject = gson.toJson(user); // {"name":"怪盗kidou","age":24}
```

解析JSON：

```java
Gson gson = new Gson();
String jsonString = "{\"name\":\"怪盗kidou\",\"age\":24}";
User user = gson.fromJson(jsonString, User.class);
```

## 二、属性重命名 @SerializedName 注解的使用

从上面POJO的生成与解析可以看出json的字段和值是的名称和类型是一一对应的，但也有一定容错机制(如第一个例子第3行将字符串的99.99转成double型，你可别告诉我都是字符串啊)，但有时候也会出现一些不和谐的情况，如：
期望的json格式

```java
{"name":"怪盗kidou","age":24,"emailAddress":"ikidou@example.com"}
```

实际

```java
{"name":"怪盗kidou","age":24,"email_address":"ikidou@example.com"}
```

这对于使用PHP作为后台开发语言时很常见的情况，php和js在命名时一般采用下划线风格，而Java中一般采用的驼峰法，让后台的哥们改吧 前端和后台都不爽，但要自己使用下划线风格时我会感到不适应，怎么办?难到没有两全齐美的方法么?

我们知道Gson在序列化和反序列化时需要使用反射，说到反射就不得不想到注解,一般各类库都将注解放到`annotations`包下，打开源码在`com.google.gson`包下果然有一个`annotations`，里面有一个`SerializedName`的注解类，这应该就是我们要找的。

那么对于json中`email_address`这个属性对应POJO的属性则变成：

```java
@SerializedName("email_address")
public String emailAddress;
```

这样的话，很好的保留了前端、后台、Android/java各自的命名习惯。

你以为这样就完了么?

如果接中设计不严谨或者其它地方可以重用该类，其它字段都一样，就`emailAddress` 字段不一样，比如有下面三种情况那怎么?重新写一个?

```java
{"name":"怪盗kidou","age":24,"emailAddress":"ikidou@example.com"}
```

```java
{"name":"怪盗kidou","age":24,"email_address":"ikidou@example.com"}
```

```java
{"name":"怪盗kidou","age":24,"email":"ikidou@example.com"}
```

**为POJO字段提供备选属性名**
`SerializedName`注解提供了两个属性，上面用到了其中一个，别外还有一个属性`alternate`，接收一个String数组。
注：`alternate`需要2.4版本

```java
@SerializedName(value = "emailAddress", alternate = {"email", "email_address"})
public String emailAddress;
```

当上面的三个属性(email_address、email、emailAddress)都中出现任意一个时均可以得到正确的结果。
注：当多种情况同时出时，以最后一个出现的值为准。

```java
Gson gson = new Gson();
String json = "{\"name\":\"怪盗kidou\",\"age\":24,\"emailAddress\":\"ikidou_1@example.com\",\"email\":\"ikidou_2@example.com\",\"email_address\":\"ikidou_3@example.com\"}";
User user = gson.fromJson(json, User.class);
System.out.println(user.emailAddress); // ikidou_3@example.com
```

## 三、Gson中使用泛型

上面了解的JSON中的Number、boolean、Object和String，现在说一下Array。

例：JSON字符串数组

```java
["Android","Java","PHP"]
```

当我们要通过Gson解析这个json时，一般有两种方式：使用数组，使用List。而List对于增删都是比较方便的，所以实际使用是还是List比较多。

数组比较简单

```java
Gson gson = new Gson();
String jsonArray = "[\"Android\",\"Java\",\"PHP\"]";
String[] strings = gson.fromJson(jsonArray, String[].class);
```

但对于List将上面的代码中的 `String[].class` 直接改为 `List<String>.class` 是行不通的。对于Java来说`List<String>` 和`List<User>` 这俩个的字节码文件只一个那就是`List.class`，这是Java泛型使用时要注意的问题 **泛型擦除**。

为了解决的上面的问题，Gson为我们提供了`TypeToken`来实现对泛型的支持，所以当我们希望使用将以上的数据解析为`List<String>`时需要这样写。

```java
Gson gson = new Gson();
String jsonArray = "[\"Android\",\"Java\",\"PHP\"]";
String[] strings = gson.fromJson(jsonArray, String[].class);
List<String> stringList = gson.fromJson(jsonArray, new TypeToken<List<String>>() {}.getType());
```

注：`TypeToken`的构造方法是`protected`修饰的,所以上面才会写成`new TypeToken<List<String>>() {}.getType()` 而不是 `new TypeToken<List<String>>().getType()`

**泛型解析对接口POJO的设计影响**
泛型的引入可以减少无关的代码，如我现在所在公司接口返回的数据分为两类：

```java
{"code":"0","message":"success","data":{}}
```

```java
{"code":"0","message":"success","data":[]}
```

我们真正需要的`data`所包含的数据，而`code`只使用一次，`message`则几乎不用。如果Gson不支持泛型或不知道Gson支持泛型的同学一定会这么定义POJO。

```java
public class UserResponse {
    public int code;
    public String message;
    public User data;
}
```

当其它接口的时候又重新定义一个`XXResponse`将`data`的类型改成XX，很明显`code`，和`message`被重复定义了多次，通过泛型的话我们可以将`code`和`message`字段抽取到一个`Result`的类中，这样我们只需要编写`data`字段所对应的POJO即可，更专注于我们的业务逻辑。如：

```java
public class Result<T> {
    public int code;
    public String message;
    public T data;
}
```

那么对于`data`字段是`User`时则可以写为 `Result<User>` ,当是个列表的时候为 `Result<List<User>>`，其它同理。

PS：嫌每次 `new TypeToken<Result<XXX>` 和 `new TypeToken<Result<List<XXX>>` 太麻烦, 想进一步封装? 查看我的另一篇博客：** 《搞定Gson泛型封装》**

## 结语

本文主要通过代码向各位读者讲解了Gson的基本用法，以后还会更新更多更高级的用法，如果你还不熟悉 **注解**和**泛型** 那么你要多多努力啦。

如果你有其它的想了解的内容(不限于Gson)请给我留言评论，水平有限，欢迎拍砖。

------

**4月6日补充**
有说看不懂Result那段怎么个简化法，下面给个两个完整的例子，User和List<User> 。

没有引入泛型之前时写法：

```java
public class UserResult {
    public int code;
    public String message;
    public User data;
}
//=========
public class UserListResult {
    public int code;
    public String message;
    public List<User> data;
}
//=========
String json = "{..........}";
Gson gson = new Gson();
UserResult userResult = gson.fromJson(json,UserResult.class);
User user = userResult.data;

UserListResult userListResult = gson.fromJson(json,UserListResult.class);
List<User> users = userListResult.data;
```

上面有两个类`UserResult`和`UserListResult`，有两个字段重复，一两个接口就算了，如果有上百个怎么办?不得累死?所以引入泛型。

```java
//不再重复定义Result类
Type userType = new TypeToken<Result<User>>(){}.getType();
Result<User> userResult = gson.fromJson(json,userType);
User user = userResult.data;

Type userListType = new TypeToken<Result<List<User>>>(){}.getType();
Result<List<User>> userListResult = gson.fromJson(json,userListType);
List<User> users = userListResult.data;
```

看出区别了么?引入了泛型之后虽然要多写一句话用于获取泛型信息，但是返回值类型很直观，也少定义了很多无关类。

# 你真的会用Gson吗?Gson使用指南（二）

**本次的主要内容：**

- Gson的流式反序列化
- Gson的流式序列化
- 使用GsonBuilder导出null值、格式化输出、日期时间及其它小功能

## 一、Gson的流式反序列化

**自动方式**

> Gson提供了`fromJson()`和`toJson()` 两个直接用于解析和生成的方法，前者实现反序列化，后者实现了序列化。同时每个方法都提供了重载方法，我常用的总共有5个。

这是我在上一篇文章开头说的，但我到最后也一直没有是哪5个，这次我给列出来之后，你就知道这次讲的是哪个了。

```java
Gson.toJson(Object);
Gson.fromJson(Reader,Class);
Gson.fromJson(String,Class);
Gson.fromJson(Reader,Type);
Gson.fromJson(String,Type);
```

好了，本节结束！

看第2、4行,Reader懂了吧

**手动方式**
手动的方式就是使用`stream`包下的`JsonReader`类来手动实现反序列化，和Android中使用pull解析XML是比较类似的。

```java
String json = "{\"name\":\"怪盗kidou\",\"age\":\"24\"}";
User user = new User();
JsonReader reader = new JsonReader(new StringReader(json));
reader.beginObject(); // throws IOException
while (reader.hasNext()) {
    String s = reader.nextName();
    switch (s) {
        case "name":
            user.name = reader.nextString();
            break;
        case "age":
            user.age = reader.nextInt(); //自动转换
            break;
        case "email":
            user.email = reader.nextString();
            break;
    }
}
reader.endObject(); // throws IOException
System.out.println(user.name);  // 怪盗kidou
System.out.println(user.age);   // 24
System.out.println(user.email); // ikidou@example.com
```

其实自动方式最终都是通过`JsonReader`来实现的，如果第一个参数是`String`类型，那么Gson会创建一个`StringReader`转换成流操作。

Gson流式解析

## 二、Gson的流式序列化

**自动方式**

Gson.toJson方法列表

所以啊，学会利用IDE的自动完成是多么重要这下知道了吧！
可以看出用红框选中的部分就是我们要找的东西。

提示：`PrintStream`(System.out) 、`StringBuilder`、`StringBuffer`和`*Writer`都实现了`Appendable`接口。

```java
Gson gson = new Gson();
User user = new User("怪盗kidou",24,"ikidou@example.com");
gson.toJson(user,System.out); // 写到控制台
```

**手动方式**

```java
JsonWriter writer = new JsonWriter(new OutputStreamWriter(System.out));
writer.beginObject() // throws IOException
        .name("name").value("怪盗kidou")
        .name("age").value(24)
        .name("email").nullValue() //演示null
        .endObject(); // throws IOException
writer.flush(); // throws IOException
//{"name":"怪盗kidou","age":24,"email":null}
```

提示：除了`beginObject`、`endObject`还有`beginArray`和`endArray`，两者可以相互嵌套，注意配对即可。`beginArray`后不可以调用`name`方法，同样`beginObject`后在调用`value`之前必须要调用`name`方法。

## 三、 使用GsonBuilder导出null值、格式化输出、日期时间

一般情况下`Gson`类提供的 API已经能满足大部分的使用场景，但我们需要更多更特殊、更强大的功能时，这时候就引入一个新的类 `GsonBuilder`。

`GsonBuilder`从名上也能知道是用于构建Gson实例的一个类，要想改变Gson默认的设置必须使用该类配置Gson。

**GsonBuilder用法**

```java
Gson gson = new GsonBuilder()
               //各种配置
               .create(); //生成配置好的Gson
```

Gson在默认情况下是不动导出值`null`的键的，如：

```java
public class User {
    //省略其它
    public String name;
    public int age;
    public String email;
}
```

```java
Gson gson = new Gson();
User user = new User("怪盗kidou",24);
System.out.println(gson.toJson(user)); //{"name":"怪盗kidou","age":24}
```

可以看出，`email`字段是没有在json中出现的，当我们在调试是、需要导出完整的json串时或API接中要求没有值必须用Null时，就会比较有用。

使用方法：

```java
Gson gson = new GsonBuilder()
        .serializeNulls()
        .create();
User user = new User("怪盗kidou", 24);
System.out.println(gson.toJson(user)); //{"name":"怪盗kidou","age":24,"email":null}
```

**格式化输出、日期时间及其它**：

这些都比较简单就不一一分开写了。

```java
Gson gson = new GsonBuilder()
        //序列化null
        .serializeNulls()
        // 设置日期时间格式，另有2个重载方法
        // 在序列化和反序化时均生效
        .setDateFormat("yyyy-MM-dd")
        // 禁此序列化内部类
        .disableInnerClassSerialization()
        //生成不可执行的Json（多了 )]}' 这4个字符）
        .generateNonExecutableJson()
        //禁止转义html标签
        .disableHtmlEscaping()
        //格式化输出
        .setPrettyPrinting()
        .create();
```

注意：内部类(Inner Class)和嵌套类(Nested Class)的区别

这次文章就到这里，欢迎提问互动，如有不对的地方请指正。

# 你真的会用Gson吗?Gson使用指南（三）

**本次的主要内容：**

- 字段过滤的几种方法
  - 基于`@Expose`注解
  - 基于版本
  - 基于访问修饰符
  - 基于策略（作者最常用）
- POJO与JSON的字段映射规则

## 一、字段过滤的几种方法

字段过滤Gson中比较常用的技巧，特别是在Android中，在处理业务逻辑时可能需要在设置的POJO中加入一些字段，但显然在序列化的过程中是不需要的，并且如果序列化还可能带来一个问题就是 **循环引用** ，那么在用Gson序列化之前为不防止这样的事件情发生，你不得不作另外的处理。

以一个商品分类`Category` 为例。

```java
{
  "id": 1,
  "name": "电脑",
  "children": [
    {
      "id": 100,
      "name": "笔记本"
    },
    {
      "id": 101,
      "name": "台式机"
    }
  ]
}
```

一个大分类，可以有很多小分类，那么显然我们在设计`Category`类时`Category`本身既可以是大分类，也可以是小分类。

```java
public class Category {
    public int id;
    public String name;
    public List<Category> children;
}
```

但是为了处理业务，我们还需要在子分类中保存父分类，最终会变成下面的情况

```java
public class Category {
    public int id;
    public String name;
    public List<Category> children;
    //因业务需要增加，但并不需要序列化
    public Category parent; 
}
```

但是上面的`parent`字段是因业务需要增加的，那么在序列化是并不需要，所以在序列化时就必须将其排除，那么在`Gson`中如何排除符合条件的字段呢?下面提供4种方法，大家可根据需要自行选择合适的方式。

##### 基于@Expose注解

**@Expose**提供了两个属性，且都有默认值，开发者可以根据需要设置不同的值。

@Expose

**@Expose** 注解从名字上就可以看出是暴露的意思，所以该注解是用于对处暴露字段的。可是我们以前用Gson的时候也没有**@Expose** 注解还是不正确的序列化为JSON了么?是的，所以该注解在使用`new Gson()` 时是不会发生作用。毕竟最常用的API要最简单，所以该注解必须和`GsonBuilder`配合使用。

**使用方法：** 简单说来就是需要导出的字段上加上**@Expose** 注解，不导出的字段不加。注意是**不导出的不加**。

```java
@Expose //
@Expose(deserialize = true,serialize = true) //序列化和反序列化都都生效
@Expose(deserialize = true,serialize = false) //反序列化时生效
@Expose(deserialize = false,serialize = true) //序列化时生效
@Expose(deserialize = false,serialize = false) // 和不写一样
```

注：根据上面的图片可以得出，所有值为`true`的属性都是可以不写的。

拿上面的例子来说就是

```java
public class Category {
    @Expose public int id;
    @Expose public String name;
    @Expose public List<Category> children;
    //不需要序列化,所以不加 @Expose 注解，
    //等价于 @Expose(deserialize = false,serialize = false)
    public Category parent; 
}
```

在使用Gson时也不能只是简单的`new Gson()`了。

```java
Gson gson = new GsonBuilder()
        .excludeFieldsWithoutExposeAnnotation()
        .create();
gson.toJson(category);
```

##### 基于版本

Gson在对基于版本的字段导出提供了两个注解 `@Since` 和 `@Until`,和`GsonBuilder.setVersion(Double)`配合使用。`@Since` 和 `@Until`都接收一个`Double`值。

Since和Until注解

使用方法：当前版本(GsonBuilder中设置的版本) **大于等于Since**的值时该字段导出，**小于Until**的值时该该字段导出。

```java
class SinceUntilSample {
    @Since(4)
    public String since;
    @Until(5)
    public String until;
}

public void sineUtilTest(double version){
        SinceUntilSample sinceUntilSample = new SinceUntilSample();
        sinceUntilSample.since = "since";
        sinceUntilSample.until = "until";
        Gson gson = new GsonBuilder().setVersion(version).create();
        System.out.println(gson.toJson(sinceUntilSample));
}
//当version <4时，结果：{"until":"until"}
//当version >=4 && version <5时，结果：{"since":"since","until":"until"}
//当version >=5时，结果：{"since":"since"}
```

注：当一个字段被同时注解时，需两者同时满足条件。

##### 基于访问修饰符

什么是修饰符? `public`、`static` 、`final`、`private`、`protected` 这些就是，所以这种方式也是比较特殊的。
使用方式：

```java
class ModifierSample {
    final String finalField = "final";
    static String staticField = "static";
    public String publicField = "public";
    protected String protectedField = "protected";
    String defaultField = "default";
    private String privateField = "private";
}
```

使用`GsonBuilder.excludeFieldsWithModifiers`构建gson,支持`int`形的**可变参数**，值由`java.lang.reflect.Modifier`提供，下面的程序排除了`privateField` 、 `finalField` 和`staticField` 三个字段。

```java
ModifierSample modifierSample = new ModifierSample();
Gson gson = new GsonBuilder()
        .excludeFieldsWithModifiers(Modifier.FINAL, Modifier.STATIC, Modifier.PRIVATE)
        .create();
System.out.println(gson.toJson(modifierSample));
// 结果：{"publicField":"public","protectedField":"protected","defaultField":"default"}
```

到此为止，Gson提供的所有注解就还有一个`@JsonAdapter`没有介绍了，而`@JsonAdapter`将和`TypeAdapter`将作为该系列第4篇也是最后一篇文章的主要内容。

##### 基于策略（自定义规则）

上面介绍的了3种排除字段的方法，说实话我除了@Expose以外，其它的都是只在Demo用上过，用得最多的就是马上要介绍的自定义规则，好处是功能强大、灵活，缺点是相比其它3种方法稍麻烦一点，但也仅仅只是想对其它3种稍麻烦一点而已。

基于策略是利用Gson提供的`ExclusionStrategy`接口，同样需要使用`GsonBuilder`,相关API 2个，分别是`addSerializationExclusionStrategy` 和`addDeserializationExclusionStrategy` 分别针对序列化和反序化时。这里以序列化为例。

例如：

```java
Gson gson = new GsonBuilder()
        .addSerializationExclusionStrategy(new ExclusionStrategy() {
            @Override
            public boolean shouldSkipField(FieldAttributes f) {
                // 这里作判断，决定要不要排除该字段,return true为排除
                if ("finalField".equals(f.getName())) return true; //按字段名排除
                Expose expose = f.getAnnotation(Expose.class); 
                if (expose != null && expose.deserialize() == false) return true; //按注解排除
                return false;
            }
            @Override
            public boolean shouldSkipClass(Class<?> clazz) {
                // 直接排除某个类 ，return true为排除
                return (clazz == int.class || clazz == Integer.class);
            }
        })
        .create();
```

有没有很强大?

## 二、 POJO与JSON的字段映射规则

之前在[你真的会用Gson吗?Gson使用指南（二）](http://www.jianshu.com/p/c88260adaf5e) 属性重命名时 介绍了`@SerializedName`这个注解的使用，本节的内容与上一次差不多的，但既然叫**映射规则**那么说的自然是有规律的情况。
还是之前User的例子，已经去除所有注解：

```java
User user = new User("怪盗kidou", 24);
user.emailAddress = "ikidou@example.com";
```

`GsonBuilder`提供了`FieldNamingStrategy`接口和`setFieldNamingPolicy`和`setFieldNamingStrategy` 两个方法。

**默认实现**
`GsonBuilder.setFieldNamingPolicy` 方法与Gson提供的另一个枚举类`FieldNamingPolicy`配合使用，该枚举类提供了5种实现方式分别为：

| FieldNamingPolicy            | 结果（仅输出emailAddress字段）                  |
| ---------------------------- | -------------------------------------- |
| IDENTITY                     | {"emailAddress":"ikidou@example.com"}  |
| LOWER_CASE_WITH_DASHES       | {"email-address":"ikidou@example.com"} |
| LOWER_CASE_WITH_UNDERSCORES  | {"email_address":"ikidou@example.com"} |
| UPPER_CAMEL_CASE             | {"EmailAddress":"ikidou@example.com"}  |
| UPPER_CAMEL_CASE_WITH_SPACES | {"Email Address":"ikidou@example.com"} |

**自定义实现**
`GsonBuilder.setFieldNamingStrategy` 方法需要与Gson提供的`FieldNamingStrategy`接口配合使用，用于实现将POJO的字段与JSON的字段相对应。上面的`FieldNamingPolicy`实际上也实现了`FieldNamingStrategy`接口，也就是说`FieldNamingPolicy`也可以使用`setFieldNamingStrategy`方法。

用法：

```java
Gson gson = new GsonBuilder()
        .setFieldNamingStrategy(new FieldNamingStrategy() {
            @Override
            public String translateName(Field f) {
                //实现自己的规则
                return null;
            }
        })
        .create();
```

**注意：** `@SerializedName`注解拥有最高优先级，在加有`@SerializedName`注解的字段上`FieldNamingStrategy`不生效！

# 你真的会用Gson吗?Gson使用指南（四）

本次文章的主要内容：

- TypeAdapter
- JsonSerializer与JsonDeserializer
- TypeAdapterFactory
- @JsonAdapter注解
- TypeAdapter与 JsonSerializer、JsonDeserializer对比
- TypeAdapter实例
- 结语
- 后期预告

## 一、TypeAdapter

`TypeAdapter` 是Gson自2.0（源码注释上说的是2.1）开始版本提供的一个抽象类，用于**接管某种类型的序列化和反序列化过程**，包含两个注要方法 `write(JsonWriter,T)` 和 `read(JsonReader)` 其它的方法都是`final`方法并最终调用这两个抽象方法。

```java
public abstract class TypeAdapter<T> {
    public abstract void write(JsonWriter out, T value) throws IOException;
    public abstract T read(JsonReader in) throws IOException;
    //其它final 方法就不贴出来了，包括`toJson`、`toJsonTree`、`toJson`和`nullSafe`方法。
}
```

**注意：**TypeAdapter 以及 JsonSerializer 和 JsonDeserializer 都需要与 `GsonBuilder.registerTypeAdapter` 示或`GsonBuilder.registerTypeHierarchyAdapter`配合使用，下面将不再重复说明。

使用示例：

```java
User user = new User("怪盗kidou", 24);
user.emailAddress = "ikidou@example.com";
Gson gson = new GsonBuilder()
        //为User注册TypeAdapter
        .registerTypeAdapter(User.class, new UserTypeAdapter())
        .create();
System.out.println(gson.toJson(user));
```

UserTypeAdapter的定义：

```java
public class UserTypeAdapter extends TypeAdapter<User> {

    @Override
    public void write(JsonWriter out, User value) throws IOException {
        out.beginObject();
        out.name("name").value(value.name);
        out.name("age").value(value.age);
        out.name("email").value(value.email);
        out.endObject();
    }

    @Override
    public User read(JsonReader in) throws IOException {
        User user = new User();
        in.beginObject();
        while (in.hasNext()) {
            switch (in.nextName()) {
                case "name":
                    user.name = in.nextString();
                    break;
                case "age":
                    user.age = in.nextInt();
                    break;
                case "email":
                case "email_address":
                case "emailAddress":
                    user.email = in.nextString();
                    break;
            }
        }
        in.endObject();
        return user;
    }
}
```

当我们为`User.class` 注册了 `TypeAdapter`之后，只要是操作`User.class` 那些之前介绍的`@SerializedName` 、`FieldNamingStrategy`、`Since`、`Until`、`Expos`通通都黯然失色，失去了效果，只会调用我们实现的`UserTypeAdapter.write(JsonWriter, User)` 方法，我想怎么写就怎么写。

再说一个场景，在该系列的第一篇文章就说到了Gson有一定的容错机制，比如将字符串 `"24"` 转成int 的`24`,但如果有些情况下给你返了个空字符串怎么办（有人给我评论问到这个问题）?虽然这是服务器端的问题，但这里我们只是做一个示范。

int型会出错是吧，根据我们上面介绍的，我注册一个TypeAdapter 把 序列化和反序列化的过程接管不就行了?

```java
Gson gson = new GsonBuilder()
        .registerTypeAdapter(Integer.class, new TypeAdapter<Integer>() {
            @Override
            public void write(JsonWriter out, Integer value) throws IOException {
                out.value(String.valueOf(value)); 
            }
            @Override
            public Integer read(JsonReader in) throws IOException {
                try {
                    return Integer.parseInt(in.nextString());
                } catch (NumberFormatException e) {
                    return -1;
                }
            }
        })
        .create();
System.out.println(gson.toJson(100)); // 结果："100"
System.out.println(gson.fromJson("\"\"",Integer.class)); // 结果：-1
```

注：测试空串的时候一定是`"\"\""`而不是`""`，`""`代表的是没有json串，`"\"\""`才代表json里的`""`。

你说这一接管就要管两样好麻烦呀，我明明只想管序列化（或反列化）的过程的，另一个过程我并不关心，难道没有其它更简单的方法么? 当然有！就是接下来要介绍的 **JsonSerializer与JsonDeserializer**。

## 二、JsonSerializer与JsonDeserializer

`JsonSerializer` 和`JsonDeserializer` 不用像`TypeAdapter`一样，必须要实现序列化和反序列化的过程，你可以据需要选择，如只接管序列化的过程就用 `JsonSerializer` ，只接管反序列化的过程就用 `JsonDeserializer` ，如上面的需求可以用下面的代码。

```java
Gson gson = new GsonBuilder()
        .registerTypeAdapter(Integer.class, new JsonDeserializer<Integer>() {
            @Override
            public Integer deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
                try {
                    return json.getAsInt();
                } catch (NumberFormatException e) {
                    return -1;
                }
            }
        })
        .create();
System.out.println(gson.toJson(100)); //结果：100
System.out.println(gson.fromJson("\"\"", Integer.class)); //结果-1
```

下面是所有数字都转成序列化为字符串的例子

```java
JsonSerializer<Number> numberJsonSerializer = new JsonSerializer<Number>() {
    @Override
    public JsonElement serialize(Number src, Type typeOfSrc, JsonSerializationContext context) {
        return new JsonPrimitive(String.valueOf(src));
    }
};
Gson gson = new GsonBuilder()
        .registerTypeAdapter(Integer.class, numberJsonSerializer)
        .registerTypeAdapter(Long.class, numberJsonSerializer)
        .registerTypeAdapter(Float.class, numberJsonSerializer)
        .registerTypeAdapter(Double.class, numberJsonSerializer)
        .create();
System.out.println(gson.toJson(100.0f));//结果："100.0"
```

注：`registerTypeAdapter`必须使用包装类型，所以`int.class`,`long.class`,`float.class`和`double.class`是行不通的。同时不能使用父类来替上面的子类型，这也是为什么要分别注册而不直接使用`Number.class`的原因。

上面特别说明了`registerTypeAdapter`不行，那就是有其它方法可行咯?当然！换成**registerTypeHierarchyAdapter **就可以使用`Number.class`而不用一个一个的当独注册啦！

**registerTypeAdapter与registerTypeHierarchyAdapter的区别：**

|      | registerTypeAdapter | registerTypeHierarchyAdapter |
| ---- | ------------------- | ---------------------------- |
| 支持泛型 | 是                   | 否                            |
| 支持继承 | 否                   | 是                            |

注：如果一个被序列化的对象本身就带有泛型，且注册了相应的`TypeAdapter`，那么必须调用`Gson.toJson(Object,Type)`，明确告诉Gson对象的类型。

```java
Type type = new TypeToken<List<User>>() {}.getType();
TypeAdapter typeAdapter = new TypeAdapter<List<User>>() {
   //略
};
Gson gson = new GsonBuilder()
        .registerTypeAdapter(type, typeAdapter)
        .create();
List<User> list = new ArrayList<>();
list.add(new User("a",11));
list.add(new User("b",22));
//注意，多了个type参数
String result = gson.toJson(list, type);
```

## 三、TypeAdapterFactory

TypeAdapterFactory,见名知意，用于创建TypeAdapter的工厂类，通过对比`Type`，确定有没有对应的`TypeAdapter`，没有就返回null，与`GsonBuilder.registerTypeAdapterFactory`配合使用。

```java
Gson gson = new GsonBuilder()
    .registerTypeAdapterFactory(new TypeAdapterFactory() {
        @Override
        public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type) {
            return null;
        }
    })
    .create();
```

## 四、@JsonAdapter注解

`JsonAdapter`相较之前介绍的`SerializedName`、`FieldNamingStrategy`、`Since`、`Until`、`Expos`这几个注解都是比较特殊的，其它的几个都是用在POJO的字段上，而这一个是用在POJO类上的，接收一个参数，且必须是`TypeAdpater`，`JsonSerializer`或`JsonDeserializer`这三个其中之一。

上面说`JsonSerializer`和`JsonDeserializer`都要配合`GsonBuilder.registerTypeAdapter`使用，但每次使用都要注册也太麻烦了，`JsonAdapter`就是为了解决这个痛点的。

使用方法（以User为例）：

```java
@JsonAdapter(UserTypeAdapter.class) //加在类上
public class User {
    public User() {
    }
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public User(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
    public String name;
    public int age;
    @SerializedName(value = "emailAddress")
    public String email;
}
```

使用时不用再使用 `GsonBuilder`去注册`UserTypeAdapter`了。
**注：**`@JsonAdapter` 仅支持 `TypeAdapter`或`TypeAdapterFactory`

```java
Gson gson = new Gson();
User user = new User("怪盗kidou", 24, "ikidou@example.com");
System.out.println(gson.toJson(user));
//结果：{"name":"怪盗kidou","age":24,"email":"ikidou@example.com"}
//为区别结果，特意把email字段与@SerializedName注解中设置的不一样
```

**注意：**`JsonAdapter`的优先级比`GsonBuilder.registerTypeAdapter`的优先级更高。

## 五、TypeAdapter与 JsonSerializer、JsonDeserializer对比

|            | TypeAdapter    | JsonSerializer、JsonDeserializer |
| ---------- | -------------- | ------------------------------- |
| 引入版本       | 2.0            | 1.x                             |
| Stream API | 支持             | 不支持*，需要提前生成`JsonElement`        |
| 内存占用       | 小              | 比`TypeAdapter`大                 |
| 效率         | 高              | 比`TypeAdapter`低                 |
| 作用范围       | 序列化 **和** 反序列化 | 序列化 **或** 反序列化                  |

## 六、TypeAdapter实例

注：这里的TypeAdapter泛指`TypeAdapter`、`JsonSerializer`和`JsonDeserializer`。
这里的TypeAdapter 上面讲了一个自动将 字符串形式的数值转换成int型时可能出现 空字符串的问题，下面介绍一个其它读者的需求：

> 服务器返回的数据中data字段类型不固定，比如请求成功data是一个List,不成功的时候是String类型，这样前端在使用泛型解析的时候，怎么去处理呢？

其实这个问题的原因主要由服务器端造成的，接口设计时没有没有保证数据的一致性，正确的数据返回姿势：**同一个接口任何情况下不得改变返回类型，要么就不要返，要么就返空值，如null、[],{}**。

但这里还是给出解决方案：
方案一：

```java
Gson gson = new GsonBuilder().registerTypeHierarchyAdapter(List.class, new JsonDeserializer<List<?>>() {
    @Override
    public List<?> deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        if (json.isJsonArray()){
            //这里要自己负责解析了
            Gson newGson = new Gson();
            return newGson.fromJson(json,typeOfT);
        }else {
            //和接口类型不符，返回空List
            return Collections.EMPTY_LIST;
        }
    }
}).create();
```

方案二：

```java
 Gson gson = new GsonBuilder().registerTypeHierarchyAdapter(List.class, new JsonDeserializer<List<?>>() {
    @Override
    public List<?> deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        if (json.isJsonArray()) {
            JsonArray array = json.getAsJsonArray();
            Type itemType = ((ParameterizedType) typeOfT).getActualTypeArguments()[0];
            List list = new ArrayList<>();
            for (int i = 0; i < array.size(); i++) {
                JsonElement element = array.get(i);
                Object item = context.deserialize(element, itemType);
                list.add(item);
            }
            return list;
        } else {
            //和接口类型不符，返回空List
            return Collections.EMPTY_LIST;
        }
    }
}).create();
```

要注意的点：

- 必须使用`registerTypeHierarchyAdapter`方法，不然对List的子类无效，但如果POJO中都是使用List，那么可以使用`registerTypeAdapter`。
- 对于是数组的情况，需要创建一个新的Gson，不可以直接使用context,不然gson又会调我们自定义的`JsonDeserializer`造成递归调用，方案二没有重新创建Gson，那么就需要提取出List<E>中E的类型，然后分别反序列化适合为E手动注册了TypeAdaper的情况。
- 从效率上推荐方案二，免去重新实例化Gson和注册其它TypeAdapter的过程。

## 结语

Gson系列总算是完成了，感觉写得越来越差了，我怕我写得太啰嗦，也不能总是大片大片的贴代码，所以可能有的地方写得并不详细，排版也不美观，但都些都不重点，重点是Gson里我们能用上的都一一介绍一遍，只要你确确实实把我这几篇文章上的内容都学会的话，以后Gson上的任何问题都不再是问题，当然可能很多内容对于实际的开发中用的并不多，但下次有什么疑难杂症就难不倒你了。

本系列不提供Demo源码，最重要的是自己实验。

写一篇文章还是要花不少时间和精力，要写示例、调式、组织语言、码字等等，加上关注的人在慢慢的增加的同时既给了我动力也给我不少压力，如有纰漏或者更好的例子都可以和我交流。