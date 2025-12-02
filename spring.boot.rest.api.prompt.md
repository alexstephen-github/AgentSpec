---
agent: agent
model: Claude Sonnet 4.5 (copilot)
name: spring.boot.rest.api
description: The template file to generate the spec file
---
 
## 1. 🎯 Core Principles
 
- **Security First**: Implement security controls at every layer. Assume a zero-trust environment.

- **Production Ready**: All code must be suitable for production, with robust configuration, logging, and error handling.

- **Developer Experience**: Ensure the project is easy to set up, run, and understand. Prioritize clear documentation and clean code.

- **Performance by Design**: Build for low latency and high throughput. Implement caching and efficient data access patterns.

- **Observability**: The system's internal state must be inferable from its external outputs (logs, metrics, traces).
 
---
 
## 2. 🛠️ Standard Technology Stack
 
| Component | Technology | Version/Standard |
|---|---|---|
| **Language** | Java | **21** |
| **Framework** | Spring Boot | **3.3+** |
| **Build Tool** | Maven | 3.9+ |
| **Database** | PostgreSQL (Prod), H2 (Dev/Test) | Latest |
| **API Spec** | OpenAPI | 3.0 |
| **Container** | Docker | Latest |
| **Caching** | Caffeine (local), Redis (distributed) | Latest |
| **Validation** | Jakarta Bean Validation | 3.0 |
| **Testing** | JUnit 5, Mockito | Latest |
 
---
 
## 3. 🏗️ Project Initialization & Structure
 
### Initialization

Generate new projects using **Spring Initializr** with the following default dependencies:

- Spring Web

- Spring Data JPA

- Spring Boot Actuator

- Spring Cache

- Jakarta Bean Validation

- Lombok

- PostgreSQL Driver

- H2 Database

- SpringDoc OpenAPI (`org.springdoc:springdoc-openapi-starter-webmvc-ui`)
 
### Standard Package Structure

Organize all source code under a root package (e.g., `com.hcl.servicename`).
 
```
com/hcl/servicename/
├── ParcelServiceApplication.java   # Main application class
├── config/                         # @Configuration beans (Cache, Security, OpenAPI)
├── controller/                     # REST controllers (API layer)
├── dto/                            # Data Transfer Objects for API requests/responses
├── model/                          # JPA Entities
├── repository/                     # Spring Data JPA repositories
├── service/                        # Business logic
└── exception/                      # Custom exceptions and global handler
```
 
---
 
## 4. 📝 Development Workflow & Best Practices
 
### Step 1: API & DTO Design (Controller Layer)

1.  **Define DTOs**: Create separate Plain Old Java Objects (POJOs) for API requests and responses in the `dto` package. Use Jakarta Bean Validation annotations (`@NotBlank`, `@Size`, `@Email`, etc.) for all incoming data. **Never expose JPA entities directly in the API.**

2.  **Create Controller**: In the `controller` package, create a `@RestController`.

3.  **Map Endpoints**: Use `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`.

4.  **API Versioning**: Prefix all routes with `/api/v1`. Example: `@RequestMapping("/api/v1/parcels")`.

5.  **Input/Output**: Accept DTOs as `@RequestBody` and return `ResponseEntity<Dto>`.
 
### Step 2: Business Logic (Service Layer)

1.  **Create Service**: In the `service` package, create a `@Service` class.

2.  **Inject Dependencies**: Use constructor injection for repositories and other services.

3.  **Implement Logic**: Write the core business logic here. This layer is responsible for coordinating data access, calling other services, and enforcing business rules.

4.  **Use Transactions**: Use `@Transactional` for methods that modify data. Use `@Transactional(readOnly = true)` for read operations to optimize performance.

5.  **Map DTOs to Entities**: The service layer is responsible for converting incoming DTOs to JPA entities before passing them to the repository, and converting entities to DTOs before returning them to the controller.
 
### Step 3: Data Persistence (Repository & Model Layer)

1.  **Create Entity**: In the `model` package, define a JPA `@Entity`. Use `@Id`, `@GeneratedValue`, and appropriate `@Column` annotations. Avoid Lombok's `@Data` on entities; prefer `@Getter`, `@Setter`, `@ToString`.

2.  **Create Repository**: In the `repository` package, create an interface that extends `JpaRepository<Entity, IdType>`. Add custom query methods as needed.
 
### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in the `exception` package (e.g., `ResourceNotFoundException extends RuntimeException`).

2.  **Global Handler**: Create a `@RestControllerAdvice` class in the `exception` package.

3.  **Handle Exceptions**: Add `@ExceptionHandler` methods for custom exceptions, validation exceptions (`MethodArgumentNotValidException`), and a fallback generic `Exception`. Return a standardized `ErrorResponse` DTO with a message and HTTP status. **Never expose stack traces.**
 
