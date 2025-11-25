Here's a detailed specification file for a Spring Boot REST API for "Parcel Management Service," adhering strictly to the provided `Agent-template.md` and including the specified coding standards.

---

# 🤖 Agent Protocol: Building Enterprise-Grade Java Microservices
## Detailed Spec File for "Parcel Management Service" API

This document details the implementation specifications for the "Parcel Management Service," a microservice designed to manage parcel shipments. It adheres to the `Agent Protocol` guidelines for building secure, scalable, and maintainable Java microservices.

---

## 1. 🎯 Core Principles (Parcel Management Service)

-   **Security First**: Implement authentication (e.g., JWT for API access), authorization (role-based access control), and secure data handling for sensitive parcel information (e.g., recipient details).
-   **Production Ready**: Robust error handling, comprehensive logging (structured logs for tracing), and externalized configuration for database, caching, and third-party integrations (e.g., notification services).
-   **Developer Experience**: Clear API documentation (OpenAPI), consistent code style, automated testing, and easy setup using Docker Compose for local development.
-   **Performance by Design**: Efficient database queries, strategic caching for frequently accessed parcel details, and asynchronous processing for non-critical operations (e.g., status updates).
-   **Observability**: Expose Prometheus metrics via Spring Boot Actuator, implement distributed tracing (e.g., Zipkin/Sleuth), and detailed application logs for troubleshooting.

---

## 2. 🛠️ Standard Technology Stack (Parcel Management Service)

| Component    | Technology                                  | Version/Standard                      |
| :----------- | :------------------------------------------ | :------------------------------------ |
| **Language** | Java                                        | **21**                                |
| **Framework** | Spring Boot                                 | **3.3+**                              |
| **Build Tool** | Maven                                       | 3.9+                                  |
| **Database** | PostgreSQL (Prod), H2 (Dev/Test)            | Latest                                |
| **API Spec** | OpenAPI                                     | 3.0                                   |
| **Container** | Docker                                      | Latest                                |
| **Caching**  | Caffeine (local), Redis (distributed, optional for prod) | Latest                                |
| **Validation** | Jakarta Bean Validation                     | 3.0                                   |
| **Testing**  | JUnit 5, Mockito, Testcontainers            | Latest                                |
| **Logging**  | SLF4J + Logback                             | Latest                                |
| **Security** | Spring Security                             | 6.x (Integrated with JWT if needed)   |
| **Tracing**  | Spring Cloud Sleuth/Micrometer Tracing      | Latest                                |

---

## 3. 🏗️ Project Initialization & Structure (Parcel Management Service)

### Initialization

Generate new projects using **Spring Initializr** (`start.spring.io`) with the following default dependencies:

-   **I/O**: Spring Web
-   **Data**: Spring Data JPA, PostgreSQL Driver, H2 Database
-   **Ops**: Spring Boot Actuator
-   **Cache**: Spring Cache Abstraction
-   **Validation**: Jakarta Bean Validation
-   **Utility**: Lombok
-   **Docs**: SpringDoc OpenAPI (`org.springdoc:springdoc-openapi-starter-webmvc-ui`)
-   **Security**: Spring Security (if authentication/authorization is required)

### Standard Package Structure

Organize all source code under the root package `com.hcl.parcel.service`.

```
com/hcl/parcel/service/
├── ParcelServiceApplication.java   # Main application class
├── config/                         # @Configuration beans (Cache, Security, OpenAPI)
│   ├── CacheConfig.java
│   └── OpenApiConfig.java
│   └── SecurityConfig.java         # (If Spring Security is used)
├── controller/                     # REST controllers (API layer)
│   └── ParcelController.java
├── dto/                            # Data Transfer Objects for API requests/responses
│   ├── ParcelRequest.java
│   ├── ParcelResponse.java
│   ├── ErrorResponse.java
│   └── ParcelStatusUpdateRequest.java
├── model/                          # JPA Entities
│   └── Parcel.java
├── repository/                     # Spring Data JPA repositories
│   └── ParcelRepository.java
├── service/                        # Business logic
│   └── ParcelService.java
└── exception/                      # Custom exceptions and global handler
    ├── GlobalExceptionHandler.java
    ├── ParcelNotFoundException.java
    └── InvalidParcelStatusException.java
```

---

## 4. 📝 Development Workflow & Best Practices (Parcel Management Service)

### Step 1: API & DTO Design (Controller Layer)

1.  **Define DTOs**:
    *   `ParcelRequest.java` (for creating/updating parcels):
        ```java
        package com.hcl.parcel.service.dto;

        import jakarta.validation.constraints.NotBlank;
        import jakarta.validation.constraints.NotNull;
        import jakarta.validation.constraints.Positive;
        import jakarta.validation.constraints.Size;
        import lombok.AllArgsConstructor;
        import lombok.Builder;
        import lombok.Getter;
        import lombok.NoArgsConstructor;
        import lombok.Setter;

        @Getter
        @Setter
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public class ParcelRequest {
            @NotBlank(message = "Sender name is required")
            @Size(max = 100, message = "Sender name cannot exceed 100 characters")
            private String senderName;

            @NotBlank(message = "Sender address is required")
            @Size(max = 255, message = "Sender address cannot exceed 255 characters")
            private String senderAddress;

            @NotBlank(message = "Recipient name is required")
            @Size(max = 100, message = "Recipient name cannot exceed 100 characters")
            private String recipientName;

            @NotBlank(message = "Recipient address is required")
            @Size(max = 255, message = "Recipient address cannot exceed 255 characters")
            private String recipientAddress;

            @NotNull(message = "Weight is required")
            @Positive(message = "Weight must be positive")
            private Double weight; // in kilograms

            @Size(max = 50, message = "Description cannot exceed 50 characters")
            private String description;
        }
        ```
    *   `ParcelResponse.java` (for returning parcel details):
        ```java
        package com.hcl.parcel.service.dto;

        import com.hcl.parcel.service.model.ParcelStatus;
        import lombok.AllArgsConstructor;
        import lombok.Builder;
        import lombok.Getter;
        import lombok.NoArgsConstructor;
        import lombok.Setter;

        import java.time.LocalDateTime;
        import java.util.UUID;

        @Getter
        @Setter
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public class ParcelResponse {
            private UUID id;
            private String trackingNumber;
            private String senderName;
            private String senderAddress;
            private String recipientName;
            private String recipientAddress;
            private Double weight;
            private String description;
            private ParcelStatus status;
            private LocalDateTime createdAt;
            private LocalDateTime lastUpdatedAt;
        }
        ```
    *   `ParcelStatusUpdateRequest.java` (for updating only status):
        ```java
        package com.hcl.parcel.service.dto;

        import com.hcl.parcel.service.model.ParcelStatus;
        import jakarta.validation.constraints.NotNull;
        import lombok.AllArgsConstructor;
        import lombok.Builder;
        import lombok.Getter;
        import lombok.NoArgsConstructor;
        import lombok.Setter;

        @Getter
        @Setter
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public class ParcelStatusUpdateRequest {
            @NotNull(message = "New status is required")
            private ParcelStatus newStatus;
        }
        ```

