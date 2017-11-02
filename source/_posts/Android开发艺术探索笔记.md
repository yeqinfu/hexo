---
title: android开发艺术探索笔记
date: 2017-11-02 14:00:58
tags: note
---



#### Activity生命周期

从整个生命周期来说，onCreate和onDestroy是配对的，分别标志着Activity的创建和销毁，并且只可能有一次调用。从Activity是否可见来说，onStart和onStop是配对的，随着用户的操作或者设备屏幕的点亮和熄灭，这两个方法可能被调用很多次；从Activity是否在前台来说，onResume和onPause是配对的，随着用户操作或者设备屏幕的额点亮和熄灭，这两个方法肯可能被调用多次。



#### 横竖屏切换

屏幕横竖屏切换导致activity的销毁和重建，onSavaInstanceState,onRestoreInstance用来销毁的数据保存和重建的现场恢复。接受的Bundle恢复工作可以在onRestoreInstance和onCreate中进行，而两个方法的区别在于，onRestoreInstanceState一旦被调用，其参数Bundle一定是有值的。

这两个方法，被调用的时机是activity异常终止才会被调用。而正常销毁并不会被调用。这里的异常终止可以是屏幕旋转，可以是因为内存不足后台的activity被回收



#### 启动模式

activity A启动了activity B，这时候B会进入A的任务栈。而有的时候我们启动activity会用ApplicationContext来启动一个activity，会报错。要加上FLAG_ACTIVITY_NEW_TASK。因为application并没有什么任务栈。



singleTop模式下，如果新的activity已经卫浴任务栈的栈顶，那么此activity不会被重建，同时它的onNewIntent方法会被回调。通过此方法参数我们可以去除当前请求的信息。如果新activity不自在栈顶，新的acitivity仍然会重建。



TaskAffinity属性主要和singleTask启动模式或者allowTaskReparenting属性配对使用。



当TashAffinity和singleTask启动模式配对使用的时候，它是具有改模式的Activity的目前任务栈的名字，待启动的activity会运行在名字和TashAffinity相同的任务栈中。



#### 启动标志FLAGS

FLAG_ACTIVITY_NEW_TASK

> 这个标记为作用是为activity指定singleTask启动模式

FLAG_ACTIVITY_SINGLE_TOP

> singleTop模式

FLAG_ACTIVITY_CLEAR_TOP

> 具有此标记为的activity当它启动时，再同一个任务栈中所有位于它上面的额activity都要出站，这个模式一般需要和FLAG_ACTIVITY_NEW_TASK配合使用，在这种情况下，被启动的activity的实例如果已经存在，那么系统就会调用它的onNewIntent。如果被启动的activity采用standard模式启动，那么它连同它之上的activity都要出栈，系统会创建新的activity实例并放入栈顶。

FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

> 具有这个编辑的activity不会出现在历史activity列表中国的，昂某些情况下我们不希望用户通过历史列表回到我们的activity的时候这个标记比较有用

#### IntentFilter的匹配规则

