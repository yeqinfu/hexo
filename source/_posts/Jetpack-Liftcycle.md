---
title: Jetpack-Liftcycle
date: 2020-05-06 09:00:00
tags: Jetpack
---

Jetpack全家桶适合使用在全新的项目中，算是一个趋势，这里大概学习一下它的原理

生命周期的控制一直都是很蛋疼的事情，无非就是销毁时候的异步线程持有了某个上下文对象，多次累计导致内存不足，对象泄漏没人管。

```Kotlin
  class MyObserver : LifecycleObserver {

            @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
            fun connectListener() {
            }

            @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
            fun disconnectListener() {
            }
        }

        lifecycle.addObserver(MyObserver())
```

之前可能会在activity ，fragment的resume stop加入很多逻辑，让其中看起来很臃肿。有了lifecycle之后，一切就被分离了。

> 一般我们现在都是使用java8来编译，所以相对于官方的注解模式来标注某一个生命周期，更加推荐使用重写的方式，gradle添加依赖
>
> ```
> implementation "androidx.lifecycle:lifecycle-common-java8:2.2.0"
> ```
>
> ```
> * <pre>
>  * class TestObserver implements DefaultLifecycleObserver {
>  *     {@literal @}Override
>  *     public void onCreate(LifecycleOwner owner) {
>  *         // your code
>  *     }
>  * }
>  * </pre>
> ```
>
> 得益于java8的接口支持默认空实现，注解的方式因为使用运行时注解，效率不高，不推荐使用。

## LifeCycle事件分发

这里不从源码追到源头，而是从事件分发的源头看事件如何传递到我们自定义的监听者对象MyObserver

分发的源头是ProcessLifecycleOwnerInitializer 这个类继承了ContentProvider 在onCreate函数中初始化了生命周期

```java
 @Override
    public boolean onCreate() {
        LifecycleDispatcher.init(getContext());
        ProcessLifecycleOwner.init(getContext());
        return true;
    }
```

一个生命周期分配器，进程生命周期所有者（翻译？）

LifecycleDispatcher类中可以看到，实际上在这里注册了所有的activity的生命周期回调。

```java

 static void init(Context context) {
        if (sInitialized.getAndSet(true)) {
            return;
        }
        ((Application) context.getApplicationContext())
                .registerActivityLifecycleCallbacks(new DispatcherActivityCallback());
    }
```

```java
DispatcherActivityCallback
  static class DispatcherActivityCallback extends EmptyActivityLifecycleCallbacks {

        @Override
        public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
            ReportFragment.injectIfNeededIn(activity);
        }

        @Override
        public void onActivityStopped(Activity activity) {
        }

        @Override
        public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
        }
    }
```

可以看到，生命周期的分发都交给了ReportFragment

```java
public static void injectIfNeededIn(Activity activity) {
        if (Build.VERSION.SDK_INT >= 29) {
            // On API 29+, we can register for the correct Lifecycle callbacks directly
            activity.registerActivityLifecycleCallbacks(
                    new LifecycleCallbacks());
        }
        // Prior to API 29 and to maintain compatibility with older versions of
        // ProcessLifecycleOwner (which may not be updated when lifecycle-runtime is updated and
        // need to support activities that don't extend from FragmentActivity from support lib),
        // use a framework fragment to get the correct timing of Lifecycle events
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
    }
```

这是一个没有UI的fragment，伴随着每一个activity被创建，它也被创建。

可以看到ComponentActivity其实也调用了一次

```java
 @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mSavedStateRegistryController.performRestore(savedInstanceState);
        ReportFragment.injectIfNeededIn(this);
        if (mContentLayoutId != 0) {
            setContentView(mContentLayoutId);
        }
    }
```

不过有做同步。为了防止没有继承ComponentActivity

> ComponentActivity 有两个，包名不同。都调用了

所以看ReportFragment如何分发的

```
@Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        dispatchCreate(mProcessListener);
        dispatch(Lifecycle.Event.ON_CREATE);
    }

    @Override
    public void onStart() {
        super.onStart();
        dispatchStart(mProcessListener);
        dispatch(Lifecycle.Event.ON_START);
    }

    @Override
    public void onResume() {
        super.onResume();
        dispatchResume(mProcessListener);
        dispatch(Lifecycle.Event.ON_RESUME);
    }

```

可以看到，在fragment中监听activity的生命周期，

> 疑问 因为这是一个空的fragment，它的生命周期回调能保持宿主activity的生命周期一致。

