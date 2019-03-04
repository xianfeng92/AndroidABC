## Activity的生命周期

### 典型情况下的生命周期

* onCreate：表示Activity正在被创建，这是生命周期的第一个方法，在这个方法中，我们可以做一些初始化的工作，比如调用onContentView去加载界面布局资源，
  初始化Activity所需数据等

* onRestart:表示Activity正在重新启动，一般情况下，当当前Activity从不可见重新变为可见时，onRestart就会被调用，这种情况一般是用户行为所导致的，
  比如用户按home键切换到桌面或者用户打开了一个新的Activity，这时当前的Activity就会被暂停，也就是onPause和onStop方法被执行了，
  接着用户又回到了这个Activity，就会出现这种情况


* onStart:表示Activity正在被启动，即将开始，这个时候Activity已经可见了，但是还没有出现在前台（还在后台），还无法和用户交互，
  这个时候我们可以理解为Activity已经启动了，但是我们还没有看见


* onResume:表示Activity已经可见了，并且出现在前台，并开始活动了，要注意这个和onStart的对比，这两个都表示Activity已经可见了，
  但是onStart的时候Activity还处于后台，onResume的时候Activity才显示到前台


* onPause:表示Activity正在停止，正常情况下，紧接着onStop就会被调用，在特殊情况下，如果这个时候再快速的回到当前Activity，
  那么onResume就会被调用，主席的理解是这个情况比较极端，用户操作很难重现这个场景，此时可以做一些数据存储，停止动画等工作，但是注意不要太耗时了，
  因为这样会影响到新的Activity的显示，onPause必须先执行完，新Activity的onResume才会执行

* onStop:表示Activity即将停止，同样可以做一些轻量级的资源回收，但是不要太耗时了

* onDestroy:表示Activity即将被销毁，这是Activity生命周期的最后一个回调，在这里我们可以做一些最后的回收工作和资源释放


正常情况下，Activity的常用生命周期用官网的一张图就足够表示了:




注意：假设当前Activity为A，如果用户打开了一个新的Activity为B，那么B的onResume是在A的onPause后执行的。所以为了保证Activity切换的流畅性，
      A的onPause里不能做耗时的操作，可以放在onDestroy中执行。还有一点，我们知道，在Activity A时，当用户按home键切换到桌面时，Activity A的 onPause和onStop
      会被执行，当再次又回到Activity A时，A的 onStart和onResume会被调用。


### 异常情况下的生命周期

* 资源相关的系统配置发生改变导致Activity被杀死并重新创建

 理解这个问题，首先要对系统的资源加载有一定的了解，拿最简单的图片来说，当我们把一张图片放在drawable中的时候，就可以通过 Resources 去获取这张图片了，同时为了兼容不同的设备，
我们可能还需要在其他一些目录下放置不同的图片，比如drawable-xhdpi之类的，当应用程序启动时，系统会根据当前设备的情况去加载合适的Resources资源，比如说横屏手机和竖屏手机会拿着两张不同的图片
（设定了landscape或者portrait状态下的图片），比如之前Activity处于竖屏，我们突然旋转屏幕，由于系统配置发生了改变，在默认情况下，Activity会被销毁并且重新创建。

当系统配置发生改变的时候，Activity会被销毁，即被onDestroy掉，同时由于Activity是异常情况下终止的，系统会调用 onSaveInstanceState 来保存当前Activity的状态，这个方法调用的时机是在onStop之前，
他和onPause没有既定的时序关系，需要强调的是，这个方法只出现在Activity被异常终止的情况下，正常情况下是不会走这个方法的，当我们onSaveInstanceState保存到Bundler对象作为参数传递给 
onRestoreInstanceState 和onCreate方法，因此我们可以通过onRestoreInstanceState和onCreate方法来判断Activity是否被重建。如果被重建了，我们就取出之前的数据恢复，从时序上来说，
onRestoreInstanceState的调用时机应该在onStart之后。

  和Activity一样，每一个View都有 onSaveInstanceState 和 onRestoreInstanceState 这两个方法，看一下他们的实现，就能知道系统能够为每一个View恢复数据关于保存和恢复View的层次结构，
