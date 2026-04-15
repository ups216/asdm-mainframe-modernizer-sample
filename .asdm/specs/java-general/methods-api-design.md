# Methods and General Programming

Best practices for method design, parameters, return types, and general programming conventions.

## Check Parameters for Validity

### Validate All Parameters

```java
// ✅ Good: Explicit parameter validation
public void process(String input, int count) {
    Objects.requireNonNull(input, "input must not be null");
    if (count < 0) {
        throw new IllegalArgumentException("count must be non-negative: " + count);
    }
    // Method body
}

// ✅ Good: Use Java's built-in methods
public void process(List<String> items) {
    Objects.checkIndex(0, items.size()); // For index validation
    Objects.requireNonNull(items);
    List<String> copy = List.copyOf(items); // Throws on null elements
}
```

## Make Defensive Copies

### Copy Mutable Parameters

```java
// ✅ Good: Defensive copy on input
public final class Period {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime()); // Defensive copy
        this.end = new Date(end.getTime());
        
        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + " after " + end);
        }
    }
    
    public Date getStart() {
        return new Date(start.getTime()); // Defensive copy on output
    }
    
    public Date getEnd() {
        return new Date(end.getTime());
    }
}

// ✅ Good: Prefer immutable types
public final class Period {
    private final Instant start;
    private final Instant end;
    
    public Period(Instant start, Instant end) {
        this.start = Objects.requireNonNull(start);
        this.end = Objects.requireNonNull(end);
        // No copies needed - Instant is immutable
    }
}
```

## Design Method Signatures Carefully

### Follow Naming Conventions

```java
// ✅ Good: Clear, verb-based names
public List<User> findUsersByRole(Role role) { }
public void removeUser(User user) { }
public boolean hasPermission(Permission perm) { }

// ✅ Good: Consistent parameter order
public User create(String name, String email) { }
public void update(Long id, String name, String email) { }

// ✅ Good: Avoid long parameter lists (max 4)
public record Address(String street, String city, String zip) {}

public User create(String name, Address address) { }

// ❌ Bad: Too many parameters
public User create(String name, String street, String city, String zip, String email, String phone) { }
```

## Use Overloading Judiciously

### Avoid Confusing Overloads

```java
// ❌ Bad: Confusing overloads
public void print(Set<Integer> set) { }
public void print(List<Integer> list) { }

// Call with empty collection - which one?
Collection<Integer> c = new HashSet<>();
print(c); // Won't compile!

// ✅ Good: Different names instead of overloads
public void printSet(Set<Integer> set) { }
public void printList(List<Integer> list) { }

// ✅ Good: Or use specific types that don't overlap
public void process(Collection<Integer> c) { }  // Handles both
```

## Use Varargs Judiciously

### Varargs for Truly Variable Arguments

```java
// ✅ Good: Varargs for variable number of arguments
public String format(String pattern, Object... args) {
    return String.format(pattern, args);
}

// ✅ Good: Require at least one argument
public int min(int first, int... rest) {
    int result = first;
    for (int val : rest) {
        if (val < result) result = val;
    }
    return result;
}

// ❌ Bad: Varargs for array parameter
public void process(String... args) { } // If always called with array
public void process(String[] args) { }  // Better if always array

// ✅ Good: Combine with static factory
public static <T> List<T> of(T... elements) {
    return List.of(elements);
}
```

## Return Empty Collections, Not Null

### Never Return Null for Collections

```java
// ❌ Bad: Returning null
public List<User> getUsers() {
    List<User> users = repository.findAll();
    return users.isEmpty() ? null : users; // Client must check null!
}

// ✅ Good: Return empty collection
public List<User> getUsers() {
    List<User> users = repository.findAll();
    return users.isEmpty() ? List.of() : users;
}

// Or simpler:
public List<User> getUsers() {
    return repository.findAll(); // Returns empty list if none found
}
```

## Return Optionals Judiciously

### Use Optional for Absent Values

