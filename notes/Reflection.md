
# Reflection

## 动态语言

__一般认为在程序运行时，允许改变程序结构或变量类型，这种语言称为动态语言__。从这个观点看，Perl，Python，Ruby 是动态语言，C++，Java，C# 不是动态语言。
尽管这样，JAVA有着一个非常突出的动态相关机制：反射（Reflection）。运用反射我们可以于运行时加载、探知、使用编译期间完全未知的classes。
换句话说，Java程序可以加载在运行时才得知名称的class，获悉其完整构造方法，并生成其对象实体、或对其属性设值、或唤起其成员方法。

静态编译：在编译时确定类型，绑定对象,即通过。
 
动态编译：运行时确定类型，绑定对象。动态编译最大限度发挥了java的灵活性，体现了多态的应用，降低类之间的藕合性。

## 反射

当一个类被加载以后，Java虚拟机就会自动产生一个Class对象。通过这个Class对象我们就能获得加载到虚拟机当中这个Class对象对应的方法、成员以及构造方法的声明和定义等信息。
反射机制的优点就是可以实现动态创建对象和编译，体现出很大的灵活性。它的缺点是对性能有影响。使用反射基本上是一种解释操作，我们可以告诉JVM，
我们希望做什么并且它满足我们的要求。这类操作总是慢于只直接执行相同的操作。

了解这些，那我们就知道了，我们可以利用反射机制在Java程序中，动态的去调用一些protected甚至是private的方法或类，这样可以很大程度上满足我们的一些比较特殊需求。

例如 Activity 的启动过程中 Activity 的对象的创建。`

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {

        。。。。。。
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } 
        。。。。。。
```

上面代码可知Activity在创建对象的时候调用了mInstrumentation.newActivity()。

以下代码位于Instrumentation中：

```
    public Activity newActivity(ClassLoader cl, String className,
            Intent intent)
            throws InstantiationException, IllegalAccessException,
            ClassNotFoundException {
            //这里的 className 就是在 manifest 中注册的 Activity name.
        return (Activity)cl.loadClass(className).newInstance();
    }
```
最终在newActivity()里返回的是利用cl.loadClass返回的Activity对象。可知，Activity对象的创建是通过反射完成的。java程序可以动态加载类定义，
而这个动态加载的机制就是通过ClassLoader来实现的，所以可想而知ClassLoader的重要性如何。 

上面说到JAVA的动态加载的机制就是通过 ClassLoader 来实现的，ClassLoader 也是实现反射的基石。ClassLoader 是JAVA提供的一个类，
顾名思义，它就是用来加载Class文件到JVM，以供程序使用的。

但是问题来了，ClassLoader加载文件到JVM，但是Android是基于DVM的，用ClassLoader加载文件进DVM肯定是不行的。于是Android提供了另外一套加载机制，
分别为 dalvik.system.DexClassLoader 和 dalvik.system.PathClassLoader，区别在于 PathClassLoader 不能直接从 zip 包中得到 dex，
因此只支持直接操作 dex 文件或者已经安装过的 apk（因为安装过的 apk 在 cache 中存在缓存的 dex 文件）。
而 DexClassLoader 可以加载外部的 apk、jar 或 dex文件，并且会在指定的 outpath 路径存放其 dex 文件。

# Reflection基本操作

## Class 的获取

反射的入口是 Class，但是反射中 Class 是没有公开的构造方法的，所以就没有办法像创建一个类一样通过 new 关键字来获取一个 Class 对象。
不过，不用担心，Java 反射中 Class 的获取可以通过下面 3 种方式:

```
public class People {
    Tool tool = new Tool();

    Class clazz1 = tool.getClass();

    Class clazz2 = Tool.class;

    Class clazz3;

    {
        try {
            //就是 Tool 这个类的全限定名称，它包括包名+类名。
            clazz3 = Class.forName("com.example.demo_reflection.Tool");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

}

```

1. 通过 Object.getClass()

对于一个对象而言，如果这个对象可以访问，那么调用 getClass()方法就可以获取到了它的相应的 Class 对象。

2. 通过 .class 标识

如果不想创建Tool这个类的实例的话，就需要通过 Tool.class 这个标识。


3. 通过 Class.forName() 方法

有时候，我们没有办法创建 Tool 的实例，甚至没有办法用 Tool.class 这样的方式去获取 Tool 的 Class 对象。

这在 Android 开发领域很常见，因为某种目的，Android 工程师把一些类加上了 @hide 注解，所示这些类就没有出现在 SDK 当中，那么，我们要获取这个并不存在于当前开发环境中的类的 Class 对象时就没有辙了吗？
答案是否定的，Java 给我们提供了 Class.forName() 这个方法。__只要给这个方法中传入一个类的全限定名称就好了，那么它就会到 Java 虚拟机中去寻找这个类有没有被加载__。如果找不到时，
它会抛出 ClassNotFoundException 这个异常，这个很好理解，因为如果查找的类没有在 JVM 中加载的话，自然要告诉开发者。


## Class 内容清单

在正常的代码编写中，我们如果要编写一个类，一般会定义它的属性和方法，如：

