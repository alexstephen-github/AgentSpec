Here is the updated `Step 8: Coding Standards` section, incorporating best practices from the provided GitHub link and refining the existing guidelines. The redundant sections (`## 10. Containerization and Deployment`, `## 11. OpenAPI/Swagger Documentation`) that were previously incorrectly nested under `Step 8` have been removed, as their content is addressed in other dedicated sections of this document.

---

### 🤖 Agent Protocol: Building Enterprise-Grade Java Microservices for Parcel Management

This document provides a comprehensive guide for AI coding agents to build secure, scalable, and maintainable microservices using Java 21 and Spring Boot 3 for the **Parcel Management Service**. Adherence to these principles is mandatory for all generated projects.

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
| **Framework** | Spring Boot | **3.3.1** |
| **Build Tool** | Maven | 3.9.6 |
| **Database** | PostgreSQL (Prod), H2 (Dev/Test) | 16.x (PostgreSQL), 2.x (H2) |
| **API Spec** | OpenAPI | 3.0 |
| **Container** | Docker | Latest |
| **Caching** | Caffeine (local), Redis (distributed) | Latest |
| **Validation** | Jakarta Bean Validation | 3.0 |
| **Testing** | JUnit 5, Mockito | Latest |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using **Spring Initializr** (`start.spring.io`) with the following default dependencies:

-   **Project:** Maven Project
-   **Language:** Java
-   **Spring Boot:** 3.3.1
-   **Group:** `com.hcl`
-   **Artifact:** `parcel-service`
-   **Name:** `parcel-service`
-   **Description:** Parcel Management Service
-   **Package Name:** `com.hcl.parcelservice`
-   **Java:** 21
-   **Dependencies:**
    -   Spring Web
    -   Spring Data JPA
    -   Spring Boot Actuator
    -   Spring Cache Abstraction
    -   Jakarta Bean Validation
    -   Lombok
    -   PostgreSQL Driver
    -   H2 Database
    -   SpringDoc OpenAPI (`org.springdoc:springdoc-openapi-starter-webmvc-ui`)
    -   Micrometer Tracing (Optional, but recommended for observability)
    -   Resilience4j (Optional, for circuit breaking/rate limiting)

### Standard Package Structure

Organize all source code under the root package `com.hcl.parcelservice`.

```
com/hcl/parcelservice/
├── ParcelServiceApplication.java   # Main application class
├── config/                         # @Configuration beans (Cache, Security, OpenAPI, Tracing)
├── controller/                     # REST controllers (API layer)
│   └── ParcelController.java
├── dto/                            # Data Transfer Objects for API requests/responses
│   ├── ParcelRequestDTO.java
│   └── ParcelResponseDTO.java
├── model/                          # JPA Entities
│   └── Parcel.java
├── repository/                     # Spring Data JPA repositories
│   └── ParcelRepository.java
├── service/                        # Business logic
│   └── ParcelService.java
└── exception/                      # Custom exceptions and global handler
    ├── ResourceNotFoundException.java
    ├── DuplicateTrackingNumberException.java
    ├── ParcelServiceException.java
    └── GlobalExceptionHandler.java
    └── ErrorResponse.java
```

---

## 4. 📝 Development Workflow & Best Practices

### Step 1: API & DTO Design (Controller Layer)

1.  **Define DTOs**: Create separate Plain Old Java Objects (POJOs) for API requests and responses in the `dto` package.
    *   **`ParcelRequestDTO.java`**: For creating/updating parcels.
    *   **`ParcelResponseDTO.java`**: For returning parcel details.
    *   Use Jakarta Bean Validation annotations (`@NotBlank`, `@Size`, `@Email`, `@NotNull`, `@Min`, `@Max`) for all incoming data.
    *   **Never expose JPA entities directly in the API.**

    **Example `ParcelRequestDTO.java`:**
    ```java
    package com.hcl.parcelservice.dto;

    import jakarta.validation.constraints.DecimalMin;
    import jakarta.validation.constraints.NotBlank;
    import jakarta.validation.constraints.NotNull;
    import jakarta.validation.constraints.Size;
    import lombok.Builder;
    import lombok.Value;

    import java.math.BigDecimal;

    @Value
    @Builder
    public class ParcelRequestDTO {
        @NotBlank(message = "Tracking number cannot be blank")
        @Size(min = 10, max = 20, message = "Tracking number must be between 10 and 20 characters")
        String trackingNumber;

        @NotBlank(message = "Sender name cannot be blank")
        @Size(max = 100, message = "Sender name too long")
        String senderName;

        @NotBlank(message = "Recipient address cannot be blank")
        String recipientAddress;

        @NotNull(message = "Weight cannot be null")
        @DecimalMin(value = "0.01", message = "Weight must be greater than 0")
        BigDecimal weightKg;

        @NotBlank(message = "Status cannot be blank")
        String status; // e.g., PENDING, IN_TRANSIT, DELIVERED
    }
    ```

    **Example `ParcelResponseDTO.java`:**
    ```java
    package com.hcl.parcelservice.dto;

    import lombok.Builder;
    import lombok.Value;

    import java.math.BigDecimal;
    import java.time.LocalDateTime;
    import java.util.UUID;

    @Value
    @Builder
    public class ParcelResponseDTO {
        UUID id;
        String trackingNumber;
        String senderName;
        String recipientAddress;
        BigDecimal weightKg;
        String status;
        LocalDateTime createdAt;
        LocalDateTime lastUpdated;
    }
    ```

