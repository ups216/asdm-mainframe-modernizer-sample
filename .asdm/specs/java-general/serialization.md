# Serialization

Best practices for Java serialization and alternatives.

## Prefer Alternatives to Java Serialization

### Use Modern Serialization Formats

```java
// ❌ Bad: Java serialization (security risks, versioning issues)
public class User implements Serializable {
    private String name;
    private String email;
}

// ✅ Good: Use JSON, Protocol Buffers, or other formats
public class User {
    private String name;
    private String email;
    
    // JSON serialization
    public String toJson() {
        return String.format("{\"name\":\"%s\",\"email\":\"%s\"}", name, email);
    }
    
    public static User fromJson(String json) {
        // Parse JSON using Jackson, Gson, etc.
    }
}

// ✅ Good: Use libraries like Jackson, Gson, Protocol Buffers
ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(user);
User user = mapper.readValue(json, User.class);
```

## Implement Serializable with Caution

### Understand the Costs

```java
// ✅ Good: Only implement Serializable when necessary
public class User {
    // Regular class - no serialization overhead
}

// If serialization is required:
public class User implements Serializable {
    private static final long serialVersionUID = 1L; // Always include
    
    private String name;
    private String email;
    
    // transient for non-serializable fields
    private transient Connection connection;
}
```

### Inheritance Concerns

```java
// Classes designed for inheritance should rarely implement Serializable

// ✅ Good: Allow subclasses to choose
public abstract class AbstractEntity {
    protected long id;
    // No Serializable
}

// Subclass can choose
public class SerializableEntity extends AbstractEntity implements Serializable {
    private static final long serialVersionUID = 1L;
}

// ✅ Good: If superclass is Serializable, ensure proper design
public abstract class AbstractList<E> implements Serializable {
    // Must have parameterless constructor for deserialization
    // of non-serializable subclasses
}
```

## Consider Custom Serialized Form

### Design for Serialization When Needed

```java
// ✅ Good: Custom serialized form
public class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;
    
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }
    
    // Custom serialization - only store the strings, not the structure
    private void writeObject(ObjectOutputStream s) throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }
    
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();
        
        // Reconstruct list from strings
        for (int i = 0; i < numElements; i++) {
            add((String) s.readObject());
        }
    }
}
```

### Serializing Singletons

```java
// ✅ Good: Enum for serializable singleton
public enum Elvis {
    INSTANCE;
    
    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };
    
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}

// ✅ Good: readResolve for non-enum singletons
public class Elvis implements Serializable {
    private static final Elvis INSTANCE = new Elvis();
    private static final long serialVersionUID = 1L;
    
    private Elvis() {}
    
    public static Elvis getInstance() { return INSTANCE; }
    
    // Preserve singleton property during deserialization
    private Object readResolve() {
        return INSTANCE;
    }
}
```

## Write readObject Methods Defensively

### Validate During Deserialization

```java
public class Period implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private Date start;
    private Date end;
    
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + " after " + end);
        }
    }
    
    // Defensive readObject for deserialization safety
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        
        // Defensively copy mutable fields
        start = new Date(start.getTime());
        end = new Date(end.getTime());
        
        // Validate invariants
        if (start.compareTo(end) > 0) {
            throw new InvalidObjectException(start + " after " + end);
        }
    }
}
```

## Serialization Proxy Pattern

### Use Proxy for Complex Serialization

```java
// ✅ Good: Serialization proxy pattern
public class Period implements Serializable {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        // Validation
    }
    
    // Proxy class
    private static class SerializationProxy implements Serializable {
        private static final long serialVersionUID = 1L;
        
        private final Date start;
        private final Date end;
        
        SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }
        
        // Reconstruct Period from proxy
        private Object readResolve() {
            return new Period(start, end);
        }
    }
    
    // Return proxy instead of this
    private Object writeReplace() {
        return new SerializationProxy(this);
    }
    
    // Prevent direct deserialization
    private void readObject(ObjectInputStream s) throws InvalidObjectException {
        throw new InvalidObjectException("Proxy required");
    }
}
```

## Best Practices Summary

### When to Use Serialization

| Use Case | Recommendation |
|----------|---------------|
| Network communication | Use JSON, Protocol Buffers |
| Persistent storage | Use database or JSON |
| Caching | Consider JSON or specialized formats |
| RMI (legacy) | Java serialization required |
| Session state in servlets | May need serialization |

### Serialization Checklist

- [ ] Include `serialVersionUID` constant
- [ ] Make mutable fields `transient` if they shouldn't be serialized
- [ ] Use defensive copying in `readObject` for mutable fields
- [ ] Validate invariants in `readObject`
- [ ] Consider serialization proxy for complex objects
- [ ] Document serialization behavior
- [ ] Test serialization round-trip

### Avoid Common Pitfalls

```java
// ❌ Bad: Changing serialVersionUID breaks compatibility
private static final long serialVersionUID = 1L;
// Later: private static final long serialVersionUID = 2L; // Breaks!

// ❌ Bad: Non-transient sensitive data
public class User implements Serializable {
    private String password; // Gets serialized!
}

// ✅ Good: Mark sensitive data as transient
public class User implements Serializable {
    private transient String password; // Not serialized
}

// ❌ Bad: Serializing internal implementation
public class HashMap<K,V> implements Serializable {
    private Entry<K,V>[] table; // Internal structure exposed
}

// ✅ Good: Custom serialization that captures logical state
private void writeObject(ObjectOutputStream s) throws IOException {
    s.writeInt(size);
    for (Entry<K,V> e : entries()) {
        s.writeObject(e.key);
        s.writeObject(e.value);
    }
}
```
