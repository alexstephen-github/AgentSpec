This document provides a comprehensive guide for AI coding agents to build secure, scalable, and maintainable microservices using Java 21 and Spring Boot 3. Adherence to these principles is mandatory for all generated projects.

---

## 1. 🎯 Core Principles

-   **Security First**: Implement security controls at every layer (API gateway, service, data). Assume a zero-trust environment. Use least privilege principles.
-   **Production Ready**: All code must be suitable for production, with robust configuration, comprehensive logging, resilient error handling, and automated testing.
-   **Developer Experience**: Ensure the project is easy to set up, run, debug, and understand. Prioritize clear documentation, clean code, and fast feedback loops.
-   **Performance by Design**: Build for low latency and high throughput. Implement efficient data access patterns, caching strategies, and asynchronous processing where appropriate.
-   **Observability**: The system's internal state must be inferable from its external outputs (structured logs, granular metrics, distributed traces). Integrate with standard observability tools.
-   **Resilience**: Design for failure. Implement circuit breakers, retries, and bulkheads to prevent cascading failures.
-   **Statelessness**: REST APIs should be stateless, ensuring each request from a client to a server contains all the information needed to understand the request.
-   **Idempotency**: Operations (especially PUT, DELETE) should produce the same result regardless of how many times they are executed, enabling safe retries.

---

## 2. 🛠️ Standard Technology Stack

| Component            | Technology                             | Version/Standard                                | Notes                                                               |
| :------------------- | :------------------------------------- | :---------------------------------------------- | :------------------------------------------------------------------ |
| **Language**         | Java                                   | **21**                                          | LTS version.                                                        |
| **Framework**        | Spring Boot                            | **3.3+**                                        | Latest stable release, aligned with Spring Framework 6.x.           |
| **Build Tool**       | Maven                                  | 3.9+                                            | Standardized build tool.                                            |
| **Database**         | PostgreSQL (Prod), H2 (Dev/Test)       | Latest                                          | PostgreSQL for relational data in production, H2 for lightweight local/test. |
| **ORM**              | Spring Data JPA / Hibernate            | Included with Spring Boot 3.3+                  |                                                                     |
| **API Spec**         | OpenAPI                                | 3.0.x                                           | Using SpringDoc OpenAPI for documentation generation.               |
| **Container**        | Docker                                 | Latest                                          | For containerization of microservices.                              |
| **Caching**          | Caffeine (local), Redis (distributed)  | Latest                                          | Caffeine for in-memory caching, Redis for cross-instance/distributed. |
| **Validation**       | Jakarta Bean Validation                | 3.0                                             | Standard for declarative validation.                                |
| **Testing**          | JUnit 5, Mockito, AssertJ, Testcontainers | Latest                                          | Comprehensive testing toolkit. Testcontainers for integration tests. |
| **Monitoring**       | Micrometer, Prometheus, Grafana        | Latest                                          | For metrics collection and visualization.                           |
| **Tracing**          | Micrometer Tracing, OpenTelemetry      | Latest                                          | For distributed transaction tracing.                                |
| **Circuit Breaker**  | Resilience4j                           | Latest                                          | For fault tolerance and resilience patterns.                        |
| **Logging**          | SLF4J + Logback                        | Latest                                          | Structured logging output.                                          |
| **Serialization**    | Jackson                                | Included with Spring Boot                       | For JSON serialization/deserialization.                             |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using **Spring Initializr** (`start.spring.io`) with the following default dependencies:

-   **Web**: Spring Web (for RESTful APIs)
-   **Data**: Spring Data JPA, PostgreSQL Driver, H2 Database
-   **Observability**: Spring Boot Actuator, Micrometer Registry Prometheus
-   **Cache**: Spring Cache, Caffeine
-   **Validation**: Jakarta Bean Validation
-   **Developer Tools**: Lombok, Spring Boot DevTools (for development only)
-   **Documentation**: SpringDoc OpenAPI (`org.springdoc:springdoc-openapi-starter-webmvc-ui`)
-   **Resilience**: Resilience4j Spring Boot 3 (`io.github.resilience4j:resilience4j-spring-boot3`)

