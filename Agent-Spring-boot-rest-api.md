Here is the more detailed spec file, with the coding standards for Java Spring Boot, Python, and Angular integrated into the `#Coding standards#` section.

---

# 🤖 Agent Protocol: Building Enterprise-Grade Java Microservices

This document provides a comprehensive guide for AI coding agents to build secure, scalable, and maintainable microservices using Java 21 and Spring Boot 3. Adherence to these principles is mandatory for all generated projects.

---

## 1. 🎯 Core Principles

-   **Security First**: Implement security controls at every layer. Assume a zero-trust environment.
-   **Production Ready**: All code must be suitable for production, with robust configuration, logging, and error handling.
-   **Developer Experience**: Ensure the project is easy to set up, run, and understand. Prioritize clear documentation and clean code.
-   **Performance by Design**: Build for low latency and high throughput. Implement caching and efficient data access patterns.
-   **Observability**: The system's internal state must be inferable from its external outputs (logs, metrics, traces).

---

## 2. 🛠️ Standard Technology Stack

| Component     | Technology                            | Version/Standard                                      |
|:--------------|:--------------------------------------|:------------------------------------------------------|
| **Language**  | Java                                  | **21**                                                |
| **Framework** | Spring Boot                           | **3.3+**                                              |
| **Build Tool**| Maven                                 | 3.9+                                                  |
| **Database**  | PostgreSQL (Prod), H2 (Dev/Test)      | Latest                                                |
| **API Spec**  | OpenAPI                               | 3.0                                                   |
| **Container** | Docker                                | Latest                                                |
| **Caching**   | Caffeine (local), Redis (distributed) | Latest                                                |
| **Validation**| Jakarta Bean Validation               | 3.0                                                   |
| **Testing**   | JUnit 5, Mockito                      | Latest                                                |
| **Tracing**   | OpenTelemetry                         | Latest (integrated with Spring Cloud Sleuth/Micrometer)|
| **Metrics**   | Micrometer                            | Latest (integrated with Prometheus/Grafana)           |
| **Logging**   | SLF4J with Logback                    | Latest                                                |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using **Spring Initializr** (`https://start.spring.io/`) with the following default dependencies:

-   Spring Web
-   Spring Data JPA
-   Spring Boot Actuator
-   Spring Cache Abstraction
-   Jakarta Bean Validation
-   Lombok
-   PostgreSQL Driver
-   H2 Database
-   SpringDoc OpenAPI (`org.springdoc:springdoc-openapi-starter-webmvc-ui`)
-   Testcontainers (for integration testing with databases)
-   Micrometer Tracing (Brave or OTel Bridge) & Prometheus (for metrics)

### Standard Package Structure

Organize all source code under a root package (e.g., `com.hcl.servicename`).

```
com/hcl/servicename/
├── ParcelServiceApplication.java   # Main application class
├── config/                         # @Configuration beans (Cache, Security, OpenAPI, Tracing, Metrics)
├── controller/                     # REST controllers (API layer)
│   └── api/                        # Optional: Subpackage for API versioning (e.g., v1)
├── dto/                            # Data Transfer Objects for API requests/responses
├── model/                          # JPA Entities
├── repository/                     # Spring Data JPA repositories
├── service/                        # Business logic interfaces and implementations
├── exception/                      # Custom exceptions and global handler
├── mapper/                         # MapStruct mappers for DTO-Entity conversion
└── util/                           # Utility classes
```

---

## 4. 📝 Development Workflow & Best Practices

### Step 1: API & DTO Design (Controller Layer)

