# ListOfReading

---------------------------------------------------------------------------------------------------------

简单记录一下工作之余学习和阅读的一些优秀文章

---------------------------------------------------------------------------------------------------------

# Github

 [learnGitBranching](https://learngitbranching.js.org/)

# Java

## Annotation

[秒懂，Java 注解 （Annotation）你可以这样学](https://blog.csdn.net/briblue/article/details/73824058)

## 反射

[细说反射](https://blog.csdn.net/briblue/article/details/74616922)

## 泛型

* 与普通的 Object 代替一切类型这样简单粗暴而言，泛型使得数据的类别可以像参数一样由外部传递进来。它提供了一种扩展能力。它更符合面向抽象开发的软件编程宗旨。

* 当具体的类型确定后，泛型又提供了一种__类型检测__的机制，只有相匹配的数据才能正常的赋值，否则编译器就不通过。所以说，它是一种类型安全检测机制，一定程度上提高了软件的安全性防止出现低级的失误。

* 泛型提高了程序代码的可读性，不必要等到运行的时候才去强制转换，在定义或者实例化阶段，程序员能够一目了然猜测出代码要操作的数据类型。

* 泛型信息只存在于__代码编译阶段__，在进入 JVM 之前，与泛型相关的信息会被擦除掉，专业术语叫做类型擦除。而类型擦除，是泛型能够与之前的 java 版本代码兼容共存的原因。

[java 泛型详解-绝对是对泛型方法讲解最详细的，没有之一](https://blog.csdn.net/s10461/article/details/53941091)

[Java 泛型，你了解类型擦除吗？](https://blog.csdn.net/briblue/article/details/76736356)

## 代理

* 代理模式可以在不修改被代理对象的基础上，通过扩展代理类，进行一些功能的附加与增强。值得注意的是，代理类和被代理类应该共同实现一个接口，或者是共同继承某个类。

* 在不修改被代理对象的源码上，进行功能的增强。

* 代理分为静态代理和动态代理两种。
  静态代理，代理类需要自己编写代码。
  动态代理，代理类通过 Proxy.newInstance() 方法生成。
  不管是静态代理还是动态代理，代理与被代理者都要实现相同接口，它们的实质是面向接口编程。
  静态代理和动态代理的区别是在于要不要开发者自己定义 Proxy 类。
  动态代理通过 Proxy 动态生成 proxy class，但是它也指定了一个 InvocationHandler 的实现类。

[轻松学，Java 中的代理模式及动态代理](https://blog.csdn.net/briblue/article/details/73928350)

-------------------------------------------------------------------------------------------------

# Android

[Android入门基础](http://hukai.me/android-training-course-in-chinese/basics/index.html)

[RecyclerView优秀文集](https://github.com/CymChad/CymChad.github.io)

[Android开发：ListView、AdapterView、RecyclerView全面解析](https://blog.csdn.net/carson_ho/article/details/51472640)

[Android Activity为什么要细化出onCreate、onStart、onResume、onPause、onStop、onDesdroy这么多方法让应用去重载？](https://blog.csdn.net/zhao_3546/article/details/12843477)

[Android ViewPager使用详解](https://blog.csdn.net/wangjinyu501/article/details/8169924)

## Android Fragment

[Android Fragment 真正的完全解析（上）](https://blog.csdn.net/lmj623565791/article/details/37970961)

[Android Fragment 真正的完全解析（下）](https://blog.csdn.net/lmj623565791/article/details/37992017)

[Android Fragment 你应该知道的一切](https://blog.csdn.net/lmj623565791/article/details/42628537)

[Android Fragment完全解析，关于碎片你所需知道的一切](https://blog.csdn.net/guolin_blog/article/details/8881711)

[Android Fragment应用实战，使用碎片向ActivityGroup说再见](https://blog.csdn.net/guolin_blog/article/details/13171191)


## Android 屏幕适配

[Android官方提供的支持不同屏幕大小的全部方法](https://blog.csdn.net/guolin_blog/article/details/8830286)

[Android 屏幕适配：最全面的解决方案](https://www.jianshu.com/p/ec5a1a30694b)

[Android 目前稳定高效的UI适配方案](https://mp.weixin.qq.com/s/X-aL2vb4uEhqnLzU5wjc4Q)

[骚年你的屏幕适配方式该升级了!-今日头条适配方案](https://juejin.im/post/5b7a29736fb9a019d53e7ee2)

[骚年你的屏幕适配方式该升级了!-SmallestWidth 限定符适配方案](https://juejin.im/post/5ba197e46fb9a05d0b142c62)

[今日头条屏幕适配方案终极版正式发布!](https://juejin.im/post/5bce688e6fb9a05cf715d1c2)

[AndroidAutoSize](https://github.com/JessYanCoding/AndroidAutoSize)


-----------------------------------------------------------------------------------------------------

# AndroidStudio

[AndroidStudio自定义类创建时自动生成的头部注释](https://blog.csdn.net/panhouye/article/details/77528918)

[build.gradle详解](https://blog.csdn.net/hebbely/article/details/79074460)



------------------------------------------------------------------------------------------------------

# OkHttp

[OkHttp使用教程](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0106/2275.html)





------------------------------------------------------------------------------------------------------

# Dagger2

* Dagger2是一款基于Java注解来实现的完全在__编译阶段__完成依赖注入的开源库，主要用于模块间解耦、提高代码的健壮性和可维护性。Dagger2在编译阶段通过apt利用Java注解自动生成Java代码，然后结合手写的代码来自动帮我们完成依赖注入的工作。

* Inject，Component，Module，Provides是dagger2中的最基础最核心的知识点。奠定了dagger2的整个依赖注入框架。
  Inject主要是用来标注目标类的依赖和依赖的构造函数
  Component它是一个桥梁，一端是目标类，另一端是目标类所依赖类的实例，它也是注入器（Injector）负责把目标类所依赖类的实例注入到目标类中，同时它也管理Module。
  Module和Provides是为解决第三方类库而生的，Module是一个简单工厂模式，Module可以包含创建类实例的方法，这些方法用Provides来标注


步骤1：查找Module中是否存在创建该类的方法。
步骤2：若存在创建类方法，查看该方法是否存在参数
    步骤2.1：若存在参数，则按从**步骤1**开始依次初始化每个参数
    步骤2.2：若不存在参数，则直接初始化该类实例，一次依赖注入到此结束
步骤3：若不存在创建类方法，则查找Inject注解的构造函数，
           看构造函数是否存在参数
    步骤3.1：若存在参数，则从**步骤1**开始依次初始化每个参数
    步骤3.2：若不存在参数，则直接初始化该类实例，一次依赖注入到此结束


[Android：dagger2让你爱不释手-基础依赖注入框架篇](https://www.jianshu.com/p/cd2c1c9f68d4)

[Android：dagger2让你爱不释手-重点概念讲解、融合篇](https://www.jianshu.com/p/1d42d2e6f4a5)

[Android：dagger2让你爱不释手-终结篇](https://www.jianshu.com/p/65737ac39c44)

[Demo Dagger2](https://github.com/xianfeng92/Demo_Dagger2)

--------------------------------------------------------------------------------

# Butterknife

[Butterknife 使用指南](https://www.jianshu.com/p/9c37856ddf09)

[Android Butterknife（黄油刀） 使用方法总结](https://blog.csdn.net/donkor_/article/details/77879630)

------------------------------------------------------------------

# Retrofit

[A type-safe HTTP client for Android and Java](http://square.github.io/retrofit/)

[Retrofit2.0使用详解](https://blog.csdn.net/ljd2038/article/details/51046512)

[JSON字符串转Java实体类](http://www.bejson.com/json2javapojo/new/)


------------------------------------------------------------------------
# Picasso

[图片加载框架－Picasso最详细的使用指南](https://www.jianshu.com/p/c68a3b9ca07a)



------------------------------------------------------------------------------

# RXJava

* 在RxJava 中，Scheduler ——调度器，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。
* RxJava 提供了对事件序列进行变换的支持，这是它的核心功能之一。所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。

### Season_zlc的“水管”系列

[给初学者的RxJava2.0教程(一)](http://www.jianshu.com/p/464fa025229e)

[给初学者的RxJava2.0教程(二)](http://www.jianshu.com/p/8818b98c44e2)

[给初学者的RxJava2.0教程(三)](http://www.jianshu.com/p/128e662906af)

[给初学者的RxJava2.0教程(四)](http://www.jianshu.com/p/bb58571cdb64)

[给初学者的RxJava2.0教程(五)](http://www.jianshu.com/p/0f2d6c2387c9)

[给初学者的RxJava2.0教程(六)](http://www.jianshu.com/p/e4c6d7989356)

[给初学者的RxJava2.0教程(七)](http://www.jianshu.com/p/9b1304435564)

[给初学者的RxJava2.0教程(八)](http://www.jianshu.com/p/a75ecf461e02)

[给初学者的RxJava2.0教程(九)](http://www.jianshu.com/p/36e0f7f43a51)


### nanchen2251的RxJava2系列

[这可能是最好的RxJava 2.x 教程（完结版）](https://www.jianshu.com/p/0cd258eecf60)

----------------------------------------------------------------------------

# Timber
[Android更好的打印方式Timber使用简单记录](https://blog.csdn.net/lin_dianwei/article/details/78646989)


-------------------------------------------------------------------------------

# Flutter

[我花了 8 小时，"掌握"了一下 Flutter | Flutter 中文站上线](https://www.jianshu.com/p/9aaabc60d8af)

----------------------------------------------------------------------------------

# API
[干货集中营 API 文档](https://gank.io/api)
