## 配置

要想使用Glide，首先需要将这个库引入到我们的项目当中。在项目的app/build.gradle文件当中添加如下依赖：

```
    api 'com.github.bumptech.glide:glide:4.8.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.8.0'
```

另外，Glide中需要用到网络功能，因此你还得在 AndroidManifest.xml 中声明一下网络权限才行：

```
<uses-permission android:name="android.permission.INTERNET"/>
```

然后我们就可以使用Glide中的任意功能了。


## 加载图片

这个很简单：

```
Glide.with(this).load(url).into(imageView)
```

## 占位图

顾名思义，占位图就是指在图片的加载过程中，我们先显示一张临时的图片，等图片加载出来了再替换成要加载的图片。

```
RequestOptions options = new RequestOptions()
        .placeholder(R.drawable.loading);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```

这里我们先创建了一个 RequestOptions 对象，然后调用它的 placeholder() 方法来指定占位图，再将占位图片的资源id传入到这个方法中。
最后，在Glide的三步走之间加入一个apply()方法，来应用我们刚才创建的 RequestOptions 对象。

除了这种加载占位图之外，还有一种异常占位图。异常占位图就是指，如果因为某些异常情况导致图片加载失败，
比如说手机网络信号不好，这个时候就显示这张异常占位图。

```
RequestOptions options = new RequestOptions()
        .placeholder(R.drawable.ic_launcher_background)
        .error(R.drawable.error)
        .diskCacheStrategy(DiskCacheStrategy.NONE);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```


## 指定图片大小

实际上，使用 Glide 在大多数情况下我们都是不需要指定图片大小的，因为 __Glide 会自动根据 ImageView 的大小来决定图片的大小__，
以此保证图片不会占用过多的内存从而引发OOM。不过，如果你真的有这样的需求，必须给图片指定一个固定的大小，
Glide仍然是支持这个功能的。修改 Glide 加载部分的代码，如下所示：

```
RequestOptions options = new RequestOptions()
        .override(200, 100);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```

仍然非常简单，这里使用 override() 方法指定了一个图片的尺寸。也就是说，Glide 现在只会将图片加载成200*100像素的尺寸，
而不会管你的 ImageView 的大小是多少了。

如果你想加载一张图片的原始尺寸的话，可以使用Target.SIZE_ORIGINAL关键字，如下所示:

```
RequestOptions options = new RequestOptions()
        .override(Target.SIZE_ORIGINAL);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```

这样的话，Glide就不会再去自动压缩图片，而是会去加载图片的原始尺寸。当然，这种写法也会面临着更高的OOM风险。


## 缓存机制

Glide 的缓存设计可以说是非常先进的，考虑的场景也很周全。在缓存这一功能上，Glide又将它分成了两个模块:

一个是内存缓存，__内存缓存的主要作用是防止应用重复将图片数据读取到内存当中__。

一个是硬盘缓存，__硬盘缓存主要作用是防止应用重复从网络或其他地方重复下载和读取数据__。

内存缓存和硬盘缓存的相互结合才构成了 Glide 极佳的图片缓存效果。

默认情况下，Glide 自动就是开启内存缓存的。也就是说，当我们使用 Glide 加载了一张图片之后，这张图片就会被缓存到内存当中。
只要在它还没从内存中被清除，下次使用 Glide 再加载这张图片都会直接从内存当中读取，而不用重新从网络或硬盘上读取了，这样无疑
就可以大幅度提升图片的加载效率。比方说你在一个 RecyclerView 当中反复上下滑动， RecyclerView 中只要是 Glide 加载过的图片
都可以直接从内存当中迅速读取并展示出来，从而大大提升了用户体验。

而 Glide 最为人性化的是，你甚至不需要编写任何额外的代码就能自动享受到这个极为便利的内存缓存功能，因为 Glide 默认就已经将它开启了。

如果你有什么特殊的原因需要禁用内存缓存功能，Glide对此提供了接口：

```
RequestOptions options = new RequestOptions()
        .skipMemoryCache(true);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```

只需要调用skipMemoryCache()方法并传入true，就表示禁用掉Glide的内存缓存功能。


