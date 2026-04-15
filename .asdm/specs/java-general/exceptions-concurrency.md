# Exceptions and Concurrency

Best practices for exception handling and concurrent programming.

## Use Exceptions Only for Exceptional Conditions

### Don't Use Exceptions for Control Flow

```java
// ❌ Bad: Exception for normal control flow
try {
    int i = 0;
    while (true) {
        array[i++] = getValue();
    }
} catch (ArrayIndexOutOfBoundsException e) {
    // Expected behavior - wrong use of exception!
}

// ✅ Good: Use normal control flow
for (int i = 0; i < array.length; i++) {
    array[i] = getValue();
}
```

## Checked vs Runtime Exceptions

### Use Checked for Recoverable, Runtime for Programming Errors

```java
// ✅ Good: Checked exception for recoverable condition
public class InsufficientFundsException extends Exception {
    private final BigDecimal shortage;
    
    public InsufficientFundsException(BigDecimal shortage) {
        super("Insufficient funds: shortage of " + shortage);
        this.shortage = shortage;
    }
    
    public BigDecimal getShortage() {
        return shortage;
    }
}

// ✅ Good: Runtime exception for programming error
public class InvalidStateException extends RuntimeException {
    public InvalidStateException(String message) {
        super(message);
    }
}

// Usage
public void withdraw(BigDecimal amount) throws InsufficientFundsException {
    if (amount.compareTo(balance) > 0) {
        throw new InsufficientFundsException(amount.subtract(balance));
    }
    balance = balance.subtract(amount);
}

public void process(Object input) {
    if (input == null) {
        throw new InvalidStateException("Input cannot be null"); // Programming error
    }
}
```

## Avoid Unnecessary Checked Exceptions

### Use Checked Exceptions Sparingly

```java
// ❌ Bad: Checked exception for simple case
public interface ObjectInput {
    Object readObject() throws ObjectInputException;
}

// Every caller must handle:
try {
    Object obj = input.readObject();
} catch (ObjectInputException e) {
    // Handle or rethrow
}

// ✅ Good: Return Optional or result object
public interface ObjectInput {
    Optional<Object> readObject();
}

// Or use result pattern:
public interface Result<T> {
    boolean isSuccess();
    T getValue();
    String getError();
}
```

## Favor Standard Exceptions

### Reuse Built-in Exception Types

```java
// ✅ Good: Standard exceptions
// IllegalArgumentException - Invalid parameter value
// IllegalStateException - Invalid state for operation
// NullPointerException - Null argument where prohibited
// IndexOutOfBoundsException - Index out of bounds
// UnsupportedOperationException - Operation not supported
// ArithmeticException - Arithmetic error (division by zero)

public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("Age cannot be negative: " + age);
    }
    this.age = age;
}

public void connect() {
    if (connected) {
        throw new IllegalStateException("Already connected");
    }
    // Connect
}
```

## Throw Exceptions Appropriate to Abstraction

### Translate Lower-Level Exceptions

```java
// ✅ Good: Exception translation
public class DatabaseService {
    public User getUser(Long id) {
        try {
            return repository.findById(id);
        } catch (SQLException e) {
            throw new DataAccessException("Failed to get user: " + id, e);
        }
    }
}

// Exception chaining preserves cause
public class DataAccessException extends RuntimeException {
    public DataAccessException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

## Document All Exceptions

### Document Thrown Exceptions

```java
/**
 * Loads properties from the specified file.
 *
 * @param file the properties file to load
 * @throws FileNotFoundException if the file does not exist
 * @throws IOException if an I/O error occurs
 * @throws IllegalArgumentException if file is null
 */
public Properties loadProperties(File file) throws IOException {
    Objects.requireNonNull(file, "file must not be null");
    // Implementation
}
```

## Include Failure-Capture Information

### Detailed Exception Messages

```java
// ❌ Bad: Vague exception message
throw new IllegalArgumentException("Invalid input");

// ✅ Good: Include all relevant information
throw new IllegalArgumentException(
    String.format("Invalid input: expected positive number, got %d at index %d", 
                  value, index));

// ✅ Good: Custom exception with structured data
public class InsufficientFundsException extends Exception {
    private final BigDecimal requested;
    private final BigDecimal available;
    
    public InsufficientFundsException(BigDecimal requested, BigDecimal available) {
        super(String.format("Requested %s but only %s available", requested, available));
        this.requested = requested;
        this.available = available;
    }
}
```

## Strive for Failure Atomicity

### Ensure Consistent State After Failure

```java
// ✅ Good: Check before modifying
public void addAll(Collection<E> c) {
    if (c == null) {
        throw new NullPointerException();
    }
    // All checks pass, now modify
    ensureCapacity(size + c.size());
    for (E e : c) {
        elements[size++] = e;
    }
}

