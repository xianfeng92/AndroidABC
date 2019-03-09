
Android系统应用程序一般是由多个Activity组成，而这些Activity是以视图的形式展现在我们面前，视图都是由一个一个的组件构成的。组件就是我们常见的Button、ImageView等等。

那么我们平时看到的Android手机中那些漂亮的界面是怎么显示出来的呢？

这就要用到Android的布局管理器了，网上有人比喻的很好：布局好比是建筑里的框架，组件按照布局的要求依次排列，就组成了漂亮的界面。

  在分析布局之前，首先看看控件：Android 中任何可视化的控件都是从 android.veiw.View 继承而来的，系统提供了两种方法来设置视图：

1. 使用XML文件来配置View的相关属性，然后在程序启动时系统根据配置文件来创建相应的View视图

   每个Android项目的源码目录下都有个res/layout目录，这个目录就是用来存放布局文件的。布局文件一般以对应的 activity 的名字命名，以 .xml 为后缀。在xml中创建组件时需要为组件指定id,如: android:id="@+id/名字".系统会自动在 gen 目录下创建相应的R资源类变量。在代码中创建每个 Activity 时，
   一般是在onCreate()方法中，调用 setContentView() 来加载指定的 xml 布局文件，然后就可以通过 findViewById() 来获得在布局文件中创建的相应id的控件了，
   如Button等。

```
private Button btnSndMag;
public void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.main);	// 加载main.xml布局文件
	btnSndMag = (Button)this.findViewById(R.id.btnSndMag); // 通过id找到对于的Button组件
	....
}
```


2. 在代码中直接使用相应的类来创建视图。


## Android系统中为我们提供的五大布局：

1. LinearLayout(线性布局)

   线性布局是按照水平或垂直的顺序将子元素(可以是控件或布局)依次按照顺序排列，每一个元素都位于前面一个元素之后。线性布局分为两种：水平方向和垂直方向的布局。
   android:layout_weight 表示子元素占据的空间大小的比例


2. RelativeLayout(相对布局)
   
   RelativeLayout继承于android.widget.ViewGroup，其按照子元素之间的位置关系完成布局的，作为Android系统五大布局中最灵活也是最常用的一种布局方式，
   非常适合于一些比较复杂的界面设计。

   注意：在引用其他子元素之前，引用的 ID 必须已经存在，否则将出现异常。

   常用的位置属性:

```
android:layout_toLeftOf 	 	该组件位于引用组件的左方
android:layout_toRightOf 		该组件位于引用组件的右方
android:layout_above 			该组件位于引用组件的上方
android:layout_below 		    	该组件位于引用组件的下方
android:layout_alignParentLeft  	该组件是否对齐父组件的左端
android:layout_alignParentRight 	该组件是否齐其父组件的右端
android:layout_alignParentTop   	该组件是否对齐父组件的顶部
android:layout_alignParentBottom  	该组件是否对齐父组件的底部
android:layout_centerInParent 	  	该组件是否相对于父组件居中
android:layout_centerHorizontal   	该组件是否横向居中
android:layout_centerVertical 	  	该组件是否垂直居中
```


3. FrameLayout(单帧布局)
  
   将所有的子元素放在整个界面的左上角，后面的子元素直接覆盖前面的子元素。


4. AbsoluteLayout(绝对布局)

   绝对布局中将所有的子元素通过设置 android:layout_x 和 android:layout_y属性，将子元素的坐标位置固定下来，即坐标(android:layout_x, android:layout_y) ，layout_x用来表示横坐标，layout_y用来表示纵坐标。屏幕左上角为坐标(0,0)，横向往右为正方，纵向往下为正方。实际应用中，这种布局用的比较少，因为 Android 终端一般机型比较多，各自的屏幕大小。分辨率等可能都不一样，如果用绝对布局，可能导致在有的终端上显示不全等。


5. TablelLayout(表格布局)



## 布局属性

![五种布局的公有属性](https://github.com/xianfeng92/AndroidABC/blob/master/images/public_Layout_attribute.png)

![](https://github.com/xianfeng92/AndroidABC/blob/master/images/specific_Layout_attribute.png)