### Step 5: Caching

1.  **Enable Caching**: Add `@EnableCaching` to the main application class.

2.  **Configure Cache**: Create a `CacheConfig` class to define cache managers (e.g., Caffeine for local, Redis for distributed).

3.  **Apply Caching**: Use `@Cacheable`, `@CachePut`, and `@CacheEvict` on service methods to cache results of expensive operations.
 
### Step 6: Configuration

- **Base Config (`application.properties`)**: Define common properties like `spring.application.name`.

- **Dev Profile (`application-dev.properties`)**: Configure H2 in-memory database, enable the H2 console, and set logging to `DEBUG`.

- **Prod Profile (`application-prod.properties`)**: Configure PostgreSQL connection details. Use placeholders for secrets (e.g., `${DB_PASSWORD}`). Disable detailed health info (`management.endpoint.health.show-details=never`).

### Step 7: `catalog-info.yaml` File Structure
 
The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository.
 
```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: # required, unique name
  description: # required, human-readable description
  tags:
    - # list of tags for discoverability
  annotations:
    # key-value pairs for tool integrations
  links:
    - url: # URL to a relevant resource
      title: # Display title for the link
      icon: # Optional: e.g., 'dashboard'
spec:
  type: # required, e.g., 'service', 'library'
  lifecycle: # required, e.g., 'experimental', 'production'
  owner: # required, the owning team or user
  system: # optional, the larger system this component belongs to
  dependsOn:
    - # list of components/resources this component depends on
  providesApis:
    - # list of APIs this component provides
```
 
---
 ### Step 8: #Coding standards#
 
