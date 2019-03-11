
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

1. Observable和Observer的关系没有被 dispose，才会回调 Observer 的 onXXXX()方法

2. Observer的onComplete()和onError() 互斥只能执行一次，因为 CreateEmitter 在回调他们两中任意一个后，都会自动dispose()

3. Observable和Observer关联时（订阅时），Observable才会开始发送数据

4. ObservableCreate 将 ObservableOnSubscribe(真正的源)->Observable

5. ObservableOnSubscribe(真正的源)需要的是发射器 ObservableEmitter

6. CreateEmitter 将 Observer->ObservableEmitter,同时它也是 Disposable

7. source.subscribe(parent)
   这句代码执行时，才开始从发送 ObservableOnSubscribe 中利用 ObservableEmitter 发送数据给 Observer。即数据是从源头 push 给终点。



## Observable

```
public abstract class Observable<T> implements ObservableSource<T> 

public interface ObservableSource<T> {
    /**
     * Subscribes the given Observer to this ObservableSource instance.
     * @param observer the Observer, not null
     * @throws NullPointerException if {@code observer} is null
     */
    void subscribe(@NonNull Observer<? super T> observer);
}

```

Observable 是一个抽象类，其实现了 ObservableSource 接口，而在 ObservableSource 仅仅有一个 subscribe 方法。

再来看看 Observable 中关于 subscribe 的实现：

```
    public final void subscribe(Observer<? super T> observer) {
        ObjectHelper.requireNonNull(observer, "observer is null");
        try {
            observer = RxJavaPlugins.onSubscribe(this, observer);

            ObjectHelper.requireNonNull(observer, "The RxJavaPlugins.onSubscribe hook returned a null Observer. 
            Please change the handler provided to RxJavaPlugins.setOnObservableSubscribe for invalid null returns. 
            Further reading: https://github.com/ReactiveX/RxJava/wiki/Plugins");

            subscribeActual(observer);
        } catch (NullPointerException e) {
            throw e;
        } catch (Throwable e) {
            Exceptions.throwIfFatal(e);
            // can't call onError because no way to know if a Disposable has been set or not
            // can't call onSubscribe because the call might have set a Subscription already
            RxJavaPlugins.onError(e);

            NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
            npe.initCause(e);
            throw npe;
        }
    }

```
Observable#subscribe 为 final，子类是无法重写的，在该方法除了做了一些基础判断（判空）之外，主要是调用了 subscribeActual 方法。

而在 Observable 中该方法是一个抽象方法，其由 Observable 的子类去实现。

```
protected abstract void subscribeActual(Observer<? super T> observer);

```

__所以一个被观察者被 subscribe 的逻辑其实是交由 Observable 子类来实现的，每个不同的被观察者可以根据自己的需求实现 "被订阅" 后的操作__。


## Create

Obserable对象是使用 create 方法来生成的，其源码如下：

```
    public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
        ObjectHelper.requireNonNull(source, "source is null");
        return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
    }

```
首先第一句代码是对传入的对象进行判空，内部实现是如果传入null，会抛异常。接着是生成一个 ObservableCreate 对象。

我们跟进去看一下这个 ObservableCreate 对象的一些基本信息：

```
public final class ObservableCreate<T> extends Observable<T> {
    final ObservableOnSubscribe<T> source;

    public ObservableCreate(ObservableOnSubscribe<T> source) {
        this.source = source;
    }
```

可以简单的看到，这个类继承了 Observalbe 类，并存储了我们刚才传进去的 ObservableOnSubscribe 对象。


## map

```
    public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
        ObjectHelper.requireNonNull(mapper, "mapper is null");
        return RxJavaPlugins.onAssembly(new ObservableMap<T, R>(this, mapper));
    }

```

可以看到其实是返回了一个 ObservableMap 对象，接受了两个参数，一个是 this，在这里指的也就是刚才的 ObservableCreate ，还有一个 Function 对象。


再跟进去看一下 ObservableMap 的基础信息：

```
public final class ObservableMap<T, U> extends AbstractObservableWithUpstream<T, U> {
    final Function<? super T, ? extends U> function;

    public ObservableMap(ObservableSource<T> source, Function<? super T, ? extends U> function) {
        super(source);
        this.function = function;
    }

```
可以看到其实构造方法和刚才的 ObservableCreate 一样，将传入的对象进行了存储。


AbstractObservableWithUpstream，我们再跟进看看：

```
// 带有source的 operator 的基类
abstract class AbstractObservableWithUpstream<T, U> extends Observable<U> implements HasUpstreamObservableSource<T> {

    /** The source consumable Observable. */
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
这个类是所有接受上一级输入的操作符(operator 如map)的基类，这里的逻辑并复杂，其实只是简单的封装了一下上一级的输入 source 和输出到下一级的数据。
分析之后可以看到，调用了 map 方法其实也是返回了一个 Observable 对象。


## subscribeOn & observeOn

首先看一下 subscribeOn：

```
    public final Observable<T> subscribeOn(Scheduler scheduler) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
    }
