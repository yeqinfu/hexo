---
title: Dagger2入门
date: 2019-03-12 10:00:58
tags: android
---



#### @Inject注解

```java
 @Inject
 LoginActivityPresenter presenter;
```

```java
 @Inject
 public LoginPresenter(ICommonView iView){
        this.iView = iView;
 }
```

使用的地方加一個注解可以省去new实例，在对应的构造方法加一个注解。这两个呼应

#### @Component注解

直译组件，想象成一个容器，module中的产物都放在容器里面，然后这个组件和要使用的地方做关联，比如和Activity做关联。activity要实例可以到这个容器取。

```java
@ActivityScope
@Component(modules = CommonModule.class)
public interface CommonComponent {
    void inject(MainActivity activity);
}
```

注入器从注入后，开始给需要的赋值

```java
 DaggerCommonComponent.
                builder().
                commonModule(new CommonModule(this)).
                build().
                inject(this);
```

#### @Module注解

```java
@Module
public class CommonModule{

    private ICommonView iView;
    public CommonModule(ICommonView iView){
        this.iView = iView;
    }


    @Provides
    @ActivityScope
    public ICommonView provideIcommonView(){
        return this.iView;
    }

}
```

被打上@module注解的类相当于一个工厂，具体地负责实例的生产，生产的方法都被打上@Provides标签

但是上面这个module的方法中提供的是ICommonView，并不是LoginPresenter实例。

所以我在上面的module中再添加了一个方法

```java
  @Provides
    @ActivityScope
    public ICommonView provideIcommonView(){
        Log.d("yeqinfu","=======provideIcommonView=====");
        return this.iView;
    }
    @Provides
    @ActivityScope
    public LoginPresenter provideLoginPresenter(){
        Log.d("yeqinfu","======provideLoginPresenter======");

        return new LoginPresenter(this.iView);
    }
```

其他保持不变，重新build后发现调用的是provideLoginPresenter这个方法，而如果把provideLoginPresenter这个方法去掉，会发现调用的是provideIcommonView这个方法。很神奇，为什么呢？

build之后可以看到每一个provide注解都会生成一个类。

![image](https://ws3.sinaimg.cn/large/c1b251b3gy1g0zwqb3q66j20by049mx9.jpg)



但是这两个生成的类在哪里被使用呢？当我们提供这两个类的时候，发现DaggerCommonComponent类中使用了CommonModule_ProvideLoginPresenterFactory

![image](https://ws1.sinaimg.cn/large/c1b251b3gy1g0zwsoj17wj20wg0cqdh8.jpg)

而如果注掉provideLoginPresenter，就会默认使用CommonModule_ProvideIcommonViewFactory

![image](https://ws3.sinaimg.cn/large/c1b251b3gy1g0zwtpxg7bj20zy0i476g.jpg)

####  @Singleton

单例注解

如果需要两个对象如下

```
    @Inject
    LoginPresenter presenter;

    @Inject
    LoginPresenter presenter2;
    Log.d("yeqinfu","---hahah->"+presenter);
    Log.d("yeqinfu","---hahah->"+presenter2);
```



![image](https://ws2.sinaimg.cn/large/c1b251b3gy1g0zxe7qcpkj20lp02t0sr.jpg)

可以看到这是通过新new出来的。而实际上有时候并不需要

![image](https://ws3.sinaimg.cn/large/c1b251b3gy1g0zxkrjsdoj20fx057jrn.jpg)



![image](https://ws2.sinaimg.cn/large/c1b251b3gy1g0zxl0u8g6j20ny05xaae.jpg)



在组件和provide方法添加单例声明，之后用的就是不会新new一个对象，组件那边不添加声明会报错，而provide方法添加声明会让里面的new LoginPresenter只被调用一次。而如果我们把provideLoginPresenter注释掉，改用

![image](https://ws2.sinaimg.cn/large/c1b251b3gy1g0zxnc9cc1j20n4054t8y.jpg)

​	那么并不会让LoginPresenter保持单例。

#### 自定义标记 @Qualifier 和 ＠Named

像上面如果provide一个对象，有多个构造方法不知道调用那个就用这两个标注做区分。

```
    @Named("context")
    @Provides
    public Person providesPersonWithContext(Context context) {
        return new Person(context);
    }

    @Named("string")
    @Provides
    public Person providesPersonWithName() {
        return new Person("yxm");
    }
```

使用的时候

```
 @Named("string")
    @Inject
    Person p1;

    @Named("context")
    @Inject
    Person p2;
```

也可以自定义标注

```
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface PersonWithContext {
}
```

```
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface PersonWithName {
}
```

```
   @PersonWithContext
    @Provides
    public Person providesPersonWithContext(Context context) {
        return new Person(context);
    }

    @PersonWithName
    @Provides
    public Person providesPersonWithName() {
        return new Person("yxm");
    }
```

```
   @PersonWithContext
    @Inject
    Person p1;

    @PersonWithContext
    @Inject
    Person p2;
```

#### 自定义生命周期@Scope

和单例一样

```
@Scope
@Documented
@Retention(RUNTIME)
public @interface Singleton {}
```

```
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityScope {
}
```

只是名字不一样，一个用来表示单例，一个是我们自定义用来维持和activity一样的生命周期，应该是这个组件在哪里注入了，就在那里，比如activity生命周期内保持单例，如果activity销毁了，这个单例也没了。

如果有另外一个activity也注入了这个组件，它是new一个和这个activity保持单例，而不是和前面一个一样。