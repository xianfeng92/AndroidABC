## Bitmap的高效加载

在介绍 Bitmap 的高效加载之前，先说一下如何高效的加载一个 Bitmap，Bitmap 在Android中指的是一张图片，BitmapFactory 类提供了
四类方法：decodeFile，decodeResource，decodeStream，decodeByteArray，分别用于支持从文件系统，资源，输入流以及字节数组
中加载出一个 Bitmap 对象。其中 decodeFile 和 decodeResource 又间接的调用 decodeStream 方法。
这四类方法最终实在 android 的底层实现的，对应的几个 native 方法。

如何高效的加载Bitmap呢？

其实核心思想也很简单，那就是采用 BitmapFactory.Options 来加载所需尺寸的图片，这里假设通过 ImageView 来显示图片，很多时候 
ImageView 并没有图片的原始尺寸那么大，这个时候把整个图片加载进来后再设给 ImageView，这显然是没有必要的，BitmapFactory.Options 
就可以按照一定的采用率来压缩图片，将缩小后的图片在 ImageView 中显示，这样可以降低内存占用，从而一定程度上避免 OOM，提高 Bitmap
加载时的性能。


通过采样率即有效的加载图片，那么到底如何回去采样率呢？

1. 将 BitmapFactory.Options 的 inJustDecodeBounds 参数设置为true并且加载图片

2. 从 BitmapFactory.Options 中取出图片的原始宽高

3. 根据采用率的规则结合目标的 View 所需计算采用率

4. 将 BitmapFactory.Options 的 inJustDecodeBounds 参数设置为 false 然后重新加载图片

经过上面的4个步骤，加载出的图片就是最终缩放的图片，当然也有可能不需要缩放，这么说一下 inJustDecodeBounds 这个参数，为true的时候， 
__BitmapFactory 只会解析图片的原始宽高，并不会去真正的加载图片，所以这是轻量级的操作__。另外需要注意的是，这个时候 BitmapFactory
获取到的图片宽高和图片位置以及程序运行的设备有关，比如同一张图放在不同的 drawable 目录下或者程序运行在不同屏幕密度的设备上，
这都可能导致 BitmapFactory 获取到不同的结果，之所以会出现这个现象，这和 Android 的资源加载机制有关。


```
    public static Bitmap decodeSampleBitmapFromResource(Resources res, int resId, int reqWidth, int reqHeight) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(res, resId, options);
        options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeResource(res, resId, options);
    }

    public static int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
        int width = options.outWidth;
        int height = options.outHeight;
        int inSampleSize = 1;

        if (height > reqHeight || width > reqWidth) {
            int halfHeight = height / 2;
            int halfWidth = width / 2;
            while (halfHeight / inSampleSize >= reqHeight && (halfWidth / inSampleSize) >= reqWidth) {
                inSampleSize *= 2;
            }
        }
        return inSampleSize;
    }
```

## Android中的缓存策略

## 缓存

当程序第一次从网络加载图片时，就将其缓存到存储设备上，这样下次使用这张图片就不用再从网络上获取了。很多时候为了提高应用的用户体验，
往往还会把图片在内存中缓存一份，这样当应用打算从网络上请求一张图片时，程序会首先从内存中获取，如果内存中没有那就从存储设备中获取，
如果存储设备中也没有，那就从网络中下载这张图片。因为从内存中加载图片比从存储设备中加载图片要快，这样即提高了程序的效率又为用户节约
了不必要的流量开销，上述缓存策略不仅仅适用于图片，也适用其他文件类型。


说到缓存策略，其实并没有统一的标准，一般来说，缓存策略主要包括缓存的添加、获取和删除这三类操作。如何添加和获取缓存这个比较好理解，
那么为什么还要删除缓存呢？这是因为不管是内存缓存还是存储设备缓存，它们的缓存大小都是有限制的。如果存储容量满了，但是程序还是需要
向其中添加缓存，这个时候就需要删除一些旧的缓存并添加新的缓存，如何定义缓存的新旧这就是一种策略，不同的策略就对应不同的缓存算法，比如
可以简单的根据文件最后的修改时间来定义新旧的缓存，当缓存满时就将最后修改时间较早的缓存移除，这就是一种缓存算法，但是这种算法并不完美。


目前常用的一种缓存算法是LRU（Least Recently Used），LRU 是近期最少使用算法，它的核心思想是当缓存满时，会优先淘汰那些近期最少使用
的缓存对象。采用LRU算法的缓存有两种，LruCache 和 DiskLruCache，LruCache 用于实现内存缓存，而 DiskLruCache 则充当存储设备缓存，
通过这二者的完美结合，可以很方便的实现一个具体很高实用价值的 ImageLoader。

