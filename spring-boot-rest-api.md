# 🤖 Agent Protocol: Building Enterprise-Grade Java Microservices for Product Catalog Service

This document provides a comprehensive guide for AI coding agents to build a secure, scalable, and maintainable **Product Catalog Microservice** using Java 21 and Spring Boot 3. Adherence to these principles is mandatory for all generated projects.

---

## 1. 🎯 Core Principles

-   **Security First**: Implement security controls at every layer. Assume a zero-trust environment.
-   **Production Ready**: All code must be suitable for production, with robust configuration, logging, and error handling.
-   **Developer Experience**: Ensure the project is easy to set up, run, and understand. Prioritize clear documentation and clean code.
-   **Performance by Design**: Build for low latency and high throughput. Implement caching and efficient data access patterns.
-   **Observability**: The system's internal state must be inferable from its external outputs (logs, metrics, traces).

---

## 2. 🛠️ Standard Technology Stack

| Component | Technology | Version/Standard |
| :-------- | :--------- | :--------------- |
| **Language** | Java | **21** |
| **Framework** | Spring Boot | **3.3+** |
| **Build Tool** | Maven | 3.9+ |
| **Database** | PostgreSQL (Prod), H2 (Dev/Test) | Latest |
| **API Spec** | OpenAPI | 3.0 |
| **Container** | Docker | Latest |
| **Caching** | Caffeine (local), Redis (distributed) | Latest |
| **Validation** | Jakarta Bean Validation | 3.0 |
| **Testing** | JUnit 5, Mockito, Testcontainers | Latest |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using **Spring Initializr** (`https://start.spring.io/`) with the following default dependencies for a Product Catalog Service:

-   Spring Web
-   Spring Data JPA
-   Spring Boot Actuator
-   Spring Cache Abstraction
-   Jakarta Bean Validation
-   Lombok
-   PostgreSQL Driver
-   H2 Database
-   SpringDoc OpenAPI (`org.springdoc:springdoc-openapi-starter-webmvc-ui`)

### Standard Package Structure

Organize all source code under a root package (e.g., `com.hcl.productcatalog`).

