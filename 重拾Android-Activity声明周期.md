---
title: Android例子-快捷方式
date: 2019-02-12 14:00:58
tags: android
---

#### android 快捷方式 -译文

现在在sdk27已经可以使用快捷方式，具体就是长按app，比如可以测试一下支付宝快捷方式，快捷方式最多可以添加五个（动态和静态合计），固定的快捷方式（pinned shortcuts）没有限制，用户可以随意创建，我们不能删除他们，只能让它们失效。

![image](https://ws3.sinaimg.cn/large/c1b251b3gy1g04lajmzj1j20u01hcqd8.jpg)



#### 添加快捷方式

##### 静态添加快捷方式

```
<activity android:name="Main">
      <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
      
      <meta-data android:name="android.app.shortcuts"
                 android:resource="@xml/shortcuts" /> 
    </activity>
```

把快捷方式的xml放置在启动器的activity。

```xml
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android" >
    <shortcut
        android:shortcutId="add_website"
        android:icon="@drawable/add"
        android:shortcutShortLabel="@string/add_new_website_short"
        android:shortcutLongLabel="@string/add_new_website"
        >
        <intent
            android:action="com.example.android.appshortcuts.ADD_WEBSITE"
            android:targetPackage="com.example.android.shortcutsample"
            android:targetClass="com.example.android.appshortcuts.Main"
            />
    </shortcut>
</shortcuts>
```

xml的内容包含了一些基本的显示文字，还有一个intent，就是从快捷方式添加快捷方式，intent跳转到我们的activity

```java
 private static final String ACTION_ADD_WEBSITE =
            "com.example.android.appshortcuts.ADD_WEBSITE";
        if (ACTION_ADD_WEBSITE.equals(getIntent().getAction())) {
            // Invoked via the manifest shortcut.
            addWebSite();
        }
```

这边处理一下点击该快捷方式的处理逻辑

##### 动态添加快捷方式

```kotlin
 fun addShortCuts(){
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N_MR1) {
            val shortcutManager =  getSystemService<ShortcutManager>(ShortcutManager::class.java)
                val shortcut =  ShortcutInfo.Builder(ShortCutActivity@this, "id1")
                        .setShortLabel("Website")
                        .setLongLabel("Open the website")
                        .setIcon(Icon.createWithResource(ShortCutActivity@this, R.drawable.ic_launcher_foreground))
                        .setIntent(Intent(Intent.ACTION_VIEW,
                                Uri.parse("https://www.mysite.example.com/")))
                        .build()
                shortcutManager!!.dynamicShortcuts = Arrays.asList(shortcut)
        } else {

        }


    }
```





##### 固定快捷方式（pinned shortcuts）

也就是跟win一样的那种快捷方式。

```kotlin
 fun addPinnedShortCuts() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val shortcutManager = getSystemService(ShortcutManager::class.java)
            if (shortcutManager!!.isRequestPinShortcutSupported) {
                // Assumes there's already a shortcut with the ID "my-shortcut".
                // The shortcut must be enabled.
                var intent=Intent(ShortCutActivity@this,MainActivity::class.java)
                intent.action = "android.intent.action.MAIN"
                val pinShortcutInfo = ShortcutInfo.Builder(ShortCutActivity@this, "my-shortcut")
                        .setShortLabel("Hime")
                        .setLongLabel("Open the website")
                        .setIntent(intent)
                        .build()

                // Create the PendingIntent object only if your app needs to be notified
                // that the user allowed the shortcut to be pinned. Note that, if the
                // pinning operation fails, your app isn't notified. We assume here that the
                // app has implemented a method called createShortcutResultIntent() that
                // returns a broadcast intent.
                val pinnedShortcutCallbackIntent = shortcutManager.createShortcutResultIntent(pinShortcutInfo)

                // Configure the intent so that your app's broadcast receiver gets
                // the callback successfully.For details, see PendingIntent.getBroadcast().
                val successCallback = PendingIntent.getBroadcast(ShortCutActivity@this, /* request code */ 0,
                        pinnedShortcutCallbackIntent, /* flags */ 0)

                shortcutManager.requestPinShortcut(pinShortcutInfo,
                        successCallback.intentSender)
            }
        } else {

        }


    }
```



![image](https://ws4.sinaimg.cn/large/c1b251b3ly1g04taq7asjj20u01hc44p.jpg)



#### 