## LruCache

LruCache 是android3.1所提供的一个缓存类，通过v4兼容包，可以兼容到早期的Android版本。LruCache是一个泛型类，它内部采用了 LinkedHashMap 
以强引用的方式存储外界的缓存对象，然后提供 get 和 put 方法来完成缓存的获取和添加操作，当缓存满时，LruCache 会移除较早使用的缓存对象，然后
再添加新的缓存对象。

* 强引用：直接的对象引用

* 软引用：当一个对象只有软引用存在时，系统内存不足时此对象会被gc回收

* 弱引用：当一个对象只有弱引用存在时，此对象会随时被gc回收


另外，LruCache 是线程安全的，下面是 LruCache 的定义：

```
public class LruCache<K, V> {
    private final LinkedHashMap<K, V> map;
```

LruCache的初始化过程：

```
                int maxMemory = (int)(Runtime.getRuntime().maxMemory() / 1024);
                int cacheSize = maxMemory / 8;
                mMemoryLruCache  = new LruCache<String,Bitmap>(cacheSize){
                    @Override
                    protected int sizeOf(String key, Bitmap value) {
                        return value.getRowBytes() * value.getHeight() / 1024;
                    }
                };
```

在上面的代码中，只需要提供缓存的总容量大小并且重写 sizeOf 即可，sizeOf 的作用是计算缓存对象的大小，这里大小的单位需要和总容量的单位一致。
对于上面的实例代码，总容量的大小是当前进程可用的八分之一，单位是 KB,而 sizeOf 则完成了 bitmap 对象的大小计算。
之所以除以1024也是为了将其单位转换成 KB。一些特殊情况下，还需要重写 entryRemoved 方法，LruCache 移除旧缓存时会调用 entryRemoved 方法，因此
可以在 entryRemoved 中完成一些资源的回收工作（如果需要的话）。
 

除了LruCache的创建外，我们还可以 get 和 put：

```
                mMemoryLruCache.get(key); // 从 LruCache 中获取一个缓存对象
                mMemoryLruCache.put(key,bitmap);// 向 LruCache 中存储一个缓存对象
```


当然， LruCache 还支持删除操作，通过 remove 方法即可删除一个指定的缓存对象。


## DiskLruCache

DiskLruCache 用于实现存储设备缓存，即磁盘缓存，它通过缓存对象写入文件系统从而实现缓存的效果。

### DiskLruCache的创建

DiskLruCache并不能通过构造方法来创建，它提供了open方法才可以：

```
public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)

```

第一个参数表示磁盘缓存在文件系统中的存储路径，缓存路径可以选择SD卡上的缓存目录，具体是指sdcard/Android/data/package_name/cache目录，
其中 package_name 表示当前应用的包名，当应用被卸载后，此目录会一并被删除。当然也可以选择 SD 卡上的其它指定目录。如果应用卸载后，希望删除缓存文件，那么
就选择 SD 上的缓存目录，如果希望保留缓存数量就应该选择 SD 卡上的其他特定目录。

第二个参数表示应用的版本号，一般设为1即可，当版本号发生改变时 DiskLruCache 会清空之前所有的缓存文件，这个特性作用不大，很多情况下即使版本号更改了还是有效

第三个参数表示单个节点所对应的数据个数，一般设为1即可

第四个参数表示缓存的总大小，比如50MB，当缓存大小超过这个设定值的时候，它会清除一些缓存来保证总大小不大于这个值


```
                long DISK_CACHE_SIZE = 1024 * 1024 *50;
                File diskCacheDir = getDiskCacheDir(this,"bitmap");
                if(!diskCacheDir.exists()){
                    diskCacheDir.mkdirs();
                }
                try {
                    mDiskLruCache = DiskLruCache.open(diskCacheDir,1,1,DISK_CACHE_SIZE);
                } catch (IOException e) {
                    e.printStackTrace();
                }
```



### DiskLruCache的缓存添加


DiskLruCache 的缓存添加的操作是通过 Editor 完成的，Editor 表示一个缓存对象的编辑对象，这里以图片缓存为例，首先需要获取图片url所对应的key，
然后根据 key 就可以通过 edit() 来获取 Editor 对象，如果一个缓存对象正在被编辑，那么edit()会返回null，之所以把url转换成key，是因为图片的 url 中很可能有特殊字符，
这将影响url在Android中的直接使用，一般采用 url 的 md5 值作为key。

