---
title: Android LifeCycle and LiveData 
date: 2019-02-20 14:00:58
tags: android
---

平时很少上android dev很多官方框架特性都没及时跟上，这个可能有点用，看下基本原理。

#### LiveData特性

+ UI一直都保持数据同步（小学翻译水平）观察者模式使用

+ 不回内存泄漏，数据类同步宿主的生命周期

+ 当宿主销毁不会导致奔溃？（这个似乎没没意义）

+ 不需要手动处理生命周期（也没意义，包含在第一点第二点中了）

+ 总是保持数据同步更新（同上）

+ 宿主销毁可以保持立马接收到本地数据（比如旋转屏幕）

  > 这里疑问，数据保持和宿主生命周期是否有冲突

+ 分享资源（多个fragment可以使用这个来数据传递）


#### 例子

```kotlin
class NameViewModel : ViewModel() {

    // Create a LiveData with a String
    val currentName: MutableLiveData<String> by lazy {
        MutableLiveData<String>()
    }

    // Rest of the ViewModel...
}
```

```kotlin
  // Get the ViewModel.
       model = ViewModelProviders.of(this).get(NameViewModel::class.java)


        // Create the observer which updates the UI.
        val nameObserver = Observer<String> { newName ->
            // Update the UI, in this case, a TextView.
            tv_name.text = newName
        }

        // Observe the LiveData, passing in this activity as the LifecycleOwner and the observer.
        model.currentName.observe(this, nameObserver)

        btn_update.setOnClickListener {
            val anotherName = "John Doe"
            model.currentName.setValue(anotherName)
        }
```