1.  **Define DTOs**: Create separate Plain Old Java Objects (POJOs) for API requests and responses in the `dto` package. Use Jakarta Bean Validation annotations (`@NotBlank`, `@Size`, `@Email`, `@Pattern`, `@PositiveOrZero`, etc.) for all incoming data. **Never expose JPA entities directly in the API.**
2.  **Create Controller**: In the `controller` package, create a `@RestController`.
3.  **Map Endpoints**: Use `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`. Ensure semantic use of HTTP methods.
4.  **API Versioning**: Prefix all routes with `/api/v1` (or `v2`, etc.). Example: `@RequestMapping("/api/v1/parcels")`. Consider using a subpackage `controller/api/v1` for cleaner separation.
5.  **Input/Output**: Accept DTOs as `@RequestBody` (validated with `@Valid`) and return `ResponseEntity<Dto>` or `ResponseEntity<List<Dto>>`. Use `@RequestParam` for query parameters, `@PathVariable` for path variables.
6.  **OpenAPI Documentation**: Use `@Operation`, `@ApiResponse`, `@Parameter` annotations from `springdoc-openapi` to enrich API documentation directly in the controller methods.

### Step 2: Business Logic (Service Layer)

1.  **Create Service Interface and Implementation**: In the `service` package, define an interface (e.g., `ParcelService`) and its `@Service` implementation (e.g., `ParcelServiceImpl`). This promotes loose coupling and easier testing.
2.  **Inject Dependencies**: Use **constructor injection** for repositories, other services, and utilities. Avoid field injection (`@Autowired` on fields).
3.  **Implement Logic**: Write the core business logic here. This layer is responsible for coordinating data access, calling other services, applying business rules, and handling transactions.
4.  **Use Transactions**: Apply `@Transactional` (from `org.springframework.transaction.annotation`) at the class or method level. Use `@Transactional(readOnly = true)` for read-only operations to optimize performance and prevent accidental writes.
5.  **Map DTOs to Entities**: The service layer is responsible for converting incoming DTOs to JPA entities before passing them to the repository, and converting entities to DTOs before returning them to the controller. Use a mapping library like **MapStruct** for efficient and type-safe conversions (create mappers in the `mapper` package).

### Step 3: Data Persistence (Repository & Model Layer)

1.  **Create Entity**: In the `model` package, define a JPA `@Entity`. Use `@Id`, `@GeneratedValue` (with `GenerationType.IDENTITY` for PostgreSQL), and appropriate `@Column` annotations. Define relationships (`@OneToMany`, `@ManyToOne`, etc.) carefully, considering fetch types and cascade options. Avoid Lombok's `@Data` on entities; prefer `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode(callSuper = false, onlyExplicitlyIncluded = true)` and `@NoArgsConstructor`, `@AllArgsConstructor` where necessary, ensuring correct handling of `@Id` in `equals`/`hashCode`.
2.  **Create Repository**: In the `repository` package, create an interface that extends `JpaRepository<Entity, IdType>`. Add custom query methods using Spring Data JPA's query method derivations or `@Query` annotations as needed. For complex queries, consider `JpaSpecificationExecutor`.

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in the `exception` package (e.g., `ResourceNotFoundException extends RuntimeException`, `InvalidInputException extends RuntimeException`).
2.  **Global Handler**: Create a `@RestControllerAdvice` class (e.g., `GlobalExceptionHandler`) in the `exception` package.
3.  **Handle Exceptions**: Add `@ExceptionHandler` methods for:
    *   Custom exceptions (e.g., `ResourceNotFoundException`).
    *   Validation exceptions (`MethodArgumentNotValidException`, `ConstraintViolationException`).
    *   Security-related exceptions (e.g., `AccessDeniedException`).
    *   Database-related exceptions (e.g., `DataIntegrityViolationException`).
    *   A fallback generic `Exception` handler.
    Return a standardized `ErrorResponse` DTO (with `timestamp`, `status`, `error`, `message`, `path`) and appropriate HTTP status codes. **Never expose stack traces directly in production error responses.** Log full stack traces internally for debugging.

### Step 5: Caching

1.  **Enable Caching**: Add `@EnableCaching` to the main application class.
2.  **Configure Cache**: Create a `CacheConfig` class in the `config` package.
    *   For local caching, configure **CaffeineCacheManager**.
    *   For distributed caching (in a microservice environment), configure **RedisCacheManager** (Spring Data Redis integration).
    *   Define specific caches and their time-to-live (TTL) and other eviction policies.
