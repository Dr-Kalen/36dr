title: Retrofit2.0基本使用和个人封装
date: 2016-01-19 21:37:26
tags: 网络篇
---

## 前言

对比目前用的算是顺手的网络框架Volley，Okhttp，Retrofit,通常优先选择Retrofit，若Retrofit无法满足开发，则选择Okhttp。前两种已经完全可以实现大部分的网络开发需要，而且开发中使用极其简约大方。
我们详细讨论Retrofit的优点和缺点：<br>
Retrofit由Square组织编写的网络框架，使用技术动态代理和反射，使用OO设计简约网络代码。<br>
<!--more-->
## Retrofit装备
* 框架下载地址：https://github.com/square/retrofit
* 官网使用说明文档：http://square.github.io/retrofit/#api-declaration
* AndroidStudio 依赖导入
``` text
compile 'com.squareup.retrofit:retrofit:2.0.0-beta2'
or
compile 'com.squareup.retrofit:retrofit:1.9.0'
```

## Retrofit1.9使用
[请阅读Retrofit1.9](/2015/10/31/20151031/)

## Retrofit2.0 使用
Retrofit使用中最重要的几个类：Retrofit, Interface，Call<br>

Retrofit使用步骤：<br>
1. 定义接口，参数声明，URL通过Annotation指定
2. 设置Retrofit相关的参数（baseUrl，数据解析）
3. 通过Retrofit.create生成一个接口实现类（动态代理类）
4. 调用接口请求数据

```
注： 2.0强制要求接口方法返回参数不能为void，同时接口方法中没有Callback参数
```

接口定义方式<br>
```java
@GET("/group/{id}/profile")  
List<User> userlist(@Path("id") int id);
```
Retrofit支持@GET、@POST、@HEAD、@PUT、@DELETA、@PATCH，和参数相关的@Path、@Field、@Multipart，表示网络请求使用的请求方式。另外如何自定义Annotation[请阅读自定义Annotation](/2015/11/01/20151101/)。<br>

生成RestAdapter动态代理类：<br>
``` java
GsonBuilder builder = new GsonBuilder();
builder.setFieldNamingStrategy(new AnnotateNaming());
Gson gson = builder.create();

Retrofit retrofit = new Retrofit.Builder()
                   .baseUrl(API_URL)
                   .addConverterFactory(GsonConverterFactory.create(gson))
                   .build();


github = retrofit.create(Interfaces.class);
```
通过Retrofit将Interfaces自定义的的网络请求接口生成动态代理类，同时设置请求的参数：
* baseUrl 设置网络请求服务器url前缀，地址必须以“/”结束
* 设置addConverterFactory，数据返回之后的解析器，默认采用GsonConverter解析，2.0中提供了六种解析方式：Gson，Jackson,Moshi,Protobuf,Wire,SimpleXML,Scalars
``` Android
Gson: com.squareup.retrofit2:converter-gson
Jackson: com.squareup.retrofit2:converter-jackson
Moshi: com.squareup.retrofit2:converter-moshi
Protobuf: com.squareup.retrofit2:converter-protobuf
Wire: com.squareup.retrofit2:converter-wire
Simple XML: com.squareup.retrofit2:converter-simplexml
Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars
```
*
Convert自定义如：
``` java
/*定义Date解析方式*/
Gson gson = new GsonBuilder()
.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES)
.registerTypeAdapter(Date.class, new DateTypeAdapter())
.create();
```

数据返回处理：<br>
Retrofit2.0中默认在主线程返回数据，则可以直接对数据进行展示和更新
如：
``` java
/* 同步处理*/
@GET("member/register/mobileExists")
Call<BaseObjectType> mobileExists(@Query("mobile") String mobile);

```

