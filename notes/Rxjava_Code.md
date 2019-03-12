
# 一个简单的例子

```
        Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("Android");
                emitter.onNext("ios");
                emitter.onNext("Other");
                emitter.onComplete();
            }
        })
                .subscribe(new Observer<String>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        
                    }

                    @Override
                    public void onNext(String s) {
                        Log.d(TAG, "onNext: "+s);
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {

                    }
                });
```

## 从create开始

```
    public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
        ObjectHelper.requireNonNull(source, "source is null");
        return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
    }

```

返回值是 Observable,参数是 ObservableOnSubscribe ,定义如下：

```
public interface ObservableOnSubscribe<T> {
    void subscribe(ObservableEmitter<T> e) throws Exception;
}
```

ObservableOnSubscribe 是一个接口，里面就一个方法,也是我们实现的那个方法：该方法的参数是 ObservableEmitter，它是关联起 Disposable 概念的一层：

```
public interface ObservableEmitter<T> extends Emitter<T> {
    void setDisposable(Disposable d);
    void setCancellable(Cancellable c);
    boolean isDisposed();
    ObservableEmitter<T> serialize();
}
```

ObservableEmitter 也是一个接口。里面方法很多，它继承了 Emitter<T> 接口。

```
public interface Emitter<T> {
    void onNext(T value);
    void onError(Throwable error);
    void onComplete();
}
```

Emitter<T>定义了我们在 ObservableOnSubscribe 中实现 subscribe() 方法里最常用的三个方法。


ObservableCreate 算是一种适配器的体现，create()需要返回的是 Observable,而我现在有的是（即 方法传入的参数）ObservableOnSubscribe 对象，ObservableCreate 
将 ObservableOnSubscribe 适配成 Observable。 其中 subscribeActual()方法表示的是被订阅时真正被执行的方法。

```
public final class ObservableCreate<T> extends Observable<T> {
    final ObservableOnSubscribe<T> source;

    public ObservableCreate(ObservableOnSubscribe<T> source) {
        this.source = source;
    }

    @Override
    protected void subscribeActual(Observer<? super T> observer) {
        CreateEmitter<T> parent = new CreateEmitter<T>(observer);
        observer.onSubscribe(parent);

        try {
            source.subscribe(parent);
        } catch (Throwable ex) {
            Exceptions.throwIfFatal(ex);
            parent.onError(ex);
        }
    }
```

OK,至此，创建流程结束，我们得到了 Observable<T> 对象，其实就是 ObservableCreate<T>。

## 到 subscribe 结束

   上面在代码在 create，返回 ObservableCreate 对象，然后调用该对象的 subscribe 完成订阅，我们知道 ObservableCreate 继承 Observable，
当调用 subscribe 方法时，会调 Observable 的 subscribe 方法：

```
    public final void subscribe(Observer<? super T> observer) {
            ...
             // 真正的订阅处
            subscribeActual(observer);
            ...
    }

```

上面代码的 subscribeActual 调用的是 ObservableCreate 中的方法。

```
    @Override
    protected void subscribeActual(Observer<? super T> observer) {
        //1 创建 CreateEmitter，也是一个适配器，可以将 Observer -> Disposable，CreateEmitter 中主要持有 observer 对象的引用，并且维护了 dispose 变量。
        CreateEmitter<T> parent = new CreateEmitter<T>(observer);
        //2 onSubscribe（）参数是 Disposable。还有一点要注意的是 onSubscribe() 是在我们执行 subscribe() 这句代码的那个线程回调的，并不受线程调度影响。
        // 给 observer 的一个回调，告诉它是否 dispose
        observer.onSubscribe(parent);

        try {
            //3 将 ObservableOnSubscribe（源头）与 CreateEmitter（Observer，终点）联系起来，即完成订阅，此时 ObservableOnSubscribe 会向 observer 传送事件
            source.subscribe(parent);
        } catch (Throwable ex) {
            Exceptions.throwIfFatal(ex);
            parent.onError(ex);
        }
    }

```

source 即 ObservableOnSubscribe 对象，在本文中是：