3.  **Apply Caching**: Use `@Cacheable`, `@CachePut`, and `@CacheEvict` on service methods to cache results of expensive read operations, update cached data after writes, and invalidate caches after deletions. Ensure cache names are descriptive.

### Step 6: Configuration

-   **Base Config (`application.properties` or `application.yml`)**: Define common properties like `spring.application.name`, default server port, common logging levels.
-   **Dev Profile (`application-dev.properties/yml`)**: Configure H2 in-memory database, enable the H2 console (`spring.h2.console.enabled=true`), and set logging to `DEBUG` or `TRACE` for application packages.
-   **Test Profile (`application-test.properties/yml`)**: Configure specific settings for integration tests, potentially using Testcontainers defaults.
-   **Prod Profile (`application-prod.properties/yml`)**: Configure PostgreSQL connection details. Use placeholders for secrets (e.g., `${DB_USERNAME}`, `${DB_PASSWORD}`) to be resolved via environment variables. Disable detailed health info (`management.endpoint.health.show-details=never`), expose only essential actuator endpoints.

### Step 7: `catalog-info.yaml` File Structure

The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository. This file is crucial for Backstage and other Developer Portal integrations.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: # required, unique kebab-case name (e.g., parcel-service)
  description: # required, human-readable description of the microservice
  tags:
    - java
    - spring-boot
    - microservice
    - rest-api
    - # add other relevant tags (e.g., logistics, inventory)
  annotations:
    # key-value pairs for tool integrations
    github.com/project-slug: # e.g., alexstephen-github/parcel-service
    sonarqube.org/project-key: # e.g., com.hcl:parcel-service
    jira/project-key: # e.g., PARCEL
    argocd/app-name: # e.g., parcel-service-prod
    # Link to runbooks, dashboards, logs (replace with actual URLs)
    backstage.io/kubernetes-id: parcel-service
    grafana/dashboard-selector: 'app=parcel-service'
    prometheus.io/query: 'sum(http_server_requests_seconds_count{uri="/api/v1/parcels"})'
    backstage.io/techdocs-ref: url:https://github.com/alexstephen-github/parcel-service/tree/main/docs
  links:
    - url: https://alexstephen-github.com/CodingStandards/blob/main/springboot-rules.txt
      title: Spring Boot Coding Standards
      icon: code
    - url: https://github.com/alexstephen-github/parcel-service
      title: GitHub Repository
      icon: github
    - url: https://<your-ci-cd-tool-url>/project/parcel-service
      title: CI/CD Pipeline
      icon: build
    - url: https://<your-monitoring-tool-url>/dashboards/parcel-service
      title: Monitoring Dashboard
      icon: dashboard
    - url: https://<your-alerting-tool-url>/alerts/parcel-service
      title: Alerts
      icon: alert
spec:
  type: service # required, indicates this is a runnable service
  lifecycle: production # required, e.g., 'experimental', 'development', 'production'
  owner: team-logistics # required, the owning team or user (e.g., 'team-finance', 'user:johndoe')
  system: logistics-management # optional, the larger system this component belongs to (e.g., 'order-processing')
  dependsOn:
    - resource:database/postgresql-logistics # Example dependency on a shared database resource
    - component:authentication-service # Example dependency on another microservice
  providesApis:
    - parcel-api-v1 # API entity reference (defined separately or inline)
