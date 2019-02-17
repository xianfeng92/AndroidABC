# Andorid屏幕适配方案

Android适配最核心的问题有两个，其一，就是适配的效率，即把设计图转化为App界面的过程是否高效，其二如何保证实现UI界面在不同尺寸和分辨率的手机中UI的一致性。

## 几个重要概念

__屏幕尺寸__

屏幕尺寸指__屏幕的对角线的长度__，单位是英寸，1英寸=2.54厘米

比如常见的屏幕尺寸有3.7、4.2、5.0、5.5、6.0等。


__屏幕分辨率__

屏幕分辨率是指__屏幕在横纵方向上的像素点数之积__，单位是px，1px=1个像素点。一般以纵向像素*横向像素，如1960*1080。


__屏幕像素密度__

屏幕像素密度是指每英寸上的像素点数，单位是dpi，即“dot per inch”的缩写。屏幕像素密度与屏幕尺寸和屏幕分辨率有关，在单一变化条件下，屏幕尺寸越小、分辨率越高，像素密度越大，反之越小。

dpi = 屏幕分辨率/屏幕尺寸

__dp、dip、dpi、sp、px__

大多数情况下，比如UI设计、Android原生API都会以px作为统一的计量单位，像是获取屏幕宽高等。

dip和dp是一个意思，都是Density Independent Pixels的缩写，即__密度无关像素__,即以dip或dp作单位的控件的显示大小与屏幕的像素密度无关。dpi是屏幕像素密度，假如一英寸里面有160个像素，这个屏幕的像素密度就是160dpi，那么在这种情况下，dp和px如何换算呢？在Android中，规定以160dpi为基准，1dip=1px，如果密度是320dpi，则1dip=2px，以此类推。

而sp，即scale-independent pixels（尺度独立性像素），与dp类似，但是可以根据文字大小首选项进行放缩，是设置字体大小的御用单位。

__mdpi、hdpi、xdpi、xxdpi__

 mdpi、hdpi、xdpi、xxdpi用来修饰Android中的drawable文件夹及values文件夹，用来区分不同像素密度下的图片和dimen值。在设计图标时，对于五种主流的像素密度（MDPI、HDPI、XHDPI、XXHDPI 和 XXXHDPI）应按照 2:3:4:6:8 的比例进行缩放。例如，一个启动图标的尺寸为48x48 dp，这表示在 MDPI 的屏幕上其实际尺寸应为 48x48 px，在 HDPI 的屏幕上其实际大小是 MDPI 的 1.5 倍 (72x72 px)，在 XDPI 的屏幕上其实际大小是 MDPI 的 2 倍 (96x96 px)，依此类推。


## Android官方提供的屏幕适配方法

