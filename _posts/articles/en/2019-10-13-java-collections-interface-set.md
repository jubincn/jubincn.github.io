### The Set Interface
A Set is a Collection that cannot contain duplicate elements. To achieve this, Set applies a stronger contract on `equals()` and `hashCode()` method.

The Java platform contains three general-purpose Set implementations: HashSet, TreeSet, and LinkedHashSet.
- HashSet
  - Hash based, best performing implementation.
  - Not sorted.
  - Insertion order not kept.
- TreeSet
  - Substantially slower than HashSet
  - Sorted.
  - Insertion order not kept.
- LinkedHashSet
  - A little bit higher cost.
  - Not sorted.
  - Insertion order kept.

Set is often used to filter duplicate element in Collections:
- Using constructor

```
Collection<Type> noDups = new HashSet<Type>(c);
```

- Stream API

```
c.stream().collect(Collectors.toSet()); // no duplicates
```

### Set Interface Basic Operations
- add()
  - return true if this set contains the specified element.
- remove()
  - return true if this set contains the specified element.

### Set Interface Bulk Operations
- `s1.containsAll(s2)`: Tell if s2 is *subset* of s1
- `s1.addAll(s2)`: s1 *union* s2
- `s1.retainAll(s2)`: s1 *intersect* s2
- `s1.removeAll(s2)`: s1 *difference* s2

### Set Interface Array Operations
