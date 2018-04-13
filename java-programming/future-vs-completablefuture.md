## **1. Introduction** {#Introduction}

This article is a guide to the functionality and use cases of the\_CompletableFuture\_class – introduced as a Java 8 Concurrency API improvement.

## **2. Asynchronous Computation in Java** {#Asynchronous}

Asynchronous computation is difficult to reason about. Usually we want to think of any computation as a series of steps. But in case of asynchronous computation,**actions represented as callbacks tend to be either scattered across the code or deeply nested inside each other**. Things get even worse when we need to handle errors that might occur during one of the steps.

The\_Future\_interface was added in Java 5 to serve as a result of an asynchronous computation, but it did not have any methods to combine these computations or handle possible errors.

**In Java 8, the**_**CompletableFuture**_**class was introduced.**Along with the\_Future\_interface, it also implemented the\_CompletionStage\_interface. This interface defines the contract for an asynchronous computation step that can be combined with other steps.

\_CompletableFuture\_is at the same time a building block and a framework with**about 50 different methods for composing, combining, executing asynchronous computation steps and handling errors**.

Such a large API can be overwhelming, but these mostly fall in several clear and distinct use cases.

## **3. Using**_**CompletableFuture**_**as a Simple**_**Future**_ {#CompletableFuture}

First of all, the_CompletableFuture\_class implements the\_Future\_interface, so you can**use it as a**_**Future**\_**implementation, but with additional completion logic**.

For example, you can create an instance of this class with a no-arg constructor to represent some future result, hand it out to the consumers and complete it at some time in the future using the\_complete\_method. The consumers may use the\_get\_method to block the current thread until this result will be provided.

In the example below we have a method that creates a_CompletableFuture\_instance, then spins off some computation in another thread and returns the\_Future_ immediately.

When the computation is done, the method completes the_Future_ by providing the result to the \_complete \_method:

```
public Future<String> calculateAsync() throws InterruptedException {
    CompletableFuture<String> completableFuture 
      = new CompletableFuture<>();
 
    Executors.newCachedThreadPool().submit(() -> {
        Thread.sleep(500);
        completableFuture.complete("Hello");
        return null;
    });
 
    return completableFuture;
}
```