### 让你的布局能充分的自适应屏幕
 
   
  __多使用 "wrap_content" 和 "match_parent"__

  使用了"wrap_content"，相应视图的宽和高就会被设定成刚好能够包含视图中内容的最小值,使用了"match_parent"(在Android API 8之前叫作"fill_parent")，就会让视图的宽和高延伸至充满整个父布局。通过使用"wrap_content"和"match_parent"来替代硬编码的方式定义视图大小，你的视图要么仅仅使用了需要的那边一点空间，要么就会充满所有可用的空间。当父布局大小改变时，使用了match parent的控件其大小会填充整个父布局，使用了warp content的控件其大小不会变化。

  __多使用RelativeLayout__

  通过多层嵌套LinearLayout和组合使用"wrap_content"和"match_parent"已经可以构建出足够复杂的布局。但是LinearLayout无法允许你准确地控制子视图之间的位置关系，所有LinearLayout中的子视图只能简单的一个挨着一个地排列。如果你需要让子视图能够有更多的排列方式，而不是简单地排成一行或一列，使用RelativeLayout将会是更好的解决方案。__RelativeLayout允许布局的子控件之间使用相对定位的方式控制控件的位置，比如你可以让一个子视图居屏幕左侧对齐，让另一个子视图居屏幕右侧对齐。__ 这样即使屏幕大小改变，视图之间的相对位置也不会改变。
  
 __使用非密度制约像素__

  由于各种屏幕的像素密度都有所不同，因此相同数量的像素在不同设备上的实际大小也有所差异，这样使用像素定义布局尺寸就会产生问题。因此，请务必使用 dp 或 sp 单位指定尺寸。dp 是一种非密度制约像素，在160dpi的屏幕上，1dp=1px。sp 也是一种基本单位，但它可根据用户的偏好文字大小进行调整（即尺度独立性像素），因此我们应将该测量单位用于定义文字大小。
 
 虽然说dp可以去除不同像素密度的问题，使得1dp在不同像素密度上面的显示效果相同，但是还是由于Android屏幕设备的多样性，如果使用dp来作为度量单位，并不是所有的屏幕的宽度都是相同的dp长度。比如说，Nexus S和Nexus One属于hdpi，屏幕宽度是320dp，而Nexus 5属于xxhdpi，屏幕宽度是360dp，Galaxy Nexus属于xhdpi，屏幕宽度是384dp，Nexus 6 属于xxxhdpi，屏幕宽度是410dp。所以说，光Google自己一家的产品就已经有这么多的标准，而且屏幕宽度和像素密度没有任何关联关系，即使我们使用dp，在320dp宽度的设备和410dp的设备上，还是会有90dp的差别。当然，我们尽量使用match_parent和wrap_content，尽可能少的用dp来指定控件的具体长宽，再结合上权重，大部分的情况我们都是可以做到适配的。

### 根据屏幕的配置来加载合适的UI布局

  布局自适应屏幕是通过__伸缩控件__(match parent)来适应各种不同屏幕大小的布局，屏幕大小变化很大时，用户体验可能就不佳。应用程序应该不仅仅实现了可自适应的布局，还应该提供一些方案根据屏幕的配置来加载不同的布局，可以通过配置限定符(configuration qualifiers)来实现。__配置限定符允许程序在运行时根据当前设备的配置自动加载合适的资源(比如为不同尺寸屏幕设计不同的布局)。__

  __使用Size限定符__

 使用Size限定符，不同屏幕大小的设备可以自动加载不同的layout。

 现在有很多的应用程序为了支持大屏设备，都会实现“two pane”模式(程序会在左侧的面板上展示一个包含子项的List，在右侧面板上展示内容)。平板和电视设备的屏幕都很大，足够同时显示两个面板，而手机屏幕一次只能显示一个面板，两个面板需要分开显示。所以，为了实现这种布局，我们需要定义以下文件：res/layout/main.xml，single-pane(默认)布局和res/layout-large/main.xml，two-pane（双面板）布局。第二个布局的目录名中包含了large限定符，那些被定义为大屏的设备(比如7寸以上的平板)会自动加载此布局，而小屏设备会加载另一个默认的布局。

### 确保正确的布局应用在正确的设备屏幕上

 __使用Smallest-width限定符__

 使用Size限定符有一个问题会让很多程序员感到头疼，large到底是指多大呢？__很多应用程序都希望能够更自由地为不同屏幕设备加载不同的布局__，不管它们是不是被系统认定为"large"。这就是Android为什么在3.2以后引入了"Smallest-width"限定符。Smallest-width限定符允许你设定一个具体的最小值(以dp为单位)来指定屏幕。例如，7寸的平板最小宽度是600dp，所以如果你想让你的UI在这种屏幕上显示two pane，在更小的屏幕上显示single pane，你可以使用sw600dp来表示你想在600dp以上宽度的屏幕上使用two pane模式。然而，使用早于Android 3.2系统的设备将无法识别sw600dp这个限定符，所以你还是同时需要使用large限定符。


 __使用布局别名__

 Smallest-width限定符仅在Android 3.2及之后的系统中有效。因而，你也需要同时使用Size限定符(small, normal, large和xlarge)来兼容更早的系统。例如，你想手机上显示single-pane界面，而在7寸平板和更大屏的设备上显示multi-pane界面，你需要提供以下文件：