2.  **Create Controller**: In the `controller` package, create a `@RestController`.

3.  **Map Endpoints**: Use `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`.

4.  **API Versioning**: Prefix all routes with `/api/v1`. Example: `@RequestMapping("/api/v1/parcels")`.

5.  **Input/Output**: Accept DTOs as `@RequestBody` and return `ResponseEntity<Dto>` or `ResponseEntity<List<Dto>>`. Use `@Valid` for request bodies.

    **Example `ParcelController.java`:**
    ```java
    package com.hcl.parcelservice.controller;

    import com.hcl.parcelservice.dto.ParcelRequestDTO;
    import com.hcl.parcelservice.dto.ParcelResponseDTO;
    import com.hcl.parcelservice.service.ParcelService;
    import jakarta.validation.Valid;
    import lombok.RequiredArgsConstructor;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.*;

    import java.util.List;
    import java.util.UUID;

    @RestController
    @RequestMapping("/api/v1/parcels")
    @RequiredArgsConstructor
    public class ParcelController {

        private final ParcelService parcelService;

        /**
         * Creates a new parcel.
         * @param requestDTO The parcel details to create.
         * @return The created parcel details.
         */
        @PostMapping
        public ResponseEntity<ParcelResponseDTO> createParcel(@Valid @RequestBody ParcelRequestDTO requestDTO) {
            ParcelResponseDTO createdParcel = parcelService.createParcel(requestDTO);
            return new ResponseEntity<>(createdParcel, HttpStatus.CREATED);
        }

        /**
         * Retrieves a parcel by its ID.
         * @param id The UUID of the parcel.
         * @return The parcel details.
         */
        @GetMapping("/{id}")
        public ResponseEntity<ParcelResponseDTO> getParcelById(@PathVariable UUID id) {
            ParcelResponseDTO parcel = parcelService.getParcelById(id);
            return ResponseEntity.ok(parcel);
        }

        /**
         * Retrieves all parcels.
         * @return A list of all parcels.
         */
        @GetMapping
        public ResponseEntity<List<ParcelResponseDTO>> getAllParcels() {
            List<ParcelResponseDTO> parcels = parcelService.getAllParcels();
            return ResponseEntity.ok(parcels);
        }

        /**
         * Updates an existing parcel.
         * @param id The UUID of the parcel to update.
         * @param requestDTO The updated parcel details.
         * @return The updated parcel details.
         */
        @PutMapping("/{id}")
        public ResponseEntity<ParcelResponseDTO> updateParcel(@PathVariable UUID id, @Valid @RequestBody ParcelRequestDTO requestDTO) {
            ParcelResponseDTO updatedParcel = parcelService.updateParcel(id, requestDTO);
            return ResponseEntity.ok(updatedParcel);
        }

        /**
         * Deletes a parcel by its ID.
         * @param id The UUID of the parcel to delete.
         * @return No content.
         */
        @DeleteMapping("/{id}")
        public ResponseEntity<Void> deleteParcel(@PathVariable UUID id) {
            parcelService.deleteParcel(id);
            return ResponseEntity.noContent().build();
        }
    }
    ```

### Step 2: Business Logic (Service Layer)

1.  **Create Service**: In the `service` package, create a `@Service` class.

2.  **Inject Dependencies**: Use constructor injection for repositories and other services.

3.  **Implement Logic**: Write the core business logic here. This layer is responsible for coordinating data access, calling other services, and enforcing business rules (e.g., status transitions, weight validation).

4.  **Use Transactions**: Use `@Transactional` for methods that modify data. Use `@Transactional(readOnly = true)` for read operations to optimize performance.