2.  **Create Controller**: `ParcelController.java`
    ```java
    package com.hcl.parcel.service.controller;

    import com.hcl.parcel.service.dto.ParcelRequest;
    import com.hcl.parcel.service.dto.ParcelResponse;
    import com.hcl.parcel.service.dto.ParcelStatusUpdateRequest;
    import com.hcl.parcel.service.service.ParcelService;
    import io.swagger.v3.oas.annotations.Operation;
    import io.swagger.v3.oas.annotations.responses.ApiResponse;
    import io.swagger.v3.oas.annotations.responses.ApiResponses;
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
         * @param parcelRequest DTO containing parcel creation details.
         * @return ResponseEntity with the created ParcelResponse and HTTP status 201.
         */
        @Operation(summary = "Create a new parcel")
        @ApiResponses(value = {
                @ApiResponse(responseCode = "201", description = "Parcel created successfully"),
                @ApiResponse(responseCode = "400", description = "Invalid input")
        })
        @PostMapping
        public ResponseEntity<ParcelResponse> createParcel(@Valid @RequestBody ParcelRequest parcelRequest) {
            ParcelResponse createdParcel = parcelService.createParcel(parcelRequest);
            return new ResponseEntity<>(createdParcel, HttpStatus.CREATED);
        }

        /**
         * Retrieves a parcel by its ID.
         * @param id The UUID of the parcel to retrieve.
         * @return ResponseEntity with the ParcelResponse and HTTP status 200.
         */
        @Operation(summary = "Get a parcel by ID")
        @ApiResponses(value = {
                @ApiResponse(responseCode = "200", description = "Parcel found"),
                @ApiResponse(responseCode = "404", description = "Parcel not found")
        })
        @GetMapping("/{id}")
        public ResponseEntity<ParcelResponse> getParcelById(@PathVariable UUID id) {
            ParcelResponse parcel = parcelService.getParcelById(id);
            return ResponseEntity.ok(parcel);
        }

        /**
         * Retrieves all parcels.
         * @return ResponseEntity with a list of ParcelResponse and HTTP status 200.
         */
        @Operation(summary = "Get all parcels")
        @ApiResponse(responseCode = "200", description = "List of all parcels")
        @GetMapping
        public ResponseEntity<List<ParcelResponse>> getAllParcels() {
            List<ParcelResponse> parcels = parcelService.getAllParcels();
            return ResponseEntity.ok(parcels);
        }

        /**
         * Updates an existing parcel.
         * @param id The UUID of the parcel to update.
         * @param parcelRequest DTO containing updated parcel details.
         * @return ResponseEntity with the updated ParcelResponse and HTTP status 200.
         */
        @Operation(summary = "Update an existing parcel")
        @ApiResponses(value = {
                @ApiResponse(responseCode = "200", description = "Parcel updated successfully"),
                @ApiResponse(responseCode = "400", description = "Invalid input"),
                @ApiResponse(responseCode = "404", description = "Parcel not found")
        })
        @PutMapping("/{id}")
        public ResponseEntity<ParcelResponse> updateParcel(@PathVariable UUID id, @Valid @RequestBody ParcelRequest parcelRequest) {
            ParcelResponse updatedParcel = parcelService.updateParcel(id, parcelRequest);
            return ResponseEntity.ok(updatedParcel);
        }

        /**
         * Updates the status of an existing parcel.
         * @param id The UUID of the parcel to update status.
         * @param statusUpdateRequest DTO containing the new status.
         * @return ResponseEntity with the updated ParcelResponse and HTTP status 200.
         */
        @Operation(summary = "Update the status of a parcel")
        @ApiResponses(value = {
                @ApiResponse(responseCode = "200", description = "Parcel status updated successfully"),
                @ApiResponse(responseCode = "400", description = "Invalid status update"),
                @ApiResponse(responseCode = "404", description = "Parcel not found")
        })
        @PatchMapping("/{id}/status")
        public ResponseEntity<ParcelResponse> updateParcelStatus(@PathVariable UUID id, @Valid @RequestBody ParcelStatusUpdateRequest statusUpdateRequest) {
            ParcelResponse updatedParcel = parcelService.updateParcelStatus(id, statusUpdateRequest.getNewStatus());
            return ResponseEntity.ok(updatedParcel);
        }

        /**
         * Deletes a parcel by its ID.
         * @param id The UUID of the parcel to delete.
         * @return ResponseEntity with HTTP status 204.
         */
        @Operation(summary = "Delete a parcel by ID")
        @ApiResponses(value = {
                @ApiResponse(responseCode = "204", description = "Parcel deleted successfully"),
                @ApiResponse(responseCode = "404", description = "Parcel not found")
        })
        @DeleteMapping("/{id}")
        public ResponseEntity<Void> deleteParcel(@PathVariable UUID id) {
            parcelService.deleteParcel(id);
            return ResponseEntity.noContent().build();
        }
    }
    ```

### Step 2: Business Logic (Service Layer)

