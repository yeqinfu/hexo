---
title: 记一次fragment内存泄漏
date: 2019-04-23 14:00:58
tags: android
---

项目中用到一个activity多个fragment的模式，其中涉及到部分fragment每次关闭都不能得到释放。这里记录解决的过程要点。内存溢出的情况如下

![image](https://ws2.sinaimg.cn/large/c1b251b3gy1g2cvvcrok4j20cc0beab7.jpg)

大体可以看到这个webfragment没有得到释放。用as的自带内存检测工具去查看内存堆是否有这个对象。先手动GC一下后把内存信息dump下来

![image](https://wx4.sinaimg.cn/large/c1b251b3gy1g2cw7kotesj20dq0kfwfd.jpg)

可以看到这个类确实还在内存中，并且有多个地方引用它，所以得不到释放。从第一张leakcanary看出相关方法可以看到createmenu方法比较可疑。因为这个方法导致了mainactivity一直持有这个fragment的引用。因为这个webviewfragment用到了toolbar并且设置了menu。

![image](https://wx2.sinaimg.cn/large/c1b251b3gy1g2cwhrtfqyj20fq09qt94.jpg)

确实是应为这个fragmentmanager一直引用webfragment导致的。

```
ArrayList<Fragment> mCreatedMenus;
```

在fragmentmanager中有一个数组强引用着调用创建menu的fragment。并且没有释放。

```
public boolean dispatchCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        if (this.mCurState < 1) {
            return false;
        } else {
            boolean show = false;
            ArrayList<Fragment> newMenus = null;

            int i;
            Fragment f;
            for(i = 0; i < this.mAdded.size(); ++i) {
                f = (Fragment)this.mAdded.get(i);
                if (f != null && f.performCreateOptionsMenu(menu, inflater)) {
                    show = true;
                    if (newMenus == null) {
                        newMenus = new ArrayList();
                    }

                    newMenus.add(f);
                }
            }

            if (this.mCreatedMenus != null) {
                for(i = 0; i < this.mCreatedMenus.size(); ++i) {
                    f = (Fragment)this.mCreatedMenus.get(i);
                    if (newMenus == null || !newMenus.contains(f)) {
                        f.onDestroyOptionsMenu();
                    }
                }
            }

            this.mCreatedMenus = newMenus;
            return show;
        }
    }
```

在fragmentmanager中，这段代码增加了webfragment的引用。所以在fragment销毁的时候调用这个方法一次应该就可以去掉引用。

> 值得注意的是，这个方法是递归调用就是如果在webframent之上如果还有fragment，而这个fragment如果也有menu那么webfragment也会得不到释放。而现在暂时没有这个情况。

在第一张图中MainActivity.mFragments这个变量在FragmentActivity中。

```
  public boolean onCreatePanelMenu(int featureId, Menu menu) {
        if (featureId == 0) {
            boolean show = super.onCreatePanelMenu(featureId, menu);
            show |= this.mFragments.dispatchCreateOptionsMenu(menu, this.getMenuInflater());
            return show;
        } else {
            return super.onCreatePanelMenu(featureId, menu);
        }
    }
```

FragmentActivity父类中有调用该方法。所以在fragment销毁的时候调用

```
 ( getActivity()).onCreatePanelMenu(0,null);
```

传0调用了dispatchCreateOptionsMenu方法。menu传null去掉menu。再次用as去dump内存对象就没有发现这个webfragment在内存之中了。





































