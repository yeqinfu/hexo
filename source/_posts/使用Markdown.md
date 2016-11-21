---
title: 使用Markdown
date: 2016-11-21 13:31:56
tags: Markdown
comments: true

---





### 使用Markdown

* Markdown可以内嵌html，比如下面的图片标签

<img src="http://img2.selfimg.com.cn/uedselfcms/2016/11/14/1479113934_VVV6k9.jpg"/>



* `&copy`写一个符号 ©

* 列表符号，`*`代表无序列表，`1.`代表有序列表,符号要和文字有一个字符空格

* 引用符号“>”

  > 这是引用符号效果

* 图片和链接

  > 图片为：`![](){ImgCap}{/ImgCap}`
  >
  > 链接为：`[]()`

  图片：![无脸男](http://img4.duitang.com/uploads/item/201402/25/20140225171138_54uGn.thumb.600_0.jpeg)

  链接：[baidu](www.baidu.com)

* 粗体和斜体 

  > 用两个 `*` 包含一段文本就是粗体的语法，用一个 `*` 包含一段文本就是斜体的语法。`_`也可以

  斜体：*这是斜体文字* 

  斜体：**这是粗体文字**

* 表格不学

* 代码引用，1左边的按钮

  ```java
  @AfterViews
  void afterViews() {
     ad_CouponList = new AD_CouponList(getActivity());
     lv_coupon.setAdapter(ad_CouponList);
     lv_coupon.setPullRefreshEnable(false);
     lv_coupon.setPullLoadEnable(true);
     lv_coupon.setAutoLoadEnable(true);
     lv_coupon.setXListViewListener(this);
     notifyDataChanged();
     unusedpage = 1;
     usedpage = 1;
     overduepage = 1;
     pagesize = 10;
     lv_coupon.setNoMoreData(false);
     Utils_Dialog.showProgressDialog(getActivity());
     loadData();
  }
  ```

* 分割线`---`

  ---

* 完结

  ​

  ​

  ​

  ​

  ​

  ​