1.  **Create Service**: `ParcelService.java`
    ```java
    package com.hcl.parcel.service.service;

    import com.hcl.parcel.service.dto.ParcelRequest;
    import com.hcl.parcel.service.dto.ParcelResponse;
    import com.hcl.parcel.service.exception.InvalidParcelStatusException;
    import com.hcl.parcel.service.exception.ParcelNotFoundException;
    import com.hcl.parcel.service.model.Parcel;
    import com.hcl.parcel.service.model.ParcelStatus;
    import com.hcl.parcel.service.repository.ParcelRepository;
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

        /**
         * Creates a new parcel based on the provided request DTO.
         * @param parcelRequest The DTO containing the details for the new parcel.
         * @return A ParcelResponse DTO of the created parcel.
         */
        @Transactional
        public ParcelResponse createParcel(ParcelRequest parcelRequest) {
            log.info("Creating new parcel with sender: {} and recipient: {}", parcelRequest.getSenderName(), parcelRequest.getRecipientName());
            Parcel parcel = mapToEntity(parcelRequest);
            parcel.setTrackingNumber(UUID.randomUUID().toString().substring(0, 8).toUpperCase()); // Simple tracking number
            parcel.setStatus(ParcelStatus.PENDING);
            parcel.setCreatedAt(LocalDateTime.now());
            parcel.setLastUpdatedAt(LocalDateTime.now());
            Parcel savedParcel = parcelRepository.save(parcel);
            log.debug("Parcel created successfully with ID: {}", savedParcel.getId());
            return mapToDto(savedParcel);
        }

        /**
         * Retrieves a parcel by its unique identifier. The result is cached.
         * @param id The UUID of the parcel to retrieve.
         * @return A ParcelResponse DTO of the found parcel.
         * @throws ParcelNotFoundException If no parcel with the given ID is found.
         */
        @Transactional(readOnly = true)
        @Cacheable(value = "parcels", key = "#id")
        public ParcelResponse getParcelById(UUID id) {
            log.debug("Attempting to retrieve parcel with ID: {}", id);
            Parcel parcel = parcelRepository.findById(id)
                    .orElseThrow(() -> {
                        log.warn("Parcel not found with ID: {}", id);
                        return new ParcelNotFoundException("Parcel not found with ID: " + id);
                    });
            log.debug("Parcel found with ID: {}", id);
            return mapToDto(parcel);
        }

        /**
         * Retrieves all parcels.
         * @return A list of ParcelResponse DTOs representing all parcels.
         */
        @Transactional(readOnly = true)
        public List<ParcelResponse> getAllParcels() {
            log.debug("Retrieving all parcels");
            List<Parcel> parcels = parcelRepository.findAll();
            return parcels.stream()
                    .map(this::mapToDto)
                    .collect(Collectors.toList());
        }

        /**
         * Updates an existing parcel identified by its ID. The cache for this parcel is updated.
         * @param id The UUID of the parcel to update.
         * @param parcelRequest The DTO containing the updated details.
         * @return A ParcelResponse DTO of the updated parcel.
         * @throws ParcelNotFoundException If no parcel with the given ID is found.
         */
        @Transactional
        @CachePut(value = "parcels", key = "#id")
        public ParcelResponse updateParcel(UUID id, ParcelRequest parcelRequest) {
            log.info("Updating parcel with ID: {}", id);
            Parcel existingParcel = parcelRepository.findById(id)
                    .orElseThrow(() -> {
                        log.warn("Parcel not found for update with ID: {}", id);
                        return new ParcelNotFoundException("Parcel not found with ID: " + id);
                    });

            existingParcel.setSenderName(parcelRequest.getSenderName());
            existingParcel.setSenderAddress(parcelRequest.getSenderAddress());
            existingParcel.setRecipientName(parcelRequest.getRecipientName());
            existingParcel.setRecipientAddress(parcelRequest.getRecipientAddress());
            existingParcel.setWeight(parcelRequest.getWeight());
            existingParcel.setDescription(parcelRequest.getDescription());
            existingParcel.setLastUpdatedAt(LocalDateTime.now());

            Parcel updatedParcel = parcelRepository.save(existingParcel);
            log.debug("Parcel with ID: {} updated successfully", id);
            return mapToDto(updatedParcel);
        }

        /**
         * Updates the status of a specific parcel.
         * @param id The UUID of the parcel to update.
         * @param newStatus The new status to set for the parcel.
         * @return A ParcelResponse DTO of the updated parcel.
         * @throws ParcelNotFoundException If no parcel with the given ID is found.
         * @throws InvalidParcelStatusException If the status transition is invalid.
         */
        @Transactional
        @CachePut(value = "parcels", key = "#id")
        public ParcelResponse updateParcelStatus(UUID id, ParcelStatus newStatus) {
            log.info("Updating status for parcel ID: {} to {}", id, newStatus);
            Parcel existingParcel = parcelRepository.findById(id)
                    .orElseThrow(() -> {
                        log.warn("Parcel not found for status update with ID: {}", id);
                        return new ParcelNotFoundException("Parcel not found with ID: " + id);
                    });

            // Basic status transition logic (can be expanded)
            if (existingParcel.getStatus() == ParcelStatus.DELIVERED && newStatus != ParcelStatus.DELIVERED) {
                log.warn("Attempted to change status of already DELIVERED parcel ID: {}", id);
                throw new InvalidParcelStatusException("Cannot change status of a delivered parcel.");
            }
            existingParcel.setStatus(newStatus);
            existingParcel.setLastUpdatedAt(LocalDateTime.now());

            Parcel updatedParcel = parcelRepository.save(existingParcel);
            log.debug("Parcel ID: {} status updated to {}", id, newStatus);
            return mapToDto(updatedParcel);
        }

        /**
         * Deletes a parcel by its unique identifier. The cache for this parcel is evicted.
         * @param id The UUID of the parcel to delete.
         * @throws ParcelNotFoundException If no parcel with the given ID is found.
         */
        @Transactional
        @CacheEvict(value = "parcels", key = "#id")
        public void deleteParcel(UUID id) {
            log.info("Deleting parcel with ID: {}", id);
            if (!parcelRepository.existsById(id)) {
                log.warn("Parcel not found for deletion with ID: {}", id);
                throw new ParcelNotFoundException("Parcel not found with ID: " + id);
            }
            parcelRepository.deleteById(id);
            log.debug("Parcel with ID: {} deleted successfully", id);
        }

        /**
         * Maps a ParcelRequest DTO to a Parcel entity.
         * @param request The ParcelRequest DTO.
         * @return A Parcel entity.
         */
        private Parcel mapToEntity(ParcelRequest request) {
            return Parcel.builder()
                    .senderName(request.getSenderName())
                    .senderAddress(request.getSenderAddress())
                    .recipientName(request.getRecipientName())
                    .recipientAddress(request.getRecipientAddress())
                    .weight(request.getWeight())
                    .description(request.getDescription())
                    .build();
        }

        /**
         * Maps a Parcel entity to a ParcelResponse DTO.
         * @param parcel The Parcel entity.
         * @return A ParcelResponse DTO.
         */
        private ParcelResponse mapToDto(Parcel parcel) {
            return ParcelResponse.builder()
                    .id(parcel.getId())
                    .trackingNumber(parcel.getTrackingNumber())
                    .senderName(parcel.getSenderName())
                    .senderAddress(parcel.getSenderAddress())
                    .recipientName(parcel.getRecipientName())
                    .recipientAddress(parcel.getRecipientAddress())
                    .weight(parcel.getWeight())
                    .description(parcel.getDescription())
                    .status(parcel.getStatus())
                    .createdAt(parcel.getCreatedAt())
                    .lastUpdatedAt(parcel.getLastUpdatedAt())
                    .build();
        }
    }
    ```

### Step 3: Data Persistence (Repository & Model Layer)

1.  **Create Entity**: `Parcel.java`
    ```java
    package com.hcl.parcel.service.model;

    import jakarta.persistence.Column;
    import jakarta.persistence.Entity;
    import jakarta.persistence.EnumType;
    import jakarta.persistence.Enumerated;
    import jakarta.persistence.GeneratedValue;
    import jakarta.persistence.GenerationType;
    import jakarta.persistence.Id;
    import jakarta.persistence.Table;
    import lombok.AllArgsConstructor;
    import lombok.Builder;
    import lombok.Getter;
    import lombok.NoArgsConstructor;
    import lombok.Setter;
    import lombok.ToString;

    import java.time.LocalDateTime;
    import java.util.UUID;

    @Entity
    @Table(name = "parcels")
    @Getter
    @Setter
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    @ToString // Use with caution, avoid exposing sensitive data in logs
    public class Parcel {

        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        @Column(name = "id", updatable = false, nullable = false)
        private UUID id;

        @Column(name = "tracking_number", unique = true, nullable = false, length = 20)
        private String trackingNumber;

        @Column(name = "sender_name", nullable = false, length = 100)
        private String senderName;

        @Column(name = "sender_address", nullable = false, length = 255)
        private String senderAddress;

        @Column(name = "recipient_name", nullable = false, length = 100)
        private String recipientName;

        @Column(name = "recipient_address", nullable = false, length = 255)
        private String recipientAddress;

        @Column(name = "weight_kg", nullable = false)
        private Double weight;

        @Column(name = "description", length = 50)
        private String description;

        @Enumerated(EnumType.STRING)
        @Column(name = "status", nullable = false, length = 20)
        private ParcelStatus status;

        @Column(name = "created_at", nullable = false, updatable = false)
        private LocalDateTime createdAt;

        @Column(name = "last_updated_at", nullable = false)
        private LocalDateTime lastUpdatedAt;

        // Constructors, getters, setters provided by Lombok annotations
    }
    ```
    `ParcelStatus.java` (Enum):
    ```java
    package com.hcl.parcel.service.model;

    public enum ParcelStatus {
        PENDING,
        IN_TRANSIT,
        DELIVERED,
        CANCELLED,
        RETURNED
    }
    ```

