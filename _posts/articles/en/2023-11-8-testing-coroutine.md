## Testing Coroutines in Android
### Invoking suspending functions in tests
`runTest{}` is a coroutine builder designed for testing. 
```
suspend fun fetchData(): String {
    delay(1000L)
    return "Hello world"
}

@Test
fun dataShouldBeHelloWorld() = runTest {
    val data = fetchData()    
    assertEquals("Hello world", data)
}
```

However, there are **additional considerations** to make, depending on what happens in your code under test:

* When your code creates new coroutines other than the top-level test coroutine that runTest creates, you will need to control how those new coroutines are scheduled by choosing the appropriate `TestDispatcher`.

* If your code moves the coroutine execution to other dispatchers (for example, by using `withContext`), `runTest` will still generally work, but delays will no longer be skipped, and tests will be less predictable as code runs on multiple threads. For these reasons, in tests you should **_inject test dispatchers_** to replace real dispatchers.

### TestDispatchers
**TestDispatchers** shalll be used if new coroutines are created during the test to make the execution of the new coroutines predictable.

`runTest` runs the test coroutine in a *TestScope*, using a *TestDispatcher*. TestDispatchers use a *TestCoroutineScheduler* to control virtual time and schedule new coroutines in a test. All TestDispatchers in a test must use the same scheduler instance.

#### StandardTestDispatcher
When you start new coroutines on a StandardTestDispatcher, they are queued up on the underlying scheduler, to be run whenever the test thread is free to use. 

#### UnconfinedTestDispatcher
When new coroutines are started on an UnconfinedTestDispatcher, they are started eagerly on the current thread.

### Injecting TestDispatchers
Code under test might use dispatchers to switch threads (using withContext) or to start new coroutines. When code is executed on multiple threads in parallel, tests can become flaky. 

In tests, replace real dispatchers with instances of TestDispatchers to ensure that all code runs on the single test thread.

### Setting the Main dispatcher
Local unit tests run on JVM instead of real android device, if your code under test references the main thread, it’ll throw an exception during unit tests. 

To replace the Main dispatcher with a TestDispatcher in all cases, use the Dispatchers.setMain and Dispatchers.resetMain functions.

```

class HomeViewModelTest { 
    @Test
    fun settingMainDispatcher() = runTest {
        val testDispatcher = UnconfinedTestDispatcher(testScheduler)      
        Dispatchers.setMain(testDispatcher)        
        
        try {           
            val viewModel = HomeViewModel()            
            viewModel.loadMessage() // Uses testDispatcher, runs its coroutine eagerly            
            assertEquals("Greetings!", viewModel.message.value)        
        } finally {
            Dispatchers.resetMain()        
       }   
    }
}
```
