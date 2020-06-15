---
title: Serializable和内部类出现的bug
date: 2020-06-15 09:00:00
tags: code
---

```
Caused by: java.lang.RuntimeException: Parcelable encountered IOException writing serializable object
at android.os.Parcel.writeSerializable(Parcel.java:1534)
        at android.os.Parcel.writeValue(Parcel.java:1482)
        at android.os.Parcel.writeList(Parcel.java:819)
        at android.os.Parcel.writeValue(Parcel.java:1431)
        at android.os.Parcel.writeArrayMapInternal(Parcel.java:731)
        at android.os.BaseBundle.writeToParcelInner(BaseBundle.java:1408)
        at android.os.Bundle.writeToParcel(Bundle.java:1133)
        at android.os.Parcel.writeBundle(Parcel.java:771)
        at android.content.Intent.writeToParcel(Intent.java:8751)
        at android.app.ActivityManagerProxy.startActivity(ActivityManagerNative.java:3260)
        at android.app.Instrumentation.execStartActivity(Instrumentation.java:1524)
        at android.app.Activity.startActivityForResult(Activity.java:4267)
        at androidx.fragment.app.FragmentActivity.startActivityForResult(FragmentActivity.java:767)
        at android.app.Activity.startActivityForResult(Activity.java:4226)
        at androidx.fragment.app.FragmentActivity.startActivityForResult(FragmentActivity.java:754)
```

## 前提

上面这个报错场景是，页面之间跳转，需要传对象过去，所以类需要实现Serializable接口，再放进bundle里面。

```
    private ArrayList<PostFoodListBody> allFoods;//要传递的数组
    
    public class PostFoodListBody implements Serializable {
      private String id;
    private String dishName;
    }//这个对象实现了接口，并且成员变量只有String类型，没有嵌套类
   
   
   //在activity外面，给allFoods添加点东西
     allFoods.add(new PostFoodListBody(){
                @Override
                public String getUniqueId() {
                    return "dd";
                }

                @Override
                public String getDisplayStr() {
                    return "test";
                }
            });
            
   //然后页面跳转就报错了

```

报错内容如前面所示。报错原因是因为匿名内部类的实现持有了外部activity的引用，而activity并没有实现序列化接口。当然你实现了也没有用，activity的所有的成员类都得一鼓劲都实现了。

## 解决方法

+ 把要传的东西通过构造函数传递

+ 写成静态内部类

  ```
   static PostFoodListBody target=new PostFoodListBody(){
          @Override
          public String getDisplayStr() {
              return "dddfdkjalf";
          }
      };
        allFoods.add(target);
  ```

  