2.  **Create Repository**: `ParcelRepository.java`
    ```java
    package com.hcl.parcel.service.repository;

    import com.hcl.parcel.service.model.Parcel;
    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.stereotype.Repository;

    import java.util.Optional;
    import java.util.UUID;

    @Repository
    public interface ParcelRepository extends JpaRepository<Parcel, UUID> {
        // Custom query example: Find a parcel by its tracking number
        Optional<Parcel> findByTrackingNumber(String trackingNumber);
    }
    ```

### Step 4: Exception Handling

1.  **Custom Exceptions**:
    *   `ParcelNotFoundException.java`
        ```java
        package com.hcl.parcel.service.exception;

        import org.springframework.http.HttpStatus;
        import org.springframework.web.bind.annotation.ResponseStatus;

        @ResponseStatus(HttpStatus.NOT_FOUND)
        public class ParcelNotFoundException extends RuntimeException {
            public ParcelNotFoundException(String message) {
                super(message);
            }
        }
        ```
    *   `InvalidParcelStatusException.java`
        ```java
        package com.hcl.parcel.service.exception;

        import org.springframework.http.HttpStatus;
        import org.springframework.web.bind.annotation.ResponseStatus;

        @ResponseStatus(HttpStatus.BAD_REQUEST)
        public class InvalidParcelStatusException extends RuntimeException {
            public InvalidParcelStatusException(String message) {
                super(message);
            }
        }
        ```
    *   `ErrorResponse.java` DTO
        ```java
        package com.hcl.parcel.service.dto;

        import lombok.AllArgsConstructor;
        import lombok.Builder;
        import lombok.Getter;
        import lombok.NoArgsConstructor;
        import lombok.Setter;

        import java.time.LocalDateTime;

        @Getter
        @Setter
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public class ErrorResponse {
            private LocalDateTime timestamp;
            private int status;
            private String error;
            private String message;
            private String path;
        }
        ```

2.  **Global Handler**: `GlobalExceptionHandler.java`
    ```java
    package com.hcl.parcel.service.exception;

    import com.hcl.parcel.service.dto.ErrorResponse;
    import jakarta.servlet.http.HttpServletRequest;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.RestControllerAdvice;

    import java.time.LocalDateTime;
    import java.util.stream.Collectors;

    @RestControllerAdvice
    @Slf4j
    public class GlobalExceptionHandler {

        /**
         * Handles ParcelNotFoundException.
         * Returns 404 Not Found.
         * @param ex The exception thrown.
         * @param request The incoming request.
         * @return ResponseEntity with ErrorResponse.
         */
        @ExceptionHandler(ParcelNotFoundException.class)
        public ResponseEntity<ErrorResponse> handleParcelNotFoundException(ParcelNotFoundException ex, HttpServletRequest request) {
            log.warn("ParcelNotFoundException: {}", ex.getMessage());
            ErrorResponse error = ErrorResponse.builder()
                    .timestamp(LocalDateTime.now())
                    .status(HttpStatus.NOT_FOUND.value())
                    .error(HttpStatus.NOT_FOUND.getReasonPhrase())
                    .message(ex.getMessage())
                    .path(request.getRequestURI())
                    .build();
            return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
        }

        /**
         * Handles InvalidParcelStatusException.
         * Returns 400 Bad Request.
         * @param ex The exception thrown.
         * @param request The incoming request.
         * @return ResponseEntity with ErrorResponse.
         */
        @ExceptionHandler(InvalidParcelStatusException.class)
        public ResponseEntity<ErrorResponse> handleInvalidParcelStatusException(InvalidParcelStatusException ex, HttpServletRequest request) {
            log.warn("InvalidParcelStatusException: {}", ex.getMessage());
            ErrorResponse error = ErrorResponse.builder()
                    .timestamp(LocalDateTime.now())
                    .status(HttpStatus.BAD_REQUEST.value())
                    .error(HttpStatus.BAD_REQUEST.getReasonPhrase())
                    .message(ex.getMessage())
                    .path(request.getRequestURI())
                    .build();
            return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
        }

        /**
         * Handles validation errors from @Valid annotations.
         * Returns 400 Bad Request.
         * @param ex The MethodArgumentNotValidException thrown.
         * @param request The incoming request.
         * @return ResponseEntity with ErrorResponse.
         */
        @ExceptionHandler(MethodArgumentNotValidException.class)
        public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex, HttpServletRequest request) {
            String errorMessage = ex.getBindingResult().getFieldErrors().stream()
                    .map(error -> error.getField() + ": " + error.getDefaultMessage())
                    .collect(Collectors.joining("; "));
            log.warn("Validation failed: {}", errorMessage);

            ErrorResponse error = ErrorResponse.builder()
                    .timestamp(LocalDateTime.now())
                    .status(HttpStatus.BAD_REQUEST.value())
                    .error(HttpStatus.BAD_REQUEST.getReasonPhrase())
                    .message("Validation failed: " + errorMessage)
                    .path(request.getRequestURI())
                    .build();
            return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
        }

        /**
         * Catches all other unhandled exceptions.
         * Returns 500 Internal Server Error.
         * @param ex The generic Exception thrown.
         * @param request The incoming request.
         * @return ResponseEntity with ErrorResponse.
         */
        @ExceptionHandler(Exception.class)
        public ResponseEntity<ErrorResponse> handleGenericException(Exception ex, HttpServletRequest request) {
            log.error("An unexpected error occurred: {}", ex.getMessage(), ex); // Log full stack trace internally
            ErrorResponse error = ErrorResponse.builder()
                    .timestamp(LocalDateTime.now())
                    .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
                    .error(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase())
                    .message("An unexpected error occurred. Please try again later.") // Generic message for client
                    .path(request.getRequestURI())
                    .build();
            return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
    ```

### Step 5: Caching

1.  **Enable Caching**: Add `@EnableCaching` to `ParcelServiceApplication.java`.

