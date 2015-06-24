---
layout: post
title:  "用CompletableFuture处理异步超时问题"
keywords: "java"
description: "JDK 8中的新的抽象 – CompletableFuture。众所周知，Java 8不到一年就会发布，CompletableFuture extends Future提供了方法，一元操作符和促进异步性以及事件驱动编程模型，它并不止步于旧版本的Java中。如果你打开JavaDoc of CompletableFuture你一定会感到震惊"
category: java 
tags: [java]
---
一天，我在改进多线程代码时被`Future.get()`阻塞住了。

```
public void serve() throws InterruptedException, ExecutionException, TimeoutException {
    final Future<Response> responseFuture = asyncCode();
    final Response response = responseFuture.get(1, SECONDS);
    send(response);
}
 
private void send(Response response) {
    //...
}
```

这是用Java写的一个Akka应用程序，使用了一个包含1000个线程的线程池——所有的线程都在阻塞在这个 `get()`中。系统的处理速度跟不上并发请求的数量。重构以后，我们干掉了所有的这些线程仅保留了一个，极大的减少了内存的占用。我们简单一点，通过一个Java 8的例子来演示。第一步是使用`CompletableFuture` 来替换简单的 `Future`（见：Tip 9）。

 * 通过控制任务提交到`ExecutorService`的方式：只需用 `CompletableFuture.supplyAsync(…, executorService)` 来代替 `executorService.submit(…)` 即可
 * 处理基于回调函数的API：使用promises
 
否则（如果你已经使用了阻塞式的API或 Future<T>）会导致很多线程被阻塞。这就是为什么现在这么多异步的API都让人很烦了。所以，让我们重写之前的代码来接收`CompletableFuture`：

```
public void serve() throws InterruptedException, ExecutionException, TimeoutException {
    final CompletableFuture<Response> responseFuture = asyncCode();
    final Response response = responseFuture.get(1, SECONDS);
    send(response);
}
```
很明显，这不能解决任何问题，我们还必须利用新的风格来编程：

```
public void serve() {
    final CompletableFuture<Response> responseFuture = asyncCode();
    responseFuture.thenAccept(this::send);
}
```
这个功能与上面功能是等同的，但是`serve()` 只会运行一小段时间（不会阻塞或等待）。只需要记住：`this::send` 将会在完成 `responseFuture` 的同一个线程内执行。如果你不想花费太大的代价来重载已经存在的线程池或`send()`方法，可以考虑通过 `thenAcceptAsync(this::send, sendPool)` 方法，但是我们失去了两个重要属性：异常传播与超时。异常传播很难实现，因为我们改变了API。当`serve()`存在的时候，异步操作可能还没有完成。如果你关心异常，可以考虑返回 `responseFutureor` 或者其他可选的机制。至少，应该有异常的日志，否则该异常就会被吞了。

```
final CompletableFuture<Response> responseFuture = asyncCode();
responseFuture.exceptionally(throwable -> {
    log.error("Unrecoverable error", throwable);
    return null;
});
```
请小心上面的代码：`exceptionally()` 试图从失败中恢复过来，返回一个可选的结果。这个地方虽可以正常的工作，但是如果对 `exceptionally()`和`withthenAccept()` 使用链式调用，即使失败了也还是会调用 `send()` 方法，返回一个null参数，或者任何其它从 `exceptionally() `方法中返回的值。

```
responseFuture
    .exceptionally(throwable -> {
        log.error("Unrecoverable error", throwable);
        return null;
    })
    .thenAccept(this::send);  //可能不是你所想的
```
丢失一秒超时的问题非常巧妙。我们原始的代码在`Future`完成之前最多等待（阻塞）1秒，否则就会抛出 `TimeoutException`。我们丢失了这个功能，更糟糕的是，单元测试超时的不是很方便，经常会跳过这个环节。为了维持超时机制，而又不破坏事件驱动的原则，我们需要建立一个额外的模块：一个在给定时间后必定会失败的Future

