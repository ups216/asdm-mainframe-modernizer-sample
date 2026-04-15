# Services

Service layer design and transaction management.

## Service Interface

Service classes must be of type interface.

```java
public interface UserService {
    UserDTO findById(Long id);
    UserDTO create(UserCreateDTO dto);
    UserDTO update(Long id, UserUpdateDTO dto);
    void delete(Long id);
    List<UserDTO> findAll();
}
```

## Service Implementation

### Required Annotations

1. All service class method implementations must be in ServiceImpl classes that implement the service class
2. All ServiceImpl classes must be annotated with `@Service`
3. All dependencies in ServiceImpl classes must be `@Autowired` without a constructor

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Override
    public UserDTO findById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found with id: " + id));
        return new UserDTO(user);
    }
}
```

## Return Types

Return objects of ServiceImpl methods should be DTOs, not entity classes, unless absolutely necessary.

```java
@Service
public class UserServiceImpl implements UserService {
    
    // ✅ Correct: Return DTO
    @Override
    public UserDTO findById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found"));
        return new UserDTO(user);
    }
    
    // ❌ Avoid: Return Entity
    @Override
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found"));
    }
}
```

## Entity Existence Check

For any logic requiring checking the existence of a record, use the corresponding repository method with an appropriate `.orElseThrow` lambda method.

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Override
    public UserDTO findById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found with id: " + id));
        return new UserDTO(user);
    }
    
    @Override
    public UserDTO findByEmail(String email) {
        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new IllegalArgumentException("User not found with email: " + email));
        return new UserDTO(user);
    }
    
    @Override
    public UserDTO update(Long id, UserUpdateDTO dto) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found with id: " + id));
        
        user.setName(dto.name());
        user.setEmail(dto.email());
        
        User savedUser = userRepository.save(user);
        return new UserDTO(savedUser);
    }
}
```

## Transaction Management

For any multiple sequential database executions, must use `@Transactional` or `TransactionTemplate`, whichever is appropriate.

### Using @Transactional Annotation

```java
@Service
public class OrderServiceImpl implements OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private OrderItemRepository orderItemRepository;
    
    @Autowired
    private ProductRepository productRepository;
    
    @Transactional
    @Override
    public OrderDTO createOrder(OrderCreateDTO dto) {
        // Create order
        Order order = new Order();
        order.setUserId(dto.userId());
        order.setStatus("PENDING");
        order = orderRepository.save(order);
        
        // Create order items
        for (OrderItemCreateDTO itemDto : dto.items()) {
            Product product = productRepository.findById(itemDto.productId())
                .orElseThrow(() -> new IllegalArgumentException("Product not found"));
            
            OrderItem item = new OrderItem();
            item.setOrder(order);
            item.setProduct(product);
            item.setQuantity(itemDto.quantity());
            item.setPrice(product.getPrice());
            orderItemRepository.save(item);
        }
        
        return new OrderDTO(order);
    }
}
```

### Using TransactionTemplate

```java
@Service
public class PaymentServiceImpl implements PaymentService {
    
    @Autowired
    private TransactionTemplate transactionTemplate;
    
    @Autowired
    private PaymentRepository paymentRepository;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Override
    public PaymentDTO processPayment(Long orderId, PaymentDTO dto) {
        return transactionTemplate.execute(status -> {
            try {
                // Update order status
                Order order = orderRepository.findById(orderId)
                    .orElseThrow(() -> new IllegalArgumentException("Order not found"));
                order.setStatus("PAID");
                orderRepository.save(order);
                
                // Create payment record
                Payment payment = new Payment();
                payment.setOrderId(orderId);
                payment.setAmount(dto.amount());
                payment.setPaymentMethod(dto.paymentMethod());
                payment = paymentRepository.save(payment);
                
                return new PaymentDTO(payment);
            } catch (Exception e) {
                status.setRollbackOnly();
                throw e;
            }
        });
    }
}
```

### Transaction Propagation

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Transactional(readOnly = true)
    @Override
    public List<UserDTO> findAll() {
        return userRepository.findAll().stream()
            .map(UserDTO::new)
            .collect(Collectors.toList());
    }
    
    @Transactional
    @Override
    public UserDTO create(UserCreateDTO dto) {
        User user = new User();
        user.setName(dto.name());
        user.setEmail(dto.email());
        user = userRepository.save(user);
        return new UserDTO(user);
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    @Override
    public void logUserActivity(Long userId, String activity) {
        // Always creates new transaction for logging
        // Even if called from another transactional method
    }
}
```

## Business Logic Patterns

### Create Operation

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Transactional
    @Override
    public UserDTO create(UserCreateDTO dto) {
        // Check for duplicates
        if (userRepository.existsByEmail(dto.email())) {
            throw new IllegalArgumentException("Email already exists: " + dto.email());
        }
        
        // Create entity
        User user = new User();
        user.setName(dto.name());
        user.setEmail(dto.email());
        user.setActive(true);
        
        // Save and return DTO
        user = userRepository.save(user);
        return new UserDTO(user);
    }
}
```

### Update Operation

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Transactional
    @Override
    public UserDTO update(Long id, UserUpdateDTO dto) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found"));
        
        // Update fields
        if (dto.name() != null) {
            user.setName(dto.name());
        }
        if (dto.email() != null && !dto.email().equals(user.getEmail())) {
            if (userRepository.existsByEmail(dto.email())) {
                throw new IllegalArgumentException("Email already exists");
            }
            user.setEmail(dto.email());
        }
        
        user = userRepository.save(user);
        return new UserDTO(user);
    }
}
```

### Delete Operation

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Transactional
    @Override
    public void delete(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found"));
        
        // Soft delete
        user.setActive(false);
        userRepository.save(user);
        
        // Or hard delete
        // userRepository.delete(user);
    }
}
```

## Best Practices

### Avoid Business Logic in Controllers

```java
// ❌ Bad: Business logic in controller
@RestController
public class UserController {
    @PostMapping
    public ResponseEntity<ApiResponse<UserDTO>> create(@RequestBody UserCreateDTO dto) {
        if (userRepository.existsByEmail(dto.email())) {
            throw new IllegalArgumentException("Email exists");
        }
        // More logic...
    }
}

// ✅ Good: Business logic in service
@Service
public class UserServiceImpl implements UserService {
    @Transactional
    public UserDTO create(UserCreateDTO dto) {
        if (userRepository.existsByEmail(dto.email())) {
            throw new IllegalArgumentException("Email exists");
        }
        // Logic...
    }
}
```

### Use Meaningful Exception Messages

```java
@Override
public UserDTO findById(Long id) {
    return userRepository.findById(id)
        .map(UserDTO::new)
        .orElseThrow(() -> new IllegalArgumentException(
            String.format("User not found with id: %d", id)
        ));
}
```

### Validate Input in Service

```java
@Override
@Transactional
public UserDTO create(UserCreateDTO dto) {
    // Validate business rules
    if (dto.name() == null || dto.name().isBlank()) {
        throw new IllegalArgumentException("Name is required");
    }
    if (dto.email() == null || !dto.email().contains("@")) {
        throw new IllegalArgumentException("Valid email is required");
    }
    
    // Check uniqueness
    if (userRepository.existsByEmail(dto.email())) {
        throw new IllegalArgumentException("Email already registered");
    }
    
    // Create...
}
```
