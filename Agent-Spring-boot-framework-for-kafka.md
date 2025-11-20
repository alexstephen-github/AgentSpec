```markdown
# 🤖 Agent Protocol: Building Enterprise-Grade Java Microservices with Kafka

This document outlines the detailed specifications for AI coding agents to build secure, scalable, and maintainable Spring Boot 3.3+ microservices integrating with Apache Kafka. Adherence to these principles and specifications is mandatory for all generated projects.

---

## 1. 🎯 Core Principles

All projects must strictly adhere to the following core principles:

-   **Security First**: Implement security controls at every layer, including Kafka communication. Assume a zero-trust environment.
-   **Production Ready**: All code must be suitable for production, with robust configuration, logging, error handling (including Kafka-specific error strategies), and monitoring.
-   **Developer Experience**: Ensure the project is easy to set up, run, and understand. Prioritize clear documentation, clean code, and local Kafka setup for development.
-   **Performance by Design**: Build for low latency and high throughput. Optimize Kafka message processing and efficient data access patterns.
-   **Observability**: The system's internal state, including Kafka consumer lag, producer throughput, and message processing metrics, must be inferable from its external outputs (logs, metrics, traces).

---

## 2. 🛠️ Standard Technology Stack

| Component         | Technology                       | Version/Standard                                         |
| :---------------- | :------------------------------- | :------------------------------------------------------- |
| **Language**      | Java                             | **21**                                                   |
| **Framework**     | Spring Boot                      | **3.3+**                                                 |
| **Build Tool**    | Maven                            | 3.9+                                                     |
| **Database**      | PostgreSQL (Prod), H2 (Dev/Test) | Latest                                                   |
| **Messaging**     | Apache Kafka                     | Latest stable (via Spring for Apache Kafka)              |
| **API Spec**      | OpenAPI                          | 3.0                                                      |
| **Container**     | Docker                           | Latest                                                   |
| **Caching**       | Caffeine (local), Redis (distributed) | Latest                                                   |
| **Validation**    | Jakarta Bean Validation          | 3.0                                                      |
| **Testing**       | JUnit 5, Mockito, Testcontainers | Latest                                                   |
| **Serialization** | JSON (default), Avro/Protobuf    | JSON (with Jackson), Avro/Protobuf (with Schema Registry) |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using **Spring Initializr** with the following default dependencies:

-   Spring Web
-   Spring Data JPA
-   Spring Boot Actuator
-   Spring Cache
-   Jakarta Bean Validation
-   Lombok
-   PostgreSQL Driver
-   H2 Database
-   SpringDoc OpenAPI (`org.springdoc:springdoc-openapi-starter-webmvc-ui`)
-   **Spring for Apache Kafka** (`org.springframework.kafka:spring-kafka`)
-   **Testcontainers** (`org.testcontainers:testcontainers`, `org.testcontainers:kafka`, `org.testcontainers:postgresql`)

### Standard Package Structure

Organize all source code under a root package (e.g., `com.hcl.kafkaservice`).

```
com/hcl/kafkaservice/
├── KafkaServiceApplication.java   # Main application class
├── config/                        # @Configuration beans (Cache, Security, OpenAPI, Kafka Listeners/Producers)
├── controller/                    # REST controllers (API layer, e.g., for producing messages via HTTP)
├── dto/                           # Data Transfer Objects for API requests/responses and Kafka message payloads
├── model/                         # JPA Entities (if state needs to be persisted)
├── repository/                    # Spring Data JPA repositories
├── service/                       # Business logic (orchestrates Kafka operations and data persistence)
├── kafka/
│   ├── producer/                  # Kafka producer classes and configuration
│   ├── consumer/                  # Kafka consumer classes, listeners, and error handling
│   └── util/                      # Kafka-specific utilities (e.g., custom serializers/deserializers)
└── exception/                     # Custom exceptions and global handler
```

---

## 4. 📝 Development Workflow & Best Practices

### Step 1: API & DTO Design (Controller Layer - if applicable)

1.  **Define DTOs**: Create separate Plain Old Java Objects (POJOs) in the `dto` package for:
    *   API requests/responses (if exposing a REST endpoint to trigger Kafka events).
    *   Kafka message payloads (use dedicated DTOs for clarity).
    *   Use Jakarta Bean Validation annotations (`@NotBlank`, `@Size`, `@Email`, etc.) for all incoming data. **Never expose JPA entities directly in the API or as Kafka payloads.**
2.  **Create Controller**: In the `controller` package, create a `@RestController` if the service exposes REST endpoints (e.g., `/api/v1/messages` to produce a message).
3.  **Map Endpoints**: Use `@PostMapping` for message production.
4.  **API Versioning**: Prefix all routes with `/api/v1`. Example: `@RequestMapping("/api/v1/events")`.
5.  **Input/Output**: Accept DTOs as `@RequestBody` and return `ResponseEntity<Dto>` for REST APIs.

### Step 2: Business Logic (Service & Kafka Layers)

1.  **Create Service**: In the `service` package, create a `@Service` class to orchestrate business logic.
2.  **Inject Dependencies**: Use constructor injection for repositories, `KafkaProducerService`, `KafkaConsumerService`, and other services.
3.  **Implement Logic**:
    *   **For Producers**:
        *   Create a `KafkaProducerService` in `kafka/producer/`.
        *   Inject `KafkaTemplate<String, KafkaMessageDto>` (or `KafkaTemplate<KeyType, ValueType>`).
        *   Map incoming DTOs (from controller or internal events) to the `KafkaMessageDto` payload.
        *   Use `kafkaTemplate.send("topicName", messageKey, kafkaMessageDto)` for publishing.
        *   Handle send results using `.addCallback()` for success/failure logging.
    *   **For Consumers**:
        *   Create `KafkaConsumerService` or `@KafkaListener` annotated methods in `kafka/consumer/` package.
        *   Methods should consume `KafkaMessageDto` objects (or `ConsumerRecord<String, KafkaMessageDto>`).
        *   Implement business logic based on the consumed message. This layer is responsible for coordinating data access, calling other services, and enforcing business rules.
        *   Map `KafkaMessageDto` to JPA entities if persistence is required.
4.  **Use Transactions**:
    *   For producer-side transactional outbox patterns, use `@Transactional` on the service method that commits data to the database and then sends to Kafka.
    *   For consumer-side atomicity, consider `ChainedKafkaTransactionManager` or ensure idempotent processing of messages. `spring.kafka.listener.ack-mode` should be carefully selected (e.g., `RECORD` for manual ack, `BATCH` for batch ack).
5.  **Map DTOs to Entities**: The service layer is responsible for converting incoming DTOs to JPA entities before passing them to the repository, and converting entities to DTOs before returning them to the controller/other services. Similarly, map Kafka payload DTOs to/from internal entities.

### Step 3: Data Persistence (Repository & Model Layer - if applicable)

1.  **Create Entity**: In the `model` package, define a JPA `@Entity` if Kafka messages require state persistence. Use `@Id`, `@GeneratedValue`, and appropriate `@Column` annotations. Avoid Lombok's `@Data` on entities; prefer `@Getter`, `@Setter`, `@ToString`.
2.  **Create Repository**: In the `repository` package, create an interface that extends `JpaRepository<Entity, IdType>`. Add custom query methods as needed.

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in the `exception` package (e.g., `ResourceNotFoundException`, `KafkaProducerException`, `KafkaConsumerProcessingException`).
2.  **Global Handler**: Create a `@RestControllerAdvice` class in the `exception` package to handle REST API exceptions.
3.  **Handle Exceptions**: Add `@ExceptionHandler` methods for custom exceptions, validation exceptions (`MethodArgumentNotValidException`), and a fallback generic `Exception`. Return a standardized `ErrorResponse` DTO with a message and HTTP status. **Never expose stack traces.**
4.  **Kafka-specific Error Handling**:
    *   **Producers**: Implement `ProducerListener` (or use `ListenableFutureCallback`) to log production failures and potentially trigger retry mechanisms or Dead Letter Queue (DLQ) if necessary.
    *   **Consumers**: Configure `DefaultErrorHandler` for `@KafkaListener` containers. This should include:
        *   Retries with back-off policies.
        *   Publishing failed messages to a dedicated Dead Letter Topic (DLT) with original headers/metadata.
        *   Logging of errors with full message details (carefully redacting sensitive info).

### Step 5: Caching

1.  **Enable Caching**: Add `@EnableCaching` to the main application class.
2.  **Configure Cache**: Create a `CacheConfig` class in `config/` to define cache managers (e.g., Caffeine for local, Redis for distributed).
3.  **Apply Caching**: Use `@Cacheable`, `@CachePut`, and `@CacheEvict` on service methods to cache results of expensive operations (e.g., lookups for data required to process Kafka messages, or results of complex computations).

### Step 6: Configuration

-   **Base Config (`application.properties`)**: Define common properties like `spring.application.name`, `spring.kafka.bootstrap-servers` (with placeholder for prod).
-   **Dev Profile (`application-dev.properties`)**: Configure H2 in-memory database, enable the H2 console, set logging to `DEBUG`. For local Kafka, specify `spring.kafka.bootstrap-servers=localhost:9092` (assuming local Docker Kafka or `docker-compose`).
    *   Example Kafka Dev Config:
        ```properties
        spring.kafka.bootstrap-servers=localhost:9092
        spring.kafka.consumer.group-id=dev-group
        spring.kafka.producer.retries=0
        spring.kafka.listener.ack-mode=BATCH
        # Enable Testcontainers for local development if needed
        # spring.test.database.replace=none
        # spring.test.kafka.brokers=${spring.embedded.kafka.brokers:localhost:9092}
        ```
-   **Prod Profile (`application-prod.properties`)**: Configure PostgreSQL connection details. Use placeholders for secrets (e.g., `${DB_PASSWORD}`, `${KAFKA_BROKERS}`). Disable detailed health info (`management.endpoint.health.show-details=never`).
    *   Example Kafka Prod Config:
        ```properties
        spring.kafka.bootstrap-servers=${KAFKA_BOOTSTRAP_SERVERS}
        spring.kafka.consumer.group-id=${KAFKA_CONSUMER_GROUP_ID}
        spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
        spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
        spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
        spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
        spring.kafka.consumer.properties.spring.json.value.default.type=com.hcl.kafkaservice.dto.KafkaMessageDto
        spring.kafka.producer.retries=3
        spring.kafka.producer.acks=all
        spring.kafka.listener.ack-mode=RECORD # Or BATCH, based on atomicity requirements
        # If using Schema Registry
        # spring.kafka.properties.schema.registry.url=${SCHEMA_REGISTRY_URL}
        # spring.kafka.properties.specific.avro.reader=true
        ```
-   **Kafka Configuration**: Create a `KafkaConfig` class in `config/` to define:
    *   `ConcurrentKafkaListenerContainerFactory` beans.
    *   `KafkaTemplate` bean.
    *   Custom `DefaultErrorHandler` for consumer error handling (e.g., DLT configuration).
    *   `JsonMessageConverter` for flexible JSON serialization/deserialization.

### Step 7: `catalog-info.yaml` File Structure

The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: kafka-message-processor-service # required, e.g., 'order-event-processor'
  description: Spring Boot microservice for processing Kafka messages and interacting with a database. # required, human-readable description
  tags:
    - java
    - spring-boot
    - kafka
    - microservice
  annotations:
    # Example for integration with a Kafka monitoring tool or Confluence page for Kafka topics
    kafka.apache.org/topics: "order-events-topic, payment-processed-topic, dead-letter-topic"
    backstage.io/techdocs-ref: url:https://confluence.hcl.com/display/KAFKA_GUIDELINES # Link to Kafka best practices
  links:
    - url: https://github.com/hcl/kafka-message-processor-service # URL to the GitHub repository
      title: GitHub Repo
      icon: code
    - url: https://confluence.hcl.com/display/KAFKA_TOPIC_SCHEMAS # Link to Kafka topic schema documentation
      title: Kafka Topic Schemas
      icon: catalog
spec:
  type: service # required, e.g., 'service', 'library'
  lifecycle: production # required, e.g., 'experimental', 'production'
  owner: team-kafka # required, the owning team or user (e.g., 'team-payments', 'team-orders')
  system: order-processing-system # optional, the larger system this component belongs to
  dependsOn:
    - resource:database:postgresql-main # Depends on the main PostgreSQL instance
    - resource:kafka:main-cluster       # Depends on the main Kafka cluster
    - resource:kafka:schema-registry    # If using Schema Registry
  providesApis:
    - kafka-events-api # API provided through Kafka messages (e.g., OrderEvent API)
    - rest-trigger-api # REST API to trigger Kafka message production
```