* res/layout/main.xml: single-pane布局
* res/layout-large: multi-pane布局
* res/layout-sw600dp: multi-pane布局


最后的两个文件是完全相同的，为了要解决这种重复，你需要使用别名技巧。例如，你可以定义以下布局：

* res/layout/main.xml, single-pane布局
* res/layout/main_twopanes.xml, two-pane布局

加入以下两个文件：

* res/values-large/layout.xml:

<resources>
    <item name="main" type="layout">@layout/main_twopanes</item>
</resources>

* res/values-sw600dp/layout.xml:

<resources>
    <item name="main" type="layout">@layout/main_twopanes</item>
</resources>


最后两个文件有着相同的内容，但是它们并没有真正去定义布局，__它们仅仅只是给main定义了一个别名main_twopanes__。这样两个layout.xml都只是引用了@layout/main_twopanes，就避免了重复定义布局文件的情况。

 __使用Orientation限定符__

  有些布局会在横屏和竖屏的情况下都显示的很好，但是多数情况下这些布局都可以再调整的。我们可以针对布局在不同屏幕尺寸和不同屏幕方向中做不同的调整.


### 提供可以根据屏幕大小自动伸缩的图片

 __使用Nine-Patch图片__


 支持不同屏幕大小通常情况下也意味着，你的图片资源也需要有自适应的能力。例如，一个按钮的背景图片必须能够随着按钮大小的改变而改变。如果你想使用普通的图片来实现上述功能，你很快就会发现结果是令人失望的，因为运行时会均匀地拉伸或压缩你的图片。解决方案是使用nine-patch图片，它是一种被特殊处理过的PNG图片，你可以指定哪些区域可以拉伸而哪些区域不可以。

## 几种高效的适配方案

### 宽高限定符适配

  为了高效的实现UI开发，出现了新的适配方案，我把它称作宽高限定符适配。简单说，就是穷举市面上所有的Android手机的宽高像素值。设定一个基准的分辨率，其他分辨率都根据这个基准分辨率来计算，在不同的尺寸文件夹内部，根据该尺寸编写对应的dimens文件。比如以480x320为基准分辨率

  宽度为320，将任何分辨率的宽度整分为320份，取值为x1-x320

  高度为480，将任何分辨率的高度整分为480份，取值为y1-y480

  这个时候，如果我们的UI设计界面使用的就是基准分辨率，那么我们就可以按照设计稿上的尺寸填写相对应的dimens引用了,而当APP运行在不同分辨率的手机中时，这些系统会根据这些dimens引用去该分辨率的文件夹下面寻找对应的值。这样基本解决了我们的适配问题，而且极大的提升了我们UI开发的效率。但是这个方案有一个致命的缺陷，那就是需要精准命中才能适配，比如1920x1080的手机就一定要找到1920x1080的限定符，否则就只能用统一的默认的dimens文件了。而使用默认的尺寸的话，UI就很可能变形，简单说，就是容错机制很差。


### 今日头条适配方案

  今日头条的适配方式，今日头条适配方案默认项目中只能以高或宽中的一个作为基准，进行适配。因为大部分市面上的 Android 设备的屏幕高宽比都不一致，特别是现在大量全面屏的问世，这个问题更加严重，不同厂商推出的全面屏手机的屏幕高宽比都可能不一致，这时只以高或宽其中的一个作为基准进行适配，就会有效的避免布局在高宽比不一致的屏幕上出现变形的问题。

