# Reactor03-走进响应式编程

开发人员一般都会写一些监控的脚本，通过定时的访问，来确定一写事件是否发生，发生的话将内容通过邮件发送。
如果对时间粒度要求宽泛还好，如果要求比较严格，就要一直访问，造成不必要的资源浪费。
为了解决这种命令式编程的痛点，响应式编程应运而生，通过设置消息的订阅者和消息的发送者来解决这个问题，
通过事件驱动，一旦事件发生，发送者将消息推送给订阅者。

举个收外卖的例子：

你今天点了一个外卖，但是你不知道何时会送达，小区外卖进不来，你只能到小区门口去拿：

* 命令式：由于不知到几点到，你就每隔5分钟去小区门口看一下，如果外卖到了就拿回来，没到进入下个循环
* 响应式：小哥到了小区门口，打电话通知到小区门口去外卖

根据上面的例子明白响应式编程这种事件驱动的好处。而且响应式将发和收拆开将同步过程拆成异步的过程，对合理利用资源提升效率更有帮助

总结起来，响应式编程（reactive programming）是一种基于数据流（data stream）
和变化传递（propagation of change）的声明式（declarative）的编程范式。

## Reactor 3介绍

Reactor 3框架是Pivotal（Spring 母公司）基于Reactive Programming思想实现的。Spring Cloud 使用的响应式web框架 Webflux由Reactor实现。

reactor3 优势特点：

1. 可以将处理的数据视为stream，这也使得reactor的用法和java8的stream很像，流式处理数据更加函数式减少副作用，更加方便用于多线程处理
2. 可读性强，流式的编写，每一环节逻辑更清晰，避免了回调地狱
3. 组合性强，可使用先前任务的结果将输入反馈给后续任务
4. 事件的生产者和消费者拆成 Publisher和Publisher， Publisher只管准备数据，在Publisher订阅之前不会发生消费情况
5. 背压，Reactor比较重要的特征，事件下游可以通过反馈调节上游的生产速度
6. cold & hot：只有在观察者订阅的时候才会使用的数据源为cold，就是说数据源是Lazy的；hot一直在变，例如数据流，你不观察数据流也不会停止等你来观察

**这种编程范式如何让Web应用更加“reactive”呢？**

我们设想这样一种场景，我们从底层数据库驱动，经过持久层、服务层、MVC层中的model，到用户的前端界面的元素，全部都采用声明式的编程范式，
从而搭建一条能够传递变化的管道，这样我们只要更新一下数据库中的数据，用户的界面上就相应的发生变化，岂不美哉？尤其重要的是，一处发生变化，
我们不需要各种命令式的调用来传递这种变化，而是由搭建好的“流水线”自动传递。

这种场景用在哪呢？比如一个日志监控系统，我们的前端页面将不再需要通过“命令式”的轮询的方式不断向服务器请求数据然后进行更新，
而是在建立好通道之后，数据流从系统源源不断流向页面，从而展现实时的指标变化曲线；再比如一个社交平台，朋友的动态、点赞和留言不是手动刷出来的，
而是当后台数据变化的时候自动体现到界面上的。

## 学习准备

官网：https://projectreactor.io/docs/core/release/reference/

中文参考文档：https://htmlpreview.github.io/?https://github.com/get-set/reactor-core/blob/master-zh/src/docs/index.html

创建maven项目，引入如下依赖

``` xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-tools</artifactId>
    <version>3.4.12</version>
</dependency>
<dependency>
    <groupId>io.projectreactor.addons</groupId>
    <artifactId>reactor-extra</artifactId>
    <version>3.4.12</version>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <scope>test</scope>
    <version>3.4.12</version>
</dependency>
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <version>5.7.2</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
</dependency>
```