5.  **Map DTOs to Entities**: The service layer is responsible for converting incoming DTOs to JPA entities before passing them to the repository, and converting entities to DTOs before returning them to the controller. A dedicated `mapper` component or static methods in the DTOs/Entities can be used for this.

    **Example `ParcelService.java`:**
    ```java
    package com.hcl.parcelservice.service;

    import com.hcl.parcelservice.dto.ParcelRequestDTO;
    import com.hcl.parcelservice.dto.ParcelResponseDTO;
    import com.hcl.parcelservice.exception.DuplicateTrackingNumberException;
    import com.hcl.parcelservice.exception.ResourceNotFoundException;
    import com.hcl.parcelservice.model.Parcel;
    import com.hcl.parcelservice.repository.ParcelRepository;
    import lombok.RequiredArgsConstructor;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.cache.annotation.CacheEvict;
    import org.springframework.cache.annotation.CachePut;
    import org.springframework.cache.annotation.Cacheable;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import java.time.LocalDateTime;
    import java.util.List;
    import java.util.UUID;
    import java.util.stream.Collectors;

    @Service
    @RequiredArgsConstructor
    @Slf4j
    public class ParcelService {

        private final ParcelRepository parcelRepository;

        @Transactional
        @CacheEvict(value = "allParcels", allEntries = true) // Invalidate allParcels cache on creation
        public ParcelResponseDTO createParcel(ParcelRequestDTO requestDTO) {
            log.info("Creating new parcel with tracking number: {}", requestDTO.getTrackingNumber());
            if (parcelRepository.existsByTrackingNumber(requestDTO.getTrackingNumber())) {
                throw new DuplicateTrackingNumberException("Parcel with tracking number " + requestDTO.getTrackingNumber() + " already exists.");
            }

            Parcel parcel = Parcel.builder()
                    .trackingNumber(requestDTO.getTrackingNumber())
                    .senderName(requestDTO.getSenderName())
                    .recipientAddress(requestDTO.getRecipientAddress())
                    .weightKg(requestDTO.getWeightKg())
                    .status(requestDTO.getStatus())
                    .createdAt(LocalDateTime.now())
                    .lastUpdated(LocalDateTime.now())
                    .build();
            Parcel savedParcel = parcelRepository.save(parcel);
            log.info("Parcel created with ID: {}", savedParcel.getId());
            return toResponseDTO(savedParcel);
        }

        @Transactional(readOnly = true)
        @Cacheable(value = "parcels", key = "#id")
        public ParcelResponseDTO getParcelById(UUID id) {
            log.debug("Fetching parcel with ID: {}", id);
            Parcel parcel = parcelRepository.findById(id)
                    .orElseThrow(() -> {
                        log.warn("Parcel not found with ID: {}", id);
                        return new ResourceNotFoundException("Parcel not found with id " + id);
                    });
            return toResponseDTO(parcel);
        }

        @Transactional(readOnly = true)
        @Cacheable(value = "allParcels") // Simple cache for all parcels, requires careful eviction strategy
        public List<ParcelResponseDTO> getAllParcels() {
            log.debug("Fetching all parcels");
            return parcelRepository.findAll().stream()
                    .map(this::toResponseDTO)
                    .collect(Collectors.toList());
        }

        @Transactional
        @CachePut(value = "parcels", key = "#id")
        @CacheEvict(value = "allParcels", allEntries = true) // Evict all entries from 'allParcels' cache
        public ParcelResponseDTO updateParcel(UUID id, ParcelRequestDTO requestDTO) {
            log.info("Updating parcel with ID: {}", id);
            Parcel existingParcel = parcelRepository.findById(id)
                    .orElseThrow(() -> {
                        log.warn("Parcel not found for update with ID: {}", id);
                        return new ResourceNotFoundException("Parcel not found with id " + id);
                    });

            if (!existingParcel.getTrackingNumber().equals(requestDTO.getTrackingNumber()) && parcelRepository.existsByTrackingNumber(requestDTO.getTrackingNumber())) {
                throw new DuplicateTrackingNumberException("Another parcel with tracking number " + requestDTO.getTrackingNumber() + " already exists.");
            }

            existingParcel.setTrackingNumber(requestDTO.getTrackingNumber());
            existingParcel.setSenderName(requestDTO.getSenderName());
            existingParcel.setRecipientAddress(requestDTO.getRecipientAddress());
            existingParcel.setWeightKg(requestDTO.getWeightKg());
            existingParcel.setStatus(requestDTO.getStatus());
            existingParcel.setLastUpdated(LocalDateTime.now());

            Parcel updatedParcel = parcelRepository.save(existingParcel);
            log.info("Parcel updated with ID: {}", updatedParcel.getId());
            return toResponseDTO(updatedParcel);
        }

        @Transactional
        @CacheEvict(value = {"parcels", "allParcels"}, key = "#id", allEntries = true) // Evict from both caches for specific ID and all entries
        public void deleteParcel(UUID id) {
            log.info("Deleting parcel with ID: {}", id);
            if (!parcelRepository.existsById(id)) {
                log.warn("Parcel not found for deletion with ID: {}", id);
                throw new ResourceNotFoundException("Parcel not found with id " + id);
            }
            parcelRepository.deleteById(id);
            log.info("Parcel deleted with ID: {}", id);
        }

        private ParcelResponseDTO toResponseDTO(Parcel parcel) {
            return ParcelResponseDTO.builder()
                    .id(parcel.getId())
                    .trackingNumber(parcel.getTrackingNumber())
                    .senderName(parcel.getSenderName())
                    .recipientAddress(parcel.getRecipientAddress())
                    .weightKg(parcel.getWeightKg())
                    .status(parcel.getStatus())
                    .createdAt(parcel.getCreatedAt())
                    .lastUpdated(parcel.getLastUpdated())
                    .build();
        }
    }
    ```