// ✅ Good: Order modifications
public void transfer(Account from, Account to, BigDecimal amount) {
    from.withdraw(amount);  // If this fails, to is unchanged
    to.deposit(amount);     // If this fails, need rollback
}

// ✅ Good: Copy-on-write for recovery
public void update(List<Item> items) {
    List<Item> copy = new ArrayList<>(items);
    modify(copy);  // Work on copy
    this.items = copy;  // Atomic update
}
```

## Don't Ignore Exceptions

### Handle or Propagate Exceptions

```java
// ❌ Bad: Empty catch block
try {
    doSomething();
} catch (Exception e) {
    // Ignored - BAD!
}

// ✅ Good: Log the exception
try {
    doSomething();
} catch (Exception e) {
    logger.error("Failed to do something", e);
}

// ✅ Good: Convert to higher-level exception
try {
    doSomething();
} catch (Exception e) {
    throw new BusinessException("Operation failed", e);
}

// ✅ Good: Propagate if appropriate
public void process() throws IOException {
    Files.readAllLines(path); // Let caller handle
}
```

## Concurrency Best Practices

### Synchronize Access to Shared Mutable Data

```java
// ❌ Bad: Unsynchronized access
public class Counter {
    private int count;
    
    public void increment() {
        count++; // Not thread-safe!
    }
}

// ✅ Good: Synchronized access
public class Counter {
    private int count;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// ✅ Good: Use atomic classes
public class Counter {
    private final AtomicInteger count = new AtomicInteger();
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public int getCount() {
        return count.get();
    }
}

// ✅ Good: Use locks for complex operations
private final Lock lock = new ReentrantLock();

public void complexOperation() {
    lock.lock();
    try {
        // Critical section
    } finally {
        lock.unlock();
    }
}
```

### Avoid Excessive Synchronization

```java
// ❌ Bad: Calling foreign method while synchronized
public class ObservableSet<E> {
    private final Set<E> set;
    private final List<Consumer<E>> observers = new ArrayList<>();
    
    public synchronized void addObserver(Consumer<E> observer) {
        observers.add(observer);
    }
    
    public synchronized void notifyElementAdded(E element) {
        for (Consumer<E> observer : observers) {
            observer.accept(element); // Unknown code may cause deadlock!
        }
    }
}

// ✅ Good: Copy before calling foreign methods
public void notifyElementAdded(E element) {
    List<Consumer<E>> snapshot;
    synchronized (this) {
        snapshot = new ArrayList<>(observers);
    }
    for (Consumer<E> observer : snapshot) {
        observer.accept(element); // Safe, no lock held
    }
}
```

### Prefer Executors and Tasks Over Threads

```java
// ❌ Bad: Creating threads directly
Thread thread = new Thread(() -> doWork());
thread.start();

// ✅ Good: Use ExecutorService
ExecutorService executor = Executors.newFixedThreadPool(10);
executor.submit(() -> doWork());

// ✅ Good: Use CompletableFuture for async operations
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return fetchData();
});

CompletableFuture<String> result = future
    .thenApply(this::processData)
    .thenCompose(this::saveData)
    .exceptionally(ex -> "Error: " + ex.getMessage());
```

### Prefer Concurrency Utilities Over wait/notify

```java
// ❌ Bad: Low-level wait/notify
public class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    
    public synchronized void put(T item) throws InterruptedException {
        while (queue.size() == MAX) {
            wait();
        }
        queue.add(item);
        notifyAll();
    }
}

// ✅ Good: Use BlockingQueue
public class BoundedBuffer<T> {
    private final BlockingQueue<T> queue = new ArrayBlockingQueue<>(MAX);
    
    public void put(T item) throws InterruptedException {
        queue.put(item);
    }
    
    public T take() throws InterruptedException {
        return queue.take();
    }
}

// ✅ Good: Use CountDownLatch, Semaphore, CyclicBarrier
CountDownLatch latch = new CountDownLatch(3);
executor.submit(() -> {
    doWork();
    latch.countDown();
});
latch.await(); // Wait for all tasks
```

### Document Thread Safety

```java
/**
 * A thread-safe counter that supports atomic increment operations.
 * 
 * <p>This class is thread-safe. All operations are atomic and can be
 * safely called from multiple threads concurrently.
 * 
 * <p>The implementation uses {@link AtomicInteger} for lock-free
 * thread safety.
 */
public class ThreadSafeCounter {
    private final AtomicInteger count = new AtomicInteger();
    
    /**
     * Atomically increments the counter.
     * This method is thread-safe.
     */
    public void increment() {
        count.incrementAndGet();
    }
}
```

### Don't Depend on Thread Scheduler

```java
// ❌ Bad: Depending on thread scheduling
Thread.sleep(100); // Hoping another thread finishes

// ✅ Good: Use proper synchronization
latch.await();
future.get(10, TimeUnit.SECONDS);
```
