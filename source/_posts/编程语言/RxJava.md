---
title: RxJava
categories: 
- 编程语言
---

**RxJava概述**

- RxJava 是一种响应式编程，来创建基于事件的异步操作库。基于事件流的链式调用、逻辑清晰简洁。
- 将事件从起点（上游）流向终点（下游），中间有很多卡片对数据进操作并传递，每个卡片获取上一个卡片传递下来的结果然后对事件进行处理然后将结果传递给下一个卡片，这样事件就从起点通过卡片一次次传递直到流向终点。

**RxJava观察者模式**

- 传统观察者是一个被观察者多过观察者，当被观察者发生改变时候及时通知所有观察者
- RXjava是一个观察者多个被观察者，被观察者像链条一样串起来，数据在被观察者之间朝着一个方向传递，直到传递给观察者 。

**RxJava原理理解**

- 被观察者通过订阅将事件按顺序依次传递给观察者。

<img src="https://img-blog.csdnimg.cn/b96e225219cd4e468462377b0957fd48.png" alt="img" style="zoom:25%;" />

**创建 Observer（观察者）**

```java
        Observer<Integer> observer = new Observer<Integer>() {
 
            // 观察者接收事件前  ，当 Observable 被订阅时，观察者onSubscribe方法会自动被调用 
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }

            // 当被观察者生产Next事件 
            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件作出响应" + value);
            }

            // 当被观察者生产Error事件 
            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            // 当被观察者生产Complete事件 
            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }
        };
       //Subscriber类 = RxJava 内置的一个实现了 Observer 的抽象类，对 Observer 接口进行了扩展 
       Subscriber<Integer> subscriber = new Subscriber<Integer>() {

           // 观察者接收事件前 ，当 Observable 被订阅时，观察者onSubscribe方法会自动被调用 
            @Override
            public void onSubscribe(Disposable d) { 
                Log.d(TAG, "开始采用subscribe连接");
            }

            // 当被观察者生产Next事件 
            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件作出响应" + value);
            }

            // 当被观察者生产Error事件 
            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            // 当被观察者生产Complete事件 
            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }
        };
```

**创建 Observable （被观察者）**

```java
        Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                // 通过 ObservableEmitter类对象产生事件并通知观察者
                // ObservableEmitter：定义需要发送的事件 & 向观察者发送事件
                     
                emitter.onNext(1);
                emitter.onComplete();
            }
        });
```

RxJava 提供了其他方法用于 创建被观察者对象Observable

```java
// 方法1：just(T...)：直接将传入的参数依次发送出来
  Observable observable = Observable.just("A", "B", "C");
  // 将会依次调用：
  // onNext("A");
  // onNext("B");
  // onNext("C");
  // onCompleted();

// 方法2：fromArray(T[]) / from(Iterable<? extends T>) : 将传入的数组 / Iterable 拆分成具体对象后，依次发送出来
  String[] words = {"A", "B", "C"};
  Observable observable = Observable.fromArray(words);
  // 将会依次调用：
  // onNext("A");
  // onNext("B");
  // onNext("C");
  // onCompleted();
```

观察者和被观察者通过subscribe订阅，订阅完成后被观察者就可以像观察者发送数据

```java
        Observable.create(new ObservableOnSubscribe<Integer>() {
       
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onComplete();
            }
        }).subscribe(new Observer<Integer>() {
   
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }
 
            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件"+ value +"作出响应"  );
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }

        });
    }
}
```