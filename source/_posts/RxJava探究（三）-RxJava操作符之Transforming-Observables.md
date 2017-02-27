title: RxJava探究（三） RxJava操作符之Transforming Observables
date: 2016-07-03 10:54:50
categories: RxJava探究
tags: RxJava
---

Transform类操作符，可以转换由一个Observable对象发出的对象。

## Buffer
顾名思义缓存的意思

### Observable<List<T>> buffer(int count, int skip)
每隔skip长度后，发出size为conut的Observable

![](http://reactivex.io/documentation/operators/images/buffer4.png)

```java
 Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9).buffer(2,4)
 .subscribe(new Action1<List<Integer>>() {
            @Override
            public void call(List<Integer> i) {
                System.out.println("buffer " + i);
            }
        });
```

每隔4个数，输出一个长度为2的集合，输出为：
```
buffer [1, 2]
buffer [5, 6]
buffer [9]
```
<!--more-->
### Observable<List<T>> buffer(long timespan, TimeUnit unit)

周期性的订阅
```java
Observable.create(new Observable.OnSubscribe<String>() {  
           @Override  
           public void call(Subscriber<? super String> subscriber) {  
               if (subscriber.isUnsubscribed()) return;  
               while (true) {  
                   subscriber.onNext("消息" + SystemClock.elapsedRealtime());  
                   SystemClock.sleep(2000);//每隔2s发送消息  
               }  
  
           }  
       }).subscribeOn(Schedulers.io()).  
               buffer(3, TimeUnit.SECONDS).//每隔3秒 取出消息  
               subscribe(new Observer<List<String>>() {  
           @Override  
           public void onCompleted() {  
               LogUtils.d("-----------------onCompleted:");  
           }  
  
           @Override  
           public void onError(Throwable e) {  
               LogUtils.d("----------------->onError:");  
           }  
  
           @Override  
           public void onNext(List<String> strings) {  
               LogUtils.d("----------------->onNext:" + strings);  
           }  
       });  
```

程序输出如下：

```
onNext:[消息370507667, 消息370509668]
onNext:[消息370511668]
onNext:[消息370513669, 消息370515669]
onNext:[消息370529168, 消息370531172]
onNext:[消息370533184]
onNext:[消息370535184, 消息370537184]
onNext:[消息370539184]
onNext:[消息370541185, 消息370543204]
onNext:[消息370545204]
onNext:[消息370547204, 消息370549204]
onNext:[消息370551204]

```