---

### Step 8: Documentation

-   **OpenAPI**: Configure an `OpenApiConfig` bean to set the API title, version, and description for any REST endpoints.
-   **JavaDoc**: Add JavaDoc comments to all public methods in controllers, services, and Kafka-related components, explaining their purpose, parameters, and return values.
-   **Kafka Topic Documentation**: Crucial to document expected message schemas, topic names, and consumer group IDs. Reference external documentation (Confluence, Schema Registry) in `catalog-info.yaml`.
-   **`README.md`**: Provide clear instructions on how to set up and run the service locally, including Kafka setup (e.g., using Docker Compose).

### Step 9: Testing

-   **Unit Tests**: Test services, controllers, and Kafka producer/consumer logic by mocking their dependencies (e.g., `KafkaTemplate`, repositories). Place tests in `src/test/java`.
-   **Integration Tests**: Use `@SpringBootTest` and `@ActiveProfiles("test")` to test the full application flow, from controller to database and Kafka.
    *   **Kafka Integration Tests**: Use `Testcontainers` (specifically `KafkaContainer`) to spin up a real Kafka broker for end-to-end testing of producers and consumers. Ensure consumer listener containers are properly started and stopped for tests.
    *   Example:
        ```java
        @SpringBootTest
        @ActiveProfiles("test")
        @EmbeddedKafka(partitions = 1, brokerProperties = { "listeners=PLAINTEXT://localhost:9092", "port=9092" }) // For quick embedded Kafka, or use Testcontainers
        @Testcontainers // If using KafkaContainer
        class KafkaServiceIntegrationTest {
            // ... Testcontainers KafkaContainer setup ...
            // ... KafkaTemplate and KafkaListener tests ...
        }
        ```