```
new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("Android");
                emitter.onNext("ios");
                emitter.onNext("Other");
                emitter.onComplete();
            }
        }
```

ObservableOnSubscribe#subscribe 中会调用 parent.onNext() 和 parent.onComplete()，parent 是 CreateEmitter 对象，如下：

```

        @Override
        public void onNext(T t) {
            if (t == null) {
                onError(new NullPointerException("onNext called with null. Null values are generally not allowed in 2.x operators and sources."));
                return;
            }
           //如果没有被 dispose，会调用 Observer 的 onNext()方法
            if (!isDisposed()) {
                observer.onNext(t);
            }
        }

        @Override
        public void onError(Throwable t) {
            if (!tryOnError(t)) {
                RxJavaPlugins.onError(t);
            }
        }

        @Override
        public void onComplete() {
            if (!isDisposed()) {
                try {
                    observer.onComplete();
                } finally {
                    dispose();
                }
            }
        }
```

## 总结

1. Observable 和 Observer 的关系没有被 dispose，才会回调 Observer 的 onXXXX()方法

2. Observer 的 onComplete() 和 onError() 互斥只能执行一次，因为 CreateEmitter 在回调他们两中任意一个后，都会自动dispose()

3. Observable 和 Observer关联时（订阅时），Observable 才会开始发送数据

4. ObservableCreate 将 ObservableOnSubscribe(真正的源)->Observable

5. ObservableOnSubscribe(真正的源)需要的是发射器 ObservableEmitter

6. CreateEmitter 将 Observer->ObservableEmitter,同时它也是 Disposable

7. source.subscribe(parent)
   这句代码执行时，才开始从发送 ObservableOnSubscribe 中利用 ObservableEmitter 发送数据给 Observer。即数据是从源头 push 给终点。


## map操作符


```
        Observable.create(new ObservableOnSubscribe<String>() { // return ObservableCreate

            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("Android");
                emitter.onNext("ios");
                emitter.onNext("Other");
                emitter.onComplete();
            }
        })
                .map(new Function<String, String>() { // return ObservableMap, 并且ObservableMap持有对 ObservableSubscribeOn 的引用
                    @Override
                    public String apply(String s) {
                        return s+s;
                    }
                })
                .subscribe(new Observer<String>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        
                    }

                    @Override
                    public void onNext(String s) {
                        Log.d(TAG, "onNext: "+s);
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {

                    }
                });

```

我们看一下map函数的源码：

```
    public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
        ObjectHelper.requireNonNull(mapper, "mapper is null");
        return RxJavaPlugins.onAssembly(new ObservableMap<T, R>(this, mapper));
    }
```

```
public final class ObservableMap<T, U> extends AbstractObservableWithUpstream<T, U> {
    final Function<? super T, ? extends U> function;

    public ObservableMap(ObservableSource<T> source, Function<? super T, ? extends U> function) {
        // super()将上游的Observable保存起来 ，用于subscribeActual()中用。
        super(source);
        // 将function变换函数类保存起来
        this.function = function;
    }

    @Override
    public void subscribeActual(Observer<? super U> t) {
        source.subscribe(new MapObserver<T, U>(t, function));
    }

```

ObservableMap 继承自 AbstractObservableWithUpstream,该类继承自 Observable，很简单，就是将上游的 ObservableSource 保存起来，做一次 wrapper，
所以它也算是装饰者模式的体现，如下:

```
abstract class AbstractObservableWithUpstream<T, U> extends Observable<U> implements HasUpstreamObservableSource<T> {

    // 将上游的 ObservableSource 保存起来
    protected final ObservableSource<T> source;

    AbstractObservableWithUpstream(ObservableSource<T> source) {
        this.source = source;
    }

    @Override
    public final ObservableSource<T> source() {
        return source;
    }

}
```

关于 ObservableSource，代表了一个标准的无背压的源数据接口，可以被 Observer 消费（订阅），如下：

```
public interface ObservableSource<T> {
    void subscribe(Observer<? super T> observer);
}
```