2.  **Configure Cache**: `CacheConfig.java`
    ```java
    package com.hcl.parcel.service.config;

    import com.github.benmanes.caffeine.cache.Caffeine;
    import org.springframework.cache.CacheManager;
    import org.springframework.cache.caffeine.CaffeineCacheManager;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    import java.util.concurrent.TimeUnit;

    @Configuration
    public class CacheConfig {

        @Bean
        public CacheManager cacheManager() {
            CaffeineCacheManager cacheManager = new CaffeineCacheManager("parcels");
            cacheManager.setCaffeine(caffeineCacheBuilder());
            return cacheManager;
        }

        Caffeine<Object, Object> caffeineCacheBuilder() {
            return Caffeine.newBuilder()
                    .initialCapacity(100)
                    .maximumSize(500)
                    .expireAfterAccess(10, TimeUnit.MINUTES)
                    .recordStats();
        }
    }
    ```

3.  **Apply Caching**: Used in `ParcelService.java` with `@Cacheable`, `@CachePut`, `@CacheEvict` as shown in **Step 2**.

### Step 6: Configuration

*   **`application.properties`**:
    ```properties
    spring.application.name=parcel-service
    server.port=8080

    # Actuator Endpoints
    management.endpoints.web.exposure.include=health,info,prometheus
    management.endpoint.health.show-details=always
    management.endpoint.health.show-components=always

    # SpringDoc OpenAPI
    springdoc.swagger-ui.path=/swagger-ui.html
    springdoc.api-docs.path=/v3/api-docs
    ```

*   **`application-dev.properties`**:
    ```properties
    # H2 Database Configuration (In-memory for development)
    spring.datasource.url=jdbc:h2:mem:parceldb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    spring.datasource.driver-class-name=org.h2.Driver
    spring.datasource.username=sa
    spring.datasource.password=
    spring.h2.console.enabled=true
    spring.h2.console.path=/h2-console

    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.show-sql=true
    spring.jpa.properties.hibernate.format_sql=true

    # Logging Level
    logging.level.com.hcl.parcel.service=DEBUG
    logging.level.org.springframework.web=INFO
    logging.level.org.hibernate.SQL=DEBUG
    logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

    # Caching (Caffeine is default, Redis could be added for dev)
    # spring.cache.type=caffeine
    ```

*   **`application-prod.properties`**:
    ```properties
    # PostgreSQL Database Configuration
    spring.datasource.url=${DB_URL:jdbc:postgresql://localhost:5432/parceldb}
    spring.datasource.username=${DB_USERNAME:parceluser}
    spring.datasource.password=${DB_PASSWORD:parcelpass}
    spring.datasource.driver-class-name=org.postgresql.Driver

    spring.jpa.hibernate.ddl-auto=validate # No automatic schema changes in production
    spring.jpa.show-sql=false

    # Logging Level (for production)
    logging.level.com.hcl.parcel.service=INFO
    logging.level.org.springframework.web=INFO

    # Management endpoints for production
    management.endpoint.health.show-details=never # Do not expose sensitive details
    management.endpoint.health.show-components=never

    # Distributed Caching (e.g., Redis)
    # spring.cache.type=redis
    # spring.data.redis.host=${REDIS_HOST:localhost}
    # spring.data.redis.port=${REDIS_PORT:6379}
    ```

### Step 7: `catalog-info.yaml` File Structure

`catalog-info.yaml` (placed in the root directory of the `parcel-service` repository)

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: parcel-service # Unique name for the component
  description: Manages the lifecycle and tracking of parcels. # Human-readable description
  tags:
    - spring-boot
    - microservice
    - logistics
    - parcels
  annotations:
    # Example annotations for integration with CI/CD, monitoring, etc.
    gitlab.com/project-slug: hcl/logistics-services/parcel-service
    sonarqube.org/project-key: hcl_parcel-service
    grafana/dashboard-url: https://grafana.example.com/d/parcel-service
  links:
    - url: https://gitlab.com/hcl/logistics-services/parcel-service
      title: Git Repository
      icon: code
    - url: https://confluence.hcl.com/pages/viewpage.action?pageId=123456
      title: Design Document
      icon: document
    - url: https://parcel-service.dev.example.com/swagger-ui.html
      title: OpenAPI Docs (Dev)
      icon: api
spec:
  type: service # This component is a service
  lifecycle: production # The service is in production
  owner: hcl-logistics-team # The team responsible for this service
  system: logistics-management # The larger system this component belongs to
  dependsOn:
    - resource:database:parceldb-postgres # Depends on the PostgreSQL database
    # Add other dependencies like messaging queues, other services, etc.
  providesApis:
    - parcel-api # The API defined by this service
```

### Step 8: #Coding standards#

The following coding standards are mandatory for the "Parcel Management Service" and all Java Spring Boot projects. These rules enhance readability, maintainability, and consistency.

```
# Spring Boot Coding Standards

This document outlines the mandatory coding standards for Spring Boot applications to ensure consistency, maintainability, and high quality across all projects.

## 1. General Java Best Practices

-   **Formatting**: Adhere to Google Java Format or an equivalent consistent formatter.
    -   *Rule*: Use a formatter to automatically apply code style. Configure IDE to format on save.
-   **Naming Conventions**: Follow standard Java naming conventions (camelCase for variables/methods, PascalCase for classes, UPPER_SNAKE_CASE for constants).
    -   *Rule*: Clear, descriptive names that reflect purpose. Avoid single-letter names unless for loop counters.
-   **Imports**: Organize imports. Remove unused imports.
    -   *Rule*: Use IDE features to optimize imports (e.g., Ctrl+Alt+O in IntelliJ).
-   **Comments**: Use Javadoc for public APIs (classes, methods). Use inline comments sparingly for complex logic or workarounds.
    -   *Rule*: Explain *why* the code does something, not *what* it does (unless complex).
-   **Avoid Magic Numbers/Strings**: Use constants or enums for literal values.
    -   *Rule*: Define constants in a dedicated `Constants` class or within the class where they are used.
-   **Error Handling**: Prefer specific exceptions over generic ones. Catch exceptions at the appropriate layer.
    -   *Rule*: Do not catch `Exception` or `Throwable` broadly.
-   **Logger Usage**: Use SLF4J for logging. Inject `Logger` instances using `@Slf4j` (Lombok) or `LoggerFactory.getLogger()`.
    -   *Rule*: Log at appropriate levels (DEBUG, INFO, WARN, ERROR). Avoid logging sensitive information.

## 2. Spring Boot Specifics

-   **Dependency Management**: Use Maven/Gradle for dependency management. Rely on Spring Boot Starters for common dependencies.
    -   *Rule*: Define dependency versions in `pom.xml` or `build.gradle` and use Spring Boot's BOM for consistent versions.
-   **Configuration**:
    -   **Externalize Configuration**: Use `application.properties`, `application.yml`, or environment variables for all configurable parameters.
    -   *Rule*: Never hardcode secrets or environment-specific values. Use `@Value` or `@ConfigurationProperties`.
    -   **Profiles**: Leverage Spring profiles (`application-dev.properties`, `application-prod.properties`) for environment-specific configurations.
    -   *Rule*: Define sensible defaults in `application.properties` and override in profile-specific files.
-   **Component Scan**:
    -   *Rule*: Keep the main application class (`@SpringBootApplication`) in the root package to allow effective component scanning.
