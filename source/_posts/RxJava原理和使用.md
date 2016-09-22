title: RXJava的个人理解
tags: RxJava 框架
date: 2016-06-23 18:03:00
---
### 前言
有一段时间大家都疯传```RxJava```很好用，工作一直很忙没有时间坐下来研究，最近公司准备搭建一个项目模板架构，于是花了2天时间学习```RxJava```。

在当下我们正在使用的有AsyncTask | Handler | Thread | Service ...作为异步处理方式，使用Android原生事件处理 | Dagger | ButterKnife | RoboGuide 等作为事件处理工具，使用Volley | Retrofit | OKHttp | 等一系列来实现网络加载，如此多的框架，各有优点和缺点，在我们使用有没有产生一种选择恐惧症呢？有些人说自己习惯就好，但是你习惯的你团队成员习惯吗？有没有想过入股有一个框架能将这三件事完美融合那该多好呀，同时又非常好用能够让大家都接收岂不更好。

<!-- more -->
现在``` RxJava ```有办法做到这一切，使用``` RxJava + Retrofit + Rx一些列扩展(RxBinding...) ```，他们都支持使用RxJava的方式，可以很完美的对接上，我很喜欢。

由于我下面提到的参考文献已经对``` RxJava + Retrofit```的使用以及RxJava本身讲得很清楚了，我也就不用再累赘，大家可以直接链接阅读，我仅仅做一个搬运工。

在不就的将来我会针对RxJava + Gson + Retrofit + MVP + Buffterknife + Material Design 封装一个项目基本Demo，用于一些懒人开发者直接拿来就可以进行项目开发，特别适合于一些外包公司和一些初学者接触开发项目不久。

#### 参考文献
1. [RxJava 与 Retrofit 结合的最佳实践](http://gank.io/post/56e80c2c677659311bed9841)
2. [是时候学习RxJava了](http://www.jianshu.com/p/8cf84f719188)
3. [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
4. [Retrofit2.0的详解使用](http://blog.36dr.net/2016/01/19/Retrofit2.0%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/)
5. [Gson高级使用和GsonBuilder设置](http://blog.36dr.net/2015/11/02/Gson%E9%AB%98%E7%BA%A7%E4%BD%BF%E7%94%A8/)
6. [注解Annotations使用和自定义](http://blog.36dr.net/2015/11/01/%E6%B3%A8%E8%A7%A3Annotations)
7. [Rxjava实战](http://www.jianshu.com/p/64aa976a46be?utm_campaign=haruki&utm_content=note&utm_medium=reader_share&utm_source=qq)