所有的 Observable 都已经实现了它,所以我们可以认为 Observable 和 ObservableSource 是相等的：

```
public abstract class Observable<T> implements ObservableSource<T> {
```

所以我们得到的 ObservableMap 对象也很简单，就是将上游的 Observable 和变换函数类Function保存起来。 
Function的定义超级简单，就是一个接口，给我一个T，还你一个R。

```
public interface Function<T, R> {
    R apply(T t) throws Exception;
}
```

subscribeActual()是订阅真正发生的地方，就是用 MapObserver 订阅上游 Observable。

```
    @Override
    public void subscribeActual(Observer<? super U> t) {
    //用 MapObserver 订阅上游 Observable。
        source.subscribe(new MapObserver<T, U>(t, function));
    }
```

MapObserver 也是装饰者模式，对终点（下游）Observer修饰。

```
    static final class MapObserver<T, U> extends BasicFuseableObserver<T, U> {
        final Function<? super T, ? extends U> mapper;

        MapObserver(Observer<? super U> actual, Function<? super T, ? extends U> mapper) {
            // super()将actual保存起来
            super(actual);
           // 保存Function变量
            this.mapper = mapper;
        }

        @Override
        public void onNext(T t) {
            //done在onError 和 onComplete以后才会是true，默认这里是false，所以跳过
            if (done) {
                return;
            }
            //默认sourceMode是0，所以跳过
            if (sourceMode != NONE) {
                downstream.onNext(null);
                return;
            }

            U v;

            try {
                //这一步执行变换,将上游传过来的 T，利用 Function 转换成下游需要的 V
                v = ObjectHelper.requireNonNull(mapper.apply(t), "The mapper function returned a null value.");
            } catch (Throwable ex) {
                fail(ex);
                return;
            }
           //变换后传递给下游Observer
            downstream.onNext(v);
        }

        @Override
        public int requestFusion(int mode) {
            return transitiveBoundaryFusion(mode);
        }

        @Nullable
        @Override
        public U poll() throws Exception {
            T t = qd.poll();
            return t != null ? ObjectHelper.<U>requireNonNull(mapper.apply(t), "The mapper function returned a null value.") : null;
        }
    }
```

订阅的过程，是从下游到上游依次订阅的:

* 即终点 Observer 订阅了 map 返回的 ObservableMap

* 然后 map 的 Observable(ObservableMap)在被订阅时，会订阅其内部保存上游 Observable，用于订阅上游的 Observer 是一个装饰者(MapObserver)，
  内部保存了下游（本例是终点）Observer，以便上游发送数据过来时，能传递给下游。

* 以此类推，直到源头 Observable 被订阅，它开始向 Observer 发送数据。数据传递的过程，当然是从上游push到下游的，

* 源头 Observable 传递数据给下游 Observer（本例就是MapObserver）,然后MapObserver接收到数据，对其变换操作后(实际的function在这一步执行)，
  再调用内部保存的下游 Observer 的 onNext() 发送数据给下游,以此类推，直到终点 Observer 。


## 线程调度subscribeOn


```
        Observable.create(new ObservableOnSubscribe<String>() { // return ObservableCreate

            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("Android");
                emitter.onNext("ios");
                emitter.onNext("Other");
                emitter.onComplete();
            }
        })
               // 在 Observable 和 Observer 之间增加了一句线程调度代码
                .subscribeOn(Schedulers.newThread()) 
                .subscribe(new Observer<String>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        
                    }

                    @Override
                    public void onNext(String s) {
                        Log.d(TAG, "onNext: "+s);
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {

                    }
                });

```

查看subscribeOn()源码：

```
    @SchedulerSupport(SchedulerSupport.CUSTOM)
    public final Observable<T> subscribeOn(Scheduler scheduler) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
    }
```

ObservableSubscribeOn 类也是一个包装类（装饰者），点进去查看：