#### 方案分析

  假设我们UI设计图是按屏幕宽度为360dp来设计的，如果在屏幕宽度为1080/(440/160)=392.7dp的设备上。使用dp是无法在不同设备上显示为同样效果的。同时还存在部分设备屏幕宽度不足360dp，这时就会导致按360dp宽度来开发实际显示不全的情况。
 
  dp和px的转换公式 ：px = dp * density 

  density = dpi/160

  可以看出，如果设计图宽为360dp，__想要保证在所有设备计算得出的px值都正好是屏幕宽度的话，我们只能修改 density 的值。__

  density 在每个设备上都是固定的，DPI / 160 = density，__屏幕的总 px 宽度 / density = 屏幕的总 dp 宽度。__
  
  假设有如下两个设备：

  设备 1，屏幕宽度为 1080px，480DPI，屏幕总 dp 宽度为 1080 / (480 / 160) = 360dp


  设备 2，屏幕宽度为 1440，560DPI，屏幕总 dp 宽度为 1440 / (560 / 160) = 411dp


  可以看到__屏幕的总 dp 宽度在不同的设备上是会变化的，但是我们在布局中填写的 dp 值却是固定不变的__。

  这会导致什么呢？假设我们布局中有一个 View 的宽度为 100dp，在设备 1 中 该 View 的宽度占整个屏幕宽度的 27.8% (100 / 360 = 0.278)
但在设备 2 中该 View 的宽度就只能占整个屏幕宽度的 24.3% (100 / 411 = 0.243)，可以看到这个 View 在像素越高的屏幕上，dp 值虽然没变，但是与屏幕的实际比例却发生了较大的变化，所以肉眼的观看效果，会越来越小，这就导致了传统的填写 dp 的屏幕适配方式产生了较大的误差。

  如果每个 View 的 dp 值是固定不变的，那我们只要保证每个设备的屏幕总 dp 宽度不变，就能保证每个 View 在所有分辨率的屏幕上与屏幕的比例都保持不变，从而完成等比例适配，并且这个屏幕总 dp 宽度如果还能保证和设计图的宽度一致的话，那我们在布局时就可以直接按照设计图上的尺寸填写 dp 值。屏幕的总 px 宽度 / density = 屏幕的总 dp 宽度
在这个公式中我们要保证 屏幕的总 dp 宽度 和 设计图总宽度 一致，并且在所有分辨率的屏幕上都保持不变，我们需要怎么做呢？屏幕的总 px 宽度 每个设备都不一致，这个值是肯定会变化的。

这时今日头条的公式就派上用场了：当前设备屏幕总宽度（单位为像素）/  设计图总宽度（单位为 dp) = density

即：__在不同尺寸和分辨率的设备上，根据设计图的总宽度，强行修改density值，来完成屏幕的适配。__

这个公式就是把上面公式中的 屏幕的总 dp 宽度 换成 设计图总宽度，原理都是一样的，只要 density 根据不同的设备进行实时计算并作出改变，就能保证 设计图总宽度 不变，也就完成了适配。

#### 如何修改density值？

  density 是 DisplayMetrics 中的成员变量。获取方式如下：getResources().getDisplayMetrics().density。

  先来熟悉下 DisplayMetrics 中和适配相关的几个变量：

  DisplayMetrics#density 就是上述的density

  DisplayMetrics#densityDpi 就是上述的dpi

  DisplayMetrics#scaledDensity 字体的缩放因子，正常情况下和density相等，但是调节系统字体大小后会改变这个值。

  大家都知道，不管你在布局文件中填写的是什么单位，最后都会被转化为 px，系统就是通过上面的方法，将你在项目中任何地方填写的单位都转换为 px 的。

  首先来看看布局文件中dp的转换，最终都是调用 TypedValue#applyDimension(int unit, float value, DisplayMetrics metrics) 来进行转换：

    public static float applyDimension(int unit, float value,
                                       DisplayMetrics metrics)
    {
        switch (unit) {
        case COMPLEX_UNIT_PX:
            return value;
        case COMPLEX_UNIT_DIP:
            return value * metrics.density;
        case COMPLEX_UNIT_SP:
            return value * metrics.scaledDensity;
        case COMPLEX_UNIT_PT:
            return value * metrics.xdpi * (1.0f/72);
        case COMPLEX_UNIT_IN:
            return value * metrics.xdpi;
        case COMPLEX_UNIT_MM:
            return value * metrics.xdpi * (1.0f/25.4f);
        }
        return 0;
    }