```
RequestOptions options = new RequestOptions()
        .diskCacheStrategy(DiskCacheStrategy.NONE);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```

调用 diskCacheStrategy() 方法并传入 DiskCacheStrategy.NONE，就可以禁用掉Glide的硬盘缓存功能了。

这个 diskCacheStrategy() 方法基本上就是 Glide 硬盘缓存功能的一切，它可以接收五种参数：

* DiskCacheStrategy.NONE： 表示不缓存任何内容

* DiskCacheStrategy.DATA： 表示只缓存原始图片

* DiskCacheStrategy.RESOURCE： 表示只缓存转换过后的图片

* DiskCacheStrategy.ALL ： 表示既缓存原始图片，也缓存转换过后的图片

* DiskCacheStrategy.AUTOMATIC： 表示让 Glide 根据图片资源智能地选择使用哪一种缓存策略（默认选项）

当我们使用 Glide 去加载一张图片的时候，Glide 默认并不会将原始图片展示出来，而是会对图片进行压缩和转换,
总之就是经过种种一系列操作之后得到的图片，就叫转换过后的图片。


## 指定加载格式

Glide其中一个非常亮眼的功能就是可以加载GIF图片，而同样作为非常出色的图片加载框架的Picasso是不支持这个功能的。

使用 Glide 加载 GIF 图并不需要编写什么额外的代码，Glide 内部会自动判断图片格式。比如我们将加载图片的URL地址改成一张GIF图，
如下所示：

```
Glide.with(this)
     .load("https://media.giphy.com/media/5mYwgGvIR2GN2g0ZZj/giphy.gif")
     .into(imageView);

```
也就是说，不管我们传入的是一张普通图片，还是一张GIF图片，Glide都会自动进行判断，并且可以正确地把它解析并展示出来。

但是如果我想指定加载格式该怎么办呢？就比如说，我希望加载的这张图必须是一张静态图片，我不需要 Glide 自动帮我判断它到底是静图还是GIF图。

想实现这个功能仍然非常简单，我们只需要再串接一个新的方法就可以了，如下所示：

```
Glide.with(this)
     .asBitmap()
     .load("https://media.giphy.com/media/5mYwgGvIR2GN2g0ZZj/giphy.gif")
     .into(imageView);

```

可以看到，这里在 with() 方法的后面加入了一个 asBitmap() 方法，这个方法的意思就是说这里只允许加载静态图片，不需要
Glide 去帮我们自动进行图片格式的判断了。如果你传入的还是一张 GIF 图的话，Glide 会展示这张 GIF 图的第一帧，而不会去播放它。


## 回调与监听

### into()方法

我们都知道 Glide 的 into() 方法中是可以传入 ImageView 的。那么 into() 方法还可以传入别的参数吗？我们可以让 Glide 加载
出来的图片不显示到 ImageView 上吗？答案是肯定的，这就需要用到自定义Target功能。


```
SimpleTarget<Drawable> simpleTarget = new SimpleTarget<Drawable>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<? super Drawable> transition) {
        imageView.setImageDrawable(resource);
    }
};

public void loadImage(View view) {
    Glide.with(this)
         .load("https://media.giphy.com/media/5mYwgGvIR2GN2g0ZZj/giphy.gif")
         .into(simpleTarget);
}
```

这里我们创建了一个SimpleTarget的实例，并且指定它的泛型是 Drawable，然后重写了 onResourceReady() 方法。
在 onResourceReady() 方法中，我们就可以获取到 Glide 加载出来的图片对象了，也就是方法参数中传过来的Drawable对象。
有了这个对象之后你可以使用它进行任意的逻辑操作，这里只是简单地把它显示到了ImageView上。


### preload()方法

Glide加载图片虽说非常智能，它会自动判断该图片是否已经有缓存了，如果有的话就直接从缓存中读取，没有的话再从网络去下载。
但是如果我希望提前对图片进行一个预加载，等真正需要加载图片的时候就直接从缓存中读取，不想再等待慢长的网络加载时间了，这该怎么办呢？

Glide专门给我们提供了预加载的接口，也就是preload()方法，我们只需要直接使用就可以了。preload()方法有两个方法重载，
一个不带参数，表示将会加载图片的原始尺寸，另一个可以通过参数指定加载图片的宽和高。

