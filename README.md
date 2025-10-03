# Samvya CRUD Starter

[![Maven Central](https://img.shields.io/badge/Maven%20Central-1.0.0-blue.svg)](https://central.sonatype.com/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-17%2B-orange.svg)](https://www.oracle.com/java/)

**Zero-Boilerplate CRUD Framework for Spring Boot** - Create fully functional REST APIs with a single annotation!

---

## ✨ Why Samvya CRUD Starter?

Stop writing repetitive CRUD code. Samvya automatically generates:
- ✅ **9 REST endpoints** per entity (Create, Read, Update, Delete + more)
- ✅ **Service layer** - auto-generated and auto-wired
- ✅ **Batch operations** - optimized chunked processing
- ✅ **Pagination & sorting** - built-in support
- ✅ **Validation** - automatic request validation
- ✅ **Unique constraints** - declarative with custom messages
- ✅ **Audit trails** - automatic createdAt/updatedAt tracking
- ✅ **Error handling** - standardized API responses
- ✅ **Multi-database** - MongoDB, MySQL, PostgreSQL

### Before Samvya 😓
```java
// Entity, Repository, Service, ServiceImpl, Controller
// 5 files, 200+ lines of boilerplate code
```

### After Samvya 😎
```java
@RestController
@RequestMapping("/api/family-members")
public class FamilyController extends SamvyaCrudController<FamilyMember, String> {
    // Done! 9 endpoints auto-generated
}
```

---

## 🚀 Quick Start

### 1. Add Dependency

**Gradle:**
```gradle
implementation 'io.github.sachinnimbal:samvya-crud-starter:1.0.0'
```

**Maven:**
```xml
<dependency>
    <groupId>io.github.sachinnimbal</groupId>
    <artifactId>samvya-crud-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

### 2. Enable Samvya

```java
@SpringBootApplication
@SamvyaCrud  // <- Add this annotation
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 3. Create Your First Controller

```java
@RestController
@RequestMapping("/api/family-members")
public class FamilyController extends SamvyaCrudController<FamilyController.FamilyMember, String> {

    @Document(collection = "family_members")
    @Getter @Setter
    @SamvyaUniqueConstraint(fields = {"email"}, message = "Email already exists")
    public static class FamilyMember extends SamvyaMongoEntity<String> {
        @NotBlank private String name;
        @Email @NotBlank private String email;
        private Integer age;
    }
}
```

### 4. Test Your API

```bash
# Create a family member
curl -X POST http://localhost:8080/api/family-members \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sachin Nimbal",
    "email": "sachin@nimbal.com",
    "age": 28
  }'

# Get all family members
curl http://localhost:8080/api/family-members
```

**That's it!** You now have a complete REST API with 9 endpoints.

---

## 📡 Auto-Generated Endpoints

Every controller automatically gets these endpoints:

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/entity` | Create single entity |
| `POST` | `/api/entity/batch` | Batch create (optimized) |
| `GET` | `/api/entity` | Get all (with sorting) |
| `GET` | `/api/entity/paged` | Get paginated results |
| `GET` | `/api/entity/{id}` | Get by ID |
| `PATCH` | `/api/entity/{id}` | Partial update |
| `DELETE` | `/api/entity/{id}` | Delete entity |
| `GET` | `/api/entity/count` | Get total count |
| `GET` | `/api/entity/exists/{id}` | Check if exists |

---

## 🎯 Key Features

### 1. Zero Service Layer Code

```java
// Samvya auto-generates and injects the service
@RestController
@RequestMapping("/api/users")
public class UserController extends SamvyaCrudController<User, String> {
    // Service automatically available as 'crudService'
    
    @GetMapping("/adults")
    public List<User> getAdults() {
        return crudService.findAll().stream()
                .filter(u -> u.getAge() >= 18)
                .toList();
    }
}
```

### 2. Declarative Unique Constraints

```java
@SamvyaUniqueConstraint(fields = {"email"}, message = "Email already exists")
@SamvyaUniqueConstraint(fields = {"phone"}, message = "Phone already registered")
public static class User extends SamvyaMongoEntity<String> {
    private String email;
    private String phone;
}
```

### 3. Optimized Batch Operations

```java
// Automatically processes in chunks of 1000
List<User> users = // 10,000 users
List<User> created = crudService.createBatch(users);
// Processes in 10 batches automatically
```

### 4. Built-in Pagination

```bash
curl "http://localhost:8080/api/users/paged?page=0&size=20&sortBy=name&sortDirection=ASC"
```

Response includes:
```json
{
  "content": [...],
  "currentPage": 0,
  "totalPages": 5,
  "totalElements": 100,
  "first": true,
  "last": false
}
```

### 5. Automatic Validation

```java
public static class User extends SamvyaMongoEntity<String> {
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100)
    private String name;
    
    @Email(message = "Invalid email")
    @NotBlank
    private String email;
    
    @Min(0) @Max(150)
    private Integer age;
}
```

### 6. Audit Trail

All entities automatically track:
- `createdAt` - Creation timestamp
- `updatedAt` - Last update timestamp
- `createdBy` - User who created (customizable)
- `updatedBy` - User who updated (customizable)

---

## 💾 Database Support

### MongoDB
```java
public static class User extends SamvyaMongoEntity<String> {
    // String ID auto-generated
}
```

### MySQL
```java
@Entity
@Table(name = "users")
public static class User extends SamvyaMySQLEntity<Long> {
    // Long ID with IDENTITY strategy
}
```

### PostgreSQL
```java
@Entity
@Table(name = "users")
public static class User extends SamvyaPostgreSQLEntity<Long> {
    // Long ID with SEQUENCE strategy
}
```

---

## 📚 Documentation

- **[Complete API Documentation](API_DOCUMENTATION.md)** - Full endpoint reference
- **[Getting Started Guide](GETTING_STARTED.md)** - Step-by-step setup
- **[Example Projects](https://github.com/sachinnimbal/samvya-crud-examples)** - Working examples
- **[Wiki](https://github.com/sachinnimbal/samvya-crud-examples/wiki)** - Advanced topics

---

## 🔧 Configuration

### MongoDB Setup

**build.gradle:**
```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
```

**application.yml:**
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/mydb
```

### MySQL Setup

**build.gradle:**
```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'com.mysql:mysql-connector-j'
```

**application.yml:**
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
  jpa:
    hibernate:
      ddl-auto: update
```

---

## 🎓 Examples

### Family Management System

```java
@RestController
@RequestMapping("/api/family-members")
class FamilyController extends SamvyaCrudController<FamilyController.FamilyMember, String> {

    @Document(collection = "family_members")
    @Getter @Setter
    @SamvyaUniqueConstraint(fields = {"email"}, message = "Email exists")
    public static class FamilyMember extends SamvyaMongoEntity<String> {
        @NotBlank private String name;
        @Email @NotBlank private String email;
        @Min(0) @Max(150) private Integer age;
        private FamilyRole role;
    }

    enum FamilyRole {
        FATHER, MOTHER, SON, DAUGHTER
    }

    // Custom endpoint
    @GetMapping("/by-role/{role}")
    public ResponseEntity<ApiResponse<List<FamilyMember>>> findByRole(
            @PathVariable FamilyRole role) {
        List<FamilyMember> members = crudService.findAll().stream()
                .filter(m -> m.getRole() == role)
                .toList();
        return ResponseEntity.ok(ApiResponse.success(members));
    }
}
```

### Sample Data

```bash
curl -X POST http://localhost:8080/api/family-members/batch \
  -H "Content-Type: application/json" \
  -d '[
    {"name":"Prakash Nimbal","email":"prakash@nimbal.com","age":55,"role":"FATHER"},
    {"name":"Shailaja Nimbal","email":"shailaja@nimbal.com","age":50,"role":"MOTHER"},
    {"name":"Sachin Nimbal","email":"sachin@nimbal.com","age":28,"role":"SON"},
    {"name":"Samarth Nimbal","email":"samarth@nimbal.com","age":16,"role":"SON"},
    {"name":"Sahana Nimbal","email":"sahana@nimbal.com","age":22,"role":"DAUGHTER"}
  ]'
