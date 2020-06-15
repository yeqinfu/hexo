---
title: Jetpack全家桶目录
date: 2020-05-15 14:00:58
tags: Jetpack
---

本着跟随谷歌的脚步，学习一下Jetpack全家桶在项目中的使用。大概使用的框架有这些

+ Databinding
+ LiveData
+ Navigation
+ Room
+ ViewModel
+ WorkManager
+ ViewPager2

基本上，全家桶全用上。所以这里使用sunflower项目进行一个全家桶最佳实践的学习。不过缺少网络请求部分的内容。Paging分页库也没有。

![sunflower](https://tva4.sinaimg.cn/large/c1b251b3gy1gewl2tjup8j21yd20jn60.jpg)

## Sunflower项目

{% asset_img sunflower.png  %}

这个项目采用全Kotlin编写，使用Jetpack全家桶，因为使用Navigation是属于单Ativity多Fragment的项目。

```kotlin
class GardenActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView<ActivityGardenBinding>(this, R.layout.activity_garden)
    }
}
```

主Activity很简单，就一个设置布局。布局包含一个Navigation的根

### Navigation导航图简单介绍

```
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <fragment
            android:id="@+id/nav_host"
            android:name="androidx.navigation.fragment.NavHostFragment"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:defaultNavHost="true"
            app:navGraph="@navigation/nav_garden"/>

    </FrameLayout>

</layout>
```

> layout这个标签是属于databinding的范畴。navigation需要一个默认的navHost这里可以看到指向nav_garden这个导航图

导航图描述的是多个Fragment之间的跳转关系，他帮助我们处理Fragment的事务，还有跳转动画，跳转传参定义。搭配ViewModel就可以实现多个Fragment的数据共享，不用说里面的Fragment更新了数据还需要EventBus通知一下外部的Fragment进行刷新。



![image](https://tva3.sinaimg.cn/large/c1b251b3gy1getawbk4i6j20kc0dlglz.jpg)



```
<?xml version="1.0" encoding="utf-8"?>
<!--
  ~ Copyright 2018 Google LLC
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~     https://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->

<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@id/view_pager_fragment">

    <fragment
        android:id="@+id/view_pager_fragment"
        android:name="com.google.samples.apps.sunflower.HomeViewPagerFragment"
        tools:layout="@layout/fragment_view_pager">

        <action
                android:id="@+id/action_view_pager_fragment_to_plant_detail_fragment"
                app:destination="@id/plant_detail_fragment"
                app:enterAnim="@anim/slide_in_right"
                app:exitAnim="@anim/slide_out_left"
                app:popEnterAnim="@anim/slide_in_left"
                app:popExitAnim="@anim/slide_out_right" />

    </fragment>

    <fragment
        android:id="@+id/plant_detail_fragment"
        android:name="com.google.samples.apps.sunflower.PlantDetailFragment"
        android:label="@string/plant_details_title"
        tools:layout="@layout/fragment_plant_detail">
        <argument
            android:name="plantId"
            app:argType="string" />
    </fragment>

</navigation>
```

导航图，描述了一个

```
app:startDestination="@id/view_pager_fragment"
```

起点是一个viewpagerfragment也就是HomeViewPagerFragment 定义了一些动画，并且支持跳转到PlantDetailFragment ，通过下面的属性进行声明

```
 app:destination="@id/plant_detail_fragment"
```

导航图的制作，是面向图形界面友好的。可以通过图形操作进行一个导航图编辑。

#### 定义了导航图之后，怎么用？

当我们定义了一个导航图之后，系统会生成fragment的跳转代码，不用我们提交fragment replace事务。

对应上面导航图，生成了一个类

```
class HomeViewPagerFragmentDirections private constructor() {
  private data class ActionViewPagerFragmentToPlantDetailFragment(
    val plantId: String
  ) : NavDirections {
    override fun getActionId(): Int = R.id.action_view_pager_fragment_to_plant_detail_fragment

    override fun getArguments(): Bundle {
      val result = Bundle()
      result.putString("plantId", this.plantId)
      return result
    }
  }

  companion object {
    fun actionViewPagerFragmentToPlantDetailFragment(plantId: String): NavDirections =
        ActionViewPagerFragmentToPlantDetailFragment(plantId)
  }
}

```

导航图中，HomeViewPagerFragment有多少个目的地，这里就会相应生成。跳转代码

```kotlin
 val direction =
                HomeViewPagerFragmentDirections.actionViewPagerFragmentToPlantDetailFragment(
                    plant.plantId
                )
            view.findNavController().navigate(direction)
```



### HomeViewPagerFragment

主Activity设置了默认起点是HomeViewPagerFragment，所以这里看这个类用到什么东西

```kotlin
class HomeViewPagerFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val binding = FragmentViewPagerBinding.inflate(inflater, container, false)
        val tabLayout = binding.tabs
        val viewPager = binding.viewPager

        viewPager.adapter = SunflowerPagerAdapter(this)

        // Set the icon and text for each tab
        TabLayoutMediator(tabLayout, viewPager) { tab, position ->
            tab.setIcon(getTabIcon(position))
            tab.text = getTabTitle(position)
        }.attach()

        (activity as AppCompatActivity).setSupportActionBar(binding.toolbar)

        return binding.root
    }

    private fun getTabIcon(position: Int): Int {
        return when (position) {
            MY_GARDEN_PAGE_INDEX -> R.drawable.garden_tab_selector
            PLANT_LIST_PAGE_INDEX -> R.drawable.plant_list_tab_selector
            else -> throw IndexOutOfBoundsException()
        }
    }

    private fun getTabTitle(position: Int): String? {
        return when (position) {
            MY_GARDEN_PAGE_INDEX -> getString(R.string.my_garden_title)
            PLANT_LIST_PAGE_INDEX -> getString(R.string.plant_list_title)
            else -> null
        }
    }
}
```

这个Fragment也是使用过Databinding来绑定ViewModel和layout的内容设定。

> fragment的layout名字为 fragment_view_pager.xml使用databinding生成一个类FragmentViewPagerBinding 用来绑定xml中声明的元素和代码之间的联系

#### ViewPager2

HomeViewPagerFragment 内容部分是一个[ViewPager2](https://github.com/android/views-widgets-samples/tree/master/ViewPager2)

新特性：

+ 支持从右到左布局

  > 需要 
  >
  > ```
  > <application    android:icon="@drawable/app_sample_code"    android:label="@string/activity_sample_code"    android:supportsRtl="true"    android:theme="@style/Theme.AppCompat.Light">
  > ```
  >
  > ```
  > <androidx.viewpager2.widget.ViewPager2    android:layoutDirection="rtl"    android:id="@+id/viewPager"    android:layout_width="match_parent"    android:layout_height="match_parent"/>
  > ```
  >
  > 设置支持

+ 支持从上到下的滚动，垂直滚动

+ 滚动动画特效

  > ```
  > ViewPager2.PageTransformer
  > ```



#### TabLayoutMediator

ViewPager2无法和 Tablayout直接绑定使用这个类来完成绑定，完成联动。

#### CollapsingToolbarLayout

这里还有简单的监听ViewPager2滑动事件，进行一个头部的收起的特效

```
 app:layout_behavior="@string/appbar_scrolling_view_behavior"
```

#### SunflowerPagerAdapter

```kotlin
const val MY_GARDEN_PAGE_INDEX = 0
const val PLANT_LIST_PAGE_INDEX = 1

class SunflowerPagerAdapter(fragment: Fragment) : FragmentStateAdapter(fragment) {

    /**
     * Mapping of the ViewPager page indexes to their respective Fragments
     */
    private val tabFragmentsCreators: Map<Int, () -> Fragment> = mapOf(
        MY_GARDEN_PAGE_INDEX to { GardenFragment() },
        PLANT_LIST_PAGE_INDEX to { PlantListFragment() }
    )

    override fun getItemCount() = tabFragmentsCreators.size

    override fun createFragment(position: Int): Fragment {
        return tabFragmentsCreators[position]?.invoke() ?: throw IndexOutOfBoundsException()
    }
}
```

看到Map的value是一个函数，返回相应的两个子Fragment。调用返回的时候使用invoke方法。

### GardenFragment 我的花园页面

布局使用Databinding，主要是一个recycleview列表和空状态的添加按钮。

```
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
                name="hasPlantings"
                type="boolean" />

    </data>

    ···省略···

</layout>
```

> 声明了变量，判断是否一棵都没种

#### GardenPlantingListViewModel

```
 private val viewModel: GardenPlantingListViewModel by viewModels {
        InjectorUtils.provideGardenPlantingListViewModelFactory(requireContext())
    }
```

在GardenFragment 声明了一个数据模型GardenPlantingListViewModel

#### by viewModels

这是一个拓展函数，帮助我们进行ViewModel初始化

```
@MainThread
inline fun <reified VM : ViewModel> Fragment.viewModels(
    noinline ownerProducer: () -> ViewModelStoreOwner = { this },
    noinline factoryProducer: (() -> Factory)? = null
) = createViewModelLazy(VM::class, { ownerProducer().viewModelStore }, factoryProducer)
```

这个拓展函数第一个参数是ownerProducer代表生命周期所有者，第二个参数代表生产者。先看第一个参数，她的参数类型是一个函数，返回值是ViewModelStoreOwner，默认参数值是{ this }，那么其实也就是Fragment本身。那其实可以简化成

```
@MainThread
inline fun <reified VM : ViewModel> Fragment.viewModels(
        noinline factoryProducer: () -> ViewModelProvider.Factory
) = createViewModelLazy(VM::class, {this.viewModelStore }, factoryProducer)

```

只有一个参数，第一个参数用this直接代替。只适用于第一个参数不传的情况。

第二个参数是提供者，是

```
 InjectorUtils.provideGardenPlantingListViewModelFactory(requireContext())
```

的返回值

#### GardenPlantingRepository 花园在种的数据仓库

这个仓库数据直连Room数据库。从表中获取的数据。所以先看Room库的数据配置内容

### AppDatabase 数据库管理

定义了数据版本和多少张表并且使用@Volatile 全局单例。整个类很简单的内容，其中一个创建数据库的方法。使用的是WorkManager来创建数据库

### WorkManager

调度任务,可以轻松地调度即使在应用退出或设备重启时仍应运行的可延迟异步任务,定义了一个只执行一次的创建数据任务。

任务内容

```

class SeedDatabaseWorker(
    context: Context,
    workerParams: WorkerParameters
) : CoroutineWorker(context, workerParams) {
    override suspend fun doWork(): Result = coroutineScope {
        try {
            applicationContext.assets.open(PLANT_DATA_FILENAME).use { inputStream ->
                JsonReader(inputStream.reader()).use { jsonReader ->
                    val plantType = object : TypeToken<List<Plant>>() {}.type
                    val plantList: List<Plant> = Gson().fromJson(jsonReader, plantType)

                    val database = AppDatabase.getInstance(applicationContext)
                    database.plantDao().insertAll(plantList)

                    Result.success()
                }
            }
        } catch (ex: Exception) {
            Log.e(TAG, "Error seeding database", ex)
            Result.failure()
        }
    }

    companion object {
        private const val TAG = "SeedDatabaseWorker"
    }
}
```

也就是简单的assets文件读取数据转成数据库一个过程，当Result.success()标志着任务完成结束。

#### GardenPlanting表 Plant表

回到AppDatabase，看下两张表结构

```
 /**
     * Indicates when the [Plant] was planted. Used for showing notification when it's time
     * to harvest the plant.
     */
    @ColumnInfo(name = "plant_date") val plantDate: Calendar = Calendar.getInstance(),

```

其中有奇怪的地方，表结构一般都是基本数据类型或者String，但是这边的表结构字段出现了Calendar对象，也就是Room支持的功能。

在AppDatabase顶部作用域有一个注解

```
@Database(entities = [GardenPlanting::class, Plant::class], version = 1, exportSchema = false)
@TypeConverters(Converters::class)
```

其中有一个类型转化器。内容如下

```
/**
 * Type converters to allow Room to reference complex data types.
 */
class Converters {
    @TypeConverter fun calendarToDatestamp(calendar: Calendar): Long = calendar.timeInMillis

    @TypeConverter fun datestampToCalendar(value: Long): Calendar =
            Calendar.getInstance().apply { timeInMillis = value }
}
```

正是这个类型转化器，帮助我们从Calendar到timeInMillis的转化。

>@TypeConverter的两个方法是成对出现的，方法名称可以任意命名，重点在入参和出参类型，必须是需要转换的类型和最终转换后的类型
>
>并且这个转化器是放在Database级别的，所以，所有的表都是可以用的。如果放在Entity或者Dao则使用范围相应变小。



#### 回到 GardenPlantingRepository 



![sunflower_viewmodel](https://tvax1.sinaimg.cn/large/c1b251b3gy1gewl37uxwqj20t50iz3zc.jpg)

花园列表，我们种植的花园数据，第一是我们种的具体植物，可能是番茄，种类是番茄，实例可能是多棵番茄。所以这里有一对多的关系。

```
/**
 * This class captures the relationship between a [Plant] and a user's [GardenPlanting], which is
 * used by Room to fetch the related entities.
 */
data class PlantAndGardenPlantings(
    @Embedded
    val plant: Plant,//表示种类信息

    @Relation(parentColumn = "id", entityColumn = "plant_id")
    val gardenPlantings: List<GardenPlanting> = emptyList()//表示在种的列表
)

```

> @Embedded 这个注解可以被用在Entity表种，或者任意实体类？ 去表明一个嵌套关系，被打上标注得字段可以用于数据库查询。
> 情况一，如果被标注得是一个Entity,那么被标注的表字段会被包含在这个外部的Entity表中。
>
> *   public class Coordinates {
>  *       double latitude;
>  *       double longitude;
>  *   }
>  *   public class Address {
>  *       String street;
>  *       {@literal @}Embedded
>  *       Coordinates coordinates;
>  *   }
>  比如。上面，Room数据库会把经纬度两个字段，作为Address的表字段。
>  如果字段名字有冲突，可以用prefix来标准被注解包含进来的新字段，比如经纬度。加前缀
>
> @Relation
>
> 关系映射，PlantAndGardenPlantings这个类包含两个字段，两张表，
> 这里用这个注解进行一个关系映射
> Plant代表植物目录中的每个物种的种植信息，他的id映射到我的花园中已经种植的具体花草信息，这个是一个一对多的关系。

最终首页的数据源

```
val plantAndGardenPlantings: LiveData<List<PlantAndGardenPlantings>> =
            gardenPlantingRepository.getPlantedGardens()
```

和我们的Room数据库进行了一个关联

### 回到GardenFragment 数据源和界面绑定

```
 private fun subscribeUi(adapter: GardenPlantingAdapter, binding: FragmentGardenBinding) {
        viewModel.plantAndGardenPlantings.observe(viewLifecycleOwner) { result ->
            binding.hasPlantings = !result.isNullOrEmpty()
            adapter.submitList(result)
        }
    }
```

viewModel实例和界面绑定对象binding进行了一个观察者模式的关联。

### PlantListFragment

同样有一个PlantListViewModel进行了一个数据关联。不再展开

#### MaskedCardView

```

/**
 * A Card view that clips the content of any shape, this should be done upstream in card,
 * working around it for now.
 */
class MaskedCardView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyle: Int = R.attr.materialCardViewStyle
) : MaterialCardView(context, attrs, defStyle) {
    @SuppressLint("RestrictedApi")
    private val pathProvider = ShapeAppearancePathProvider()
    private val path: Path = Path()
    private val shapeAppearance: ShapeAppearanceModel = ShapeAppearanceModel(
        context,
        attrs,
        defStyle,
        R.style.Widget_MaterialComponents_CardView
    )
    private val rectF = RectF(0f, 0f, 0f, 0f)

    override fun onDraw(canvas: Canvas) {
        canvas.clipPath(path)
        super.onDraw(canvas)
    }

    @SuppressLint("RestrictedApi")
    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        rectF.right = w.toFloat()
        rectF.bottom = h.toFloat()
        pathProvider.calculatePath(shapeAppearance, 1f, rectF, path)
        super.onSizeChanged(w, h, oldw, oldh)
    }
}
```

MaterialCardView是material库支持圆角的一个类。但是这里顶部是一张图片，右上角的圆角设置会失效。所以这个自定义view重新进行一个裁剪。

>  @JvmOverloads 使用这个注解，可以省去自定义个view的多个构造

#### Plant表

```

    /**
     * Determines if the plant should be watered.  Returns true if [since]'s date > date of last
     * watering + watering Interval; false otherwise.
     */
    fun shouldBeWatered(since: Calendar, lastWateringDate: Calendar) =
        since > lastWateringDate.apply { add(DAY_OF_YEAR, wateringInterval) }
     
```

   这里用到kotlin默认重载>号调用compareTo