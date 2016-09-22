title: Retrofit1.9基本使用和个人封装
date: 2015-10-31 11:37:26
tags: 网络篇
---
## 前言

对比目前用的算是顺手的网络框架Volley，Okhttp，Retrofit,通常优先选择Retrofit，若Retrofit无法满足开发，则选择Okhttp。前两种已经完全可以实现大部分的网络开发需要，而且开发中使用极其简约大方。
我们详细讨论Retrofit的优点和缺点：<br>
Retrofit由Square组织编写的网络框架，使用技术动态代理和反射，使用OO设计简约网络代码。目前最新版本是2.0版本，鉴于某些仍在使用1.x版本，因此我们对两个版本依依分析和使用。<br>
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

## Retrofit2.0使用
[请阅读Retrofit2.0](/2016/01/19/Retrofit2.0基本使用[20160119]/)

## Retrofit1.9 使用
Retrofit使用中最重要的几个类：RestAdapter, Interface，Convert,RequestInterceptor,CallbackRunnable。<br>

Retrofit使用步骤：<br>
1. 定义接口，参数声明，URL通过Annotation指定
2. 设置RestAdapter相关的参数（endpoint，线程池，数据解析，特定网络访问组件，请求拦截器，请求前后AOP处理等）
3. 通过RestAdapter生成一个接口实现类（动态代理类）
4. 调用接口请求数据

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

RestAdapter restAdapter = new RestAdapter.Builder()
.setProfiler(new DRProfiler())
.setRequestInterceptor(interceptor)
.setEndpoint(API_URL)
.setConverter(new GsonConverter(gson))
.build();

restAdapter.setLogLevel(RestAdapter.LogLevel.FULL);
github = restAdapter.create(Interfaces.class);
```
通过RestAdapter将GitHubService自定义的的网络请求接口生成动态代理类，同时设置请求的参数：
* setEndPoint 设置网络请求服务器url，endpoint即时域名，若不设置其他则默认采用系统配置。
* 设置converter，数据返回之后的解析器，默认采用GsonConverter解析，可以自己扩展XMLConvertert用于解析XML数据格式。
* 设置Client，对于Android默认判断采用OKhttp，若无则在2.3以后使用HttpURLConnection，2.3以前则使用HttpClient。
* 设置请求拦截器RequestInterceptor， 当我们需要同一处理所有请求中的头部信息等可用拦截方式。
* 设置执行线程池， 默认使用后台线程Executor。
* 设置数据返回线程池，默认单独提供主线程Executor线程，即返回数据处理可以直接更新UI数据。
* 设置Profiler, 用于拦截数据请求之前和请求结束之后的回调，采用AOP代理设计。

Convert自定义如：
``` java
/*定义Date解析方式*/
Gson gson = new GsonBuilder()
.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES)
.registerTypeAdapter(Date.class, new DateTypeAdapter())
.create();

/*定义XML数据转换*/
RestAdapter restAdapter = new RestAdapter.Builder()
.setEndpoint("https://api.soundcloud.com")
.setConverter(new SimpleXMLConverter())
.build();

```

自定义Client

若不采用自带的Client，默认先考虑Okhttp，若没有配置则判断是否大于2.3版本，若大于则用HTTPclient，否在使用HttpUrlConnection.
``` Android
builder.setClient(new OkClient(OkHttpUtils.getInstance(context)));
```
自定义一个OkHttpClient然后设置相关属性（timeout时间，缓存，等）然后通过builder.setClient设置网络访问的client

数据返回处理：<br>
* 若是同步调用则直接返回数据，返回数据与请求属于同一线程，则不能直接更新UI因为不在UI主线程中。
* 若非同步调用通过RequestInterceptorTape拦截，然后判断是Observe(RXJava)观察模式，则采用RxJava方式回调,默认不特别处理是Callback。若不了解RxJava请阅读[RxJava原理与使用](http://www.baidu.com)
* 若是Callback则交由回调线程处理，采用UI主线程返回数据。
如：
``` java
/* 同步处理*/
@GET("/users/list")
Response userList();
/*异步Callback回调*/
@GET("/users/list")
void userList(Callback<Response> cb);
/*异步RxJava回调*/
@GET("/users/list")
Observable<Response> userList();
```
## 总结

Retrofit对输入和输出做了封装，通过TypedOutput向服务器发送数据，通过TypedInput读取服务器返回的数据。

通过MultipartTypedOutput支持文件上传，读取服务器数据时，如果要求直接返回未解析的Response，Restonse会被转换为TypedByteArray，所以不能是大文件类的

Retrofit支持不同的Log等级，当为LogLevel.Full时会把Request及Response的Body打印出来，所以如果包含文件就不行了。

Retrofit默认使用GsonConverter，所以要想获取原始数据不要Retrofit解析，要么自定义Conveter，要么直接返回Response了，返回Response也比较麻烦

总体来说Retrofit看起来很好用，不过要求服务端返回数据最好要规范，不然如果请求成功返回一种数据结构，请求失败返回另一种数据结构，不好用Converter解析，接口的定义也不好定义，除非都返回Response，或自定义Converter所有接口都返回String


### Retrofit 1.9 个人封装
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
NetworkRequest.getInstance().list(new Callback<BaseSequenceType<Product>>() {


            @Override
            public void success(BaseSequenceType<Product> productBaseSequenceType, Response response) {
                System.out.println(productBaseSequenceType.state + " " + productBaseSequenceType.message);
                List<Product> products = productBaseSequenceType.getList();

                for (Product product : products) {
                    System.out.println(product);
                }
            }

            @Override
            public void failure(RetrofitError error) {

            }
        });
```
封装之后的Demo下载地址：https://github.com/Dr-Kalen/RetrofitDemo
## 资源链接
1. [Retrofit 部分源码剖析](http://www.cnblogs.com/angeldevil/p/3757335.html)
