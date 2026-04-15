# REST Controllers

REST controller design and API routing best practices.

## Basic Controller Structure

### Required Annotations

1. Must annotate controller classes with `@RestController`
2. Must specify class-level API routes with `@RequestMapping`
3. All dependencies in class methods must be `@Autowired` without a constructor

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    // HTTP handlers
}
```

## HTTP Method Mappings

### Resource-Based Routes

Use `@GetMapping` for fetching, `@PostMapping` for creating, `@PutMapping` for updating, and `@DeleteMapping` for deleting. Keep paths resource-based, avoiding verbs.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // ✅ Correct: Resource-based paths
    @GetMapping
    public ResponseEntity<ApiResponse<List<UserDTO>>> getAll() {}
    
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDTO>> getById(@PathVariable Long id) {}
    
    @PostMapping
    public ResponseEntity<ApiResponse<UserDTO>> create(@RequestBody UserCreateDTO dto) {}
    
    @PutMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDTO>> update(@PathVariable Long id, @RequestBody UserUpdateDTO dto) {}
    
    @DeleteMapping("/{id}")
    public ResponseEntity<ApiResponse<Void>> delete(@PathVariable Long id) {}
    
    // ❌ Avoid: Verb-based paths
    // @GetMapping("/get-all")
    // @PostMapping("/create")
    // @PutMapping("/update/{id}")
    // @DeleteMapping("/delete/{id}")
}
```

## Response Types

Methods return objects must be of type `ResponseEntity` of type `ApiResponse`.

```java
@GetMapping("/{id}")
public ResponseEntity<ApiResponse<UserDTO>> getById(@PathVariable Long id) {
    UserDTO user = userService.findById(id);
    ApiResponse<UserDTO> response = new ApiResponse<>("success", "User found", user);
    return ResponseEntity.ok(response);
}

@PostMapping
public ResponseEntity<ApiResponse<UserDTO>> create(@RequestBody UserCreateDTO dto) {
    UserDTO user = userService.create(dto);
    ApiResponse<UserDTO> response = new ApiResponse<>("success", "User created", user);
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

## Exception Handling

All class method logic must be implemented in try-catch block(s). Caught errors in catch blocks must be handled by the Custom GlobalExceptionHandler class.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDTO>> getById(@PathVariable Long id) {
        try {
            UserDTO user = userService.findById(id);
            ApiResponse<UserDTO> response = new ApiResponse<>("success", "User found", user);
            return ResponseEntity.ok(response);
        } catch (IllegalArgumentException e) {
            throw e; // Let GlobalExceptionHandler handle it
        } catch (Exception e) {
            throw new RuntimeException("Failed to retrieve user", e);
        }
    }
    
    @PostMapping
    public ResponseEntity<ApiResponse<UserDTO>> create(@RequestBody UserCreateDTO dto) {
        try {
            UserDTO user = userService.create(dto);
            ApiResponse<UserDTO> response = new ApiResponse<>("success", "User created successfully", user);
            return ResponseEntity.status(HttpStatus.CREATED).body(response);
        } catch (IllegalArgumentException e) {
            throw e;
        } catch (Exception e) {
            throw new RuntimeException("Failed to create user", e);
        }
    }
}
```