+ livedata默认可以使用，但是ViewModelProviders会提示找不到。这是因为support包依赖了livedata

  找不到ViewModelProviders手动添加以下依赖

  ```
  api "android.arch.lifecycle:extensions:1.1.1"
  ```

  从依赖树可以看到导入情况（gradle view）

  ![image](https://wx2.sinaimg.cn/large/c1b251b3gy1g0amy96q05j20g708cjs1.jpg)

这个简单例子，可以看到tv_name一直观察着数据的变化。

#### LifeCycle基本原理

lifecycle已经加入support包内，activity,fragment直接可以获取该对象。添加对activity对象的生命周期的监听只需要

```kotlin
  lifecycle.addObserver(NameObserver())
```

之后NameObserver这个实例的相应方法会收到回调。我们以activity为例子，查看添加getLifecycle方法的地方在SupportActivity这个方法返回的是一个子类LifecycleRegistry它继承Lifecycle

![image](https://wx3.sinaimg.cn/large/c1b251b3gy1g0aops3s8yj20tq052dgd.jpg)

```java
 /**
     * Creates a new LifecycleRegistry for the given provider.
     * <p>
     * You should usually create this inside your LifecycleOwner class's constructor and hold
     * onto the same instance.
     *
     * @param provider The owner LifecycleOwner
     */
    public LifecycleRegistry(@NonNull LifecycleOwner provider) {
        mLifecycleOwner = new WeakReference<>(provider);
        mState = INITIALIZED;
    }
```

这边一个弱引用，外面的实例是一个activity，这边弱引用。

> 在我的理解中，内存泄漏通常是因为一个长生命周期的对象持有了一个短生命周期对象或者说会被结束的对象（总之就是生命周期不一致），导致短生命周期对象想要被回收的时候，得不到回收，因为长生命周期的对象持有对短生命周期对象的引用。所以如果这时候长生命周期的对象用弱引用，那么根据弱引用的特性，这个句柄就会被释放，对象会被回收。

这里用了弱引用保证切断了activity可能被mLifecycleOwner长期持有的可能，为什么可能会被长期持有呢？

因为

```java
public Lifecycle getLifecycle() {
        return this.mLifecycleRegistry;
    }
```

鬼知道开发者拿到这个对象会不会把他怎样，比如赋值给一个全局静态对象。导致泄漏

Lifecycle这个抽象类主要就是添加观察者，移除观察者还有生命周期对象的定义这几个。

```java
@Override
    public void addObserver(@NonNull LifecycleObserver observer) {
        State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
        ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
        ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);

        if (previous != null) {
            return;
        }
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            // it is null we should be destroyed. Fallback quickly
            return;
        }

        boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent;
        State targetState = calculateTargetState(observer);
        mAddingObserverCounter++;
        while ((statefulObserver.mState.compareTo(targetState) < 0
                && mObserverMap.contains(observer))) {
            pushParentState(statefulObserver.mState);
            statefulObserver.dispatchEvent(lifecycleOwner, upEvent(statefulObserver.mState));
            popParentState();
            // mState / subling may have been changed recalculate
            targetState = calculateTargetState(observer);
        }

        if (!isReentrance) {
            // we do sync only on the top level.
            sync();
        }
        mAddingObserverCounter--;
    }
```

看到添加观察者这个方法中。有一个ObserverWithState这个类包含以下这些东西

```java
static class ObserverWithState {
        State mState;
        GenericLifecycleObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
            mLifecycleObserver = Lifecycling.getCallback(observer);
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

然后看一下State这个类是干什么用的，如下

##### State class 

这边涉及到一个事件枚举，一个状态枚举

```java
 public enum Event {
        /**
         * Constant for onCreate event of the {@link LifecycleOwner}.
         */
        ON_CREATE,
        /**
         * Constant for onStart event of the {@link LifecycleOwner}.
         */
        ON_START,
        /**
         * Constant for onResume event of the {@link LifecycleOwner}.
         */
        ON_RESUME,
        /**
         * Constant for onPause event of the {@link LifecycleOwner}.
         */
        ON_PAUSE,
        /**
         * Constant for onStop event of the {@link LifecycleOwner}.
         */
        ON_STOP,
        /**
         * Constant for onDestroy event of the {@link LifecycleOwner}.
         */
        ON_DESTROY,
        /**
         * An {@link Event Event} constant that can be used to match all events.
         */
        ON_ANY
    }

```

```java
 /**
     * Lifecycle states. You can consider the states as the nodes in a graph and
     * {@link Event}s as the edges between these nodes.
     */
public enum State {
        /**
         * Destroyed state for a LifecycleOwner. After this event, this Lifecycle will not dispatch
         * any more events. For instance, for an {@link android.app.Activity}, this state is reached
         * <b>right before</b> Activity's {@link android.app.Activity#onDestroy() onDestroy} call.
         */
        DESTROYED,

        /**
         * Initialized state for a LifecycleOwner. For an {@link android.app.Activity}, this is
         * the state when it is constructed but has not received
         * {@link android.app.Activity#onCreate(android.os.Bundle) onCreate} yet.
         */
        INITIALIZED,

        /**
         * Created state for a LifecycleOwner. For an {@link android.app.Activity}, this state
         * is reached in two cases:
         * <ul>
         *     <li>after {@link android.app.Activity#onCreate(android.os.Bundle) onCreate} call;
         *     <li><b>right before</b> {@link android.app.Activity#onStop() onStop} call.
         * </ul>
         */
        CREATED,

        /**
         * Started state for a LifecycleOwner. For an {@link android.app.Activity}, this state
         * is reached in two cases:
         * <ul>
         *     <li>after {@link android.app.Activity#onStart() onStart} call;
         *     <li><b>right before</b> {@link android.app.Activity#onPause() onPause} call.
         * </ul>
         */
        STARTED,

        /**
         * Resumed state for a LifecycleOwner. For an {@link android.app.Activity}, this state
         * is reached after {@link android.app.Activity#onResume() onResume} is called.
         */
        RESUMED;

        /**
         * Compares if this State is greater or equal to the given {@code state}.
         *
         * @param state State to compare with
         * @return true if this State is greater or equal to the given {@code state}
         */
        public boolean isAtLeast(@NonNull State state) {
            return compareTo(state) >= 0;
        }
    }
```

这两个枚举的关系用官方的注释就可以很好的理解

```
 /**
     * Lifecycle states. You can consider the states as the nodes in a graph and
     * {@link Event}s as the edges between these nodes.
     */
```

我们把状态想象成一个点，把事件想象成两点之间的边如下图

![image](https://wx4.sinaimg.cn/large/c1b251b3gy1g0covxu1d5j213a0eggns.jpg)

从第一行的生命周期来映射出各个状态，我们的Lifecycle只有这几个比较关键的状态INITIALIZED，CREATED，STARTED，RESUMED，DESTROYED这几个状态从我们的生命周期上来说覆盖面积是不一样的。比如CREATED就比较长，但是当我们处于RESUMED的状态的时候，就肯定是处于STARTED，CREATED之上的状态了。

> 这几个状态的关键是，RESUMED标志着界面已经是可见的状态了,并且显示到前台可以和用户交互了
>
> STARTED这个状态标志可见但是没有到前台不能喝用户交互

所以跟随源码，我们就可以理解这个方法了

```java
static State getStateAfter(Event event) {
        switch (event) {
            case ON_CREATE:
            case ON_STOP:
                return CREATED;
            case ON_START:
            case ON_PAUSE:
                return STARTED;
            case ON_RESUME:
                return RESUMED;
            case ON_DESTROY:
                return DESTROYED;
            case ON_ANY:
                break;
        }
        throw new IllegalArgumentException("Unexpected event value " + event);
    }
```

举个例子，启动一个activity到可见可交互

```
onCreate->onStart->onResume
```

那么这个状态会从

```
CREATED->STARTED->RESUMED
```

如果又启动其它Activity那么这个activity就会再经历

```
onPause->onStop
```

那么状态

```
RESUMED->CREATED
```

可以看到这个activity回退到CREATED这个状态



##### ObserverWithState&GenericLifecycleObserver

我们回到ObserverWithState这个类中，看到这个GenericLifecycleObserver成员变量和我们传进来的observer（NameObserver）有联系。

```java
 mLifecycleObserver = Lifecycling.getCallback(observer);
```

```
 void dispatchEvent(LifecycleOwner owner, Event event) {
            State newState = getStateAfter(event);
            mState = min(mState, newState);
            mLifecycleObserver.onStateChanged(owner, event);
            mState = newState;
        }
```

肯定是等会别的类调用这个dispatchEvent然后才会回调我们自定义的NameObserver这个类中的方法。所以先看

Lifecycling里面怎么写的。

getCallback这个方法中，可以看到包含一些已经定义好的监听器Observer

```
FullLifecycleObserver
GenericLifecycleObserver
```

其实最后还是帮你把自定义的监听器（NameObserver）进行进一步的封装修饰变成GenericLifecycleObserver返回。

我们的自定义的NameObserver肯定是往下走，走到这一步

```java
  final Class<?> klass = object.getClass();
        int type = getObserverConstructorType(klass);
        if (type == GENERATED_CALLBACK) {
            List<Constructor<? extends GeneratedAdapter>> constructors =
                    sClassToAdapters.get(klass);
            if (constructors.size() == 1) {
                GeneratedAdapter generatedAdapter = createGeneratedAdapter(
                        constructors.get(0), object);
                return new SingleGeneratedAdapterObserver(generatedAdapter);
            }
            GeneratedAdapter[] adapters = new GeneratedAdapter[constructors.size()];
            for (int i = 0; i < constructors.size(); i++) {
                adapters[i] = createGeneratedAdapter(constructors.get(i), object);
            }
            return new CompositeGeneratedAdaptersObserver(adapters);
        }
```

```java
 private static int getObserverConstructorType(Class<?> klass) {
        if (sCallbackCache.containsKey(klass)) {
            return sCallbackCache.get(klass);
        }
        int type = resolveObserverCallbackType(klass);
        sCallbackCache.put(klass, type);
        return type;
    }

```

可以看到它把我们的自定义监听器通过类名进行了缓存，其中重要的两个变量

```
private static final int REFLECTIVE_CALLBACK = 1;（反射回调）//直译
private static final int GENERATED_CALLBACK = 2;（已经生成回调）
```

把我们的自定义监听器分成两类。

看这个方法怎么分类的

```java
  // anonymous class bug:35073837
        if (klass.getCanonicalName() == null) {
            return REFLECTIVE_CALLBACK;
        }
```

getCanonicalName本地类或者匿名类或者数组类型的时候getCanonicalName()方法将会返回null，从这个备注看出曾经因为这个出现了bug，如果我们的自定义监听器是一个匿名内部类把它规划为REFLECTIVE_CALLBACK

```java
 @Nullable
    private static Constructor<? extends GeneratedAdapter> generatedConstructor(Class<?> klass) {
        try {
            Package aPackage = klass.getPackage();
            String name = klass.getCanonicalName();
            final String fullPackage = aPackage != null ? aPackage.getName() : "";
            final String adapterName = getAdapterName(fullPackage.isEmpty() ? name :
                    name.substring(fullPackage.length() + 1));

            @SuppressWarnings("unchecked") final Class<? extends GeneratedAdapter> aClass =
                    (Class<? extends GeneratedAdapter>) Class.forName(
                            fullPackage.isEmpty() ? adapterName : fullPackage + "." + adapterName);
            Constructor<? extends GeneratedAdapter> constructor =
                    aClass.getDeclaredConstructor(klass);
            if (!constructor.isAccessible()) {
                constructor.setAccessible(true);
            }
            return constructor;
        } catch (ClassNotFoundException e) {
            return null;
        } catch (NoSuchMethodException e) {
            // this should not happen
            throw new RuntimeException(e);
        }
    }
```

这段代码返回的就是你传入的kclass的构造器但是为什么要重新Class.forName得到aClass然后再getDeclaredConstructor，而不是直接klass调用getDeclaredConstructor？

>其实仔细一看原来不一样
>
>它找的是com.ppandroid.androidsample.livedata.NameObserver_LifecycleAdapter
>
>所以返回null

```java
  if (constructor != null) {
            sClassToAdapters.put(klass, Collections
                    .<Constructor<? extends GeneratedAdapter>>singletonList(constructor));
            return GENERATED_CALLBACK;
        }
```

从这里看出，如果返回不为空，说明这个类（自定义监听器）的对应adapter已经被缓存了。那么返回这个监听器的类型就是

```
GENERATED_CALLBACK//已经生成的监听器
```

如果为空，也就是第一次调用就会继续往下走

hasLifecycleMethods这个方法是判断自定义监听器里面是否包含了监听器注解的方法。如果我们没写注解监听，那么会继续往下走

```java
  Class<?> superclass = klass.getSuperclass();
        List<Constructor<? extends GeneratedAdapter>> adapterConstructors = null;
        if (isLifecycleParent(superclass)) {
            if (getObserverConstructorType(superclass) == REFLECTIVE_CALLBACK) {
                return REFLECTIVE_CALLBACK;
            }
            adapterConstructors = new ArrayList<>(sClassToAdapters.get(superclass));
        }
```

如果没有，那么这段代码会去找当前监听器的父类，看看父类是否声明了注解方法。这边有一个递归寻找的逻辑。

如果递归到底还是没找到，还会继续往下走。

```java
 for (Class<?> intrface : klass.getInterfaces()) {
            if (!isLifecycleParent(intrface)) {
                continue;
            }
            if (getObserverConstructorType(intrface) == REFLECTIVE_CALLBACK) {
                return REFLECTIVE_CALLBACK;
            }
            if (adapterConstructors == null) {
                adapterConstructors = new ArrayList<>();
            }
            adapterConstructors.addAll(sClassToAdapters.get(intrface));
        }
```

这段是判断我们自定义的监听器所有实现的接口是否有注解

> 考虑得真全面啊 

如果还没找到，返回

```
REFLECTIVE_CALLBACK
```

> 总结，也就是自定义监听器，要所有接口都实现了对应的_LifecycleAdapter才会返回GENERATED_CALLBACK
>
> 不然都是REFLECTIVE_CALLBACK

##### 回退到getCallback

这下清除看到根据不同类型的callback来确定是否需要new一个新的监听器适配器，如果是已经生成的callback还需要判断是否是有实现多接口的逻辑，返回不同的对象如下

```
SingleGeneratedAdapterObserver
CompositeGeneratedAdaptersObserver
```

这些适配器都是继承GenericLifecycleObserver 实现了onStateChanged这个方法，在这个方法被调用的时候分别去反射回调我们定义的注解方法。



#### 回退addObserver

根据以上，我们可以看到我们自定义的监听器是如何被调用并且缓存起来的。

```java
        ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);

```

然后把生成的监听器缓存到mObserverMap，如果如果之前已经缓存过了，previous会不为null，而如果没有缓存过，逻辑继续往下走。

```java
 LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            // it is null we should be destroyed. Fallback quickly
            return;
        }
