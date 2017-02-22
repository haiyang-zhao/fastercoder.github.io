title: RxJava探究（一）RxJava初体验
date: 2016-05-02 22:15:47
categories: RxJava探究
tags: RxJava

---

## RxJava是什么
最近Android社区中，RxJava可算是非常火，以至于你不会RxJava就不好意思说你是一个Android开发工程师，本着学习的态度，来体验一下RxJava。

RxJava是一种响应式编程，基于观察者模式（于观察者模式稍稍有差别）在处理异步回调时有着得天独厚的优势，在处理复杂的列表过滤、变化、转换等时，RxJava为我们提供了全新的思想。


## 如何使用RxJava

### Gradle中引入

```java
compile 'io.reactivex:rxjava:1.1.0' 
```
	
### 创建被观察者、事件源Observable

```Java
   //被观察者，事件源
   Observable<String> myObservable = Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("Hello world!");
                subscriber.onCompleted();
            }
        });
```
定义Observable对象myObservable,只是传达一个“Hello world!”字符串就OK了，下面我们创建Subscriber来处理myObservable发出的"Hello world!"
### 创建订阅者、观察者对象Subscriber

```Java
//订阅者
Subscriber<String> mySubscriber = new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                System.out.println(s);
            }
        };
```
定义Subscriber对象mySubscriber，打印myObservable对象发出的字符串"Hellow world!"
### 关联观察者和被观察者，即就是让被观察者订阅观察者对象
通过subscribe方法建立观察者和被观察者之间的联系

```Java
myObservable.subscribe(mySubscriber);
```

关联后程序就会打印"Hellow world!"

### 简化代码

RxJava中提供多种创建Observale对象的方法，上面我们的Observale对象仅仅是发出了一个字符串就结束了，那么我们可以用Observable.just()方法。
Observable.just(): Returns an Observable that emits a single item and then completes. 返回发出单一事件就结束的Observale对象。

```Java
Observable<String> myObservable = Observable.just("Hello world!");
```

上面的例子中Subscriber对象只关心onNext()方法，所以可以用Action1类来处理

```java
   //如果只需要在onNext处理,可以用Action1
        Action1<String> onNextAction = new Action1<String>() {
            @Override
            public void call(String s) {
                System.out.println(s);
            }
        };
```
subscribe方法有一个重载版本，接受三个Action1类型的参数，分别对应OnNext，OnComplete， OnError函数。

```java
  myObservable.subscribe(onNextAction, onErrorAction, onCompleteAction);
```

我们只关心onNext,所以上面的代码可以简写为
```java
       //just用来创建只发出一个事件就结束的Observable对象
        Observable<String> myObservable = Observable.just("Hello world!");
        //如果只需要在onNext处理,可以用Action1
        Action1<String> onNextAction = new Action1<String>() {
            @Override
            public void call(String s) {
                System.out.println(s);
            }
        };
        myObservable.subscribe(onNextAction);
```

最终的代码可以是
```java
      //上面可以简写为
        Observable.just("Hello world!")
                .subscribe(new Action1<String>() {
                    @Override
                    public void call(String s) {
                        System.out.println(s);
                    }
                });
```




    