preload()方法的用法也非常简单，直接使用它来替换into()方法即可，如下所示：

```
Glide.with(this)
     .load("https://media.giphy.com/media/5mYwgGvIR2GN2g0ZZj/giphy.gif")
     .preload();
```

### submit()方法

那么首先我们还是先来看下基本用法。submit()方法有两个方法重载：

* submit()

* submit(int width, int height)

其中 submit() 方法是用于下载原始尺寸的图片，而 submit(int width, int height) 则可以指定下载图片的尺寸。

当调用了 submit() 方法后会立即返回一个 FutureTarget 对象，然后 Glide 会在后台开始下载图片文件。接下来我们调用 FutureTarget 的
get() 方法就可以去获取下载好的图片文件了，如果此时图片还没有下载完，那么 get() 方法就会阻塞住，一直等到图片下载完成才会有值返回。


```
public void downloadImage() {
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                String url = "https://media.giphy.com/media/5mYwgGvIR2GN2g0ZZj/giphy.gif";
                final Context context = getApplicationContext();
                FutureTarget<File> target = Glide.with(context)
                        .asFile()
                        .load(url)
                        .submit();
                final File imageFile = target.get();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(context, imageFile.getPath(), Toast.LENGTH_LONG).show();
                    }
                });
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }).start();
}
```

首先，submit() 方法必须要用在子线程当中，因为刚才说了 FutureTarget 的 get() 方法是会阻塞线程的，因此这里的第一步就是 new 
了一个 Thread。在子线程当中，我们先获取了一个 Application Context，这个时候不能再用 Activity 作为 Context 了，因为会有 
Activity 销毁了但子线程还没执行完这种可能出现。



### listener()方法

其实listener()方法的作用非常普遍，它可以用来监听Glide加载图片的状态。举个例子，比如说我们刚才使用了 preload() 方法来对图片进行预加载，
但是我怎样确定预加载有没有完成呢？还有如果 Glide 加载图片失败了，我该怎样调试错误的原因呢？

答案都在listener()方法当中。

```
Glide.with(this)
     .load("http://www.guolin.tech/book.png")
     .listener(new RequestListener<Drawable>() {
         @Override
         public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
             return false;
         }

         @Override
         public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
             return false;
         }
     })
     .into(imageView);
```

这里我们在 into() 方法之前串接了一个 listener() 方法，然后实现了一个 RequestListener 的实例。其中 RequestListener 需要实现两个方法，
一个 onResourceReady() 方法，一个 onLoadFailed() 方法。从方法名上就可以看出来了，当图片加载完成的时候就会回调 onResourceReady() 方法，
而当图片加载失败的时候就会回调onLoadFailed()方法，onLoadFailed()方法中会将失败的GlideException参数传进来，这样我们就可以定位具体失败的原因了。

onResourceReady() 方法和 onLoadFailed() 方法都有一个布尔值的返回值，返回 false 就表示这个事件没有被处理，还会继续向下传递，返回 true 就表示
这个事件已经被处理掉了，从而不会再继续向下传递。举个简单点的例子，如果我们在 RequestListener 的 onResourceReady() 方法中返回了true，
那么就不会再回调 Target 的 onResourceReady() 方法了。


### 图片变换

图片变换的意思就是说，Glide 从加载了原始图片到最终展示给用户之前，又进行了一些变换处理，从而能够实现一些更加丰富的图片效果，
如图片圆角化、圆形化、模糊化等等。

添加图片变换的用法非常简单，我们只需要在 RequestOptions 中串接 transforms() 方法，并将想要执行的图片变换操作作为参数
传入transforms()方法即可，如下所示：

```
RequestOptions options = new RequestOptions()
        .transforms(...);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```

至于具体要进行什么样的图片变换操作，这个通常都是需要我们自己来写的。不过 Glide 已经内置了几种图片变换操作，我们可以直接拿来使用，
比如CenterCrop、FitCenter、CircleCrop等。


但所有的内置图片变换操作其实都不需要使用 transform() 方法，Glide 为了方便我们使用直接提供了现成的API：