```

---

### Step 8: Coding standards

Adherence to the following coding standards is mandatory for code written in these languages/frameworks.

#### 8.1. Spring Boot Java Coding Standards

*   **Package Naming**: Use lowercase for package names (e.g., `com.example.service`).
*   **Class Naming**: Use PascalCase for class names (e.g., `UserService`, `ProductController`).
*   **Method Naming**: Use camelCase for method names (e.g., `getUserById`, `saveProduct`).
*   **Variable Naming**: Use camelCase for local variables and instance variables (e.g., `firstName`, `totalAmount`).
*   **Constant Naming**: Use screaming snake case for constants (e.g., `MAX_RETRIES`, `DEFAULT_PAGE_SIZE`).
*   **Interface Naming**: Use PascalCase (e.g., `UserService`). Avoid `I` prefix unless strictly necessary to avoid name collisions.
*   **File Naming**: Match the class name (e.g., `UserService.java`).
*   **Brace Style**: Use Egyptian brace style (opening brace on the same line as the declaration).
*   **Indentation**: Use 4 spaces for indentation.
*   **Line Length**: Keep lines under 120 characters.
*   **Comments**: Use Javadoc for public API, single-line comments for internal explanations.
*   **Annotations**: Place annotations on a separate line above the element they modify.
*   **Logging**: Use SLF4J with a logger instance named `log` (e.g., `private static final Logger log = LoggerFactory.getLogger(MyClass.class);`).
*   **Error Handling**: Throw specific exceptions, handle them globally using `@ControllerAdvice`.
*   **DTOs**: Use DTOs for data transfer between layers and API, avoiding direct entity exposure.
*   **Services**: Encapsulate business logic in service layer.
*   **Repositories**: Use Spring Data JPA repositories for data access.
*   **Configuration**: Centralize application configuration in `application.yml` or `application.properties`.
*   **Dependency Injection**: Prefer constructor injection for mandatory dependencies.
*   **Immutability**: Favor immutable objects where possible, especially for DTOs.
*   **Testing**: Write unit tests for individual components and integration tests for service layers.
*   **Naming Conventions for REST Endpoints**:
    *   Resource names in plural (e.g., `/users`, `/products`).
    *   Use HTTP methods for actions (GET for retrieve, POST for create, PUT for update, DELETE for delete).
    *   Avoid verbs in endpoint names (e.g., `/getAllUsers` should be `/users`).
    *   Use path variables for resource identification (e.g., `/users/{id}`).
    *   Use query parameters for filtering, sorting, pagination (e.g., `/users?status=active&page=1`).
*   **Security**: Implement Spring Security, OAuth2/JWT for authentication and authorization.
*   **API Versioning**: Use URI versioning (e.g., `/api/v1/users`).
*   **Caching**: Use Spring Cache annotations (`@Cacheable`, `@CachePut`, `@CacheEvict`).
*   **Database Interactions**: Use `@Transactional` for methods that modify data.
*   **Lombok Usage**: Use `@Getter`, `@Setter`, `@NoArgsConstructor`, `@AllArgsConstructor` on DTOs; avoid `@Data` on JPA entities.
*   **Jackson**: Use Jackson annotations for JSON serialization/deserialization control.
*   **Build Tool**: Use Maven with a consistent `pom.xml` structure.
*   **Health Checks**: Leverage Spring Boot Actuator for health, metrics, and info endpoints.
*   **Asynchronous Operations**: Use `@Async` with proper `ThreadPoolTaskExecutor` configuration.
*   **Validation**: Use Jakarta Bean Validation (`@Valid`) on DTOs in controllers.

#### 8.2. Python Coding Standards (PEP 8 Adherence)

*   **Indentation**: 4 spaces per indentation level.
*   **Line Length**: Max 79 characters for code, 72 for docstrings/comments.
*   **Blank Lines**: Use 2 blank lines between top-level function/class definitions, 1 blank line between method definitions.
*   **Imports**: Imports on separate lines, grouped by standard library, third-party, local application, each group sorted alphabetically. Absolute imports are preferred.
*   **Naming Conventions**:
    *   Modules: `lowercase_with_underscores`.
    *   Packages: `lowercase_with_underscores`.
    *   Classes: `PascalCase`.
    *   Functions/Methods: `lowercase_with_underscores`.
    *   Variables: `lowercase_with_underscores`.
    *   Constants: `UPPERCASE_WITH_UNDERSCORES`.
    *   Protected members: `_single_leading_underscore`.
    *   Private members: `__double_leading_underscore`.
*   **Docstrings**: Use triple double quotes for docstrings for modules, classes, functions, and methods. Follow reStructuredText format if applicable for documentation generation.
*   **Comments**: Use `#` for inline comments, complete sentences.
*   **Whitespace**:
    *   No whitespace immediately inside parentheses, brackets or braces.
    *   No whitespace immediately before a comma, semicolon, or colon.
    *   Always put a space after a comma, semicolon, or colon.
    *   Surround binary operators with a single space on either side.
