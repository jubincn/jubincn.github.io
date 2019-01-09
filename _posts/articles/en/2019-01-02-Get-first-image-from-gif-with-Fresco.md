## Get first image from GIF with Fresco Sync & Async
Fresco did not provides a Sync way to get bitmap from DataSource, but we can use SimpleSettableFuture from ReactNative. However, we can write our own SettableFuture like what ReactNative done.

### Sync, With Fresco
```kotlin
object FrescoUtils {
    private const val TAG = "FrescoUtils"

    // Inspired by https://github.com/facebook/fresco/issues/2229#issuecomment-434061873
    @WorkerThread
    fun getBitmapFromAnimatedImageUri(gifUri: Uri, timeout: Long, unit: TimeUnit): Bitmap? {
        val request = ImageRequest.fromUri(gifUri) ?: return null

        // ATTENTION!!! future.set() should be called
        val future = SimpleSettableFuture<Bitmap?>()

        val dataSource =Fresco.getImagePipeline().fetchDecodedImage(request, "animatedImage")
        if (dataSource != null) {
            dataSource.subscribe(object : BaseDataSubscriber<CloseableReference<CloseableImage?>?>() {
                override fun onNewResultImpl(paramDataSource: DataSource<CloseableReference<CloseableImage?>?>?) {
                    if (paramDataSource != null && paramDataSource.hasResult()) {
                        paramDataSource.result?.get()?.use {
                            if (it is CloseableAnimatedImage) {
                                val bitmap = FrescoUtils.getFirstBitmapFromAnimatedImage(it)
                                future.set(bitmap)
                            }
                        }
                    }
                }

                override fun onCancellation(dataSource: DataSource<CloseableReference<CloseableImage?>?>?) {
                    super.onCancellation(dataSource)
                    future.set(null)
                }

                override fun onFailureImpl(dataSource: DataSource<CloseableReference<CloseableImage?>?>) {
                    future.set(null)
                }
            }, UiThreadImmediateExecutorService.getInstance())
        } else {
            future.set(null)
        }

        return try {
                future.get(timeout, unit)
            } catch (e: Exception) {
                null
            }
    }

    @WorkerThread
    fun getFirstBitmapFromAnimatedImage(animatedImg: CloseableAnimatedImage?): Bitmap? {
        if (animatedImg == null
            || animatedImg.imageResult == null
            || animatedImg.imageResult.image == null
            || animatedImg.imageResult.image.frameDurations == null
            || animatedImg.width <= 0
            || animatedImg.height <= 0
        ) {
            return null
        }

        var imageFrame: AnimatedImageFrame? = null
        var bitmap: Bitmap? = null
        try {
            bitmap = Bitmap.createBitmap(
                animatedImg.width,
                animatedImg.height,
                Bitmap.Config.ARGB_8888
            )
            imageFrame = animatedImg.image?.getFrame(0)

            imageFrame?.renderFrame(imageFrame.width, imageFrame.height, bitmap)
        } catch (e: IllegalArgumentException) {
            Log.printErrStackTrace(TAG, e, "exception when create bitmap from ImageFrame")
        } finally {
            imageFrame?.dispose()
        }

        return bitmap
    }
}
```

### SimpleSettableFuture from ReactNative
```java
/**
 * Copyright (c) 2015-present, Facebook, Inc.
 * All rights reserved.
 *
 * This source code is licensed under the BSD-style license found in the
 * LICENSE file in the root directory of this source tree. An additional grant
 * of patent rights can be found in the PATENTS file in the same directory.
 */

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