### Standard Package Structure

Organize all source code under a root package (e.g., `com.hcl.servicename`). The preferred approach is **Package by Feature**, with logical layering within each feature.

```
com/hcl/servicename/
├── Application.java               # Main Spring Boot application class (e.g., ParcelServiceApplication.java)
├── common/                        # Common utilities, base DTOs, shared converters, generic exceptions
│   ├── config/                    # Shared configurations (e.g., global web config)
│   ├── util/                      # Generic utility classes
│   └── validation/                # Custom validation annotations
├── config/                        # Application-specific @Configuration beans (e.g., CacheConfig, SecurityConfig, OpenApiConfig)
├── featureA/                      # Package for a specific business feature (e.g., 'parcel', 'user', 'order')
│   ├── controller/                # REST controllers for Feature A
│   │   └── FeatureAController.java
│   ├── dto/                       # Data Transfer Objects (Request/Response) for Feature A
│   │   ├── FeatureARequest.java
│   │   └── FeatureAResponse.java
│   ├── model/                     # JPA Entities for Feature A
│   │   └── FeatureAEntity.java
│   ├── repository/                # Spring Data JPA repositories for Feature A
│   │   └── FeatureARepository.java
│   ├── service/                   # Business logic for Feature A
│   │   ├── IFeatureAService.java  # Interface for Feature A service
│   │   └── FeatureAServiceImpl.java # Implementation
│   └── exception/                 # Feature-specific exceptions
├── featureB/                      # Another business feature package
│   ├── controller/
│   ├── dto/
│   ├── model/
│   ├── repository/
│   └── service/
└── exception/                     # Global exception handler, common application-wide custom exceptions
    ├── GlobalExceptionHandler.java
    └── ResourceNotFoundException.java

```

---

## 4. 📝 Development Workflow & Best Practices

### Step 1: API & DTO Design (Controller Layer)

1.  **Define DTOs**: Create separate Plain Old Java Objects (POJOs) for API requests and responses in the `featureX.dto` package. These should represent the contract of your API.
    *   Use Jakarta Bean Validation annotations (`@NotBlank`, `@Size`, `@Email`, `@Pattern`, `@Valid`, `@NotNull`) for all incoming data validation.
    *   **Never expose JPA entities directly in the API.**
    *   Use `@Schema` from `io.swagger.v3.oas.annotations.media` for OpenAPI schema details (description, examples).
2.  **Create Controller**: In the `featureX.controller` package, create a `@RestController`.
3.  **Map Endpoints**:
    *   Use `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping` for appropriate HTTP methods.
    *   **API Versioning**: Prefix all routes with `/api/v{version}`. Example: `@RequestMapping("/api/v1/parcels")`. Path versioning is preferred.
    *   Design endpoints to be resource-oriented and intuitive (e.g., `/api/v1/users/{userId}/orders`).
4.  **Input/Output**:
    *   Accept DTOs as `@RequestBody` (with `@Valid` for validation) or `@RequestParam`/`@PathVariable`.
    *   Return `ResponseEntity<Dto>` for explicit control over HTTP status and headers.
    *   For collection resources, implement pagination (e.g., `Page<Dto>`) and filtering capabilities.
5.  **HATEOAS (Optional but Recommended for REST maturity)**: Consider adding hypermedia controls using Spring HATEOAS to enrich responses with links to related resources.

### Step 2: Business Logic (Service Layer)

1.  **Create Service Interface & Implementation**: In the `featureX.service` package, define an interface (`IFeatureAService`) and its implementation (`FeatureAServiceImpl`). Use `@Service` on the implementation.
2.  **Inject Dependencies**: Use constructor injection for repositories, other services, and external clients. Avoid field injection.
3.  **Implement Logic**: Write the core business logic here. This layer is responsible for orchestrating data access, calling other internal/external services, enforcing business rules, and handling exceptions.
4.  **Use Transactions**:
    *   Apply `@Transactional` (from `org.springframework.transaction.annotation`) to service methods that modify data to ensure atomicity.
    *   Use `@Transactional(readOnly = true)` for read operations to optimize performance and prevent accidental writes.
    *   Configure transaction propagation as needed.
