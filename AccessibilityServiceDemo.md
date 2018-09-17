---
title: Accessibility Service Demo
date: 2018-09-07 09:00:00
tags: code
---

### 无障碍服务

添加权限

> ```
> <uses-permission android:name="android.permission.BIND_ACCESSIBILITY_SERVICE"/>
> ```

添加service的相关声明

```java
  <service
            android:name=".AccessibilityDemoService"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE"
            >
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService"/>
            </intent-filter>
        </service>
```




































