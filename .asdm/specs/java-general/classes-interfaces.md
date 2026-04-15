# Classes and Interfaces

Best practices for class design, interfaces, composition, and inheritance.

## Minimize Accessibility

### Encapsulate Implementation Details

```java
// ✅ Good: Private fields with accessor methods
public class User {
    private String name;
    private String email;
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}

// ❌ Bad: Public mutable fields
public class User {
    public String name;
    public String email;
}
```

## Accessor Methods Over Public Fields

### Use Accessors for Public Classes

```java
// ❌ Bad: Public fields
public class Point {
    public double x;
    public double y;
}

// ✅ Good: Private fields with accessors
public class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
}
```

## Minimize Mutability

### Favor Immutability

```java
// ✅ Good: Immutable class
public final class Complex {
    private final double real;
    private final double imag;
    
    public Complex(double real, double imag) {
        this.real = real;
        this.imag = imag;
    }
    
    public double getReal() { return real; }
    public double getImag() { return imag; }
    
    public Complex add(Complex c) {
        return new Complex(real + c.real, imag + c.imag);
    }
    
    // No setters - all operations return new instances
}
```

### Rules for Immutable Classes

1. Don't provide methods that modify state
2. Ensure the class can't be extended (final class or private constructor)
3. Make all fields final
4. Make all fields private
5. Ensure exclusive access to mutable components

## Favor Composition Over Inheritance

### Use Composition Instead of Inheritance

```java
// ❌ Bad: Inheritance for code reuse
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    // Problem: addAll calls add internally, double-counting
}

// ✅ Good: Composition with forwarding
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    
    public InstrumentedSet(Set<E> s) {
        super(s);
    }
    
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
}
```

## Design for Inheritance or Prohibit It

### Document Self-Use Patterns

```java
// ✅ Good: Document how methods use each other
public abstract class AbstractList<E> {
    /**
     * Adds the specified element to the end of this list.
     * This implementation calls add(size(), e).
     * 
     * @implNote This method calls add(size(), e).
     */
    public boolean add(E e) {
        add(size(), e);
        return true;
    }
}
```

### Prevent Inheritance When Not Designed For It

```java
// ✅ Good: Final class or private constructor
public final class UtilityClass {
    private UtilityClass() {
        throw new AssertionError("Cannot instantiate");
    }
}
```

## Prefer Interfaces to Abstract Classes

### Define Behavior with Interfaces

```java
// ✅ Good: Interface defines behavior
public interface Singer {
    AudioClip sing(Song s);
}

public interface Songwriter {
    Song compose(int chartPosition);
}

// Multiple inheritance with interfaces
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}
```

## Design Interfaces for Posterity

### Consider Default Methods Carefully

```java
// ✅ Good: Default method with clear contract
public interface Collection<E> {
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        for (Iterator<E> it = iterator(); it.hasNext(); ) {
            if (filter.test(it.next())) {
                it.remove();
                removed = true;
            }
        }
        return removed;
    }
}
```

## Use Interfaces Only to Define Types

### Avoid Constant Interfaces

```java
// ❌ Bad: Constant interface
public interface PhysicalConstants {
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
}

// ✅ Good: Utility class for constants
public final class PhysicalConstants {
    private PhysicalConstants() {}
    
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
}
```

## Prefer Class Hierarchies to Tagged Classes

### Replace Tagged Classes with Hierarchy

```java
// ❌ Bad: Tagged class
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    final Shape shape;
    double length, width;  // For RECTANGLE
    double radius;          // For CIRCLE
    
    double area() {
        switch(shape) {
            case RECTANGLE: return length * width;
            case CIRCLE: return Math.PI * radius * radius;
        }
    }
}

// ✅ Good: Class hierarchy
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;
    
    Circle(double radius) { this.radius = radius; }
    
    @Override double area() { return Math.PI * radius * radius; }
}

class Rectangle extends Figure {
    final double length, width;
    
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override double area() { return length * width; }
}
```

## Favor Static Member Classes

### Choose the Right Nested Class Type

```java
public class OuterClass {
    // ✅ Static member class - can exist independently
    public static class StaticMemberClass {
        // No reference to enclosing instance
    }
    
    // Nonstatic member class (inner class) - has reference to enclosing instance
    public class InnerClass {
        // Can access OuterClass.this
    }
    
    // Anonymous class - single-use
    public void method() {
        Runnable r = new Runnable() {
            @Override public void run() { }
        };
    }
    
    // Local class - defined within method
    public void anotherMethod() {
        class LocalClass { }
    }
}
```

## Single Top-Level Class per File

### One Class Per Source File

```java
// ✅ Good: One top-level class per file
// file: User.java
public class User { }

// file: UserService.java
public class UserService { }

// ❌ Bad: Multiple top-level classes in one file
// file: Classes.java
public class User { }
class UserService { }  // Separate into own file
```
