Here is a more detailed spec file for a Spring Boot REST API, incorporating the requested coding standards and additional enterprise-grade best practices, referenced from the `Agent-template.md`.

---

# 🤖 Agent Protocol: Building Enterprise-Grade Java Microservices

This document provides a comprehensive guide for AI coding agents to build secure, scalable, and maintainable microservices using Java 21 and Spring Boot 3. Adherence to these principles is mandatory for all generated projects.

---

## 1. 🎯 Core Principles

-   **Security First**: Implement security controls at every layer. Assume a zero-trust environment. Conduct regular security reviews and static/dynamic analysis.
-   **Production Ready**: All code must be suitable for production, with robust configuration, logging, and error handling. Include health checks, metrics, and tracing.
-   **Developer Experience**: Ensure the project is easy to set up, run, and understand. Prioritize clear documentation, clean code, and fast feedback loops.
-   **Performance by Design**: Build for low latency and high throughput. Implement caching, efficient data access patterns, and asynchronous processing where appropriate.
-   **Observability**: The system's internal state must be inferable from its external outputs (logs, metrics, traces). Integrate with centralized monitoring solutions.
-   **API First**: Design APIs contract-first using OpenAPI specifications. Ensure consistency and intuitive consumption.
-   **Resilience**: Design for failure. Implement circuit breakers, retries, and bulkheads.

---

## 2. 🛠️ Standard Technology Stack

| Component            | Technology                             | Version/Standard                                                                                                                                                                                                                                                                         |
| :------------------- | :------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Language**         | Java                                   | **21**                                                                                                                                                                                                                                                                                 |
| **Framework**        | Spring Boot                            | **3.3+** (using Spring 6, Spring Framework 6, Jakarta EE 10)                                                                                                                                                                                                                             |
| **Build Tool**       | Maven                                  | 3.9+ (with `spring-boot-maven-plugin`, `maven-compiler-plugin`, `maven-surefire-plugin`, `jacoco-maven-plugin`, `maven-enforcer-plugin`)                                                                                                                                               |
| **Database**         | PostgreSQL (Prod), H2 (Dev/Test)       | Latest stable version (with `connection pooling` like HikariCP). Ensure robust connection string handling.                                                                                                                                                                             |
| **API Spec**         | OpenAPI                                | 3.0 (generated via SpringDoc OpenAPI)                                                                                                                                                                                                                                                  |
| **Container**        | Docker                                 | Latest (multi-stage builds, non-root users, minimal base images)                                                                                                                                                                                                                       |
| **Caching**          | Caffeine (local), Redis (distributed)  | Latest (ensure robust eviction policies and cache invalidation strategies)                                                                                                                                                                                                             |
| **Validation**       | Jakarta Bean Validation                | 3.0 (with Hibernate Validator implementation)                                                                                                                                                                                                                                          |
| **Testing**          | JUnit 5, Mockito, AssertJ              | Latest (Testcontainers for integration tests, Pact for CDC testing)                                                                                                                                                                                                                    |
| **Linting/Static Analysis** | SonarQube, Checkstyle, SpotBugs        | Integrate into CI/CD pipeline. Adhere to configured quality gates.                                                                                                                                                                                                       |
| **Logging**          | SLF4J with Logback                     | Structured logging (JSON format) with custom appenders for external log aggregation (e.g., ELK Stack, Splunk).                                                                                                                                                                         |
| **Monitoring/Tracing** | Micrometer, Prometheus, Grafana, OpenTelemetry | Automatically expose metrics via `/actuator/prometheus`. Integrate with distributed tracing systems (e.g., Jaeger, Zipkin).                                                                                                                                                            |
| **Messaging (Optional)** | Apache Kafka, RabbitMQ                 | Latest (for event-driven architectures, asynchronous processing, and inter-service communication).                                                                                                                                                                                     |
| **API Gateway (Optional)** | Spring Cloud Gateway                   | For centralized routing, security, and rate limiting in a microservice ecosystem.                                                                                                                                                                                                      |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using **Spring Initializr** (`start.spring.io`) with the following default dependencies. Ensure the project uses Maven and generates a JAR file.