-   **Lombok**: Use Lombok annotations judiciously.
    -   *Rule*: `@Getter`, `@Setter`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@RequiredArgsConstructor`, `@Slf4j` are preferred.
    -   *Avoid*: `@Data` on JPA entities to prevent issues with `@EqualsAndHashCode` and `@ToString` default implementations causing performance or stack overflow issues. Explicitly use specific annotations.
-   **Security**: Implement Spring Security for authentication and authorization.
    -   *Rule*: Follow principle of least privilege. Secure all endpoints by default and open specifically.
-   **Actuator**: Include Spring Boot Actuator for monitoring and management.
    -   *Rule*: Configure which endpoints are exposed and ensure sensitive endpoints are secured in production.

## 3. REST API Guidelines

-   **API Versioning**: Prefix all REST routes with `/api/v1` (or appropriate version).
    -   *Rule*: Use URI versioning (`/api/v1/resource`) for simplicity.
-   **Endpoint Design**:
    -   *Rule*: Use plural nouns for resource paths (e.g., `/api/v1/users`, `/api/v1/products`).
    -   *Rule*: Use HTTP methods (GET, POST, PUT, DELETE, PATCH) correctly for CRUD operations.
-   **DTOs (Data Transfer Objects)**:
    -   *Rule*: Always use DTOs for request and response payloads. Never expose JPA entities directly through the API.
    -   *Rule*: Apply Jakarta Bean Validation annotations (`@NotBlank`, `@Size`, `@NotNull`, `@Email`, etc.) on incoming DTOs.
-   **Response Handling**:
    -   *Rule*: Return `ResponseEntity<?>` from controllers to explicitly control HTTP status codes and headers.
    -   *Rule*: Use appropriate HTTP status codes (200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error).
-   **Exception Handling**:
    -   *Rule*: Implement global exception handling using `@RestControllerAdvice` to centralize error responses.
    -   *Rule*: Return standardized error payloads (e.g., an `ErrorResponse` DTO with timestamp, status, error message).
    -   *Rule*: Never expose internal stack traces to the client in production.

## 4. Service Layer

-   **Transaction Management**:
    -   *Rule*: Apply `@Transactional` (from `org.springframework.transaction.annotation`) to service methods that perform data modifications.
    -   *Rule*: Use `@Transactional(readOnly = true)` for read-only operations to optimize performance and prevent accidental writes.
-   **Business Logic Encapsulation**:
    -   *Rule*: The service layer should contain the core business logic, orchestrating data access, DTO-to-entity mapping, and applying business rules.
-   **Dependency Injection**:
    -   *Rule*: Prefer constructor injection (`@RequiredArgsConstructor` with `final` fields) over field injection for improved testability and immutability.

## 5. Persistence Layer (JPA/Hibernate)

-   **Entities**:
    -   *Rule*: Entities (`@Entity`) should be simple POJOs. Avoid business logic directly within entities.
    -   *Rule*: Use specific Lombok annotations (`@Getter`, `@Setter`, `ToString` if needed, but carefully) instead of `@Data` on entities.
    -   *Rule*: Ensure proper `@Id`, `@GeneratedValue`, and `@Column` annotations.
-   **Repositories**:
    -   *Rule*: Extend Spring Data JPA's `JpaRepository` for common CRUD operations.
    -   *Rule*: Define custom query methods adhering to Spring Data JPA's naming conventions or use `@Query` annotations for complex queries.
-   **N+1 Problem**:
    -   *Rule*: Be aware of and address the N+1 select problem using `FetchType.LAZY` and `JOIN FETCH` queries where appropriate.

## 6. Testing

-   **Unit Tests**:
    -   *Rule*: Write unit tests for individual components (services, controllers, utilities) by mocking dependencies.
    -   *Rule*: Use JUnit 5 and Mockito.
-   **Integration Tests**:
    -   *Rule*: Write integration tests for critical flows using `@SpringBootTest` to load the full application context.
    -   *Rule*: Use `@ActiveProfiles("test")` and dedicated test configurations.
    -   *Rule*: Employ Testcontainers for external dependencies (databases, message queues) to ensure reliable and isolated testing.
-   **Test Coverage**:
    -   *Rule*: Aim for a minimum of 80% test coverage for critical business logic.

## 7. Documentation

-   **OpenAPI/Swagger**:
    -   *Rule*: Use SpringDoc OpenAPI to generate interactive API documentation.
    -   *Rule*: Annotate controllers and DTOs with `@Operation`, `@ApiResponse`, `@Schema` to enrich documentation.
-   **Javadoc**:
    -   *Rule*: All public classes, interfaces, methods, and important fields must have comprehensive Javadoc comments.
```

### Step 9: Documentation

*   **OpenAPI**: `OpenApiConfig.java`
    ```java
    package com.hcl.parcel.service.config;

    import io.swagger.v3.oas.models.OpenAPI;
    import io.swagger.v3.oas.models.info.Contact;
    import io.swagger.v3.oas.models.info.Info;
    import io.swagger.v3.oas.models.info.License;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class OpenApiConfig {

        @Bean
        public OpenAPI customOpenAPI() {
            return new OpenAPI()
                    .info(new Info()
                            .title("Parcel Management API")
                            .version("1.0.0")
                            .description("API for managing parcel shipments, including creation, tracking, and status updates.")
                            .termsOfService("http://swagger.io/terms/")
                            .contact(new Contact()
                                    .name("HCL Logistics Team")
                                    .url("https://hcl.com/support")
                                    .email("logistics.support@hcl.com"))
                            .license(new License()
                                    .name("Apache 2.0")
                                    .url("http://springdoc.org")));
        }
    }
    ```
*   **JavaDoc**: Examples provided throughout `ParcelController.java` and `ParcelService.java`. Every public method should have `@param`, `@return`, and `@throws` where applicable.

### Step 10: Testing

