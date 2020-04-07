## How Java Enum works in switch statement
### Look Into Bytecode
I write a demo code which will be compiled to bytecode later, which will show how switch with Enum works internally.
```Java
public class EnumDemo {

    enum BlendMode {FADE, ADD, MIX}

    public static void main(String[] args) {
        testSwitch(BlendMode.ADD);
    }

    private static void testSwitch(BlendMode mode) {
        switch (mode) {
            case FADE:
                doSomething("fade");
                break;
            case ADD:
                doSomething("add");
                break;
            case MIX:
                doSomething("mix");
                break;
        }
    }

    private static void doSomething(String s) {

    }
}
```
The switch case part of bytecode:
```Java bytecode
public class tech/jubin/demo/EnumDemo {
  ... truncated ...

  // access flags 0xA
  private static testSwitch(Ltech/jubin/demo/EnumDemo$BlendMode;)V
   L0
    LINENUMBER 12 L0
    GETSTATIC tech/jubin/demo/EnumDemo$1.$SwitchMap$tech$jubin$demo$EnumDemo$BlendMode : [I
    ALOAD 0
    INVOKEVIRTUAL tech/jubin/demo/EnumDemo$BlendMode.ordinal ()I
    IALOAD
    TABLESWITCH
      1: L1
      2: L2
      3: L3
      default: L4

      ... truncated ...
```
From the bytecode we can find that for switch statement, a mapping is created first and switch will use enum's ordinal for TABLESWITCH.

### Why The Mapping is necessary
Unlike Android application, Java application might not compiled together. In this case, if the orinal of enum from a library is changed, the application which depend on the library might break. To avoid this, javac will add a mapping to the place switch enum is used.


## How Kotlin Enum works in when statement
Kotlin Enum class will be translated to Java Enum and it also add a mapping to where the when statement with enum is called.

### when statement decompiled to Java code
These code snippet is from blog [When using enums and R8…](https://medium.com/androiddevelopers/when-using-enums-and-r8-3f8f314c0a13)
``` Java
public final class BlendingKt$WhenMappings {
    public static final int[] $EnumSwitchMapping$0 =
            new int[BlendMode.values().length];

    static {
        $EnumSwitchMapping$0[BlendMode.OPAQUE.ordinal()] = 1;
        $EnumSwitchMapping$0[BlendMode.TRANSPARENT.ordinal()] = 2;
        $EnumSwitchMapping$0[BlendMode.FADE.ordinal()] = 3;
        $EnumSwitchMapping$0[BlendMode.ADD.ordinal()] = 4;
    }
}
```

## R8 Optimization
Android application is different from general Java application in that all codes are compiled together, thus the ordinal can be fixed during compilation.

Proguard is a separate program which will run after dex, and as a result, it has no access to compile time resources and can't do the optimisation.

R8 is run after d8 and can be seen as a part of d8 compilation. Thus it has access to all compile time resources. As a result, the mappings are not necessary anymore and can be removed with R8, saving us runtime memory and speed up our application.

R8 optimized code:
```Java
public static void blend(@NotNull BlendMode b) {
    switch (b.ordinal()) {
        case 0: {
            src();
            break;
        }
        // ...
    }
}
```

## References
1. [D8, R8 and enums - Kotlin Vocabulary](https://www.youtube.com/watch?v=lTo03M2HzFY&list=PLWz5rJ2EKKc_T0fSZc9obnmnWcjvmJdw_&index=3)
2. [When using enums and R8…](https://medium.com/androiddevelopers/when-using-enums-and-r8-3f8f314c0a13)
3. [R8 Optimization: Enum Switch Maps](https://jakewharton.com/r8-optimization-enum-switch-maps/)