### Step 3: Data Persistence (Repository & Model Layer)

1.  **Create Entity**: In the `model` package, define a JPA `@Entity`. Use `@Id`, `@GeneratedValue`, and appropriate `@Column` annotations.
    *   Use `UUID` for primary keys.
    *   Avoid Lombok's `@Data` on entities; prefer `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode(of = "id")`, and `@NoArgsConstructor`, `@AllArgsConstructor` with a `@Builder` pattern for controlled entity creation.

    **Example `Parcel.java`:**
    ```java
    package com.hcl.parcelservice.model;

    import jakarta.persistence.*;
    import lombok.*;
    import org.hibernate.annotations.GenericGenerator;
    import org.hibernate.annotations.JdbcTypeCode;
    import org.hibernate.type.SqlTypes;

    import java.math.BigDecimal;
    import java.time.LocalDateTime;
    import java.util.UUID;

    @Entity
    @Table(name = "parcels")
    @Getter
    @Setter
    @ToString
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    @EqualsAndHashCode(of = "id")
    public class Parcel {

        @Id
        @GeneratedValue(generator = "UUID")
        @GenericGenerator(
            name = "UUID",
            strategy = "org.hibernate.id.UUIDGenerator"
        )
        @JdbcTypeCode(SqlTypes.CHAR) // Explicitly map UUID to CHAR/VARCHAR for cross-DB compatibility
        @Column(name = "id", updatable = false, nullable = false, columnDefinition = "CHAR(36)")
        private UUID id;

        @Column(name = "tracking_number", unique = true, nullable = false, length = 20)
        private String trackingNumber;

        @Column(name = "sender_name", nullable = false, length = 100)
        private String senderName;

        @Column(name = "recipient_address", nullable = false)
        private String recipientAddress;

        @Column(name = "weight_kg", precision = 10, scale = 3, nullable = false)
        private BigDecimal weightKg;

        @Column(name = "status", nullable = false, length = 50)
        private String status; // Consider an Enum for status: e.g., @Enumerated(EnumType.STRING) ParcelStatus status;

        @Column(name = "created_at", nullable = false, updatable = false)
        private LocalDateTime createdAt;

        @Column(name = "last_updated", nullable = false)
        private LocalDateTime lastUpdated;

        @PrePersist
        protected void onCreate() {
            if (this.id == null) {
                this.id = UUID.randomUUID(); // Ensure UUID is generated even if not explicitly set
            }
            this.createdAt = LocalDateTime.now();
            this.lastUpdated = LocalDateTime.now();
        }

        @PreUpdate
        protected void onUpdate() {
            this.lastUpdated = LocalDateTime.now();
        }
    }
    ```

