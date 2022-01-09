# Reactor15-自定义数据流

这一小节介绍如何通过定义相应的事件（onNext、onError和onComplete） 创建一个 Flux 或 Mono。
Reactor提供了generate、create、push和handle等方法，所有这些方法都使用 sink（池）来生成数据流。

sink，顾名思义，就是池子，可以想象一下厨房水池的样子。

下面介绍到的方法都有一个sink提供给方法使用者，通常至少会暴露三个方法给我们，next、error和complete。
next和error相当于两个下水口，我们不断将自定义的数据放到next口，Reactor就会帮我们串成一个Publisher数据流，
直到有一个错误数据放到error口，或按了一下complete按钮，数据流就会终止了。

## generate

generate是一种同步地，逐个地发出数据的方法。因为它提供的sink是一个SynchronousSink，
而且其next()方法在每次回调的时候最多只能被调用一次。

generate方法有三种签名：
``` java
public static <T> Flux<T> generate(Consumer<SynchronousSink<T>> generator)

public static <T, S> Flux<T> generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator) 

public static <T, S> Flux<T> generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator, Consumer<? super S> stateConsumer)
```

**1）使用SynchronousSink生成数据流**

``` java
public void testGenerate1() {
    final AtomicInteger count = new AtomicInteger(1);   // 1
    Flux.generate(sink -> {
        sink.next(count.get() + " : " + new Date());   // 2
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        if (count.getAndIncrement() >= 5) {
            sink.complete();     // 3
        }
    }).subscribe(System.out::println);  // 4
}
```

1. 用于计数；
2. 向“池子”放自定义的数据；
3. 告诉generate方法，自定义数据已发完；
4. 触发数据流。

输出结果为每1秒钟打印一下时间，共打印5次。

**2）增加一个伴随状态**

对于上边的例子来说，count用于记录状态，当值达到5之后就停止计数。由于在lambda内部使用，
因此必须是final类型的，且不能是原生类型（如int）或不可变类型（如Integer）。

如果使用第二个方法签名，上边的例子可以这样改：

``` java
public void testGenerate2() {
    Flux.generate(
            () -> 1,    // 1
            (count, sink) -> {      // 2
                sink.next(count + " : " + new Date());
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if (count >= 5) {
                    sink.complete();
                }
                return count + 1;   // 3
            }).subscribe(System.out::println);
}
```

1. 初始化状态值；
2. 第二个参数是BiFunction，输入为状态和sink；
3. 每次循环都要返回新的状态值给下次使用。

**3）完成后处理**

第三个方法签名除了状态、sink外，还有一个Consumer，这个Consumer在数据流发完后执行。
``` java
public void testGenerate3() {
    Flux.generate(
            () -> 1,
            (count, sink) -> {
                sink.next(count + " : " + new Date());
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if (count >= 5) {
                    sink.complete();
                }
                return count + 1;
            }, System.out::println)     // 1
            .subscribe(System.out::println);
}
```

1. 最后将count值打印出来。

如果 state 使用了数据库连接或者其他需要进行清理的资源，这个 Consumer lambda 可以用来在最后完成资源清理任务。

## create

create是一个更高级的创建Flux的方法，其生成数据流的方式既可以是同步的，也可以是异步的，并且还可以每次发出多个元素。

create用到了FluxSink，后者同样提供 next，error 和 complete 等方法。与generate不同的是，create不需要状态值，
另一方面，它可以在回调中触发多个事件（即使事件是发生在未来的某个时间）。

> [!NOTE]
> create 常用的场景就是将现有的 API 转为响应式，比如监听器的异步方法。

先编写一个事件源：
``` java
public class MyEventSource {

    private List<MyEventListener> listeners;

    public MyEventSource() {
        this.listeners = new ArrayList<>();
    }

    public void register(MyEventListener listener) {    // 1
        listeners.add(listener);
    }

    public void newEvent(MyEvent event) {
        for (MyEventListener listener : listeners) {
            listener.onNewEvent(event);     // 2
        }
    }

    public void eventStopped() {
        for (MyEventListener listener : listeners) {
            listener.onEventStopped();      // 3
        }
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class MyEvent {   // 4
        private Date timeStemp;
        private String message;
    }
}
```

1. 注册监听器；
2. 向监听器发出新事件；
3. 告诉监听器事件源已停止；
4. 事件类，使用了lombok注解。