*   **Unit Tests**: `ParcelServiceTest.java` (using Mockito for `ParcelRepository`).
    ```java
    package com.hcl.parcel.service.service;

    import com.hcl.parcel.service.dto.ParcelRequest;
    import com.hcl.parcel.service.dto.ParcelResponse;
    import com.hcl.parcel.service.exception.InvalidParcelStatusException;
    import com.hcl.parcel.service.exception.ParcelNotFoundException;
    import com.hcl.parcel.service.model.Parcel;
    import com.hcl.parcel.service.model.ParcelStatus;
    import com.hcl.parcel.service.repository.ParcelRepository;
    import org.junit.jupiter.api.BeforeEach;
    import org.junit.jupiter.api.Test;
    import org.junit.jupiter.api.extension.ExtendWith;
    import org.mockito.InjectMocks;
    import org.mockito.Mock;
    import org.mockito.junit.jupiter.MockitoExtension;

    import java.time.LocalDateTime;
    import java.util.Arrays;
    import java.util.List;
    import java.util.Optional;
    import java.util.UUID;

    import static org.junit.jupiter.api.Assertions.*;
    import static org.mockito.ArgumentMatchers.any;
    import static org.mockito.Mockito.*;

    @ExtendWith(MockitoExtension.class)
    class ParcelServiceTest {

        @Mock
        private ParcelRepository parcelRepository;

        @InjectMocks
        private ParcelService parcelService;

        private UUID parcelId;
        private Parcel parcel;
        private ParcelRequest parcelRequest;

        @BeforeEach
        void setUp() {
            parcelId = UUID.randomUUID();
            parcel = Parcel.builder()
                    .id(parcelId)
                    .trackingNumber("TRK12345")
                    .senderName("John Doe")
                    .senderAddress("123 Main St")
                    .recipientName("Jane Smith")
                    .recipientAddress("456 Oak Ave")
                    .weight(5.0)
                    .description("Book")
                    .status(ParcelStatus.PENDING)
                    .createdAt(LocalDateTime.now())
                    .lastUpdatedAt(LocalDateTime.now())
                    .build();

            parcelRequest = ParcelRequest.builder()
                    .senderName("John Doe")
                    .senderAddress("123 Main St")
                    .recipientName("Jane Smith")
                    .recipientAddress("456 Oak Ave")
                    .weight(5.0)
                    .description("Book")
                    .build();
        }

        @Test
        void createParcel_shouldReturnCreatedParcel() {
            when(parcelRepository.save(any(Parcel.class))).thenReturn(parcel);

            ParcelResponse response = parcelService.createParcel(parcelRequest);

            assertNotNull(response);
            assertEquals(parcel.getSenderName(), response.getSenderName());
            assertEquals(ParcelStatus.PENDING, response.getStatus());
            verify(parcelRepository, times(1)).save(any(Parcel.class));
        }

        @Test
        void getParcelById_whenParcelExists_shouldReturnParcel() {
            when(parcelRepository.findById(parcelId)).thenReturn(Optional.of(parcel));

            ParcelResponse response = parcelService.getParcelById(parcelId);

            assertNotNull(response);
            assertEquals(parcelId, response.getId());
            assertEquals("TRK12345", response.getTrackingNumber());
            verify(parcelRepository, times(1)).findById(parcelId);
        }

        @Test
        void getParcelById_whenParcelDoesNotExist_shouldThrowException() {
            when(parcelRepository.findById(parcelId)).thenReturn(Optional.empty());

            assertThrows(ParcelNotFoundException.class, () -> parcelService.getParcelById(parcelId));
            verify(parcelRepository, times(1)).findById(parcelId);
        }

        @Test
        void getAllParcels_shouldReturnListOfParcels() {
            when(parcelRepository.findAll()).thenReturn(Arrays.asList(parcel, parcel));

            List<ParcelResponse> responses = parcelService.getAllParcels();

            assertNotNull(responses);
            assertEquals(2, responses.size());
            verify(parcelRepository, times(1)).findAll();
        }

        @Test
        void updateParcel_whenParcelExists_shouldReturnUpdatedParcel() {
            ParcelRequest updatedRequest = ParcelRequest.builder()
                    .senderName("Updated John Doe")
                    .senderAddress("Updated 123 Main St")
                    .recipientName("Updated Jane Smith")
                    .recipientAddress("Updated 456 Oak Ave")
                    .weight(6.0)
                    .description("Updated Book")
                    .build();
            Parcel updatedParcel = Parcel.builder()
                    .id(parcelId)
                    .trackingNumber(parcel.getTrackingNumber())
                    .senderName("Updated John Doe")
                    .senderAddress("Updated 123 Main St")
                    .recipientName("Updated Jane Smith")
                    .recipientAddress("Updated 456 Oak Ave")
                    .weight(6.0)
                    .description("Updated Book")
                    .status(parcel.getStatus())
                    .createdAt(parcel.getCreatedAt())
                    .lastUpdatedAt(LocalDateTime.now())
                    .build();

            when(parcelRepository.findById(parcelId)).thenReturn(Optional.of(parcel));
            when(parcelRepository.save(any(Parcel.class))).thenReturn(updatedParcel);

            ParcelResponse response = parcelService.updateParcel(parcelId, updatedRequest);

            assertNotNull(response);
            assertEquals("Updated John Doe", response.getSenderName());
            assertEquals(6.0, response.getWeight());
            verify(parcelRepository, times(1)).findById(parcelId);
            verify(parcelRepository, times(1)).save(any(Parcel.class));
        }

        @Test
        void updateParcel_whenParcelDoesNotExist_shouldThrowException() {
            when(parcelRepository.findById(parcelId)).thenReturn(Optional.empty());

            assertThrows(ParcelNotFoundException.class, () -> parcelService.updateParcel(parcelId, parcelRequest));
            verify(parcelRepository, times(1)).findById(parcelId);
            verify(parcelRepository, never()).save(any(Parcel.class));
        }

        @Test
        void updateParcelStatus_whenValidTransition_shouldUpdateStatus() {
            parcel.setStatus(ParcelStatus.IN_TRANSIT);
            when(parcelRepository.findById(parcelId)).thenReturn(Optional.of(parcel));
            when(parcelRepository.save(any(Parcel.class))).thenReturn(parcel);

            ParcelResponse response = parcelService.updateParcelStatus(parcelId, ParcelStatus.DELIVERED);

            assertNotNull(response);
            assertEquals(ParcelStatus.DELIVERED, response.getStatus());
            verify(parcelRepository, times(1)).findById(parcelId);
            verify(parcelRepository, times(1)).save(any(Parcel.class));
        }

        @Test
        void updateParcelStatus_whenInvalidTransition_shouldThrowException() {
            parcel.setStatus(ParcelStatus.DELIVERED); // Already delivered
            when(parcelRepository.findById(parcelId)).thenReturn(Optional.of(parcel));

            assertThrows(InvalidParcelStatusException.class, () -> parcelService.updateParcelStatus(parcelId, ParcelStatus.IN_TRANSIT));
            verify(parcelRepository, times(1)).findById(parcelId);
            verify(parcelRepository, never()).save(any(Parcel.class));
        }


        @Test
        void deleteParcel_whenParcelExists_shouldDeleteParcel() {
            when(parcelRepository.existsById(parcelId)).thenReturn(true);
            doNothing().when(parcelRepository).deleteById(parcelId);

            parcelService.deleteParcel(parcelId);

            verify(parcelRepository, times(1)).existsById(parcelId);
            verify(parcelRepository, times(1)).deleteById(parcelId);
        }

        @Test
        void deleteParcel_whenParcelDoesNotExist_shouldThrowException() {
            when(parcelRepository.existsById(parcelId)).thenReturn(false);

            assertThrows(ParcelNotFoundException.class, () -> parcelService.deleteParcel(parcelId));
            verify(parcelRepository, times(1)).existsById(parcelId);
            verify(parcelRepository, never()).deleteById(parcelId);
        }
    }
    ```