2.  **Create Repository**: In the `repository` package, create an interface that extends `JpaRepository<Entity, IdType>`. Add custom query methods as needed (e.g., `findByTrackingNumber`).

    **Example `ParcelRepository.java`:**
    ```java
    package com.hcl.parcelservice.repository;

    import com.hcl.parcelservice.model.Parcel;
    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.stereotype.Repository;

    import java.util.Optional;
    import java.util.UUID;

    @Repository
    public interface ParcelRepository extends JpaRepository<Parcel, UUID> {
        Optional<Parcel> findByTrackingNumber(String trackingNumber);
        boolean existsByTrackingNumber(String trackingNumber);
    }
    ```

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in the `exception` package (e.g., `ResourceNotFoundException`, `DuplicateResourceException`).

    **Example `ResourceNotFoundException.java`:**
    ```java
    package com.hcl.parcelservice.exception;

    import org.springframework.http.HttpStatus;
    import org.springframework.web.bind.annotation.ResponseStatus;

    @ResponseStatus(HttpStatus.NOT_FOUND)
    public class ResourceNotFoundException extends RuntimeException {
        public ResourceNotFoundException(String message) {
            super(message);
        }

        public ResourceNotFoundException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    ```

    **Example `DuplicateTrackingNumberException.java`:**
    ```java
    package com.hcl.parcelservice.exception;

    import org.springframework.http.HttpStatus;
    import org.springframework.web.bind.annotation.ResponseStatus;

    @ResponseStatus(HttpStatus.CONFLICT)
    public class DuplicateTrackingNumberException extends RuntimeException {
        public DuplicateTrackingNumberException(String message) {
            super(message);
        }
    }
    ```

    **Example `ErrorResponse.java` (DTO for error messages):**
    ```java
    package com.hcl.parcelservice.exception;

    import lombok.Builder;
    import lombok.Value;
    import java.time.LocalDateTime;
    import java.util.List;

    @Value
    @Builder
    public class ErrorResponse {
        LocalDateTime timestamp;
        int status;
        String error;
        String message;
        String path;
        List<String> details; // For validation errors, to list multiple field errors
    }
    ```

2.  **Global Handler**: Create a `@RestControllerAdvice` class in the `exception` package.

3.  **Handle Exceptions**: Add `@ExceptionHandler` methods for custom exceptions, validation exceptions (`MethodArgumentNotValidException`), and a fallback generic `Exception`. Return a standardized `ErrorResponse` DTO with a message and HTTP status. **Never expose stack traces to the client.**

    **Example `GlobalExceptionHandler.java`:**
    ```java
    package com.hcl.parcelservice.exception;

    import lombok.extern.slf4j.Slf4j;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.RestControllerAdvice;
    import org.springframework.web.context.request.WebRequest;

    import java.time.LocalDateTime;
    import java.util.List;
    import java.util.stream.Collectors;

    @RestControllerAdvice
    @Slf4j
    public class GlobalExceptionHandler {

        @ExceptionHandler(ResourceNotFoundException.class)
        public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex, WebRequest request) {
            log.warn("ResourceNotFoundException: {}", ex.getMessage());
            ErrorResponse errorResponse = ErrorResponse.builder()
                    .timestamp(LocalDateTime.now())
                    .status(HttpStatus.NOT_FOUND.value())
                    .error(HttpStatus.NOT_FOUND.getReasonPhrase())
                    .message(ex.getMessage())
                    .path(request.getDescription(false))
                    .build();
            return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
        }

        @ExceptionHandler(DuplicateTrackingNumberException.class)
        public ResponseEntity<ErrorResponse> handleDuplicateTrackingNumberException(DuplicateTrackingNumberException ex, WebRequest request) {
            log.warn("DuplicateTrackingNumberException: {}", ex.getMessage());
            ErrorResponse errorResponse = ErrorResponse.builder()
                    .timestamp(LocalDateTime.now())
                    .status(HttpStatus.CONFLICT.value())
                    .error(HttpStatus.CONFLICT.getReasonPhrase())
                    .message(ex.getMessage())
                    .path(request.getDescription(false))
                    .build();
            return new ResponseEntity<>(errorResponse, HttpStatus.CONFLICT);
        }

        @ExceptionHandler(MethodArgumentNotValidException.class)
        public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex, WebRequest request) {
            List<String> errors = ex.getBindingResult().getFieldErrors().stream()
                    .map(error -> error.getField() + ": " + error.getDefaultMessage())
                    .collect(Collectors.toList());
            String primaryMessage = errors.isEmpty() ? "Validation failed" : errors.get(0);

            log.warn("Validation Error: {}", errors);
            ErrorResponse errorResponse = ErrorResponse.builder()
                    .timestamp(LocalDateTime.now())
                    .status(HttpStatus.BAD_REQUEST.value())
                    .error(HttpStatus.BAD_REQUEST.getReasonPhrase())
                    .message("Validation Failed: " + primaryMessage)
                    .path(request.getDescription(false))
                    .details(errors)
                    .build();
            return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
        }

        @ExceptionHandler(Exception.class)
        public ResponseEntity<ErrorResponse> handleGlobalException(Exception ex, WebRequest request) {
            log.error("An unexpected error occurred: {}", ex.getMessage(), ex); // Log full stack trace for unexpected errors
            ErrorResponse errorResponse = ErrorResponse.builder()
                    .timestamp(LocalDateTime.now())
                    .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
                    .error(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase())
                    .message("An unexpected error occurred. Please try again later.") // Generic message for clients
                    .path(request.getDescription(false))
                    .build();
            return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
    ```

### Step 5: Caching