To spin off the computation, we use the\_Executor\_API which is described in the article[“Introduction to Thread Pools in Java”](http://www.baeldung.com/thread-pool-java-and-guava), but this method of creating and completing a \_CompletableFuture \_can be used together with any concurrency mechanism or API including raw threads.

Notice that**the**_**calculateAsync**_**method returns a**_**Future**_**instance**.

We simply call the method, receive the_Future_ instance and call the\_get\_method on it when we’re ready to block for the result.

Also notice that the_get\_method throws some checked exceptions, namely\_ExecutionException_ \(encapsulating an exception that occurred during a computation\) and_InterruptedException_\(an exception signifying that a thread executing a method was interrupted\):

```
Future<String> completableFuture = calculateAsync();
 
// ... 
 
String result = completableFuture.get();
assertEquals("Hello", result);
```

**If you already know the result of a computation**, you can use the static \_completedFuture \_method with an argument that represents a result of this computation. Then the \_get \_method of the \_Future \_will never block, immediately returning this result instead.

```
Future<String> completableFuture = 
  CompletableFuture.completedFuture("Hello");
 
// ...
 
String result = completableFuture.get();
assertEquals("Hello", result);
```

As an alternative scenario, you may want to **cancel the execution of a**_**Future**_.

Suppose we didn’t manage to find a result and decided to cancel an asynchronous execution altogether. This can be done with the_Future_‘s _cancel\_method. This method receives a \_boolean\_argument\_mayInterruptIfRunning_, but in the case of_CompletableFuture\_it has no effect, as interrupts are not used to control processing for\_CompletableFuture_.

Here’s a modified version of the asynchronous method:

```
public Future<String> calculateAsyncWithCancellation() throws InterruptedException {
    CompletableFuture<String> completableFuture = new CompletableFuture<>();
 
    Executors.newCachedThreadPool().submit(() -> {
        Thread.sleep(500);
        completableFuture.cancel(false);
        return null;
    });
 
    return completableFuture;
}
```

When we block on the result using the\_Future.get\(\)\_method, it throws \_CancellationException \_if the future is canceled:

```
Future<String> future = calculateAsyncWithCancellation();
future.get(); // CancellationException
```

## **4.**_**CompletableFuture**_**with Encapsulated Computation Logic** {#CompletableFuture}

The code above allows us to pick any mechanism of concurrent execution, but what if we want to skip this boilerplate and simply execute some code asynchronously?

Static methods_runAsync\_and\_supplyAsync\_allow us to create a\_CompletableFuture\_instance out of\_Runnable\_and\_Supplier_ functional types correspondingly.

Both\_Runnable\_and\_Supplier\_are functional interfaces that allow passing their instances as lambda expressions thanks to the new Java 8 feature.

The\_Runnable\_interface is the same old interface that is used in threads and it does not allow to return a value.

The\_Supplier\_interface is a generic functional interface with a single method that has no arguments and returns a value of a parameterized type.

This allows to **provide an instance of the **_**Supplier **_**as a lambda expression that does the calculation and returns the result**. This is as simple as:

```
CompletableFuture<String> future
  = CompletableFuture.supplyAsync(() -> "Hello");
 
// ...
 
assertEquals("Hello", future.get());
```

## **5. Processing Results of Asynchronous Computations** {#Processing}

The most generic way to process the result of a computation is to feed it to a function. The \_thenApply \_method does exactly that: accepts a \_Function \_instance, uses it to process the result and returns a \_Future \_that holds a value returned by a function:

```
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<String> future = completableFuture
  .thenApply(s -> s + " World");
 
assertEquals("Hello World", future.get());
```

If you don’t need to return a value down the _Future \_chain, you can use an instance of the \_Consumer \_functional interface. Its single method takes a parameter and returns \_void_.

There’s a method for this use case in the_CompletableFuture_— the\_thenAccept\_method receives a\_Consumer\_and passes it the result of the computation. The final \_future.get\(\)\_call returns an instance of the \_Void\_type.

```
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<Void> future = completableFuture
  .thenAccept(s -> System.out.println("Computation returned: " + s));
 
future.get();
```

At last, if you neither need the value of the computation nor want to return some value at the end of the chain, then you can pass a \_Runnable \_lambda to the \_thenRun \_method. In the following example, after the \_future.get\(\)\_method is called, we simply print a line in the console:

```
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<Void> future = completableFuture
  .thenRun(() -> System.out.println("Computation finished."));
 
future.get();

```

**6. Combining Futures**

The best part of the_CompletableFuture\_API is the**ability to combine**_**CompletableFuture**\_**instances in a chain of computation steps**.

The result of this chaining is itself a\_CompletableFuture\_that allows further chaining and combining. This approach is ubiquitous in functional languages and is often referred to as a monadic design pattern.

**In the following example we use the**_**thenCompose**_**method to chain two**_**Futures**_**sequentially.**

Notice that this method takes a function that returns a _CompletableFuture \_instance. The argument of this function is the result of the previous computation step. This allows us to use this value inside the next\_CompletableFuture_‘s lambda:

```
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " World"));
 
assertEquals("Hello World", completableFuture.get());
```

The_thenCompose\_method together with\_thenApply_ implement basic building blocks of the monadic pattern. They closely relate to the\_map\_and\_flatMap\_methods of\_Stream\_and\_Optional\_classes also available in Java 8.

Both methods receive a function and apply it to the computation result, but the _thenCompose_ \(_flatMap_\) method**receives a function that returns another object of the same type**. This functional structure allows composing the instances of these classes as building blocks.

If you want to execute two independent_Futures_ and do something with their results, use the _thenCombine \_method that accepts a \_Future \_and a\_Function_ with two arguments to process both results:

```
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCombine(CompletableFuture.supplyAsync(
      () -> " World"), (s1, s2) -> s1 + s2));
 
assertEquals("Hello World", completableFuture.get());
```

A simpler case is when you want to do something with two_Futures_‘ results, but don’t need to pass any resulting value down a \_Future \_chain. The \_thenAcceptBoth \_method is there to help:

```
CompletableFuture future = CompletableFuture.supplyAsync(() -> "Hello")
  .thenAcceptBoth(CompletableFuture.supplyAsync(() -> " World"),
    (s1, s2) -> System.out.println(s1 + s2));
```

## **7. Running Multiple **_**Futures **_**in Parallel** {#Multiple}

When we need to execute multiple\_Futures\_in parallel, we usually want to wait for all of them to execute and then process their combined results.

The \_CompletableFuture.allOf \_static method allows to wait for completion of all of the \_Futures \_provided as a var-arg:

```
CompletableFuture<String> future1  
  = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2  
  = CompletableFuture.supplyAsync(() -> "Beautiful");
CompletableFuture<String> future3  
  = CompletableFuture.supplyAsync(() -> "World");
 
CompletableFuture<Void> combinedFuture 
  = CompletableFuture.allOf(future1, future2, future3);
 
// ...
 
combinedFuture.get();
 
assertTrue(future1.isDone());
assertTrue(future2.isDone());
assertTrue(future3.isDone());
```

Notice that the return type of the_CompletableFuture.allOf\(\)\_is a \_CompletableFuture&lt;Void&gt;_. The limitation of this method is that it does not return the combined results of all_Futures_. Instead you have to manually get results from_Futures_. Fortunately,\_CompletableFuture.join\(\)\_method and Java 8 Streams API makes it simple:

```
String combined = Stream.of(future1, future2, future3)
  .map(CompletableFuture::join)
  .collect(Collectors.joining(" "));
 
assertEquals("Hello Beautiful World", combined);
```

The_CompletableFuture.join\(\)\_method is similar to the\_get\_method, but it throws an unchecked exception in case the\_Future_ does not complete normally. This makes it possible to use it as a method reference in the\_Stream.map\(\)\_method.

## **8. Handling Errors** {#Handling}

For error-handling in a chain of asynchronous computation steps,\_throw/catch\_idiom had to be adapted in a similar fashion.

Instead of catching an exception in a syntactic block, the_CompletableFuture_ class allows you to handle it in a special\_handle\_method. This method receives two parameters: a result of a computation \(if it finished successfully\) and the exception thrown \(if some computation step did not complete normally\).

In the following example we use the \_handle \_method to provide a default value when the asynchronous computation of a greeting was finished with an error because no name was provided:

```
String name = null;
 
// ...
 
CompletableFuture<String> completableFuture  
  =  CompletableFuture.supplyAsync(() -> {
      if (name == null) {
          throw new RuntimeException("Computation error!");
      }
      return "Hello, " + name;
  })}).handle((s, t) -> s != null ? s : "Hello, Stranger!");
 
assertEquals("Hello, Stranger!", completableFuture.get());
```

As an alternative scenario, suppose we want to manually complete the \_Future \_with a value, as in the first example, but also to have the ability to complete it with an exception. The \_completeExceptionally \_method is intended for that. The \_completableFuture.get\(\)\_method in the following example throws an \_ExecutionException \_with a \_RuntimeException \_as its cause:

```
CompletableFuture<String> completableFuture = new CompletableFuture<>();
 
// ...
 
completableFuture.completeExceptionally(
  new RuntimeException("Calculation failed!"));
 
// ...
 
completableFuture.get(); // ExecutionException

```

In the example above we could have handled the exception with the\_handle\_method asynchronously, but with the\_get\_method we can use a more typical approach of a synchronous exception processing.

## **9. Async Methods** {#Async}

Most methods of the fluent API in\_CompletableFuture\_class have two additional variants with the\_Async\_postfix. These methods are usually intended for**running a corresponding step of execution in another thread**.

The methods without the_Async\_postfix run the next execution stage using a calling thread. The\_Async_ method without the_Executor\_argument runs a step using the common\_fork/join\_pool implementation of\_Executor\_that is accessed with the\_ForkJoinPool.commonPool\(\)\_method. The\_Async_ method with an_Executor\_argument runs a step using the passed\_Executor_.

Here’s a modified example that processes the result of a computation with a \_Function \_instance. The only visible difference is the \_thenApply Async \_method. But under the hood the application of a function is wrapped into a \_ForkJoinTask \_instance \(for more information on the \_fork/join \_framework, see the article[“Guide to the Fork/Join Framework in Java”](http://www.baeldung.com/java-fork-join)\). This allows to parallelize your computation even more and use system resources more efficiently.

```
CompletableFuture<String> completableFuture  
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<String> future = completableFuture
  .thenApplyAsync(s -> s + " World");
 
assertEquals("Hello World", future.get());
```

## **10. Conclusion** {#Conclusion}

In this article we’ve described the methods and typical use cases of the\_CompletableFuture\_class.

The source code for the article is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-concurrency).