-   **Coverage**: Aim for a minimum of 80% test coverage.

---

## 5. 🐳 Containerization & Deployment

### Dockerfile

Create a multi-stage `Dockerfile`:

1.  **Build Stage**: Use a `maven` base image to build the application JAR.
    ```dockerfile
    FROM maven:3.9.5-eclipse-temurin-21-alpine AS build
    WORKDIR /app
    COPY pom.xml .
    COPY src ./src
    RUN mvn clean package -DskipTests
    ```
2.  **Runtime Stage**: Use a minimal JRE image (e.g., `eclipse-temurin:21-jre-alpine`).
    ```dockerfile
    FROM eclipse-temurin:21-jre-alpine
    WORKDIR /app
    COPY --from=build /app/target/*.jar app.jar
    ```
3.  **Security**: Create and run as a non-root user.
    ```dockerfile
    RUN addgroup -S spring && adduser -S spring -G spring
    USER spring:spring
    ```
4.  **Health Check**: Add a `HEALTHCHECK` instruction that calls the `/actuator/health` endpoint.
    ```dockerfile
    HEALTHCHECK --interval=30s --timeout=10s --retries=5 CMD curl --fail http://localhost:8080/actuator/health || exit 1
    ```
5.  **Execution**: Use `ENTRYPOINT ["java", "-jar", "app.jar"]`.
    ```dockerfile
    ENTRYPOINT ["java", "-jar", "app.jar"]
    ```

### Environment Variables

All secrets (database passwords, API keys, Kafka broker addresses, SASL credentials, Schema Registry URLs) **must** be supplied via environment variables, not hardcoded in properties files.

Example environment variables:

-   `SPRING_DATASOURCE_URL`
-   `SPRING_DATASOURCE_USERNAME`
-   `SPRING_DATASOURCE_PASSWORD`
-   `KAFKA_BOOTSTRAP_SERVERS`
-   `KAFKA_SECURITY_PROTOCOL` (e.g., `SASL_SSL`)
-   `KAFKA_SASL_MECHANISM` (e.g., `PLAIN`)
-   `KAFKA_SASL_JAAS_CONFIG`
-   `KAFKA_CONSUMER_GROUP_ID`
-   `SCHEMA_REGISTRY_URL` (if using Avro/Protobuf)

---

This protocol ensures consistency, security, and quality across all generated Kafka-enabled microservices. The agent must validate its output against these guidelines before completing a task.
```