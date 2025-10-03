# Samvya CRUD Starter - Help Guide

Complete guide for getting help, troubleshooting, and understanding Samvya CRUD Starter.

---

## 📖 Table of Contents

1. [Getting Started](#getting-started)
2. [Common Issues](#common-issues)
3. [FAQ](#faq)
4. [Configuration Help](#configuration-help)
5. [Code Examples](#code-examples)
6. [Debugging Tips](#debugging-tips)
7. [Performance Tuning](#performance-tuning)
8. [Migration Guide](#migration-guide)
9. [Getting Support](#getting-support)
10. [Contributing](#contributing)

---

## Getting Started

### Prerequisites Checklist

Before starting, ensure you have:

- [ ] Java 17 or higher installed
- [ ] Spring Boot 3.x project created
- [ ] Maven or Gradle configured
- [ ] Database (MongoDB/MySQL/PostgreSQL) installed and running
- [ ] IDE (IntelliJ IDEA, VS Code, Eclipse) set up

### Installation Steps

#### Step 1: Add Samvya Dependency

**Gradle (build.gradle):**
```gradle
dependencies {
    implementation 'io.github.sachinnimbal:samvya-crud-starter:1.0.0'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
}
```

**Maven (pom.xml):**
```xml
<dependency>
    <groupId>io.github.sachinnimbal</groupId>
    <artifactId>samvya-crud-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

#### Step 2: Configure Database

**For MongoDB (application.yml):**
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/testdb
      database: testdb
```

**For MySQL (application.yml):**
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/testdb
    username: root
    password: password
  jpa:
    hibernate:
      ddl-auto: update
```

#### Step 3: Enable Samvya

```java
@SpringBootApplication
@SamvyaCrud  // <- Add this annotation
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### Step 4: Create Your First Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController extends SamvyaCrudController<UserController.User, String> {

    @Document(collection = "users")
    @Getter @Setter
    public static class User extends SamvyaMongoEntity<String> {
        @NotBlank private String name;
        @Email private String email;
    }
}
```

#### Step 5: Run and Test

```bash
# Start your application
./gradlew bootRun

# Test the API
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Sachin Nimbal","email":"sachin@nimbal.com"}'
```

---

## Common Issues

### Issue 1: "Service bean not found"

**Error Message:**
```
Service bean not found: userService
```

**Causes:**
1. Missing `@SamvyaCrud` annotation on main application class
2. Entity doesn't extend correct base class
3. Controller doesn't properly extend `SamvyaCrudController`

**Solutions:**

✅ **Solution 1: Add @SamvyaCrud**
```java
@SpringBootApplication
@SamvyaCrud  // <- Make sure this is present
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

✅ **Solution 2: Verify Entity Extends Base Class**
```java
// For MongoDB - use SamvyaMongoEntity
public static class User extends SamvyaMongoEntity<String> { }

// For MySQL - use SamvyaMySQLEntity
public static class User extends SamvyaMySQLEntity<Long> { }

// For PostgreSQL - use SamvyaPostgreSQLEntity
public static class User extends SamvyaPostgreSQLEntity<Long> { }
```

✅ **Solution 3: Verify Controller Extends SamvyaCrudController**
```java
@RestController
@RequestMapping("/api/users")
public class UserController extends SamvyaCrudController<User, String> {
    // Generic types must match: <EntityType, IDType>
}
```

---

### Issue 2: Database Connection Failed

#### MongoDB Connection Error

**Error Message:**
```
MongoTemplate not available
```

**Solutions:**

✅ **Add MongoDB Dependency**
```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
```

✅ **Configure Connection**
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/mydb
```

✅ **Verify MongoDB is Running**
```bash
# Check if MongoDB is running
mongosh

# Or check the service
systemctl status mongod  # Linux
brew services list | grep mongodb  # macOS
```

#### MySQL/PostgreSQL Connection Error

**Error Message:**
```
EntityManager not available
```

**Solutions:**

✅ **Add JPA and Driver Dependencies**
```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'com.mysql:mysql-connector-j'  // For MySQL
// OR
runtimeOnly 'org.postgresql:postgresql'  // For PostgreSQL
```

✅ **Configure Datasource**
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
```

✅ **Verify Database is Running**
```bash
# MySQL
mysql -u root -p

# PostgreSQL
psql -U postgres
```

---

### Issue 3: Validation Not Working

**Problem:** Validation annotations are ignored

**Solutions:**

✅ **Add Validation Dependency**
```gradle
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

✅ **Use Correct Package**
```java
// Correct - jakarta.validation
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Email;

// Wrong - javax.validation (old)
import javax.validation.constraints.NotBlank;  // DON'T USE
```

✅ **Add Validation Annotations**
```java
public static class User extends SamvyaMongoEntity<String> {
    @NotBlank(message = "Name is required")
    private String name;
    
    @Email(message = "Invalid email format")
    @NotBlank(message = "Email is required")
    private String email;
}
```

---

### Issue 4: Unique Constraint Not Enforced

**Problem:** Duplicate entries are allowed despite `@SamvyaUniqueConstraint`

**Solutions:**

✅ **Verify Annotation Syntax**
```java
@SamvyaUniqueConstraint(
    fields = {"email"},  // <- Field name must match exactly
    message = "Email already exists"
)
public static class User extends SamvyaMongoEntity<String> {
    private String email;  // <- Field name
}
```

✅ **Check Multiple Constraints**
```java
@SamvyaUniqueConstraint(fields = {"email"}, message = "Email exists")
@SamvyaUniqueConstraint(fields = {"phone"}, message = "Phone exists")
public static class User extends SamvyaMongoEntity<String> {
    private String email;
    private String phone;
}
```

✅ **For MongoDB: Verify Indexes**
```bash
mongosh
use mydb
db.users.getIndexes()
# Should show unique index on email field
```

---

### Issue 5: 404 Not Found on Endpoints

**Problem:** Endpoints return 404

**Solutions:**

✅ **Verify Base Path**
```java
@RequestMapping("/api/users")  // <- This is your base path
public class UserController extends SamvyaCrudController<User, String> { }

// Endpoints will be:
// POST   /api/users
// GET    /api/users
// GET    /api/users/{id}
// etc.
```

✅ **Check Application is Running**
```bash
# Check logs for startup banner
curl http://localhost:8080/actuator/health
```

✅ **Verify Port Configuration**
```yaml
server:
  port: 8080  # Default port
```

---

### Issue 6: Batch Insert Slow

**Problem:** Batch creation takes too long

**Solutions:**

✅ **Configure Batch Size (JPA)**
```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 1000
        order_inserts: true
        order_updates: true
```

✅ **Disable SQL Logging in Production**
```yaml
spring:
  jpa:
    show-sql: false  # Turn off in production
```

✅ **Increase Connection Pool**
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
```

✅ **Monitor Batch Processing**
```
# Check logs for batch timing
INFO: Batch creation completed: 10000 entities | Total: 8542 ms | Avg: 0.854 ms/entity
```

---

### Issue 7: Audit Fields Not Populated

**Problem:** `createdAt` and `updatedAt` are null

**Solutions:**

✅ **Enable Auditing for MongoDB**
```java
@SpringBootApplication
@SamvyaCrud
@EnableMongoAuditing  // <- Add this
public class Application { }
```

✅ **Enable Auditing for JPA**
```java
@SpringBootApplication
@SamvyaCrud
@EnableJpaAuditing  // <- Add this
public class Application { }
```

---

## FAQ

### General Questions

**Q: Do I need to write service layer code?**
A: No! Samvya auto-generates and injects the service layer.

**Q: Can I add custom endpoints?**
A: Yes! Just add methods to your controller:
```java
@GetMapping("/custom")
public ResponseEntity<List<User>> customEndpoint() {
    return ResponseEntity.ok(crudService.findAll());
}
```

**Q: What databases are supported?**
A: MongoDB, MySQL, and PostgreSQL are fully supported.

**Q: Is Samvya production-ready?**
A: Yes! It includes error handling, validation, auditing, and performance optimizations.

**Q: Can I customize the auto-generated endpoints?**
A: Yes, you can override methods in your controller.

### Technical Questions

**Q: What is the default batch size?**
A: 1000 entities per batch.

**Q: How do pagination parameters work?**
A: Use `page` (0-based), `size`, `sortBy`, and `sortDirection`:
```bash
/api/users/paged?page=0&size=20&sortBy=name&sortDirection=ASC
```

**Q: Can I use Samvya with existing projects?**
A: Yes! Just add the dependency and create controllers.

**Q: Does Samvya work with Spring Security?**
A: Yes! Samvya works seamlessly with Spring Security.

**Q: How do I version my API?**
A: Use versioned request mappings:
```java
@RequestMapping("/api/v1/users")
public class UserControllerV1 { }

@RequestMapping("/api/v2/users")
public class UserControllerV2 { }
```

**Q: Can I use DTOs instead of entities?**
A: Yes! Transform in custom endpoints:
```java
@PostMapping("/register")
public ResponseEntity<UserDTO> register(@RequestBody UserDTO dto) {
    User user = convertToEntity(dto);
    User created = crudService.create(user);
    return ResponseEntity.ok(convertToDTO(created));
}
```

---

## Configuration Help

### MongoDB Configuration Options

```yaml
spring:
  data:
    mongodb:
      # Simple URI
      uri: mongodb://localhost:27017/mydb
      
      # Or detailed config
      host: localhost
      port: 27017
      database: mydb
      username: admin
      password: secret
      authentication-database: admin
      
      # MongoDB Atlas
      # uri: mongodb+srv://user:pass@cluster.mongodb.net/mydb
```

### MySQL Configuration Options

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb?createDatabaseIfNotExist=true
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
    
    # Connection pooling
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      
  jpa:
    hibernate:
      ddl-auto: update  # Options: create, create-drop, update, validate, none
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        format_sql: true
```

### PostgreSQL Configuration Options

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: password
    driver-class-name: org.postgresql.Driver
    
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
```

### Samvya-Specific Configuration

```yaml
samvya:
  database:
    auto-create: true  # Auto-create database if not exists
  theme: dark  # Banner theme: dark or light
```

---

## Code Examples

### Example 1: Family Management System

```java
@RestController
@RequestMapping("/api/family")
public class FamilyController extends SamvyaCrudController<FamilyController.Member, String> {

    @Document(collection = "family_members")
    @Getter @Setter
    @SamvyaUniqueConstraint(fields = {"email"}, message = "Email exists")
    public static class Member extends SamvyaMongoEntity<String> {
        @NotBlank private String name;
        @Email @NotBlank private String email;
        @Min(0) @Max(150) private Integer age;
        private Role role;
    }

    enum Role { FATHER, MOTHER, SON, DAUGHTER }

    @GetMapping("/adults")
    public ResponseEntity<ApiResponse<List<Member>>> getAdults() {
        List<Member> adults = crudService.findAll().stream()
                .filter(m -> m.getAge() != null && m.getAge() >= 18)
                .toList();
        return ResponseEntity.ok(ApiResponse.success(adults));
    }
}
```

**Test:**
```bash
curl -X POST http://localhost:8080/api/family/batch \
  -H "Content-Type: application/json" \
  -d '[
    {"name":"Prakash Nimbal","email":"prakash@nimbal.com","age":55,"role":"FATHER"},
    {"name":"Sachin Nimbal","email":"sachin@nimbal.com","age":28,"role":"SON"}
  ]'
```

### Example 2: Product Catalog (MySQL)

```java
@RestController
@RequestMapping("/api/products")
public class ProductController extends SamvyaCrudController<ProductController.Product, Long> {

    @Entity
    @Table(name = "products")
    @Getter @Setter
    @SamvyaUniqueConstraint(fields = {"sku"}, message = "SKU exists")
    public static class Product extends SamvyaMySQLEntity<Long> {
        @NotBlank private String name;
        @NotBlank private String sku;
        @DecimalMin("0.01") private BigDecimal price;
        @Min(0) private Integer stock;
    }

    @GetMapping("/low-stock")
    public ResponseEntity<ApiResponse<List<Product>>> getLowStock() {
        List<Product> products = crudService.findAll().stream()
                .filter(p -> p.getStock() < 10)
                .toList();
        return ResponseEntity.ok(ApiResponse.success(products));
    }
}
```

### Example 3: Order System (PostgreSQL)

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController extends SamvyaCrudController<OrderController.Order, Long> {

    @Entity
    @Table(name = "orders")
    @Getter @Setter
    @SamvyaUniqueConstraint(fields = {"orderNumber"}, message = "Order exists")
    public static class Order extends SamvyaPostgreSQLEntity<Long> {
        @NotBlank private String orderNumber;
        @NotNull private BigDecimal totalAmount;
        @NotNull private OrderStatus status;
        @NotNull private LocalDateTime orderDate;
    }

    enum OrderStatus { PENDING, CONFIRMED, SHIPPED, DELIVERED }

    @GetMapping("/pending")
    public ResponseEntity<ApiResponse<List<Order>>> getPending() {
        List<Order> orders = crudService.findAll().stream()
                .filter(o -> o.getStatus() == OrderStatus.PENDING)
                .toList();
        return ResponseEntity.ok(ApiResponse.success(orders));
    }
}
```

---

## Debugging Tips

### Enable Debug Logging

```yaml
logging:
  level:
    io.github.sachinnimbal: DEBUG
    org.springframework.data: DEBUG
    org.hibernate: DEBUG  # For JPA
```

### Check Startup Logs

Look for:
```
             █████████                                                       
            ███░░░░░███                                                      
           ░███    ░░░  ██████  █████████████  █████ ██████████ ████ ██████  
           ░░█████████ ░░░░░███░░███░░███░░███░░███ ░░███░░███ ░███ ░░░░░███ 
            ░░░░░░░░███ ███████ ░███ ░███ ░███ ░███  ░███ ░███ ░███  ███████ 
            ███    ░██████░░███ ░███ ░███ ░███ ░░███ ███  ░███ ░███ ███░░███ 
            ░░█████████░░█████████████░███ █████ ░░█████   ░░███████░░████████
             ░░░░░░░░░  ░░░░░░░░░░░░░ ░░░ ░░░░░   ░░░░░     ░░░░░███ ░░░░░░░░ 
                                                            ███ ░███          
                                                            ░░██████           
                                                             ░░░░░░  
                            CRUD Framework - Version 1.0.0               
                        Lightweight & High-Performance CRUD  

INFO: Service registered: userService for entity: User
INFO: CRUD Controller initialized: UserController
```

### Test Individual Endpoints

```bash
# Test creation
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com"}' -v

# Test retrieval
curl http://localhost:8080/api/users -v

# Test pagination
curl "http://localhost:8080/api/users/paged?page=0&size=10" -v
```

### Common Log Messages

```
# Success
INFO: MongoDB entity created with ID: 123 | Time taken: 45 ms

# Validation Error
WARN: Validation failed for entity: User

# Duplicate Error
WARN: Duplicate entity detected: email already exists

# Not Found
WARN: Entity not found: User with id: 123
```

---

## Performance Tuning

### 1. Database Indexes

**MongoDB:**
```java
@Document(collection = "users")
@CompoundIndex(def = "{'email': 1}", unique = true)
@CompoundIndex(def = "{'name': 1, 'age': -1}")
public static class User extends SamvyaMongoEntity<String> { }
```

**MySQL/PostgreSQL:**
```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email"),
    @Index(name = "idx_name_age", columnList = "name, age")
})
public static class User extends SamvyaMySQLEntity<Long> { }
```

### 2. Connection Pooling

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
      connection-timeout: 30000
      idle-timeout: 600000
```

### 3. Batch Configuration

```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 1000
        order_inserts: true
        order_updates: true
```

### 4. Query Optimization

```java
// Bad - loads all into memory
public List<User> findAdults() {
    return crudService.findAll().stream()
            .filter(u -> u.getAge() >= 18)
            .toList();
}

// Good - uses pagination
public Page<User> findAdults(Pageable pageable) {
    return crudService.findAll(pageable);
}
```

---

## Migration Guide

### From Manual CRUD to Samvya

**Before:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService service;
    
    @PostMapping
    public User create(@RequestBody User user) {
        return service.save(user);
    }
    
    @GetMapping
    public List<User> getAll() {
        return service.findAll();
    }
    
    // ... 50+ more lines
}
```

**After:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController extends SamvyaCrudController<User, String> {
    // Done! All endpoints auto-generated
}
```

### Migration Steps

1. **Add Samvya dependency**
2. **Add `@SamvyaCrud` to main class**
3. **Replace service injection with controller extension**
4. **Update entity to extend base class**
5. **Remove manual repository and service classes**
6. **Test all endpoints**

---

## Getting Support

### Documentation

- **Complete API Docs**: [API_DOCUMENTATION.md](API_DOCUMENTATION.md)
- **README**: [README.md](README.md)
- **GitHub Wiki**: [Wiki](https://github.com/sachinnimbal/samvya-crud-examples/wiki)

### Community Support

- **GitHub Issues**: [Report Bug](https://github.com/sachinnimbal/samvya-crud-examples/issues)
- **GitHub Discussions**: Ask questions

### Direct Contact

- **Email**: sachinnimbal9@gmail.com
- **GitHub**: [@sachinnimbal](https://github.com/sachinnimbal)

### Before Asking for Help

Please provide:
1. Samvya version
2. Spring Boot version
3. Database type and version
4. Error logs (full stack trace)
5. Minimal reproducible example
6. What you've already tried

---

## Contributing

We welcome contributions!

### How to Contribute

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Update documentation
6. Submit a pull request

### Development Setup

```bash
git clone https://github.com/sachinnimbal/samvya-crud-examples.git
cd samvya-crud-examples
./gradlew build
./gradlew test
```

### Contribution Guidelines

- Write clear commit messages
- Add tests for new features
- Update documentation
- Follow existing code style
- Keep PRs focused and small

---

## Useful Commands

### Gradle Commands

```bash
# Build project
./gradlew build

# Run tests
./gradlew test

# Run application
./gradlew bootRun

# Clean build
./gradlew clean build

# Check dependencies
./gradlew dependencies
```

### Maven Commands

```bash
# Build project
mvn clean install

# Run tests
mvn test

# Run application
mvn spring-boot:run

# Package
mvn package
```

### Database Commands

**MongoDB:**
```bash
# Start MongoDB
mongod

# Connect to MongoDB
mongosh

# Show databases
show dbs

# Use database
use mydb

# Show collections
show collections

# Query collection
db.users.find()

# Create index
db.users.createIndex({email: 1}, {unique: true})
```

**MySQL:**
```bash
# Connect to MySQL
mysql -u root -p

# Show databases
SHOW DATABASES;

# Use database
USE mydb;

# Show tables
SHOW TABLES;

# Query table
SELECT * FROM users;

# Create index
CREATE INDEX idx_email ON users(email);
```

**PostgreSQL:**
```bash
# Connect to PostgreSQL
psql -U postgres

# List databases
\l

# Connect to database
\c mydb

# List tables
\dt

# Query table
SELECT * FROM users;

# Create index
CREATE INDEX idx_email ON users(email);
```

---

## Testing Checklist

### Before Deploying

- [ ] All endpoints return expected status codes
- [ ] Validation works correctly
- [ ] Unique constraints are enforced
- [ ] Pagination works as expected
- [ ] Sorting works correctly
- [ ] Error messages are clear
- [ ] Audit fields are populated
- [ ] Batch operations complete successfully
- [ ] Performance is acceptable
- [ ] Logs are informative

### Test Commands

```bash
# Test create
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Sachin Nimbal","email":"sachin@nimbal.com"}'

# Test get all
curl http://localhost:8080/api/users

# Test pagination
curl "http://localhost:8080/api/users/paged?page=0&size=10"

# Test sorting
curl "http://localhost:8080/api/users?sortBy=name&sortDirection=ASC"

# Test get by ID (replace with actual ID)
curl http://localhost:8080/api/users/123

# Test update
curl -X PATCH http://localhost:8080/api/users/123 \
  -H "Content-Type: application/json" \
  -d '{"age":29}'

# Test delete
curl -X DELETE http://localhost:8080/api/users/123

# Test count
curl http://localhost:8080/api/users/count

# Test exists
curl http://localhost:8080/api/users/exists/123

# Test batch create
curl -X POST http://localhost:8080/api/users/batch \
  -H "Content-Type: application/json" \
  -d '[{"name":"User1","email":"user1@test.com"},{"name":"User2","email":"user2@test.com"}]'
```

---

## Quick Reference

### Entity Base Classes

| Database | Base Class | ID Type |
|----------|------------|---------|
| MongoDB | `SamvyaMongoEntity<String>` | String |
| MySQL | `SamvyaMySQLEntity<Long>` | Long |
| PostgreSQL | `SamvyaPostgreSQLEntity<Long>` | Long |

### Validation Annotations

| Annotation | Purpose |
|-----------|---------|
| `@NotNull` | Field cannot be null |
| `@NotBlank` | String cannot be null/empty |
| `@Email` | Email validation |
| `@Size(min, max)` | String/collection size |
| `@Min(value)` | Minimum number |
| `@Max(value)` | Maximum number |
| `@Pattern(regexp)` | Regex validation |

### HTTP Status Codes

| Code | Status | When Used |
|------|--------|-----------|
| 200 | OK | Successful GET/PATCH/DELETE |
| 201 | Created | Successful POST |
| 400 | Bad Request | Validation error |
| 404 | Not Found | Entity not found |
| 409 | Conflict | Unique constraint violation |
| 500 | Internal Server Error | Server error |

### Auto-Generated Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/entity` | Create single |
| POST | `/api/entity/batch` | Batch create |
| GET | `/api/entity` | Get all |
| GET | `/api/entity/paged` | Paginated |
| GET | `/api/entity/{id}` | Get by ID |
| PATCH | `/api/entity/{id}` | Update |
| DELETE | `/api/entity/{id}` | Delete |
| GET | `/api/entity/count` | Count |
| GET | `/api/entity/exists/{id}` | Exists |

---

## Troubleshooting Flowchart

```
Problem occurred
    ↓
Is it a startup issue?
    ├─ Yes → Check @SamvyaCrud annotation
    │         Check database configuration
    │         Check dependencies
    │         Review startup logs
    │
    └─ No → Is it a runtime issue?
            ├─ Yes → Is it validation?
            │         ├─ Yes → Check validation annotations
            │         │         Check request format
            │         │
            │         └─ No → Is it a database issue?
            │                   ├─ Yes → Check connection
            │                   │         Check credentials
            │                   │         Check database exists
            │                   │
            │                   └─ No → Is it a performance issue?
            │                             ├─ Yes → Enable batch processing
            │                             │         Add indexes
            │                             │         Use pagination
            │                             │
            │                             └─ No → Check documentation
            │                                       Contact support
```

---

## Environment-Specific Tips

### Development Environment

```yaml
spring:
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create-drop

logging:
  level:
    io.github.sachinnimbal: DEBUG
```

### Production Environment

```yaml
spring:
  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

logging:
  level:
    io.github.sachinnimbal: INFO
    
samvya:
  database:
    auto-create: false
```

### Testing Environment

```yaml
spring:
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create-drop

logging:
  level:
    io.github.sachinnimbal: DEBUG
```

---

## Best Practices Summary

1. **Always use validation** - Add appropriate constraints
2. **Define unique constraints** - Use `@SamvyaUniqueConstraint`
3. **Use pagination** - For large datasets
4. **Add indexes** - For frequently queried fields
5. **Handle errors** - Return meaningful error messages
6. **Test thoroughly** - Test all endpoints before deployment
7. **Monitor performance** - Check logs for timing information
8. **Version your API** - Use versioned endpoints
9. **Document custom endpoints** - Add JavaDoc comments
10. **Keep it simple** - Let Samvya handle the boilerplate

---

## Additional Resources

### Official Documentation
- [Complete API Documentation](API_DOCUMENTATION.md)
- [README](README.md)
- [GitHub Repository](https://github.com/sachinnimbal/samvya-crud-examples)

### Tutorials
- Quick Start Guide
- MongoDB Integration
- MySQL/PostgreSQL Setup
- Advanced Features

### Community
- GitHub Discussions
- Email Support: sachinnimbal9@gmail.com

---

## Version Information

**Current Version:** 1.0.0  
**Release Date:** October 2025  
**Compatibility:** Spring Boot 3.x, Java 17+  
**License:** Apache 2.0

---

## Need More Help?

If you can't find the answer in this guide:

1. Check the [Complete API Documentation](API_DOCUMENTATION.md)
2. Browse [Example Projects](https://github.com/sachinnimbal/samvya-crud-examples)
3. Search [GitHub Issues](https://github.com/sachinnimbal/samvya-crud-examples/issues)
4. Ask on [GitHub Discussions](https://github.com/sachinnimbal/samvya-crud-examples/discussions)
5. Email: sachinnimbal9@gmail.com
---

**Last Updated:** October 2025  
**Maintained by:** Sachin Nimbal

**Thank you for using Samvya CRUD Starter!**