1.  **General Coding Practices:**
    *   **Clarity and Readability:** Code should be self-documenting. Use meaningful names for variables, methods, and classes.
    *   **DRY (Don't Repeat Yourself):** Avoid code duplication. Extract common logic into reusable methods or classes.
    *   **KISS (Keep It Simple, Stupid):** Prefer simple solutions over complex ones. Avoid over-engineering.
    *   **Single Responsibility Principle (SRP):** Each class or method should have one clear responsibility.
    *   **SOLID Principles:** Adhere to SOLID principles (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion).
    *   **Comments:** Use comments sparingly, primarily for explaining *why* something is done, not *what* is done (which should be clear from the code itself). Javadoc should be used for public APIs.
    *   **Error Handling:** Implement robust error handling. Catch specific exceptions, not just generic `Exception`. Provide meaningful error messages.
    *   **Logging:** Use a consistent logging framework (e.g., SLF4J with Logback). Log at appropriate levels (DEBUG, INFO, WARN, ERROR). Avoid logging sensitive information.
    *   **Immutability:** Favor immutable objects where possible, especially for DTOs and configuration objects.
    *   **Null Safety:** Minimize `null` usage. Use `Optional` for potentially absent values where appropriate, but don't overuse it.
    *   **Resource Management:** Ensure resources (e.g., database connections, file streams) are properly closed, typically using try-with-resources.

2.  **Java Specific Guidelines:**
    *   **Naming Conventions:**
        *   Packages: `lowercase.separated.by.dots`
        *   Classes/Interfaces/Enums: `PascalCase`
        *   Methods: `camelCase`
        *   Variables: `camelCase`
        *   Constants (static final): `UPPER_SNAKE_CASE`
    *   **Whitespace:**
        *   Use 4 spaces for indentation.
        *   Place a space before and after binary operators (`=`, `+`, `-`, `*`, `/`, etc.).
        *   Place a space after commas in argument lists.
        *   Place a space after keywords like `if`, `for`, `while`, `catch`.
    *   **Braces:** Use K&R style (opening brace on the same line as the statement, closing brace on its own line).
    *   **Line Length:** Aim for a maximum of 120 characters per line.
    *   **Imports:** Organize imports. Avoid wildcard imports (`import com.example.*`).
    *   **Annotations:** Place annotations on separate lines above the element they annotate.
    *   **Generics:** Use generics to improve type safety and avoid raw types.
    *   **Streams API:** Use Java 8 Streams API for collections processing when it improves readability.
    *   **Encapsulation:** Use access modifiers (`private`, `protected`, `public`) appropriately. Expose only what is necessary.
    *   **`equals()` and `hashCode()`:** Override both or neither. Consider using Lombok's `@EqualsAndHashCode`.
    *   **`toString()`:** Provide a meaningful `toString()` implementation for debugging, but avoid including sensitive data.
    *   **Enums:** Use enums for fixed sets of constants.

3.  **Spring Boot Specific Guidelines:**
    *   **Dependency Injection:** Prefer constructor injection for mandatory dependencies. Field injection (using `@Autowired` directly on fields) is generally discouraged as it makes testing harder and hides dependencies.
    *   **Configuration:**
        *   Use `@Configuration` classes for application-wide bean definitions.
        *   Externalize configuration using `application.properties` or `application.yaml`.
        *   Use Spring profiles (`@Profile`) for environment-specific configurations.
        *   Store sensitive information in environment variables or a secure vault (e.g., HashiCorp Vault), not directly in property files.
    *   **REST Controllers:**
        *   Use `@RestController` (combines `@Controller` and `@ResponseBody`).
        *   Map specific HTTP methods using `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`.
        *   Return `ResponseEntity` for more control over HTTP status and headers.
        *   Handle exceptions using `@ControllerAdvice` and `@ExceptionHandler` for global exception handling.
        *   Validate incoming requests using Jakarta Bean Validation (`@Valid`, `@Validated`).
        *   Use DTOs (Data Transfer Objects) for request and response bodies. Never expose JPA entities directly through the API.
    *   **Services:**
        *   Annotate business logic classes with `@Service`.
        *   Methods that modify data should be `@Transactional`. Read-only operations can benefit from `@Transactional(readOnly = true)`.
        *   Services should orchestrate calls to repositories and other services, performing business logic.
    *   **Repositories:**
        *   Extend `JpaRepository` (or `CrudRepository`, `PagingAndSortingRepository`) for basic CRUD operations.
        *   Define custom query methods following Spring Data JPA naming conventions or using `@Query` annotations.
    *   **Entities:**
        *   Annotate JPA entities with `@Entity`.
        *   Use `@Table` for table name if different from class name, `@Column` for column names.
        *   Primary keys should be annotated with `@Id` and `@GeneratedValue`.
        *   Avoid Lombok's `@Data` on JPA entities due to potential issues with `equals`/`hashCode` and lazy loading. Prefer `@Getter`, `@Setter`, `@NoArgsConstructor`, `@AllArgsConstructor` (if needed), and explicit `toString`, `equals`, `hashCode` or `@ToString.Exclude` for collections.
    *   **Testing:**
        *   Write unit tests for services and components by mocking dependencies.
        *   Write integration tests using `@SpringBootTest` to test the full application context.
        *   Use `@WebMvcTest` for testing controllers in isolation.
        *   Use Testcontainers for integration tests involving databases or other external services.
        *   Strive for good test coverage, focusing on critical paths and edge cases.
    *   **Actuator:** Use Spring Boot Actuator for monitoring and managing the application. Secure actuator endpoints in production.
    *   **Caching:** Use Spring's caching annotations (`@Cacheable`, `@CachePut`, `@CacheEvict`) for optimizing expensive operations. Configure a suitable cache manager (e.g., Caffeine, Redis).
    *   **Asynchronous Operations:** Use `@Async` for methods that can run in the background without blocking the caller. Ensure proper thread pool configuration.
    *   **Scheduled Tasks:** Use `@Scheduled` for simple recurring tasks.
    *   **Security:**
        *   Use Spring Security for authentication and authorization.
        *   Implement appropriate security headers (CSP, XSS, etc.).
        *   Secure API endpoints (e.g., using JWT, OAuth2).
        *   Sanitize all user inputs to prevent injection attacks.
        *   Encrypt sensitive data at rest and in transit.

4.  **API Design Guidelines (RESTful Principles):**
    *   **Resource-Oriented:** Design APIs around resources (nouns), not actions (verbs). E.g., `/users` instead of `/getUsers`.
    *   **HTTP Methods:** Use appropriate HTTP methods for actions:
        *   `GET`: Retrieve resources (idempotent, safe).
        *   `POST`: Create new resources, or perform non-idempotent operations.
        *   `PUT`: Update/replace a resource (idempotent).
        *   `PATCH`: Partially update a resource (non-idempotent).
        *   `DELETE`: Remove a resource (idempotent).
    *   **Status Codes:** Return appropriate HTTP status codes (2xx for success, 4xx for client errors, 5xx for server errors).
    *   **Versioning:** Version your API (e.g., `/api/v1/resources`).
    *   **Pagination, Filtering, Sorting:** Support these features for collection resources using query parameters.
    *   **Hypermedia (HATEOAS):** Consider HATEOAS for discoverability, providing links to related resources.
    *   **Idempotency:** Design operations to be idempotent where applicable (e.g., PUT, DELETE).
    *   **Security:** Implement authentication, authorization, and rate limiting.

5.  **Build Tool (Maven) Guidelines:**
    *   **`pom.xml` Organization:** Keep the `pom.xml` clean and well-organized.
    *   **Dependency Management:** Use Spring Boot's parent POM for dependency version management.
    *   **Plugins:** Use necessary plugins (e.g., `spring-boot-maven-plugin`, `maven-compiler-plugin`).
    *   **Profiles:** Use Maven profiles for different build environments (e.g., `dev`, `prod`, `test`).
    *   **Reproducible Builds:** Ensure builds are reproducible.

6.  **Git and Version Control:**
    *   **Branching Strategy:** Use a clear branching strategy (e.g., GitFlow, GitHub Flow).
    *   **Commit Messages:** Write clear, concise, and descriptive commit messages (e.g., Conventional Commits).
    *   **Pull Requests/Merge Requests:** Use PRs/MRs for code review.
    *   **Code Review:** Ensure code reviews are thorough and constructive.
 
### Step 9: Documentation

- **OpenAPI**: Configure a `OpenApiConfig` bean to set the API title, version, and description.

- **JavaDoc**: Add JavaDoc comments to all public methods in controllers and services, explaining their purpose, parameters, and return values.
 
### Step 10: Testing

- **Unit Tests**: Test services and controllers by mocking their dependencies (e.g., repositories). Place tests in `src/test/java`.

- **Integration Tests**: Use `@SpringBootTest` and `@ActiveProfiles("test")` to test the full application flow, from controller to database. Use Testcontainers for database integration tests.

- **Coverage**: Aim for a minimum of 80% test coverage.
 
---
 
## 5. 🐳 Containerization & Deployment
 
### Dockerfile

Create a multi-stage `Dockerfile`:

1.  **Build Stage**: Use a `maven` base image to build the application JAR.

2.  **Runtime Stage**: Use a minimal JRE image (e.g., `eclipse-temurin:21-jre-alpine`).

3.  **Security**: Create and run as a non-root user.

4.  **Health Check**: Add a `HEALTHCHECK` instruction that calls the `/actuator/health` endpoint.

5.  **Execution**: Use `ENTRYPOINT ["java", "-jar", "app.jar"]`.
 
### Environment Variables

All secrets (database passwords, API keys) **must** be supplied via environment variables, not hardcoded in properties files.
 
---
 
This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.
 
Sample Agent.md
 
Platform engineer agent can create this..  
 
Here the details and the best practices we need to give link to appropriate confluence, team or github link so that it will be more realistic 
 
then the consumer while building a microservice, he can download this agent.md file from IDP and use his specific development

---

### GitHub Copilot Prompt File Template

```
/create-spring-boot-rest-api
"""
Generate a Spring Boot 3.3+ REST API project structure and essential files for a service that manages '{resource_name}' resources.

Follow these specifications:
- **Project Structure**: Use standard package structure as defined in the 'spring.boot.rest.api' spec file, including `controller`, `service`, `repository`, `model`, `dto`, `config`, `exception`.
- **Resource**: The primary resource is '{resource_name}' with fields: {resource_fields_with_types}.
- **CRUD Operations**: Implement basic CRUD (Create, Read, Update, Delete) endpoints for '{resource_name}'.
- **DTOs**: Create separate Request and Response DTOs for '{resource_name}'.
- **Validation**: Apply Jakarta Bean Validation annotations to DTOs for common constraints (e.g., @NotBlank, @Size).
- **Persistence**: Use Spring Data JPA for H2 database (for development/testing) and PostgreSQL (for production readiness). Create an appropriate JPA entity.
- **Error Handling**: Include a basic `@RestControllerAdvice` for global exception handling, catching `ResourceNotFoundException` and `MethodArgumentNotValidException`.
- **OpenAPI**: Include SpringDoc OpenAPI dependency and a basic configuration.
- **Build Tool**: Use Maven.
- **Lombok**: Include Lombok dependency.

Provide the following files:
1.  `pom.xml`
2.  `SpringApplication.java` (main class)
3.  `{ResourceName}Controller.java`
4.  `{ResourceName}Service.java`
5.  `{ResourceName}Repository.java`
6.  `{ResourceName}Model.java` (JPA Entity)
7.  `{ResourceName}RequestDTO.java`
8.  `{ResourceName}ResponseDTO.java`
9.  `GlobalExceptionHandler.java`
10. `ResourceNotFoundException.java`
11. `OpenApiConfig.java`
12. `application.properties` (base config)
13. `application-dev.properties` (H2 config)
14. `application-prod.properties` (PostgreSQL config placeholders)
15. `Dockerfile` (multi-stage)
16. `catalog-info.yaml` (template with placeholders)

Replace `{ResourceName}` with the capitalized singular form of '{resource_name}'.
Replace `{resource_name}` with the lowercase singular form.
Replace `{resource_fields_with_types}` with the actual field definitions.

Example usage: `/create-spring-boot-rest-api resource_name=Product resource_fields_with_types="id: UUID, name: String, description: String, price: BigDecimal"`
"""
```