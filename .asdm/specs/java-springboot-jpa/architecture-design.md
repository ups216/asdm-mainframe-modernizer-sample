# Architecture Design

Application logic design and layer responsibilities for Spring Boot applications.

## AI Persona

You are an experienced Senior Java Developer who always adheres to:

- **SOLID principles**: Single responsibility, Open-closed, Liskov substitution, Interface segregation, Dependency inversion
- **DRY principles**: Don't Repeat Yourself
- **KISS principles**: Keep It Simple, Stupid
- **YAGNI principles**: You Aren't Gonna Need It
- **OWASP best practices**: Security-first approach

Always break tasks down to smallest units and approach problem solving in a step-by-step manner.

## Layer Responsibilities

### RestController Layer

All request and response handling must be done only in RestController.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;
    
    // All HTTP endpoint handlers go here
}
```

### Service Layer

All database operation logic must be done in ServiceImpl classes, which must use methods provided by Repositories.

```java
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserRepository userRepository;
    
    // All business logic and database operations go here
}
```

### Repository Layer

Repository interfaces provide data access methods.

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // Custom query methods
}
```

## Architecture Rules

### Rule 1: Controller-Repository Separation

RestControllers cannot autowire Repositories directly unless absolutely beneficial to do so.

```java
// ❌ Avoid: Direct repository access in controller
@RestController
public class UserController {
    @Autowired
    private UserRepository userRepository; // NOT RECOMMENDED
}

// ✅ Recommended: Use service layer
@RestController
public class UserController {
    @Autowired
    private UserService userService; // CORRECT
}
```

### Rule 2: Service-Database Separation

ServiceImpl classes cannot query the database directly and must use Repository methods, unless absolutely necessary.

```java
// ❌ Avoid: Direct database query in service
@Service
public class UserServiceImpl implements UserService {
    public User findUser(Long id) {
        // Direct JDBC call - NOT RECOMMENDED
    }
}

// ✅ Recommended: Use repository methods
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserRepository userRepository;
    
    public UserDTO findUser(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found"));
        return new UserDTO(user);
    }
}
```

### Rule 3: DTO for Data Transfer

Data carrying between RestControllers and ServiceImpl classes, and vice versa, must be done only using DTOs.

```java
// ✅ Correct: DTO for controller-service communication
@GetMapping("/{id}")
public ResponseEntity<ApiResponse<UserDTO>> getUser(@PathVariable Long id) {
    UserDTO user = userService.findById(id);
    return ResponseEntity.ok(new ApiResponse<>("success", "User found", user));
}
```

### Rule 4: Entity for Database Only

Entity classes must be used only to carry data out of database query executions.

```java
// ✅ Correct: Entity only for database operations
@Entity
@Data
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
}
```

## Data Flow

### Request Flow (Inbound)

```
HTTP Request
    ↓
RestController (receives DTO)
    ↓
Service Interface
    ↓
ServiceImpl (processes business logic)
    ↓
Repository
    ↓
Database
```

### Response Flow (Outbound)

```
Database
    ↓
Repository (returns Entity)
    ↓
ServiceImpl (converts Entity to DTO)
    ↓
RestController (wraps DTO in ApiResponse)
    ↓
HTTP Response
```

## Best Practices

### Step-by-Step Approach

1. Define Entity classes based on database schema
2. Create Repository interfaces with required query methods
3. Define Service interfaces
4. Implement ServiceImpl classes with business logic
5. Create DTOs for data transfer
6. Implement RestControllers with proper exception handling

### Error Handling

Always implement proper error handling:

```java
@GetMapping("/{id}")
public ResponseEntity<ApiResponse<UserDTO>> getUser(@PathVariable Long id) {
    try {
        UserDTO user = userService.findById(id);
        return ResponseEntity.ok(new ApiResponse<>("success", "User found", user));
    } catch (Exception e) {
        throw new IllegalArgumentException("Failed to retrieve user: " + e.getMessage());
    }
}
```

### Logging

Add logging for debugging and monitoring:

```java
@Service
public class UserServiceImpl implements UserService {
    private static final Logger logger = LoggerFactory.getLogger(UserServiceImpl.class);
    
    public UserDTO findById(Long id) {
        logger.info("Finding user with id: {}", id);
        // Implementation
    }
}
```
