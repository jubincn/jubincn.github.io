## launch
Starts a new coroutine and doesn't return the result to the caller. 

## async&await
Starts a new coroutine and allows you to return a result with a suspend function called await.

## Return Type
Launch return a Job, while async return a Deffered implementation.

## Error Handling
try catch will work for both. For unhandled exceptions, launch will throw it directly while async will store the exception in deffered object and throw it when await() is called.