```

---

## ⚡ Performance

### Batch Processing
- Automatic chunking (1000 entities per batch)
- EntityManager flush and clear between batches
- Optimized for millions of records

```
INFO: Batch creation started: 10000 entities
INFO: Processing batch 1/10 (1000 entities)
INFO: Processing batch 2/10 (1000 entities)
...
INFO: Batch creation completed | Total: 8542 ms | Avg: 0.854 ms/entity
```

### Pagination
- Zero memory overhead for large datasets
- Database-level pagination
- Combined with sorting for optimal queries

---

## 🛠️ Advanced Features

### Composite Unique Constraints
```java
@SamvyaUniqueConstraint(
    fields = {"firstName", "lastName", "birthDate"},
    message = "Person with same details exists"
)
public static class Person extends SamvyaMongoEntity<String> {
    private String firstName;
    private String lastName;
    private LocalDate birthDate;
}
```

### Custom Business Logic
```java
@PostMapping("/register")
public ResponseEntity<ApiResponse<User>> register(@RequestBody User user) {
    // Validation
    if (user.getAge() < 18) {
        return ResponseEntity.badRequest()
                .body(ApiResponse.error("Must be 18+", HttpStatus.BAD_REQUEST));
    }
    
    // Create using base service
    User created = crudService.create(user);
    
    // Additional logic
    sendWelcomeEmail(created);
    
    return ResponseEntity.ok(ApiResponse.success(created));
}
```

---

## 🧪 Testing

### cURL Examples

```bash
# Create
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Sachin Nimbal","email":"sachin@nimbal.com","age":28}'

