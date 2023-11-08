## Best practices for coroutines in Android

#### Inject Dispatchers
Easier to test.

#### Suspend functions should be safe to call from the main thread
Using withContext and specify dispatchers in coroutine.

#### The ViewModel should create coroutines
Hide suspend functions and flows from UI Layer.

#### The data and business layer should expose suspend functions and Flows

#### Make your coroutine cancellable
Cancellation in coroutines is cooperative, which means that when a coroutine's Job is cancelled, the coroutine isn't cancelled until it suspends or checks for cancellation.

Check your coroutine is active using `ensureActive` . If your coroutine is inactive, `CancellationException`  will be thrown.