## IncompatibleClassChangeError using Kotlin with Java

## Stack Trace
java.lang.IncompatibleClassChangeError: Expected 'java.lang.String com.example.jubin.componiontest.Child.TAG' to be a instance field rather than a static field (declaration of 'com.example.jubin.componiontest.Child' appears in /data/app/com.example.jubin.componiontest--ve-LsY3llWd0XWRXem1Kw==/base.apk)
       at com.example.jubin.componiontest.Child.visitInner(Child.kt:12)
       at com.example.jubin.componiontest.MainActivity$onCreate$1.onClick(MainActivity.kt:15)
       at android.view.View.performClick(View.java:6597)
       at android.view.View.performClickInternal(View.java:6574)
       at android.view.View.access$3100(View.java:778)
       at android.view.View$PerformClick.run(View.java:25885)
       at android.os.Handler.handleCallback(Handler.java:873)
       at android.os.Handler.dispatchMessage(Handler.java:99)
       at android.os.Looper.loop(Looper.java:193)
       at android.app.ActivityThread.main(ActivityThread.java:6680)
       at java.lang.reflect.Method.invoke(Native Method)
       at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
       at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:858)

## Symptom
The codes can compile, but will crash at runtime.

## Code Snippet
### Parent.java
```Java
package com.example.jubin.componiontest2;

public abstract class Parent {
    protected final String TAG = "ParentTag";

    public abstract void visitInner();
}
```

### Child.kt
```Java
package com.example.jubin.componiontest2

import android.util.Log

class Child : Parent() {
    companion object {
        private val TAG = "ChildTag"
    }

    override fun visitInner() {

        Log.d(TAG, "CHILD, tag: ${Child.TAG}")
    }
}
```
## Cause
`TAG` in `Log.d(TAG, "CHILD, tag: ${Child.TAG}")` refer to `TAG` in parent instead of local one. The comparison result of bytecode is :
![Bytecode Comparison](/../assets/pic/20181202-compare.png)