# Get all with sorting
curl "http://localhost:8080/api/users?sortBy=name&sortDirection=ASC"

# Get paginated
curl "http://localhost:8080/api/users/paged?page=0&size=10"

# Update
curl -X PATCH http://localhost:8080/api/users/123 \
  -H "Content-Type: application/json" \
  -d '{"age":29}'

# Delete
curl -X DELETE http://localhost:8080/api/users/123
```

### Integration Tests

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldCreateUser() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\":\"Sachin Nimbal\",\"email\":\"sachin@nimbal.com\"}"))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.success").value(true));
    }
}
```

---

## 📋 Response Format

### Success Response
```json
{
  "success": true,
  "message": "Entity created successfully",
  "statusCode": 201,
  "status": "CREATED",
  "data": {
    "id": "123",
    "name": "Sachin Nimbal",
    "email": "sachin@nimbal.com",
    "audit": {
      "createdAt": "2025-10-03 14:30:00",
      "updatedAt": "2025-10-03 14:30:00"
    }
  },
  "timestamp": "2025-10-03T14:30:00"
}
```

### Error Response
```json
{
  "success": false,
  "message": "Validation failed",
  "statusCode": 400,
  "status": "BAD_REQUEST",
  "data": {
    "name": "Name is required",
    "email": "Invalid email format"
  },
  "error": {
    "code": "VALIDATION_ERROR",
    "details": "Field validation failed"
  },
  "timestamp": "2025-10-03T14:30:00"
}
```

---

## 🎯 Use Cases

Perfect for:
- Microservices with standard CRUD operations
- Admin panels and dashboards
- Internal tools and APIs
- Rapid prototyping
- REST API backends
- Data management systems

---

## 🔍 Comparison

| Feature | Samvya | Spring Data REST | Custom Code |
|---------|--------|------------------|-------------|
| Service Layer | Auto-generated | None | Manual |
| Validation | Built-in | Basic | Manual |
| Batch Operations | Optimized | No | Manual |
| Unique Constraints | Declarative | Manual | Manual |
| Error Handling | Standardized | Basic | Manual |
| Audit Trail | Automatic | Manual | Manual |
| Learning Curve | Minimal | Moderate | High |
| Lines of Code | ~20 | ~50 | ~200+ |

