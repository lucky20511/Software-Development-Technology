## **1. Introduction** {#Introduction}

This article is a guide to the functionality and use cases of the_CompletableFuture_class – introduced as a Java 8 Concurrency API improvement.

## **2. Asynchronous Computation in Java** {#Asynchronous}

Asynchronous computation is difficult to reason about. Usually we want to think of any computation as a series of steps. But in case of asynchronous computation,**actions represented as callbacks tend to be either scattered across the code or deeply nested inside each other**. Things get even worse when we need to handle errors that might occur during one of the steps.

The_Future_interface was added in Java 5 to serve as a result of an asynchronous computation, but it did not have any methods to combine these computations or handle possible errors.

**In Java 8, the**_**CompletableFuture**_**class was introduced.**Along with the_Future_interface, it also implemented the_CompletionStage_interface. This interface defines the contract for an asynchronous computation step that can be combined with other steps.

_CompletableFuture_is at the same time a building block and a framework with**about 50 different methods for composing, combining, executing asynchronous computation steps and handling errors**.

Such a large API can be overwhelming, but these mostly fall in several clear and distinct use cases.

## **3. Using**_**CompletableFuture**_**as a Simple**_**Future**_ {#CompletableFuture}

First of all, the_CompletableFuture_class implements the_Future_interface, so you can**use it as a**_**Future**_**implementation, but with additional completion logic**.

For example, you can create an instance of this class with a no-arg constructor to represent some future result, hand it out to the consumers and complete it at some time in the future using the_complete_method. The consumers may use the_get_method to block the current thread until this result will be provided.

In the example below we have a method that creates a_CompletableFuture_instance, then spins off some computation in another thread and returns the_Future_ immediately.

When the computation is done, the method completes the_Future_ by providing the result to the _complete _method:

```

```

