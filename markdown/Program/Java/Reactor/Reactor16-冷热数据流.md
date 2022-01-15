# Reactor16-冷热数据流

我们讨论的大部分发布者，无论是Flux还是Mono，都有一个特点：订阅前什么都不会发生。当我们“创建”了一个Flux的时候，我们只是“声明”/“组装”了它，
但是如果不调用.subscribe来订阅它，它就不会开始发出元素。

但是我们对“数据流”（尤其是乍听到这个词的时候）会有种天然的感觉，就是无论有没有订阅者，它始终在按照自己的步伐发出数据。
就像假设一个人没有一个粉丝，他也可以发微博一样。

以上这两种数据流分别称为“冷”序列和“热”序列。所以我们一直在介绍的Reactor3的发布者就属于“冷”的发布者。不过有少数的例外，
比如just生成的就是一个“热”序列，它直接在组装期就拿到数据，如果之后有谁订阅它，就重新发送数据给订阅者。
Reactor 中多数其他的“热”发布者是扩展自Processor 的（下节会介绍到）。

## 冷热数据流例子
下面我们通过对比了解一下两种不同的发布者的效果，首先是我们熟悉的“冷”发布者：
``` java
public void testCodeSequence() {
    Flux<String> source = Flux.fromIterable(Arrays.asList("blue", "green", "orange", "purple")).map(
            String::toUpperCase);
    source.subscribe(d -> System.out.println("Subscriber 1: " + d));
    System.out.println();
    source.subscribe(d -> System.out.println("Subscriber 2: " + d));
}
```

我们对发布者source进行了两次订阅，每次订阅都导致它把数据流从新发一遍，执行结果如下
```
Subscriber 1: BLUE
Subscriber 1: GREEN
Subscriber 1: ORANGE
Subscriber 1: PURPLE

Subscriber 2: BLUE
Subscriber 2: GREEN
Subscriber 2: ORANGE
Subscriber 2: PURPLE
```

然后再看一个“热”发布者的例子：
``` java
public void testHotSequence() {
    UnicastProcessor<String> hotSource = UnicastProcessor.create();
    Flux<String> hotFlux = hotSource.publish()
            .autoConnect()
            .map(String::toUpperCase);
    hotFlux.subscribe(d -> System.out.println("Subscriber 1 to Hot Source: "+d));

    hotSource.onNext("blue");
    hotSource.onNext("green");

    hotFlux.subscribe(d -> System.out.println("Subscriber 2 to Hot Source: "+d));

    hotSource.onNext("orange");
    hotSource.onNext("purple");
    hotSource.onComplete();
}
```

这个热发布者是一个UnicastProcessor，我们可以使用它的onNext等方法手动发出元素。上边的例子中，
hotSource发出两个元素后第二个订阅者才开始订阅，所以第二个订阅者只能收到之后的元素：
```
Subscriber 1 to Hot Source: BLUE
Subscriber 1 to Hot Source: GREEN
Subscriber 1 to Hot Source: ORANGE
Subscriber 2 to Hot Source: ORANGE
Subscriber 1 to Hot Source: PURPLE
Subscriber 2 to Hot Source: PURPLE
```

由此可见，UnicastProcessor是一个热发布者。

有时候，你不仅想要在某一个订阅者订阅之后才开始发出数据，可能还希望在多个订阅者“到齐”之后才开始发送数据。
ConnectableFlux的用意便在于此。Flux API 中有两种常用的返回ConnectableFlux的方式：publish和replay。

* publish会尝试满足各个不同订阅者的需求（也就是回压），并综合这些请求反馈给源。假设有某个订阅者的需求为 0，
发布者会暂停向所有订阅者发出元素。
* replay将对第一个订阅后产生的数据进行缓存，最多缓存数量取决于配置（时间/缓存大小）。它会对后续接入的订阅者重新发送数据。

ConnectableFlux提供了多种对订阅的管理方式。包括：

1. connect，当有足够的订阅接入后，可以对 flux 手动执行一次。它会触发对上游源的订阅。
2. autoConnect(n)与connect类似，不过是在有 n 个订阅的时候自动触发。
3. refCount(n)不仅能够在订阅者接入的时候自动触发，还会检测订阅者的取消动作。如果订阅者全部取消订阅，则会将源“断开连接”，
再有新的订阅者接入的时候才会继续“连上”发布者。
4. refCount(n, Duration)增加了一个倒计时：一旦订阅者数量太低了，它会等待 Duration 参数指定的时间，如果没有新的订阅者接入才会与源断开连接。