1.  **Enable Caching**: Add `@EnableCaching` to the main application class.

    **`ParcelServiceApplication.java`:**
    ```java
    package com.hcl.parcelservice;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cache.annotation.EnableCaching;

    @SpringBootApplication
    @EnableCaching
    public class ParcelServiceApplication {

        public static void main(String[] args) {
            SpringApplication.run(ParcelServiceApplication.class, args);
        }
    }
    ```

2.  **Configure Cache**: Create a `CacheConfig` class to define cache managers (e.g., Caffeine for local, Redis for distributed).

    **Example `config/CacheConfig.java`:**
    ```java
    package com.hcl.parcelservice.config;

    import com.github.benmanes.caffeine.cache.Caffeine;
    import org.springframework.cache.CacheManager;
    import org.springframework.cache.caffeine.CaffeineCacheManager;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Profile;
    import org.springframework.data.redis.cache.RedisCacheConfiguration;
    import org.springframework.data.redis.cache.RedisCacheManager;
    import org.springframework.data.redis.connection.RedisConnectionFactory;
    import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
    import org.springframework.data.redis.serializer.RedisSerializationContext;
    import org.springframework.data.redis.serializer.StringRedisSerializer;

    import java.time.Duration;
    import java.util.concurrent.TimeUnit;

    @Configuration
    public class CacheConfig {

        // Local Cache (Caffeine) for dev/test
        @Bean
        @Profile({"dev", "test"})
        public CacheManager caffeineCacheManager() {
            CaffeineCacheManager cacheManager = new CaffeineCacheManager("parcels", "allParcels");
            cacheManager.setCaffeine(Caffeine.newBuilder()
                    .initialCapacity(100)
                    .maximumSize(500)
                    .expireAfterAccess(10, TimeUnit.MINUTES)
                    .recordStats());
            return cacheManager;
        }

        // Distributed Cache (Redis) for production
        @Bean
        @Profile("prod")
        public CacheManager redisCacheManager(RedisConnectionFactory connectionFactory) {
            RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofMinutes(10)) // Default TTL for all caches
                    .disableCachingNullValues()
                    .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                    .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

            return RedisCacheManager.builder(connectionFactory)
                    .cacheDefaults(defaultCacheConfig)
                    .withCacheConfiguration("parcels", defaultCacheConfig.entryTtl(Duration.ofMinutes(15))) // Specific TTL for 'parcels' cache
                    .withCacheConfiguration("allParcels", defaultCacheConfig.entryTtl(Duration.ofMinutes(5))) // Specific TTL for 'allParcels' cache
                    .build();
        }
    }
    ```

3.  **Apply Caching**: Use `@Cacheable`, `@CachePut`, and `@CacheEvict` on service methods to cache results of expensive operations. (See `ParcelService.java` example above).

### Step 6: Configuration

-   **Base Config (`application.properties`)**: Define common properties like `spring.application.name`.

    ```properties
    # application.properties
    spring.application.name=parcel-service
    server.port=8080

    # SpringDoc OpenAPI Configuration
    springdoc.swagger-ui.path=/swagger-ui.html
    springdoc.api-docs.path=/v3/api-docs

    # Actuator Endpoints
    management.endpoints.web.exposure.include=*
    management.endpoint.health.show-details=always # In dev, never in prod
    management.metrics.tags.application=${spring.application.name}

    # Logging
    logging.level.com.hcl.parcelservice=INFO
    logging.level.org.springframework=INFO
    logging.pattern.level=%5p [${spring.application.name},%X{traceId:-},%X{spanId:-}]
    ```

-   **Dev Profile (`application-dev.properties`)**: Configure H2 in-memory database, enable the H2 console, and set logging to `DEBUG`.

    ```properties
    # application-dev.properties
    spring.h2.console.enabled=true
    spring.h2.console.path=/h2-console
    spring.datasource.url=jdbc:h2:mem:parcelsdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    spring.datasource.driver-class-name=org.h2.Driver
    spring.datasource.username=sa
    spring.datasource.password=
    spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
    spring.jpa.hibernate.ddl-auto=update # or create for development, ensure data loss is acceptable
    spring.jpa.show-sql=true
    spring.jpa.properties.hibernate.format_sql=true

    # Enable Caffeine for dev
    spring.cache.type=caffeine

    logging.level.com.hcl.parcelservice=DEBUG
    logging.level.org.springframework=DEBUG
    management.endpoint.health.show-details=always
    ```

