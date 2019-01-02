# Convert Async to Sync Call Using Future and CountDownLatch
If a sync method call is wanted while API only provides async callback, we can con convert it to sync method call with Future and CountDownLatch.

## Async callback example
The following code snippet is a demo from Fresco DataSource callback. We have three callbacks and will convert it to sync method call.

```kotlin
dataSource.subscribe(object : BaseDataSubscriber<CloseableReference<CloseableImage?>?>() {
                override fun onNewResultImpl(paramDataSource: DataSource<CloseableReference<CloseableImage?>?>?) {
                    // callback
                }

                override fun onCancellation(dataSource: DataSource<CloseableReference<CloseableImage?>?>?) {
                    // callback
                }

                override fun onFailureImpl(dataSource: DataSource<CloseableReference<CloseableImage?>?>) {                    
                    // callback
                }
            }, UiThreadImmediateExecutorService.getInstance())
```

## Using Future to convert Async to Sync method call
### Future Interface

```Future``` is an Interface which represents the result of an asynchronous computation. We can use ```Future.get()``` or ```Future.get(long timeout, TimeUnit unit)``` to get result as a sync method call.

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

### Add Future to our demo
We will implement a custom future later and put "XXXFuture" here to illustrate the usage of Future.

```kotlin
val future = XXXFuture
dataSource.subscribe(object : BaseDataSubscriber<CloseableReference<CloseableImage?>?>() {
                override fun onNewResultImpl(paramDataSource: DataSource<CloseableReference<CloseableImage?>?>?) {
                    future.set(paramDataSource)
                }

                override fun onCancellation(dataSource: DataSource<CloseableReference<CloseableImage?>?>?) {                  
                    future.set(null)
                }

                override fun onFailureImpl(dataSource: DataSource<CloseableReference<CloseableImage?>?>) {                    
                    future.set(null)
                }
            }, UiThreadImmediateExecutorService.getInstance())
val bitmapDataSource = future.get()
```

### Implement a custom Future with CountDownLatch
There is a simple implementation in Facebook's ReactNative library named ```SimpleSettableFuture```

```java

package com.facebook.react.common.futures;

import javax.annotation.Nullable;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

/**
 * A super simple Future-like class that can safely notify another Thread when a value is ready.
 * Does not support canceling.
 */
public class SimpleSettableFuture<T> implements Future<T> {
  private final CountDownLatch mReadyLatch = new CountDownLatch(1);
  private @Nullable T mResult;
  private @Nullable Exception mException;

  /**
   * Sets the result. If another thread has called {@link #get}, they will immediately receive the
   * value. set or setException must only be called once.
   */
  public void set(@Nullable T result) {
    checkNotSet();
    mResult = result;
    mReadyLatch.countDown();
  }

  /**
   * Sets the exception. If another thread has called {@link #get}, they will immediately receive
   * the exception. set or setException must only be called once.
   */
  public void setException(Exception exception) {
    checkNotSet();
    mException = exception;
    mReadyLatch.countDown();
  }

  @Override
  public boolean cancel(boolean mayInterruptIfRunning) {
    throw new UnsupportedOperationException();
  }

  @Override
  public boolean isCancelled() {
    return false;
  }

  @Override
  public boolean isDone() {
    return mReadyLatch.getCount() == 0;
  }

  @Override
  public @Nullable T get() throws InterruptedException, ExecutionException {
    mReadyLatch.await();
    if (mException != null) {
      throw new ExecutionException(mException);
    }

    return mResult;
  }

  /**
   * Wait up to the timeout time for another Thread to set a value on this future. If a value has
   * already been set, this method will return immediately.
   *
   * NB: For simplicity, we catch and wrap InterruptedException. Do NOT use this class if you
   * are in the 1% of cases where you actually want to handle that.
   */
  @Override
  public @Nullable T get(long timeout, TimeUnit unit) throws
      InterruptedException, ExecutionException, TimeoutException {
    if (!mReadyLatch.await(timeout, unit)) {
      throw new TimeoutException("Timed out waiting for result");
    }
    if (mException != null) {
      throw new ExecutionException(mException);
    }

    return mResult;
  }

  /**
   * Convenience wrapper for {@link #get()} that re-throws get()'s Exceptions as
   * RuntimeExceptions.
   */
  public @Nullable T getOrThrow() {
    try {
      return get();
    } catch (InterruptedException | ExecutionException e) {
      throw new RuntimeException(e);
    }
  }

  /**
   * Convenience wrapper for {@link #get(long, TimeUnit)} that re-throws get()'s Exceptions as
   * RuntimeExceptions.
   */
  public @Nullable T getOrThrow(long timeout, TimeUnit unit) {
    try {
      return get(timeout, unit);
    } catch (InterruptedException | ExecutionException | TimeoutException e) {
      throw new RuntimeException(e);
    }
  }

  private void checkNotSet() {
    if (mReadyLatch.getCount() == 0) {
      throw new RuntimeException("Result has already been set!");
    }
  }
}
```
