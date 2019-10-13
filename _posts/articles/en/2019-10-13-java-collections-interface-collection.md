# Java Collection Framework

## Key elements in Collection Framework
1. Interfaces
2. Implementations
3. Algorithms

## Interfaces
- Collection
  - Set
    - SortedSet
  - List
  - Queue
  - Deque
- Map
  - SortedMap

### The Collection Interface

#### Collection Basic Operations
- size()
- isEmpty()
- contains()
- add(Object o)
- remove(Object o)
- iterator()

#### Collection Bulk Operations
- containsAll()
- addAll()
- removeAll()
- retainAll()
- clear()

#### Traverse Collections
##### Aggregate Operations
In JDK 8 and later, the preferred method of iterating over a collection is to obtain a stream and perform aggregate operations on it.

- stream()

```
myShapesCollection.stream()
  .filter(e -> e.getColor() == Color.RED)
  .forEach(e -> System.out.println(e.getName()));
```

- parallelStream()

```
myShapesCollection.parallelStream()
  .filter(e -> e.getColor() == Color.RED)
  .forEach(e -> System.out.println(e.getName()));
```

- map() & collect()

```
String joined = elements.stream()
    .map(Object::toString)
    .collect(Collectors.joining(", "));
```

- collect()

```
int total = employees.stream()
  .collect(Collectors.summingInt(Employee::getSalary)));
```

##### For-each Construct

```
for (Object o : collection)
    System.out.println(o);
```    

##### Iterator

An Iterator is an object that enables you to traverse through a collection and to remove elements from the collection selectively, if desired.
```
public interface Iterator<E> {
    boolean hasNext();
    E next();
    void remove(); //optional
}
```
Use Iterator instead of the for-each construct when you need to:
- Remove the current element.
- Iterate over multiple collections in parallel.
```
static void filter(Collection<?> c) {
    for (Iterator<?> it = c.iterator(); it.hasNext(); )
        if (!cond(it.next()))
            it.remove();
}
```

#### Collection Array Operations
- toArray()
  - return a new Object Array containing elements in old one
- toArray(T[] a)
  - return a new T Array containing elements in old one