准备一个监听器接口，它可以监听上边第2和3的两种事件：（1）新的MyEvent到来；（2）事件源停止。如下：
``` java
public interface MyEventListener {
    void onNewEvent(MyEventSource.MyEvent event);
    void onEventStopped();
}
```
下面的测试方法逻辑是：创建一个监听器注册到事件源，这个监听器收到事件回调的时候通过Flux.create的sink将一系列事件转换成异步的事件流：
``` java
public void testCreate() throws InterruptedException {
    MyEventSource eventSource = new MyEventSource();    // 1
    Flux.create(sink -> {
                eventSource.register(new MyEventListener() {    // 2
                    @Override
                    public void onNewEvent(MyEventSource.MyEvent event) {
                        sink.next(event);       // 3
                    }

                    @Override
                    public void onEventStopped() {
                        sink.complete();        // 4
                    }
                });
            }
    ).subscribe(System.out::println);       // 5

    for (int i = 0; i < 20; i++) {  // 6
        Random random = new Random();
        TimeUnit.MILLISECONDS.sleep(random.nextInt(1000));
        eventSource.newEvent(new MyEventSource.MyEvent(new Date(), "Event-" + i));
    }
    eventSource.eventStopped(); // 7
}
```

1. 事件源；
2. 向事件源注册用匿名内部类创建的监听器；
3. 监听器在收到事件回调的时候通过sink将事件再发出；
4. 监听器再收到事件源停止的回调的时候通过sink发出完成信号；
5. 触发订阅（这时候还没有任何事件产生）；
6. 循环产生20个事件，每个间隔不超过1秒的随机时间；
7. 最后停止事件源。

运行一下这个测试方法，20个MyEvent陆续打印出来。

如果将上边的create方法换成generate方法，则会报出异常：
```
java.lang.IllegalStateException: The generator didn't call any of the SynchronousSink method
```
证明generate并不支持异步的方式，也就是说generate必须提前把要的数据准备好后同步的发出去。不能等到将来某个不确定时刻才知道数据是个啥。

create方法还有一个变体方法push，适合生成事件流。与 create类似，push 也可以是异步地， 并且能够使用以上各种回压策略。
所以上边的例子可以替换为push方法。区别在于，push方法中，调用next、complete或error的必须是同一个线程。不再演示。

除了next、complete或error方法外，FluxSink还有onRequest方法，这个方法可以用来响应下游订阅者的请求事件。从而不仅可以像上一个例子那样，
上游在数据就绪的时候将其推送到下游，同时下游也可以从上游拉取已经就绪的数据。这是一种推送/拉取混合的模式。比如：
``` java
public void testOnRequest() {
    Flux<String> bridge = Flux.create(sink -> {
        myMessageProcessor.register(
                new MyMessageListener<String>() {
                    public void onMessage(List<String> messages) {
                        for (String s : messages) {
                            sink.next(s);   // 1
                        }
                    }
                });
        sink.onRequest(n -> {   // 2
            List<String> messages = myMessageProcessor.request(n);  // 3
            for (String s : message) {
                sink.next(s);
            }
        });
    });
}
```

1. push方式，主动向下游发出数据；
2. 在下游发出请求时被调用；
3. 响应下游的请求，查询是否有可用的message。

## 页面持续输出
有时候需要将后台的事件持续输出到页面来做监控，比如日志、容器重启事件等等，这种常见正是响应式接口大显身手的时候。

比如我要将上面create生成的自定义事件输出到页面持续监控。

先定义一个启动类，用于产生这个事件流
```java
@Component
public class FluxEventRunner implements CommandLineRunner {
    private Flux<MyEventSource.MyEvent> data;

    public Flux<MyEventSource.MyEvent> getData() {
        return data;
    }

    @Override
    public void run(String... args) throws Exception {
        MyEventSource eventSource = new MyEventSource();
        data = Flux.create(sink -> {
                    eventSource.register(new MyEventListener() {
                        @Override
                        public void onNewEvent(MyEventSource.MyEvent event) {
                            sink.next(event);
                        }
                        @Override
                        public void onEventStopped() {
                            sink.complete();
                        }
                    });
                }
        );

        for (int i = 0; i < 20; i++) {
            Random random = new Random();
            TimeUnit.MILLISECONDS.sleep(random.nextInt(1000));
            eventSource.newEvent(new MyEventSource.MyEvent(new Date(), "Event-" + i));
        }
        eventSource.eventStopped();
    }
}
```

然后定义API接口，注意只需要将返回类型定义为`APPLICATION_STREAM_JSON_VALUE`即可。
```java
@RestController
@RequestMapping(value = "/events", produces = MediaType.APPLICATION_STREAM_JSON_VALUE)
public class EventStreamController {
    @Autowired
    private FluxEventRunner fluxEventRunner;

    @GetMapping
    public Flux<MyEventSource.MyEvent> stream() {  // 2
        return fluxEventRunner.getData();
    }
}
```
页面即可持续打印。请注意，订阅开始是发生在你访问这个接口的时候，通过异步方式产生的数据流在未订阅的时候是不会产生数据的。
这个跟同步方式（Flux.just或者Flux.generate）生成的Flux要有区别的。并且同步生成的有个问题是Controller接口要等到这个Flux结束后才有返回。
也就是一次性将所有数据都返回回去，达不到动态显示每一条的效果。所以这种场景只能用异步方式。

