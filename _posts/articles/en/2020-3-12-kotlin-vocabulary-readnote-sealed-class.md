## Sealed classes
### Sealed class vs Enum class vs Ordinary class
#### Enum class
##### pros
* Used in when block, IDE will help check all branches.

##### cons
* Can't contain more infomation

#### Ordinary class
##### pros
* Flexible, add any state to it.

##### cons
* Can't use as enums

#### Sealed class
* Easy to extend, work as enums, has IDE help

### Sealed Class
```kotlin
sealed class Result<out T : Any> {
    class Success<out T : Any>(val data: T) : Result<T>()
    sealed class Error(val exception: Exception) : Result<Nothing>() {
        class RecoverableError(exception: Exception) : Error(exception)
        class NonRecoverableError(exception: Exception) : Error(exception)
    }

    object InProgress : Result<Nothing>()
}

fun handleResult(result: Result<Int>) {
    val action = when (result) {
        is Result.Success -> TODO()
        is Result.Error.RecoverableError -> TODO()
        is Result.Error.NonRecoverableError -> TODO()
        Result.InProgress -> TODO()
    }
}

```
### How sealed class do in jvm
Let's take a look at decompiled java code for sealed class above:
#### Decompiled Java code (excerpts)
```java
public abstract class Result {
   private Result() {
   }

   // $FF: synthetic method
   public Result(DefaultConstructorMarker $constructor_marker) {
      this();
   }

   @Metadata(
      mv = {1, 1, 16},
      bv = {1, 0, 3},
      k = 1,
      d1 = {"\u0000\u0012\n\u0002\u0018\u0002\n\u0000\n\u0002\u0010\u0000\n\u0002\u0018\u0002\n\u0002\b\u0006\u0018\u0000*\n\b\u0001\u0010\u0001 \u0001*\u00020\u00022\b\u0012\u0004\u0012\u0002H\u00010\u0003B\r\u0012\u0006\u0010\u0004\u001a\u00028\u0001¢\u0006\u0002\u0010\u0005R\u0013\u0010\u0004\u001a\u00028\u0001¢\u0006\n\n\u0002\u0010\b\u001a\u0004\b\u0006\u0010\u0007¨\u0006\t"},
      d2 = {"LResult$Success;", "T", "", "LResult;", "data", "(Ljava/lang/Object;)V", "getData", "()Ljava/lang/Object;", "Ljava/lang/Object;", "coroutines"}
   )
   public static final class Success extends Result {
      @NotNull
      private final Object data;

      @NotNull
      public final Object getData() {
         return this.data;
      }

      public Success(@NotNull Object data) {
         Intrinsics.checkParameterIsNotNull(data, "data");
         super((DefaultConstructorMarker)null);
         this.data = data;
      }
   }
   ...
}
```
0. All child class info are in Metadata.
1. Default constructor of Result is private.
2. public constructor of Result contain a param with type DefaultConstructorMarker.
3. DefaultConstructorMarker is package private and thus application has no access to it.

#### DefaultConstructorMarker
```
package kotlin.jvm.internal;

final class DefaultConstructorMarker {
    private DefaultConstructorMarker() {}
}
```
It's Java file with package visibility, and as a result, application code can't use it.
I don't know why the kotlin class have access to it, and one of my guess is these markers are read by kotlin compiler and will be removed in binary codes since they are only internal markers.
