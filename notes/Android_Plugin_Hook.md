
# Android Hook技术介绍和应用

[Demo](https://github.com/xianfeng92/Android_Dy_Load)

## 什么是 Hook?
   
   Hook 英文翻译过来就是「钩子」的意思，那我们在什么时候使用这个「钩子」呢？在 Android 操作系统中系统维护着自己的一套事件分发机制。应用程序，包括应用触发事件和后台逻辑处理，也是根据事件流程一步步地向下执行。而「钩子」的意思，就是在事件传送到终点前截获并监控事件的传输，像个钩子钩上事件一样，并且能够在钩上事件时，处理一些自己特定的事件。Hook 的这个本领，使它能够将自身的代码「融入」被勾住（Hook）的程序的进程中，成为目标进程的一个部分。API Hook 技术是一种用于改变 API 执行结果的技术，能够将系统的 API 函数执行重定向。在 Android系统中使用了沙箱机制，普通用户程序的进程空间都是独立的，程序的运行互不干扰。这就使我们希望通过一个程序改变其他程序的某些行为的想法不能直接实现，但是 Hook 的出现给我们开拓了解决此类问题的道路。

## 使用 Java 反射实现 API Hook
 
   通过对 Android 平台的虚拟机注入与 Java 反射的方式，来改变 Android 虚拟机调用函数的方式（ClassLoader），从而达到 Java 函数重定向的目的，这里我们将此类操作称为 Java API Hook。


## 寻找Hook点的原则

   什么样的对象比较好Hook呢？一般来说，静态变量和单例变量是相对不容易改变，是一个比较好的hook点，而普通的对象有易变的可能，当Andorid源码版本变化时可能会不一样，处理难度比较大。
   我们根据这个原则找到所谓的Hook点。

## 寻找Hook点

  通常点击一个Button就开始Activity跳转了，这中间发生了什么，我们如何Hook,来实现Activity启动的拦截呢？

```
  public void onClick(View v) {
        Intent intent = new Intent(MainActivity.this,ProxyActivity.class);
        startActivity(intent);
    }
```

我们的目的是要拦截startActivity方法，跟踪源码，发现最后启动Activity是由Instrumentation类的execStartActivity做到的。其实这个类相当于启动Activity的中间者，启动Activity中间都是由它来操作的,源码如下:

```
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        IApplicationThread whoThread = (IApplicationThread) contextThread;
        Uri referrer = target != null ? target.onProvideReferrer() : null;
        if (referrer != null) {
            intent.putExtra(Intent.EXTRA_REFERRER, referrer);
        }
        if (mActivityMonitors != null) {
            synchronized (mSync) {
                final int N = mActivityMonitors.size();
                for (int i=0; i<N; i++) {
                    final ActivityMonitor am = mActivityMonitors.get(i);
                    ActivityResult result = null;
                    if (am.ignoreMatchingSpecificIntents()) {
                        result = am.onStartActivity(intent);
                    }
                    if (result != null) {
                        am.mHits++;
                        return result;
                    } else if (am.match(who, null, intent)) {
                        am.mHits++;
                        if (am.isBlocking()) {
                            return requestCode >= 0 ? am.getResult() : null;
                        }
                        break;
                    }
                }
            }
        }
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            int result = ActivityManager.getService()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }

```

其中 ActivityManager.getService() 获取到的是一个 AMS 的一个代理对象,用于IPC:

```
    public static IActivityManager getService() {
        return IActivityManagerSingleton.get();
    }
    private static final Singleton<IActivityManager> IActivityManagerSingleton =
            new Singleton<IActivityManager>() {
                @Override
                protected IActivityManager create() {
                    final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
                    final IActivityManager am = IActivityManager.Stub.asInterface(b);
                    return am;
                }
            };

    // Singleton helper class for lazily initialization.
	public abstract class Singleton<T> {
	    private T mInstance;

	    protected abstract T create();

	    public final T get() {
	        synchronized (this) {
	            if (mInstance == null) {
	                mInstance = create();
	            }
	            return mInstance;
	        }
	    }
	}
```


所有 Activity 启动是一个跨进程的通信的过程，所以真正启动 Activity 的是通过远端服务 ActivityManagerService 来启动的。其中 IActivityManagerSingleton 是借助 Singleton 实现的单例模式.而在内部可以看到先从 ServiceManager 中获取到AMS远端服务的 Binder 对象，然后使用 asInterface 方法转化成本地化对象，我们目的是拦截 startActivity ,所以改变 IActivityManager 对象可以做到这个一点，这里 IActivityManagerSingleton 又是静态的，根据Hook原则，这是一个比较好的Hook点。

##  Hook掉startActivity，输出日志
   
   我们先实现一个小需求，启动 Activity 的时候打印一条日志，写一个工具类HookUtil。

```
/**
 * Created By apple on 2019/3/16
 * github: https://github.com/xianfeng92
 */
public class HookUtil {
    private static final String TAG = "HookUtil";

    private Class<?> proxyActivity;
    private Context context;

    public HookUtil(Class<?> proxyActivity,Context context){
        this.proxyActivity = proxyActivity;
        this.context = context;
    }

    public void hookAMS(){

        try {
            Class<?> ActivityManager = Class.forName("android.app.ActivityManager");
            Field defaultField = ActivityManager.getDeclaredField("IActivityManagerSingleton");
            defaultField.setAccessible(true);
            // 这里返回的是一个 SingleTon 对象,接下来是获取 SingleTon 对象中的 ActivityManager
            Object defaultValue = defaultField.get(null);


            //反射 SingleTon,调用其 get 方法去获取 ActivityManager
            Class<?> SingletonClass = Class.forName("android.util.Singleton");
            Field mInstance = SingletonClass.getDeclaredField("mInstance");
            mInstance.setAccessible(true);
            // 到这里已经拿到 ActivityManager 对象,即 AMS 的一个本地化对象
            Object iActivityManagerObject = mInstance.get(defaultValue);

            //反射出一个新的 IActivityManager 对象,用其替换掉真实的 ActivityManager
            Class<?> IActivityManagerIntercept = Class.forName("android.app.IActivityManager");
            AmsInvocationHandler handler = new AmsInvocationHandler(iActivityManagerObject);
            Object proxy = Proxy.newProxyInstance(Thread.currentThread().
                    getContextClassLoader(), new Class<?>[]{IActivityManagerIntercept}, handler);
            //现在替换掉这个对象
            mInstance.set(defaultValue, proxy);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private class AmsInvocationHandler implements InvocationHandler{

        private Object iActivityManagerObject;

        public AmsInvocationHandler(Object iActivityManagerObject){
            this.iActivityManagerObject = iActivityManagerObject;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            Log.d(TAG, "invoke: "+method.getName());
            // check the method system call
            if ("startActivity".contains(method.getName())){
                Log.d(TAG, "invoke: "+"Hook到Activity已经开始启动");
            }
            return method.invoke(iActivityManagerObject,args);
        }
    }
}
```


结合注释应该很容易看懂，在 Application 中配置一下:

```
@Override
    public void onCreate() {
        super.onCreate();
        HookUtil hookUtil = new HookUtil(ProxyActivity.class,this);
        hookUtil.hookAMS();
    }
```

大功告成~~~

##  无需注册，启动Activity

  如下, SecActivity 没有在清单文件中注册，怎么去启动 SecActivity ？

```
    @Override
    public void onClick(View v) {
        Intent intent = new Intent(MainActivity.this,SecActivity.class);
        startActivity(intent);
    }

```

这个思路可以是这样，上面已经拦截了启动 Activity 流程，在 invoke 中我们可以得到启动参数intent信息，那么就在这里，我们可以自己构造一个假的Activity信息的intent，
这个Intent启动的Activity是在清单文件中注册的，当真正启动的时候（ActivityManagerService校验清单文件之后），用真实的 Intent 把代理的 Intent 在调换过来，然后启动即可。


首先获取真实启动参数intent信息

```
@Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            Log.d(TAG, "invoke: "+method.getName());
            // check the method system call
            if ("startActivity".contains(method.getName())){
                Log.d(TAG, "invoke: "+"Hook到Activity已经开始启动");
                Intent intent = null;
                int index = 0;
                for(int i = 0; i < args.length;i++){
                    Object arg = args[i];
                    if (arg instanceof Intent){
                        //说明找到了 startActivity 的Intent参数
                        intent = (Intent) args[i];
                        //这个意图是不能被启动的，因为 Acitivity 没有在清单文件中注册
                        index = i;
                        Log.d(TAG, "invoke: "+i);
                    }
                }
                //伪造一个代理的Intent，代理 Intent 启动的是 proxyActivity
                Intent proxyIntent = new Intent();
                ComponentName componentName = new ComponentName(context,proxyActivity);
                proxyIntent.setComponent(componentName);
                proxyIntent.putExtra("oldIntent",intent);
                args[index] = proxyIntent;
            }
            return method.invoke(iActivityManagerObject,args);
        }
    }

```

有了上面的两个步骤,这个代理的 Intent 是可以通过 ActivityManagerService 检验的，因为我在清单文件中注册过

```
        <activity android:name=".ProxyActivity"/>

```


为了不启动 ProxyActivity，现在我们需要找一个合适的时机，把真实的 Intent 换过来，启动我们真正想启动的 Activity。看过 Activity 的启动流程的都知道这个过程是由Handler发送消息来实现的，可是通过 Handler 处理消息的代码来看，消息的分发处理是有顺序的，下面是 Handler 处理消息的代码:

```
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```


handler处理消息的时候，首先去检查是否实现了 callback 接口，如果有实现的话，那么会直接执行接口方法，然后才是 handleMessage 方法，最后才是执行重写的handleMessage 方法，我们一般大部分时候都是重写了 handleMessage方法,而 ActivityThread 主线程用的正是重写的方法，这种方法的优先级是最低的，我们完全可以实现接口来替换掉系统 Handler 的处理过程。

```
 public void hookSystemHandler() {
        try {
            Class<?> activityThreadClass = Class.forName("android.app.ActivityThread");
            Method currentActivityThreadMethod = activityThreadClass.getDeclaredMethod("currentActivityThread");
            currentActivityThreadMethod.setAccessible(true);
            //获取主线程对象
            Object activityThread = currentActivityThreadMethod.invoke(null);
            //获取mH字段
            Field mH = activityThreadClass.getDeclaredField("mH");
            mH.setAccessible(true);
            //获取 Handler
            Handler handler = (Handler) mH.get(activityThread);
            //获取原始的mCallBack字段
            Field mCallBack = Handler.class.getDeclaredField("mCallback");
            mCallBack.setAccessible(true);
            //这里设置了我们自己实现了接口的CallBack对象
            mCallBack.set(handler, new ActivityThreadHandlerCallback(handler)) ;
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

上面的一顿操作,其实就是将源码中 final H mH = new H(), mH 替换成我们自己实现的 handler.

自定义Callback类

```
private class ActivityThreadHandlerCallback implements Handler.Callback{

        private Handler handler;

        public ActivityThreadHandlerCallback(Handler handler){
            this.handler = handler;
        }

        @Override
        public boolean handleMessage(Message msg) {
            Log.d(TAG, "handleMessage: "+msg);
            // 当为 startActivity 时,我们将 ActivityClientRecord 中的 proxyIntent 再换回 realIntent,然后在交给ActivityThread中的默认的handler来处理
            if (msg.what == 100) {
                Log.d(TAG, "handleMessage: handleLaunchActivity");
                handleLaunchActivity(msg);
            }
            handler.handleMessage(msg);
            return true;
        }

        private void handleLaunchActivity(Message msg){

            Object obj = msg.obj;// ActivityClientRecord
            try {
                Field intentField = obj.getClass().getDeclaredField("intent");
                intentField.setAccessible(true);
                Intent proxyIntent = (Intent) intentField.get(obj);
                Intent realIntent = proxyIntent.getParcelableExtra("oldIntent");
                if (realIntent != null){
                    proxyIntent.setComponent(realIntent.getComponent());
                }

            } catch (Exception e) {
                e.printStackTrace();
            }

        }
    }
```

执行，点击 MainActivity 中的按钮成功跳转到了 SecActivity.


在 Android P 中,启动 activity 的启动流程发生的变化,上述代码在Android P上运行,其实是会启动 ProxyActivity 的,即

```
if (msg.what == 100) {
                Log.d(TAG, "handleMessage: handleLaunchActivity");
                handleLaunchActivity(msg);
            }
```

没有执行到. 通过log可以发现,当我们启动一个activity时,handleMessage 中收到的 msg.what 为159,此时 msg.obj 为一个 ClientTransaction 对象,然后由 
TransactionExecutor 来接收 ClientTransaction ,完成activity的启动.



PS: Android P 中启动流程主要变化:

1. 取消了ActivityThread 里面编号110之前的Message即:

```
   public static final int LAUNCH_ACTIVITY         = 100;
        public static final int PAUSE_ACTIVITY          = 101;
        public static final int PAUSE_ACTIVITY_FINISHING= 102;
        public static final int STOP_ACTIVITY_SHOW      = 103;
        public static final int STOP_ACTIVITY_HIDE      = 104;
        public static final int SHOW_WINDOW             = 105;
        public static final int HIDE_WINDOW             = 106;
        public static final int RESUME_ACTIVITY         = 107;
        public static final int SEND_RESULT             = 108;
        public static final int DESTROY_ACTIVITY        = 109;
```


新增了:

```
        public static final int EXECUTE_TRANSACTION = 159;
```


2. ActivityThread 继承了 ClientTransactionHandler ，将原先放在 ActivityThread 里面的 handleLaunchActivity、handlerStartActivity 等等抽取出来，放到 ClientTransactionHandler 类中，作为抽象方法。

3. 新增了ClientLifecycleManager、ClientTransactionHandler、ClientTransaction、TransactionExecutor、TransactionExecutorHelper、ClientTransactionItem

* ClientLifecycleManager:作为ActivityStackSupervisor和ClientTransaction的桥梁

* ClientTransactionHandler:由ActivityThread实现

* ClientTransaction：管理ClientTransactionItem

* TransactionExecutor：执行ClientTransactionItem

* TransactionExecutorHelper：协助TransactionExecutor执行ClientTransactionItem

* ClientTransactionItem：真正执行生命周期的类，每一个生命周期对应一个Item

这样可以减轻 ActivityThread 的压力，将生命周期抽成一个具体的对象，更有利于生命周期的管理。以及降低了耦合度。


## 总结一下：

Hook 的选择点：静态变量和单例，因为一旦创建对象，它们不容易变化，非常容易定位。

Hook 过程：

* 寻找 Hook 点，原则是静态变量或者单例对象，尽量 Hook public 的对象和方法。

* 选择合适的代理方式，如果是接口可以用动态代理。

* 偷梁换柱——用代理对象替换原始对象。

* Android 的 API 版本比较多，方法和类可能不一样，所以要做好 API


# 参考

[Android插件化系列第（一）篇---Hook技术之Activity的启动过程拦截](https://www.jianshu.com/p/69bfbda302df)
[理解 Android Hook 技术以及简单实战](https://www.jianshu.com/p/4f6d20076922)

