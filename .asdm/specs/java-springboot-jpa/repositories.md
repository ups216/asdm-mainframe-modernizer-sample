# Repositories

Repository/DAO patterns and JPQL best practices.

## Basic Repository Structure

### Required Annotations and Structure

1. Must annotate repository classes with `@Repository`
2. Repository classes must be of type interface
3. Must extend `JpaRepository` with the entity and entity ID as parameters

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

## Query Methods

### Method Naming Convention

Spring Data JPA automatically creates queries based on method names:

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Find by single field
    Optional<User> findByEmail(String email);
    
    // Find by multiple fields
    Optional<User> findByEmailAndName(String email, String name);
    
    // Find all by condition
    List<User> findByActiveTrue();
    
    // Count
    long countByActiveTrue();
    
    // Exists
    boolean existsByEmail(String email);
    
    // Delete
    void deleteByEmail(String email);
}
```

## JPQL Queries

### Basic JPQL

Must use JPQL for all `@Query` type methods, unless specified otherwise.

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmailCustom(@Param("email") String email);
    
    @Query("SELECT u FROM User u WHERE u.active = true ORDER BY u.name")
    List<User> findAllActiveUsers();
}
```

### Update Queries

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.id = :id")
    int deactivateUser(@Param("id") Long id);
}
```

### Native Queries

Use native queries when JPQL is insufficient:

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Query(value = "SELECT * FROM users WHERE email LIKE %:domain", nativeQuery = true)
    List<User> findByEmailDomain(@Param("domain") String domain);
}
```

## EntityGraph for N+1 Problem

Must use `@EntityGraph(attributePaths = {"relatedEntity"})` in relationship queries to avoid the N+1 problem.

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    // Without EntityGraph: N+1 problem
    // Each order triggers a separate query for user
    
    // With EntityGraph: Single query with JOIN
    @EntityGraph(attributePaths = {"user"})
    List<Order> findAll();
    
    // Multiple relationships
    @EntityGraph(attributePaths = {"user", "orderItems"})
    Optional<Order> findById(Long id);
    
    // With custom query
    @EntityGraph(attributePaths = {"user"})
    @Query("SELECT o FROM Order o WHERE o.status = :status")
    List<Order> findByStatus(@Param("status") String status);
}
```

## DTO Projections

Must use a DTO as the data container for multi-join queries with `@Query`.

### Interface-Based Projection

```java
// Projection interface
public interface UserNameAndEmail {
    Long getId();
    String getName();
    String getEmail();
}

// Repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Query("SELECT u.id as id, u.name as name, u.email as email FROM User u")
    List<UserNameAndEmail> findAllNamesAndEmails();
}
```

### Class-Based Projection (DTO)

```java
// DTO class
public record UserSummary(Long id, String name, String email) {}

// Repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Query("SELECT new com.example.dto.UserSummary(u.id, u.name, u.email) FROM User u")
    List<UserSummary> findAllSummaries();
}
```

### Constructor Expression

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    @Query("SELECT new com.example.dto.OrderSummary(" +
           "o.id, o.orderDate, o.total, u.name) " +
           "FROM Order o JOIN o.user u")
    List<OrderSummary> findAllOrderSummaries();
}
```

## Pagination and Sorting

### Pagination

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    Page<User> findByActiveTrue(Pageable pageable);
    
    @Query("SELECT u FROM User u WHERE u.active = true")
    Page<User> findActiveUsers(Pageable pageable);
}

// Usage in service
public Page<UserDTO> getActiveUsers(int page, int size) {
    Pageable pageable = PageRequest.of(page, size);
    Page<User> userPage = userRepository.findByActiveTrue(pageable);
    return userPage.map(UserDTO::new);
}
```

### Sorting

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    List<User> findByActiveTrueOrderByCreatedAtDesc();
    
    List<User> findByActiveTrue(Sort sort);
}

// Usage
Sort sort = Sort.by("createdAt").descending();
List<User> users = userRepository.findByActiveTrue(sort);
```

### Combined Pagination and Sorting

```java
public Page<UserDTO> getActiveUsers(int page, int size, String sortBy, String direction) {
    Sort sort = Sort.by(Sort.Direction.fromString(direction), sortBy);
    Pageable pageable = PageRequest.of(page, size, sort);
    Page<User> userPage = userRepository.findByActiveTrue(pageable);
    return userPage.map(UserDTO::new);
}
```

## Best Practices

### Use Optional for Single Results

```java
// ✅ Correct: Returns Optional
Optional<User> findByEmail(String email);

// ❌ Avoid: Returns null on no result
User findByEmail(String email);
```

### Naming Conventions

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // ✅ Good naming
    Optional<User> findByEmail(String email);
    List<User> findByActiveTrue();
    boolean existsByEmail(String email);
    
    // ❌ Avoid vague names
    User getOne(String email);
    List<User> getAll();
}
```

### Avoid Select N+1

```java
// ❌ Causes N+1 problem
List<Order> orders = orderRepository.findAll();
for (Order order : orders) {
    User user = order.getUser(); // Triggers separate query for each order
}

// ✅ Use EntityGraph
@EntityGraph(attributePaths = {"user"})
List<Order> orders = orderRepository.findAll();
```

### Batch Operations

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Modifying
    @Query("UPDATE User u SET u.active = :active WHERE u.id IN :ids")
    int updateActiveStatus(@Param("ids") List<Long> ids, @Param("active") boolean active);
    
    @Modifying
    @Query("DELETE FROM User u WHERE u.active = false")
    int deleteInactiveUsers();
}
```

### Transaction Requirements

For update/delete queries, ensure transactional context:

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Transactional
    public void deactivateUsers(List<Long> ids) {
        userRepository.updateActiveStatus(ids, false);
    }
}
```