*   **Integration Tests**: `ParcelControllerIntegrationTest.java` (using `@SpringBootTest`, `Testcontainers` for PostgreSQL).
    ```java
    package com.hcl.parcel.service.controller;

    import com.fasterxml.jackson.databind.ObjectMapper;
    import com.hcl.parcel.service.dto.ParcelRequest;
    import com.hcl.parcel.service.dto.ParcelStatusUpdateRequest;
    import com.hcl.parcel.service.model.ParcelStatus;
    import com.hcl.parcel.service.repository.ParcelRepository;
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

    import java.util.UUID;

    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
    import static org.hamcrest.Matchers.*;

    @SpringBootTest
    @AutoConfigureMockMvc
    @ActiveProfiles("test")
    @Testcontainers
    class ParcelControllerIntegrationTest {

        @Autowired
        private MockMvc mockMvc;

        @Autowired
        private ObjectMapper objectMapper;

        @Autowired
        private ParcelRepository parcelRepository; // Use repository to set up and tear down test data

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
            registry.add("spring.jpa.hibernate.ddl-auto", () -> "create-drop");
        }

        @BeforeEach
        void setUp() {
            parcelRepository.deleteAll(); // Ensure a clean state before each test
        }

        @AfterEach
        void tearDown() {
            parcelRepository.deleteAll(); // Clean up after each test
        }

        @Test
        void createParcel_shouldReturnCreatedParcel() throws Exception {
            ParcelRequest request = ParcelRequest.builder()
                    .senderName("Test Sender")
                    .senderAddress("123 Test St")
                    .recipientName("Test Recipient")
                    .recipientAddress("456 Test Ave")
                    .weight(10.5)
                    .description("Test Item")
                    .build();

            mockMvc.perform(post("/api/v1/parcels")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(request)))
                    .andExpect(status().isCreated())
                    .andExpect(jsonPath("$.id").exists())
                    .andExpect(jsonPath("$.trackingNumber").exists())
                    .andExpect(jsonPath("$.senderName", is("Test Sender")))
                    .andExpect(jsonPath("$.status", is("PENDING")));
        }

        @Test
        void getParcelById_whenParcelExists_shouldReturnParcel() throws Exception {
            // Setup a parcel directly via repository
            ParcelRequest createRequest = ParcelRequest.builder()
                    .senderName("Existing Sender").senderAddress("Addr1")
                    .recipientName("Existing Recipient").recipientAddress("Addr2")
                    .weight(2.0).description("Exist Item").build();
            UUID parcelId = parcelRepository.save(parcelService.mapToEntity(createRequest)).getId();

            mockMvc.perform(get("/api/v1/parcels/{id}", parcelId)
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.id", is(parcelId.toString())))
                    .andExpect(jsonPath("$.senderName", is("Existing Sender")));
        }

        @Test
        void getParcelById_whenParcelDoesNotExist_shouldReturnNotFound() throws Exception {
            UUID nonExistentId = UUID.randomUUID();
            mockMvc.perform(get("/api/v1/parcels/{id}", nonExistentId)
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isNotFound())
                    .andExpect(jsonPath("$.message", containsString("Parcel not found")));
        }

        @Test
        void updateParcel_shouldReturnUpdatedParcel() throws Exception {
            // Setup initial parcel
            ParcelRequest initialRequest = ParcelRequest.builder()
                    .senderName("Initial Sender").senderAddress("Initial Addr1")
                    .recipientName("Initial Recipient").recipientAddress("Initial Addr2")
                    .weight(1.0).description("Initial Item").build();
            UUID parcelId = parcelRepository.save(parcelService.mapToEntity(initialRequest)).getId();

            // Prepare update request
            ParcelRequest updateRequest = ParcelRequest.builder()
                    .senderName("Updated Sender").senderAddress("Updated Addr1")
                    .recipientName("Updated Recipient").recipientAddress("Updated Addr2")
                    .weight(3.0).description("Updated Item").build();

            mockMvc.perform(put("/api/v1/parcels/{id}", parcelId)
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(updateRequest)))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.id", is(parcelId.toString())))
                    .andExpect(jsonPath("$.senderName", is("Updated Sender")))
                    .andExpect(jsonPath("$.weight", is(3.0)));
        }

        @Test
        void updateParcelStatus_shouldUpdateStatus() throws Exception {
            // Setup initial parcel
            ParcelRequest initialRequest = ParcelRequest.builder()
                    .senderName("Status Sender").senderAddress("Status Addr1")
                    .recipientName("Status Recipient").recipientAddress("Status Addr2")
                    .weight(1.0).description("Status Item").build();
            UUID parcelId = parcelRepository.save(parcelService.mapToEntity(initialRequest)).getId();

            // Prepare status update request
            ParcelStatusUpdateRequest statusUpdateRequest = ParcelStatusUpdateRequest.builder()
                    .newStatus(ParcelStatus.IN_TRANSIT).build();

            mockMvc.perform(patch("/api/v1/parcels/{id}/status", parcelId)
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(statusUpdateRequest)))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.id", is(parcelId.toString())))
                    .andExpect(jsonPath("$.status", is("IN_TRANSIT")));
        }

        @Test
        void deleteParcel_shouldReturnNoContent() throws Exception {
            // Setup a parcel to delete
            ParcelRequest deleteRequest = ParcelRequest.builder()
                    .senderName("Delete Sender").senderAddress("Delete Addr1")
                    .recipientName("Delete Recipient").recipientAddress("Delete Addr2")
                    .weight(1.0).description("Delete Item").build();
            UUID parcelId = parcelRepository.save(parcelService.mapToEntity(deleteRequest)).getId();

            mockMvc.perform(delete("/api/v1/parcels/{id}", parcelId)
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isNoContent());

            // Verify it's actually deleted
            mockMvc.perform(get("/api/v1/parcels/{id}", parcelId)
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isNotFound());
        }
    }
    ```
*   **Coverage**: Aim for 80% minimum coverage on `com.hcl.parcel.service` package.

---

## 5. 🐳 Containerization & Deployment (Parcel Management Service)

### Dockerfile

`Dockerfile` (in the root directory of the project)

```dockerfile
# Stage 1: Build the application
FROM maven:3.9.6-eclipse-temurin-21-alpine AS builder

WORKDIR /app

# Copy the Maven project files
COPY pom.xml .
COPY src ./src

# Build the Spring Boot application
RUN mvn clean install -DskipTests

# Stage 2: Create the final image
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Create a non-root user and group
RUN addgroup --system springuser && adduser --system --ingroup springuser springuser
USER springuser:springuser

# Copy the built JAR from the builder stage
COPY --from=builder /app/target/*.jar app.jar

# Expose the port the application runs on
EXPOSE 8080

# Health check (Actuator endpoint)
HEALTHCHECK --interval=30s --timeout=10s --retries=5 CMD curl --fail http://localhost:8080/actuator/health || exit 1

# Command to run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Environment Variables

All sensitive configuration parameters for production **must** be supplied via environment variables.

*   `DB_URL`: PostgreSQL JDBC connection URL (e.g., `jdbc:postgresql://postgres-host:5432/parceldb`)
*   `DB_USERNAME`: Database username
*   `DB_PASSWORD`: Database password
*   `REDIS_HOST`: Redis host for distributed caching (if enabled)
*   `REDIS_PORT`: Redis port
*   `SPRING_PROFILES_ACTIVE`: `prod` (or other active profile)

---