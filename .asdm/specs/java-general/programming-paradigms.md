# Programming Paradigms

Guidelines for functional programming, data-oriented programming, and concurrency patterns.

## Functional Programming Guidelines

### Use Immutable Objects

```java
// ✅ Good: Immutable class
public final class Money {
    private final BigDecimal amount;
    private final Currency currency;
    
    public Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }
    
    public Money add(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("Different currencies");
        }
        return new Money(amount.add(other.amount), currency);
    }
    
    // No setters, all fields final
}
```

### Don't Mutate State

```java
// ❌ Bad: Mutating state
public List<String> processNames(List<String> names) {
    for (int i = 0; i < names.size(); i++) {
        names.set(i, names.get(i).toUpperCase());
    }
    return names;
}

// ✅ Good: Return new collection
public List<String> processNames(List<String> names) {
    return names.stream()
        .map(String::toUpperCase)
        .collect(Collectors.toList());
}

// ✅ Good: Use immutable collections
public List<String> processNames(List<String> names) {
    return names.stream()
        .map(String::toUpperCase)
        .toList(); // Java 16+ unmodifiable list
}
```

### Pure Functions

```java
// ❌ Bad: Side effects
public class Calculator {
    private int result;
    
    public void add(int value) {
        result += value; // Mutates state
    }
}

// ✅ Good: Pure function - no side effects
public class Calculator {
    public int add(int a, int b) {
        return a + b; // No state mutation
    }
}

// ✅ Good: Functional style
public static int sum(List<Integer> numbers) {
    return numbers.stream()
        .reduce(0, Integer::sum); // Pure transformation
}
```

### Higher-Order Functions

```java
// ✅ Good: Functions as parameters
public <T> List<T> filter(List<T> list, Predicate<T> predicate) {
    return list.stream()
        .filter(predicate)
        .collect(Collectors.toList());
}

// ✅ Good: Function composition
public Function<String, String> pipeline = 
    String::trim
    .andThen(String::toLowerCase)
    .andThen(s -> s.replaceAll("\\s+", "-"));

String result = pipeline.apply("  Hello World  "); // "hello-world"
```

## Data-Oriented Programming Pillars

### 1. Separate Code from Data

```java
// ✅ Good: Data is separate from behavior
public record Customer(
    String id,
    String name,
    String email,
    Address address
) {}

// Behavior in separate service
public class CustomerService {
    public Customer validate(Customer customer) {
        // Validation logic
    }
    
    public Customer normalizeEmail(Customer customer) {
        return new Customer(
            customer.id(),
            customer.name(),
            customer.email().toLowerCase(),
            customer.address()
        );
    }
}
```

### 2. Represent Data with Generic Data Structures

```java
// ✅ Good: Generic data structures
public record Entity(
    Map<String, Object> attributes
) {
    @SuppressWarnings("unchecked")
    public <T> T get(String key, Class<T> type) {
        return (T) attributes.get(key);
    }
    
    public Entity with(String key, Object value) {
        Map<String, Object> newAttrs = new HashMap<>(attributes);
        newAttrs.put(key, value);
        return new Entity(newAttrs);
    }
}
```

### 3. Data Should Be Immutable

```java
// ✅ Good: Immutable data with records
public record Order(
    String id,
    List<OrderLine> lines,
    OrderStatus status
) {
    // Create new instance for updates
    public Order withStatus(OrderStatus newStatus) {
        return new Order(id, lines, newStatus);
    }
    
    public Order addLine(OrderLine line) {
        List<OrderLine> newLines = new ArrayList<>(lines);
        newLines.add(line);
        return new Order(id, Collections.unmodifiableList(newLines), status);
    }
}
```

### 4. Use Pure Functions to Manipulate Data

```java
// ✅ Good: Pure transformation functions
public class OrderTransformers {
    
    public static Order applyDiscount(Order order, BigDecimal percentage) {
        List<OrderLine> discountedLines = order.lines().stream()
            .map(line -> line.withPrice(
                line.price().multiply(BigDecimal.ONE.subtract(percentage))))
            .toList();
        return new Order(order.id(), discountedLines, order.status());
    }
    
    public static BigDecimal calculateTotal(Order order) {
        return order.lines().stream()
            .map(OrderLine::subtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

### 5. Keep Data Flat and Denormalized

```java
// ❌ Bad: Deeply nested structure
public record Order(
    Customer customer,
    Map<String, Map<String, Object>> metadata
) {}

// ✅ Good: Flat structure
public record Order(
    String customerId,
    String customerName,
    String customerEmail,
    List<OrderLine> lines,
    BigDecimal total
) {}
```

### 6. Keep Data Generic Until Specific

```java
// ✅ Good: Start generic, specialize when needed
public record Event(
    String type,
    Instant timestamp,
    Map<String, Object> payload
) {
    public static Event create(String type, Map<String, Object> payload) {
        return new Event(type, Instant.now(), payload);
    }
    
    // Convert to specific type when needed
    public UserCreatedEvent asUserCreatedEvent() {
        return new UserCreatedEvent(
            (String) payload.get("userId"),
            (String) payload.get("email")
        );
    }
}
```

### 7. Data Integrity Through Validation Functions

```java
// ✅ Good: Validation as pure functions
public class CustomerValidator {
    
