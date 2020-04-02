# ReadNote of Collections and sequences in Kotlin
Kotlin Standard Library offers two ways of working with containers: eagerly - with collections, and lazily - with sequences.

## Eagerly vs Lazily
Eagerly: All elements in a collection will do next transformation immediately.
Lazily: All transformation will be executed immediately for elements in sequence one by one.

### Collection transformation vs Sequence transformation
![Eagerly vs Lazily] ({{ site.url }}/assets/eagerly_vs_lazily.gif)

### How it works
#### map function from Collection
For Collection, the map function will execute immediately.
```Kotlin
public inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R> {
    return mapTo(ArrayList<R>(collectionSizeOrDefault(10)), transform)
}

public inline fun <T, R, C : MutableCollection<in R>> Iterable<T>.mapTo(destination: C, transform: (T) -> R): C {
    for (item in this)
        destination.add(transform(item))
    return destination
}

```
### map function from Sequence
Map function from Sequence is an intermediate function which will wrap transformation to a TransformingSequence object.
```Kotlin
public fun <T, R> Sequence<T>.map(transform: (T) -> R): Sequence<R> {
    return TransformingSequence(this, transform)
}

public inline fun <T> Sequence<T>.first(predicate: (T) -> Boolean): T {
    for (element in this) if (predicate(element)) return element
    throw NoSuchElementException("Sequence contains no element matching the predicate.")
}
```

#### intermediate function vs terminal function
Intermediate function like map() will not do transformation immediately, instead, it will return an intermediate object.

Terminal function like first() will execute transformation as soon as it is called.


### Performance
The order of transformation matters. Transformation in Collection is kind of like bfs, while sequence is dfs.



## References
[Collections and sequences in Kotlin] (https://medium.com/androiddevelopers/collections-and-sequences-in-kotlin-55db18283aca)