这里用到的DisplayMetrics正是从Resources中获得的。

再看看图片的decode，BitmapFactory#decodeResourceStream方法:

    public static Bitmap decodeResourceStream(@Nullable Resources res, @Nullable TypedValue value,
            @Nullable InputStream is, @Nullable Rect pad, @Nullable Options opts) {
        validate(opts);
        if (opts == null) {
            opts = new Options();
        }

        if (opts.inDensity == 0 && value != null) {
            final int density = value.density;
            if (density == TypedValue.DENSITY_DEFAULT) {
                opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;
            } else if (density != TypedValue.DENSITY_NONE) {
                opts.inDensity = density;
            }
        }
        
        if (opts.inTargetDensity == 0 && res != null) {
            opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
        }
        
        return decodeStream(is, pad, opts);
    }

可见也是通过 DisplayMetrics 中的值来计算的。

当然还有些其他dp转换的场景，基本都是通过 DisplayMetrics 来计算的，这里不再详述。因此，想要满足上述需求，我们只需要修改 DisplayMetrics 中和 dp 转换相关的变量即可。

下面假设设计图宽度是360dp，以宽维度来适配。

那么适配后的 density = 设备真实宽(单位px)/360，接下来只需要把我们计算好的 density 在系统中修改下即可。通用代码实现如下：

    public static void setCustomDensity(@Nullable Activity activity, @Nullable Application application,int baseDp){
        final DisplayMetrics appDisplayMetrics = application.getResources().getDisplayMetrics();
        final float targetDensity = appDisplayMetrics.widthPixels / baseDp;
        final int targetDensityDpi = (int)(160*targetDensity);
        appDisplayMetrics.density = appDisplayMetrics.scaledDensity = targetDensity;
        appDisplayMetrics.densityDpi = targetDensityDpi;

        final DisplayMetrics activityDisplayMetrics = activity.getResources().getDisplayMetrics();
        activityDisplayMetrics.density = activityDisplayMetrics.scaledDensity = targetDensity;
        activityDisplayMetrics.densityDpi = targetDensityDpi;
    }


同时在 Activity#onCreate 方法中调用下。代码比较简单，也没有涉及到系统非公开api的调用，因此理论上不会影响app稳定性。


#### 优缺点

##### 优点

* 使用成本非常低，操作非常简单，使用该方案后在页面布局时不需要额外的代码和操作，这点可以说完虐其他屏幕适配方案


* 侵入性非常低，该方案和项目完全解耦，在项目布局时不会依赖哪怕一行该方案的代码，而且使用的还是 Android 官方的 API，意味着当你遇到什么问题无法解决，想切换为其他屏幕适配方案时，基本不需要更改之前的代码，整个切换过程几乎在瞬间完成，会少很多麻烦，节约很多时间，试错成本接近于 0


* 可适配三方库的控件和系统的控件(不止是是 Activity 和 Fragment，Dialog、Toast 等所有系统控件都可以适配)，由于修改的 density 在整个项目中是全局的，所以只要一次修改，项目中的所有地方都会受益


* 不会有任何性能的损耗

##### 缺点

  只需要修改一次 density，项目中的所有地方都会自动适配，这个看似解放了双手，减少了很多操作，但是实际上反应了一个缺点，那就是只能一刀切的将整个项目进行适配，但适配范围是不可控的
