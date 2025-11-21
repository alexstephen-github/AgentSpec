Here's the detailed `Agent.md` spec file for building enterprise-grade microservices, with the requested coding standards integrated and other sections enhanced.

```markdown
# 🤖 Agent Protocol: Building Enterprise-Grade Microservices
 
This document provides a comprehensive guide for AI coding agents to build secure, scalable, and maintainable microservices using various modern technologies, with a primary focus on Java 21 and Spring Boot 3. Adherence to these principles is mandatory for all generated projects.
 
---
 
## 1. 🎯 Core Principles
 
-   **Security First**: Implement security controls at every layer (transport, application, data). Assume a zero-trust environment, enforce least privilege, and integrate with enterprise identity solutions (e.g., OAuth2, OpenID Connect). Conduct regular security reviews and static/dynamic analysis.
    *   *Relevant Link*: [HCL Security Standards](https://confluence.hcl.com/display/SEC/Security+Standards) (Placeholder for your organization's security standards)

-   **Production Ready**: All code must be suitable for production environments from day one. This includes robust configuration management, structured and actionable logging, comprehensive error handling with appropriate alerts, and automated deployment pipelines. Performance and resilience must be baked in, not bolted on.
    *   *Relevant Link*: [HCL Production Readiness Checklist](https://confluence.hcl.com/display/OPS/Production+Readiness+Checklist) (Placeholder for your organization's production readiness guide)

-   **Developer Experience**: Ensure the project is easy to set up, run, debug, test, and understand. Prioritize clear, concise documentation (README, API docs, architectural decision records), clean code following established patterns, and fast feedback loops (e.g., rapid local development, quick CI/CD).
    *   *Relevant Link*: [HCL Developer Onboarding Guide](https://confluence.hcl.com/display/DEV/Developer+Onboarding+Guide) (Placeholder for your organization's developer onboarding guide)

-   **Performance by Design**: Build for low latency and high throughput. Employ efficient algorithms and data structures. Implement caching strategies (local and distributed) where appropriate, optimize database queries, and consider asynchronous processing for long-running tasks. Profile and benchmark critical paths.
    *   *Relevant Link*: [HCL Performance Engineering Guidelines](https://confluence.hcl.com/display/PERF/Performance+Engineering+Guidelines) (Placeholder for your organization's performance guidelines)

-   **Observability**: The system's internal state must be inferable from its external outputs. Implement comprehensive logging (structured JSON preferred), metrics (Micrometer, Prometheus), and distributed tracing (OpenTelemetry, Zipkin) to enable effective monitoring, alerting, and troubleshooting in production.
    *   *Relevant Link*: [HCL Observability Playbook](https://confluence.hcl.com/display/OBS/Observability+Playbook) (Placeholder for your organization's observability playbook)

-   **Resilience**: Design for failure. Implement circuit breakers, retries with backoff, bulkheads, and graceful degradation strategies to ensure the service remains available and responsive even under stress or partial failure of dependent services.
    *   *Relevant Link*: [HCL Resilience Patterns](https://confluence.hcl.com/display/ARCH/Resilience+Patterns) (Placeholder for your organization's resilience patterns)
 
---
 
## 2. 🛠️ Standard Technology Stack
 
| Component | Technology | Version/Standard | Justification |
|---|---|---|---|
| **Language** | Java | **21** | Latest LTS with modern language features (Records, Pattern Matching, Virtual Threads previews for future). |
| **Framework** | Spring Boot | **3.3+** | Industry standard for microservices, rapid development, convention over configuration, extensive ecosystem. |
| **Build Tool** | Maven | 3.9+ | Widely adopted, robust dependency management, comprehensive plugin ecosystem. |
| **Database** | PostgreSQL (Prod), H2 (Dev/Test) | Latest | PostgreSQL: Robust, open-source, ACID compliant, excellent community support. H2: Lightweight in-memory for fast local development and testing. |
| **API Spec** | OpenAPI | 3.0 | Standardized machine-readable API documentation, client generation, and validation. |
| **Container** | Docker | Latest | Industry standard for containerization, ensuring consistent deployment environments. |
| **Caching** | Caffeine (local), Redis (distributed) | Latest | Caffeine: High-performance, in-memory cache for single-instance applications. Redis: Distributed, high-speed key-value store for shared caching across instances. |
| **Validation** | Jakarta Bean Validation | 3.0 | Standardized annotation-based validation for Java beans. |
| **Testing** | JUnit 5, Mockito, AssertJ, Testcontainers | Latest | JUnit 5: Modern testing framework. Mockito: Powerful mocking library. AssertJ: Fluent assertions. Testcontainers: Ephemeral database/service containers for integration tests. |
| **Monitoring/Tracing** | Micrometer, Spring Boot Actuator, OpenTelemetry | Latest | Micrometer: Vendor-neutral metrics facade. Actuator: In-built production-ready features. OpenTelemetry: Standard for distributed tracing. |
| **Security** | Spring Security | 6.x | Comprehensive security framework for Spring applications. |
| **Linting/Formatting** | Spotless, Checkstyle | Latest | Automated code style enforcement and static analysis. |

---
 
## 3. 🏗️ Project Initialization & Structure
 
### Initialization

Generate new projects using **Spring Initializr** (`start.spring.io`) with the following default dependencies. Ensure the selected Java version is 21 and Spring Boot version is 3.3+.

-   **Web**: Spring Web
-   **Data**: Spring Data JPA, PostgreSQL Driver, H2 Database
-   **Observability**: Spring Boot Actuator, Springdoc OpenAPI (for `/v3/api-docs` and Swagger UI)
    *   Add `org.springdoc:springdoc-openapi-starter-webmvc-ui` explicitly in `pom.xml`.
-   **Cache**: Spring Cache, Caffeine
-   **Validation**: Jakarta Bean Validation
-   **Utilities**: Lombok (Optional, but widely used. If used, ensure proper IDE plugin configuration and awareness of potential pitfalls with entities).
-   **Security**: Spring Security (if authentication/authorization is required)
 
### Standard Package Structure

Organize all source code under a root package (e.g., `com.hcl.servicename`, where `servicename` is replaced by the actual service name in kebab-case, e.g., `parcel-service`).

```
src/main/java/com/hcl/servicename/
├── ParcelServiceApplication.java   # Main application class with @SpringBootApplication
├── config/                         # @Configuration classes (Cache, Security, OpenAPI, WebConfig, etc.)
├── controller/                     # REST controllers (API layer, handles HTTP requests/responses)
│   └── validation/                 # Custom validation annotations and logic
├── dto/                            # Data Transfer Objects (Request/Response bodies, never expose entities)
│   ├── request/                    # DTOs for incoming requests
│   └── response/                   # DTOs for outgoing responses
├── model/                          # JPA Entities (@Entity classes, define database schema)
├── repository/                     # Spring Data JPA repositories (data access layer)
├── service/                        # Business logic layer (@Service classes, core application logic)
│   └── impl/                       # Optional: Implementation classes if interfaces are used for services
├── mapper/                         # DTO-Entity mappers (e.g., using MapStruct)
├── util/                           # Generic utility classes (helpers, constants)
└── exception/                      # Custom exceptions and global exception handler (@RestControllerAdvice)

