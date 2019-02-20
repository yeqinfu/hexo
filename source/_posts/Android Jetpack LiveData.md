---
title: Android Jetpack LiveData ViewModel
date: 2019-02-20 14:00:58
tags: android
---

```java
 // Other code to setup the activity...
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

这边需要搞懂，

+ 为什么点击按钮的时候，nameObserver的方法会得到回调，然后更新了界面的内容。
+ 还有就是界面销毁后这个对象怎么销毁的。
+ 多个fragment共享数据问题

```java
 @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull FragmentActivity activity,
            @Nullable Factory factory) {
        Application application = checkApplication(activity);
        if (factory == null) {
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        return new ViewModelProvider(ViewModelStores.of(activity), factory);
    }
```

ViewModelProvider是一个单例

```java
 @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        ViewModel viewModel = mViewModelStore.get(key);

        if (modelClass.isInstance(viewModel)) {
            //noinspection unchecked
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }

        viewModel = mFactory.create(modelClass);
        mViewModelStore.put(key, viewModel);
        //noinspection unchecked
        return (T) viewModel;
    }
```

这里进行指定类的对象创建。并且有存储起来。

nameObserver创建一个监听字符串的监听器，接收变动回调。

再看

```
 model.currentName.observe(this, nameObserver)
```

把当前的currentName进行和监听器关联。MutableLiveData就是LiveData，看observe方法

```java
 LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        owner.getLifecycle().addObserver(wrapper);
```



LifecycleBoundObserver这个对我们监听器进行了一层封装。然后加入到mObservers列表中,可以看到这个监听器实例伴随的宿主只能一个，不然抛出异常。

然后我们看下setValue的执行流程

```
private void considerNotify(ObserverWrapper observer) {
        if (!observer.mActive) {
            return;
        }
        // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
        //
        // we still first check observer.active to keep it as the entrance for events. So even if
        // the observer moved to an active state, if we've not received that event, we better not
        // notify for a more predictable notification order.
        if (!observer.shouldBeActive()) {
            observer.activeStateChanged(false);
            return;
        }
        if (observer.mLastVersion >= mVersion) {
            return;
        }
        observer.mLastVersion = mVersion;
        //noinspection unchecked
        observer.mObserver.onChanged((T) mData);
    }
```

最后找到在这个方法中执行了回调更改了界面值。

#### 如果当前宿主销毁了，当前的对象和监听器会如何？

从observe方法可以看出，这边把自定义监听器添加到了宿主的生命周期之中了。所以会跟随着activity的生命周期来确定是否要移除这个监听器。

```
 owner.getLifecycle().addObserver(wrapper);
```









