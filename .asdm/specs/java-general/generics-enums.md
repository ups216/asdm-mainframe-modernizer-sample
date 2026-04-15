# Generics and Enums

Best practices for generics, enums, and annotations.

## Don't Use Raw Types

### Always Use Type Parameters

```java
// ❌ Bad: Raw type - no type safety
List names = new ArrayList();
names.add("John");
String name = (String) names.get(0); // Cast required

// ✅ Good: Generic type - compile-time type safety
List<String> names = new ArrayList<>();
names.add("John");
String name = names.get(0); // No cast needed

// ✅ Good: Use unbounded wildcard for unknown type
Collection<?> c = new ArrayList<String>();
// Can only read as Object, cannot add
```

## Eliminate Unchecked Warnings

### Fix Warnings, Don't Suppress

```java
// ❌ Bad: Warning about unchecked cast
Set<Lark> exaltation = new HashSet(); // Warning

// ✅ Good: Use diamond operator
Set<Lark> exaltation = new HashSet<>(); // No warning

// If suppression is necessary, document why
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    // Safe because array is created with correct type
    return (T[]) Arrays.copyOf(elements, size, a.getClass());
}
```

## Prefer Lists to Arrays

### Use Lists Instead of Arrays

```java
// ❌ Bad: Covariant arrays - runtime error possible
Object[] objects = new Long[1];
objects[0] = "string"; // ArrayStoreException at runtime

// ✅ Good: Invariant lists - compile-time safety
List<Object> objects = new ArrayList<Long>(); // Compile error
List<Long> longs = new ArrayList<>();
List<? extends Number> numbers = longs; // OK

// Array creation with generics not allowed
// ❌ Bad: List<String>[]
List<String>[] stringLists = new List<String>[1]; // Won't compile

// ✅ Good: Use List of List
List<List<String>> stringLists = new ArrayList<>();
```

## Favor Generic Types

### Make Types Generic

```java
// ❌ Bad: Object-based collection
public class Stack {
    private Object[] elements;
    private int size;
    
    public void push(Object e) { ... }
    public Object pop() { ... }
}

// ✅ Good: Generic collection
public class Stack<E> {
    private E[] elements;
    private int size;
    
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[16]; // Safe cast
    }
    
    public void push(E e) { ... }
    public E pop() { ... }
}
```

## Favor Generic Methods

### Make Methods Generic

```java
// ❌ Bad: Cast required by caller
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}

// ✅ Good: Generic method
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}

// Generic factory method
public static <K, V> HashMap<K, V> newHashMap() {
    return new HashMap<>();
}
```

## Bounded Wildcards for API Flexibility

### Use PECS: Producer-Extends, Consumer-Super

```java
// ✅ Good: Extends for producer (read-only)
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}

// ✅ Good: Super for consumer (write-only)
public void popAll(Collection<? super E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}

// Example usage
Stack<Number> stack = new Stack<>();
Iterable<Integer> integers = ...;
stack.pushAll(integers); // Integer extends Number

Collection<Object> objects = ...;
stack.popAll(objects); // Object is supertype of Number
```

## Generics and Varargs

### Use @SafeVarargs for Generic Varargs

```java
// ✅ Good: Safe use of generic varargs
@SafeVarargs
public static <T> List<T> asList(T... elements) {
    List<T> result = new ArrayList<>();
    for (T element : elements) {
        result.add(element);
    }
    return result;
}

// ❌ Bad: Unsafe generic varargs - heap pollution
@SafeVarargs
public static <T> T[] toArray(T... elements) {
    return elements; // Returns array - unsafe!
}
```

## Use Enums Instead of Int Constants

### Replace Int Constants with Enums

```java
// ❌ Bad: Int constants
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

// ✅ Good: Enum
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }

// Enum with data and behavior
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6);
    
    private final double mass;
    private final double radius;
    private final double surfaceGravity;
    
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

## Instance Fields Instead of Ordinals

### Don't Use ordinal()

```java
// ❌ Bad: Using ordinal
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET;
    
    public int numberOfMusicians() {
        return ordinal() + 1; // Fragile!
    }
}

// ✅ Good: Instance field
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5);
    
    private final int numberOfMusicians;
    
    Ensemble(int size) {
        this.numberOfMusicians = size;
    }
    
    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```

## EnumSet and EnumMap

### Use Specialized Collections for Enums

```java
// ✅ Good: EnumSet for enum subsets
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    public void applyStyles(Set<Style> styles) { ... }
}

// Usage
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));

// ✅ Good: EnumMap for enum-keyed maps
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
    new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
}
```

## Extensible Enums with Interfaces

### Use Interfaces for Extensible Enums

```java
// ✅ Good: Interface for extensible enum
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") { public double apply(double x, double y) { return x + y; } },
    MINUS("-") { public double apply(double x, double y) { return x - y; } };
    
    private final String symbol;
    BasicOperation(String symbol) { this.symbol = symbol; }
    @Override public String toString() { return symbol; }
}

// Extension in another package
public enum ExtendedOperation implements Operation {
    EXP("^") { public double apply(double x, double y) { return Math.pow(x, y); } };
    
    private final String symbol;
    ExtendedOperation(String symbol) { this.symbol = symbol; }
}
```

## Prefer Annotations to Naming Patterns

### Use Annotations for Metadata

```java
// ✅ Good: Annotation definition
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
    Class<? extends Throwable> expected() default NoException.class;
}

// ✅ Good: Annotation usage
public class FooTest {
    @Test
    public void testNormal() { ... }
    
    @Test(expected = ArithmeticException.class)
    public void testException() {
        int i = 1 / 0;
    }
}
```

## Consistently Use @Override

### Always Use @Override for Override Methods

```java
public class Bigram {
    private final char first;
    private final char second;
    
    // ✅ Good: @Override catches errors
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Bigram)) return false;
        Bigram b = (Bigram) o;
        return b.first == first && b.second == second;
    }
    
    // Without @Override, typo would create overload instead of override
    // public boolean equals(Bigram b) { ... } // Mistake!
}
```

## Marker Interfaces

### Use Marker Interfaces for Type Definition

```java
// ✅ Good: Marker interface
public interface Serializable {
    // No methods - just marks the type
}

// Can be checked at compile time
if (obj instanceof Serializable) { ... }

// ✅ Alternative: Marker annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Serializable { }
```
