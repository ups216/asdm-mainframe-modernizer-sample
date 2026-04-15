# Lambdas and Streams

Best practices for lambda expressions, method references, and stream operations.

## Prefer Lambdas to Anonymous Classes

### Use Lambdas for Functional Interfaces

```java
// ❌ Bad: Anonymous class
Collections.sort(words, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});

// ✅ Good: Lambda expression
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));

// ✅ Better: Comparator method
Collections.sort(words, Comparator.comparingInt(String::length));

// ✅ Best: List.sort with method reference
words.sort(Comparator.comparingInt(String::length));
```

## Prefer Method References to Lambdas

### Use Method References for Readability

```java
// ❌ Lambda where method reference is clearer
map.merge(key, 1, (count, incr) -> count + incr);

// ✅ Method reference
map.merge(key, 1, Integer::sum);

// Method reference types
String::toLowerCase          // Virtual method reference: s -> s.toLowerCase()
String::compareToIgnoreCase  // Virtual method reference: (s1, s2) -> s1.compareToIgnoreCase(s2)
String::new                  // Constructor reference: s -> new String(s)
String[]::new               // Array constructor reference: n -> new String[n]
```

## Use Standard Functional Interfaces

### Prefer Standard Interfaces Over Custom

```java
// ✅ Standard functional interfaces in java.util.function
// UnaryOperator<T>  - T apply(T t)
// BinaryOperator<T> - T apply(T t1, T t2)
// Predicate<T>      - boolean test(T t)
// Function<T,R>     - R apply(T t)
// Supplier<T>       - T get()
// Consumer<T>       - void accept(T t)

// Use specialized primitive interfaces when possible
IntPredicate even = i -> i % 2 == 0;  // Avoids boxing

// ❌ Bad: Custom interface when standard exists
interface UnaryOperator<T> {
    T apply(T arg);
}

// ✅ Good: Use standard java.util.function.UnaryOperator
```

## Use Streams Judiciously

### Know When to Use Streams vs Loops

```java
// ✅ Good: Stream for data transformation pipeline
List<String> topNames = names.stream()
    .map(String::trim)
    .filter(s -> !s.isEmpty())
    .map(String::toUpperCase)
    .limit(10)
    .collect(Collectors.toList());

// ✅ Good: Traditional loop for complex logic
for (Card card : cards) {
    if (card.rank() == Rank.ACE) {
        // Complex logic that's hard to express in stream
        if (card.suit() == Suit.SPADES) {
            // Multiple side effects or complex conditions
        }
    }
}

// ✅ Good: Combine streams with traditional code
public boolean isAnagram(String s1, String s2) {
    if (s1.length() != s2.length()) return false;
    
    return s1.chars()
        .sorted()
        .boxed()
        .collect(Collectors.toList())
        .equals(
            s2.chars()
                .sorted()
                .boxed()
                .collect(Collectors.toList())
        );
}
```

## Prefer Side-Effect-Free Functions

### Avoid Side Effects in Stream Pipelines

```java
// ❌ Bad: Side effects in stream
List<String> result = new ArrayList<>();
stream.filter(s -> {
    result.add(s);  // Side effect!
    return s.length() > 3;
});

// ✅ Good: Pure function with collector
List<String> result = stream
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());

// ❌ Bad: Modifying state in forEach
Map<String, Integer> counts = new HashMap<>();
words.forEach(w -> counts.merge(w, 1, Integer::sum)); // Side effect

// ✅ Good: Use collectors for grouping
Map<String, Long> counts = words.stream()
    .collect(Collectors.groupingBy(w -> w, Collectors.counting()));
```

## Prefer Collection to Stream as Return Type

### Return Collection for Flexibility

```java
// ✅ Good: Return Collection or Iterable
public Collection<ProcessHandle> getProcesses() {
    return ProcessHandle.allProcesses()
        .collect(Collectors.toList());
}

// ❌ Bad: Return Stream when Collection is needed
public Stream<ProcessHandle> getProcesses() {
    return ProcessHandle.allProcesses(); // Can't iterate twice
}

// ✅ Good: Stream for truly lazy or infinite sequences
public Stream<BigInteger> primes() {
    return Stream.iterate(BigInteger.valueOf(2), BigInteger::nextProbablePrime);
}
```

## Use Caution with Parallel Streams

### Parallel Streams Are Not Always Faster

```java
// ❌ Bad: Parallel for small or sequential operations
stream.parallel()  // May be slower due to overhead
    .filter(...)
    .collect(...);

// ✅ Good: Parallel for CPU-intensive operations on large datasets
long count = primes().parallel()
    .map(p -> BigInteger.valueOf(p).nextProbablePrime())
    .limit(100_000)
    .count();

// ✅ Good: Parallel with ArrayList, HashMap, array (splittable)
list.parallelStream()
    .filter(...)
    .collect(...);

// ❌ Bad: Parallel with LinkedList, Stream.iterate (poor splitting)
Stream.iterate(0, i -> i + 1)
    .parallel()  // Won't help much
    .limit(100000)
    ...
```

## Common Stream Patterns

### Mapping and Filtering

```java
// Map to new type
List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList());

// FlatMap for nested structures
List<String> characters = words.stream()
    .flatMap(word -> word.chars().mapToObj(c -> String.valueOf((char) c)))
    .distinct()
    .collect(Collectors.toList());

// Filter with predicate
List<String> longWords = words.stream()
    .filter(w -> w.length() > 5)
    .collect(Collectors.toList());
```

### Reduction Operations

```java
// Simple reduce
int sum = numbers.stream().reduce(0, Integer::sum);

// Collect to various types
Set<String> set = words.stream().collect(Collectors.toSet());
Map<String, Integer> map = words.stream()
    .collect(Collectors.toMap(w -> w, String::length));

// Grouping
Map<Category, List<Product>> byCategory = products.stream()
    .collect(Collectors.groupingBy(Product::category));

// Partitioning
Map<Boolean, List<Integer>> byEvenOdd = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

// Statistics
IntSummaryStatistics stats = numbers.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));
// stats.getCount(), getSum(), getAverage(), getMin(), getMax()
```

### Finding and Matching

```java
// Find first
Optional<String> first = words.stream()
    .filter(w -> w.startsWith("A"))
    .findFirst();

// Find any (better for parallel)
Optional<String> any = words.parallelStream()
    .filter(w -> w.length() > 10)
    .findAny();

// Match operations
boolean anyMatch = words.stream().anyMatch(w -> w.isEmpty());
boolean allMatch = words.stream().allMatch(w -> w.length() > 0);
boolean noneMatch = words.stream().noneMatch(w -> w.isEmpty());
```

### Working with Optionals

```java
// ✅ Good: OrElse for default value
String result = optional.orElse("default");

// ✅ Good: OrElseGet for lazy default
String result = optional.orElseGet(() -> computeDefault());

// ✅ Good: OrElseThrow for exception
String result = optional.orElseThrow(() -> new NoSuchElementException());

// ✅ Good: IfPresent for side effects
optional.ifPresent(value -> System.out.println(value));

// ✅ Good: Map and filter on Optional
Optional<Integer> length = optional
    .filter(s -> !s.isEmpty())
    .map(String::length);

// ✅ Good: FlatMap for nested Optionals
Optional<User> user = Optional.ofNullable(id)
    .flatMap(repository::findById);
```
