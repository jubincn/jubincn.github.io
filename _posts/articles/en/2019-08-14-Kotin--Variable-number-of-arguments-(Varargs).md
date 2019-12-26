## IllegalFormatConversionException
IllegalFormatConversionException was thrown when call TextView.setText2(R.strinig.placeholder, 2) and it works well with TextView.setText1(R.strinig.placeholder, 2). The key to resolve this error is use of  spread operator.


#### Original Kotlin Codes
```
private fun TextView.setText1(@StringRes resId: Int, vararg formatArgs: Any) {
     this.context.getString(resId, *formatArgs)
}

private fun TextView.setText2(@StringRes resId: Int, vararg formatArgs: Any) {
    this.context.getString(resId, formatArgs)
}

private fun TextView.setText3(@StringRes resId: Int, count: Any) {
    this.context.getString(resId, count)
}
```

#### Decompiled java code
```
public final class DemoKt {
   private static final void setText1(@NotNull TextView $this$setText1, @StringRes int resId, Object... formatArgs) {
      $this$setText1.getContext().getString(resId, Arrays.copyOf(formatArgs, formatArgs.length));
   }

   private static final void setText2(@NotNull TextView $this$setText2, @StringRes int resId, Object... formatArgs) {
      $this$setText2.getContext().getString(resId, new Object[]{formatArgs});
   }

   private static final void setText3(@NotNull TextView $this$setText3, @StringRes int resId, Object count) {
      $this$setText3.getContext().getString(resId, new Object[]{count});
   }
}
```


### References
https://stackoverflow.com/questions/40025639/android-kotlin-stringres-quantitystring
https://kotlinlang.org/docs/reference/functions.html#variable-number-of-arguments-varargs
