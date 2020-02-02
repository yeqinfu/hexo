---
title: rxjava基本使用
date: 2020-02-01 14:00:58
tags: code
---



## 认识一些操作符

### CompositeDisposable

> 用来收集多个disposables 对象，在activity fragment生命周期结束的时候调用 clear方法进行快速批量取消订阅以防内存泄漏

```java
     disposables.add(sampleObservable()
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeWith(new DisposableObserver<String>() {
                    @Override
                    public void onComplete() {
                        textView.append(" onComplete");
                        textView.append(AppConstant.LINE_SEPARATOR);
                        Log.d(TAG, " onComplete");
                    }

                    @Override
                    public void onError(Throwable e) {
                        textView.append(" onError : " + e.getMessage());
                        textView.append(AppConstant.LINE_SEPARATOR);
                        Log.d(TAG, " onError : " + e.getMessage());
                    }

                    @Override
                    public void onNext(String value) {
                        textView.append(" onNext : value : " + value);
                        textView.append(AppConstant.LINE_SEPARATOR);
                        Log.d(TAG, " onNext value : " + value);
                    }
                }));
                
                 @Override
    protected void onDestroy() {
        super.onDestroy();
        disposables.clear(); // do not send event after activity has been destroyed
    }
```

显然这种方式管理事件流太low了，add这个方法失去了链式调用的意义

### Single Completable Maybe的使用场景具体用例？

> Single 只发射一条单一的数据，或者错误通知，不能发射完成状态，数据和通知只能发射一个
>
> completable 发射完成或者错误的通知不能发射数据
>
> maybe  发射单一数据或者错误完成通知。完成和异常互斥

### Flowable 响应式拉取在订阅对象设定拉取数量

```java
                    public void onSubscribe(Subscription s) {
                        s.request(2);//设置Subscriber的消费能力为2
                    }
```



它的最佳实践？

异步背压缓冲池有限制导致会抛出一些异常，为的是解决上游发射数据过快导致内存异常，而背压的缓存策略会有一些是导致上游数据丢失的情况。

```java
public void demo18() {
        Flowable
                .create(new FlowableOnSubscribe<Integer>() {
                    @Override
                    public void subscribe(FlowableEmitter<Integer> e) throws Exception {
                        int i = 0;
                        while (true) {
                            if (e.requested() == 0) continue;//此处添加代码，让flowable按需发送数据
                            System.out.println("发射---->" + i);
                            i++;
                            e.onNext(i);
                        }
                    }
                }, BackpressureStrategy.MISSING)
                .subscribeOn(Schedulers.newThread())
                .observeOn(Schedulers.newThread())
                .subscribe(new Subscriber<Integer>() {
                    private Subscription mSubscription;

                    @Override
                    public void onSubscribe(Subscription s) {
                        s.request(1);            //设置初始请求数据量为1
                        mSubscription = s;
                    }

                    @Override
                    public void onNext(Integer integer) {
                        try {
                            Thread.sleep(50);
                            System.out.println("接收------>" + integer);
                            mSubscription.request(1);//每接收到一条数据增加一条请求量
                        } catch (InterruptedException ignore) {
                        }
                    }

                    @Override
                    public void onError(Throwable t) {
                    }

                    @Override
                    public void onComplete() {
                    }
                });
    }
```



此代码为数据流可控的情况下，解决上游数据发射过快，背压缓冲爆掉的问题，通过拉取订阅按需控制上游数据来解决问题，而如果上游发射数据不可控，又不希望内存溢出，又希望数据丢失，那要啥策略？（我估计不可能）

### switchmap 可以使用在首页搜索功能

### takewhile 使用在token刷新的功能上

## rxjava原理

![备注 2020年2月2日_0](https://tva3.sinaimg.cn/large/c1b251b3gy1gbi54iode5j214abl44qr.jpg)