    public static ValidationResult validate(Customer customer) {
        return ValidationResult.combine(
            validateEmail(customer.email()),
            validateName(customer.name()),
            validateAddress(customer.address())
        );
    }
    
    private static ValidationResult validateEmail(String email) {
        if (email == null || email.isBlank()) {
            return ValidationResult.error("Email is required");
        }
        if (!email.contains("@")) {
            return ValidationResult.error("Invalid email format");
        }
        return ValidationResult.success();
    }
}

public record ValidationResult(boolean valid, List<String> errors) {
    public static ValidationResult success() {
        return new ValidationResult(true, List.of());
    }
    
    public static ValidationResult error(String message) {
        return new ValidationResult(false, List.of(message));
    }
    
    public static ValidationResult combine(ValidationResult... results) {
        List<String> allErrors = Arrays.stream(results)
            .flatMap(r -> r.errors().stream())
            .toList();
        return new ValidationResult(allErrors.isEmpty(), allErrors);
    }
}
```

### 8. Flexible and Generic Data Access

```java
// ✅ Good: Generic data access pattern
public interface DataStore<T> {
    Optional<T> findById(String id);
    List<T> findAll();
    T save(T entity);
    void delete(String id);
}

// Works with any data type
public class InMemoryDataStore<T> implements DataStore<T> {
    private final Map<String, T> data = new ConcurrentHashMap<>();
    
    @Override
    public Optional<T> findById(String id) {
        return Optional.ofNullable(data.get(id));
    }
    
    @Override
    public T save(T entity) {
        String id = generateId();
        data.put(id, entity);
        return entity;
    }
}
```

### 9. Explicit and Traceable Data Transformation

```java
// ✅ Good: Explicit transformation pipeline
public class DataPipeline {
    
    public record TransformStep<T, R>(
        String name,
        Function<T, R> transform
    ) {
        public R apply(T input) {
            System.out.println("Executing step: " + name);
            return transform.apply(input);
        }
    }
    
    public static <T> T process(T input, List<TransformStep<T, T>> steps) {
        T result = input;
        for (TransformStep<T, T> step : steps) {
            result = step.apply(result);
        }
        return result;
    }
}

// Usage
List<TransformStep<Customer, Customer>> steps = List.of(
    new TransformStep<>("validate", CustomerValidator::validateAndFix),
    new TransformStep<>("normalize", CustomerNormalizer::normalize),
    new TransformStep<>("enrich", CustomerEnricher::enrich)
);

Customer processed = DataPipeline.process(customer, steps);
```

### 10. Unidirectional Data Flow

```java
// ✅ Good: Event-driven unidirectional flow
public class EventStore {
    private final List<Event> events = new ArrayList<>();
    private final List<Consumer<Event>> subscribers = new ArrayList<>();
    
    public void publish(Event event) {
        events.add(event);
        subscribers.forEach(sub -> sub.accept(event));
    }
    
    public void subscribe(Consumer<Event> subscriber) {
        subscribers.add(subscriber);
    }
}

// State derived from events (unidirectional)
public class CustomerState {
    private final Map<String, Customer> customers = new HashMap<>();
    
    public void apply(Event event) {
        switch (event.type()) {
            case "CustomerCreated" -> {
                Customer customer = event.asCustomerCreated();
                customers.put(customer.id(), customer);
            }
            case "CustomerUpdated" -> {
                Customer update = event.asCustomerUpdated();
                Customer existing = customers.get(update.id());
                customers.put(update.id(), merge(existing, update));
            }
        }
    }
}
```

## Concurrency Guidelines

### Try Not to Maintain State in Classes

```java
// ❌ Bad: Mutable state
public class Counter {
    private int count = 0;
    
    public void increment() {
        count++;
    }
}

// ✅ Good: Stateless with atomic operations
public class Counter {
    private final AtomicInteger count = new AtomicInteger();
    
    public int increment() {
        return count.incrementAndGet();
    }
}

// ✅ Good: Immutable with functional updates
public record Counter(int value) {
    public Counter increment() {
        return new Counter(value + 1);
    }
}

// ✅ Good: Thread-local state when needed
public class RequestHandler {
    private static final ThreadLocal<RequestContext> context = 
        ThreadLocal.withInitial(RequestContext::new);
    
    public void handle(Request request) {
        context.get().setRequest(request);
        // Process
        context.remove();
    }
}
```

### Thread-Safe Data Sharing

```java
// ✅ Good: Immutable message passing
public record Message(
    String id,
    String type,
    Map<String, Object> payload
) {
    // Safe to share between threads - immutable
}

// ✅ Good: Concurrent data structures
public class EventProcessor {
    private final ConcurrentMap<String, EventHandler> handlers = 
        new ConcurrentHashMap<>();
    private final BlockingQueue<Event> queue = 
        new LinkedBlockingQueue<>();
    
    public void start() {
        ExecutorService executor = Executors.newFixedThreadPool(4);
        for (int i = 0; i < 4; i++) {
            executor.submit(this::processEvents);
        }
    }
    
    private void processEvents() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Event event = queue.take();
                EventHandler handler = handlers.get(event.type());
                if (handler != null) {
                    handler.handle(event);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```