```
com/hcl/productcatalog/
├── ProductCatalogApplication.java   # Main application class
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

1.  **Define DTOs**: Create separate Plain Old Java Objects (POJOs) for API requests and responses in the `dto` package.
    *   `ProductRequestDTO.java`: For creating and updating products.
    *   `ProductResponseDTO.java`: For returning product details.
    *   Use Jakarta Bean Validation annotations (`@NotBlank`, `@DecimalMin`, `@Min`, `@Size`, etc.) for all incoming data.
    *   **Never expose JPA entities directly in the API.**

    **Example `ProductRequestDTO.java`:**
    ```java
    package com.hcl.productcatalog.dto;

    import jakarta.validation.constraints.DecimalMin;
    import jakarta.validation.constraints.Min;
    import jakarta.validation.constraints.NotBlank;
    import jakarta.validation.constraints.NotNull;
    import jakarta.validation.constraints.Size;
    import lombok.AllArgsConstructor;
    import lombok.Getter;
    import lombok.NoArgsConstructor;
    import lombok.Setter;

    import java.math.BigDecimal;

    @Getter
    @Setter
    @NoArgsConstructor
    @AllArgsConstructor
    public class ProductRequestDTO {

        @NotBlank(message = "Product name cannot be empty")
        @Size(min = 2, max = 100, message = "Product name must be between 2 and 100 characters")
        private String name;

        @Size(max = 500, message = "Product description cannot exceed 500 characters")
        private String description;

        @NotNull(message = "Product price cannot be null")
        @DecimalMin(value = "0.01", message = "Product price must be greater than 0")
        private BigDecimal price;

        @NotNull(message = "Product stock cannot be null")
        @Min(value = 0, message = "Product stock cannot be negative")
        private Integer stock;

        @NotBlank(message = "Product category cannot be empty")
        @Size(min = 2, max = 50, message = "Product category must be between 2 and 50 characters")
        private String category;
    }
    ```

    **Example `ProductResponseDTO.java`:**
    ```java
    package com.hcl.productcatalog.dto;

    import lombok.AllArgsConstructor;
    import lombok.Getter;
    import lombok.NoArgsConstructor;
    import lombok.Setter;

    import java.math.BigDecimal;

    @Getter
    @Setter
    @NoArgsConstructor
    @AllArgsConstructor
    public class ProductResponseDTO {
        private Long id;
        private String name;
        private String description;
        private BigDecimal price;
        private Integer stock;
        private String category;
    }
    ```

2.  **Create Controller**: In the `controller` package, create a `@RestController` (e.g., `ProductController`).
3.  **Map Endpoints**: Use `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`.
4.  **API Versioning**: Prefix all routes with `/api/v1`. Example: `@RequestMapping("/api/v1/products")`.
5.  **Input/Output**: Accept DTOs as `@RequestBody` and return `ResponseEntity<Dto>`.

    **Example `ProductController.java`:**
    ```java
    package com.hcl.productcatalog.controller;

    import com.hcl.productcatalog.dto.ProductRequestDTO;
    import com.hcl.productcatalog.dto.ProductResponseDTO;
    import com.hcl.productcatalog.service.ProductService;
    import jakarta.validation.Valid;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.*;

    import java.util.List;

    @RestController
    @RequestMapping("/api/v1/products")
    public class ProductController {

        private final ProductService productService;

        public ProductController(ProductService productService) {
            this.productService = productService;
        }

        @PostMapping
        public ResponseEntity<ProductResponseDTO> createProduct(@Valid @RequestBody ProductRequestDTO productRequestDTO) {
            ProductResponseDTO createdProduct = productService.createProduct(productRequestDTO);
            return new ResponseEntity<>(createdProduct, HttpStatus.CREATED);
        }

        @GetMapping("/{id}")
        public ResponseEntity<ProductResponseDTO> getProductById(@PathVariable Long id) {
            ProductResponseDTO product = productService.getProductById(id);
            return ResponseEntity.ok(product);
        }

        @GetMapping
        public ResponseEntity<List<ProductResponseDTO>> getAllProducts() {
            List<ProductResponseDTO> products = productService.getAllProducts();
            return ResponseEntity.ok(products);
        }

        @PutMapping("/{id}")
        public ResponseEntity<ProductResponseDTO> updateProduct(@PathVariable Long id, @Valid @RequestBody ProductRequestDTO productRequestDTO) {
            ProductResponseDTO updatedProduct = productService.updateProduct(id, productRequestDTO);
            return ResponseEntity.ok(updatedProduct);
        }

        @DeleteMapping("/{id}")
        public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
            productService.deleteProduct(id);
            return ResponseEntity.noContent().build();
        }
    }
    ```

### Step 2: Business Logic (Service Layer)

1.  **Create Service**: In the `service` package, create a `@Service` class (e.g., `ProductService`).
2.  **Inject Dependencies**: Use constructor injection for repositories and other services.
3.  **Implement Logic**: Write the core business logic here (e.g., validations beyond DTO annotations, complex calculations, orchestration). This layer is responsible for coordinating data access, calling other services, and enforcing business rules.
4.  **Use Transactions**: Use `@Transactional` for methods that modify data. Use `@Transactional(readOnly = true)` for read operations to optimize performance.
5.  **Map DTOs to Entities**: The service layer is responsible for converting incoming DTOs to JPA entities before passing them to the repository, and converting entities to DTOs before returning them to the controller. A dedicated `ProductMapper` utility class can be used for this.

    **Example `ProductService.java`:**
    ```java
    package com.hcl.productcatalog.service;

    import com.hcl.productcatalog.dto.ProductRequestDTO;
    import com.hcl.productcatalog.dto.ProductResponseDTO;
    import com.hcl.productcatalog.exception.ProductNotFoundException;
    import com.hcl.productcatalog.model.Product;
    import com.hcl.productcatalog.repository.ProductRepository;
    import org.springframework.cache.annotation.CacheEvict;
    import org.springframework.cache.annotation.CachePut;
    import org.springframework.cache.annotation.Cacheable;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import java.util.List;
    import java.util.stream.Collectors;

    @Service
    public class ProductService {

        private final ProductRepository productRepository;

        public ProductService(ProductRepository productRepository) {
            this.productRepository = productRepository;
        }

        @Transactional
        public ProductResponseDTO createProduct(ProductRequestDTO requestDTO) {
            Product product = new Product();
            product.setName(requestDTO.getName());
            product.setDescription(requestDTO.getDescription());
            product.setPrice(requestDTO.getPrice());
            product.setStock(requestDTO.getStock());
            product.setCategory(requestDTO.getCategory());
            Product savedProduct = productRepository.save(product);
            return convertToResponseDTO(savedProduct);
        }

        @Cacheable(value = "products", key = "#id")
        @Transactional(readOnly = true)
        public ProductResponseDTO getProductById(Long id) {
            Product product = productRepository.findById(id)
                    .orElseThrow(() -> new ProductNotFoundException("Product not found with id: " + id));
            return convertToResponseDTO(product);
        }

        @Transactional(readOnly = true)
        public List<ProductResponseDTO> getAllProducts() {
            return productRepository.findAll().stream()
                    .map(this::convertToResponseDTO)
                    .collect(Collectors.toList());
        }

        @CachePut(value = "products", key = "#id")
        @Transactional
        public ProductResponseDTO updateProduct(Long id, ProductRequestDTO requestDTO) {
            Product existingProduct = productRepository.findById(id)
                    .orElseThrow(() -> new ProductNotFoundException("Product not found with id: " + id));

            existingProduct.setName(requestDTO.getName());
            existingProduct.setDescription(requestDTO.getDescription());
            existingProduct.setPrice(requestDTO.getPrice());
            existingProduct.setStock(requestDTO.getStock());
            existingProduct.setCategory(requestDTO.getCategory());

            Product updatedProduct = productRepository.save(existingProduct);
            return convertToResponseDTO(updatedProduct);
        }

        @CacheEvict(value = "products", key = "#id")
        @Transactional
        public void deleteProduct(Long id) {
            if (!productRepository.existsById(id)) {
                throw new ProductNotFoundException("Product not found with id: " + id);
            }
            productRepository.deleteById(id);
        }

        private ProductResponseDTO convertToResponseDTO(Product product) {
            ProductResponseDTO dto = new ProductResponseDTO();
            dto.setId(product.getId());
            dto.setName(product.getName());
            dto.setDescription(product.getDescription());
            dto.setPrice(product.getPrice());
            dto.setStock(product.getStock());
            dto.setCategory(product.getCategory());
            return dto;
        }
    }
    ```

### Step 3: Data Persistence (Repository & Model Layer)

1.  **Create Entity**: In the `model` package, define a JPA `@Entity` (e.g., `Product`).
    *   Use `@Id`, `@GeneratedValue`, `@Table`, and appropriate `@Column` annotations.
    *   Avoid Lombok's `@Data` on entities; prefer `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode(onlyExplicitlyIncluded = true)` for safety with JPA.

    **Example `Product.java`:**
    ```java
    package com.hcl.productcatalog.model;

    import jakarta.persistence.*;
    import lombok.Getter;
    import lombok.Setter;
    import lombok.ToString;
    import lombok.EqualsAndHashCode;
    import lombok.NoArgsConstructor;
    import lombok.AllArgsConstructor;

    import java.math.BigDecimal;

    @Entity
    @Table(name = "products")
    @Getter
    @Setter
    @ToString
    @EqualsAndHashCode(onlyExplicitlyIncluded = true) // Important for JPA entities
    @NoArgsConstructor
    @AllArgsConstructor
    public class Product {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        @EqualsAndHashCode.Include // Include ID in equals/hashCode
        private Long id;

        @Column(nullable = false, length = 100)
        private String name;

        @Column(length = 500)
        private String description;

        @Column(nullable = false, precision = 10, scale = 2)
        private BigDecimal price;

        @Column(nullable = false)
        private Integer stock;

        @Column(nullable = false, length = 50)
        private String category;
    }
    ```

2.  **Create Repository**: In the `repository` package, create an interface that extends `JpaRepository<Entity, IdType>` (e.g., `ProductRepository`). Add custom query methods as needed.

    **Example `ProductRepository.java`:**
    ```java
    package com.hcl.productcatalog.repository;

    import com.hcl.productcatalog.model.Product;
    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.stereotype.Repository;

    import java.util.List;

    @Repository
    public interface ProductRepository extends JpaRepository<Product, Long> {
        // Example custom query methods
        List<Product> findByCategory(String category);
        List<Product> findByNameContainingIgnoreCase(String name);
    }
    ```

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in the `exception` package (e.g., `ProductNotFoundException extends RuntimeException`).

    **Example `ProductNotFoundException.java`:**
    ```java
    package com.hcl.productcatalog.exception;

    import org.springframework.http.HttpStatus;
    import org.springframework.web.bind.annotation.ResponseStatus;

    @ResponseStatus(HttpStatus.NOT_FOUND) // Sets the HTTP status code
    public class ProductNotFoundException extends RuntimeException {
        public ProductNotFoundException(String message) {
            super(message);
        }
    }
    ```

2.  **Global Handler**: Create a `@RestControllerAdvice` class in the `exception` package (e.g., `GlobalExceptionHandler`).
3.  **Handle Exceptions**: Add `@ExceptionHandler` methods for custom exceptions, validation exceptions (`MethodArgumentNotValidException`), and a fallback generic `Exception`. Return a standardized `ErrorResponse` DTO with a message and HTTP status. **Never expose stack traces.**

    **Example `ErrorResponse.java` (DTO for error messages):**
    ```java
    package com.hcl.productcatalog.exception;

    import lombok.AllArgsConstructor;
    import lombok.Getter;
    import lombok.Setter;

    import java.time.LocalDateTime;

    @Getter
    @Setter
    @AllArgsConstructor
    public class ErrorResponse {
        private LocalDateTime timestamp;
        private int status;
        private String error;
        private String message;
        private String path;
    }
    ```

    **Example `GlobalExceptionHandler.java`:**
    ```java
    package com.hcl.productcatalog.exception;

    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.context.request.WebRequest;

    import java.time.LocalDateTime;
    import java.util.stream.Collectors;

    @ControllerAdvice
    public class GlobalExceptionHandler {

        @ExceptionHandler(ProductNotFoundException.class)
        public ResponseEntity<ErrorResponse> handleProductNotFoundException(
                ProductNotFoundException ex, WebRequest request) {
            ErrorResponse errorResponse = new ErrorResponse(
                    LocalDateTime.now(),
                    HttpStatus.NOT_FOUND.value(),
                    HttpStatus.NOT_FOUND.getReasonPhrase(),
                    ex.getMessage(),
                    request.getDescription(false)
            );
            return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
        }

        @ExceptionHandler(MethodArgumentNotValidException.class)
        public ResponseEntity<ErrorResponse> handleValidationExceptions(
                MethodArgumentNotValidException ex, WebRequest request) {
            String errorMessage = ex.getBindingResult().getFieldErrors().stream()
                    .map(error -> error.getField() + ": " + error.getDefaultMessage())
                    .collect(Collectors.joining(", "));

            ErrorResponse errorResponse = new ErrorResponse(
                    LocalDateTime.now(),
                    HttpStatus.BAD_REQUEST.value(),
                    HttpStatus.BAD_REQUEST.getReasonPhrase(),
                    "Validation failed: " + errorMessage,
                    request.getDescription(false)
            );
            return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
        }

        @ExceptionHandler(Exception.class)
        public ResponseEntity<ErrorResponse> handleAllUncaughtException(
                Exception ex, WebRequest request) {
            ErrorResponse errorResponse = new ErrorResponse(
                    LocalDateTime.now(),
                    HttpStatus.INTERNAL_SERVER_ERROR.value(),
                    HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase(),
                    "An unexpected error occurred: " + ex.getMessage(), // Avoid exposing internal details in production
                    request.getDescription(false)
            );
            return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
    ```

### Step 5: Caching

1.  **Enable Caching**: Add `@EnableCaching` to the main application class (`ProductCatalogApplication.java`).
2.  **Configure Cache**: Create a `CacheConfig` class (in `config` package) to define cache managers (e.g., Caffeine for local, Redis for distributed).
    *   For this example, a simple Caffeine setup is assumed.

    **Example `CacheConfig.java`:**
    ```java
    package com.hcl.productcatalog.config;

    import com.github.benmanes.caffeine.cache.Caffeine;
    import org.springframework.cache.CacheManager;
    import org.springframework.cache.annotation.EnableCaching;
    import org.springframework.cache.caffeine.CaffeineCacheManager;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    import java.util.concurrent.TimeUnit;

    @Configuration
    @EnableCaching
    public class CacheConfig {

        @Bean
        public CacheManager cacheManager() {
            CaffeineCacheManager cacheManager = new CaffeineCacheManager("products"); // Define cache name
            cacheManager.setCaffeine(caffeineCacheBuilder());
            return cacheManager;
        }

        Caffeine<Object, Object> caffeineCacheBuilder() {
            return Caffeine.newBuilder()
                    .initialCapacity(100)
                    .maximumSize(500)
                    .expireAfterAccess(10, TimeUnit.MINUTES)
                    .weakKeys()
                    .recordStats();
        }
    }
    ```

3.  **Apply Caching**: Use `@Cacheable`, `@CachePut`, and `@CacheEvict` on service methods (as shown in `ProductService.java` above) to cache results of expensive operations.
    *   `@Cacheable(value = "products", key = "#id")`: For `getProductById`.
    *   `@CachePut(value = "products", key = "#id")`: For `updateProduct`.
    *   `@CacheEvict(value = "products", key = "#id")`: For `deleteProduct`.

### Step 6: Configuration

-   **Base Config (`application.properties`)**: Define common properties like `spring.application.name`.
    ```properties
    spring.application.name=product-catalog-service
    server.port=8080

    # Actuator
    management.endpoints.web.exposure.include=health,info,prometheus
    management.endpoint.health.show-details=always
    ```
-   **Dev Profile (`application-dev.properties`)**: Configure H2 in-memory database, enable the H2 console, and set logging to `DEBUG`.
    ```properties
    spring.datasource.url=jdbc:h2:mem:productdb;DB_CLOSE_DELAY=-1
    spring.datasource.driverClassName=org.h2.Driver
    spring.datasource.username=sa
    spring.datasource.password=
    spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
    spring.h2.console.enabled=true
    spring.h2.console.path=/h2-console

    spring.jpa.hibernate.ddl-auto=update # For development, consider 'create-drop' or 'update'
    spring.jpa.show-sql=true
    spring.jpa.properties.hibernate.format_sql=true

    logging.level.com.hcl.productcatalog=DEBUG
    ```
-   **Prod Profile (`application-prod.properties`)**: Configure PostgreSQL connection details. Use placeholders for secrets (e.g., `${DB_PASSWORD}`). Disable detailed health info (`management.endpoint.health.show-details=never`).
    ```properties
    spring.datasource.url=jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
    spring.datasource.username=${DB_USERNAME}
    spring.datasource.password=${DB_PASSWORD}
    spring.datasource.driver-class-name=org.postgresql.Driver
    spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect

    spring.jpa.hibernate.ddl-auto=validate # Use 'validate' or 'none' in production
    spring.jpa.show-sql=false

    management.endpoint.health.show-details=never
    logging.level.com.hcl.productcatalog=INFO
    ```

### Step 7: `catalog-info.yaml` File Structure

The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: product-catalog-service # required, unique name
  description: Manages product information including creation, retrieval, updates, and deletion. # required, human-readable description
  tags:
    - springboot
    - java
    - microservice
    - product
    - catalog
  annotations:
    # Example: link to SonarQube, Prometheus dashboard
    sonarqube.org/project-key: hcl-product-catalog-service
    prometheus.io/port: '8080'
    prometheus.io/path: '/actuator/prometheus'
  links:
    - url: https://github.com/hcl/product-catalog-service # URL to the GitHub repository
      title: GitHub Repo
      icon: code
    - url: https://product-catalog-service.hcl.com/api/v1/swagger-ui.html # URL to the deployed Swagger UI
      title: API Documentation
      icon: docs
spec:
  type: service # required, e.g., 'service', 'library'
  lifecycle: production # required, e.g., 'experimental', 'production'
  owner: hcl-product-team # required, the owning team or user
  system: ecommerce-platform # optional, the larger system this component belongs to
  dependsOn:
    - resource:database/product-db # Example: depends on a PostgreSQL database resource
    - component:authentication-service # Example: depends on an authentication service
  providesApis:
    - product-catalog-api # Defines an API provided by this service
```

### Step 8: #Coding standards#

The following coding standards are mandatory for the Spring Boot Product Catalog Service. For the full and latest set of rules, please refer to the official repository: [Spring Boot Rules](https://github.com/alexstephen-github/CodingStandards/blob/main/springboot-rules.txt).

```
# Spring Boot Coding Standards
 
## General Java Standards
* Follow Oracle Java Code Conventions (indentation, naming, formatting).
* Use meaningful names for classes, methods, variables (e.g., `calculateTotalPrice` instead of `calc`).
* Avoid magic numbers/strings; use constants or enums.
* Write concise methods; single responsibility principle.
* Use `@Override` annotation for overridden methods.
* Prefer immutability where possible (e.g., DTOs).
* Handle exceptions gracefully; avoid catching `Exception` directly without specific handling.
* Use JavaDoc for public classes and methods.
* Keep class fields private and provide getters/setters.
 
## Spring Boot Specific Standards
 
### Configuration
* Use `application.properties` or `application.yml` for configuration.
* Externalize sensitive configurations using environment variables or Spring Cloud Config.
* Define application-specific properties in separate configuration classes using `@ConfigurationProperties`.
* Prefer constructor injection over field injection for dependencies (`@Autowired` on constructor).
 
### Controllers
* Annotate REST controllers with `@RestController`.
* Map endpoints clearly using `@RequestMapping`, `@GetMapping`, `@PostMapping`, etc.
* Use `@RequestBody` and `@ResponseBody` for JSON payload handling.
* Validate incoming requests using Jakarta Bean Validation (`@Valid` on DTOs).
* Return `ResponseEntity<?>` for explicit control over HTTP status codes.
* Handle exceptions globally using `@ControllerAdvice` and `@ExceptionHandler`.
* Keep controllers thin; delegate business logic to service layer.
 
### Services
* Annotate service classes with `@Service`.
* Encapsulate business logic within service methods.
* Use `@Transactional` for methods that modify the database, ensuring atomicity.
* Follow the Single Responsibility Principle for service classes and methods.
* Map DTOs to entities and vice versa within the service layer (or dedicated mappers).
 
### Repositories
* Extend Spring Data JPA interfaces (`JpaRepository`, `CrudRepository`).
* Define custom queries using method names or `@Query` annotation.
* Avoid complex business logic in repositories; keep them focused on data access.
 
### DTOs and Entities
* Use separate DTOs for request/response payloads; never expose entities directly.
* Use Lombok annotations (`@Getter`, `@Setter`, `@NoArgsConstructor`, `@AllArgsConstructor`) for DTOs.
* For JPA entities, prefer explicit getters/setters over `@Data` to avoid issues with `equals`/`hashCode` and lazy loading.
* Use Jakarta Bean Validation annotations (`@NotNull`, `@Size`, `@Pattern`) on DTO fields.
 
### Logging
* Use SLF4J and Logback for logging.
* Configure logging levels (e.g., `INFO` for production, `DEBUG` for development).
* Log meaningful messages with appropriate log levels. Avoid logging sensitive information.
 
### Testing
* Write unit tests for services, controllers, and utilities.
* Use `@SpringBootTest` for integration tests.
* Employ Mockito for mocking dependencies in unit tests.
* Use Testcontainers for database integration tests.
* Aim for high test coverage.
 
### Security
* Enable Spring Security for authentication and authorization.
* Implement JWT, OAuth2, or other appropriate security mechanisms.
* Secure endpoints using `@PreAuthorize` or request matchers.
* Sanitize all user inputs to prevent injection attacks.
* Implement rate limiting and other protective measures.
```

### Step 9: Documentation

-   **OpenAPI**: Configure an `OpenApiConfig` bean (in `config` package) to set the API title, version, and description. This will automatically generate Swagger UI at `/swagger-ui.html`.

    **Example `OpenApiConfig.java`:**
    ```java
    package com.hcl.productcatalog.config;

    import io.swagger.v3.oas.models.ExternalDocumentation;
    import io.swagger.v3.oas.models.OpenAPI;
    import io.swagger.v3.oas.models.info.Info;
    import io.swagger.v3.oas.models.info.License;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class OpenApiConfig {

        @Bean
        public OpenAPI productCatalogOpenAPI() {
            return new OpenAPI()
                    .info(new Info().title("Product Catalog API")
                            .description("API for managing product information in the HCL E-commerce Platform.")
                            .version("v1.0.0")
                            .license(new License().name("Apache 2.0").url("http://springdoc.org")))
                    .externalDocs(new ExternalDocumentation()
                            .description("Product Catalog Service Wiki Documentation")
                            .url("https://hcl.atlassian.net/wiki/spaces/DOCS/pages/product-catalog-service-documentation"));
        }
    }
    ```

-   **JavaDoc**: Add JavaDoc comments to all public classes and methods in controllers, services, repositories, and DTOs/Entities, explaining their purpose, parameters, and return values.

    **Example JavaDoc for `ProductService.java` method:**
    ```java
    /**
     * Retrieves a product by its unique identifier.
     * The result is cached for subsequent requests.
     *
     * @param id The unique identifier of the product.
     * @return A {@link ProductResponseDTO} containing the product details.
     * @throws ProductNotFoundException if no product is found with the given ID.
     */
    @Cacheable(value = "products", key = "#id")
    @Transactional(readOnly = true)
    public ProductResponseDTO getProductById(Long id) {
        // ... implementation ...
    }
    ```

### Step 10: Testing

-   **Unit Tests**: Test services and controllers by mocking their dependencies (e.g., repositories). Place tests in `src/test/java` under the corresponding package structure. Use Mockito for mocking.

    **Example: `ProductServiceTest.java` (Unit Test)**
    ```java
    package com.hcl.productcatalog.service;

    import com.hcl.productcatalog.dto.ProductRequestDTO;
    import com.hcl.productcatalog.dto.ProductResponseDTO;
    import com.hcl.productcatalog.exception.ProductNotFoundException;
    import com.hcl.productcatalog.model.Product;
    import com.hcl.productcatalog.repository.ProductRepository;
    import org.junit.jupiter.api.BeforeEach;
    import org.junit.jupiter.api.Test;
    import org.junit.jupiter.api.extension.ExtendWith;
    import org.mockito.InjectMocks;
    import org.mockito.Mock;
    import org.mockito.junit.jupiter.MockitoExtension;

    import java.math.BigDecimal;
    import java.util.Arrays;
    import java.util.List;
    import java.util.Optional;

    import static org.junit.jupiter.api.Assertions.*;
    import static org.mockito.ArgumentMatchers.any;
    import static org.mockito.Mockito.*;

    @ExtendWith(MockitoExtension.class)
    class ProductServiceTest {

        @Mock
        private ProductRepository productRepository;

        @InjectMocks
        private ProductService productService;

        private Product product;
        private ProductRequestDTO productRequestDTO;
        private ProductResponseDTO productResponseDTO;

        @BeforeEach
        void setUp() {
            product = new Product(1L, "Laptop", "High performance laptop", new BigDecimal("1200.00"), 10, "Electronics");
            productRequestDTO = new ProductRequestDTO("Laptop", "High performance laptop", new BigDecimal("1200.00"), 10, "Electronics");
            productResponseDTO = new ProductResponseDTO(1L, "Laptop", "High performance laptop", new BigDecimal("1200.00"), 10, "Electronics");
        }

        @Test
        void createProduct_Success() {
            when(productRepository.save(any(Product.class))).thenReturn(product);
            ProductResponseDTO result = productService.createProduct(productRequestDTO);
            assertNotNull(result);
            assertEquals(product.getName(), result.getName());
            verify(productRepository, times(1)).save(any(Product.class));
        }

        @Test
        void getProductById_Found() {
            when(productRepository.findById(1L)).thenReturn(Optional.of(product));
            ProductResponseDTO result = productService.getProductById(1L);
            assertNotNull(result);
            assertEquals(product.getName(), result.getName());
            verify(productRepository, times(1)).findById(1L);
        }

        @Test
        void getProductById_NotFound() {
            when(productRepository.findById(2L)).thenReturn(Optional.empty());
            assertThrows(ProductNotFoundException.class, () -> productService.getProductById(2L));
            verify(productRepository, times(1)).findById(2L);
        }

        @Test
        void getAllProducts_ReturnsList() {
            Product product2 = new Product(2L, "Mouse", "Wireless Mouse", new BigDecimal("25.00"), 50, "Electronics");
            when(productRepository.findAll()).thenReturn(Arrays.asList(product, product2));
            List<ProductResponseDTO> results = productService.getAllProducts();
            assertNotNull(results);
            assertEquals(2, results.size());
            verify(productRepository, times(1)).findAll();
        }

        @Test
        void updateProduct_Success() {
            when(productRepository.findById(1L)).thenReturn(Optional.of(product));
            when(productRepository.save(any(Product.class))).thenReturn(product);

            ProductRequestDTO updatedDto = new ProductRequestDTO("Updated Laptop", "Improved performance", new BigDecimal("1300.00"), 8, "Electronics");
            ProductResponseDTO result = productService.updateProduct(1L, updatedDto);

            assertNotNull(result);
            assertEquals("Updated Laptop", result.getName());
            assertEquals(new BigDecimal("1300.00"), result.getPrice());
            verify(productRepository, times(1)).findById(1L);
            verify(productRepository, times(1)).save(any(Product.class));
        }

        @Test
        void updateProduct_NotFound() {
            when(productRepository.findById(2L)).thenReturn(Optional.empty());
            ProductRequestDTO updatedDto = new ProductRequestDTO("Non Existent", "Desc", new BigDecimal("100.00"), 5, "Cat");
            assertThrows(ProductNotFoundException.class, () -> productService.updateProduct(2L, updatedDto));
            verify(productRepository, times(1)).findById(2L);
            verify(productRepository, never()).save(any(Product.class));
        }

        @Test
        void deleteProduct_Success() {
            when(productRepository.existsById(1L)).thenReturn(true);
            doNothing().when(productRepository).deleteById(1L);
            productService.deleteProduct(1L);
            verify(productRepository, times(1)).existsById(1L);
            verify(productRepository, times(1)).deleteById(1L);
        }

        @Test
        void deleteProduct_NotFound() {
            when(productRepository.existsById(2L)).thenReturn(false);
            assertThrows(ProductNotFoundException.class, () -> productService.deleteProduct(2L));
            verify(productRepository, times(1)).existsById(2L);
            verify(productRepository, never()).deleteById(anyLong());
        }
    }
    ```

-   **Integration Tests**: Use `@SpringBootTest` and `@ActiveProfiles("test")` to test the full application flow, from controller to database. Use Testcontainers for database integration tests to ensure a clean, isolated database for each test run.

    **Example: `ProductControllerIntegrationTest.java` (Integration Test)**
    ```java
    package com.hcl.productcatalog.controller;

    import com.fasterxml.jackson.databind.ObjectMapper;
    import com.hcl.productcatalog.dto.ProductRequestDTO;
    import com.hcl.productcatalog.model.Product;
    import com.hcl.productcatalog.repository.ProductRepository;
    import org.junit.jupiter.api.AfterEach;
    import org.junit.jupiter.api.BeforeEach;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.http.MediaType;
    import org.springframework.test.context.ActiveProfiles;
    import org.springframework.test.context.DynamicPropertyRegistry;
    import org.springframework.test.context.DynamicPropertySource;
    import org.springframework.test.web.servlet.MockMvc;
    import org.testcontainers.containers.PostgreSQLContainer;
    import org.testcontainers.junit.jupiter.Container;
    import org.testcontainers.junit.jupiter.Testcontainers;

    import java.math.BigDecimal;
    import java.util.List;

    import static org.hamcrest.Matchers.hasSize;
    import static org.hamcrest.Matchers.is;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

    @SpringBootTest
    @AutoConfigureMockMvc
    @ActiveProfiles("test") // Use a dedicated test profile if needed
    @Testcontainers
    class ProductControllerIntegrationTest {

        @Autowired
        private MockMvc mockMvc;

        @Autowired
        private ProductRepository productRepository;

        @Autowired
        private ObjectMapper objectMapper;

        @Container
        static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
                .withDatabaseName("testdb")
                .withUsername("testuser")
                .withPassword("testpass");

        @DynamicPropertySource
        static void configureProperties(DynamicPropertyRegistry registry) {
            registry.add("spring.datasource.url", postgres::getJdbcUrl);
            registry.add("spring.datasource.username", postgres::getUsername);
            registry.add("spring.datasource.password", postgres::getPassword);
            registry.add("spring.jpa.hibernate.ddl-auto", () -> "create-drop"); // Ensures fresh schema for each test run
        }

        @BeforeEach
        void setUp() {
            productRepository.deleteAll(); // Clean up before each test
        }

        @AfterEach
        void tearDown() {
            productRepository.deleteAll(); // Clean up after each test
        }

        @Test
        void createProduct_ShouldReturnCreatedProduct() throws Exception {
            ProductRequestDTO newProduct = new ProductRequestDTO("Keyboard", "Mechanical keyboard", new BigDecimal("75.00"), 20, "Peripherals");

            mockMvc.perform(post("/api/v1/products")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(newProduct)))
                    .andExpect(status().isCreated())
                    .andExpect(jsonPath("$.name", is("Keyboard")))
                    .andExpect(jsonPath("$.price", is(75.00)));

            List<Product> products = productRepository.findAll();
            assertEquals(1, products.size());
            assertEquals("Keyboard", products.get(0).getName());
        }

        @Test
        void createProduct_InvalidInput_ShouldReturnBadRequest() throws Exception {
            ProductRequestDTO invalidProduct = new ProductRequestDTO("", "No name", new BigDecimal("-10.00"), -5, "");

            mockMvc.perform(post("/api/v1/products")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(invalidProduct)))
                    .andExpect(status().isBadRequest())
                    .andExpect(jsonPath("$.message").exists());
        }

        @Test
        void getProductById_ShouldReturnProduct() throws Exception {
            Product savedProduct = productRepository.save(new Product(null, "Monitor", "4K Monitor", new BigDecimal("300.00"), 15, "Displays"));

            mockMvc.perform(get("/api/v1/products/{id}", savedProduct.getId()))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.id", is(savedProduct.getId().intValue())))
                    .andExpect(jsonPath("$.name", is("Monitor")));
        }

        @Test
        void getProductById_NotFound_ShouldReturnNotFound() throws Exception {
            mockMvc.perform(get("/api/v1/products/{id}", 99L))
                    .andExpect(status().isNotFound())
                    .andExpect(jsonPath("$.message").value("Product not found with id: 99"));
        }

        @Test
        void getAllProducts_ShouldReturnListOfProducts() throws Exception {
            productRepository.save(new Product(null, "Monitor", "4K Monitor", new BigDecimal("300.00"), 15, "Displays"));
            productRepository.save(new Product(null, "Webcam", "HD Webcam", new BigDecimal("50.00"), 30, "Peripherals"));

            mockMvc.perform(get("/api/v1/products"))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$", hasSize(2)))
                    .andExpect(jsonPath("$[0].name", is("Monitor")))
                    .andExpect(jsonPath("$[1].name", is("Webcam")));
        }

        @Test
        void updateProduct_ShouldReturnUpdatedProduct() throws Exception {
            Product savedProduct = productRepository.save(new Product(null, "Speaker", "Bluetooth Speaker", new BigDecimal("100.00"), 25, "Audio"));
            ProductRequestDTO updatedProduct = new ProductRequestDTO("Updated Speaker", "New Description", new BigDecimal("120.00"), 20, "Audio");

            mockMvc.perform(put("/api/v1/products/{id}", savedProduct.getId())
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(updatedProduct)))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.name", is("Updated Speaker")))
                    .andExpect(jsonPath("$.price", is(120.00)));

            Product foundProduct = productRepository.findById(savedProduct.getId()).orElseThrow();
            assertEquals("Updated Speaker", foundProduct.getName());
        }

        @Test
        void deleteProduct_ShouldReturnNoContent() throws Exception {
            Product savedProduct = productRepository.save(new Product(null, "Headphones", "Noise Cancelling", new BigDecimal("150.00"), 40, "Audio"));

            mockMvc.perform(delete("/api/v1/products/{id}", savedProduct.getId()))
                    .andExpect(status().isNoContent());

            assertFalse(productRepository.existsById(savedProduct.getId()));
        }

        @Test
        void deleteProduct_NotFound_ShouldReturnNotFound() throws Exception {
            mockMvc.perform(delete("/api/v1/products/{id}", 99L))
                    .andExpect(status().isNotFound())
                    .andExpect(jsonPath("$.message").value("Product not found with id: 99"));
        }
    }
    ```

-   **Coverage**: Aim for a minimum of 80% test coverage for the core business logic (services).

---

## 5. 🐳 Containerization & Deployment

### Dockerfile

Create a multi-stage `Dockerfile` in the project root:

1.  **Build Stage**: Use a `maven` base image to build the application JAR.
2.  **Runtime Stage**: Use a minimal JRE image (e.g., `eclipse-temurin:21-jre-alpine`).
3.  **Security**: Create and run as a non-root user.
4.  **Health Check**: Add a `HEALTHCHECK` instruction that calls the `/actuator/health` endpoint.
5.  **Execution**: Use `ENTRYPOINT ["java", "-jar", "app.jar"]`.

**Example `Dockerfile`:**
```dockerfile
# Stage 1: Build the application
FROM maven:3.9.5-eclipse-temurin-21-alpine AS build

WORKDIR /app

# Copy the pom.xml and download dependencies to leverage Docker cache
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy the rest of the source code
COPY src ./src

# Build the JAR file
RUN mvn package -DskipTests

# Stage 2: Create the runtime image
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Create a non-root user and switch to it for security best practices
RUN addgroup --system spring && adduser --system --ingroup spring spring
USER spring:spring

# Copy the built JAR from the build stage
COPY --from=build /app/target/*.jar app.jar

# Expose the port the application runs on
EXPOSE 8080

# Define a health check endpoint for container orchestration systems
HEALTHCHECK --interval=30s --timeout=10s --retries=5 \
  CMD curl --fail http://localhost:8080/actuator/health || exit 1

# Command to run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Environment Variables

All secrets (database passwords, API keys) **must** be supplied via environment variables, not hardcoded in properties files. For the Product Catalog Service, examples include `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`, `DB_PASSWORD`.

---

This protocol ensures consistency, security, and quality across the generated Product Catalog microservice. The agent must validate its output against these guidelines before completing a task.