```

判断宿主的类（activity）是否已经销毁

```java
  boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent;
        State targetState = calculateTargetState(observer);
```

```java
 private State calculateTargetState(LifecycleObserver observer) {
        Entry<LifecycleObserver, ObserverWithState> previous = mObserverMap.ceil(observer);

        State siblingState = previous != null ? previous.getValue().mState : null;
        State parentState = !mParentStates.isEmpty() ? mParentStates.get(mParentStates.size() - 1)
                : null;
        return min(min(mState, siblingState), parentState);
    }
```

calculateTargetState这个方法可以看到它找到我们之定义的监听器之前的上一个监听器（显然第一次的话肯定为null）。siblingState这个暂定叫做兄弟监听器，也就是我们前一个监听器，所有的监听器都是链式存储，可以根据当前监听器找到上一个或者下一个。

```
min(min(mState, siblingState), parentState)
```

可以看到两次比较，比较出当前监听器和兄弟监听器和父类监听器哪个最小，也就是枚举比较。

> 父类监听器目前这里不知道哪里赋值。找到最小的然后返回

```java
while ((statefulObserver.mState.compareTo(targetState) < 0
                && mObserverMap.contains(observer))) {
            pushParentState(statefulObserver.mState);
            statefulObserver.dispatchEvent(lifecycleOwner, upEvent(statefulObserver.mState));
            popParentState();
            // mState / subling may have been changed recalculate
            targetState = calculateTargetState(observer);
        }
```

看到监听器最小监听器和当前监听器的比较，小于的时候进入循环体。在这里把他加入mParentStates的列表中。并且进行事件分发。

其实这段循环，应该是宿主（activity）如果添加了多个监听器，每个监听器可能又是多实现的监听接口（LifecycleObserver）监听接口，事件分发的顺序。

#### 事件分发

事件分发的入口是ProcessLifecycleOwnerInitializer这个类

LifecycleDispatcher

```
 static void init(Context context) {
        if (sInitialized.getAndSet(true)) {
            return;
        }
        ((Application) context.getApplicationContext())
                .registerActivityLifecycleCallbacks(new DispatcherActivityCallback());
    }
```

注册之后可以监听到每个新建的activity的生命周期，然后分发事件。

fragment同理

```
registerFragmentLifecycleCallbacks
```



#### 总结

lifecycle是可以提供生命周期的回调功能的一个类。



