系统的工作流程是这样的：首先Activity被意外终止时，Activity会调用onSaveInstanceState去保存数据，然后Activity会委托Window去保存数据，接着Window再委托上面的顶级容器去保存数据，顶级容器是一个ViewGroup，
一般来说他可能是一个DecorView,最后顶层容器再去一一通知他的子元素来保存数据，这样整个数据保存过程就完成了，可以发现，这是一种典型的委托思想，上层委托下层，父容器委托子容器，去处理一件事件。


### 资源内存不足导致低优先级的Activity被杀死

Activity按照优先级的从高往低，可以分为三种：


1. 前台Activity:正在和用户交互的Activity，优先级最高

2. 可见但非前台Activity:比如对话框，此Activity可见但是位于后台无法和用户直接交互

3. 后台Activity:已经被暂停的Activity，比如执行了onStop，优先级最低


当系统内存不足的时候，系统就会按照上述优先级去杀死目标Activity所在的进程，并且在后续通过onSaveInstanceState和onRestoreInstanceState来存储和恢复数据，如果一个进程中没有四大组件在执行，
那么这个进程将很快被系统杀死，因此，一些后台工作不适合脱离四大组建而独立运行在后台中，这样进程很容易就被杀死了，比较好的方法就是将后台工作放在Service中从而保证了进程有一定的优先级。


系统配置中有很多内容，如果当某项内容发生改变后，我们不想系统重新创建，就可以给configChangs属性加上orientation这个值

```
android:configChanges="orientation"
```


## Activity的启动模式

首先说一下Activity为什么需要启动模式，我们知道，在默认的情况下，当我们多次启动同一个Activity的时候，系统会创建多个实例并把他们一一放入任务栈中，
当我们点击back键的时候会发现这些Activity会一一回退，任务栈是一种先进先出的栈结构，这个好理解， 每按一次back键就有一个activity退出栈，知道栈空为止，
当这个栈为空的时候，系统就会回收这个任务栈。知道了Activity的启动模式，我们可发现一个问题：多次启动同一个Activity会创建多个实例，这样是不是很逗，
Activity在设计的时候不可能不考虑到这个问题，所以他提供了启动模式来修改系统的默认行为，目前有四种启动模式：

### standard

标准模式，这也是系统的默认模式，每次启动一个Activity都会重新创建一个实例，是否这个实例已经存在，被创建的实例的生命周期符合典型情况下Activity的生命周期，
如上述：onCreate(),onStart();onResume()都会被调用，这是一种典型的多实例实现，一个任务栈都可以有多个实例，每个实例都可以属于不同的任务栈，在这种模式下，
__谁启动了这个Activity，那么这个Activity就运行在启动它的Activity所在的栈内__，比如Activity A启动了Activity B（B是标准模式），那么B就会进入到A所在的栈内，
当我们用ApplicationContext去启动standard模式的Activity的时候就会报错,这是因为 ApplicationContext 并没有所谓的任务栈。

### singleTop

栈顶复用模式，在这个模式下，__如果新的Activity已经位于任务栈的栈顶，那么此Activity不会被重新创建，同时他的onNewIntent方法会被调用__，
通过此方法的参数我们可以取出当前请求的信息，需要注意的是，这个Activity的onCreate,onStart不会被系统调用，因为他并没有发生改变，
__如果新Activity已存在但不是在栈顶，那么新Activity则会重新创建__，举个例子，假设现在栈内的情况为ABCD，其中ABCD为四个Activity，A位于栈底，
D位于栈顶，这个时候假设要再启动D，如果D的启动模式为singleTop，那么站栈内的情况仍然是ABCD，如果D的启动模式是standard，那么由于D会被重新创建，导致情况就是ABCDD

### singleTask