5.  **Map DTOs to Entities**: The service layer is responsible for converting incoming DTOs to JPA entities (and vice-versa) before passing them to the repository or returning them to the controller. Use MapStruct for complex mappings to reduce boilerplate.
6.  **Resilience**: Apply Resilience4j annotations (`@CircuitBreaker`, `@Retry`, `@RateLimiter`, `@TimeLimiter`) on service methods that interact with external systems or potentially unstable internal services.

### Step 3: Data Persistence (Repository & Model Layer)

1.  **Create Entity**: In the `featureX.model` package, define a JPA `@Entity`.
    *   Use `@Id`, `@GeneratedValue`, and appropriate `@Column` annotations for schema definition.
    *   Use `@Enumerated(EnumType.STRING)` for enums.
    *   Define relationships (`@OneToMany`, `@ManyToOne`, etc.) with careful consideration for fetching strategies (lazy vs. eager) and cascade types.
    *   **Avoid Lombok's `@Data` on entities; prefer `@Getter`, `@Setter`, `@ToString` (with caution to avoid infinite loops for relationships), `@NoArgsConstructor`, `@AllArgsConstructor` specifically.** Avoid `equals()` and `hashCode()` on entities unless manually implemented to use business keys or a constant value, especially when using proxies.
2.  **Create Repository**: In the `featureX.repository` package, create an interface that extends `JpaRepository<Entity, IdType>`.
    *   Add custom query methods following Spring Data JPA naming conventions (e.g., `findByStatusAndCreatedDateBetween`).
    *   For complex queries, use `@Query` annotations or consider QueryDSL.

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in the `exception` or `featureX.exception` package (e.g., `ResourceNotFoundException extends RuntimeException`, `InvalidInputException`).
2.  **Global Handler**: Create a `@RestControllerAdvice` class in the `exception` package (e.g., `GlobalExceptionHandler`).
3.  **Handle Exceptions**:
    *   Add `@ExceptionHandler` methods for custom exceptions, validation exceptions (`MethodArgumentNotValidException`), security exceptions, and a fallback generic `Exception`.
    *   Return a standardized `ProblemDetail` (Spring 6 standard) or a custom `ErrorResponse` DTO with a clear message, error code, and appropriate HTTP status.
    *   **Never expose internal stack traces or sensitive implementation details to the client.**
    *   Map exceptions to appropriate HTTP status codes (e.g., `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `500 Internal Server Error`).

### Step 5: Caching

1.  **Enable Caching**: Add `@EnableCaching` to the main application class.
2.  **Configure Cache**: Create a `CacheConfig` class in the `config` package to define cache managers (e.g., Caffeine for local, Redis for distributed).
    *   Provide explicit cache names.
    *   Configure cache eviction policies (TTL, TTI, size-based).
3.  **Apply Caching**: Use `@Cacheable`, `@CachePut`, `@CacheEvict` on service methods to cache results of expensive, frequently accessed, and relatively static operations.
    *   Define clear cache keys using SpEL.
    *   Ensure cache eviction strategies align with data freshness requirements.

### Step 6: Configuration

-   **Base Config (`application.yml` or `application.properties`)**: Use YAML for better readability and hierarchy. Define common properties like `spring.application.name`, server port.
-   **Profiles**:
    -   **Dev Profile (`application-dev.yml`)**: Configure H2 in-memory database, enable the H2 console, set logging to `DEBUG`.
    -   **Prod Profile (`application-prod.yml`)**: Configure PostgreSQL connection details. Use placeholders for secrets (e.g., `${DB_PASSWORD}`). Disable detailed health info (`management.endpoint.health.show-details=never`). Enable structured logging.
    -   **Test Profile (`application-test.yml`)**: Minimal configuration for integration tests, typically using an in-memory database or Testcontainers.
-   **Externalization**: Leverage Spring Cloud Config Server or environment variables for externalizing configuration in production environments.
-   **Strongly Typed Configuration**: Use `@ConfigurationProperties` to bind properties to type-safe configuration classes.

### Step 7: `catalog-info.yaml` File Structure

The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository. This integrates with Backstage or similar developer portals.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: # required, unique name for the microservice (e.g., parcel-service)
  description: # required, human-readable description of the microservice's purpose and functionality
  tags:
    - # list of tags for discoverability (e.g., 'java', 'springboot', 'rest', 'microservice', 'parcel-management')
  annotations:
    # key-value pairs for tool integrations (e.g., CI/CD pipelines, monitoring dashboards)
    github.com/project-slug: hcl-org/parcel-service # GitHub repository slug
    grafana/dashboard-selector: 'app=parcel-service' # Selector for Grafana dashboards
    sonarqube.org/project-key: 'hcl-org_parcel-service' # SonarQube project key
    backstage.io/techdocs-ref: dir:. # Reference to TechDocs documentation
  links:
    - url: https://hcl-org.atlassian.net/jira/software/projects/PARCEL/board # URL to Jira project board
      title: Jira Board
      icon: dashboard
    - url: https://jenkins.hcl-org.com/job/parcel-service-ci # URL to CI pipeline
      title: Jenkins CI Pipeline
      icon: ci
spec:
  type: service # required, e.g., 'service', 'library', 'website'
  lifecycle: production # required, e.g., 'experimental', 'development', 'production', 'deprecated'
  owner: hcl-parcel-team # required, the owning team or user in Backstage
  system: hcl-logistics # optional, the larger system this component belongs to (e.g., 'Logistics-Platform')
  dependsOn:
    - resource:default/hcl-parcel-db # Example: Dependency on a PostgreSQL database resource
    - component:default/user-service # Example: Dependency on another microservice
  providesApis:
    - parcel-api # API this component exposes
```

---

### Step 8: Coding standards

## Spring Boot Coding Standards

These standards apply to all Spring Boot applications developed within our organization.

### 1. Project Setup
-   **Spring Initializr:** Always use Spring Initializr to bootstrap new projects.
-   **Dependencies:** Only include necessary dependencies. Regularly review and remove unused ones.
-   **Build Tool:** Standardize on Maven. Ensure `pom.xml` is well-structured and uses property versions for consistency.

### 2. Code Structure
-   **Package by Feature:** Organize packages around business features rather than technical layers (e.g., `com.example.app.user` instead of `com.example.app.controller`).
-   **Layered Architecture:** Adhere to a strict layered architecture (Controller -> Service -> Repository).
    -   **Controller:** Only handle HTTP request/response mapping, input validation. Avoid business logic.
    -   **Service:** Encapsulate business logic, orchestrate calls to repositories and other services.
    -   **Repository:** Handle data persistence operations.
    -   **DTOs:** Use Data Transfer Objects for API input/output. Never expose entities directly.
-   **Configuration:** Place all Spring configuration classes under a `config` package.
-   **Exceptions:** Custom exceptions and global exception handlers should reside in an `exception` package.

### 3. Naming Conventions
-   **Classes:** PascalCase (e.g., `UserService`, `OrderController`).
-   **Methods:** camelCase (e.g., `getUserById`, `processOrder`).
-   **Variables:** camelCase (e.g., `userName`, `orderId`).
-   **Constants:** SCREAMING_SNAKE_CASE (e.g., `DEFAULT_PAGE_SIZE`).
-   **Interfaces:** Prefix with `I` is discouraged (e.g., `UserRepository` instead of `IUserRepository`).
-   **Repositories:** Follow Spring Data JPA conventions (e.g., `UserRepository`).

### 4. API Design (RESTful Principles)
-   **Resource-Oriented:** Design APIs around resources (e.g., `/users`, `/products`).
-   **HTTP Methods:** Use appropriate HTTP methods for CRUD operations (GET for retrieve, POST for create, PUT/PATCH for update, DELETE for delete).
-   **Versioning:** Implement API versioning (e.g., `/api/v1/users`). Path versioning is preferred.
-   **Pagination & Filtering:** Support pagination (`page`, `size`) and filtering (`?status=active`).
-   **Consistent Responses:**
    -   **Success:** Return appropriate HTTP status codes (200 OK, 201 Created, 204 No Content).
    -   **Error:** Return standardized JSON error objects with a clear message and appropriate HTTP status (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error). Never expose stack traces.

### 5. Error Handling
-   **Global Exception Handling:** Use `@ControllerAdvice` to centralize exception handling.
-   **Custom Exceptions:** Create specific custom exceptions for business-level errors (e.g., `ResourceNotFoundException`, `InvalidInputException`).
-   **Meaningful Messages:** Provide clear, user-friendly error messages.

### 6. Logging
-   **SLF4J + Logback:** Standardize on SLF4J as the logging facade and Logback as the implementation.
-   **Logging Levels:** Use appropriate logging levels (TRACE, DEBUG, INFO, WARN, ERROR).
-   **Structured Logging:** Prefer structured logging (JSON format) for easier parsing and analysis.
-   **Sensitive Data:** Never log sensitive information (passwords, PII).

### 7. Security
-   **Input Validation:** Perform rigorous input validation at the API boundary using Jakarta Bean Validation.
-   **Authentication & Authorization:** Integrate Spring Security for robust authentication and authorization.
-   **CORS:** Properly configure Cross-Origin Resource Sharing.
-   **Dependency Vulnerabilities:** Regularly scan for known vulnerabilities using tools like Snyk or OWASP Dependency-Check.
-   **Secrets Management:** Use environment variables or a dedicated secrets management solution (e.g., Vault, AWS Secrets Manager) for sensitive configurations. Avoid hardcoding.

### 8. Testing
-   **Unit Tests:** Write comprehensive unit tests for business logic in service classes, mocking external dependencies.
-   **Integration Tests:** Use `@SpringBootTest` and Testcontainers for integration tests with actual database instances.
-   **Controller Tests:** Use `@WebMvcTest` for testing controllers in isolation, mocking service layer.
-   **Test Coverage:** Aim for a minimum of 80% line coverage for business logic.

### 9. Performance
-   **Database Optimization:** Use efficient queries, proper indexing.
-   **Caching:** Implement caching (e.g., Caffeine, Redis) for frequently accessed, slow-changing data.
-   **Asynchronous Operations:** Use `@Async` or message queues for non-blocking operations where appropriate.

### 10. Documentation
-   **OpenAPI/Swagger:** Document all REST APIs using SpringDoc OpenAPI.
-   **JavaDoc:** Add JavaDoc comments for all public classes, methods, and complex logic.
-   **README.md:** Provide a clear `README.md` with setup instructions, API endpoints, and deployment guidance.

### 11. Code Quality
-   **Lombok:** Use Lombok annotations judiciously (`@Getter`, `@Setter`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@Builder`, `@Slf4j`). Avoid `@Data` on JPA entities.
-   **Immutability:** Favor immutable objects where possible.
-   **Code Linting:** Use SonarQube or similar tools to enforce code quality rules and identify potential issues.
-   **Duplication:** Avoid code duplication (DRY principle).
-   **Code Reviews:** All code must go through a peer review process.

---

**Note on other Coding Standards:**
The provided links for Python (`https://www.projectrules.ai/rules/python_standards`) and Angular (`https://www.projectrules.ai/rules/angular`) define coding standards for different technology stacks. While these are crucial for projects using those languages/frameworks, they are not directly applicable to *this* Java Spring Boot microservice specification. Separate agent protocols or cross-language coding standards documents should reference those.

---

### Step 9: Documentation

-   **OpenAPI (SpringDoc)**:
    *   Configure a `OpenApiConfig` bean in the `config` package to set the API title, version, description, and contact information.
    *   Use `@Tag`, `@Operation`, `@ApiResponse`, `@Parameter`, `@Schema` annotations (from `io.swagger.v3.oas.annotations`) on controllers, methods, and DTOs to provide rich, explicit documentation for the API contract.
    *   Generate `openapi.json` at build time.
-   **JavaDoc**: Add comprehensive JavaDoc comments to all public classes, interfaces, methods, and complex fields in controllers, services, and DTOs. Explain their purpose, parameters, return values, and any pre/post conditions or exceptions.
-   **README.md**: Create a detailed `README.md` file at the project root covering:
    *   Project overview and purpose.
    *   Prerequisites (Java, Maven, Docker).
    *   Build and run instructions (local, Docker).
    *   Available API endpoints (linking to Swagger UI).
    *   Key configuration properties.
    *   Testing instructions.
    *   Deployment notes.

### Step 10: Testing

-   **Unit Tests (`src/test/java`)**:
    *   Focus on isolated testing of business logic in service classes and utility methods.
    *   Use Mockito for mocking dependencies (repositories, other services, external clients).
    *   Aim for high code coverage (minimum 80% for service layer).
-   **Integration Tests (`src/test/java`)**:
    *   **Controller Layer**: Use `@WebMvcTest` to test controllers in isolation, focusing on request mapping, validation, and JSON serialization/deserialization, mocking the service layer.
    *   **Data Layer**: Use `@DataJpaTest` to test JPA repositories and entity mappings against an in-memory database or Testcontainers.
    *   **Full Application Flow**: Use `@SpringBootTest` (with `webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT`) and `TestRestTemplate` or `WebTestClient` to test the full application context, from controller to database.
    *   **Testcontainers**: Employ Testcontainers for spinning up real database instances (e.g., PostgreSQL), message queues (e.g., Kafka), or other external services for integration tests, ensuring realistic test environments. Use `@Testcontainers` and `@Container`.
-   **Property-Based Testing (Optional)**: For complex business rules, consider using libraries like jqwik or Hypothesis to generate test cases.
-   **Chaos Engineering (Optional)**: For production environments, consider simulating failures using tools like Chaos Monkey or Kubernetes chaos experiments.

---

## 5. 🐳 Containerization & Deployment

### Dockerfile

Create a multi-stage `Dockerfile` in the project root:

1.  **Build Stage**:
    *   Use a robust `maven` base image (e.g., `maven:3.9-eclipse-temurin-21-alpine`) to compile and package the application into a JAR.
    *   Copy `pom.xml` and source code.
    *   Perform `mvn clean install -DskipTests`.
2.  **Runtime Stage**:
    *   Use a minimal JRE image (e.g., `eclipse-temurin:21-jre-alpine` or even better, a `distroless` image like `gcr.io/distroless/java21-debian12`) to reduce image size and attack surface.
    *   **Security**: Create a dedicated non-root user and group (`addgroup --gid 1001 appgroup && adduser --uid 1001 --ingroup appgroup --home /app --shell /sbin/nologin appuser`). Switch to this user (`USER appuser`).
    *   Copy the built JAR from the build stage.
    *   **Health Check**: Add a `HEALTHCHECK` instruction that calls the `/actuator/health` endpoint (e.g., `HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 CMD curl --fail http://localhost:8080/actuator/health || exit 1`).
    *   **Execution**: Use `ENTRYPOINT ["java", "-jar", "/app/app.jar"]`. Pass JVM arguments (e.g., `-Xmx512m`) via environment variables or directly in `ENTRYPOINT`.

### Environment Variables

All sensitive secrets (database passwords, API keys, external service credentials) **must** be supplied via environment variables at runtime, not hardcoded in properties files or included in the Docker image.

-   Adhere to 12-Factor App principles.
-   In Kubernetes environments, use `ConfigMaps` for non-sensitive configuration and `Secrets` for sensitive data, which are then exposed as environment variables or mounted files.

---

This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.

---

### Sample Agent.md

Platform engineer agent can create this.

Here the details and the best practices we need to give link to appropriate confluence, team or github link so that it will be more realistic.
Then the consumer while building a microservice, he can download this agent.md file from IDP and use his specific development standards and best practices.