src/test/java/com/hcl/servicename/
├── controller/                     # Unit and integration tests for controllers
├── service/                        # Unit tests for services
└── repository/                     # Integration tests for repositories (e.g., with Testcontainers)

src/main/resources/
├── application.properties          # Base configuration, common for all environments
├── application-dev.properties      # Development specific configurations (H2, debug logging, H2 console)
├── application-prod.properties     # Production specific configurations (PostgreSQL, refined logging)
├── application-test.properties     # Test specific configurations (e.g., Testcontainers setup)
├── logback-spring.xml              # Custom Logback configuration for structured logging
└── db/migration/                   # Flyway/Liquibase scripts for schema migration (e.g., V1__initial_schema.sql)
```

**Naming Conventions**:
*   Classes: PascalCase (e.g., `ParcelServiceApplication`, `ParcelController`)
*   Interfaces: PascalCase, often ending with `Service` or `Repository` (e.g., `ParcelService`, `ParcelRepository`)
*   Methods: camelCase (e.g., `getParcelById`, `createParcel`)
*   Variables: camelCase (e.g., `parcelId`, `requestDto`)
*   Constants: SCREAMING_SNAKE_CASE (e.g., `DEFAULT_PAGE_SIZE`)
*   Packages: kebab-case for service name in root, otherwise lowerCamelCase (e.g., `com.hcl.parcel-service.controller`, `com.hcl.parcel-service.dto`)
 
---
 
## 4. 📝 Development Workflow & Best Practices
 
### Step 1: API & DTO Design (Controller Layer)

1.  **Define DTOs**: Create separate Plain Old Java Objects (POJOs) or Java Records for API requests and responses in the `dto/request` and `dto/response` packages respectively.
    *   Use **immutable DTOs** (final fields, constructor-based initialization, or Java Records) for enhanced thread safety and predictability.
    *   Apply **Jakarta Bean Validation** annotations (`@NotBlank`, `@Size`, `@Email`, `@Pattern`, `@Min`, `@Max`, `@PastOrPresent`, `@Future`, `@Valid` for nested DTOs) for all incoming data validation.
    *   Consider using **MapStruct** for automatic mapping between DTOs and entities in the `mapper` package to reduce boilerplate.
    *   **Crucially, never expose JPA entities directly in the API layer.** This prevents data leakage and decouples your API contract from your internal persistence model.

2.  **Create Controller**: In the `controller` package, create a `@RestController` annotated class. Each controller should represent a single resource or logical group of related resources.

3.  **Map Endpoints**: Use standard Spring Web annotations (`@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`) to define HTTP methods and paths.
    *   Follow **RESTful principles**: resource-based URLs (nouns, not verbs), proper use of HTTP methods for CRUD operations, and appropriate HTTP status codes (200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error).
    *   Use `@PathVariable` for resource identifiers and `@RequestParam` for filtering, sorting, or pagination parameters.

4.  **API Versioning**: Prefix all routes with `/api/v1` (or `v2`, `v3` for future versions) to allow for API evolution without breaking existing clients. Example: `@RequestMapping("/api/v1/parcels")`.

5.  **Input/Output**: Accept DTOs as `@RequestBody` (ensure `@Valid` is used for automatic validation) and return `ResponseEntity<Dto>` for fine-grained control over HTTP status codes, headers, and body.

### Step 2: Business Logic (Service Layer)

1.  **Create Service**: In the `service` package, create an interface (e.g., `ParcelService`) and an implementation class (e.g., `ParcelServiceImpl`) annotated with `@Service`. This promotes good design and testability.

2.  **Inject Dependencies**: Use **constructor injection** for all dependencies (repositories, other services, utility classes). Avoid `@Autowired` on fields.

3.  **Implement Logic**: Write the core business logic here. This layer is responsible for:
    *   Coordinating data access across multiple repositories.
    *   Calling other internal or external services.
    *   Enforcing complex business rules and invariants.
    *   Orchestrating DTO-to-entity and entity-to-DTO mapping (often delegated to a dedicated `mapper` component).
    *   Handling transactional boundaries.

4.  **Use Transactions**: Apply `@Transactional` from `org.springframework.transaction.annotation` on service methods that modify data (create, update, delete).
    *   Use `@Transactional(readOnly = true)` for read-only operations to optimize database performance (can bypass dirty checking, allows database to optimize).
    *   Understand transaction propagation and isolation levels.

5.  **Map DTOs to Entities**: The service layer (or a dedicated mapper) is responsible for converting incoming DTOs to JPA entities before passing them to the repository, and converting entities to DTOs before returning them to the controller. This maintains separation of concerns.

### Step 3: Data Persistence (Repository & Model Layer)

1.  **Create Entity**: In the `model` package, define a JPA `@Entity`.
    *   Use `@Id`, `@GeneratedValue(strategy = GenerationType.IDENTITY)` for primary keys (prefer `IDENTITY` for PostgreSQL).
    *   Use appropriate `@Column` annotations for column mapping, length constraints, nullability (`nullable = false`).
    *   Define relationships using `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`. Understand fetch types (Lazy vs. Eager) to prevent N+1 issues.
    *   **Avoid Lombok's `@Data` on entities**. Instead, use `@Getter`, `@Setter` for fields, `@ToString` (excluding sensitive fields and collections to prevent stack overflow), `@EqualsAndHashCode` (excluding ID and collections to avoid performance issues and inconsistent hash codes for unsaved entities), and a no-arg constructor (required by JPA). Entities should represent the database schema accurately.
    *   Consider using **Flyway or Liquibase** for database schema migrations. Place scripts in `src/main/resources/db/migration`.

2.  **Create Repository**: In the `repository` package, create an interface that extends `JpaRepository<Entity, IdType>`.
    *   Leverage Spring Data JPA's derived query methods (e.g., `findByLastNameAndFirstName`).
    *   Add custom query methods using `@Query` annotation for more complex queries (JPQL or native SQL).
    *   Use `@EntityGraph` or `JOIN FETCH` clauses within `@Query` annotations to eagerly fetch related entities and prevent N+1 select problems.

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in the `exception` package (e.g., `ResourceNotFoundException`, `InvalidInputException`, `OperationNotAllowedException`). These should extend `RuntimeException`. Provide clear error messages.

2.  **Global Handler**: Create a `@RestControllerAdvice` class (e.g., `GlobalExceptionHandler`) in the `exception` package. This centralizes all exception handling logic.

3.  **Handle Exceptions**:
    *   Add `@ExceptionHandler` methods for custom exceptions, standard Spring exceptions (e.g., `MethodArgumentNotValidException` for `@Valid` failures, `ConstraintViolationException` for service layer validation failures), and a fallback generic `Exception` handler.
    *   Return a standardized `ErrorResponse` DTO containing fields like `timestamp`, `status`, `error` (HTTP status reason), `message` (developer-friendly), `path`, and `details` (list of validation errors).
    *   **Never expose internal stack traces or sensitive implementation details** in error responses to clients. Log full stack traces internally at `ERROR` level.

### Step 5: Caching

1.  **Enable Caching**: Add `@EnableCaching` to the main application class.

2.  **Configure Cache**: Create a `CacheConfig` class (in `config` package) to define cache managers.
    *   For **Caffeine (local)**: Configure cache names, expiry policies (TTL, TTI), and maximum size.
    *   For **Redis (distributed)**: Configure `RedisCacheManager`, connection details, and serialization strategies (e.g., JSON for DTOs).
    *   Integrate with Spring Cache Abstraction.

3.  **Apply Caching**: Use `@Cacheable`, `@CachePut`, and `@CacheEvict` on service methods (not controller or repository) to cache results of expensive operations.
    *   `@Cacheable`: Caches method results.
    *   `@CachePut`: Updates the cache with the method's result (always executes the method).
    *   `@CacheEvict`: Removes entries from the cache.
    *   Use the `key` and `condition` attributes for fine-grained control.
    *   Consider cache consistency strategies (e.g., write-through, write-back, invalidate-on-write).

### Step 6: Configuration

-   **Base Config (`application.properties` or `application.yml`)**: Define common properties like `spring.application.name`, default port, context path.
-   **Dev Profile (`application-dev.properties`)**: Configure H2 in-memory database, enable the H2 console (`spring.h2.console.enabled=true`, `spring.h2.console.path=/h2-console`), and set logging to `DEBUG` for application-specific packages (`logging.level.com.hcl.servicename=DEBUG`).
-   **Prod Profile (`application-prod.properties`)**: Configure PostgreSQL connection details using placeholders for sensitive values (e.g., `spring.datasource.password=${DB_PASSWORD}`). Disable detailed health info for security (`management.endpoint.health.show-details=never`). Set logging to `INFO` or `WARN` for most packages.
-   **Test Profile (`application-test.properties`)**: Configure specific settings for integration tests, often disabling some components or pointing to an in-memory database.
-   **Externalization**: All sensitive configurations (API keys, passwords, database URLs) **must** be provided via environment variables or a secure configuration server (e.g., Spring Cloud Config, HashiCorp Vault), not committed directly into the repository.
-   **Custom Properties**: Define custom properties using `@ConfigurationProperties` for type-safe configuration.

### Step 7: `catalog-info.yaml` File Structure
 
The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository. It's crucial for Backstage and internal developer portals.
 
```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: # required, unique name, e.g., "parcel-service"
  description: # required, human-readable description of the microservice
  tags:
    - java
    - spring-boot
    - microservice
    - rest-api
  annotations:
    # Key-value pairs for tool integrations and operational metadata
    github.com/project-slug: hcl-org/parcel-service # Link to GitHub repository
    jira.com/project-key: PARCEL                     # Link to Jira project
    jenkins.io/job-full-name: team-x/parcel-service-ci # CI/CD pipeline link
    sonarqube.org/project-key: com.hcl.parcel-service  # SonarQube project
    grafana.com/dashboard-selector: "app=parcel-service" # Grafana dashboard
    datadoghq.com/monitor-id: "12345" # Example for DataDog
  links:
    - url: https://confluence.hcl.com/display/SVC/Parcel+Service+Docs # Link to Confluence documentation
      title: Service Documentation
      icon: dashboard
    - url: https://jenkins.hcl.com/job/parcel-service-cd # Link to CD pipeline
      title: CD Pipeline
      icon: cicd
    - url: https://grafana.hcl.com/d/parcel-service # Link to service dashboard
      title: Operational Dashboard
      icon: monitor
