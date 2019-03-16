
# ClassLoader

Android 使用 Dalvik 虚拟机加载可执行程序，不能直接加载基于 class 的jar, 其需要将 class 转化为 dex 字节码，从而执行代码。优化后的字节码文件可以存在一个 .jar 中，只要其内部存放的是.dex即可。Dalvik 虚拟机如同其他Java虚拟机一样，在运行程序时首先需要将对应的类加载到内存中。而在 Java 标准的虚拟机中，类加载可以从 class 文件中读取，也可以是其他形式的二进制流。因此，我们常常利用这一点，在程序运行时手动加载 Class , 从而达到代码动态加载执行的目的。

只不过Android平台上虚拟机运行的是 Dex 字节码,一种对 class 文件优化的产物,传统 Class 文件是一个Java源码文件会生成一个.class文件，而Android是把所有 Class 文件进行合并，优化，然后生成一个最终的 class.dex ,目的是把不同class文件重复的东西只需保留一份, 如果我们的 Android 应用不进行 dex 拆分处理,最后一个应用的 apk 
只会有一个dex文件。

关于动态加载apk，理论上可以用到的有 DexClassLoader、PathClassLoader和URLClassLoader。

* DexClassLoader ：可以加载文件系统上的jar、dex、apk

* PathClassLoader ：可以加载 /data/data 目录下的apk，这也意味着，它只能加载已经安装的apk,apk 被安装之后，APK 文件的代码以及资源会被系统存放在固定的目录
  (比如:/data/data/package_name/1.apk),系统在进行类加载的时候，会自动去这一个或者几个特定的路径来寻找这个类.


* URLClassLoader ：可以加载java中的jar，但是由于dalvik不能直接识别jar，所以此方法在android中无法使用，尽管还有这个类


关于jar、dex和apk，dex和apk是可以直接加载的，因为它们都是或者内部有dex文件，而原始的jar是不行的，必须转换成 dalvik 所能识别的字节码文件，转换工具可以使用android sdk中 build-tools 目录下的 dx

转换命令 ：dx --dex --output=dest.jar src.jar


## 动态加载未安装 apk 的资源

 宿主程序调起未安装的apk，一个很大的问题就是资源如何访问，具体来说就是，凡是以R开头的资源都不能访问了，因为宿主程序中并没有apk中的资源，所以通过R来加载资源是行不通的，程序会报错：无法找到某某id所对应的资源。我们知道，activity 的工作主要是由 ContextImpl 来完成的， 它在 activity 中是一个叫做 mBase 的成员变量。注意到 Context 中有如下两个抽象方法，看起来是和资源有关的，实际上 context 就是通过它们来获取资源的，这两个抽象方法的真正实现在 ContextImpl 中。也即是说，只要我们自己实现这两个方法，就可以解决资源问题了。

```

    /** Return an AssetManager instance for your application's package. */
    public abstract AssetManager getAssets();

    /** Return a Resources instance for your application's package. */
    public abstract Resources getResources();

```
 
下面看一下如何实现这两个方法

首先要加载apk中的资源:

```
  protected void loadResources() {
        try {
            AssetManager assetManager = AssetManager.class.newInstance();
            Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);
            addAssetPath.invoke(assetManager, mDexPath);
            mAssetManager = assetManager;
        } catch (Exception e) {
            e.printStackTrace();
        }
        Resources superRes = super.getResources();
        mResources = new Resources(mAssetManager, superRes.getDisplayMetrics(),
                superRes.getConfiguration());
        mTheme = mResources.newTheme();
        mTheme.setTo(super.getTheme());
    }
```

加载的方法是通过反射，通过调用 AssetManager 中的 addAssetPath 方法，我们可以将一个 apk 中的资源加载到 Resources 中，由于addAssetPath是隐藏api我们无法直接调用，所以只能通过反射，下面是它的声明，通过注释我们可以看出，传递的路径可以是zip文件也可以是一个资源目录，而apk就是一个zip，所以直接将apk的路径传给它，资源就加载到 AssetManager 中了，然后再通过 AssetManager 来创建一个新的 Resources 对象，这个对象就是我们可以使用的 apk 中的资源了，这样我们的问题就解决了。

```
 /**
     * Add an additional set of assets to the asset manager.  This can be
     * either a directory or ZIP file.  Not for use by applications.  Returns
     * the cookie of the added asset, or 0 on failure.
     * {@hide}
     */
    public final int addAssetPath(String path) {
        int res = addAssetPathNative(path);
        return res;
    }
```


其次是要实现那两个抽象方法:

```
 @Override
    public AssetManager getAssets() {
        return mAssetManager == null ? super.getAssets() : mAssetManager;
    }
 
    @Override
    public Resources getResources() {
        return mResources == null ? super.getResources() : mResources;
    }
```

这样一来，在apk中就可以通过R来访问资源了。


### 动态加载apk中的图片



[Android apk动态加载机制的研究](https://blog.csdn.net/singwhatiwanna/article/details/22597587)

[Android apk动态加载机制的研究（二）：资源加载和activity生命周期管理](https://blog.csdn.net/singwhatiwanna/article/details/23387079)