*   **Type Hinting**: Use type hints for function arguments and return values.
*   **Error Handling**: Use `try-except-finally` blocks for graceful error handling. Be specific with exception types.
*   **Logging**: Use Python's `logging` module. Configure log levels appropriately.
*   **String Formatting**: Prefer f-strings for string interpolation (Python 3.6+).
*   **Virtual Environments**: Always use virtual environments for project dependencies.
*   **Testing**: Use `unittest` or `pytest` for testing.
*   **List Comprehensions**: Prefer list comprehensions over `for` loops for simple transformations.
*   **Generators**: Use generators for large datasets to save memory.
*   **Decorators**: Use decorators for cross-cutting concerns (e.g., logging, authentication).
*   **README**: Include a comprehensive `README.md` file.

#### 8.3. Angular Coding Standards

*   **File Naming**:
    *   General: `feature-name.type.ts` (e.g., `user-list.component.ts`, `auth.service.ts`).
    *   Modules: `feature-name.module.ts`.
    *   Routing Modules: `feature-name-routing.module.ts`.
*   **Component Naming**:
    *   Selector: `app-feature-name` (e.g., `app-user-list`). Use `kebab-case`.
    *   Class: `FeatureNameComponent` (e.g., `UserListComponent`). Use `PascalCase`.
*   **Service Naming**:
    *   Class: `FeatureNameService` (e.g., `AuthService`). Use `PascalCase`.
*   **Directive Naming**:
    *   Selector: `appFeatureName` (e.g., `appHighlight`). Use `camelCase`.
    *   Class: `FeatureNameDirective`. Use `PascalCase`.
*   **Pipe Naming**:
    *   Name: `featureName` (e.g., `currencyFormat`). Use `camelCase`.
    *   Class: `FeatureNamePipe`. Use `PascalCase`.
*   **Module Naming**:
    *   Class: `FeatureNameModule` (e.g., `AuthModule`). Use `PascalCase`.
*   **Indentation**: 2 spaces for TypeScript, HTML, CSS.
*   **Line Length**: Max 120 characters.
*   **Imports**: Organize imports alphabetically.
*   **Typing**: Use strong typing with interfaces and types.
*   **Component Structure**:
    *   Keep components focused on presentation. Delegate business logic to services.
    *   Use `OnPush` change detection strategy for performance.
    *   Prefer `@Input()` and `@Output()` for component communication.
*   **Service Structure**:
    *   Mark services with `@Injectable({ providedIn: 'root' })` for tree-shakable providers.
    *   Encapsulate API calls and complex logic.
*   **Routing**:
    *   Use lazy loading for feature modules to improve initial load time.
    *   Define routes in a dedicated routing module.
*   **RxJS**:
    *   Use RxJS for asynchronous operations and event handling.
    *   Handle subscriptions to prevent memory leaks (e.g., `takeUntil`, `async` pipe).
    *   Use operators for data manipulation (e.g., `map`, `filter`, `switchMap`).
*   **Templates (HTML)**:
    *   Use structural directives (`*ngIf`, `*ngFor`) effectively.
    *   Use attribute directives (`[ngClass]`, `[ngStyle]`) for dynamic styling.
    *   Avoid complex logic in templates; move it to the component class.
*   **Styling (CSS/SCSS)**:
    *   Use component-scoped styles (view encapsulation).
    *   Prefer BEM (Block Element Modifier) or similar methodologies for class naming if global styles are needed.
*   **Testing**:
    *   Write unit tests for components, services, and pipes.
    *   Use TestBed for component testing.
    *   Aim for high test coverage.
*   **Error Handling**: Implement global error handling using `ErrorHandler` or `HttpInterceptor`.
*   **Interceptors**: Use HTTP Interceptors for common tasks like adding auth headers, logging, or error handling.
*   **State Management**: Consider NGRX or NGXS for complex state management in large applications.
*   **Accessibility (A11y)**: Design and develop with accessibility in mind (ARIA attributes, semantic HTML).

