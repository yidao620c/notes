# Reactor06-处理错误

数据在map等方法处理的过程中有可能会出现一些异常情况，针对异常的处理一般会有如下几种处理方式：

* 场景1: 捕获，并返回一个静态的默认值。
* 场景2: 捕获，并执行一个异常处理方法，计算新的数据来顶替。
* 场景3: 捕获，不提供异常处理方法，直接动态计算一个候补值来顶替。
* 场景4: 捕获，记录错误日志，然后跳过这个异常后继续。
* 场景5: 捕获，记录错误日志，然后抛出该异常。
* 场景6: 捕获，并再包装为某一个业务相关的异常，然后再抛出。
* 场景7: 使用 finally 来清理资源。
* 场景8: 使用 Java 7 引入的 `try-with-resource`自动清理机制。

## 返回默认值

最简单的遇到错误，直接替换成默认值
``` java
public void showOnErrorReturn1(int val) {
    Flux<String> flux = Flux.just(10, val, 30)
            .map(this::convert1)
            .onErrorReturn("RECOVERED");
    flux.subscribe(System.out::println);
}
```

还可以选择对异常进行 Predicate,以决定是否要恢复,如以下示例所示，仅在异常消息以 "20" 结尾时响应默认值。其他时候仍然跑出异常

``` java
public void showOnErrorReturn2(int val) {
    Flux<String> flux = Flux.just(val, 30)
            .map(this::convert1)
            .onErrorReturn(e -> e.getMessage().endsWith("20"), "recovered20");
    flux.subscribe(System.out::println);
}
```

## 异常处理方法计算新的值

如果你不只是想要在发生错误的时候给出默认值,而是希望提供一种更安全的处理数据的方式,可以使用 onErrorResume。

``` java
public void showOnErrorResume(int val) {
    Flux<String> flux = Flux.just(val, 30)
            .map(this::convert1)
            .onErrorResume(e -> getFromCache(val));
    flux.subscribe(System.out::println);
}
```

## 动态候补值

有时候并不想提供一个错误处理方法,而是想在接收到错误的时候计算一个候补的值。

``` java
public void showOnErrorResume2(int val) {
    Flux<String> flux = Flux.just(val, 30)
            .map(this::convert1)
            .onErrorResume(e -> Mono.just(myWrapper.fromError(e)));
    flux.subscribe(System.out::println);
}
```

## 跳过异常
对于特定异常如果不想理它可以跳过。使用onErrorContinue来实现。

``` java
@Test
public void onErrorContinue() {
    Flux.interval(Duration.ofMillis(100))
            .map(i -> {
                if (i == 2)
                    throw new RuntimeException("fake a mistake");
                return String.valueOf(100 / (i - 5));
            })
            // 遇到error之后跳过，可以通过不同错误类型做不同处理
            .onErrorContinue((err, val) -> log
                    .error("处理第{}个元素时遇到错误，错误类型为：{}， 错误信息为： {}", val, err.getClass(), err.getMessage()))
            .subscribe(log::info);
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

## 记录日志并重新抛出异常

``` java
public void showDoOnError(int val) {
    Flux<String> flux = Flux.just(val, 30)
            .map(this::convert1)
            .doOnError(e -> {
                log.error("opps, error", e);
            });
    flux.subscribe(System.out::println);
}
```

## 记录日志并抛出业务异常

```
/**
 * 捕获并重新抛出自定义的业务异常。类似如下场景
 * try {
 * return callExternalService(k);
 * }
 * catch (Throwable error) {
 * throw new BusinessException("oops, SLA exceeded", error);
 * }
 */
public void showOnErrorMap(int val) {
    Flux<String> flux = Flux.just(val, 30)
            .map(this::convert1)
            .onErrorMap(original -> new BusinessError("SLA exceeded", original));
    flux.subscribe(System.out::println);
}
```

## 使用finally清理资源

``` java
/**
 * 使用 finally 代码块清理资源
 * <p>
 * Stats stats = new Stats();
 * stats.startTimer();
 * try {
 * doSomethingDangerous();
 * }
 * finally {
 * stats.stopTimerAndRecordTiming();
 * }
 */
public void showDoFinally(int val) {
    Flux<String> flux = Flux.just(val, 30)
            .map(this::convert1)
            .doFinally(type -> {
                if (type == SignalType.CANCEL)
                    System.out.println("cancel type");
            })
            .take(1); //在发出 1 个元素后取消流.
    flux.subscribe(System.out::println);
}
```

## 使用`try-with-resource`自动清理机制

``` java
 /**
 * 使用 Java 7 引入的 "try-with-resource"
 * <p>
 * try (SomeAutoCloseable disposableInstance = new SomeAutoCloseable()) {
 * return disposableInstance.toString();
 * }
 */
public void showUsing(int val) {
    AtomicBoolean isDisposed = new AtomicBoolean();
    Disposable disposableInstance = new Disposable() {
        @Override
        public void dispose() {
            isDisposed.set(true);
        }

        @Override
        public String toString() {
            return "DISPOSABLE";
        }
    };

    Flux<String> flux = Flux.just(val, 30)
            .map(this::convert1)
            .take(1); //在发出 1 个元素后取消流.
    Flux.using(() -> disposableInstance,
                    disposable -> flux,
                    Disposable::dispose)
            .subscribe(System.out::println);
}
```

## 错误重试

还有一个用于错误处理的操作符你可能会用到，就是 retry，见文知意，用它可以对出现错误的序列进行重试。

``` java
public void showRetry() throws InterruptedException {
    Flux.interval(Duration.ofMillis(250))
            .map(input -> {
                if (input < 3)
                    return "tick " + input;
                throw new RuntimeException("boom");
            })
            .retry(1)
            .elapsed()
            .subscribe(System.out::println, System.err::println);
    Thread.sleep(2100);
}

/**
 * 错误重试 retryWhen
 */
public void showRetryWhen() {
    Flux<String> flux = Flux
            .<String>error(new Exception())
            .doOnError(System.out::println)
            .retryWhen(Retry.from(companion -> companion.take(3)));
    flux.subscribe(System.out::println);
}
```

## 受检异常的处理

总体来说，所有的操作符自身都可能包含触发异常的代码，或自定义的可能导致失败的代码， 所以它们都自带一些错误处理方式。

一般来说，一个不受检异常总是由 onError 传递。例如，在一个 map 方法中抛出 RuntimeException 会被翻译为一个 onError 事件。
但是Reactor无法处理一个受检异常，因此你需要将操作符中的受检异常转换成不受检异常。

Reactor 有一个工具类 Exceptions，可以确保在收到受检异常的时候将其包装（wrap）起来。

* 如果需要，可以使用 Exceptions.propagate 方法来包装异常，它同样会首先调用 throwIfFatal，并且不会包装 RuntimeException。
* 使用 Exceptions.unwrap 方法来得到原始的未包装的异常（追溯最初的异常）。

``` java
/**
 * 受检异常处理方法，map里面的处理必须catch处理掉CheckedException，并将其包装成RuntimeException
 */
public void showCheckedExcpetion() {
    Flux<String> converted = Flux
            .range(1, 10)
            .map(i -> {
                try {
                    return convert2(i);
                } catch (IOException e) {
                    throw Exceptions.propagate(e);
                }
            });

    // 订阅时如果还要处理这个异常
    converted.subscribe(
            v -> System.out.println("RECEIVED: " + v),
            e -> {
                if (Exceptions.unwrap(e) instanceof IOException) {
                    System.out.println("Something bad happened with I/O");
                } else {
                    System.out.println("Something bad happened");
                }
            }
    );
}
```