```
public final class ObservableSubscribeOn<T> extends AbstractObservableWithUpstream<T, T> {
    // 保存线程调度器
    final Scheduler scheduler;

    public ObservableSubscribeOn(ObservableSource<T> source, Scheduler scheduler) {
   // 保存ObservableSource
        super(source);
        this.scheduler = scheduler;
    }

    @Override
    public void subscribeActual(final Observer<? super T> observer) {
        // 创建一个包装Observer
        final SubscribeOnObserver<T> parent = new SubscribeOnObserver<T>(observer);
         // 调用 下游（终点）Observer.onSubscribe()方法,所以onSubscribe()方法执行在 订阅处所在的线程
        observer.onSubscribe(parent);
        // setDisposable()是为了将子线程的操作加入Disposable管理中
        parent.setDisposable(scheduler.scheduleDirect(new SubscribeTask(parent)));
    }

    final class SubscribeTask implements Runnable {
        private final SubscribeOnObserver<T> parent;

        SubscribeTask(SubscribeOnObserver<T> parent) {
            this.parent = parent;
        }

        @Override
        public void run() {
            // 此时已经运行在相应的 Scheduler 的线程中
            source.subscribe(parent);
        }
    }
```

ObservableSubscribeOn自身同样是个包装类，同样继承AbstractObservableWithUpstream。创建了一个SubscribeOnObserver类，该类按照套路，
应该也是实现了Observer、Disposable接口的包装类，让我们看一下：
```
    static final class SubscribeOnObserver<T> extends AtomicReference<Disposable> implements Observer<T>, Disposable {
        // 真正的下游（终点）观察者
        final Observer<? super T> downstream;
        // 用于保存上游的Disposable，以便在自身dispose时，连同上游一起dispose
        final AtomicReference<Disposable> upstream;

        SubscribeOnObserver(Observer<? super T> downstream) {
            this.downstream = downstream;
            this.upstream = new AtomicReference<Disposable>();
        }
        // onSubscribe()方法由上游调用，传入 Disposable,在本类中赋值给this.upstream，加入管理。
        @Override
        public void onSubscribe(Disposable d) {
            DisposableHelper.setOnce(this.upstream, d);
        }
        // 直接调用下游观察者的对应方法
        @Override
        public void onNext(T t) {
            downstream.onNext(t);
        }

        @Override
        public void onError(Throwable t) {
            downstream.onError(t);
        }

        @Override
        public void onComplete() {
            downstream.onComplete();
        }
        // 取消订阅时，连同上游Disposable一起取消
        @Override
        public void dispose() {
            DisposableHelper.dispose(upstream);
            DisposableHelper.dispose(this);
        }

        @Override
        public boolean isDisposed() {
            return DisposableHelper.isDisposed(get());
        }
        // 这个方法在subscribeActual()中被手动调用，为了将Schedulers返回的Worker加入管理
        void setDisposable(Disposable d) {
            DisposableHelper.setOnce(this, d);
        }
    }
```

其他都好理解，稍微不好理解的应该是下面两句代码：

```
 //ObservableSubscribeOn类
        // setDisposable()是为了将子线程的操作加入Disposable管理中
        parent.setDisposable(scheduler.scheduleDirect(new Runnable() {
            @Override
            public void run() {
            // 此时已经运行在相应的Scheduler 的线程中
                source.subscribe(parent);
            }
        }));

        // SubscribeOnObserver类
        // 这个方法在 subscribeActual()中被手动调用，为了将Schedulers返回的Worker加入管理
        void setDisposable(Disposable d) {
            DisposableHelper.setOnce(this, d);
        }
```

其中 scheduler.scheduleDirect(new Runnable()..)方法源码如下：

```
    @NonNull
    public Disposable scheduleDirect(@NonNull Runnable run) {
        return scheduleDirect(run, 0L, TimeUnit.NANOSECONDS);
    }

```
从注释和方法名我们可以看出，这个传入的Runnable会立刻执行。再继续往里面看：