```
RequestOptions options = new RequestOptions()
        .centerCrop();

RequestOptions options = new RequestOptions()
        .fitCenter();

RequestOptions options = new RequestOptions()
        .circleCrop();

```

当然，这些内置的图片变换 API 其实也只是对 transform() 方法进行了一层封装而已，它们背后的源码仍然还是借助 transform() 方法来实现的。


### 自定义模块

自定义模块功能可以将更改 Glide配置，替换 Glide 组件等操作独立出来，使得我们能轻松地对 Glide 的各种配置进行自定义，并且又和Glide的
图片加载逻辑没有任何交集，这也是一种低耦合编程方式的体现。

首先定义一个我们自己的模块类，并让它继承自 AppGlideModule，如下所示：

```
@GlideModule
public class MyAppGlideModule extends AppGlideModule {

    @Override
    public void applyOptions(Context context, GlideBuilder builder) {

    }

    @Override
    public void registerComponents(Context context, Glide glide, Registry registry) {

    }

}
```

可以看到，在 MyAppGlideModule 类当中，我们重写了 applyOptions() 和 registerComponents() 方法，这两个方法分别就是用来更改 
Glide 配置以及替换 Glide 组件的。在 MyAppGlideModule 类在上面，加入了一个 @GlideModule 的注解,这个注解已经能够让 Glide 
识别到这个自定义模块了。这样的话，就将 Glide 自定义模块的功能完成了。后面只需要在 applyOptions() 和 registerComponents() 
这两个方法中加入具体的逻辑，就能实现更改 Glide 配置或者替换 Glide 组件的功能了。


### 自定义模块的原理

Glide的实例到底是在哪里创建的呢？我们来看下Glide类中的 initializeGlide 方法的源码，如下所示：

```
public class Glide {
   ...

  private static void initializeGlide(@NonNull Context context, @NonNull GlideBuilder builder) {
    Context applicationContext = context.getApplicationContext();
    GeneratedAppGlideModule annotationGeneratedModule = getAnnotationGeneratedGlideModules();
    List<com.bumptech.glide.module.GlideModule> manifestModules = Collections.emptyList();
    if (annotationGeneratedModule == null || annotationGeneratedModule.isManifestParsingEnabled()) {
      manifestModules = new ManifestParser(applicationContext).parse();
    }

    ...

    for (com.bumptech.glide.module.GlideModule module : manifestModules) {
      module.applyOptions(applicationContext, builder);
    }
    if (annotationGeneratedModule != null) {
      annotationGeneratedModule.applyOptions(applicationContext, builder);
    }
    Glide glide = builder.build(applicationContext);
    for (com.bumptech.glide.module.GlideModule module : manifestModules) {
      module.registerComponents(applicationContext, glide, glide.registry);
    }
    if (annotationGeneratedModule != null) {
      annotationGeneratedModule.registerComponents(applicationContext, glide, glide.registry);
    }
    applicationContext.registerComponentCallbacks(glide);
    Glide.glide = glide;
  }
    ...
}

```
上面代码可以看出，我们通过注解或者Manifest中配置我们的自定义模块。由于可以自定义任意多个模块，因此这里我们将会得到一个 GlideModule 的 List 集合。
创建了一个 GlideBuilder 对象，并通过一个循环调用每一个 GlideModule 的applyOptions()方法，同时也把GlideBuilder对象作为参数传入到这个方法中。
而 applyOptions() 方法就是我们可以加入自己的逻辑的地方了。

再往下的一步就非常关键了，这里调用了 GlideBuilder 的 build()方法，并返回了一个 Glide 对象。也就是说，Glide对象的实例就是在这里创建的了，
那么我们跟到这个方法当中瞧一瞧：