这样不是很好吗？这样本来是很好的，但是应用到这个方案是就不好了，因为我上面的原理也分析了，这个方案依赖于设计图尺寸，但是项目中的系统控件、三方库控件、等非我们项目自身设计的控件，它们的设计图尺寸并不会和我们项目自身的设计图尺寸一样。
当这个适配方案不分类型，将所有控件都强行使用我们项目自身的设计图尺寸进行适配时，这时就会出现问题，当某个系统控件或三方库控件的设计图尺寸和和我们项目自身的设计图尺寸差距非常大时，这个问题就越严重


### smallestWidth适配

   系统会根据当前设备屏幕的 最小宽度 来匹配 values-sw<N>dp，为什么不是根据 宽度 来匹配，而要加上 最小 这两个字呢？

   这就要说到，移动设备都是允许屏幕可以旋转的，当屏幕旋转时，屏幕的高宽就会互换，加上 最小 这两个字，是因为这个方案是不区分屏幕方向的，它只会把屏幕的高度和宽度中值最小的一方认为是 最小宽度，这个 最小宽度 是根据屏幕来定的，是固定不变的，意思是不管您怎么旋转屏幕，只要这个屏幕的高度大于宽度，那系统就只会认定宽度的值为 最小宽度，反之如果屏幕的宽度大于高度，那系统就会认定屏幕的高度的值为 最小宽度。


#### smallestWidth 的值是怎么算的？

   假设设备的屏幕信息是 1920 * 1080、480 dpi

   根据上面的规则我们要在屏幕的高度和宽度中选择值最小的一方作为最小宽度，1080 < 1920，明显 1080 px 就是我们要找的 最小宽度 的值， 但 最小宽度 的单位是 dp，所以我们要把 px 转换为 dp 下面的公式一定不能再忘了！ px / density = dp，DPI / 160 = density，所以最终的公式是 px / (DPI / 160) = dp 现在我们已经算出了当前设备的最小宽度是（1080/(480/160)） 360 dp，我们晓得系统会根据这个 最小宽度 帮助我们匹配到 values-sw360dp 文件夹下的 dimens.xml 文件，如果项目中没有 values-sw360dp 这个文件夹，系统才会去匹配相近的 values-sw<N>dp 文件夹。dimens.xml 文件是整个方案的核心所在，所以接下来我们再来看看 values-sw360dp 文件夹中的这个 dimens.xml 是根据什么原理生成的。

#### 如何生成dimens.xml

  因为我们在项目布局中引用的 dimens 的实际值，来源于根据当前设备屏幕的 最小宽度 所匹配的 values-sw<N>dp 文件夹中的 dimens.xml，所以搞清楚 dimens.xml 的生成原理，有助于我们理解 smallestWidth 限定符屏幕适配方案。
说到 dimens.xml 的生成，就要涉及到两个因数：

  * 第一个因素是 __最小宽度基准值__

  * 第二个因素就是您的项目需要适配哪些 最小宽度，通俗理解就是需要生成多少个 values-sw<N>dp 文件夹


__第一个因素__

  最小宽度基准值 是什么意思呢？简单理解就是您需要把设备的屏幕宽度分为多少份，假设我们现在把项目的 最小宽度基准值 定为 360，那这个方案就会理解为您想把所有设备的屏幕宽度都分为 360 份，方案会帮您在 dimens.xml 文件中生成 1 到 360 的 dimens 引用。values-sw360dp 指的是当前设备屏幕的 最小宽度 为 360dp (该设备高度大于宽度，则最小宽度就是宽度，所以该设备宽度为 360dp)，把屏幕宽度分为 360 份，刚好每份等于 1dp，所以每个引用都递增 1dp，值最大的 dimens 引用 dp_360 值也是 360dp，刚好覆盖屏幕宽度。

  说到这里，那大家就应该就会明白我为什么会说 smallestWidth 限定符屏幕适配方案 的原理也同样是按百分比进行布局，如果在布局中，一个 View 的宽度引用 dp_100，那不管运行到哪个设备上，这个 View 的宽度都是当前设备屏幕总宽度的 360分之100，前提是项目提供有当前设备屏幕对应的 values-sw<N>dp，如果没有对应的 values-sw<N>dp，就会去寻找相近的 values-sw<N>dp，这时就会存在误差了，至于误差是大是小，这就要看您的第二个因数怎么分配了
