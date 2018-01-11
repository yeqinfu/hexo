---
title: ToastUtils的完美单例
date: 2018-01-11 14:00:58
tags: code
---

有点经验的人会想到Toast要写成工具类，单例模式。这样就不会用户一直点一直点，然后不点了，然后后面toast还在一直弹一直弹，弹到前面点的相应次数为止。这个很恶心。所以网上随便看看Toast的单例模式有人会这么写，甚至说我项目里面的Toast工具也是这么写的

```java
public class ToastUtils {  
      
    /** 之前显示的内容 */  
    private static String oldMsg ;  
    /** Toast对象 */  
    private static Toast toast = null ;  
    /** 第一次时间 */  
    private static long oneTime = 0 ;  
    /** 第二次时间 */  
    private static long twoTime = 0 ;  
      
    /** 
     * 显示Toast 
     * @param context 
     * @param message 
     */  
    public static void showToast(Context context,String message){  
        if(toast == null){  
            toast = Toast.makeText(context, message, Toast.LENGTH_SHORT);  
            toast.show() ;  
            oneTime = System.currentTimeMillis() ;  
        }else{  
            twoTime = System.currentTimeMillis() ;  
            if(message.equals(oldMsg)){  
                if(twoTime - oneTime > Toast.LENGTH_SHORT){  
                    toast.show() ;  
                }  
            }else{  
                oldMsg = message ;  
                toast.setText(message) ;  
                toast.show() ;  
            }  
        }  
        oneTime = twoTime ;  
    }
```

很多写法大同小异，都是用到单例和静态变量。问题在于，它们有一个共同问题，如上代码所示，toast这个变量是静态的，不被回收，那么context对象作为toast的私有变量也得不到回收。所以会造成始终有一个Activity回收不了，而回收不了的那个Activity就是第一次生成toast实例的那个activity。

那么既要保持单例模式，又要做到activity的及时回收，这怎么弄？

直接贴我的代码

```java
/*
 * Created by yeqinfu on 17-9-27 上午9:35
 * Copyright (c) JXT All rights reserved.
 */

package com.ppandroid.app.utils.toast;

import android.app.Activity;
import android.app.Application;
import android.os.Bundle;
import android.os.Handler;
import android.text.TextUtils;
import android.widget.Toast;

import com.ppandroid.app.base.SampleApplicationLike;
import com.ppandroid.app.utils.DebugLog;
import com.ppandroid.app.utils.activitymanager.ActivityManager;

/**
 * 既要保持单例模式，又要保证静态变量toast内部中的context能够及时回收？？
 * 这边是有问题的，toast这个私有变量是intance静态变量的成员变量，instance不回收
 * toast也就不回收，toast不回收，context也就不回收。导致AC_Login这个页面第一次
 * 用到toast之后，生成instance对象之后，AC_Login的上下文永远被instance持有，所以 一直都有AC_Login内存泄漏的提示。
 * 要保持单例又要回收两者不可得兼
 * ==============================以上是针对历史版本代码的单例Toast问题的解释==============================
 * 要保持单例防止多次toast，应该是保持页面实例内的单例，而不是全局单例
 * 而现在改进的写法，保证下次toast的时候mContext对象会被替换成
 */
public class ToastUtilAdapter {
	private static ToastUtilAdapter	instance;

	private Toast					toast;
	private Handler					handler;

	private ToastUtilAdapter() {
		handler = new Handler();
	}

	public static ToastUtilAdapter getInstance() {
		if (instance == null) {
			synchronized (ToastUtilAdapter.class) {
				if (instance == null) {
					instance = new ToastUtilAdapter();
				}
			}
		}
		return instance;
	}

	private Activity mContext;
	public void toast(final String toastMsg) {
		if (TextUtils.isEmpty(toastMsg)) {
			return;
		}
		handler.post(new Runnable() {
			@Override
			public void run() {
			    if (mContext==null){
			        mContext=getContext();
                    toast = Toast.makeText(mContext, toastMsg, Toast.LENGTH_SHORT);
                }else{
			        if (ActivityManager.getActivityManager().isInStack(mContext)){//如果上次保留的mContext还在栈内
                        toast.setText(toastMsg);
                        toast.setDuration(Toast.LENGTH_SHORT);
                    }else{//不在栈内移除引用从新new一个toast对象
                        mContext=getContext();
                        toast = Toast.makeText(mContext, toastMsg, Toast.LENGTH_SHORT);
                    }
                }

				/*	
				这个是旧代码的单例写法，会导致第一个使用的Activity如AC_Login上下文泄漏
				if (toast == null) {
						 toast = Toast.makeText(getContext(), toastMsg, Toast.LENGTH_SHORT);
					}
					else {
						toast.setText(toastMsg);
						toast.setDuration(Toast.LENGTH_SHORT);
					}*/
                DebugLog.d("======================="+mContext);
				toast.show();
			}
		});
	}

	/**
	 * 这边改成获取当前栈顶的activity对象作为上下文
	 * 增加一个activity的监听，如果当前的mContext被回收，则及时释放引用
	 * @return
	 */
	private Activity getContext() {
	    Activity current=ActivityManager.getActivityManager().currentActivity();
        SampleApplicationLike.getContext().registerActivityLifecycleCallbacks(new Application.ActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle bundle) {

            }

            @Override
            public void onActivityStarted(Activity activity) {

            }

            @Override
            public void onActivityResumed(Activity activity) {

            }

            @Override
            public void onActivityPaused(Activity activity) {

            }

            @Override
            public void onActivityStopped(Activity activity) {

            }

            @Override
            public void onActivitySaveInstanceState(Activity activity, Bundle bundle) {

            }

            @Override
            public void onActivityDestroyed(Activity activity) {
                if (activity==mContext){
                    mContext=null;
                    DebugLog.d("====================释放==============="+activity);
                }

            }
        });
		return current;
	}

	public void toast(int resId) {

		toast(getContext().getResources().getString(resId));
	}

}

```

如代码所示，mContext对象一直是作为Toast的上下文对象，但是加了一个ActivityLifecycleCallbacks，当对象呗销毁的时候，及时释放mContext这个强引用。

而在mContext为空的时候，从一个维护Activtiy的Stack取出栈顶的Activity作为Toast的上下文对象。核心就是及时释放这个单例的mContext对象。也就是Toast它不应该是一个Application级别的全局单例，而是作为一个页面activity级别的单例而存在。

另外，网络上的getaplicationcontext()这样的Context作为Toast的上下文是不可行的，好像跟window token有关。