## Complete Controller Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public ResponseEntity<ApiResponse<List<UserDTO>>> getAll() {
        try {
            List<UserDTO> users = userService.findAll();
            ApiResponse<List<UserDTO>> response = new ApiResponse<>(
                "success", 
                "Users retrieved successfully", 
                users
            );
            return ResponseEntity.ok(response);
        } catch (Exception e) {
            throw new RuntimeException("Failed to retrieve users", e);
        }
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDTO>> getById(@PathVariable Long id) {
        try {
            UserDTO user = userService.findById(id);
            ApiResponse<UserDTO> response = new ApiResponse<>(
                "success", 
                "User found", 
                user
            );
            return ResponseEntity.ok(response);
        } catch (IllegalArgumentException e) {
            throw e;
        } catch (Exception e) {
            throw new RuntimeException("Failed to retrieve user", e);
        }
    }
    
    @GetMapping("/email/{email}")
    public ResponseEntity<ApiResponse<UserDTO>> getByEmail(@PathVariable String email) {
        try {
            UserDTO user = userService.findByEmail(email);
            ApiResponse<UserDTO> response = new ApiResponse<>(
                "success", 
                "User found", 
                user
            );
            return ResponseEntity.ok(response);
        } catch (IllegalArgumentException e) {
            throw e;
        } catch (Exception e) {
            throw new RuntimeException("Failed to retrieve user", e);
        }
    }
    
    @PostMapping
    public ResponseEntity<ApiResponse<UserDTO>> create(@RequestBody UserCreateDTO dto) {
        try {
            UserDTO user = userService.create(dto);
            ApiResponse<UserDTO> response = new ApiResponse<>(
                "success", 
                "User created successfully", 
                user
            );
            return ResponseEntity.status(HttpStatus.CREATED).body(response);
        } catch (IllegalArgumentException e) {
            throw e;
        } catch (Exception e) {
            throw new RuntimeException("Failed to create user", e);
        }
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDTO>> update(
            @PathVariable Long id, 
            @RequestBody UserUpdateDTO dto) {
        try {
            UserDTO user = userService.update(id, dto);
            ApiResponse<UserDTO> response = new ApiResponse<>(
                "success", 
                "User updated successfully", 
                user
            );
            return ResponseEntity.ok(response);
        } catch (IllegalArgumentException e) {
            throw e;
        } catch (Exception e) {
            throw new RuntimeException("Failed to update user", e);
        }
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<ApiResponse<Void>> delete(@PathVariable Long id) {
        try {
            userService.delete(id);
            ApiResponse<Void> response = new ApiResponse<>(
                "success", 
                "User deleted successfully", 
                null
            );
            return ResponseEntity.ok(response);
        } catch (IllegalArgumentException e) {
            throw e;
        } catch (Exception e) {
            throw new RuntimeException("Failed to delete user", e);
        }
    }
}
```

## Pagination Support

```java
@GetMapping
public ResponseEntity<ApiResponse<Page<UserDTO>>> getAll(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "asc") String direction) {
    try {
        Page<UserDTO> users = userService.findAll(page, size, sortBy, direction);
        ApiResponse<Page<UserDTO>> response = new ApiResponse<>(
            "success", 
            "Users retrieved successfully", 
            users
        );
        return ResponseEntity.ok(response);
    } catch (Exception e) {
        throw new RuntimeException("Failed to retrieve users", e);
    }
}
```

## Search and Filtering

```java
@GetMapping("/search")
public ResponseEntity<ApiResponse<List<UserDTO>>> search(
        @RequestParam(required = false) String name,
        @RequestParam(required = false) String email,
        @RequestParam(required = false) Boolean active) {
    try {
        List<UserDTO> users = userService.search(name, email, active);
        ApiResponse<List<UserDTO>> response = new ApiResponse<>(
            "success", 
            "Search completed", 
            users
        );
        return ResponseEntity.ok(response);
    } catch (Exception e) {
        throw new RuntimeException("Search failed", e);
    }
}
```

## Path Variables and Request Parameters

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    // Path variable
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<OrderDTO>> getById(@PathVariable Long id) {
        // ...
    }
    
    // Multiple path variables
    @GetMapping("/{orderId}/items/{itemId}")
    public ResponseEntity<ApiResponse<OrderItemDTO>> getItem(
            @PathVariable Long orderId,
            @PathVariable Long itemId) {
        // ...
    }
    
    // Request parameters
    @GetMapping
    public ResponseEntity<ApiResponse<List<OrderDTO>>> getByStatus(
            @RequestParam(required = false) String status,
            @RequestParam(defaultValue = "0") int page) {
        // ...
    }
    
    // Date parameters
    @GetMapping("/by-date")
    public ResponseEntity<ApiResponse<List<OrderDTO>>> getByDateRange(
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate from,
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate to) {
        // ...
    }
}
```

## Best Practices

### Use Meaningful HTTP Status Codes

```java
// 200 OK - Successful GET, PUT
return ResponseEntity.ok(response);

// 201 CREATED - Successful POST
return ResponseEntity.status(HttpStatus.CREATED).body(response);

// 204 NO CONTENT - Successful DELETE (if no body needed)
return ResponseEntity.noContent().build();

// 400 BAD REQUEST - Validation errors (handled by GlobalExceptionHandler)
// 404 NOT FOUND - Resource not found (handled by GlobalExceptionHandler)
```

### Validate Request Body

```java
@PostMapping
public ResponseEntity<ApiResponse<UserDTO>> create(
        @Valid @RequestBody UserCreateDTO dto) {
    // @Valid triggers validation annotations in DTO
    UserDTO user = userService.create(dto);
    return ResponseEntity.status(HttpStatus.CREATED)
        .body(new ApiResponse<>("success", "User created", user));
}
```

### Document API

```java
@RestController
@RequestMapping("/api/users")
@Tag(name = "User Management", description = "APIs for user operations")
public class UserController {
    
    @Operation(summary = "Get user by ID", description = "Returns a single user by their ID")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "User found"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDTO>> getById(
            @Parameter(description = "User ID") @PathVariable Long id) {
        // ...
    }
}
```