```
public class Tool {

    private static final String TAG = "Tool";

    private String color = "blue";
    private int size = 10;

    public int getSize() {
        return size;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public void setSize(int size) {
        this.size = size;
    }
    
    public void action(){
        Log.d(TAG, "action: do something");
    }

    static class inner{
        
    }

}


```

Class 对象也有名字，涉及到的 API 有:

```
Class.getName();

Class.getSimpleName();

Class.getCanonicalName();

```
当 Class 代表一个引用时
getName() 方法返回的是一个二进制形式的字符串，比如"com.example.demo_reflection.Tool"。

当 Class 代表一个基本数据类型，比如 int.class 的时候
getName() 方法返回的是它们的关键字，比如 int.class 的名字是 int。

当 Class 代表的是基础数据类型的数组时 比如 int[][][] 这样的 3 维数组时,getName() 返回 [[[I 这样的字符串。

getName() 返回 [[[I 这样的字符串。

为什么会这样呢？这是因为，Java 本身对于这一块制定了相应规则，在元素的类型前面添加相应数量的 [ 符号，用 [ 的个数来提示数组的维度，并且值得注意的是，对于基本类型或者是类，都有相应的编码，所谓的编码大多数是用一个大写字母来指示某种类型。需要注意的是__类或者是接口的类型编码是 L类名;的形式,后面有一个分号__。

```
   private void test(){
        try {
            Class clazz = Class.forName("com.example.demo_reflection.Tool");
            // com.example.demo_reflection.Tool
            String name1 = clazz.getName();
            // Tool
            String name2 = clazz.getSimpleName();
            //com.example.demo_reflection.Tool
            String name3 = clazz.getCanonicalName();
            Log.d(TAG, "test: "+name1 +" "+name2+ " " + name3);
        } catch (ClassNotFoundException e) {
            Log.d(TAG, "test: error");
        }

        Tool[] tools = new Tool[2];
        String name4 = tools.getClass().getName();
        // [Lcom.example.demo_reflection.Tool;
        Log.d(TAG, "test: "+name4);

        Class clazz1 = Tool.inner.class;
        // com.example.demo_reflection.Tool$inner   inner
        Log.d(TAG, "test: "+clazz1.getName()+" "+clazz1.getSimpleName());
    }

```

对于内部类，所以通过 getName() 方法获取到的是二进制形式的全限定类名，并且类名前面还有个 $ 符号，getSimpleName() 则直接返回了 Inner，去掉了包名限定。

Canonical 是官方、标准的意思，那么 getCanonicalName() 自然就是返回一个 Class 对象的官方名字，这个官方名字 canonicalName 是 Java 语言规范制定的。
如果 Class 对象没有 canonicalName 的话就返回 null。


## Class 获取修饰符

通常，Java 开发中定义一个类，往往是要通过许多修饰符来配合使用的。它们大致分为 4 类。

* 用来限制作用域，如 public、protected、priviate

* 用来提示子类复写，abstract

* 用来标记为静态类 static

* 注解

Java 反射提供了 API 去获取这些修饰符。

```
  private void testModifier(){
        try {
            Class clazz = Class.forName("com.example.demo_reflection.Tool");
            //testModifier: public
            Log.d(TAG, "testModifier: "+ Modifier.toString(Tool.class.getModifiers()));
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

```
调用 Modifier.toString() 方法就可以打印出一个类的所有修饰符。


## 获取 Class 的成员


一个类的成员包括属性、方法。对应到 Class 中就是 Field、Method、Constructor。

### 获取 Filed

获取指定名字的属性有 2 个 API

```
public Field getDeclaredField(String name)
                       throws NoSuchFieldException,
                              SecurityException;

public Field getField(String name)
               throws NoSuchFieldException,
                      SecurityException
```

两者的区别就是 getDeclaredField() 获取的是 Class 中被 private 修饰的属性。 getField() 方法获取的是非私有属性，并且 getField() 在当前 Class 获取不到时会向祖先类获取。

```
//获取所有的属性，但不包括从父类继承下来的属性
public Field[] getDeclaredFields() throws SecurityException

//获取自身的所有的 public 属性，包括从父类继承下来的。
public Field[] getFields() throws SecurityException 

```

__所以反射是无法从一个子类去获取父类私有Field__。


### 获取 Method

类或者接口中的方法对应到 Class 就是 Method。

```
public Method getDeclaredMethod(String name, Class<?>... parameterTypes)

public Method getMethod(String name, Class<?>... parameterTypes)

public Method[] getDeclaredMethods() throws SecurityException

public Method getMethod(String name, Class<?>... parameterTypes) 

```

```
    private void testMethod(){
        try {
            Class clazz = Class.forName("com.example.demo_reflection.Tool");
            Method method = clazz.getMethod("action");
            Log.d(TAG, "testMethod: "+method);
            // 通过反射调用tool的action方法
            method.invoke(clazz.newInstance());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }

```

## 获取 Constructor

Java 反射把构造器从方法中单独拎出来了，用 Constructor 表示。

```
public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)

public Constructor<T> getConstructor(Class<?>... parameterTypes)

public Constructor<?>[] getDeclaredConstructors() throws SecurityException 

public Constructor<?>[] getConstructors() throws SecurityException 

```

我们获取到了 Field、Method、Constructor,这正是反射机制中开始的地方，我们运用反射的目的就是为了获取和操控 Class 对象中的这些成员。

## Field 的操控

### Field 类型的获取

```
public Type getGenericType() {}

public Class<?> getType() {}

```
注意，两者返回的类型不一样，getGenericType() 方法能够获取到泛型类型。

### Field 修饰符的获取

同 Class 一样，Field 也有很多修饰符。通过 getModifiers() 方法就可以轻松获取。

```
public int getModifiers() {}

```

### Field 内容的读取与赋值

这个应该是反射机制中对于 Field 最主要的目的了,Field 这个类定义了一系列的 get 和set 方法来获取和设置不同类型的值。

```
    private void testField(){
        try {
            Class clazz = Class.forName("com.example.demo_reflection.Tool");
            Tool tool = (Tool) clazz.newInstance();
            Field[] fields = clazz.getFields();
            // 查看 Tool 中所有属性
            for (Field field : fields){
                Log.d(TAG, "testField: "+field.getName()+" "+field.getType());
            }
            // 根据属性名，找到对应属性，此处为size属性
            Field field = clazz.getDeclaredField("size");
            // 如果该属性为 private，需要设置 setAccessible
            field.setAccessible(true);
            // 获取属性值
            int a = field.getInt(tool);
            Log.d(TAG, "testField: "+a);
            // 修改该属性的值
            field.set(tool,1111);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }
    }
```

### Method 的操控

Method 获取方法参数

```
public Parameter[] getParameters() {}

```
返回的是一个 Parameter 数组，在反射中 Parameter 对象就是用来映射方法中的参数。经常使用的方法有：

```
// 获取参数名字
public String getName() {}

// 获取参数类型
public Class<?> getType() {}

// 获取参数的修饰符
public int getModifiers() {}

```

当然，有时候我们不需要参数的名字，只要参数的类型就好了，通过 Method 中下面的方法获取。

```
// 获取所有的参数类型
public Class<?>[] getParameterTypes() {}

// 获取所有的参数类型，包括泛型
public Type[] getGenericParameterTypes() {}

```

Method 获取返回值类型

```
// 获取返回值类型
public Class<?> getReturnType() {}


// 获取返回值类型包括泛型
public Type getGenericReturnType() {}

```

Method 方法的执行，这个应该是整个反射机制的核心内容了，很多时候运用反射目的其实就是为了以非常规手段执行 Method。

```
public Object invoke(Object obj, Object... args) {}
```

Method 调用 invoke() 的时候，存在许多细节：

* invoke() 方法中第一个参数 Object 实质上是 Method 所依附的 Class 对应的类的实例，如果这个方法是一个静态方法，那么 ojb 为 null
  后面的可变参数 Object 对应的自然就是参数。

* invoke() 返回的对象是 Object，所以实际上执行的时候要进行强制转换。

* 在对 Method 调用 invoke() 的时候，如果方法本身会抛出异常，那么这个异常就会经过包装，由 Method 统一抛出 InvocationTargetException。
  而通过 InvocationTargetException.getCause() 可以获取真正的异常。


### Constructor 的操控

在平常开发的时候，构造器也称构造方法，但是在反射机制中却把它与 Method 分离开来，单独用 Constructor 这个类表示。

Constructor 同 Method 差不多，但是它特别的地方在于，它能够创建一个对象。

在 Java 反射机制中有两种方法可以用来创建类的对象实例：Class.newInstance() 和 Constructor.newInstance()。

官方文档建议开发者使用后面这种方法，下面是原因：

* Class.newInstance() 只能调用无参的构造方法，而 Constructor.newInstance() 则可以调用任意的构造方法

* Class.newInstance() 通过构造方法直接抛出异常，而 Constructor.newInstance() 会把抛出来的异常包装到 InvocationTargetException 里面去，这个和 Method 行为一致

* Class.newInstance() 要求构造方法能够被访问，而 Constructor.newInstance() 却能够访问 private 修饰的构造器

# 总结:

* Java 中的反射是非常规编码方式

* Java 反射机制的操作入口是获取 Class 文件。 有 Class.forName()、 .class 和 Object.getClass() 3 种

* 获取 Class 对象后还不够，需要获取它的 Members，包含 Field、Method、Constructor

* Field 操作主要涉及到类别的获取，及数值的读取与赋值

* Method 算是反射机制最核心的内容，通常的反射都是为了调用某个 Method 的 invoke() 方法

* 通过 Class.newInstance() 和 Constructor.newInstance() 都可以创建类的对象实例，但推荐后者。因为它适应于任何构造方法，而前者只会调用可见的无参数的构造方法


# 参考

[细说反射，Java 和 Android 开发者必须跨越的坎](https://frank909.blog.csdn.net/article/details/74616922)
[反射技术在android中的应用](https://blog.csdn.net/tiefeng0606/article/details/51700866)
[浅谈JAVA反射机制在Android应用开发中的应用](https://www.oschina.net/question/163910_27112)