```java
// ✅ Good: Optional for absent result
public Optional<User> findById(Long id) {
    return repository.findById(id);
}

// ✅ Good: Never return null Optional
public Optional<String> getName() {
    return Optional.ofNullable(name); // Handles null correctly
}

// ❌ Bad: Optional for collection results
public Optional<List<User>> getUsers() { } // Return empty List instead

// ❌ Bad: Optional for performance-critical code
public Optional<Integer> getCount() { } // Use int with sentinel value

// ✅ Good: Optional as stream element
stream.filter(Optional::isPresent)
    .map(Optional::get)
    .collect(Collectors.toList());

// ✅ Good: Optional in combination
Optional<String> result = maybeUser
    .map(User::getEmail)
    .filter(StringUtils::isNotEmpty);
```

## Write Documentation

### Document All Public API Elements

```java
/**
 * Returns the element at the specified position in this list.
 *
 * @param index index of the element to return
 * @return the element at the specified position
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= size()})
 */
public E get(int index) {
    // Implementation
}

/**
 * Creates and returns a copy of this object.
 * The clone method performs a shallow copy of this instance.
 *
 * @return a clone of this instance
 * @implSpec This method creates a shallow copy.
 */
@Override
public Object clone() {
    // Implementation
}

// Use @param, @return, @throws for methods
// Use {@code} for code snippets
// Use @implSpec for implementation requirements
```

## General Programming Best Practices

### Minimize Scope of Local Variables

```java
// ❌ Bad: Declare at top
int i;
String s;
boolean found;
// ... many lines of code ...

// ✅ Good: Declare where first used
for (int i = 0; i < list.size(); i++) {
    String s = list.get(i);
    // Use immediately
}
```

### Prefer For-Each Loops

```java
// ❌ Bad: Traditional for loop when index not needed
for (Iterator<String> i = list.iterator(); i.hasNext(); ) {
    String s = i.next();
    // Process s
}

// ✅ Good: For-each loop
for (String s : list) {
    // Process s
}

// ❌ Bad: Index loop for arrays
for (int i = 0; i < array.length; i++) {
    doSomething(array[i]);
}

// ✅ Good: For-each loop
for (Element e : array) {
    doSomething(e);
}
```

### Know and Use Libraries

```java
// ❌ Bad: Reinventing the wheel
public static int random(int n) {
    return Math.abs(rnd.nextInt()) % n; // Broken for some n!
}

// ✅ Good: Use library methods
Random rnd = new Random();
int value = rnd.nextInt(n); // Correct for all n

// ✅ Good: Use standard libraries
// java.util.Collections, java.util.Arrays, java.util.stream
// Apache Commons, Guava for additional utilities
```

### Avoid Float and Double for Exact Values

```java
// ❌ Bad: Float/Double for money
double funds = 1.00;
for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price;
}
// Result: funds = 0.3999999999999999

// ✅ Good: Use BigDecimal or long (cents)
BigDecimal funds = new BigDecimal("1.00");
BigDecimal price = new BigDecimal("0.10");
while (funds.compareTo(price) >= 0) {
    funds = funds.subtract(price);
    price = price.add(new BigDecimal("0.10"));
}

// ✅ Good: Use int or long for cents
int fundsInCents = 100; // $1.00
int priceInCents = 10;  // $0.10
```

### Prefer Primitives to Boxed Primitives

```java
// ❌ Bad: Unnecessary boxing
Integer sum = 0;
for (Integer i : integers) {
    sum += i; // Creates many Integer objects!
}

// ✅ Good: Use primitive
int sum = 0;
for (int i : integers) {
    sum += i;
}

// ❌ Bad: Comparing boxed primitives with ==
Integer a = 1000;
Integer b = 1000;
if (a == b) { ... } // false! Different objects

// ✅ Good: Use equals
if (a.equals(b)) { ... } // true
```

### Beware String Concatenation

```java
// ❌ Bad: String concatenation in loop
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i; // Creates 1000 String objects!
}

// ✅ Good: Use StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();

// ✅ Good: Use String.join or collectors
String result = items.stream().collect(Collectors.joining(", "));
```

### Refer to Objects by Interfaces

```java
// ❌ Bad: Concrete type declaration
ArrayList<String> list = new ArrayList<>();
HashMap<String, Integer> map = new HashMap<>();

// ✅ Good: Interface type declaration
List<String> list = new ArrayList<>();
Map<String, Integer> map = new HashMap<>();
```