```

再看一下 ObserveOn：

```
    public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        ObjectHelper.verifyPositive(bufferSize, "bufferSize");
        return RxJavaPlugins.onAssembly(new ObservableObserveOn<T>(this, scheduler, delayError, bufferSize));
    }
```

可以看到，这里分别返回了 ObservableSubscribeOn 和 ObservableObserveOn 对象。照旧我们先看看这两个类的基础信息：


```
public final class ObservableSubscribeOn<T> extends AbstractObservableWithUpstream<T, T> {
    final Scheduler scheduler;

    public ObservableSubscribeOn(ObservableSource<T> source, Scheduler scheduler) {
        super(source);
        this.scheduler = scheduler;
    }

```




```
public final class ObservableObserveOn<T> extends AbstractObservableWithUpstream<T, T> {
    final Scheduler scheduler;
    final boolean delayError;
    final int bufferSize;
    public ObservableObserveOn(ObservableSource<T> source, Scheduler scheduler, boolean delayError, int bufferSize) {
        super(source);
        this.scheduler = scheduler;
        this.delayError = delayError;
        this.bufferSize = bufferSize;
    }

```

  同样的保存了传进去的基础信息。


其实到这里我们可以看出，上面的整个过程其实就是通过对象的嵌套，形成了一条完整的链。




## 逆向逐级订阅

### 跟踪subscribe

上面创建链的最后生成了一个 ObservableObserveOn 对象，看看 ObservableObserveOn.subscribe方法的实现:


```
    @Override
    protected void subscribeActual(Observer<? super T> observer) {
        if (scheduler instanceof TrampolineScheduler) {
            source.subscribe(observer);
        } else {
            Scheduler.Worker w = scheduler.createWorker();

            source.subscribe(new ObserveOnObserver<T>(observer, w, delayError, bufferSize));
        }
    }

```
可以看到，这里通过 Scheduler 创建了一个wroker对象，然后调用了source(上一级)的 subscribe 方法，并通过已有的 observer 对象生成一个 
ObserveOnObserver 对象作为传参。

再来看看 ObservableObserveOn 的上级 ObservableSubscribeOn 的subscribe方法的实现：

```
    @Override
    public void subscribeActual(final Observer<? super T> observer) {
        final SubscribeOnObserver<T> parent = new SubscribeOnObserver<T>(observer);
        // 先在当前线程执行订阅者的 onSubscribe 方法
        observer.onSubscribe(parent);
        // 然后在指定的线程中执行 source(上一级)的 subscribe
        parent.setDisposable(scheduler.scheduleDirect(new SubscribeTask(parent)));
    }

    
    final class SubscribeTask implements Runnable {
        private final SubscribeOnObserver<T> parent;

        SubscribeTask(SubscribeOnObserver<T> parent) {
            this.parent = parent;
        }

        @Override
        public void run() {
            source.subscribe(parent);
        }
    }

```

根据我们最开始的业务逻辑，我们这里的 scheduler 应该对应IO线程，也就是__说往下执行的 subscribe 操作都是执行再IO线程中的__。


看到这里也大概知道套路了,和刚才一样，会一直沿着整条链返回，一个一个订阅对应 Observable 并生成新的嵌套的 Observer。

那么就接着往下看。应该来到了ObservableMap.subscribe方法了:

```
    @Override
    public void subscribeActual(Observer<? super U> t) {
        source.subscribe(new MapObserver<T, U>(t, function));
    }
```


可以看到也是封装了我们传进去的 Function 参数，然后调用上一级 source.subscribe，也就是 ObservableCreate.subscribe，也就到了链的一开始。



```
    @Override
    protected void subscribeActual(Observer<? super T> observer) {
        // 首先是创建了CreateEmitter对象
        CreateEmitter<T> parent = new CreateEmitter<T>(observer);
        // 然后调用了订阅者 observer 的 onSubscribe 方法
        observer.onSubscribe(parent);

        try {
            // 这里的 source 就是我们一开始创建的 ObservableOnSubscribe 对象
            source.subscribe(parent);
        } catch (Throwable ex) {
            Exceptions.throwIfFatal(ex);
            parent.onError(ex);
        }
    }
```

终于回到了第一级，可以看到，一样的封装了 observer 订阅者，然后调用了 source.subscribe 方法，这个 source 来自我们一开始调用 Observable.create 时传进来的参数，
而 subscribe 方法就是我们一开始执行login()方法的地方。

也就是说，在刚才所有的逆序遍历过程中，下一级的 Observable 会生成的对应的 Observer 订阅上一级的 source。



## 执行任务链





# 总结

1. 创建任务链，每一步都会返回对应的 Observable 对象。

2. 逆向逐级订阅。每一步都会生成对应的 Observer 对上一步生成的 Observable 进行订阅

3. 执行任务链。执行任务链之后，每一步都会通知对应的 Observer，从而完成整个任务链的调用