-   **Prod Profile (`application-prod.properties`)**: Configure PostgreSQL connection details. Use placeholders for secrets (e.g., `${DB_PASSWORD}`). Disable detailed health info (`management.endpoint.health.show-details=never`).

    ```properties
    # application-prod.properties
    spring.datasource.url=${DB_URL:jdbc:postgresql://localhost:5432/parcel_db}
    spring.datasource.username=${DB_USERNAME:parcel_user}
    spring.datasource.password=${DB_PASSWORD}
    spring.datasource.driver-class-name=org.postgresql.Driver
    spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
    spring.jpa.hibernate.ddl-auto=validate # or none for production
    spring.jpa.show-sql=false
    spring.jpa.properties.hibernate.format_sql=false

    # Enable Redis for prod
    spring.cache.type=redis
    spring.data.redis.host=${REDIS_HOST:localhost}
    spring.data.redis.port=${REDIS_PORT:6379}
    spring.data.redis.password=${REDIS_PASSWORD:} # Optional if Redis requires auth

    logging.level.com.hcl.parcelservice=INFO
    logging.level.org.springframework=WARN # Reduce verbosity for prod
    management.endpoint.health.show-details=never # Important for security
    management.endpoint.health.roles=ACTUATOR_ADMIN # If security is enabled for actuator endpoints
    ```

### Step 7: `catalog-info.yaml` File Structure

The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: parcel-service # required, unique name for the service
  description: Manages the lifecycle and tracking of parcels. # required, human-readable description
  tags:
    - java
    - spring-boot
    - microservice
    - parcels
    - logistics
    - api
  annotations:
    backstage.io/source-location: url:https://github.com/hcl-org/parcel-service/tree/main # Link to the source code repository
    # sonarqube.org/project-key: hcl-org_parcel-service # Example SonarQube project key
    # jenkins.io/job-full-name: hcl-org/parcel-service/main # Example Jenkins job
    # argocd/app-name: parcel-service-prod # Example ArgoCD application name
  links:
    - url: https://github.com/hcl-org/parcel-service # URL to the GitHub repository
      title: GitHub Repo
      icon: github
    - url: https://hcl-org.atlassian.net/jira/software/projects/PARCEL/board # URL to Jira Board for project tracking
      title: Jira Board
      icon: dashboard
    - url: https://confluence.hcl-org.com/display/PARCEL/Home # URL to Confluence documentation
      title: Confluence Docs
      icon: docs
    - url: http://localhost:8080/swagger-ui.html # OpenAPI UI in development
      title: OpenAPI Dev UI
      icon: docs
spec:
  type: service # required, e.g., 'service', 'library', 'website'
  lifecycle: production # required, e.g., 'experimental', 'production', 'deprecated'
  owner: group:parcel-management-team # required, the owning team or user (e.g., 'user:john.doe' or 'group:platform-team')
  system: logistics-management-system # optional, the larger system this component belongs to
  dependsOn:
    - resource:database/parcel-db-postgresql # This service depends on a PostgreSQL database resource
    - component:notification-service # Depends on a notification service for status updates
  providesApis:
    - parcel-api # This service provides the Parcel API
