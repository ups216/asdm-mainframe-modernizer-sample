# Response Handling

API response format and exception handling patterns.

## ApiResponse Class

Standard response wrapper for all API responses.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ApiResponse<T> {
    private String result;    // "success" or "error"
    private String message;   // success or error message
    private T data;           // return object from service class, if successful
}
```

### Usage Examples

#### Success Response

```java
@GetMapping("/{id}")
public ResponseEntity<ApiResponse<UserDTO>> getById(@PathVariable Long id) {
    UserDTO user = userService.findById(id);
    ApiResponse<UserDTO> response = new ApiResponse<>(
        "success",
        "User found",
        user
    );
    return ResponseEntity.ok(response);
}
```

#### Success Response with Created Status

```java
@PostMapping
public ResponseEntity<ApiResponse<UserDTO>> create(@RequestBody UserCreateDTO dto) {
    UserDTO user = userService.create(dto);
    ApiResponse<UserDTO> response = new ApiResponse<>(
        "success",
        "User created successfully",
        user
    );
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

#### Success Response with No Data

```java
@DeleteMapping("/{id}")
public ResponseEntity<ApiResponse<Void>> delete(@PathVariable Long id) {
    userService.delete(id);
    ApiResponse<Void> response = new ApiResponse<>(
        "success",
        "User deleted successfully",
        null
    );
    return ResponseEntity.ok(response);
}
```

## GlobalExceptionHandler Class

Centralized exception handling for all controllers.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    public static ResponseEntity<ApiResponse<?>> errorResponseEntity(String message, HttpStatus status) {
        ApiResponse<?> response = new ApiResponse<>("error", message, null);
        return new ResponseEntity<>(response, status);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiResponse<?>> handleIllegalArgumentException(IllegalArgumentException ex) {
        return errorResponseEntity(ex.getMessage(), HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(NoSuchElementException.class)
    public ResponseEntity<ApiResponse<?>> handleNoSuchElementException(NoSuchElementException ex) {
        return errorResponseEntity(ex.getMessage(), HttpStatus.NOT_FOUND);
    }
}
```

## Extended Exception Handlers

### Complete GlobalExceptionHandler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    public static ResponseEntity<ApiResponse<?>> errorResponseEntity(String message, HttpStatus status) {
        ApiResponse<?> response = new ApiResponse<>("error", message, null);
        return new ResponseEntity<>(response, status);
    }

    // 400 Bad Request - Validation errors
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiResponse<?>> handleIllegalArgumentException(IllegalArgumentException ex) {
        logger.warn("IllegalArgumentException: {}", ex.getMessage());
        return errorResponseEntity(ex.getMessage(), HttpStatus.BAD_REQUEST);
    }

    // 400 Bad Request - Bean validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<?>> handleValidationException(MethodArgumentNotValidException ex) {
        String errorMessage = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.joining(", "));
        
        logger.warn("Validation error: {}", errorMessage);
        return errorResponseEntity(errorMessage, HttpStatus.BAD_REQUEST);
    }

    // 404 Not Found
    @ExceptionHandler(NoSuchElementException.class)
    public ResponseEntity<ApiResponse<?>> handleNoSuchElementException(NoSuchElementException ex) {
        logger.warn("NoSuchElementException: {}", ex.getMessage());
        return errorResponseEntity(ex.getMessage(), HttpStatus.NOT_FOUND);
    }

    // 404 Not Found - Entity not found
    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ApiResponse<?>> handleEntityNotFoundException(EntityNotFoundException ex) {
        logger.warn("EntityNotFoundException: {}", ex.getMessage());
        return errorResponseEntity(ex.getMessage(), HttpStatus.NOT_FOUND);
    }

    // 409 Conflict - Duplicate entry
    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<ApiResponse<?>> handleDataIntegrityViolationException(DataIntegrityViolationException ex) {
        String message = "Data integrity violation";
        if (ex.getCause() instanceof ConstraintViolationException) {
            message = "Duplicate entry or constraint violation";
        }
        logger.warn("DataIntegrityViolationException: {}", message);
        return errorResponseEntity(message, HttpStatus.CONFLICT);
    }

    // 500 Internal Server Error
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<?>> handleGenericException(Exception ex) {
        logger.error("Unexpected error: ", ex);
        return errorResponseEntity("An unexpected error occurred", HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

## Custom Exceptions

### Creating Custom Exceptions

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
    
    public ResourceNotFoundException(String resourceName, Long id) {
        super(String.format("%s not found with id: %d", resourceName, id));
    }
}

public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
    
    public DuplicateResourceException(String resourceName, String field, String value) {
        super(String.format("%s already exists with %s: %s", resourceName, field, value));
    }
}

public class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
}
```

### Handling Custom Exceptions

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiResponse<?>> handleResourceNotFoundException(ResourceNotFoundException ex) {
        return errorResponseEntity(ex.getMessage(), HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ApiResponse<?>> handleDuplicateResourceException(DuplicateResourceException ex) {
        return errorResponseEntity(ex.getMessage(), HttpStatus.CONFLICT);
    }

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ApiResponse<?>> handleBusinessException(BusinessException ex) {
        return errorResponseEntity(ex.getMessage(), HttpStatus.UNPROCESSABLE_ENTITY);
    }
}
```

### Using Custom Exceptions in Service

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDTO findById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
        return new UserDTO(user);
    }
    
    @Override
    public UserDTO create(UserCreateDTO dto) {
        if (userRepository.existsByEmail(dto.email())) {
            throw new DuplicateResourceException("User", "email", dto.email());
        }
        
        User user = new User();
        user.setName(dto.name());
        user.setEmail(dto.email());
        user = userRepository.save(user);
        
        return new UserDTO(user);
    }
}
```

## Response Formats

### Success Response

```json
{
    "result": "success",
    "message": "User found",
    "data": {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com"
    }
}
```

### Error Response

```json
{
    "result": "error",
    "message": "User not found with id: 999",
    "data": null
}
```

### List Response

```json
{
    "result": "success",
    "message": "Users retrieved successfully",
    "data": [
        {
            "id": 1,
            "name": "John Doe",
            "email": "john@example.com"
        },
        {
            "id": 2,
            "name": "Jane Smith",
            "email": "jane@example.com"
        }
    ]
}
```

### Paginated Response

```json
{
    "result": "success",
    "message": "Users retrieved successfully",
    "data": {
        "content": [...],
        "totalElements": 100,
        "totalPages": 10,
        "number": 0,
        "size": 10,
        "first": true,
        "last": false
    }
}
```

## Best Practices

### Consistent Error Messages

```java
// ✅ Good: Clear, actionable message
throw new IllegalArgumentException("Email already exists: " + email);

// ❌ Bad: Vague message
throw new IllegalArgumentException("Invalid data");
```

### Log Appropriate Severity

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    // Client errors - log as warning
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiResponse<?>> handleIllegalArgumentException(IllegalArgumentException ex) {
        logger.warn("Client error: {}", ex.getMessage());
        return errorResponseEntity(ex.getMessage(), HttpStatus.BAD_REQUEST);
    }

    // Server errors - log as error with stack trace
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<?>> handleGenericException(Exception ex) {
        logger.error("Server error: ", ex);
        return errorResponseEntity("An unexpected error occurred", HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### Never Expose Internal Details

```java
// ❌ Bad: Exposes internal details
@ExceptionHandler(Exception.class)
public ResponseEntity<ApiResponse<?>> handleGenericException(Exception ex) {
    return errorResponseEntity(ex.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
}

// ✅ Good: Generic message for internal errors
@ExceptionHandler(Exception.class)
public ResponseEntity<ApiResponse<?>> handleGenericException(Exception ex) {
    logger.error("Internal error: ", ex);
    return errorResponseEntity("An unexpected error occurred", HttpStatus.INTERNAL_SERVER_ERROR);
}
```

### Use Appropriate HTTP Status Codes

| Status | When to Use |
|--------|-------------|
| 200 OK | Successful GET, PUT |
| 201 CREATED | Successful POST |
| 204 NO CONTENT | Successful DELETE (no body) |
| 400 BAD REQUEST | Validation errors, invalid input |
| 404 NOT FOUND | Resource not found |
| 409 CONFLICT | Duplicate resource, constraint violation |
| 422 UNPROCESSABLE ENTITY | Business rule violation |
| 500 INTERNAL SERVER ERROR | Unexpected server errors |