---

### Step 9: Documentation

-   **OpenAPI**: Configure an `OpenApiConfig` bean in the `config` package to set the API title, version, description, and security schemes (e.g., JWT bearer token). Use `springdoc-openapi` annotations in controllers for detailed endpoint documentation.
-   **JavaDoc**: Add comprehensive JavaDoc comments to all public classes, interfaces, enums, and methods in controllers, services, repositories, and DTOs. Explain their purpose, parameters, return values, and any exceptions thrown.
-   **README.md**: A detailed `README.md` at the project root explaining how to set up, run, test, and deploy the service, along with key architectural decisions.
-   **Architecture Decision Records (ADRs)**: Use ADRs for significant architectural choices to document the context, decision, and consequences.

### Step 10: Testing

-   **Unit Tests**: Test individual components (services, controllers, utilities) in isolation by mocking their dependencies. Use JUnit 5 and Mockito. Aim for granular testing of business logic. Place tests in `src/test/java`.
-   **Integration Tests**: Use `@SpringBootTest` and `@ActiveProfiles("test")` to test the full application flow, from controller to database. Leverage **Testcontainers** for ephemeral, real database instances (e.g., PostgreSQL) to ensure robust data layer testing without relying on local setups.
-   **Component Tests**: Test specific layers of the application (e.g., `@WebMvcTest` for controllers, `@DataJpaTest` for repositories).
-   **Test Coverage**: Aim for a minimum of 80% line coverage and 70% branch coverage for core business logic, measured using JaCoCo. Integrate coverage reporting into CI/CD.

---

## 5. 🐳 Containerization & Deployment

### Dockerfile

Create a multi-stage `Dockerfile` in the project root:

1.  **Build Stage**: Use a `maven` base image (e.g., `maven:3.9.6-eclipse-temurin-21-alpine`) to build the application JAR. Copy source, build with `mvn clean package -DskipTests`.
2.  **Runtime Stage**: Use a minimal JRE image (e.g., `eclipse-temurin:21-jre-alpine`). This reduces image size and attack surface.
3.  **Security**:
    *   Create a dedicated **non-root user** and group (e.g., `spring`) and run the application as this user (`USER spring`).
    *   Set appropriate file permissions.
    *   Avoid installing unnecessary tools.
4.  **Health Check**: Add a `HEALTHCHECK` instruction that calls the `/actuator/health` endpoint after a short delay, with a reasonable interval, timeout, and retry count. Example: `HEALTHCHECK --interval=30s --timeout=10s --retries=3 CMD curl --fail http://localhost:8080/actuator/health || exit 1`.
5.  **Execution**: Use `ENTRYPOINT ["java", "-jar", "app.jar"]` with `-Dspring.profiles.active=prod` to ensure the production profile is active.
6.  **Resource Limits**: Consider adding JVM memory arguments (e.g., `-Xmx256m`) directly in the `ENTRYPOINT` based on expected container limits.

### Environment Variables

All secrets (database passwords, API keys, third-party service credentials) **must** be supplied via environment variables, not hardcoded in properties files. Use Spring Boot's externalized configuration capabilities.

### Deployment Strategy

-   **Kubernetes/OpenShift**: Design for container orchestration platforms. Ensure readiness and liveness probes are configured.
-   **Configuration Management**: Utilize external configuration services (e.g., Spring Cloud Config, Kubernetes ConfigMaps/Secrets) for managing application properties in different environments.
-   **CI/CD Pipeline**: Automate build, test, containerization, and deployment processes using tools like Jenkins, GitLab CI, GitHub Actions, or ArgoCD.

---

This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.

---

**Sample Agent.md**

Platform engineer agent can create this..

Here the details and the best practices we need to give link to appropriate confluence, team or github link so that it will be more realistic

then the consumer while building a microservice, he can download this agent.md file from IDP and use his specific development