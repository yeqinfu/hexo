---
title: Kotlin委托模式
date: 2018-01-23 14:00:58
tags: code
---



设计模式之代理模式（委托模式）

Kotlin中委托实现关键字by



```Kotlin
 interface ISports {
        fun doSports()

    }
    class SwimForSports: ISports{
        override fun doSports() {
            println("do swim")
        }
    }


    class SportsManager(sport: ISports): ISports by sport

    fun main() {
        val swimSports= SwimForSports()
        var m=SportsManager(swimSports)
        m.doSports()// Log：do swim
        Log.d("yeqinfu","-------"+(m is ISports))
    }
//echo yeqinfu: -------true

```

实现了ISports接口，在形式上可以理解为SportsManager 返回值是一个ISports 类型，实际由sport 对象代理



### lazy

延迟委托

```Kotlin
public fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)
```

查看Lazy接口

```kotlin
/**
 * Represents a value with lazy initialization.
 *
 * To create an instance of [Lazy] use the [lazy] function.
 */
public interface Lazy<out T> {
    /**
     * Gets the lazily initialized value of the current Lazy instance.
     * Once the value was initialized it must not change during the rest of lifetime of this Lazy instance.
     */
    public val value: T
    /**
     * Returns `true` if a value for this Lazy instance has been already initialized, and `false` otherwise.
     * Once this function has returned `true` it stays `true` for the rest of lifetime of this Lazy instance.
     */
    public fun isInitialized(): Boolean
}
```

这个接口存了值，和是否已经初始化了的变量，再看SynchronizedLazyImpl

```kotlin

private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                }
                else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }

    override fun isInitialized(): Boolean = _value !== UNINITIALIZED_VALUE

    override fun toString(): String = if (isInitialized()) value.toString() else "Lazy value not initialized yet."

    private fun writeReplace(): Any = InitializedLazyImpl(value)
}
```



重写了value的get方法，_value会被赋值一次，并且是线程同步的。之后 _value再也不会是null值，就一直直接返回。

可能会有疑惑就是lazy函数返回值是一个Lazy<T>类型，

```Kotlin
val d:Int by lazy {
        2
    }
```

怎么通过赋值，赋值给这个变量d呢

```Kotlin
public inline operator fun <T> Lazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value
```

by 执行了Lazy<T> 的getValue方法 实现赋值的。



### Observable

```kotlin
  var name: String by Delegates.observable("wang", {
        kProperty, oldName, newName ->
        println("kProperty：${kProperty.name} | oldName:$oldName | newName:$newName")
    })
```



这个委托会帮我们监控属性的变化，当set被调用的时候，会执行我们的lambda表达式查看方法如下

```kotlin
 public inline fun <T> observable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit): ReadWriteProperty<Any?, T> = object : ObservableProperty<T>(initialValue) {
        override fun afterChange(property: KProperty<*>, oldValue: T, newValue: T) = onChange(property, oldValue, newValue)
    }
```



涉及到内联关键字

所谓的内联函数就是在被调用的地方直接展开，没有参数压栈和返回时候的参数出栈资源释放的操作，提高了程序的执行速度

#### kotlin inline noinline crossinline

[参考](http://blog.csdn.net/u013009899/article/details/78584994)

一个函数中，如果存在一个lambda表达式，在该lambda中不支持直接进行return退出该函数，比如：

```kotlin
fun outterFun() {
    innerFun {
        //return  //错误，不支持直接return
        //只支持通过标签，返回innerFun
        return@innerFun
    }

    //如果是匿名或者具名函数，则支持
    var f = fun(){
        return
    }
}

fun innerFun(a: () -> Unit) {}
```

除非是inline函数：

```kotlin
fun outterFun() {
    innerFun {
        return  //支持直接返回outterFun
    }
}

inline fun innerFun(a: () -> Unit) {}
```

如果是内联

```kotlin
 fun outterFun() {
        innerFun {
            //return  //错误，不支持直接return
            //只支持通过标签，返回innerFun
            Log.d("======","=============innerFun===============")
            return
        }
        Log.d("======","=============innerFun2222===============")
        //如果是匿名或者具名函数，则支持
        var f = fun(){
            return
        }
    }

   inline fun innerFun(a: () -> Unit) {
        a()
    }
//echo ======: =============innerFun===============
```

可以理解，方法执行的时候被直接展开，innerFun2222没有得到输出，因为内联函数return了



###### noinline

> 如果你只想被（作为参数）传给一个内联函数的 lamda 表达式中只有一些被内联，你可以用 noinline 修饰符标记一些函数参数：

##### crossinline

crossinline 的作用是让被标记的lambda表达式不允许非局部返回。 

```kotlin
 fun outterFun() {
        innerFun {
           
            Log.d("======","=============innerFun===============")
            return@innerFun
        }
        Log.d("======","=============innerFun2222===============")
        //如果是匿名或者具名函数，则支持
        var f = fun(){
            return
        }
    }

   inline fun innerFun(crossinline a: () -> Unit) {
        a()
    }
 //echo =============innerFun===============
 //=============innerFun2222===============

```

不允许局部返回之后有两个echo 日志









