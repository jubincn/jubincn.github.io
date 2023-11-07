## Introduction
### Feature
#### Lightweight
##### Suspension
Multi coroutinues can be run on one thread due to support for suspension, which doesn't block the thread where coroutine is running.

#### Fewer Memory Leaks
##### Structured Concurrency
New coroutines can only be launched in a specific CoroutineScope which delimits the lifetime of the coroutine. 

#### Built-in cancellation support
Cancellation is propagated automatically through the running coroutine hierarchy.

#### Jetpack integration

## Advanced Coroutine Concepts
### Suspend - Resume (Stack Frame)
Kotlin uses a stack frame to manage which function is running along with any local variables. When suspending a coroutine, the current stack frame is copied and saved for later. When resuming, the stack frame is copied back from where it was saved, and the function starts running again. 

### Dispatchers
#### Assign Dispatchers
1. Dispatchers.Main
2. Dispatchers.IO
3. Dispatchers.Default

withContext(Dispatchers.IO)

#### withContext() Performance
withContext() does not add extra overhead compared to an equivalent callback-based implementation. Furthermore, it's possible to optimize withContext() calls beyond an equivalent callback-based implementation in some situations. 

### Start a Coroutine
#### async
1. Starts a new coroutine and allows you to return a result with a suspend function called await
2. Exception: Async expects an eventual call to await, it holds exceptions and rethrows them as part of the await call. The exception will not show in crash metrics.
#### launch
1. Starts a new coroutine and doesn't return the result to the caller. 

#### Parallel Decomposition
**All coroutines that are started inside a suspend function must be stopped when that function returns**, so you likely need to guarantee that those coroutines finish before returning. 

With structured concurrency in Kotlin, you can** define a coroutineScope** that starts one or more coroutines. 

Then, using **await() (for a single coroutine)** or **awaitAll() (for multiple coroutines)**, you can guarantee that these coroutines finish before returning from the function.
##### Summary
Build a coroutineScope with async and await them to finish
```
suspend fun fetchTwoDocs() = coroutineScope {
    val deferredOne = async { fetchDoc(1) }        
    val deferredTwo = async { fetchDoc(2) }       
    deferredOne.await()        
    deferredTwo.await()    
}
```

### Coroutines concepts
#### CoroutineScope
A CoroutineScope keeps track of any coroutine it creates using launch or async. ```scope.cancel()```

```
class ExampleClass { 
    ...
    fun exampleMethod() {
        // Handle to the coroutine, you can control its lifecycle
        val job = scope.launch { 
            // New coroutine       
        }
        if (...) {
            // Cancel the coroutine started above, this doesn't affect the scope           
            // this coroutine was launched in            
            job.cancel()        
        }
    }
}
```

#### CoroutineContext
A **CoroutineContext** defines the behavior of a coroutine using the following set of elements:
* Job: Controls the lifecycle of the coroutine.
* CoroutineDispatcher: Dispatches work to the appropriate thread.
* CoroutineName: The name of the coroutine, useful for debugging.
* CoroutineExceptionHandler: Handles uncaught exceptions.


##### Parent CoroutineContext explained
The resulting parent CoroutineContext of a coroutine can be different from the CoroutineContext of the parent since it’s calculated based on this formula:
>Parent context = Defaults + inherited CoroutineContext + arguments

* Some elements have default values: `Dispatchers.Default` is the default of CoroutineDispatcher and `“coroutine”` the default of CoroutineName.
* The **inherited CoroutineContext** is the CoroutineContext of the CoroutineScope or coroutine that created it.
* **Arguments** passed in the coroutine builder will take precedence over those elements in the inherited context.
![Parent CoroutineContext](/../assets/pic/20231107_parent_context.png)

##### New CoroutineContext
>New coroutine context = parent CoroutineContext + Job()

```
val job = scope.launch(Dispatchers.IO) {// new coroutine}
```

![New CoroutineContext](/../assets/pic/20231107_new_coroutine_context.png)


##### Job
A job is a handle to coroutine. If a job is not passed into CoroutineContext, Job() will be used and it will cancel all coroutines in the scope for uncaught exceptions.
```
val job = scope.launch {
    // New coroutine        
}
```
###### Job Lifecycle
A Job can go through a set of states: New, Active, Completing, Completed, Cancelling and Cancelled. 
![Job Lifecycle](/../assets/pic/20231107_job_lifecycle.png)


##### CoroutineDispatcher
Kotlin coroutines use dispatchers to determine which threads are used for coroutine execution. Coroutines can suspend themselves, and the dispatcher is responsible for resuming them.

`withContext` is usually called to switch dispatcher.

###### Customize Dispatcher
* Private thread pools can be created with newSingleThreadContext and newFixedThreadPoolContext.
* An arbitrary java.util.concurrent.Executor can be converted to a dispatcher with the asCoroutineDispatcher extension function.

##### CoroutineName
The name of the coroutine, useful for debugging.

##### CoroutineExceptionHandler
An optional element in the coroutine context to handle uncaught exceptions.

* Normally, uncaught exceptions can only result from root coroutines as all children coroutines delegate  handling of their exceptions to their parent.
* Coroutines running with `SupervisorJob` do not propagate exceptions to their parent and treated like root coroutines.
* A coroutine that was created using async always catches all its exceptions and represents them in the resulting Deferred object, so it cannot result in uncaught exceptions.

###### Uncaught exceptions with no handler

* If exception is CancellationException, it is ignored, as these exceptions are used to cancel coroutines.
* Otherwise, if there is a Job in the context, then Job.cancel is invoked.
* Otherwise, as a last resort, the exception is processed in a platform-specific manner:
    * On JVM, all instances of CoroutineExceptionHandler found via ServiceLoader, as well as the current thread's Thread.uncaughtExceptionHandler, are invoked.

### References
[Coroutines: first things first](https://medium.com/androiddevelopers/coroutines-first-things-first-e6187bf3bb21)
[Developer Android Kotlin Coroutine](https://developer.android.com/kotlin/coroutines)