```
    @NonNull
    public Disposable scheduleDirect(@NonNull Runnable run, long delay, @NonNull TimeUnit unit) {
        //class Worker implements Disposable ，Worker本身是实现了Disposable
        final Worker w = createWorker();

        final Runnable decoratedRun = RxJavaPlugins.onSchedule(run);

        DisposeTask task = new DisposeTask(decoratedRun, w);
        //开始在Worker的线程执行任务，
        w.schedule(task, delay, unit);

        return task;
    }
```

我们总结一下subscribeOn(Schedulers.xxx())的过程：

* 返回一个 ObservableSubscribeOn 包装类对象

* 上一步返回的对象被订阅时，回调该类中的 subscribeActual() 方法，在其中会立刻将线程切换到对应的 Schedulers.xxx() 线程

* 在切换后的线程中，执行 source.subscribe(parent)，对上游(终点)Observable订阅

* 上游(终点) Observable 开始发送数据，上游发送数据仅仅是调用下游观察者对应的 onXXX() 方法而已，所以此时操作是在切换后的线程中进行


subscribeOn(Schedulers.xxx())切换线程N次，总是以第一次为准，或者说离源 Observable 最近的那次为准。

为什么？ 

* 订阅流程从下游往上游传递

* 在subscribeActual()里开启了 Scheduler 的工作，source.subscribe(parent)。从这一句开始切换了线程，所以在这之上的代码都是在切换后的线程里的了

* 但如果连续切换，最上面的切换最晚执行,此时线程变成了最上面的 subscribeOn(xxxx) 指定的线程

* 而数据 push 时，是从上游到下游的，所以会在离源头最近的那次 subscribeOn(xxxx) 的线程里 push 数据（onXXX()）给下游



## 线程调度 observeOn

增加一个 observeOn(AndroidSchedulers.mainThread()),就完成了观察者线程的切换。

```
                .subscribeOn(Schedulers.newThread())
                .map(new Function<String, String>() {
                    @Override
                    public String apply(String s) {
                        return s+s;
                    }
                })
                .observeOn(Schedulers.io())
```


```
public final class ObservableObserveOn<T> extends AbstractObservableWithUpstream<T, T> {\
   //本例是 AndroidSchedulers.mainThread()
    final Scheduler scheduler;
    //默认false
    final boolean delayError;
    //默认128
    final int bufferSize;
    public ObservableObserveOn(ObservableSource<T> source, Scheduler scheduler, boolean delayError, int bufferSize) {
        super(source);
        this.scheduler = scheduler;
        this.delayError = delayError;
        this.bufferSize = bufferSize;
    }

    @Override
    protected void subscribeActual(Observer<? super T> observer) {
        if (scheduler instanceof TrampolineScheduler) {
            source.subscribe(observer);
        } else {
            // 创建出一个 主线程的Worker
            Scheduler.Worker w = scheduler.createWorker();
           // 订阅上游数据源， 
            source.subscribe(new ObserveOnObserver<T>(observer, w, delayError, bufferSize));
        }
    }
```

本例中，就是两步：

创建一个AndroidSchedulers.mainThread()对应的Worker

用ObserveOnObserver订阅上游数据源。这样当数据从上游push下来，会由 ObserveOnObserver 对应的onXXX()处理。

