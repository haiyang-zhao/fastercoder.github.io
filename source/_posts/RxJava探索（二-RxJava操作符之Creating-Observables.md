title: RxJava探究（二） RxJava操作符之Creating Observables
date: 2016-07-02 12:44:59
categories: RxJava探究
tags: Rxjava
---

RxJava的强大之处就是他提供非常丰富的操作符，超过上百种，这些操作符给我们解决问题带来很多方便。本文主要讲解创建Observale对象一类的操作符，参照官方文档 <http://reactivex.io/documentation/operators.html#creating>。

### Create
Observable.create()方法用于创建一个Observale对象，该方法接收一个Observer观察者对象。

![](http://reactivex.io/documentation/operators/images/create.c.png)

```java
	Observable.create(new Observable.OnSubscribe<Integer>() {
            @Override
            public void call(Subscriber<? super Integer> subscriber) {
                try {
                    if (!subscriber.isUnsubscribed()) {
                        for (int i = 1; i < 5; i++) {
                            subscriber.onNext(i);
                        }

                        subscriber.onCompleted();
                    }
                } catch (Exception e) {

                    subscriber.onError(e);
                }
            }
        }).subscribe(new Subscriber<Integer>() {
            @Override
            public void onCompleted() {
                System.out.println("completed !");

            }

            @Override
            public void onError(Throwable e) {
                System.out.println("onError " + e.getMessage());
            }

            @Override
            public void onNext(Integer item) {
                System.out.println("Next " + item);

            }
        });
```
subscrible(subscriber)方法调用后会执行OnSubscribe对象的call(subscriber)方法，从而完成回调。
打印结果如下：

```java
Next 1
Next 2
Next 3
Next 4
completed !
```

### Defer 
观察者订阅时创建被观察者对象，对于每个观察者都会创建一个新的被观察者对象,也就是说在观察者订阅之前不创建这个Observable，为每一个观察者创建一个新的Observable。

![](http://reactivex.io/documentation/operators/images/defer.c.png)

如下代码：

```java
class SomeType {
        private String value;

        public void setValue(String value) {
            this.value = value;
        }

        public Observable<String> valueObservable() {
            return Observable.just(value);
        }


    }

    @Test
    public void defer_just() {

        SomeType someType = new SomeType();
        Observable<String> observable1 = someType.valueObservable();
        someType.setValue("Some Value");
        observable1.subscribe(new Action1<String>() {
            @Override
            public void call(String value) {
                System.out.println(value);
            }
        });
    }
```
程序输出是"Some Value" 而不是null

### Just

   将某个对象转化为Observable对象，可以使一个数字、一个字符串、数组、Iterate对象等，是一种非常快捷的创建Observable对象的方法,和Defer比较相似
   ![](http://reactivex.io/documentation/operators/images/just.c.png)
   
代码如下：
```java
class SomeType {
        private String value;

        public void setValue(String value) {
            this.value = value;
        }

        public Observable<String> valueObservableFromJust() {
            return Observable.just(value);
        }

        public Observable<String> valueObservableFromDefer() {
            return Observable.defer(new Func0<Observable<String>>() {
                @Override
                public Observable<String> call() {
                    return Observable.just(value);
                }
            });
        }
    }

    @Test
    public void defer_just() {


        SomeType someType = new SomeType();
        Observable<String> observable1 = someType.valueObservableFromJust();
        Observable<String> observable2 = someType.valueObservableFromDefer();
        someType.setValue("Some Value");
        observable1.subscribe(new Action1<String>() {
            @Override
            public void call(String value) {
                System.out.println(value);
            }
        });

        observable2.subscribe(new Action1<String>() {
            @Override
            public void call(String value) {
                System.out.println(value);
            }
        });

    }
```
程序输出如下：
```java
null
Some Value
```
由此可知，just和订阅无关，创建时是什么状态订阅时也是什么状态。

### From
类似Just,可以把一个Future, Iterable或者Array类型转换为Observable对象。

![](http://reactivex.io/documentation/operators/images/from.c.png)

```java

    @Test
    public void from() {
        Observable.from(Arrays.asList(1, 2, 3, 4, 5))
                .subscribe(new Action1<Integer>() {
                    @Override
                    public void call(Integer item) {
                        System.out.println(item);
                    }
                });

		//简化为
        Observable.just(1, 2, 3, 4, 5).subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer item) {
                System.out.println(item);
            }
        });

    }
```
输出结果为：
```java
1
2
3
4
5
1
2
3
4
5
```