-   Spring Web (for REST API)
-   Spring Data JPA (for database interaction)
-   Spring Boot Actuator (for monitoring and management endpoints)
-   Spring Cache (for caching abstractions)
-   Jakarta Bean Validation (for request payload validation)
-   Lombok (for reducing boilerplate code in DTOs/Entities, ensure correct setup with IDE plugins)
-   PostgreSQL Driver
-   H2 Database (for in-memory development/test database)
-   SpringDoc OpenAPI (`org.springdoc:springdoc-openapi-starter-webmvc-ui`) (for API documentation)
-   Testcontainers (`org.testcontainers:postgresql`, `org.testcontainers:junit-jupiter`) (for robust integration testing)

### Standard Maven `pom.xml` Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version> <!-- Use the latest stable Spring Boot 3.3.x -->
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.hcl.servicename</groupId>
    <artifactId>parcel-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>parcel-service</name>
    <description>Demo project for Spring Boot Parcel Service</description>

    <properties>
        <java.version>21</java.version>
        <springdoc-openapi.version>2.5.0</springdoc-openapi.version> <!-- Use the latest stable -->
        <testcontainers.version>1.19.8</testcontainers.version> <!-- Use the latest stable -->
        <jacoco.version>0.8.12</jacoco.version>
        <sonar.organization>your-org</sonar.organization>
        <sonar.host.url>https://sonarcloud.io</sonar.host.url>
    </properties>

    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <!-- Database Drivers -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!-- API Documentation -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>${springdoc-openapi.version}</version>
        </dependency>
        <!-- Caching -->
        <dependency>
            <groupId>com.github.ben-manes.caffeine</groupId>
            <artifactId>caffeine</artifactId>
        </dependency>
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- Testing Dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Add other optional dependencies here, e.g., Spring Security, Redis, Kafka -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok-mapstruct-binding</artifactId>
                            <version>0.2.0</version> <!-- If using MapStruct -->
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
            <!-- JaCoCo Plugin for Code Coverage -->
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>${jacoco.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <excludes>
                        <exclude>**/config/**</exclude>
                        <exclude>**/dto/**</exclude>
                        <exclude>**/model/**</exclude>
                        <exclude>**/exception/**</exclude>
                        <exclude>**/*Application.java</exclude>
                    </excludes>
                </configuration>
            </plugin>
            <!-- Maven Enforcer Plugin for build rules (e.g., forbidding certain dependencies) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <version>3.4.1</version>
                <executions>
                    <execution>
                        <id>enforce-maven-version</id>
                        <goals>
                            <goal>enforce</goal>
                        </goals>
                        <configuration>
                            <rules>
                                <requireMavenVersion>
                                    <version>[3.9.0,)</version>
                                </requireMavenVersion>
                                <requireJavaVersion>
                                    <version>${java.version}</version>
                                </requireJavaVersion>
                                <bannedDependencies>
                                    <excludes>
                                        <exclude>commons-logging</exclude>
                                    </excludes>
                                </bannedDependencies>
                            </rules>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <!-- SpotBugs Plugin for static analysis -->
            <plugin>
                <groupId>com.github.spotbugs</groupId>
                <artifactId>spotbugs-maven-plugin</artifactId>
                <version>4.8.5.0</version>
                <configuration>
                    <effort>Max</effort>
                    <threshold>Low</threshold>
                    <failOnError>true</failOnError>
                    <includeFilterFile>${project.basedir}/spotbugs-include.xml</includeFilterFile>
                    <excludeFilterFile>${project.basedir}/spotbugs-exclude.xml</excludeFilterFile>
                    <formats>
                        <format>HTML</format>
                        <format>XML</format>
                    </formats>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!-- Checkstyle Plugin for coding style enforcement -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>3.4.0</version>
                <configuration>
                    <configLocation>checkstyle.xml</configLocation>
                    <encoding>UTF-8</encoding>
                    <consoleOutput>true</consoleOutput>
                    <failsOnError>true</failsOnError>
                    <linkXRef>false</linkXRef>
                </configuration>
                <executions>
                    <execution>
                        <id>validate</id>
                        <phase>validate</phase>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

### Standard Package Structure

Organize all source code under a root package (e.g., `com.hcl.servicename`).

```
com/hcl/servicename/
├── ParcelServiceApplication.java   # Main application class, enable @EnableCaching, @EnableAspectJAutoProxy (if AOP used)
├── config/                         # @Configuration beans (Cache, Security, OpenAPI, DataSource, etc.)
│   ├── CacheConfig.java
│   ├── SecurityConfig.java         # If Spring Security is enabled
│   └── OpenApiConfig.java
├── controller/                     # REST controllers (API layer), handle HTTP requests
│   └── ParcelController.java
├── dto/                            # Data Transfer Objects for API requests/responses, immutable if possible
│   ├── ParcelRequest.java
│   ├── ParcelResponse.java
│   └── ErrorResponse.java          # Standardized error response DTO
├── model/                          # JPA Entities, represent database tables
│   ├── Parcel.java
│   └── AuditableEntity.java        # Base class for auditing fields
├── repository/                     # Spring Data JPA repositories, abstract data access
│   └── ParcelRepository.java
├── service/                        # Business logic, orchestrate data access and enforce rules
│   ├── ParcelService.java
│   └── MapperService.java          # Optional: for DTO-Entity mapping using MapStruct
└── exception/                      # Custom exceptions and global handler for consistent error responses
    ├── ResourceNotFoundException.java
    └── GlobalExceptionHandler.java
└── util/                           # Utility classes (e.g., validators, converters)
```

### `src/main/resources` Structure

```
src/main/resources/
├── application.properties          # Base application configuration
├── application-dev.properties      # Development-specific configuration
├── application-prod.properties     # Production-specific configuration
├── logback-spring.xml              # Logback configuration for structured logging
├── static/                         # Static web content (e.g., index.html for Swagger UI)
└── checkstyle.xml                  # Checkstyle configuration file
└── spotbugs-exclude.xml            # SpotBugs exclusion filter
└── spotbugs-include.xml            # SpotBugs inclusion filter
```

---

## 4. 📝 Development Workflow & Best Practices

### Step 1: API & DTO Design (Controller Layer)

1.  **Define DTOs**: Create separate Plain Old Java Objects (POJOs) for API requests and responses in the `dto` package. Use Jakarta Bean Validation annotations (`@NotBlank`, `@Size`, `@Email`, `@Pattern`, `@NotNull`, `@Min`, `@Max`, `@PastOrPresent`, `@FutureOrPresent`, etc.) for all incoming data.
    *   **Immutability**: Prefer immutable DTOs (using Java records or Lombok's `@Value` / `@Builder`).
    *   **Never expose JPA entities directly in the API.** Map entities to DTOs and vice-versa.
    *   **OpenAPI Annotations**: Use `io.swagger.v3.oas.annotations.media.Schema` for detailed schema descriptions, examples, and constraints.
2.  **Create Controller**: In the `controller` package, create a `@RestController`.
3.  **Map Endpoints**: Use `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`.
    *   **HTTP Methods Semantics**: Adhere to RESTful principles (e.g., POST for creation, PUT for full update, PATCH for partial update, DELETE for removal).
    *   **Idempotency**: Ensure PUT and DELETE operations are idempotent.
4.  **API Versioning**: Prefix all routes with `/api/v1`. Example: `@RequestMapping("/api/v1/parcels")`. Consider header or content negotiation versioning for more flexibility, but path versioning is simpler.
5.  **Input/Output**: Accept DTOs as `@RequestBody` (with `@Valid` for validation) and return `ResponseEntity<Dto>` for explicit control over HTTP status, headers, and body.
    *   **Response Statuses**: Return appropriate HTTP status codes (200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 500 Internal Server Error).
6.  **OpenAPI Documentation**: Use `io.swagger.v3.oas.annotations.Operation`, `io.swagger.v3.oas.annotations.responses.ApiResponse` to describe endpoints, parameters, and responses.

### Step 2: Business Logic (Service Layer)

1.  **Create Service**: In the `service` package, create a `@Service` class. Implement interfaces for easier testing and abstraction.
2.  **Inject Dependencies**: Use **constructor injection** for repositories and other services. Avoid `@Autowired` on fields.
3.  **Implement Logic**: Write the core business logic here. This layer is responsible for coordinating data access, calling other services, enforcing business rules, and handling complex workflows.
    *   **Single Responsibility Principle**: Each service method should ideally perform one well-defined task.
    *   **Domain Logic**: Encapsulate domain-specific rules and validations.
4.  **Use Transactions**: Use `@Transactional` (from `org.springframework.transaction.annotation`) for methods that modify data. Specify `propagation` and `rollbackFor` as needed. Use `@Transactional(readOnly = true)` for read operations to optimize performance and prevent accidental writes.
5.  **Map DTOs to Entities**: The service layer is responsible for converting incoming DTOs to JPA entities before passing them to the repository, and converting entities to DTOs before returning them to the controller. Use tools like **MapStruct** for efficient and type-safe mapping.
6.  **Logging**: Use `org.slf4j.Logger` for structured logging of business events, errors, and debugging information. Include correlation IDs.

### Step 3: Data Persistence (Repository & Model Layer)

1.  **Create Entity**: In the `model` package, define a JPA `@Entity`.
    *   Use `@Id`, `@GeneratedValue(strategy = GenerationType.IDENTITY)` for primary keys.
    *   Use appropriate `@Column` annotations for column names, nullability (`nullable = false`), and uniqueness (`unique = true`).
    *   Use `@Audited` (if using Hibernate Envers) or embed `AuditableEntity` for automatic population of `createdBy`, `createdDate`, `lastModifiedBy`, `lastModifiedDate` fields (using Spring Data JPA's `@EnableJpaAuditing` and `@EntityListeners(AuditingEntityListener.class)`).
    *   **Avoid Lombok's `@Data` on entities; prefer `@Getter`, `@Setter`, `@ToString` (with `@Exclude` on relations to prevent infinite loops), `@NoArgsConstructor`, `@AllArgsConstructor` specifically.** Avoid `equals()` and `hashCode()` on entities, especially before ID generation, or use `@EqualsAndHashCode.Exclude` on ID field.
    *   **Lazy Loading**: Configure relationships (`@OneToMany`, `@ManyToOne`, etc.) with `FetchType.LAZY` by default to avoid N+1 problems. Use `JOIN FETCH` or `@EntityGraph` for specific queries that require eager loading.
2.  **Create Repository**: In the `repository` package, create an interface that extends `JpaRepository<Entity, IdType>`.
    *   Add custom query methods following Spring Data JPA naming conventions (e.g., `findByStatusAndCreatedDateBetween`).
    *   Use `@Query` annotation for more complex JPQL or native SQL queries.
    *   Consider using `PagingAndSortingRepository` for pagination and sorting.

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in the `exception` package (e.g., `ResourceNotFoundException extends RuntimeException`, `InvalidInputException extends RuntimeException`, `ConflictException extends RuntimeException`).
2.  **Global Handler**: Create a `@RestControllerAdvice` class in the `exception` package.
3.  **Handle Exceptions**: Add `@ExceptionHandler` methods for:
    *   Custom exceptions, mapping them to appropriate HTTP status codes (e.g., `ResourceNotFoundException` -> 404 NOT_FOUND, `InvalidInputException` -> 400 BAD_REQUEST, `ConflictException` -> 409 CONFLICT).
    *   Validation exceptions (`MethodArgumentNotValidException`, `ConstraintViolationException`).
    *   Spring framework exceptions (e.g., `HttpMessageNotReadableException`).
    *   A fallback generic `Exception` handler for unexpected errors, returning 500 INTERNAL_SERVER_ERROR.
    *   Return a standardized `ErrorResponse` DTO with a clear message, error code, and timestamp.
    *   **Never expose stack traces or internal implementation details in error responses.** Log stack traces internally at ERROR level.

### Step 5: Caching

1.  **Enable Caching**: Add `@EnableCaching` to the main application class.
2.  **Configure Cache**: Create a `CacheConfig` class (in `config` package) to define cache managers (e.g., `CaffeineCacheManager` for local caching, `RedisCacheManager` for distributed caching). Define cache names and TTLs.
3.  **Apply Caching**: Use `@Cacheable`, `@CachePut`, and `@CacheEvict` on service methods (or repository methods for read-heavy entities) to cache results of expensive operations.
    *   Specify `cacheNames` and `key` using SpEL expressions (e.g., `key="#id"` or `key="#user.id"`).
    *   Consider `condition` and `unless` for conditional caching.
    *   Implement proper cache invalidation strategies (e.g., `@CacheEvict` on updates/deletes).

### Step 6: Configuration

-   **Base Config (`application.properties` or `application.yml`)**: Define common properties like `spring.application.name`, `server.port`. Prefer `.yml` for readability and hierarchical structure.
-   **Dev Profile (`application-dev.properties/.yml`)**: Configure H2 in-memory database, enable the H2 console (`spring.h2.console.enabled=true`, `spring.h2.console.path=/h2-console`), and set logging to `DEBUG`.
-   **Prod Profile (`application-prod.properties/.yml`)**: Configure PostgreSQL connection details. Use placeholders for secrets (e.g., `spring.datasource.password=${DB_PASSWORD}`). Disable detailed health info (`management.endpoint.health.show-details=never`). Disable H2 console.
-   **Externalized Configuration**: Utilize Spring Cloud Config, Kubernetes ConfigMaps/Secrets, or environment variables for sensitive and environment-specific properties. **Never commit secrets to version control.**
-   **Logging Configuration (`logback-spring.xml`)**: Define appenders (console, file), log levels for specific packages, and a structured JSON format for logs (e.g., using Logstash Encoder).

### Step 7: `catalog-info.yaml` File Structure

The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: parcel-service # required, unique name for the service (e.g., kebab-case)
  description: Manages parcel information and tracking, including creation, updates, and retrieval. # required, human-readable description
  tags:
    - java
    - spring-boot
    - microservice
    - rest-api
    - parcel-management
  annotations:
    backstage.io/techdocs-ref: url:https://github.com/hcl/parcel-service/blob/main/README.md # Link to README or techdocs
    # sonarqube.web.url: https://sonarcloud.io/dashboard?id=com.hcl.servicename:parcel-service # Link to SonarQube dashboard
    # jenkins.io/job-full-name: MyTeam/my-pipeline/parcel-service # Link to Jenkins job
    # prometheus.io/alert-description: High latency for parcel service # Example monitoring link
  links:
    - url: https://github.com/hcl/parcel-service # URL to the source code repository
      title: GitHub Repo
      icon: github
    - url: https://api.hcl.com/v1/parcel-service/swagger-ui.html # URL to the service's Swagger UI
      title: OpenAPI Docs
      icon: docs
    - url: https://confluence.hcl.com/display/DEV/Parcel+Service+Design # URL to a relevant design document
      title: Design Document
      icon: dashboard
spec:
  type: service # required, e.g., 'service', 'library', 'website'
  lifecycle: production # required, e.g., 'experimental', 'production', 'deprecated'
  owner: hcl-logistics-team # required, the owning team or user (e.g., defined in Backstage Groups)
  system: logistics-platform # optional, the larger system this component belongs to (e.g., 'logistics-platform')
  dependsOn:
    - resource:default/postgresql-parcel-db # Example database dependency
    - component:default/user-service # Example microservice dependency
  providesApis:
    - parcel-api # Reference to an API definition in an api-docs.yaml if available
```

---

### Step 8: Coding Standards

**Note**: The following sections are replaced with content directly fetched from the provided URLs.

---

## Java Spring Boot Coding Standards

1.  **Naming Conventions**:
    *   Packages: `com.company.project.module.submodule` (lowercase).
    *   Classes: `PascalCase` (e.g., `UserService`, `OrderController`).
    *   Interfaces: `PascalCase` (e.g., `UserRepository`, `PaymentGateway`).
    *   Methods: `camelCase` (e.g., `getUserById`, `processOrder`).
    *   Variables: `camelCase` (e.g., `userName`, `orderId`).
    *   Constants: `SCREAMING_SNAKE_CASE` (e.g., `MAX_RETRIES`).
    *   Enums: `PascalCase` for enum type, `SCREAMING_SNAKE_CASE` for enum values.
    *   Annotation attributes: `camelCase`.

2.  **Code Formatting**:
    *   Indentation: 4 spaces, no tabs.
    *   Line Length: Max 120 characters.
    *   Braces: K&R style (opening brace on the same line as the statement).
    *   Whitespace: Use judiciously for readability (e.g., around operators, after commas).
    *   Imports: Organize alphabetically, no wildcard imports (`*`).

3.  **Spring-Specific**:
    *   **Dependency Injection**: Prefer constructor injection over field injection (`@Autowired` on fields is discouraged).
    *   **Component Scopes**: Use `@Scope` judiciously; default to singleton.
    *   **Transactions**: Apply `@Transactional` at the service layer; be explicit about `readOnly` and `rollbackFor`.
    *   **REST Controllers**:
        *   Use `@RestController` and `@RequestMapping`.
        *   Return `ResponseEntity` for explicit control over HTTP status and headers.
        *   Validate input DTOs using `@Valid` and `@RequestBody`.
        *   Handle exceptions globally using `@ControllerAdvice` and `@ExceptionHandler`.
    *   **Data Access**:
        *   Leverage Spring Data JPA interfaces (`JpaRepository`).
        *   Avoid `N+1` query issues with `JOIN FETCH` or `@EntityGraph`.
        *   Separate JPA entities from DTOs.
    *   **Configuration**:
        *   Use `@Configuration` classes for application-wide bean definitions.
        *   Externalize configuration using `application.properties`/`application.yml` and environment variables.
        *   Group related configurations.

4.  **Error Handling**:
    *   Throw specific, custom exceptions for business rule violations.
    *   Catch and wrap low-level exceptions where appropriate.
    *   Provide meaningful error messages and HTTP status codes.

5.  **Logging**:
    *   Use SLF4J and Logback.
    *   Log at appropriate levels (DEBUG, INFO, WARN, ERROR).
    *   Avoid logging sensitive information.
    *   Include correlation IDs for tracing requests across microservices.

6.  **Concurrency**:
    *   Be mindful of thread safety in shared mutable state.
    *   Use concurrent collections and mechanisms where appropriate.

7.  **Security**:
    *   Sanitize all user input.
    *   Implement authentication and authorization (Spring Security).
    *   Protect against common vulnerabilities (OWASP Top 10).
    *   Encrypt sensitive data at rest and in transit.

8.  **Testing**:
    *   Write unit tests for isolated components.
    *   Write integration tests for component interactions and API endpoints.
    *   Use Mockito for mocking dependencies.
    *   Use Testcontainers for database and external service testing.

9.  **Comments & Documentation**:
    *   Use JavaDoc for public classes, interfaces, methods, and complex fields.
    *   Add inline comments for non-obvious code logic.
    *   Keep comments up-to-date with code changes.

## Python Coding Standards (PEP 8 Compliant)

1.  **Naming Conventions**:
    *   Modules: `lowercase_with_underscores`.
    *   Packages: `lowercase_with_underscores`.
    *   Classes: `CamelCase`.
    *   Functions/Methods: `lowercase_with_underscores`.
    *   Variables: `lowercase_with_underscores`.
    *   Constants: `UPPERCASE_WITH_UNDERSCORES`.
    *   Private variables/methods: Prefix with single underscore (`_private_var`).
    *   Magic methods: Double underscore prefix and suffix (`__init__`).

2.  **Code Formatting**:
    *   Indentation: 4 spaces.
    *   Line Length: Max 79 characters for code, 72 for docstrings.
    *   Blank Lines: Two blank lines between top-level functions/classes, one blank line between methods within a class.
    *   Whitespace: Avoid excessive whitespace. Use around operators, after commas.
    *   Imports: Group standard library, third-party, and local imports, separated by blank lines. Alphabetize within each group. Absolute imports preferred.

3.  **Docstrings**:
    *   Use docstrings for all public modules, classes, functions, and methods.
    *   Follow reStructuredText or Google style.

4.  **Error Handling**:
    *   Use `try...except` blocks to handle expected errors gracefully.
    *   Be specific in `except` clauses (avoid bare `except`).
    *   Raise specific exceptions.

5.  **Type Hinting**:
    *   Use type hints for function arguments and return values.
    *   Use `typing` module for complex types.

6.  **Logging**:
    *   Use the `logging` module.
    *   Configure loggers, handlers, and formatters.
    *   Log at appropriate levels.

7.  **Concurrency**:
    *   Understand GIL limitations for CPU-bound tasks.
    *   Use `asyncio` for I/O-bound concurrent operations.
    *   Use `threading` or `multiprocessing` for CPU-bound tasks if necessary.

8.  **Testing**:
    *   Use `unittest` or `pytest` for testing.
    *   Write clear, concise test cases.
    *   Aim for good test coverage.

9.  **Readability**:
    *   Write explicit code over implicit.
    *   Avoid clever one-liners that reduce readability.
    *   Use meaningful variable and function names.

## Angular Coding Standards

1.  **Naming Conventions**:
    *   **Folders**: `kebab-case` (e.g., `user-profile`, `shared-components`).
    *   **Components**: `kebab-case` for file names (e.g., `user-list.component.ts`), `PascalCase` for class names (`UserListComponent`), `kebab-case` for selectors (`app-user-list`).
    *   **Directives**: `kebab-case` for file names, `PascalCase` for class names, `camelCase` for selectors.
    *   **Services**: `kebab-case` for file names, `PascalCase` for class names.
    *   **Modules**: `kebab-case` for file names, `PascalCase` for class names.
    *   **Interfaces**: Prefix with `I` (e.g., `IUser`).
    *   **Pipes**: `kebab-case` for file names, `PascalCase` for class names, `camelCase` for pipe names.

2.  **Folder Structure**:
    *   Feature-driven: Group related components, services, and modules within a feature folder.
    *   `src/app/`:
        *   `core/`: Singletons, global services (e.g., authentication, error handling).
        *   `shared/`: Reusable components, directives, pipes, interfaces (e.g., `button`, `modal`).
        *   `features/`: Top-level feature modules (e.g., `users`, `products`).
        *   `environments/`: Environment-specific configurations.

3.  **Components**:
    *   **One Component Per File**: Adhere to the single responsibility principle.
    *   **Encapsulation**: Use `OnPush` change detection strategy for performance.
    *   **Lifecycle Hooks**: Implement `OnInit`, `OnDestroy`, etc., as needed.
    *   **Input/Output**: Use `@Input()` and `@Output()` for parent-child communication.
    *   **Templating**: Keep templates clean; move complex logic to the component class.

4.  **Services**:
    *   **Single Responsibility**: Each service should do one thing well.
    *   **Stateless by Default**: Prefer stateless services.
    *   **Injectable**: Use `@Injectable({ providedIn: 'root' })` for tree-shakable services.
    *   **Observable Pattern**: Use RxJS Observables for asynchronous operations.

5.  **Modules**:
    *   **Feature Modules**: Group related components, services, and routes.
    *   **Shared Module**: Import and re-export `CommonModule` and shared components.
    *   **Core Module**: Contain application-wide singletons.

6.  **Routing**:
    *   Define routes in feature modules.
    *   Use lazy loading for feature modules to improve initial load time.
    *   Implement route guards for authentication/authorization.

7.  **Error Handling**:
    *   Implement a global error handler for client-side errors.
    *   Use RxJS `catchError` operator for HTTP request errors.

8.  **State Management**:
    *   Use NgRx, Akita, or simple RxJS services for complex state management.

9.  **Styling**:
    *   Use SCSS or LESS.
    *   Encapsulate styles within components (`@Component` `styleUrls`).
    *   Follow BEM or similar methodologies for CSS class naming.

10. **Testing**:
    *   Write unit tests for components, services, pipes, and directives.
    *   Use `TestBed` for component testing.
    *   Test asynchronous operations using `async` and `fakeAsync`.

11. **Performance**:
    *   Lazy load modules.
    *   Use `OnPush` change detection.
    *   TrackBy function for `*ngFor` loops.
    *   Optimize images.

12. **Accessibility (a11y)**:
    *   Ensure semantic HTML.
    *   Use ARIA attributes where necessary.

13. **Linting & Formatting**:
    *   Use TSLint (or ESLint with TypeScript support) and Prettier.
    *   Adhere to Angular CLI defaults where appropriate.

---

### Step 9: Documentation

-   **OpenAPI**: Configure a `OpenApiConfig` bean to set the API title, version, description, license information, contact details, and server URLs. Enable Swagger UI (`/swagger-ui.html`) and Swagger JSON (`/v3/api-docs`).
-   **JavaDoc**: Add JavaDoc comments to all public classes, interfaces, methods, and complex fields in controllers, services, repositories, and DTOs/models, explaining their purpose, parameters, return values, and exceptions.
-   **README.md**: Provide a comprehensive `README.md` at the project root, including:
    *   Project description.
    *   Setup instructions (prerequisites, build, run).
    *   API documentation link.
    *   Development guidelines.
    *   Contribution guidelines.
    *   Tech stack overview.

### Step 10: Testing

-   **Unit Tests**: Test services and controllers by mocking their dependencies (e.g., repositories, other services) using Mockito. Focus on business logic. Place tests in `src/test/java` in corresponding package structure.
-   **Integration Tests**: Use `@SpringBootTest` and `@ActiveProfiles("test")` to test the full application flow, from controller to database.
    *   **Database Testing**: Use **Testcontainers** (with JUnit 5 extension) for spinning up real PostgreSQL database instances in Docker for integration tests, ensuring realistic environment.
    *   **Web Layer Testing**: Use `@WebMvcTest` for testing controllers in isolation with mocked service layer.
    *   **Data JPA Testing**: Use `@DataJpaTest` for testing repository layer with an in-memory database or Testcontainers.
-   **Consumer-Driven Contract (CDC) Testing**: Implement Pact for defining and verifying API contracts between microservices.
-   **Coverage**: Aim for a minimum of 80% test coverage for critical business logic. Use JaCoCo Maven plugin to generate coverage reports.
-   **Performance Testing**: Use tools like JMeter or Gatling for load and stress testing.
-   **Security Testing**: Integrate OWASP ZAP or similar tools into CI/CD for automated security scans.

---

## 5. 🐳 Containerization & Deployment

### Dockerfile

Create a multi-stage `Dockerfile` in the project root:

1.  **Build Stage**: Use a `maven:3.9-eclipse-temurin-21` base image to build the application JAR. Copy source code, build with `mvn clean install -DskipTests`.
2.  **Runtime Stage**: Use a minimal JRE image (e.g., `eclipse-temurin:21-jre-alpine` or `distroless/java21-debian12`).
3.  **Security**:
    *   Create and run as a non-root user (`USER spring`).
    *   Drop unnecessary capabilities.
    *   Copy only the necessary JAR and resources.
4.  **Health Check**: Add a `HEALTHCHECK` instruction that calls the `/actuator/health/readiness` and `/actuator/health/liveness` endpoints to ensure application responsiveness.
    ```dockerfile
    HEALTHCHECK --interval=30s --timeout=10s --retries=5 CMD curl -f http://localhost:8080/actuator/health || exit 1
    ```
5.  **Execution**: Use `ENTRYPOINT ["java", "-jar", "/app/app.jar"]`.
6.  **Environment Variables**: Define `JAVA_TOOL_OPTIONS` for JVM tuning (e.g., memory limits `Xmx`, `Xms`) or other runtime options.

### Environment Variables

All secrets (database passwords, API keys, third-party service credentials) **must** be supplied via environment variables, not hardcoded in properties files. Use Spring Boot's externalized configuration and `application-prod.yml` placeholders (e.g., `${DB_PASSWORD}`).

### Kubernetes Deployment (Optional, but Recommended for Production)

-   **Deployment Manifests**: Provide Kubernetes `Deployment`, `Service`, `Ingress`, `ConfigMap`, and `Secret` manifests.
-   **Readiness/Liveness Probes**: Configure `readinessProbe` and `livenessProbe` using `/actuator/health/readiness` and `/actuator/health/liveness` endpoints respectively.
-   **Resource Limits**: Define CPU and memory requests and limits to prevent resource exhaustion.
-   **Helm Charts**: For complex deployments, provide Helm charts for easy packaging and deployment to Kubernetes clusters.
-   **Observability**: Ensure proper integration with Kubernetes native monitoring (Prometheus, Grafana) and centralized logging.

---

This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.