```
 static final class ObserveOnObserver<T> extends BasicIntQueueDisposable<T>
    implements Observer<T>, Runnable {
        // 下游的观察者
        final Observer<? super T> downstream;
        // 对应Scheduler里的Worker
        final Scheduler.Worker worker;
        final boolean delayError;
        final int bufferSize;
        // 上游被观察者 push 过来的数据都存在这里
        SimpleQueue<T> queue;

        Disposable upstream;
        // 如果onError了，保存对应的异常
        Throwable error;
        //是否完成
        volatile boolean done;
        //是否取消
        volatile boolean disposed;
        // 代表同步发送 异步发送 
        int sourceMode;

        boolean outputFused;

        ObserveOnObserver(Observer<? super T> actual, Scheduler.Worker worker, boolean delayError, int bufferSize) {
            this.downstream = actual;
            this.worker = worker;
            this.delayError = delayError;
            this.bufferSize = bufferSize;
        }

        @Override
        public void onSubscribe(Disposable d) {
            if (DisposableHelper.validate(this.upstream, d)) {
                this.upstream = d;
               //创建一个queue 用于保存上游 onNext() push的数据
                queue = new SpscLinkedArrayQueue<T>(bufferSize);
               //回调下游观察者onSubscribe方法
                downstream.onSubscribe(this);
            }
        }

        @Override
        public void onNext(T t) {
            // 执行过error / complete 会是true
            if (done) {
                return;
            }
            // 如果数据源类型不是异步的， 默认不是
            if (sourceMode != QueueDisposable.ASYNC) {
           // 将上游push过来的数据 加入 queue里
                queue.offer(t);
            }
          // 开始进入对应 Workder 线程，在线程里将queue里的 t 取出发送给下游Observer
            schedule();
        }

        @Override
        public void onError(Throwable t) {
            if (done) {
                RxJavaPlugins.onError(t);
                return;
            }
            error = t;
            done = true;
            //开始调度
            schedule();
        }

        @Override
        public void onComplete() {
            if (done) {
                return;
            }
            done = true;
            //开始调度
            schedule();
        }

        @Override
        public void dispose() {
            if (!disposed) {
                disposed = true;
                upstream.dispose();
                worker.dispose();
                if (getAndIncrement() == 0) {
                    queue.clear();
                }
            }
        }

        @Override
        public boolean isDisposed() {
            return disposed;
        }

        void schedule() {
            if (getAndIncrement() == 0) {
                //该方法需要传入一个线程， 注意看本类实现了Runnable的接口，所以查看对应的run()方法
                worker.schedule(this);
            }
        }
 

         //从这里开始，这个方法已经是在Workder对应的线程里执行的了
        @Override
        public void run() {
            if (outputFused) {
                drainFused();
            } else {
            //取出queue里的数据发送
                drainNormal();
            }
        }

        boolean checkTerminated(boolean d, boolean empty, Observer<? super T> a) {
            if (disposed) {
                queue.clear();
                return true;
            }
            if (d) {
                Throwable e = error;
                if (delayError) {
                    if (empty) {
                        disposed = true;
                        if (e != null) {
                            a.onError(e);
                        } else {
                            a.onComplete();
                        }
                        worker.dispose();
                        return true;
                    }
                } else {
                    if (e != null) {
                        disposed = true;
                        queue.clear();
                        a.onError(e);
                        worker.dispose();
                        return true;
                    } else
                    if (empty) {
                        disposed = true;
                        a.onComplete();
                        worker.dispose();
                        return true;
                    }
                }
            }
            return false;
        }
    }

```

* ObserveOnObserver实现了Observer和Runnable接口。

* 在onNext()里，先不切换线程，将数据加入队列queue。然后开始切换线程，在另一线程中，从queue里取出数据，push给下游Observer

* 所以observeOn()影响的是其下游的代码，且多次调用仍然生效。因为其切换线程代码是在Observer里onXXX()做的，这是一个主动的push行为（影响下游）。

关于多次调用生效问题。对比 subscribeOn() 切换线程是在 subscribeActual()里做的，只是主动切换了上游的订阅线程，从而影响其发射数据时所在的线程。而直到真正发射数据之前，任何改变线程的行为，都会生效（影响发射数据的线程）。所以subscribeOn()只生效一次。observeOn()是一个主动的行为，并且切换线程后会立刻发送数据，所以会生效多次。


### 线程调度subscribeOn()：

* 内部先切换线程，在切换后的线程中对上游Observable进行订阅，这样上游发送数据时就是处于被切换后的线程里了。也因此多次切换线程，最后一次切换（离源数据最近）的生效。

* 内部订阅者接收到数据后，直接发送给下游Observer,引入内部订阅者是为了控制线程（dispose）

* 线程切换发生在Observable中


### 线程调度observeOn()：

* 使用装饰的 Observer 对上游 Observable 进行订阅

* 在Observer中onXXX()方法里，将待发送数据存入队列，同时请求切换线程处理真正 push 数据给下游

* 多次切换线程，都会对下游生效