其实 smallestWidth 限定符屏幕适配方案 的原理和 今日头条屏幕适配方案 挺像的，__今日头条屏幕适配方案 是根据屏幕的宽度或高度动态调整每个设备的 density (每 dp 占当前设备屏幕多少像素)，而 smallestWidth 限定符屏幕适配方案 同样是根据屏幕的宽度动态调整每个设备 每份占的 dp 值。__


__第二个因素__

   第二个因数是需要适配哪些 最小宽度？比如您想适配的 最小宽度 有 320dp、360dp、400dp、411dp、480dp，那方案就会为您的项目生成 values-sw320dp、values-sw360dp、values-sw400dp、values-sw411dp、values-sw480dp 这几个资源文件夹。方案会为您需要适配的 最小宽度，在项目中生成一系列对应的 values-sw<N>dp，在前面也说了，如果某个设备没有为它提供对应的 values-sw<N>dp，那它就会去寻找相近的 values-sw<N>dp，但如果这个相近的 values-sw<N>dp 与期望的 values-sw<N>dp 差距太大，那适配效果也就会大打折扣。

 那是不是 values-sw<N>dp 文件夹生成的越多，覆盖越多市面上的设备，就越好呢？

 也不是，因为每个 values-sw<N>dp 文件夹其实都会占用一定的 App 体积，values-sw<N>dp 文件夹越多，App 的体积也就会越大。所以一定要合理分配 values-sw<N>dp，以越少的 values-sw<N>dp 文件夹，覆盖越多的机型。

#### 优缺点

##### 优点

* 非常稳定，极低概率出现意外

* 不会有任何性能的损耗

* 适配范围可自由控制，不会影响其他三方库


##### 缺点

* 在布局中引用 dimens 的方式，虽然学习成本低，但是在日常维护修改时较麻烦

* 侵入性高，如果项目想切换为其他屏幕适配方案，因为每个 Layout 文件中都存在有大量 dimens 的引用，这时修改起来工作量非常巨大，切换成本非常高昂

* 无法覆盖全部机型，想覆盖更多机型的做法就是生成更多的资源文件，但这样会增加 App 体积，在没有覆盖的机型上还会出现一定的误差，所以有时需要在适配效果和占用空间上做一些抉择


### AndroidAutoSize

[AndroidAutoSize]（https://juejin.im/post/5bce688e6fb9a05cf715d1c2）

[AndroidAutoSize Demo](https://github.com/JessYanCoding/AndroidAutoSize)


# 参考

[Android官方提供的支持不同屏幕大小的全部方法](https://blog.csdn.net/guolin_blog/article/details/8830286)

[Android屏幕适配全攻略(最权威的官方适配指导)](https://blog.csdn.net/zhaokaiqiang1992/article/details/45419023)

[Android 目前稳定高效的UI适配方案](https://mp.weixin.qq.com/s/X-aL2vb4uEhqnLzU5wjc4Q)

[一种极低成本的Android屏幕适配方式](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMjE0MQ==&mid=2247484502&idx=2&sn=a60ea223de4171dd2022bc2c71e09351&scene=21#wechat_redirect)

[屏幕兼容性概览](https://developer.android.com/guide/practices/screens_support)

[今日头条屏幕适配方案终极版正式发布!](https://juejin.im/post/5bce688e6fb9a05cf715d1c2)

[SmallestWidth 限定符适配方案](https://juejin.im/post/5ba197e46fb9a05d0b142c62)

[今日头条适配方案](https://juejin.im/post/5b7a29736fb9a019d53e7ee2)

[一种非常好用的Android屏幕适配](https://www.jianshu.com/p/1302ad5a4b04)

