---
title: Kotlin泛型函数
date: 2018-01-23 09:00:58
tags: code
---



### let

```kotlin
inline fun <T, R> T.let(f: (T) -> R): R = f(this)
```

方法声明中看出，接受的参数是一个函数f，这个f的入参就是T，返回值就是R。然后let函数的返回值类型为R整个let的方法体就是f(this)。上面这个函数可以改成看的清晰点的：

```kotlin
inline fun <T, R> T.let(f: (T) -> R): R {
       return  f(this)
} 
```



在运用let函数的时候可以这样写,入参是大括号的闭包

```kotlin
 var d=Intent()
 var re=d?.let({
            haha(d)
        })
fun haha(i:Intent){}
```

lambda简写，第一个中省略小括号，省略小括号并且把闭包放在外面

```kotlin
var re=d?.let(){
            haha(d)
 }


var re=d?.let{
            haha(d)
}
```

闭包默认有一个参数被省略

```kotlin
var re=d?.let(){it->
            haha(it)
}
```

也可以写成这样,下划线代表参数占位符

```kotlin
 var re=d?.let{_->
            haha(it)
 }
```

let函数加？可以让it保证不为空，let函数只有d不为空的时候被执行

###  with

```kotlin
inline fun <T, R> with(receiver: T, f: T.() -> R): R = receiver.f()
```

with接收一个对象receiver，和一个函数，这个函数会作为这个对象的扩展函数执行。最后返回这个函数的返回值。

```kotlin
f: (T) -> R
f: T.() -> R
```

这两个表达式的区别，第一个代表一个方法，名字叫f 参数表示一个T，返回值是R。第二个代表名字叫f，是T的扩展函数，返回值为R。

```kotlin
val list: MutableList<String> = mutableListOf("A", "B", "C")
  val change: Any
  change = with(list) {
      add("D")
      add("E")
      this.add("F")
      size
  }
```

简写后，大括号代表拓展函数体，add方法是list的方法，可以加this，也可以不加list，最后一句表达式就是返回值。

### apply

```kotlin
inline fun <T> T.apply(f: T.() -> Unit): T { f(); return this }
```

apply被定义为T的拓展函数，接受一个扩展函数，返回T本身。和with类似，但是返回的一直都是this，即T



### safeRun

```kotlin
 /**
     * 自定义泛型函数 结合let函数
     * 如果fragement没有被attached到Activity就不执行
     * 这边可以做异步返回的检查工作
     */
    inline fun <T> T.safeRun(f: (T) -> Unit) {
        if (isAdded) {
           f(this)
        }
    }
```



项目中，会出现一个异步异常，就是如果异步返回的时候fragment已经被销毁，比如多个viewpager迅速切换，这时候在返回的回调函数调用设置界面就会出现异常奔溃，是因为fragment已经没有和activity关联了。所以，自定义了这个扩展函数，使用如下

```kotlin
   response?.safeRun {
                        tv_totalKwh.text=it.message.analysisCateSum
                       
                    }
```

在这个方法体执行之前，做了不为null的检查和fragment的attched的检查。





