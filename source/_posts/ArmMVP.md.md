---
title: ArmsMVP学习
date: 2019-03-17 14:00:58
tags: android
---

项目中用到这个框架，所以需要学习它。理清这几点

+ 这个框架集成了哪些框架，做了什么整合工作
+ Arms的整体框架逻辑

从我们继承的BaseApplication开始入手

![image](https://ws1.sinaimg.cn/large/c1b251b3gy1g16659rc6gj20ru0gsmz2.jpg)

BaseApplication的内容很简单，所有的逻辑都放在AppDelegate中。

#### AppDelegate

```java
public AppDelegate(@NonNull Context context) {

        //用反射, 将 AndroidManifest.xml 中带有 ConfigModule 标签的 class 转成对象集合（List<ConfigModule>）
        this.mModules = new ManifestParser(context).parse();

        //遍历之前获得的集合, 执行每一个 ConfigModule 实现类的某些方法
        for (ConfigModule module : mModules) {

            //将框架外部, 开发者实现的 Application 的生命周期回调 (AppLifecycles) 存入 mAppLifecycles 集合 (此时还未注册回调)
            module.injectAppLifecycle(context, mAppLifecycles);

            //将框架外部, 开发者实现的 Activity 的生命周期回调 (ActivityLifecycleCallbacks) 存入 mActivityLifecycles 集合 (此时还未注册回调)
            module.injectActivityLifecycle(context, mActivityLifecycles);
        }
    }

```

构造函数加载子工程的配置文件。

```
 <!-- Arms 配置 -->
        <meta-data
                android:name="com.thread0.demo.app.CalendarConfiguration"
                android:value="ConfigModule"/>
```

比如以上的配置被加载进去。通过反射得到这个类的实例，这个配置可以多个。必须实现ConfigModule这个接口。	子工程实现的AppLifecycle，ActivityLifecycle在构造函数就被收集到这个AppDelegate代理类中。

在这个代理类中也是使用了一些注入

```
     //注册框架内部已实现的 Activity 生命周期逻辑
        mApplication.registerActivityLifecycleCallbacks(mActivityLifecycle);

        //注册框架内部已实现的 RxLifecycle 逻辑
        mApplication.registerActivityLifecycleCallbacks(mActivityLifecycleForRxLifecycle);
```

mActivityLifecycle，mActivityLifecycleForRxLifecycle这两个变量在这里注入

```
 @Inject
    @Named("ActivityLifecycle")
    protected Application.ActivityLifecycleCallbacks mActivityLifecycle;
    @Inject
    @Named("ActivityLifecycleForRxLifecycle")
    protected Application.ActivityLifecycleCallbacks mActivityLifecycleForRxLifecycle;
```

以这两个实例为例子，追溯这两个是如何被赋值的。

被注入的变量都是有一个组件提供接口，也就是DaggerAppComponent对应的工厂类是AppModule

```
 @Binds
    @Named("ActivityLifecycle")
    abstract Application.ActivityLifecycleCallbacks bindActivityLifecycle(ActivityLifecycle activityLifecycle);

    @Binds
    @Named("ActivityLifecycleForRxLifecycle")
    abstract Application.ActivityLifecycleCallbacks bindActivityLifecycleForRxLifecycle(ActivityLifecycleForRxLifecycle activityLifecycleForRxLifecycle);
```

看到相应的方法，恰好这不是一个@Provides注解，而是一个@Binds注解，这个注解意味着提供的方法可以是一个抽象方法，以bindActivityLifecycle方法为例子，最终提供的是ActivityLifecycle这个最终实现类。点击这个类，可以看到

```java
public class ActivityLifecycle implements Application.ActivityLifecycleCallbacks {

    @Inject
    AppManager mAppManager;
    @Inject
    Application mApplication;
    @Inject
    Cache<String, Object> mExtras;
    @Inject
    Lazy<FragmentManager.FragmentLifecycleCallbacks> mFragmentLifecycle;
    @Inject
    Lazy<List<FragmentManager.FragmentLifecycleCallbacks>> mFragmentLifecycles;

    @Inject
    public ActivityLifecycle() {
    }
```

看到这个类的构造被打上了注入标签，说明就是这个构造函数被实例化。还发现这个类还依赖其他的注入，以AppManager为例子

```
        this.activityLifecycleProvider = DoubleCheck.provider(ActivityLifecycle_Factory.create(this.provideAppManagerProvider, this.applicationProvider, this.provideExtrasProvider, this.fragmentLifecycleProvider, this.provideFragmentLifecyclesProvider));

```

可以看到这些依赖的注入都是由apt生成代码里面另外提供的。最后看到生成的类

```
AppModule_ProvideAppManagerFactory
```

由实现get方法，这个是Factory接口提供的，也是Provider提供的。

```
public interface Factory<T> extends Provider<T> {
}
```



跑远了。。。。



然后回到AppDelegate，也就是说arms本身就向系统注册了mActivityLifecycle这个用来记录系统的activity的所有生命周期。同时也允许子工程另外添加activityLifecycle这些回调。

逻辑都在这个代理类中，如果项目不能继承它的baseapp就需要拷贝它的baseapp的代码到我们自己实现的子类中。



#### BaseActivity

这个类的逻辑也很简单。需要看到继承这个基类后需要实现的一些方法。比如setupActivityComponent。

刚才上面看到了这个框架本身就提供了activity生命周期的回调。ActivityLifecycle

```
  public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        //如果 intent 包含了此字段,并且为 true 说明不加入到 list 进行统一管理
        boolean isNotAdd = false;
        if (activity.getIntent() != null)
            isNotAdd = activity.getIntent().getBooleanExtra(AppManager.IS_NOT_ADD_ACTIVITY_LIST, false);

        if (!isNotAdd)
            mAppManager.addActivity(activity);

        //配置ActivityDelegate
        if (activity instanceof IActivity) {
            ActivityDelegate activityDelegate = fetchActivityDelegate(activity);
            if (activityDelegate == null) {
                Cache<String, Object> cache = getCacheFromActivity((IActivity) activity);
                activityDelegate = new ActivityDelegateImpl(activity);
                //使用 IntelligentCache.KEY_KEEP 作为 key 的前缀, 可以使储存的数据永久存储在内存中
                //否则存储在 LRU 算法的存储空间中, 前提是 Activity 使用的是 IntelligentCache (框架默认使用)
                cache.put(IntelligentCache.getKeyOfKeep(ActivityDelegate.ACTIVITY_DELEGATE), activityDelegate);
            }
            activityDelegate.onCreate(savedInstanceState);
        }

        registerFragmentCallbacks(activity);
    }

```

在这里，判断了是不是实现了IActivity这个接口，来对应匹配ActivityDelegate代理类。而这个代理类的实现

```

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        //如果要使用 EventBus 请将此方法返回 true
        if (iActivity.useEventBus()){
            //注册到事件主线
            EventBusManager.getInstance().register(mActivity);
        }

        //这里提供 AppComponent 对象给 BaseActivity 的子类, 用于 Dagger2 的依赖注入
        iActivity.setupActivityComponent(ArmsUtils.obtainAppComponentFromContext(mActivity));
    }

```

就调用了注册组件的方法。fragment同理。

现在还差，这些缓存到底有什么作用。































