# DTOs (Data Transfer Objects)

Data Transfer Object patterns and validation.

## DTO Structure

### Using Records

Must be of type record, unless specified in a prompt otherwise.

```java
// Simple DTO using record
public record UserDTO(
    Long id,
    String name,
    String email
) {
    public UserDTO(User user) {
        this(user.getId(), user.getName(), user.getEmail());
    }
}
```

### Compact Canonical Constructor

Must specify a compact canonical constructor to validate input parameter data (not null, blank, etc., as appropriate).

```java
public record UserCreateDTO(
    String name,
    String email,
    String password
) {
    public UserCreateDTO {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be null or blank");
        }
        if (email == null || email.isBlank()) {
            throw new IllegalArgumentException("Email cannot be null or blank");
        }
        if (password == null || password.length() < 8) {
            throw new IllegalArgumentException("Password must be at least 8 characters");
        }
    }
}
```

## DTO Types

### Response DTO

```java
public record UserDTO(
    Long id,
    String name,
    String email,
    boolean active,
    LocalDateTime createdAt
) {
    public UserDTO(User user) {
        this(
            user.getId(),
            user.getName(),
            user.getEmail(),
            user.isActive(),
            user.getCreatedAt()
        );
    }
}
```

### Create DTO

```java
public record UserCreateDTO(
    String name,
    String email,
    String password
) {
    public UserCreateDTO {
        Objects.requireNonNull(name, "Name is required");
        Objects.requireNonNull(email, "Email is required");
        Objects.requireNonNull(password, "Password is required");
        
        if (!email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
            throw new IllegalArgumentException("Invalid email format");
        }
        if (password.length() < 8) {
            throw new IllegalArgumentException("Password must be at least 8 characters");
        }
    }
    
    public User toEntity() {
        User user = new User();
        user.setName(name);
        user.setEmail(email);
        user.setPassword(password);
        return user;
    }
}
```

### Update DTO

```java
public record UserUpdateDTO(
    String name,
    String email
) {
    public UserUpdateDTO {
        // Allow partial updates - validation only if provided
        if (email != null && !email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
            throw new IllegalArgumentException("Invalid email format");
        }
    }
}
```

### List/Summary DTO

```java
public record UserSummaryDTO(
    Long id,
    String name,
    String email
) {
    public static UserSummaryDTO from(User user) {
        return new UserSummaryDTO(
            user.getId(),
            user.getName(),
            user.getEmail()
        );
    }
}
```

### Nested DTO

```java
public record OrderDTO(
    Long id,
    String status,
    BigDecimal total,
    UserSummaryDTO user,
    List<OrderItemDTO> items,
    LocalDateTime orderDate
) {
    public OrderDTO(Order order) {
        this(
            order.getId(),
            order.getStatus(),
            calculateTotal(order.getItems()),
            new UserSummaryDTO(order.getUser()),
            order.getItems().stream()
                .map(OrderItemDTO::new)
                .collect(Collectors.toList()),
            order.getOrderDate()
        );
    }
    
    private static BigDecimal calculateTotal(List<OrderItem> items) {
        return items.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

public record OrderItemDTO(
    Long id,
    String productName,
    int quantity,
    BigDecimal price
) {
    public OrderItemDTO(OrderItem item) {
        this(
            item.getId(),
            item.getProduct().getName(),
            item.getQuantity(),
            item.getPrice()
        );
    }
}
```

## Validation Patterns

### Null Checks

```java
public record ProductDTO(
    Long id,
    String name,
    String description,
    BigDecimal price
) {
    public ProductDTO {
        Objects.requireNonNull(name, "Name cannot be null");
        Objects.requireNonNull(price, "Price cannot be null");
    }
}
```

### Range Validation

```java
public record ProductCreateDTO(
    String name,
    BigDecimal price,
    int stock
) {
    public ProductCreateDTO {
        Objects.requireNonNull(name, "Name is required");
        Objects.requireNonNull(price, "Price is required");
        
        if (price.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Price must be greater than zero");
        }
        if (stock < 0) {
            throw new IllegalArgumentException("Stock cannot be negative");
        }
    }
}
```

### Enum Validation

```java
public enum OrderStatus {
    PENDING, PROCESSING, SHIPPED, DELIVERED, CANCELLED
}

public record OrderUpdateDTO(
    String status
) {
    public OrderUpdateDTO {
        if (status != null) {
            try {
                OrderStatus.valueOf(status);
            } catch (IllegalArgumentException e) {
                throw new IllegalArgumentException(
                    "Invalid status. Allowed values: " + Arrays.toString(OrderStatus.values())
                );
            }
        }
    }
}
```

### Collection Validation

```java
public record OrderCreateDTO(
    Long userId,
    List<OrderItemCreateDTO> items
) {
    public OrderCreateDTO {
        Objects.requireNonNull(userId, "User ID is required");
        Objects.requireNonNull(items, "Items are required");
        
        if (items.isEmpty()) {
            throw new IllegalArgumentException("Order must have at least one item");
        }
    }
}
```

## DTO Conversion Patterns

### Factory Method in DTO

```java
public record UserDTO(
    Long id,
    String name,
    String email
) {
    public static UserDTO from(User user) {
        return new UserDTO(
            user.getId(),
            user.getName(),
            user.getEmail()
        );
    }
}
```

### Constructor from Entity

```java
public record UserDTO(
    Long id,
    String name,
    String email
) {
    public UserDTO(User user) {
        this(user.getId(), user.getName(), user.getEmail());
    }
}
```

### Mapper Utility Class

```java
public final class UserMapper {
    
    private UserMapper() {}
    
    public static UserDTO toDTO(User user) {
        return new UserDTO(
            user.getId(),
            user.getName(),
            user.getEmail()
        );
    }
    
    public static User toEntity(UserCreateDTO dto) {
        User user = new User();
        user.setName(dto.name());
        user.setEmail(dto.email());
        return user;
    }
    
    public static List<UserDTO> toDTOList(List<User> users) {
        return users.stream()
            .map(UserMapper::toDTO)
            .collect(Collectors.toList());
    }
}
```

## Best Practices

### Keep DTOs Immutable

```java
// ✅ Good: Immutable record
public record UserDTO(Long id, String name, String email) {}

// ❌ Avoid: Mutable class
public class UserDTO {
    private Long id;
    private String name;
    // getters/setters...
}
```

### Separate DTOs by Purpose

```java
// Create operation
public record UserCreateDTO(String name, String email, String password) {}

// Update operation
public record UserUpdateDTO(String name, String email) {}

// Response
public record UserDTO(Long id, String name, String email, LocalDateTime createdAt) {}
```

### Avoid Exposing Sensitive Data

```java
// ❌ Bad: Exposes password
public record UserDTO(Long id, String name, String email, String password) {}

// ✅ Good: Password excluded
public record UserDTO(Long id, String name, String email) {}
```

### Use Meaningful Names

```java
// ✅ Good: Clear purpose
public record UserRegistrationDTO(...) {}
public record UserProfileDTO(...) {}
public record UserLoginDTO(...) {}

// ❌ Avoid: Generic names
public record UserInputDTO(...) {}
public record UserOutputDTO(...) {}
```
