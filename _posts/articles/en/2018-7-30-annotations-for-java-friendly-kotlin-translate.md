
[原文地址](https://proandroiddev.com/annotations-for-your-java-friendly-kotlin-code-badfbedec14)

对于部分使用kotlin，部分使用java的android项目而言，java和kotlin的相互调用是个大坑。这篇文章讲了kotlin里常用的与java调用相关的注解。

## Java注解
### JvmField
#### 用途
让Kotlin编译器将这个字段编译为public，而不是生成getter/setter。

#### 常用场景
在Companion或其他object中的字段。

#### 原理
看下源码，简单说就是getter/setter -> public field

---
##### 没有JvmField的时候

###### object
```
object Constants {    
    val PERMISSIONS = listOf("Internet", "Location")
}
```


```
public final class Constant {
   @NotNull
   private static final List PERMISSIONS;
   public static final Constant INSTANCE;

   @NotNull
   public final List getPERMISSIONS() {
      return PERMISSIONS;
   }

   static {
      Constant var0 = new Constant();
      INSTANCE = var0;
      PERMISSIONS = CollectionsKt.listOf(new String[]{"Internet", "Location"});
   }
}
```
###### class
```
class Constant {
    val PERMISSIONS = listOf("Internet", "Location")
}
```
```
public final class Constant {
   @NotNull
   private final List PERMISSIONS = CollectionsKt.listOf(new String[]{"Internet", "Location"});

   @NotNull
   public final List getPERMISSIONS() {
      return this.PERMISSIONS;
   }
}
```

##### 有JvmField的时候
###### object
```
object Constant {
    @JvmField
    val PERMISSIONS = listOf("Internet", "Location")
}
```

```
public final class Constant {
   @JvmField
   @NotNull
   public static final List PERMISSIONS;
   public static final Constant INSTANCE;

   static {
      Constant var0 = new Constant();
      INSTANCE = var0;
      PERMISSIONS = CollectionsKt.listOf(new String[]{"Internet", "Location"});
   }
}
```
###### class
```
class Constant {
    @JvmField
    val PERMISSIONS = listOf("Internet", "Location")
}
```
```
public final class Constant {
   @JvmField
   @NotNull
   public final List PERMISSIONS = CollectionsKt.listOf(new String[]{"Internet", "Location"});
}

```
### JvmStatic
* 用在方法上：生成一个静态方法。
* 用在字段上：生成静态的getter和setter

#### 用在方法上
不带@JvmStatic的时候，这个方法是一个实例方法，调用时需要使用Utils.INSTANCE.doSomething()

```
object Utils {
    fun doSomething(){  }
}
```
```
public final class Utils {
   public static final Utils INSTANCE;

   public final void doSomething() {
   }

   static {
      Utils var0 = new Utils();
      INSTANCE = var0;
   }
}

```
使用JvmStatic后，Kotlin编译器会将此方法转为static方法，此时可以直接用Utils.doSomething()

```
object Utils {
    @JvmStatic
    fun doSomething(){  }
}
```

```
public final class Utils {
   public static final Utils INSTANCE;

   @JvmStatic
   public static final void doSomething() {
   }

   static {
      Utils var0 = new Utils();
      INSTANCE = var0;
   }
}
```
#### 用在变量上
```
object Utils {
    var values = listOf("Test 1", "Test 2")
}
```

```
public final class Utils {
   @NotNull
   private static List values;
   public static final Utils INSTANCE;

   @NotNull
   public final List getValues() {
      return values;
   }

   public final void setValues(@NotNull List var1) {
      Intrinsics.checkParameterIsNotNull(var1, "<set-?>");
      values = var1;
   }

   static {
      Utils var0 = new Utils();
      INSTANCE = var0;
      values = CollectionsKt.listOf(new String[]{"Test 1", "Test 2"});
   }
}
```
对于变量，JvmStatic的作用是将setter和getter改为static方法。
```
object Utils {
    @JvmStatic
    var values = listOf("Test 1", "Test 2")
}
```
```
public final class Utils {
   @NotNull
   private static List values;
   public static final Utils INSTANCE;  

   @NotNull
   public static final List getValues() {
      return values;
   }

   public static final void setValues(@NotNull List var0) {
      Intrinsics.checkParameterIsNotNull(var0, "<set-?>");
      values = var0;
   }

   static {
      Utils var0 = new Utils();
      INSTANCE = var0;
      values = CollectionsKt.listOf(new String[]{"Test 1", "Test 2"});
   }
}
```
#### 不能使用JvmStatic的场景
当一个成员变量含有open, override或const的modifier时，不能使用@JvmStatic注解。

### JvmOverloads
在Kotlin中，有默认构造方法，这些方法在使用kotlin调用的时候很方便，相当于定义了多个构造函数。但使用Java调用时，却最多只能用两个：
1. 含有全部参数的构造函数。
2. 在所有参数都有默认参数的时候，可以调用它的无参构造函数。

#### Kotlin中带默认参数构造函数的使用
下面这个构造函数，三个参数都带有默认值
```
class User constructor (
        val name: String = "Test",
        val lastName: String = "Testy",
        val age: Int = 0
)
```
使用时可以这么用：
```
val user1 = User()
val user2 = User(name = "Bruno")
val user3 = User(name = "Bruno", lastName = "Aybar")
val user4 = User(name = "Bruno", lastName = "Aybar", age = 21)
val user5 = User(lastName = "Aybar")
val user6 = User(lastName = "Aybar", age = 21)
val user7 = User(age = 21)
val user8 = User(age = 21, name = "Bruno")
```
@JvmOverloads能部分解决这个问题，使用@JvmOverloads后，这里会多出三个构造方法
```
public final class User {
   @JvmOverloads
   public User(@NotNull String name, @NotNull String lastName) {
      this(name, lastName, 0, 4, (DefaultConstructorMarker)null);
   }

   @JvmOverloads
   public User(@NotNull String name) {
      this(name, (String)null, 0, 6, (DefaultConstructorMarker)null);
   }

   @JvmOverloads
   public User() {
      this((String)null, (String)null, 0, 7, (DefaultConstructorMarker)null);
   }
}
```
### @file:JvmName @JvmName @get:JvmName
给某个文件或者方法，字段换一个Java调用时使用的名字。
#### @file:JvmName
```
//file name: Utils.kt
fun doSomething() { ... }
```
在Java中可以这么调用：
```
UtilsKt.doSomething();
```
加上@file:JvmName("Utils")后，就可以使用Utils这个名字来调用了。
```
//file name: Utils.kt
@file:JvmName("Utils")
fun doSomething() { ... }
```
在Java中调用：
```
Utils.doSomething();
```
方法名也可以改，例如：
```
//file name: Utils.kt
@file:JvmName("Utils")
@JvmName("doSomethingElse")
fun doSomething() { ... }
```
在Java中调用：
```
//Java
Utils.doSomethingElse();
```
在Kotlin中调用：
```
//Kotlin
Utils.doSomething()
```
