---
title: AndroidX-KTX
date: 2020-05-06 09:00:00
tags: android
---

项目中如果使用Kotlin，我们可以使用KTX这个Kotlin拓展程序包，帮助我们完成很多事情。它的主要作用就是封装了大量的拓展函数，内联函数，扩展属性，协程等。使用它，可以让kotlin代码更加简洁

![image](https://tvax3.sinaimg.cn/large/c1b251b3gy1geih4ey74xj20ea0h4mxs.jpg)

可以看到这个核心包有大量的帮助类。

## 举例

```kotlin
sharedPreferences
            .edit()  // create an Editor
            .putBoolean("key", value)
            .apply() // write to disk asynchronously 
```

我们使用Kotlin编写sp一般如上面所示。而使用KTX可以变成

```kotlin
// SharedPreferences.edit extension function signature from Android KTX - Core
    // inline fun SharedPreferences.edit(
    //         commit: Boolean = false,
    //         action: SharedPreferences.Editor.() -> Unit)

    // Commit a new value asynchronously
    sharedPreferences.edit { putBoolean("key", value) }

    // Commit a new value synchronously
    sharedPreferences.edit(commit = true) { putBoolean("key", value) }
```

看起来是简洁了一些，我们看内联函数源码

```kotlin
@SuppressLint("ApplySharedPref")
inline fun SharedPreferences.edit(
    commit: Boolean = false,
    action: SharedPreferences.Editor.() -> Unit
) {
    val editor = edit()
    action(editor)
    if (commit) {
        editor.commit()
    } else {
        editor.apply()
    }
}
```

这个内联函数，接收两个参数，第一个是可选的commit，第二个是一个无返回值的闭包函数，它在内部帮我们完成了一点点东西。

#### 运算符重载集合串联

```kotlin
// Combine 2 ArraySets into 1.
    val combinedArraySet = arraySetOf(1, 2, 3) + arraySetOf(4, 5, 6)

    // Combine with numbers to create a new sets.
    val newArraySet = combinedArraySet + 7 + 8
    
```

可以看到arraySetOf这个内联函数源码

```kotlin
/** Returns a new [ArraySet] with the specified contents. */
fun <T> arraySetOf(vararg values: T): ArraySet<T> {
    val set = ArraySet<T>(values.size)
    @Suppress("LoopToCallChain") // Causes needless copy to a list.
    for (value in values) {
        set.add(value)
    }
    return set
}
```

外部传一个多参数，返回一个ArraySet并且设置了值。同理+号被运算符重载，功能变成了两个set的合并

```Kotlin
/**
 * Returns a set containing all elements of the original set and the given [elements] collection,
 * which aren't already in this set.
 * The returned set preserves the element iteration order of the original set.
 */
public operator fun <T> Set<T>.plus(elements: Iterable<T>): Set<T> {
    val result = LinkedHashSet<T>(mapCapacity(elements.collectionSizeOrNull()?.let { this.size + it } ?: this.size * 2))
    result.addAll(this)
    result.addAll(elements)
    return result
}
/**
 * Returns a set containing all elements of the original set and then the given [element] if it isn't already in this set.
 * 
 * The returned set preserves the element iteration order of the original set.
 */
public operator fun <T> Set<T>.plus(element: T): Set<T> {
    val result = LinkedHashSet<T>(mapCapacity(size + 1))
    result.addAll(this)
    result.add(element)
    return result
}
```

new了一个对象，把this和传参进行了一个合并。

#### Lifecycle KTX

```kotlin
  lifecycleScope.launch { 
            //DO SOMETHING
        }
```

我们启动协程的时候协程销毁会伴随声明周期。

```Kotlin

/**
 * [CoroutineScope] tied to this [LifecycleOwner]'s [Lifecycle].
 *
 * This scope will be cancelled when the [Lifecycle] is destroyed.
 *
 * This scope is bound to
 * [Dispatchers.Main.immediate][kotlinx.coroutines.MainCoroutineDispatcher.immediate].
 */
val LifecycleOwner.lifecycleScope: LifecycleCoroutineScope
    get() = lifecycle.coroutineScope
```

LifecycleOwnerKT这个拓展类给LifecycleOwner就声明了一个拓展属性lifecycleScope，是LifecycleCoroutineScope类型

返回的是协程的生命周期类

```Kotlin
/**
 * [CoroutineScope] tied to this [Lifecycle].
 *
 * This scope will be cancelled when the [Lifecycle] is destroyed.
 *
 * This scope is bound to
 * [Dispatchers.Main.immediate][kotlinx.coroutines.MainCoroutineDispatcher.immediate]
 */
val Lifecycle.coroutineScope: LifecycleCoroutineScope
    get() {
        while (true) {
            val existing = mInternalScopeRef.get() as LifecycleCoroutineScopeImpl?
            if (existing != null) {
                return existing
            }
            val newScope = LifecycleCoroutineScopeImpl(
                this,
                SupervisorJob() + Dispatchers.Main.immediate
            )
            if (mInternalScopeRef.compareAndSet(null, newScope)) {
                newScope.register()
                return newScope
            }
        }
    }
```

最终的协程声明周期控制的实现类是LifecycleCoroutineScopeImpl

```Kotlin
internal class LifecycleCoroutineScopeImpl(
    override val lifecycle: Lifecycle,
    override val coroutineContext: CoroutineContext
) : LifecycleCoroutineScope(), LifecycleEventObserver {
    init {
        // in case we are initialized on a non-main thread, make a best effort check before
        // we return the scope. This is not sync but if developer is launching on a non-main
        // dispatcher, they cannot be 100% sure anyways.
        if (lifecycle.currentState == Lifecycle.State.DESTROYED) {
            coroutineContext.cancel()
        }
    }

    fun register() {
        launch(Dispatchers.Main.immediate) {
            if (lifecycle.currentState >= Lifecycle.State.INITIALIZED) {
                lifecycle.addObserver(this@LifecycleCoroutineScopeImpl)
            } else {
                coroutineContext.cancel()
            }
        }
    }

    override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
        if (lifecycle.currentState <= Lifecycle.State.DESTROYED) {
            lifecycle.removeObserver(this)
            coroutineContext.cancel()
        }
    }
}
```

看到了对DESTROYED状态进行了监听，控制了协程的取消。

拓展包中大量的拓展函数不一一举例。

## 撰写我们自己的拓展函数

#### 场景1

我们在一个编辑页面有很多editext，每个都需要进行一个判空处理。所以就会变成这样

```java
 if (TextUtils.isEmpty(iivProductName.getInputText())) {
                ToastUtil.showToast("请输入产品名称");
                return;
            }
            if (TextUtils.isEmpty(iivPurchaseNumber.getInputText())) {
                ToastUtil.showToast("请输入进货数量");
                return;
            }
            if (TextUtils.isEmpty(iivShelfLife.getInputText())) {
                ToastUtil.showToast("请输入保质期");
                return;
            }
```

如果这个页面有十个这样得输入框，你就需要十个这样的判断，十分难受。而如果使用拓展函数变成这样。

```Kotlin

    /**
     * 检查多个editext是否是空的，提示语默认使用hint
     * 不然提示输入为空，
     */
    inline fun checkEditextNoEmpty(vararg editTexts: EditText, crossinline f: () -> Unit) {
        for (item in editTexts) {
            if (TextUtils.isEmpty(item.text)) {
                var msg = item.hint.toString()
                if (TextUtils.isEmpty(msg)) {
                    item.error = "输入为空！"
                } else {
                    item.error = msg
                }
                return
            }
        }
        f.invoke()
    }
```

接收的参数是不定数量的editext,还有一个闭包。进行一个循环判空，默认使用editext的hint进行提示，如果没有使用输入为空提示。这里可以使用ditext的error方法或者使用toast提示。

那么我们的书写就变成了这样

```Kotlin
 checkEditextNoEmpty(et_product_name, et_address_street, et_unit_name, et_input_phone, et_number, et_print_number) {
            
        }
```

需要几个判断就传几个参数就可以了。然后闭包中就可以保证这些都不是空的了。

#### 其他场景

```
 fun List<String>.removeStr(str:String):String{
        var result=""
        this.forEach {
            if (str == it){

            }else{
                result+=it
            }
        }
        return result
    }
```

可以看到，这个是帮助我们list中去除某个字符串的拓展函数

```Kotlin
 /**
     * 判断两个字符串内容是不是一样得
     * 比如ABD BDA它内容一样，那就应该一样
     */
    fun String.isSameString(str:String):Boolean{
        var sorts=this.trim().toSortedSet()
        var sort2=str.trim().toSortedSet()
        if (sort2.size!=sorts.size){//长度不一样
            return false
        }else{//长度一样
            return sort2.containsAll(sorts)

        }
    }
```

这个拓展函数在我的项目中是有使用场景的。是考试做题的时候的选择顺序。拼接成字符串，然后检查两个字符串是否内容一致。

还有很多，可以根据业务场景抽象出来的拓展函数，比如判断是否登录的拓展函数等等。





























