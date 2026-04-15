# Creating and Destroying Objects

Best practices for object creation, singletons, builders, and object lifecycle management.

## Static Factory Methods

### Prefer Static Factory Methods Over Constructors

```java
// ✅ Good: Static factory method with descriptive name
public class User {
    private final String name;
    
    private User(String name) {
        this.name = name;
    }
    
    public static User of(String name) {
        return new User(name);
    }
    
    public static User defaultUser() {
        return new User("Anonymous");
    }
}

// Usage
User user = User.of("John");
User defaultUser = User.defaultUser();
```

### Advantages of Static Factory Methods

```java
public class Complex {
    private final double real;
    private final double imag;
    
    private Complex(double real, double imag) {
        this.real = real;
        this.imag = imag;
    }
    
    // Named constructors - more readable
    public static Complex fromCartesian(double real, double imag) {
        return new Complex(real, imag);
    }
    
    public static Complex fromPolar(double r, double theta) {
        return new Complex(r * Math.cos(theta), r * Math.sin(theta));
    }
    
    // Can return cached instances
    public static Complex zero() {
        return new Complex(0, 0);
    }
    
    // Can return subtypes
    public static Number parse(String s) {
        // Return appropriate Number subtype based on input
    }
}
```

## Builder Pattern

### Use Builder for Many Constructor Parameters

```java
// ✅ Good: Builder pattern for complex objects
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    
    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;
        
        // Optional parameters - initialized to defaults
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        public Builder calories(int val) {
            calories = val;
            return this;
        }
        
        public Builder fat(int val) {
            fat = val;
            return this;
        }
        
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
    
    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}

// Usage
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100)
    .sodium(35)
    .carbohydrate(27)
    .build();
```

## Singleton Pattern

### Enforce Singleton with Private Constructor or Enum

```java
// ✅ Approach 1: Enum singleton (recommended)
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() { ... }
}

// Usage
Elvis.INSTANCE.leaveTheBuilding();

// ✅ Approach 2: Static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    
    private Elvis() { }
    
    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```

## Prevent Instantiation

### Enforce Noninstantiability with Private Constructor

```java
// ✅ Good: Utility class that cannot be instantiated
public class UtilityClass {
    // Suppress default constructor
    private UtilityClass() {
        throw new AssertionError("Utility class cannot be instantiated");
    }
    
    public static int add(int a, int b) {
        return a + b;
    }
}
```

## Dependency Injection

### Prefer Dependency Injection Over Hardwiring Resources

```java
// ❌ Bad: Hardwired resource
public class SpellChecker {
    private static final Lexicon dictionary = new Lexicon();
    
    private SpellChecker() {}
    
    public static boolean isValid(String word) { ... }
}

// ✅ Good: Dependency injection
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) { ... }
}
```

## Avoid Unnecessary Objects

### Reuse Immutable Objects

```java
// ❌ Bad: Creates new String every time
String s = new String("stringette");

// ✅ Good: Reuses single String instance
String s = "stringette";

// ❌ Bad: Creates new object each call
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}

// ✅ Good: Use primitives when possible
int sum = 0;
for (int i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i; // Uses primitive int
}

// ❌ Bad: Uses boxed primitive
Integer sum = 0;
for (int i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i; // Creates many Integer objects
}
```

## Eliminate Obsolete References

### Memory Leak Prevention

```java
// ❌ Bad: Memory leak in stack implementation
public class Stack {
    private Object[] elements;
    private int size = 0;
    
    public Object pop() {
        if (size == 0) throw new EmptyStackException();
        return elements[--size]; // Reference not cleared!
    }
}

// ✅ Good: Clear obsolete references
public class Stack {
    private Object[] elements;
    private int size = 0;
    
    public Object pop() {
        if (size == 0) throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
}
```

## Try-With-Resources

### Prefer try-with-resources Over try-finally

```java
// ❌ Bad: try-finally with multiple resources
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[1024];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}

// ✅ Good: try-with-resources
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[1024];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

## Methods Common to All Objects

### Override equals Correctly

```java
@Override
public boolean equals(Object o) {
    // Reflexive: x.equals(x) == true
    // Symmetric: x.equals(y) == y.equals(x)
    // Transitive: x.equals(y) && y.equals(z) => x.equals(z)
    // Consistent: Multiple calls return same result
    // Non-null: x.equals(null) == false
    
    if (o == this) return true;
    if (!(o instanceof MyClass)) return false;
    MyClass that = (MyClass) o;
    return this.field == that.field;
}
```

### Always Override hashCode When Overriding equals

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}

// Or use Objects.hash()
@Override
public int hashCode() {
    return Objects.hash(areaCode, prefix, lineNum);
}
```

### Always Override toString

```java
@Override
public String toString() {
    return String.format("User[name=%s, email=%s]", name, email);
}
```

### Implement Comparable

```java
public final class PhoneNumber implements Comparable<PhoneNumber> {
    @Override
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0) {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0) {
                result = Short.compare(lineNum, pn.lineNum);
            }
        }
        return result;
    }
}
```