### 高级处理
1. 添加网络拦截器interceptor
一个典型应用场景是所有http请求需要加上api key,在Retrofit2之前，可以通过RequestInterceptor实现
``` Android
final RequestInterceptor authorizationInterceptor = new RequestInterceptor() {
        @Override
        public void intercept(RequestInterceptor.RequestFacade request) {
            request.addQueryParam(MovieDbApi.PARAM_API_KEY, "YOUR_API_KEY");
        }
    };
```
然后设置
``` Android
RestAdapter restAdapter = new RestAdapter.Builder()
                .setEndpoint(Interfaces.END_POINT)
                .setRequestInterceptor(authorizationInterceptor)
                .build();
movieDbApi = restAdapter.create(Interfaces.class);
```

2. 添加访问内容日志logging
在开发中打印网络请求和返回结果是非常重要的，如果你在Retrofit中启用了这个功能，你会发现实现该功能的方法已经不可用了。
那是因为你现在必须依靠okhttp提供的日志系统，HttpLoggingInterceptor.
首先声明一个新的依赖，因为它不是okhttp的一部分所以要另外添加依赖：
``` Android
compile 'com.squareup.okhttp3:logging-interceptor:3.1.2'
```
然后，创建一个interceptor实例：
``` Android
HttpLoggingInterceptor httpLoggingInterceptor = new HttpLoggingInterceptor();
httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
OkHttpClient okHttpClient = new OkHttpClient.Builder().addInterceptor(httpLoggingInterceptor).build();
```

3. Gson转换器
具体已经在上文解释，唯一需要说明使用前需要加入依赖包
``` Android
compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta4'
```
4.
## 总结
Retrofit默认没有指定解析方式，需要首先指定，如：使用GsonConverter。

总体来说Retrofit看起来很好用，不过要求服务端返回数据最好要规范，不然如果请求成功返回一种数据结构，请求失败返回另一种数据结构，不好用Converter解析，接口的定义也不好定义。


### Retrofit 2.0 个人封装
通常开发中客户端与服务器会定义一种通用的协议：*head+body* 形式，head包含数据的状态，提示文字等等。body的形式分为：字段，对象，数组。若是字段则创建一个对象定义head和body，若是对象则出现json中套有对象，如果强制按照Retrofit方式开发，则需要创建两个对象来获得数据,导致项目中出现量的不必要的对象。开发中出现只是字段的毕竟是少数情况，那么以此类推所有数据访问都需要创建两个对象，假若是
数组更加显得开发困难，因此个人针对Retrofit进行了访问数据返回的特别处理，采用泛型定义我们需要的对象实体。<br>

衍生三个实体类：<br>
* BaseType 针对body为字段
* BaseObjectType 针对body为object形式
* BaseSequenceType 针对body为数组形式。

BaseType定义
``` java
public  class BaseType {

    public String message;

    public boolean state;
}
```
用于获取基本的数据结构，若除这些字段还有其他，则可以自己集成BaseType扩展。<br>

BaseObjectType定义

``` java
public class BaseObjectType<T> extends BaseType {

    public T data;

    public  T getObject(){
        return data;
    }

}
```
用于获取body为对象情况，当数据解析之后，通过getObject方式获得我们需要的实体对象。<br>

BaseSequenceType定义
``` java
public class BaseSequenceType <T>  extends BaseType{


    private List<T> data;

    public List<T> getList(){
        return data;
    }

    public int getSize(){
        return data == null ? 0 : data.size();
    }


}
```
用于获取body为数组的情况，当数据返回之后通过getList方式获得实体数组。<br>

具体通过Retrofit和泛型如何访问数据，如下：
``` java

Call<BaseObjectType> call =  NetworkRequest.getInstance().mobileExists("18708140959");
try {
    BaseObjectType response  = call.execute().body();
    Toast.makeText(MainActivity.this, response.message, Toast.LENGTH_SHORT).show();
} catch (IOException e) {
    e.printStackTrace();
}
```
封装之后的Demo下载地址：https://github.com/Dr-Kalen/RetrofitDemo2.0