![image](https://tvax1.sinaimg.cn/large/c1b251b3gy1geirri0d5mj20lq0k3wg3.jpg)



可以看到关键方法injectIfNeededIn有两种生命周期派发的模式，第一种是SDK大于等于29的时候，直接就是往activity注册一个回调，然后最后调用dispath方法，来分发事件，第二种是当SDK<29的时候，通过Fragment的生命周期变化来派发生命周期变化的事件。

```java
 static void dispatch(@NonNull Activity activity, @NonNull Lifecycle.Event event) {
        if (activity instanceof LifecycleRegistryOwner) {
            ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
            return;
        }

        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
    }
```

调用dispatch方法，最后会拿到LifecycleRegistryOwner或者LifecycleOwner 然后调用handleLifecycleEvent方法分发事件。

```java
  private void moveToState(State next) {
        if (mState == next) {
            return;
        }
        mState = next;
        if (mHandlingEvent || mAddingObserverCounter != 0) {
            mNewEventOccurred = true;
            // we will figure out what to do on upper level.
            return;
        }
        mHandlingEvent = true;
        sync();
        mHandlingEvent = false;
    }

```

上面方法，除了去重状态，还有一个备注比较有意思。当sync被正在调用的时候，外面的生命周期又发生了改变。这边是不会继续调用sync方法而是return掉了，备注写着会在上层处理。

>也就是说，如果生命周期变化很快，可能回调会漏掉。这边的写法并且一个标志位mNewEventOccurred
>
>标志新事件已经发生了。

```java
 // happens only on the top of stack (never in reentrance),
    // so it doesn't have to take in account parents
    private void sync() {
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            throw new IllegalStateException("LifecycleOwner of this LifecycleRegistry is already"
                    + "garbage collected. It is too late to change lifecycle state.");
        }
        while (!isSynced()) {
            mNewEventOccurred = false;
            // no need to check eldest for nullability, because isSynced does it for us.
            if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                backwardPass(lifecycleOwner);
            }
            Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
            if (!mNewEventOccurred && newest != null
                    && mState.compareTo(newest.getValue().mState) > 0) {
                forwardPass(lifecycleOwner);
            }
        }
        mNewEventOccurred = false;
    }
private boolean isSynced() {
        if (mObserverMap.size() == 0) {
            return true;
        }
        State eldestObserverState = mObserverMap.eldest().getValue().mState;
        State newestObserverState = mObserverMap.newest().getValue().mState;
        return eldestObserverState == newestObserverState && mState == newestObserverState;
    }
```

sync方法，看关键部分，判断是否Synced可以看到，需要监听生命周期的自定义监听器都放在了mObserverMap中，如果长度为0代表全部同步了。然后判断最老的和最新的监听器是否状态一致，是否都是最新状态，代表着监听器全部监听完成。否者进行事件分发。

```
private void forwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
                mObserverMap.iteratorWithAdditions();
        while (ascendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                pushParentState(observer.mState);
                observer.dispatchEvent(lifecycleOwner, upEvent(observer.mState));
                popParentState();
            }
        }
    }
    static class ObserverWithState {
        State mState;
        LifecycleEventObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
            mLifecycleObserver = Lifecycling.lifecycleEventObserver(observer);
            mState = initialState;
        }

        void dispatchEvent(LifecycleOwner owner, Event event) {
            State newState = getStateAfter(event);
            mState = min(mState, newState);
            mLifecycleObserver.onStateChanged(owner, event);
            mState = newState;
        }
    }
```

最终调用了onStateChanged方法也就是LifecycleEventObserver的接口方法，这个接口被FullLifecycleObserverAdapter实现，FullLifecycleObserverAdapter被DefaultLifecycleObserver继承。

如果你采用的是注解的方法。

```java
 static class ObserverWithState {
        State mState;
        LifecycleEventObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
            mLifecycleObserver = Lifecycling.lifecycleEventObserver(observer);
            mState = initialState;
        }

        void dispatchEvent(LifecycleOwner owner, Event event) {
            State newState = getStateAfter(event);
            mState = min(mState, newState);
            mLifecycleObserver.onStateChanged(owner, event);
            mState = newState;
        }
    }
```



可以看到最后依靠的是运行时注解

```
/**
 * An internal implementation of {@link LifecycleObserver} that relies on reflection.
 */
class ReflectiveGenericLifecycleObserver implements LifecycleEventObserver {
    private final Object mWrapped;
    private final CallbackInfo mInfo;

    ReflectiveGenericLifecycleObserver(Object wrapped) {
        mWrapped = wrapped;
        mInfo = ClassesInfoCache.sInstance.getInfo(mWrapped.getClass());
    }

    @Override
    public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Event event) {
        mInfo.invokeCallbacks(source, event, mWrapped);
    }
}
```

> 效率会低一些，毕竟是反射

## 总结

生命周期兜兜转转，其实还是得靠在初始化的时候，注册每个activity的生命周期回调来进行的。

















