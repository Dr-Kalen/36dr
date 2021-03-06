title: Gson高级使用和GsonBuilder设置
date: 2015-11-02 19:14:36
tags: Parse
---
## 概述
在开发领域中数据传递有很多形式，通常数据调用交互采用XML，JSON，数据流，纯文本等形式；越来越多数据调用采用JSON，因为JSON数据结构简单，数据字节长度短，既简单又快速何乐而不为呢？<br>

从JSON的结构入手，所有json数据最终分为三种情况：
1. 标量（Scalar)，也就是单纯的字符串或则数字形式
2. 序列（Sequence)，也就是若干数据按照一定顺序并列在一起又称“数组”
3. 映射（Mapping)，也就是key/value键值对

<!-- more -->
Json的规格非常简单,此文章就不一一描述：
``` json
"{"name":"kalen", "age":22}"
```

默认Gson只能序列和反序列基本数据类型和Date类型，其他类型如枚举都需要自定义解析器*registerTypeAdapter*
在Android开发是使用：
``` xml
<dependency>
   <groupId>com.google.code.gson</groupId>
   <artifactId>gson</artifactId>
   <version>2.1</version>
</dependency>
or
compile 'com.google.code.gson:gson:2.3.1'
```
---

Gson 快速使用

1. 普通对象（Mapping)数据解析
``` java
  String json_str = "{"name":"kalen", "age":22}";
  Gson gson = new Gson();
  User user = gson.fromGson(json_str, User.class);
```
通过Gson中的fromGson即可将JSON数据解析并且赋值到User对象中，Gson原理则采用反射机制实现，具体可Google查询Gson原理。

2. 数组数据（Sequence)数据解析
``` java
Type listType = new TypeToken<List<String>>() {}.getType();//数组对应gson中的类型
List<String> target = new LinkedList<String>();//gson需要的转换对象或则数据来源
target.add("blah");
Gson gson = new Gson();
String json = gson.toJson(target, listType);
List<String> target2 = gson.fromJson(json, listType);
```

## Gson高级使用
1.GsonBuilder

Gson是通过GsonBuilder生成，设定Gson的序列化和返序列号数据如：
``` java
Gson gson = new GsonBuilder()     
.registerTypeAdapter(Id.class, new IdTypeAdapter())   
.enableComplexMapKeySerialization()
.serializeNulls()   
.setDateFormat(DateFormat.LONG)  
.setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE)//会把字段首字母大写
.setPrettyPrinting()
.setVersion(1.0)    
.create();
```
以上则是单独对Id类设置了独立解析方式，以及设置时间的解析格式为长时间形式。

2.@Expose注解

如果采用new Gson()方式创建Gson则@Expose则没有任何效果，若采用GsonBuilder创建Gson并且调用了excludeFieldsWithoutExposeAnnotation则@Expose将会影响toJson和fromGson序列化和反序列化数据。如：
``` java
public class User {
    @Expose private String firstName;
    @Expose(serialize = false) private String lastName;
    @Expose(serialize = false, deserialize = false)
    private String emailAddress;
    private String password;
 }
```
例子中password不管是toJson还是fromJson都不会用到，emailAddress和lastName在序列化(fromJson)时将被采用，emailAddress在反序列化（toJson）时将不被采用。<br>

可以通过@SerializedName对序列字段进行重命名，也可以自定义注解然后设置Gson字段解析策略setFieldNamingStrategy,具体在[Retrofit Demo](/2015/10/31/20151031/)中有应用。

3.GsonBuilder方法解释

* setFieldNamingPolicy 设置序列字段的命名策略(UPPER_CAMEL_CASE,UPPER_CAMEL_CASE_WITH_SPACES,LOWER_CASE_WITH_UNDERSCORES,LOWER_CASE_WITH_DASHES)
* addDeserializationExclusionStrategy 设置反序列化时字段采用策略ExclusionStrategy，如反序列化时不要某字段，当然可以采用@Expore代替。
* excludeFieldsWithoutExposeAnnotation 设置没有@Expore则不序列化和反序列化
* addSerializationExclusionStrategy 设置序列化时字段采用策略，如序列化时不要某字段，当然可以采用@Expore代替。
* registerTypeAdapter 为某特定对象设置固定的序列和反序列方式，实现JsonSerializer和JsonDeserializer接口
* setFieldNamingStrategy 设置字段序列和反序列时名称显示，也可以通过@Serializer代替
* setPrettyPrinting 设置gson转换后的字符串为一个比较好看的字符串
* setDateFormat 设置默认Date解析时对应的format格式

---
参考链接：http://www.jianshu.com/p/3108f1e44155