```
  @NonNull
  Glide build(@NonNull Context context) {
    if (sourceExecutor == null) {
      sourceExecutor = GlideExecutor.newSourceExecutor();
    }

    if (diskCacheExecutor == null) {
      diskCacheExecutor = GlideExecutor.newDiskCacheExecutor();
    }

    if (animationExecutor == null) {
      animationExecutor = GlideExecutor.newAnimationExecutor();
    }

    if (memorySizeCalculator == null) {
      memorySizeCalculator = new MemorySizeCalculator.Builder(context).build();
    }

    if (connectivityMonitorFactory == null) {
      connectivityMonitorFactory = new DefaultConnectivityMonitorFactory();
    }

    if (bitmapPool == null) {
      int size = memorySizeCalculator.getBitmapPoolSize();
      if (size > 0) {
        bitmapPool = new LruBitmapPool(size);
      } else {
        bitmapPool = new BitmapPoolAdapter();
      }
    }

    if (arrayPool == null) {
      arrayPool = new LruArrayPool(memorySizeCalculator.getArrayPoolSizeInBytes());
    }

    if (memoryCache == null) {
      memoryCache = new LruResourceCache(memorySizeCalculator.getMemoryCacheSize());
    }

    if (diskCacheFactory == null) {
      diskCacheFactory = new InternalCacheDiskCacheFactory(context);
    }

    if (engine == null) {
      engine =
          new Engine(
              memoryCache,
              diskCacheFactory,
              diskCacheExecutor,
              sourceExecutor,
              GlideExecutor.newUnlimitedSourceExecutor(),
              GlideExecutor.newAnimationExecutor(),
              isActiveResourceRetentionAllowed);
    }

    RequestManagerRetriever requestManagerRetriever =
        new RequestManagerRetriever(requestManagerFactory);

    return new Glide(
        context,
        engine,
        memoryCache,
        bitmapPool,
        arrayPool,
        requestManagerRetriever,
        connectivityMonitorFactory,
        logLevel,
        defaultRequestOptions.lock(),
        defaultTransitionOptions);
  }
```

这个方法中会创建 BitmapPool、MemoryCache、DiskCache等对象的实例，并在最后一行创建一个Glide对象的实例，然后将前面创建的
这些实例传入到 Glide 对象当中，以供后续的图片加载操作使用。createGlide()方法中创建任何对象的时候都做了一个空检查，只有在
对象为空的时候才会去创建它的实例。也就是说，如果我们可以在 applyOptions() 方法中提前就给这些对象初始化并赋值，那么在 createGlide()
方法中就不会再去重新创建它们的实例了，从而也就实现了更改 Glide 配置的功能。


现在继续回到 Glide 的 initializeGlide 方法中，得到了 Glide 对象的实例之后，接下来又通过一个循环调用了每一个 GlideModule 的 
registerComponents() 方法，在这里我们可以加入替换 Glide 的组件的逻辑。


### 更改Glide配置

刚才在分析自定义模式工作原理的时候其实就已经提到了，如果想要更改 Glide 的默认配置，其实只需要在 applyOptions() 方法中提前将 Glide 
的配置项进行初始化就可以了。那么Glide一共有哪些配置项呢？这里做了一个列举：



* setMemoryCache()

  用于配置Glide的内存缓存策略，默认配置是 LruResourceCache。

* setBitmapPool() 

  用于配置Glide的Bitmap缓存池，默认配置是 LruBitmapPool。

* setDiskCache()

  用于配置Glide的硬盘缓存策略，默认配置是 InternalCacheDiskCacheFactory。

* setDiskCacheService()

  用于配置 Glide 读取缓存中图片的异步执行器，默认配置是 FifoPriorityThreadPoolExecutor，也就是先入先出原则。

* setResizeService()

  用于配置 Glide 读取非缓存中图片的异步执行器，默认配置也是 FifoPriorityThreadPoolExecutor。

* setDecodeFormat()
 
  用于配置 Glide 加载图片的解码模式，默认配置是RGB_565。


其实 Glide 的这些默认配置都非常科学且合理，使用的缓存算法也都是效率极高的，因此在绝大多数情况下我们并不需要去修改这些默认配置，这也是Glide用法能如此简洁的一个原因。


### 更简单的组件替换


Glide官方给我们提供了非常简便的HTTP组件替换方式。并且除了支持OkHttp3之外，还支持OkHttp2和Volley。

```
    // Glide HTTP组件替换
    implementation 'com.github.bumptech.glide:okhttp3-integration:4.5.0'
```

































[探究Glide的自定义模块功能](https://blog.csdn.net/guolin_blog/article/details/78179422)









