```

---
### Step 8: Coding Standards

Adherence to these coding standards is crucial for maintaining a high-quality, readable, and maintainable codebase.

-   **Naming Conventions**:
    *   **Packages**: All lowercase (e.g., `com.hcl.parcelservice.controller`).
    *   **Classes/Interfaces/Enums**: PascalCase (e.g., `ParcelService`, `ParcelStatus`).
    *   **Methods/Variables/Fields**: camelCase (e.g., `createParcel`, `trackingNumber`, `log`).
    *   **Constants**: `UPPER_SNAKE_CASE` (e.g., `MAX_WEIGHT_KG`).
    *   **Boolean**: Prefix with `is`, `has`, `can` (e.g., `isDelivered`, `hasTrackingNumber`).
    *   Avoid abbreviations unless widely understood within the domain.

-   **Code Structure and Formatting**:
    *   **Indentation**: Use 4 spaces for indentation.
    *   **Line Length**: Aim for a maximum of 120 characters per line.
    *   **Braces**: Use K&R style (opening brace on the same line as the control statement).
    *   **Blank Lines**: Use judiciously to separate logical blocks of code for readability.
    *   **File Structure**: One top-level class per `.java` file.
    *   **Class Members Order**: Follow a consistent order (e.g., static fields, instance fields, constructors, public methods, protected methods, private methods).

-   **Comments and Documentation (JavaDoc)**:
    *   **JavaDoc**: All public/protected classes, interfaces, enums, methods, and constructors **must** have comprehensive JavaDoc comments, explaining their purpose, parameters (`@param`), return values (`@return`), and exceptions (`@throws`).
    *   **Inline Comments**: Use sparingly for complex logic or workarounds (explain *why*, not *what*).
    *   **Remove Dead Code**: Commented-out code should be removed, not left in the codebase.

-   **Readability and Clarity**:
    *   Write clear, concise, and self-documenting code.
    *   Avoid overly complex or deeply nested logic; refactor into smaller, focused methods.
    *   Use meaningful variable and method names that convey their purpose.

-   **Static Analysis & Code Reviews**:
    *   **Static Analysis**: Integrate and address findings from tools like SonarQube, Checkstyle, and PMD. These tools will be part of the CI/CD pipeline.
    *   **Code Reviews**: All code changes require mandatory peer code reviews before merging.

-   **Error Handling**:
    *   Use specific custom exceptions (e.g., `ResourceNotFoundException`) rather than generic `Exception`.
    *   Catch exceptions as close to their source as possible, and re-throw if the caller needs to handle them.
    *   Log exceptions at the point of handling using appropriate log levels (typically `WARN` or `ERROR`).
    *   Provide meaningful and user-friendly error messages, without exposing sensitive internal details.

-   **Logging**:
    *   Use SLF4J with Logback for all logging.
    *   Use appropriate logging levels (`DEBUG`, `INFO`, `WARN`, `ERROR`).
    *   Do not log sensitive personal or security information (e.g., passwords, full credit card numbers).
    *   Employ parameterized logging to avoid String concatenation, which can impact performance (e.g., `log.info("Processing parcel {}", parcelId)` instead of `log.info("Processing parcel " + parcelId)`).

---

### Step 9: Documentation

-   **OpenAPI**: Configure a `OpenApiConfig` bean to set the API title, version, and description. This enhances the automatically generated Swagger UI. Use `@Operation`, `@ApiResponse`, `@Parameter`, `@Tag` from `io.swagger.v3.oas.annotations` to enrich API documentation directly in the controller methods.

    **Example `config/OpenApiConfig.java`:**
    ```java
    package com.hcl.parcelservice.config;

    import io.swagger.v3.oas.annotations.OpenAPIDefinition;
    import io.swagger.v3.oas.annotations.info.Contact;
    import io.swagger.v3.oas.annotations.info.Info;
    import io.swagger.v3.oas.annotations.info.License;
    import io.swagger.v3.oas.annotations.servers.Server;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    @OpenAPIDefinition(
        info = @Info(
            title = "Parcel Management Service API",
            version = "v1.0",
            description = "API for managing and tracking parcels within the logistics system.",
            contact = @Contact(
                name = "Parcel Management Team",
                email = "parcel.support@hcl.com"
            ),
            license = @License(
                name = "Apache 2.0",
                url = "http://www.apache.org/licenses/LICENSE-2.0.html"
            )
        ),
        servers = {
            @Server(url = "http://localhost:8080", description = "Development Server"),
            @Server(url = "https://parcel-service.hcl.com", description = "Production Server")
        }
    )
    public class OpenApiConfig {
        // Configuration is done via annotations on the class
    }
    ```

-   **JavaDoc**: Add JavaDoc comments to all public classes, methods, and significant fields in controllers and services, explaining their purpose, parameters, and return values.

### Step 10: Testing

-   **Unit Tests**: Test services and controllers by mocking their dependencies (e.g., repositories). Place tests in `src/test/java`.
    *   Example: Test `ParcelService` by mocking `ParcelRepository`.
-   **Integration Tests**: Use `@SpringBootTest` and `@ActiveProfiles("test")` to test the full application flow, from controller to database. Use Testcontainers for database integration tests (e.g., PostgreSQL container).
    *   Example: Use `MockMvc` to test `ParcelController` endpoints with an in-memory or Testcontainers database.
-   **Coverage**: Aim for a minimum of 80% test coverage. Use JaCoCo Maven plugin for reporting.

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
# --- Build Stage ---
FROM maven:3.9.6-eclipse-temurin-21-alpine AS builder

WORKDIR /app

# Copy the pom.xml and download dependencies to leverage Docker cache
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy all source code
COPY src ./src

# Build the application
RUN mvn clean install -DskipTests

# --- Runtime Stage ---
FROM eclipse-temurin:21-jre-alpine

# Set non-root user for security
RUN addgroup --system spring && adduser --system --ingroup spring spring
USER spring:spring

WORKDIR /app

# Copy the built JAR from the builder stage
COPY --from=builder /app/target/*.jar app.jar

# Expose the application port
EXPOSE 8080

# Define a health check endpoint for container orchestration
HEALTHCHECK --interval=30s --timeout=10s --retries=5 \
  CMD curl --fail http://localhost:8080/actuator/health || exit 1

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Environment Variables

All secrets (database passwords, API keys, Redis passwords) **must** be supplied via environment variables, not hardcoded in properties files. Use Spring Boot's externalized configuration to bind these.

---

This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.