## connect的例子
``` java
public void testConnectableFlux1() throws InterruptedException {
    Flux<Integer> source = Flux.range(1, 3).doOnSubscribe(s -> System.out.println("上游收到订阅"));

    ConnectableFlux<Integer> co = source.publish();

    co.subscribe(System.out::println, e -> {
    }, () -> {
    });
    co.subscribe(System.out::println, e -> {
    }, () -> {
    });

    System.out.println("订阅者完成订阅操作");
    Thread.sleep(500);
    System.out.println("还没有连接上");

    co.connect();
}
```

输出如下，可见当connect的时候，上游才真正收到订阅请求。
```
订阅者完成订阅操作
还没有连接上
上游收到订阅
1
1
2
2
3
3
```

## autoConnect的例子
``` java
public void testConnectableFluxAutoConnect() throws InterruptedException {
    Flux<Integer> source = Flux.range(1, 3).doOnSubscribe(s -> System.out.println("上游收到订阅"));

    // 需要两个订阅者才自动连接
    Flux<Integer> autoCo = source.publish().autoConnect(2);

    autoCo.subscribe(System.out::println, e -> {}, () -> {});
    System.out.println("第一个订阅者完成订阅操作");
    Thread.sleep(500);
    System.out.println("第二个订阅者完成订阅操作");
    autoCo.subscribe(System.out::println, e -> {}, () -> {});
}
```

输出如下，可见，只有两个订阅者都完成订阅之后，上游才收到订阅请求，并开始发出数据。
```
第一个订阅者完成订阅操作
第二个订阅者完成订阅操作
上游收到订阅
1
1
2
2
3
3
```

## refCononect的例子
``` java
public void testConnectableFluxRefConnect() throws InterruptedException {
    Flux<Long> source = Flux.interval(Duration.ofMillis(500))
            .doOnSubscribe(s -> System.out.println("上游收到订阅"))
            .doOnCancel(() -> System.out.println("上游发布者断开连接"));

    Flux<Long> refCounted = source.publish().refCount(2, Duration.ofSeconds(2));

    System.out.println("第一个订阅者订阅");
    Disposable sub1 = refCounted.subscribe(l -> System.out.println("sub1: " + l));

    TimeUnit.SECONDS.sleep(1);
    System.out.println("第二个订阅者订阅");
    Disposable sub2 = refCounted.subscribe(l -> System.out.println("sub2: " + l));

    TimeUnit.SECONDS.sleep(1);
    System.out.println("第一个订阅者取消订阅");
    sub1.dispose();

    TimeUnit.SECONDS.sleep(1);
    System.out.println("第二个订阅者取消订阅");
    sub2.dispose();

    TimeUnit.SECONDS.sleep(1);
    System.out.println("第三个订阅者订阅");
    Disposable sub3 = refCounted.subscribe(l -> System.out.println("sub3: " + l));

    TimeUnit.SECONDS.sleep(1);
    System.out.println("第三个订阅者取消订阅");
    sub3.dispose();

    TimeUnit.SECONDS.sleep(3);
    System.out.println("第四个订阅者订阅");
    Disposable sub4 = refCounted.subscribe(l -> System.out.println("sub4: " + l));
    TimeUnit.SECONDS.sleep(1);
    System.out.println("第五个订阅者订阅");
    Disposable sub5 = refCounted.subscribe(l -> System.out.println("sub5: " + l));
    TimeUnit.SECONDS.sleep(2);
}
```

输出如下：
```
第一个订阅者订阅
第二个订阅者订阅
上游收到订阅
sub1: 0
sub2: 0
第一个订阅者取消订阅
sub1: 1
sub2: 1
sub2: 2
第二个订阅者取消订阅
第三个订阅者订阅
sub3: 3
sub3: 4
sub3: 5
sub3: 6
sub3: 7
第三个订阅者取消订阅
上游发布者断开连接
第四个订阅者订阅
第五个订阅者订阅
上游收到订阅
sub4: 0
sub5: 0
sub4: 1
sub5: 1
sub4: 2
sub5: 2
sub4: 3
sub5: 3
```

这里详细解释下执行过程。

本例中，refCount设置为最少两个订阅者接入是才开始发出数据，当所有订阅者都取消时，如果不能在两秒内接入新的订阅者，则上游会断开连接。

上边的例子中，随着前两个订阅者相继取消订阅，第三个订阅者及时（在2秒内）开始订阅，所以上游会继续发出数据，而且根据输出可以看出是“热序列”。

当第三个订阅者取消后，第四个订阅者没能及时开始订阅，所以上游发布者断开连接。当第五个订阅者订阅之后，第四和第五个订阅者相当于开始了新一轮的订阅。
