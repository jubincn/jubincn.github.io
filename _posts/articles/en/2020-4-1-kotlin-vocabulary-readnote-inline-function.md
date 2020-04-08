## When to use inline function
1. For function with higher order function (lambda), make the function inline will save some memory by avoiding Function object creation.

2. If the reference for higher order function is used, inline can't be applied to it.

3. Inline a function will grow the size of app, and then better do it when the function is simple and contains only few lines of codes.

4. noinline to annotate which higher order function should not converted to inline


## References
[Inline functions - Kotlin Vocabulary](https://www.youtube.com/watch?v=wAQCs8-a6mg)