```
   private String hashKeyFormUrl(String url){
        String cacheKey;
        try {
            MessageDigest mDigest = MessageDigest.getInstance("MD5");
            mDigest.update(url.getBytes());
            cacheKey = bytesToHexString(mDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            cacheKey = String.valueOf(url.hashCode());
        }
        return cacheKey;
    }

    private String bytesToHexString(byte[] digest) {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < digest.length; i++) {
            String hex = Integer.toHexString(0xFF&digest[i]);
            if(hex.length() == 1){
                sb.append('0');
            }
            sb.append(hex);
        }
        return sb.toString();
    }
```

将图片的 url 转为 key 之后，就可以获取 Editor 对象了，对于这个key来说，如果当前不存在其他 Editor 对象，那么edit()就会返回一个新的Editor对象，
通过它就可以得到一个文件输出流，需要注意的是，由于前面的 open 设置了一个节点只能有一个数据，因此下面的 DISK_CACHE_SIZE 的常量直接设置为 0 即可：

```
                int DISK_CACHE_INDEX = 0;
                String key = hashKeyFormUrl(url);
                try {
                    DiskLruCache.Editor editor = mDiskLruCache.edit(key);
                    if(editor != null){
                        OutputStream outputStream = editor.newOutputStream(DISK_CACHE_INDEX);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
```


有了文件输出流，接下来要怎么做，其实是这样的，当从网络下载图片的时候，图片就可以写入文件系统了：

```
 private boolean downloadUrlToStream(String urlString,OutputStream outputStream){
        int IO_BUFFER_SIZE = 0;
        HttpURLConnection urlConnection = null;
        BufferedOutputStream out = null;
        BufferedInputStream in = null;
        try {
            URL url = new URL(urlString);
            urlConnection = (HttpURLConnection) url.openConnection();
            in = new BufferedInputStream(urlConnection.getInputStream(),IO_BUFFER_SIZE);
            out = new BufferedOutputStream(outputStream,IO_BUFFER_SIZE);
            int b ;
            while ((b = in.read()) != -1){
                out.write(b);
            }
            return true;
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(urlConnection != null){
                urlConnection.disconnect();
            }
        }
        return false;
    }
```

经过上面的步骤，其实并没有真正的将图片写入文件系统，还必须通过 Editor 的 commit 来提交操作，如果这个图片下载过程发生了异常，
那么还可以通过 Editor 的 abort 来回退操作,这个过程如下所示：

```
              DiskLruCache.Editor editor = mDiskLruCache.edit(key);
                    if(editor != null){
                        OutputStream outputStream = editor.newOutputStream(DISK_CACHE_INDEX);
                        if(downloadUrlToStream(url,outputStream)){
                            editor.commit();
                        }else {
                            editor.abort();
                        }
                    }
```

经过上面几个步骤，图片已经被正确的写入到文件系统了，接下来图片获取的操作就不需要请求网络了。


### DiskLruCache的缓存查找

和缓存的添加过程类似，缓存查找过程中也需要将 url 转换为key。然后通过 DiskLruCache 的get方法得到一个 Snapshot 对象，接着再通过 Snapshot 
对象即可得到缓存的文件输入流，有了文件输入流，自然就可以得到 Bitmap 对象了。为了避免加载图片过程中导致的OOM问题，一般不建议直接加载原始图片。
上面介绍的缩放，实际上对 FileInputStream 的缩放存在问题，原因是 FileInputStaeam 是一种有序的文件流，而两次 decodeStream 调用影响了文件流的位置属性，
导致了第二次 decodeStream 时得的是null。为了解决这个问题，可以通过文件流得到它所对应的文件描述符，然后再通过 BitmapFactory.decodeFileDescriptor 方法
来加载一张缩放后的图片，这个过程的实现如下：

```
        Bitmap bitmap = null;
                String keys = hashKeyFormUrl(url);
                try {
                    DiskLruCache.Snapshot snapshot = mDiskLruCache.get(keys);
                    if(snapshot != null){
                        FileInputStream fileInputStream = (FileInputStream) snapshot.getInputStream(DISK_CACHE_INDEX);
                        FileDescriptor fileDescriptor = fileInputStream.getFD();
                        bitmap = mImageResizer.decodeSampledBitmapFromFileDescriptor(fileDescriptor,reqWidth,reqHeight);
                        if(bitmap != null){
                            addBitmapToMemoryCache(keys,bitmap);
                        }
                    }
```