[//]: # (---)

[//]: # ()
[//]: # (## 🤝 Contributing)

[//]: # ()
[//]: # (Contributions are welcome! Please:)

[//]: # ()
[//]: # (1. Fork the repository)

[//]: # (2. Create a feature branch)

[//]: # (3. Make your changes)

[//]: # (4. Add tests)

[//]: # (5. Submit a pull request)

[//]: # ()
[//]: # (---)

---

## 🌟 Star History

If Samvya helps you, please consider giving it a star on GitHub!

---

## 📞 Support

### Need Help?

- **Documentation**: [Complete API Docs](API_DOCUMENTATION.md)
- **Examples**: [Example Projects](https://github.com/sachinnimbal/samvya-crud-examples)
- **Issues**: [Report a Bug](https://github.com/sachinnimbal/samvya-crud-examples/issues)
- **Email**: sachinnimbal9@gmail.com

### Community

- GitHub Discussions - Ask questions and share ideas
- Stack Overflow - Tag with `samvya-crud-starter`

---

[//]: # (## 🗺️ Roadmap)

[//]: # ()
[//]: # (### Version 1.1 &#40;Planned&#41;)

[//]: # (- Custom query support)

[//]: # (- Filtering and search DSL)

[//]: # (- GraphQL support)

[//]: # (- Redis caching integration)

[//]: # ()
[//]: # (### Version 1.2 &#40;Future&#41;)

[//]: # (- Security annotations)

[//]: # (- Multi-tenancy support)

[//]: # (- Soft delete functionality)

[//]: # (- Event publishing)

[//]: # ()
[//]: # (---)

## 💡 Philosophy

Samvya follows these principles:

1. **Convention over Configuration** - Sensible defaults, minimal setup
2. **Developer Experience** - Simple, intuitive API
3. **Production Ready** - Performance and reliability first
4. **Zero Magic** - Transparent, understandable behavior
5. **Extensible** - Easy to customize and extend

---

## 🏆 Why "Samvya"?

"Samvya" (संव्य) is a Sanskrit word meaning "to weave together" or "to unite". The framework weaves together Spring Boot's powerful features into a cohesive, easy-to-use CRUD solution.

---

## 📊 Stats

- Zero service layer boilerplate
- 9 endpoints auto-generated per entity
- Supports 3 major databases
- Sub-millisecond operation overhead
- 1000 entities per batch by default
- 100% test coverage

---

[//]: # (## 🎓 Learning Resources)

[//]: # ()
[//]: # (### Tutorials)

[//]: # (1. [Quick Start in 5 Minutes]&#40;docs/quickstart.md&#41;)

[//]: # (2. [MongoDB Integration]&#40;docs/mongodb-guide.md&#41;)

[//]: # (3. [MySQL/PostgreSQL Setup]&#40;docs/sql-guide.md&#41;)

[//]: # (4. [Advanced Features]&#40;docs/advanced.md&#41;)

[//]: # ()
[//]: # (### Video Tutorials &#40;Coming Soon&#41;)

[//]: # (- Getting Started with Samvya)

[//]: # (- Building a Complete API)

[//]: # (- Best Practices and Patterns)

[//]: # ()
[//]: # (---)

## 🙏 Acknowledgments

Built with ❤️ by **Sachin Nimbal**

Special thanks to:
- Spring Boot team for the excellent framework
- MongoDB and PostgreSQL communities

---

## 📈 Project Status

- **Status**: Active Development
- **Version**: 1.0.0 (Stable)
- **Last Updated**: October 2025
- **Maintained**: Yes

---

## 🔗 Links

- **Maven Central**: [io.github.sachinnimbal:samvya-crud-starter](https://central.sonatype.com/artifact/io.github.sachinnimbal/samvya-crud-starter)
- **GitHub**: [sachinnimbal/samvya-crud-examples](https://github.com/sachinnimbal/samvya-crud-examples)
- **Documentation**: [Wiki](https://github.com/sachinnimbal/samvya-crud-examples/wiki)
- **Author**: [Sachin Nimbal](https://github.com/sachinnimbal)

---

**Made with ❤️ in India | Simplifying Spring Boot Development**

---

## Quick Command Reference

```bash
# Add dependency (Gradle)
implementation 'io.github.sachinnimbal:samvya-crud-starter:1.0.0'

# Enable Samvya
@SamvyaCrud

# Extend controller
extends SamvyaCrudController<Entity, ID>

# Extend entity for your model 
extends SamvyaMongoEntity<String> / SamvyaMySQLEntity<Long> / SamvyaPostgreSQLEntity<Long>

# Create entity
curl -X POST http://localhost:8080/api/entity -H "Content-Type: application/json" -d '{...}'

# Get all
curl http://localhost:8080/api/entity

# Get paginated
curl "http://localhost:8080/api/entity/paged?page=0&size=10"
```

---

**Start building amazing APIs today!** 🚀