栈内复用模式，这是一种单实例模式，在这种模式下，__只要Activity在一个栈内存在，那么多次启动此Activity都不会创建实例，和singTop一样，系统也会回调其onNewIntent方法__，
具体一点，当一个具有singleTask模式的Activity请求启动后，比如 Activity A，__系统首先会去寻找是否存在A想要的任务栈（如何知道Activity想要哪个任务栈？见PS)__，
如果不存在，就创建一个任务栈，然后创建A的实例把A放进栈中。
如果存在A所需要的栈，这个时候就要看A是否在栈中有实例存在，如果实例存在，那么系统就会把A调到栈顶并调用它的onNewIntent方法，如果实例不存在，就创建A的实例并且把A压入栈中。


* 比如目前任务栈S1中的情况为ABC，这个时候Activity D以singleTask模式请求启动，其所需的任务栈为S2，由于S2和D的实例都不存在，所以系统会先创建任务栈S2，然后创建D的实例将其入栈到S2

* 另外一种情况，假设D所需的任务栈为S1，其他情况如如上面的一样，那么由于S1已经存在，所以系统会直接创建D的实例并将其引入到S1中

* 如果D所需要的任务栈为S1，并且当前任务栈S1的情况为ABCD，根据栈内复用的原则，此时D不会被重新创建，系统会把D切换到栈顶并且调用其oNnNewIntent方法，
  同时由于singleTask默认具有clearTop的效果，会导致栈内所有在D上面的Activity全部出栈，于是最终S1中的情况为AD

### singleInstance

单实例模式，这是一种加强的singleTask的模式，他除了具有singleTask的所有属性之外，还加强了一点，那就是__此模式下的Activity只能单独的处于一个任务栈中__，
换句话说，比如Activity A是singleInstance模式，当A启动的时候，系统会为创建创建一个新的任务栈，然后A独立在这个任务栈中，由于栈内复用的特性，
后续的请求均不会创建新的Activity，除非这个独特的任务栈被系统销毁了。



PS:

在singkleTask启动模式中，多次提到了某个Activity所需的任务栈，什么是Activity所需的任务栈尼？这要从一个参数说起：TaskAffinity（任务相关性）。
这个参数标示了__一个Activity所需要的任务栈的名字__。默认情况下，所有的Activity所需要的任务栈的名字为应用的包名。
当然，我们可以为每个Activity都单独指定 TaskAffinity，这个属性值必须必须不能和包名相同，否则就相当于没有指定。
TaskAffinity属性主要和 singleTask 启动模式或者 allowTaskReparenting属性配合使用，另外，__任务栈分为前台任务栈和后台任务栈__，
后台任务栈中的Activity位于暂停状态，用户可以通过切换将后台任务栈再次调为前台。


## Activity 的 Flags

* FLAG_ ACTIVITY_ NEW _ TASK

  这个标志位的作用是为Activity指向‘singleTask’启动模式，其效果和XML中指定该模式相同

* FLAG_ ACTIVITY_ SINGLE _ TOP

  这个标志位的作用是为Activity指向‘singleTop’启动模式，其效果和XML中指定该模式相同

* FLAG_ ACTIVITY_ CLEAR _ TOP

  具有此标记位的Activity,当他启动时，在同一个任务栈中所有位于他上面的Activity都要出栈，这个模式一般需要和FLAG_ ACTIVITY_ NEW _ TASK配合使用，
  在这种情况下，被启动的Activity的实例如果已经存在，那么系统就会调用它的onNewIntent,如果被启动的Activity采用标准模式，那么他连同他之上的Activity都要出栈，
  系统会创建新的Activity实例并放入栈顶

* FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

  具有此标记位的Activity，不会出现在历史Activity的列表当中，当某种情况下我们不希望用户通过历史列表回到我们的 Activity 的时候就使用这个标记位了。
  他等同于在XML中指定Activity的属性：

   ```
   android:excludeFromRecents="true"
   ```





















