spec:
  type: service # This is a backend service
  lifecycle: production # e.g., 'experimental', 'development', 'production', 'deprecated'
  owner: team-parcels # required, the owning team or user, e.g., 'team-backend'
  system: shipping-logistics # optional, the larger system this component belongs to, e.g., 'customer-management'
  providesApis:
    - parcel-api # The API exposed by this service
  consumesApis:
    - user-management-api # Example: if it calls user management service
    - inventory-service-api
  dependsOn:
    - resource:database/parcel-db # Dependency on the database resource
    - component:redis-cache-cluster # Dependency on a Redis cluster
```
 
---
 ### Step 8: Coding Standards
 
This section outlines mandatory coding standards for various technologies. Adhering to these ensures consistency, readability, and maintainability across all projects within the enterprise.
 
#### Java Spring Boot Coding Standards
 
1.  Use `final` keyword for immutable variables, especially method parameters and local variables, where appropriate.
2.  Favor functional interfaces and lambda expressions for concise code, especially with Streams API.
3.  Utilize Java Records for immutable data carriers (DTOs, configurations) where applicable.
4.  Apply `var` judiciously for local variable type inference to improve readability with complex types, but avoid it where explicit type clarity is paramount.
5.  Use `Optional` for representing the potential absence of a value, returning it from methods where a `null` might otherwise be returned. Avoid `Optional` as method parameters or fields.
6.  Ensure proper use of `try-with-resources` for all `AutoCloseable` resources (e.g., streams, database connections).
7.  Write comprehensive Javadoc for all public API (classes, methods, fields) explaining purpose, parameters, return values, and exceptions.
8.  Maintain a clear separation of concerns using distinct layers (controller, service, repository, model).
9.  Use constructor injection for dependencies. Avoid field injection (`@Autowired` on fields).
10. Ensure all DTOs are immutable (e.g., using `final` fields and a constructor, or Java Records).
11. Apply Jakarta Bean Validation (`@Valid`, `@NotBlank`, `@Size`, etc.) consistently on DTOs at the controller layer.
12. Handle exceptions globally using `@RestControllerAdvice` and `@ExceptionHandler`. Return consistent `ErrorResponse` DTOs.
13. Leverage Spring's stereotype annotations (`@Controller`, `@Service`, `@Repository`, `@Component`, `@Configuration`).
14. Use `@Transactional` judiciously on service methods for database write operations. Use `readOnly = true` for read operations.
15. Configure database connection pooling (e.g., HikariCP) appropriately.
16. Implement comprehensive unit, integration, and end-to-end tests. Aim for high test coverage (e.g., >80%).
17. Write meaningful commit messages following a conventional commit specification.
18. Ensure proper logging with SLF4J and Logback, using appropriate log levels (DEBUG, INFO, WARN, ERROR).
19. Externalize all configurations using Spring Boot's `application.properties` or `application.yml` and environment variables for sensitive data.
20. Follow REST principles for API design (statelessness, resource-based URLs, appropriate HTTP methods, status codes).
21. Secure APIs using Spring Security (e.g., OAuth2, JWT).
22. Document APIs using SpringDoc OpenAPI.
23. Ensure all code is formatted consistently using tools like Spotless or Maven Formatter Plugin.
24. Avoid N+1 select problems in JPA using `@EntityGraph` or `JOIN FETCH`.
25. Utilize Spring Boot Actuator for monitoring and management.
 
#### Python Coding Standards
 
1.  **PEP 8 Compliance**: Adhere strictly to PEP 8 style guide for maximum readability. Use linters like `Flake8` or `Pylint`.
2.  **Meaningful Names**: Use descriptive variable, function, class, and module names.
3.  **Imports**: Organize imports according to PEP 8 (standard library, third-party, local application, each group sorted alphabetically).
4.  **Docstrings**: Write clear and concise docstrings for all modules, classes, and public functions/methods, explaining their purpose, arguments, and return values. Use reStructuredText or Google style.
5.  **Type Hinting**: Use type hints consistently for function parameters and return values to improve code clarity and enable static analysis.
6.  **Exception Handling**: Use specific exceptions where possible, avoid bare `except:` clauses. Log exceptions appropriately.
7.  **Logging**: Implement proper logging using Python's `logging` module, not `print()` statements for production.
8.  **List Comprehensions**: Prefer list comprehensions, dictionary comprehensions, and generator expressions over explicit `for` loops when appropriate for conciseness and efficiency.
9.  **Context Managers**: Use `with` statements for resources that need to be explicitly closed (e.g., files, database connections).
10. **Immutability**: Favor immutable data structures where appropriate (e.g., tuples instead of lists if content doesn't change).
11. **Unit Tests**: Write comprehensive unit tests for all functions and classes, using `unittest` or `pytest`. Aim for high test coverage.
12. **Virtual Environments**: Always use virtual environments for project dependencies.
13. **Dependency Management**: Use `requirements.txt` or `pyproject.toml` (with Poetry or PDM) to manage dependencies precisely.
14. **Configuration**: Externalize configurations, especially sensitive data, using environment variables or dedicated configuration files (e.g., `.env`, YAML).
15. **API Design (for web frameworks)**: Follow RESTful principles, use appropriate HTTP status codes, and validate input.
16. **Security**: Sanitize all user inputs, protect against common vulnerabilities (e.g., SQL injection, XSS), and handle sensitive data securely.
17. **Performance**: Be mindful of algorithm complexity. Profile code for bottlenecks.
18. **Concurrency**: Use appropriate concurrency models (e.g., `asyncio`, `threading`, `multiprocessing`) based on the task's nature (I/O-bound vs. CPU-bound).
19. **Code Review**: Participate in and conduct thorough code reviews.
20. **Version Control**: Use Git for version control and follow a consistent branching strategy.
 
#### Angular Framework Coding Standards
 
1.  **Consistent Naming Conventions**: Follow Angular style guide conventions (e.g., `component-name.component.ts`, `service-name.service.ts`).
2.  **Single Responsibility Principle (SRP)**: Each component, service, or module should have a single, well-defined purpose.
3.  **Folder Structure**: Organize files by feature, not by type. Group related components, services, and modules in their own feature folder.
4.  **Module Organization**: Use feature modules to lazy load parts of the application and keep the `AppModule` lean.
5.  **Smart vs. Dumb Components**: Differentiate between container/smart components (handle logic, data) and presentational/dumb components (display data, emit events).
6.  **Services for Business Logic**: All business logic, data fetching, and shared functionality should reside in services, not directly in components.
7.  **RxJS Best Practices**:
    *   Use `pipe()` for chaining operators.
    *   Unsubscribe from observables to prevent memory leaks (e.g., using `takeUntil`, `async` pipe, `ngOnDestroy`).
    *   Favor higher-order mapping operators (`switchMap`, `mergeMap`, `concatMap`, `exhaustMap`) over nested subscriptions.
8.  **Lifecycle Hooks**: Understand and use Angular lifecycle hooks appropriately (e.g., `OnInit`, `OnDestroy`, `AfterViewInit`).
9.  **Template Syntax**: Use property binding (`[]`) and event binding (`()`) correctly. Avoid excessive logic in templates.
10. **Change Detection**: Be aware of Angular's change detection mechanism. Use `OnPush` strategy where possible for performance optimization.
11. **Forms**: Use reactive forms for complex forms and validation.
12. **Routing**: Implement lazy loading for feature modules to improve initial load performance. Protect routes with guards.
13. **Error Handling**: Implement global error handling for API calls and application-level errors.
14. **Unit and Integration Tests**: Write comprehensive tests for components, services, and pipes using Karma/Jasmine or Jest. Aim for high coverage.
15. **Linter and Formatter**: Integrate `ESLint` and `Prettier` to enforce coding style and catch potential issues.
16. **Accessibility (a11y)**: Design and implement user interfaces with accessibility in mind (e.g., ARIA attributes, semantic HTML).
17. **Performance Optimization**: Lazy loading, trackBy functions, OnPush strategy, minimizing change detection cycles, optimizing images.
18. **Security**: Sanitize user inputs, protect against XSS, CSRF, and other common web vulnerabilities.
19. **Internationalization (i18n)**: Design the application for easy translation and localization.
20. **State Management**: Consider state management solutions (e.g., NgRx, Akita, services) for complex applications.
 
### Step 9: Documentation

-   **OpenAPI (Swagger UI)**: Configure a `OpenApiConfig` bean (in `config` package) to set the API title, version, description, and contact information. Use `@Operation`, `@ApiResponse`, `@Parameter` annotations in controllers for detailed API documentation. The `springdoc-openapi-starter-webmvc-ui` dependency provides Swagger UI at `/swagger-ui.html` and OpenAPI JSON at `/v3/api-docs`.
-   **JavaDoc**: Add comprehensive JavaDoc comments to all public classes, interfaces, methods, and fields in controllers, services, repositories, and DTOs. Explain purpose, parameters (`@param`), return values (`@return`), and exceptions (`@throws`).
-   **README.md**: A detailed `README.md` at the project root explaining how to build, run, test, and deploy the service. Include API endpoints, environment variables, and essential setup instructions.
-   **Architectural Decision Records (ADRs)**: Document significant architectural decisions and their rationale to capture historical context and facilitate future understanding.
    *   *Relevant Link*: [HCL ADR Template](https://github.com/hcl-org/adr-templates) (Placeholder)

### Step 10: Testing

-   **Unit Tests**: Test individual components (services, utilities, mappers) in isolation by mocking their dependencies (e.g., repositories, other services) using Mockito. Focus on business logic. Place tests in `src/test/java/<root-package>/service/`.
-   **Integration Tests**: Use `@SpringBootTest` and `@ActiveProfiles("test")` to test the full application flow, from controller to database.
    *   For database integration tests, use **Testcontainers** to spin up real database instances (e.g., PostgreSQL) in Docker containers, ensuring tests run against an environment similar to production.
    *   Use `MockMvc` for testing controller endpoints without starting a full server.
    *   Place tests in `src/test/java/<root-package>/controller/` and `src/test/java/<root-package>/repository/`.
-   **Test Data Management**: Use data factories, builders, or dedicated test utility classes to create realistic test data efficiently. Avoid hardcoding test data.
-   **Coverage**: Aim for a minimum of 80% test coverage for critical business logic (services, controllers). Use JaCoCo plugin for reporting.
-   **Contract Testing**: Consider using Spring Cloud Contract or Pact for consumer-driven contract testing to ensure API compatibility between microservices.
    *   *Relevant Link*: [HCL Contract Testing Guide](https://confluence.hcl.com/display/QA/Contract+Testing+Guide) (Placeholder)
 
---
 
## 5. 🐳 Containerization & Deployment
 
### Dockerfile

Create a multi-stage `Dockerfile` in the project root:

1.  **Build Stage (`maven` image)**:
    *   Use a `maven:3.9-eclipse-temurin-21-alpine` base image.
    *   Copy `pom.xml` and source code.
    *   Build the application JAR using `mvn clean package -DskipTests`.

2.  **Runtime Stage (`jre` image)**:
    *   Use a minimal JRE image like `eclipse-temurin:21-jre-alpine` for a smaller image footprint and fewer vulnerabilities.
    *   **Security**: Create a non-root user and group (`addgroup -S spring && adduser -S spring -G spring`) and run the application as this user (`USER spring`). This mitigates potential container escapes.
    *   Set working directory (`WORKDIR /app`).
    *   Copy the built JAR from the build stage (`COPY --from=build /target/*.jar app.jar`).
    *   **Health Check**: Add a `HEALTHCHECK` instruction to monitor application health (`HEALTHCHECK --interval=30s --timeout=10s --retries=5 CMD curl --fail http://localhost:8080/actuator/health || exit 1`).
    *   **Execution**: Use `ENTRYPOINT ["java", "-Dspring.profiles.active=prod", "-jar", "app.jar"]`. Specify the production profile.
    *   Expose application port (`EXPOSE 8080`).

### Environment Variables

All secrets (database passwords, API keys, third-party service credentials) **must** be supplied via environment variables, not hardcoded in properties files or Docker images. Use appropriate secrets management solutions in production (e.g., Kubernetes Secrets, HashiCorp Vault, AWS Secrets Manager, Azure Key Vault).
    *   *Relevant Link*: [HCL Secrets Management Guide](https://confluence.hcl.com/display/SEC/Secrets+Management+Guide) (Placeholder)

### Kubernetes Readiness

-   **Liveness and Readiness Probes**: Ensure `/actuator/health/liveness` and `/actuator/health/readiness` endpoints are correctly configured and utilized in Kubernetes deployments.
-   **Resource Limits**: Define CPU and memory limits/requests in Kubernetes deployment manifests to ensure stable resource allocation.
-   **Configuration Management**: Utilize Kubernetes ConfigMaps for non-sensitive configurations and Secrets for sensitive data.
-   **Logging**: Ensure container logs are directed to `stdout`/`stderr` for collection by a log aggregation system (e.g., ELK stack, Splunk).
    *   *Relevant Link*: [HCL Kubernetes Deployment Standards](https://confluence.hcl.com/display/OPS/Kubernetes+Deployment+Standards) (Placeholder)

### CI/CD Integration

-   Automate building, testing, scanning (security, quality), and deployment using a robust CI/CD pipeline (e.g., Jenkins, GitLab CI, GitHub Actions, Azure DevOps).
-   Ensure all code changes trigger the pipeline, running unit/integration tests and static analysis checks.
-   Implement automated semantic versioning for artifacts.
    *   *Relevant Link*: [HCL CI/CD Pipeline Reference](https://confluence.hcl.com/display/DEVOPS/CI_CD+Pipeline+Reference) (Placeholder)
 
---
 
This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.
 
Platform engineers or lead developers are responsible for maintaining the referenced confluence, team, or GitHub links to ensure they are current and relevant.
 
---
 
### Sample Agent.md
 
Platform engineer agent can create this..  
 
Here the details and the best practices we need to give link to appropriate confluence, team or github link so that it will be more realistic 
 
then the consumer while building a microservice, he can download this agent.md file from IDP and use his specific development environment.
```