```
public static <T> CompletableFuture<T> failAfter(Duration duration) {
    final CompletableFuture<T> promise = new CompletableFuture<>();
    scheduler.schedule(() -> {
        final TimeoutException ex = new TimeoutException("Timeout after " + duration);
        return promise.completeExceptionally(ex);
    }, duration.toMillis(), MILLISECONDS);
    return promise;
}
 
private static final ScheduledExecutorService scheduler =
        Executors.newScheduledThreadPool(
                1,
                new ThreadFactoryBuilder()
                        .setDaemon(true)
                        .setNameFormat("failAfter-%d")
                        .build());
```

这个很简单：我们创建一个promise（没有后台任务或线程池的 Future），然后在给定的 `java.time.Duration` 之后会抛出 `TimeoutException` 异常。如果在某个地方调用 `get()` 获取这个 Future，阻塞的时间到达这个指定的时间后会抛出 `TimeoutException`。
实际上，它是一个包装了 `TimeoutException` 的 `ExecutionException`，这个无需多说。注意，我使用了`scheduler` 一个线程的线程池。这不仅仅是为了教学的目的：这是“1个线程应当能满足任何人的需求”的场景。`failAfter()` 本身没多大的用处，但是如果和 `ourresponseFuture` 一起使用，我们就能解决这个问题了。

```
final CompletableFuture<Response> responseFuture = asyncCode();
final CompletableFuture<Response> oneSecondTimeout = failAfter(Duration.ofSeconds(1));
responseFuture
        .acceptEither(oneSecondTimeout, this::send)
        .exceptionally(throwable -> {
            log.error("Problem", throwable);
            return null;
        });
```

这里还做了很多其他事情。在后台的任务接收 `responseFuture` 时，我们也创建了一个“合成”的 `oneSecondTimeout` future，这在成功的时候永远不会执行，但是在1秒后就会导致任务失败。现在我们联合这两个叫做 `acceptEither`，这个操作将执行先完成 Future 的代码块，而简单的忽略 `responseFuture` 或 `oneSecondTimeout` 中运行比较慢的那个。如果 asyncCode() 代码在1秒内执行完成，`this::send` 就会被调用，而 `oneSecondTimeout` 异常就不会抛出。但是，如果 `asyncCode()` 执行真的很慢，`oneSecondTimeout` 异常就先抛出。由于一个异常导致任务失败，`exceptionallyerror` 处理器就会被调用，而不是 `this::send` 方法。你可以选择执行 `send()` 或者 `exceptionally`，但是不能两个都执行。当如，如果我们有两个“普通”的 Future 正常执行完成了，则最先响应的那个将调用 `send()` 方法，后面的就会被丢弃。

这个不是最清晰的解决方案。更清晰的方案是包装原始的 Future，然后保证它能在给定的时间内执行。这种操作对 `com.twitter.util.Future` 是可行的（Scala叫做 `within()`），但是 `scala.concurrent.Future` 中没有这个功能（据推测是为了鼓励使用前面的方式）。我们暂时不讨论Scala背后如何执行的，先实现类似 `CompletableFuture` 的操作。它接受一个 Future 作为输入，然后返回一个 Future，这个 Future 在后台任务完成时候执行完成。但是，如果底层的 Future 执行的时间太长，就或抛出异常：

```
public static <T> CompletableFuture<T> within(CompletableFuture<T> future, Duration duration) {
    final CompletableFuture<T> timeout = failAfter(duration);
    return future.applyToEither(timeout, Function.identity());
}
```
这引导我们实现最终清晰的灵活的解决方案：

```
final CompletableFuture<Response> responseFuture = within(
        asyncCode(), Duration.ofSeconds(1));
responseFuture
        .thenAccept(this::send)
        .exceptionally(throwable -> {
            log.error("Unrecoverable error", throwable);
            return null;
        });
```
希望你喜欢这篇文章，因为你已经知道在Java里，实现响应式编程不再是什么问题。

[原文地址：http://www.javacodegeeks.com/2014/12/asynchronous-timeouts-with-completablefuture.html](http://www.javacodegeeks.com/2014/12/asynchronous-timeouts-with-completablefuture.html)