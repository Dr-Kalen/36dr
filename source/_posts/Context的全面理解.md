title: Context的全面理解
date: 2016-05-20 23:34:51
subtitle: Context的全面理解
keywords:  Context,Context的全面理解
description: Context的全面理解
author: Kalen
tags:
layout: timeline
---
### 1. 什么是Context
在Android中Context分为ActivityContext和Application Context，Context是一个抽象类是所有上下文的基类。

### 2. Activity Context 和Application Context 的区别
* application context生命周期比较长，伴随应用程序的存在而存在与activity的生命周期无关，Activity Context 则只能存活于Activity 生命周期内
* 两者的使用范围和用于有部分差异，具体可以查看图示

![](http://img.blog.csdn.net/20150104183450879)
数字1：启动Activity在这些类中是可以的，但是需要创建一个新的task。一般情况不推荐.
数字2：在这些类中去layout inflate是合法的，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。
数字3：在receiver为null时允许，在4.2或以上的版本中，用于获取黏性广播的当前值。（可以无视）
注：ContentProvider、BroadcastReceiver之所以在上述表格中，是因为在其内部方法中都有一个context用于使用。

<!--more-->
### 3. Context可以做什么
通过Context可以访问App全局信息的接口，例如：

- 启动Activity、Service、发送广播
- 访问APP中的资源和公开的方法
- 获取assets中的资源
- 对APK进行管线管理

由以上来看，Context可以算是对APK无所不知，可以操作资源、代码以及其他信息。

在创建Activity、Service、Application时都会自动创建Context，因此各自维护着自己的上下午，如果要统计一个app中的
``` text
Context数量=Activity数+Service数+application
```

如何使用Context做这些操作呢，我们举例说明：

* 执行APK中某类的方法
``` Android
Context c = createPackageContext("chroya.demo", Context.CONTEXT_INCLUDE_CODE | Context.CONTEXT_IGNORE_SECURITY);
//载入这个类
Class clazz = c.getClassLoader().loadClass("chroya.demo.Main");
//新建一个实例
Object owner = clazz.newInstance();
//获取print方法，传入参数并执行
Object obj = clazz.getMethod("print", String.class).invoke(owner, "Hello”)；
```
ok，这样，我们就调用了chroya.demo包的Main类的print方法，执行结果。其中需要对createPackageContext的参数说明
1、packageName  包名，要得到Context的包名
2、flags  标志位，有CONTEXT_INCLUDE_CODE和CONTEXT_IGNORE_SECURITY两个选项。CONTEXT_INCLUDE_CODE的意思是包括代码，也就是说可以执行这个包里面的代码。
CONTEXT_IGNORE_SECURITY的意思是忽略安全警告，如果不加这个标志的话，有些功能是用不了的，会出现安全警告。<br>

* 访问apk中的资源

在ContextImpl中存在一个mResources变量，变量值的来源如下代码，它就是我们在通过Context.getResource()方法返回的结果，而          resource中存在一个ArrayMap用于存储所有的资源对象，因为一个app针对不同分辨率、不同方向等拥有多套资源，因此ArrayMap中会是多个对象，又ResourceManager使用的单例模式，因此每个Activity使用的是同一套资源但不一定是同一个资源对象。
``` Android
 mResources = mResourcesManager.getTopLevelResources(mPackageInfo.getResDir(),Display.DEFAULT_DISPLAY, null, compatInfo,activityToken);
 ```
 ### 4. Context造成内存泄露
通常造成Context内存泄露的原因是因为，系统gc执行时无法销毁Context，我们举例说明：<br>

我们都知道应用当屏幕旋转时会销毁当前Activity然后重新创建一个新的Activity，当我们activity中有图片加载时而我们又希望保持这个图片不重新加载，我们可能采用一个静态变量来保持这个Drawable，此时当屏幕旋转的时候由于Drawable与View有关联，Drawable保存了view的引用，而view保存了Activity引用，因此导致在旋转屏幕是无法将activity销毁，而造成内存泄露，程序崩溃。<br>

<font color="#FF1493">防止Context导致内存泄露，我们应该注意保存Activity中的对象与Activity是同一个生命周期，对已需要非常长的周期对象可以采用Application Context。</font><br>
### 5. Unable to add window — token null is not for an application
此错误一般是在弹出框时出现异常如：
``` Android
　Dialog dialog = new Dialog(getApplicationContext());
```
　　或者
``` Android
   Dialog dialog = new Dialog(getApplication());
```
由于dialog是一个窗口，因此需要一个拥有窗口令牌的Token，而在packageInfo存在的情况下getApplication与getApplication返回的是同一个application context，application context并没有窗口令牌，因此会出现这种异常，正确的做法：
``` Android
   AlertDialog.Builder builder = new AlertDialog.Builder(this);
```

[1] 具体创建时代码：

创建Application时：创建Application的时机在创建handleBindApplication()方法
//创建Application时同时创建的ContextIml实例
``` Android
 2 private final void handleBindApplication(AppBindData data){
 3     …
 4     ///创建Application对象
 5     Application app = data.info.makeApplication(data.restrictedBackupMode, null);
 6     …
 7 }
 8 public Application makeApplication(boolean forceDefaultAppClass, Instrumentation instrumentation) {
 9     …
10     try {
11         java.lang.ClassLoader cl = getClassLoader();
12         ContextImpl appContext = new ContextImpl();    //创建一个ContextImpl对象实例
13         appContext.init(this, null, mActivityThread);  //初始化该ContextIml实例的相关属性
14         ///新建一个Application对象
15         app = mActivityThread.mInstrumentation.newApplication(
16                 cl, appClass, appContext);
17        appContext.setOuterContext(app);  //将该Application实例传递给该ContextImpl实例
18     }
19     …
20 }
```
创建Activity时通过startActivity()或startActivityForResult()请求启动一个Activity时，如果系统检测需要新建一个Activity对象时回调handleLaunchActivity()方法。
//创建一个Activity实例时同时创建ContextIml实例
``` Android
private final void handleLaunchActivity(ActivityRecord r, Intent customIntent) {
    …
    Activity a = performLaunchActivity(r, customIntent);  //启动一个Activity
}
private final Activity performLaunchActivity(ActivityRecord r, Intent customIntent) {
    …
    Activity activity = null;
    try {
        //创建一个Activity对象实例
        java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
        activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
    }
    if (activity != null) {
        ContextImpl appContext = new ContextImpl();      //创建一个Activity实例
        appContext.init(r.packageInfo, r.token, this);   //初始化该ContextIml实例的相关属性
        appContext.setOuterContext(activity);            //将该Activity信息传递给该ContextImpl实例
        …
    }
    …
}
```
创建Service时通过startService或者bindService时，如果系统检测到需要新创建一个Service实例，就会回调handleCreateService()方法
 //创建一个Service实例时同时创建ContextIml实例
 ``` Android
private final void handleCreateService(CreateServiceData data){
    …
    //创建一个Service实例
    Service service = null;
    try {
        java.lang.ClassLoader cl = packageInfo.getClassLoader();
        service = (Service) cl.loadClass(data.info.name).newInstance();
    } catch (Exception e) {
    }
    …
    ContextImpl context = new ContextImpl(); //创建一个ContextImpl对象实例
    context.init(packageInfo, null, this);   //初始化该ContextIml实例的相关属性
    //获得我们之前创建的Application对象信息
    Application app = packageInfo.makeApplication(false, mInstrumentation);
    //将该Service信息传递给该ContextImpl实例
    context.setOuterContext(service);
    …
}
```
