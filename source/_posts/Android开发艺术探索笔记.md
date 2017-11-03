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

#### IPC通信

为四大应用主见分配进程时候。进程名字以“:”开头的进程属于当前应用的私有进程，其他应用的组件不可以和他泡在同一个进程中，而进程名不以“:”开头的进程属于全局进程。其他应用可以同过ShareUID方式和它跑在同一个进程中。

两个应用通过shareUID跑在同一个进程汇总是有要求的。需要这两个应用具有相同的ShareUID和签名才可以。他们可以互相访问对方的私有数据，data目录。组件信息。如果跑在同一个进程中。除了能共享data目录，组件信息。还可以共享内存数据，或者说他们看起来就像是一个应用的两个部分。

##### 序列化Serializable

序列化的类指定serialVersionUID，是为了反序列化的时候，系统会去检测文件中的这个值，如果一致，说明序列化的类的版本和当前类的版本是相同的。如果不一致，说明当前类和序列化的类相比发生了某些变化，比如成员变量的数量，类型可能发生了变化。这个时候是无法正常反序列化的。

如果不手动指定这个值，反序列化时，当前类所有改变的，比如增加或者珊瑚了某些成员变量，那么系统就会重新计算当前类的hash值，并且把它复制给serialVersionUID。这样导致不一致。

如果我们手动指定这个值，在类删除和增加成员变量的时候，可以最大限度回复数据。如果不指定，就直接挂了。而如果类结构发生了非常规性改变，比如修改了类名，修改了成员变量的类型。这个时候尽管serialVersionUID 尽管验证通过了，但是反序列化还是会失败。因为累结构有了毁灭性的改变。

静态成员变量补数据对象，所有不会参与序列化过程。用了transient关键字标记的成员变量不参与序列化过程。

##### Parcelable接口

内容描述功能有describeContents方法来描述，几乎在所有情况下这个方法都应该返回0，仅当当前对象中存在文件描述符时，此方法返回1。

##### Binder