To spin off the computation, we use the_Executor_API which is described in the article[“Introduction to Thread Pools in Java”](http://www.baeldung.com/thread-pool-java-and-guava), but this method of creating and completing a _CompletableFuture _can be used together with any concurrency mechanism or API including raw threads.

Notice that**the**_**calculateAsync**_**method returns a**_**Future**_**instance**.

We simply call the method, receive the_Future_ instance and call the_get_method on it when we’re ready to block for the result.

Also notice that the_get_method throws some checked exceptions, namely_ExecutionException_ \(encapsulating an exception that occurred during a computation\) and_InterruptedException_\(an exception signifying that a thread executing a method was interrupted\):

```

```

**If you already know the result of a computation**, you can use the static _completedFuture _method with an argument that represents a result of this computation. Then the _get _method of the _Future _will never block, immediately returning this result instead.

```

```

As an alternative scenario, you may want to**cancel the execution of a**_**Future**_.

Suppose we didn’t manage to find a result and decided to cancel an asynchronous execution altogether. This can be done with the_Future_‘s _cancel_method. This method receives a _boolean_argument_mayInterruptIfRunning_, but in the case of_CompletableFuture_it has no effect, as interrupts are not used to control processing for_CompletableFuture_.

Here’s a modified version of the asynchronous method:

```

```

When we block on the result using the_Future.get\(\)_method, it throws _CancellationException _if the future is canceled:

```

```

## **4.**_**CompletableFuture**_**with Encapsulated Computation Logic** {#CompletableFuture}

The code above allows us to pick any mechanism of concurrent execution, but what if we want to skip this boilerplate and simply execute some code asynchronously?

Static methods_runAsync_and_supplyAsync_allow us to create a_CompletableFuture_instance out of_Runnable_and_Supplier_ functional types correspondingly.

Both_Runnable_and_Supplier_are functional interfaces that allow passing their instances as lambda expressions thanks to the new Java 8 feature.

The_Runnable_interface is the same old interface that is used in threads and it does not allow to return a value.

The_Supplier_interface is a generic functional interface with a single method that has no arguments and returns a value of a parameterized type.

This allows to **provide an instance of the **_**Supplier **_**as a lambda expression that does the calculation and returns the result**. This is as simple as:

```

```

## **5. Processing Results of Asynchronous Computations** {#Processing}

The most generic way to process the result of a computation is to feed it to a function. The _thenApply _method does exactly that: accepts a _Function _instance, uses it to process the result and returns a _Future _that holds a value returned by a function:

```

```

If you don’t need to return a value down the _Future _chain, you can use an instance of the _Consumer _functional interface. Its single method takes a parameter and returns _void_.

There’s a method for this use case in the_CompletableFuture_— the_thenAccept_method receives a_Consumer_and passes it the result of the computation. The final _future.get\(\)_call returns an instance of the _Void_type.

```

```

At last, if you neither need the value of the computation nor want to return some value at the end of the chain, then you can pass a _Runnable _lambda to the _thenRun _method. In the following example, after the _future.get\(\)_method is called, we simply print a line in the console:

```

```

**6. Combining Futures**

The best part of the_CompletableFuture_API is the**ability to combine**_**CompletableFuture**_**instances in a chain of computation steps**.

The result of this chaining is itself a_CompletableFuture_that allows further chaining and combining. This approach is ubiquitous in functional languages and is often referred to as a monadic design pattern.

**In the following example we use the**_**thenCompose**_**method to chain two**_**Futures**_**sequentially.**

Notice that this method takes a function that returns a _CompletableFuture _instance. The argument of this function is the result of the previous computation step. This allows us to use this value inside the next_CompletableFuture_‘s lambda:

```

```

The_thenCompose_method together with_thenApply_ implement basic building blocks of the monadic pattern. They closely relate to the_map_and_flatMap_methods of_Stream_and_Optional_classes also available in Java 8.

Both methods receive a function and apply it to the computation result, but the _thenCompose_ \(_flatMap_\) method**receives a function that returns another object of the same type**. This functional structure allows composing the instances of these classes as building blocks.

If you want to execute two independent_Futures_ and do something with their results, use the _thenCombine _method that accepts a _Future _and a_Function_ with two arguments to process both results:

```

```

A simpler case is when you want to do something with two_Futures_‘ results, but don’t need to pass any resulting value down a _Future _chain. The _thenAcceptBoth _method is there to help:

```

```

## **7. Running Multiple **_**Futures **_**in Parallel** {#Multiple}

When we need to execute multiple_Futures_in parallel, we usually want to wait for all of them to execute and then process their combined results.

The _CompletableFuture.allOf _static method allows to wait for completion of all of the _Futures _provided as a var-arg:

```

```

Notice that the return type of the_CompletableFuture.allOf\(\)_is a _CompletableFuture&lt;Void&gt;_. The limitation of this method is that it does not return the combined results of all_Futures_. Instead you have to manually get results from_Futures_. Fortunately,_CompletableFuture.join\(\)_method and Java 8 Streams API makes it simple:

```

```

The_CompletableFuture.join\(\)_method is similar to the_get_method, but it throws an unchecked exception in case the_Future_ does not complete normally. This makes it possible to use it as a method reference in the_Stream.map\(\)_method.

## **8. Handling Errors** {#Handling}

For error-handling in a chain of asynchronous computation steps,_throw/catch_idiom had to be adapted in a similar fashion.

Instead of catching an exception in a syntactic block, the_CompletableFuture_ class allows you to handle it in a special_handle_method. This method receives two parameters: a result of a computation \(if it finished successfully\) and the exception thrown \(if some computation step did not complete normally\).

In the following example we use the _handle _method to provide a default value when the asynchronous computation of a greeting was finished with an error because no name was provided:

```

```

As an alternative scenario, suppose we want to manually complete the _Future _with a value, as in the first example, but also to have the ability to complete it with an exception. The _completeExceptionally _method is intended for that. The _completableFuture.get\(\)_method in the following example throws an _ExecutionException _with a _RuntimeException _as its cause:

```

```

In the example above we could have handled the exception with the_handle_method asynchronously, but with the_get_method we can use a more typical approach of a synchronous exception processing.

## **9. Async Methods** {#Async}

Most methods of the fluent API in_CompletableFuture_class have two additional variants with the_Async_postfix. These methods are usually intended for**running a corresponding step of execution in another thread**.

The methods without the_Async_postfix run the next execution stage using a calling thread. The_Async_ method without the_Executor_argument runs a step using the common_fork/join_pool implementation of_Executor_that is accessed with the_ForkJoinPool.commonPool\(\)_method. The_Async_ method with an_Executor_argument runs a step using the passed_Executor_.

Here’s a modified example that processes the result of a computation with a _Function _instance. The only visible difference is the _thenApply Async _method. But under the hood the application of a function is wrapped into a _ForkJoinTask _instance \(for more information on the _fork/join _framework, see the article[“Guide to the Fork/Join Framework in Java”](http://www.baeldung.com/java-fork-join)\). This allows to parallelize your computation even more and use system resources more efficiently.

```

```

## **10. Conclusion** {#Conclusion}

In this article we’ve described the methods and typical use cases of the_CompletableFuture_class.

The source code for the article is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-concurrency).

