## Detect Local Uri in Android

```kotlin
object FileUtils {
    private val LOCAL_URI_PREFIXES = Arrays.asList(
        "file://", "content://"
    )

    fun isLocalUri(uriStr: String): Boolean {
        for (localPrefix in LOCAL_URI_PREFIXES) {
            if (uriStr.startsWith(localPrefix)) {
                return true
            }
        }
        return false
    }

    fun isLocalUri(uri: Uri): Boolean {
        return isLocalUri(uri.toString())
    }
}
```
