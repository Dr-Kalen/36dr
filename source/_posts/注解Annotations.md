title: 注解Annotations使用和自定义
date: 2015-11-01 22:33:51
tags: 注解
---
## 前言
Android默认提供Annotation框架同时网络也冲刺着各种Android Annotation框架，诸如此类框架很多，但是用过之后使用都比较复杂，同时还有一些无法避免的bug。如在Android开发时经常会遇到获得界面中的View使用方法findViewById特别是在Adapter中需要对所有使用的组件变量赋值，每次调用findViewById就显得多余的开发，若采用注解则会方便开发，精简代码的结构和提示代码阅读性，当然由于Annotation是反射机制设置变量值，则在性能上会相对于普通方式差，在此硬件比拼的时代，此性能几乎可以忽略。

<!--more-->
## 原理和自定义
以前言中提到Android开发多次调用findViewById方法的困惑，我们通过自定义Annotation来解决此问题。如下：
1. 定义注解类：
``` java
@Target(ElementType.FIELD)//表示用在字段上  
@Retention(RetentionPolicy.RUNTIME)//表示在生命周期是运行时  
public @interface ViewInject {  
    int value() default 0;  //返回注解中的信息
}  
```

2. 定义一个BaseActivity作为需要注解的基类，用于实现注解的功能（反射注入数据）
``` java
public abstract class BaseActivity extends FragmentActivity {  
    /**
     * get content view layout id
     *  
     * @return
     */  
    public abstract int getLayoutId();  


    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(getLayoutId());  
        autoInjectAllField();  
    }  
    /**
     * 解析注解
     */  
    public void autoInjectAllField() {  
        try {  
            Class<?> clazz = this.getClass();  
            Field[] fields = clazz.getDeclaredFields();//获得Activity中声明的字段  
            for (Field field : fields) {  
                // 查看这个字段是否有我们自定义的注解类标志的  
                if (field.isAnnotationPresent(ViewInject.class)) {  
                    ViewInject inject = field.getAnnotation(ViewInject.class);  
                    int id = inject.value();  
                    if (id > 0) {  
                        field.setAccessible(true);  
                        field.set(this, this.findViewById(id));//给我们要找的字段设置值  
                    }  
                }  
            }  
        } catch (IllegalAccessException e) {  
            e.printStackTrace();  
        } catch (IllegalArgumentException e) {  
            e.printStackTrace();  
        }  
    }  
}  
```
3. 注解使用

``` java
public class TestActivity extends BaseActivity {  

    @ViewInject(R.id.claim_statement)  
    private WebView mWebView;  


    @Override  
    public int getLayoutId() {  
        // TODO Auto-generated method stub  
        return R.layout.activity_claim;  
    }  

}  
```

## 详细说明

系统中方法经常会使用@Override和@Deprecated注解，我们看看系统注解如何定义：
``` java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```
我们发现这两个注解用到了其他注解@Target,@Retention,@Documented；这些注解称之为元注解。<br>
1. @Target用于标明注解用于的地方，它的值是一个枚举值：
  * CONSTRUCTOR:用于描述构造器
  * FIELD:用于描述域
  * LOCAL_VARIABLE:用于描述局部变量
  * METHOD:用于描述方法
  * PACKAGE:用于描述包
  * PARAMETER:用于描述参数
  * TYPE:用于描述类、接口(包括注解类型) 或enum声明
2. @Retention用于描述注解的生命周期即在什么情况有效，它的值：
  * SOURCE:在源文件中有效（即源文件保留编译时忽略）
  * CLASS:在class文件中有效（即class保留JVM忽略）
  * RUNTIME:在运行时有效（即运行时保留）

3. @Documented表明这个注解应该被javadoc工具记录。

因此当我们定义的注解中：
``` java
@Target(ElementType.FIELD)//表示用在字段上  
@Retention(RetentionPolicy.RUNTIME)//表示在生命周期是运行时  
public @interface ViewInject {  
    int value() default 0;  //返回注解中的信息

    String name() default "default name"
}  
```
就很好解释了，ViewInject注解是用于变量并且运行时有效，用于获得注解中的内容，内容信息为int并且默认为0.<br>

注解类中方法value()表示获取注解中默认文字比如@ViewInject(R.id.name)获取的R.id.name的值，如果是name或则其他方法，则表示需要在注解中使用属性才能获取。及获取@ViewInject(name="name")中的name属性值。<br>
``` text
由此，我们可以根据自己喜好，自己的功能需要，自定义合适项目的注解和注解的解析方式。
Android 注解请学习“Android Annotation”框架。
```
