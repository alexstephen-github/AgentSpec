agent: agent
model: Claude Sonnet 4.5 (copilot)
name: django.rest.api
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
| **Language** | Python | **3.10+** |
| **Framework** | Django, Django REST Framework | **5.0+ (Django), 3.15+ (DRF)** |
| **Build Tool** | Poetry / Pip | Latest |
| **Database** | PostgreSQL (Prod), SQLite (Dev/Test) | Latest |
| **API Spec** | OpenAPI (DRF Spectacular) | 3.0 |
| **Container** | Docker | Latest |
| **Caching** | Django's Cache API (local), Redis (distributed) | Latest |
| **Validation** | DRF Serializers, Django Forms | Latest |
| **Testing** | Pytest, Django's TestCase, DRF APITestCase | Latest |
 
---
 
## 3. 🏗️ Project Initialization & Structure
 
### Initialization

Generate new projects using **Django-admin** and **startapp** with the following default dependencies:

-   Django
-   Django REST Framework (`djangorestframework`)
-   DRF Spectacular (`drf-spectacular`)
-   Psycopg2 (PostgreSQL driver)
-   Django Filter (`django-filter`)
-   Celery (for async tasks, if needed)
 
### Standard Package Structure

Organize all source code under a root project (e.g., `myproject_name`) and logical applications (e.g., `users`, `products`).
 
```
myproject_name/
├── manage.py                       # Django's command-line utility
├── myproject_name/                 # Project root package
│   ├── asgi.py
│   ├── settings/                   # Configuration files
│   │   ├── __init__.py
│   │   ├── base.py                 # Base settings (common to all environments)
│   │   ├── dev.py                  # Development-specific settings
│   │   └── prod.py                 # Production-specific settings
│   ├── urls.py                     # Root URL routing
│   └── wsgi.py
├── apps/                           # Directory for Django applications
│   └── service_name/               # Example Django application
│       ├── __init__.py
│       ├── admin.py                # Django Admin configurations
│       ├── models.py               # Django Models (ORM entities)
│       ├── migrations/             # Database migrations
│       ├── urls.py                 # Application-specific URL routing
│       ├── views.py                # DRF ViewSets / APIViews (API layer)
│       ├── serializers.py          # DRF Serializers (DTOs for API requests/responses)
│       ├── services.py             # Business logic (optional, for complex logic)
│       ├── exceptions.py           # Custom exceptions and handlers
│       └── tests/                  # Application tests
│           ├── __init__.py
│           ├── test_models.py
│           ├── test_serializers.py
│           └── test_views.py
└── requirements.txt / pyproject.toml # Project dependencies
```
 
---
 
## 4. 📝 Development Workflow & Best Practices
 
### Step 1: API & Serializer Design (View Layer)

1.  **Define Serializers**: Create separate `serializers.py` files within each app for DRF `Serializer` or `ModelSerializer`. Use DRF validation fields and methods (`required`, `min_length`, `validators`, `validate_field`). **Never expose Django models directly in the API responses; always use serializers.**

2.  **Create Views**: In the `views.py` file of your application, create DRF `APIView`, `GenericAPIView`, `ViewSet`, or `ModelViewSet` classes.

3.  **Map Endpoints**: In `app_name/urls.py`, define URL patterns and include them in the project's root `urls.py`. Use DRF's `DefaultRouter` for ViewSets.

4.  **API Versioning**: Prefix all routes with `/api/v1/`. Example: `path('api/v1/parcels/', ...)` or `/api/v1/` in `urls.py`.

5.  **Input/Output**: Accept data via serializers (e.g., `serializer = YourSerializer(data=request.data)`). Return DRF `Response` objects containing serialized data.
 
### Step 2: Business Logic (Service Layer Concept)

1.  **Implement Logic**: Business logic primarily resides in:
    *   **Model Managers**: Custom managers attached to models (`MyModel.objects.custom_method()`).
    *   **Model Methods**: Methods directly on Django model instances.
    *   **Service Modules**: For more complex or cross-cutting concerns, create a `services.py` file within an app to encapsulate business logic that might involve multiple models or external interactions.
2.  This layer is responsible for coordinating data access, calling external services, and enforcing business rules.
3.  **Use Transactions**: Use Django's `transaction.atomic()` decorator or context manager for methods that modify data to ensure atomicity. Use `transaction.atomic()` for read operations only if specific isolation levels are required; otherwise, reads are typically outside explicit transactions.
4.  **Map Serializers to Models**: The `serializer.save()` method in DRF handles the conversion of validated serializer data to Django model instances and their persistence.
 
### Step 3: Data Persistence (Model Layer)

1.  **Create Model**: In the `models.py` file, define Django `Model` classes. Use `models.Field` types (e.g., `CharField`, `IntegerField`, `DateTimeField`, `ForeignKey`). Define `__str__` methods for better representation.
2.  **Migrations**: Generate and apply database migrations using `python manage.py makemigrations` and `python manage.py migrate`.
3.  **No Explicit Repository Layer**: Django's ORM (Object-Relational Mapper) serves as the data access layer, abstracting repository patterns. Custom query logic can be added to Model Managers.
 
### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific Python exceptions in the `exceptions.py` file of your application (e.g., `ResourceNotFoundException(Exception)`).

2.  **Global Handler**: Configure DRF's `exception_handler` in `settings.py` or implement a custom middleware.

3.  **Handle Exceptions**: In your custom `exception_handler` or views, catch specific exceptions (e.g., `Http404`, `ValidationError`, custom exceptions). Return a standardized DRF `Response` with a clear message and appropriate HTTP status code (e.g., `status.HTTP_404_NOT_FOUND`, `status.HTTP_400_BAD_REQUEST`). **Never expose raw stack traces in production.**
 
### Step 5: Caching

1.  **Enable Caching**: Configure cache settings in `settings.py` using Django's `CACHES` dictionary.

2.  **Configure Cache**: Define cache backends (e.g., `django.core.cache.backends.locmem.LocMemCache` for local, `django_redis` for Redis).

3.  **Apply Caching**: Use Django's cache API (`cache.get()`, `cache.set()`), view-level decorators (`@cache_page`), or per-site caching. For specific query results or expensive computations, use `cache.memoize` (if available via external libraries or custom implementation).
 
### Step 6: Configuration

-   **Base Config (`settings/base.py`)**: Define common properties like `INSTALLED_APPS`, `MIDDLEWARE`, `DATABASES` (defaulting to SQLite).

-   **Dev Profile (`settings/dev.py`)**: Set `DEBUG = True`, configure SQLite database, enable Django Debug Toolbar, and set logging to `DEBUG`.

-   **Prod Profile (`settings/prod.py`)**: Set `DEBUG = False`, configure PostgreSQL connection details (using environment variables), set `ALLOWED_HOSTS`, configure logging for production, and disable detailed error messages (`DEFAULT_EXCEPTION_RENDERER`). Use environment variables for all secrets (e.g., `os.environ.get('DB_PASSWORD')`).

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
  type: service # required, e.g., 'service', 'library'
  lifecycle: production # required, e.g., 'experimental', 'production'
  owner: # required, the owning team or user
  system: # optional, the larger system this component belongs to
  dependsOn:
    - # list of components/resources this component depends on
  providesApis:
    - # list of APIs this component provides
```
 
---
 ### Step 8: #Coding standards#
 
**Django/Django REST Framework Best Practices**
 
**1. Use DRF ViewSets/APIViews for RESTful Services:**
  - Leverage `rest_framework.viewsets.ModelViewSet` for standard CRUD operations on a single model. Use `rest_framework.views.APIView` or `rest_framework.generics.*APIView` for more custom or less resource-centric endpoints.
 
**2. Use `services.py` for Business Logic (Optional but Recommended):**
  - For complex logic that spans multiple models or doesn't directly fit in a view or model, create a `services.py` module within your app. This helps keep views lean and focused on HTTP request/response handling.
 
**3. Use Django Models and ORM for Data Access:**
  - Define your database schema using Django's `models.Model`. Leverage the Django ORM for all database interactions. Use custom Model Managers for reusable query logic.
 
**4. Prefer Dependency Injection (where applicable) or Clear Imports:**
  - While Python doesn't have native DI frameworks like Spring, structure your code with clear function/class arguments or direct imports (`from .services import my_service_function`) to indicate dependencies.
 
**5. Avoid Direct Model Exposure in API:**
  - Never return Django model instances directly from your REST APIs. Always use DRF Serializers to define the exact data structure for requests and responses, decoupling the API from your database schema.
 
**6. Use DRF Serializers for Input Validation and Data Transformation:**
  - Apply `rest_framework.serializers.Serializer` or `ModelSerializer` for all incoming data validation and outgoing data serialization. Use field-level and object-level `validate_` methods for complex validation rules.
 
**7. Implement Global Exception Handling:**
  - Configure a custom `EXCEPTION_HANDLER` in `settings.py` or use Django middleware to handle exceptions consistently across your application, returning standardized JSON error responses.
 
**8. Log Effectively:**
  - Use Python's built-in `logging` module. Configure logging levels and handlers in `settings.py`. Avoid logging sensitive information and stack traces in production error responses.
 
**9. Use Django Settings for Configuration:**
  - Externalize configuration using `settings.py`. Use environment variables (`os.environ`) for sensitive data and environment-specific settings (e.g., `settings/dev.py`, `settings/prod.py`).
 
**10. Secure Your Endpoints:**
  - Implement DRF permissions (`rest_framework.permissions`) and authentication classes (`rest_framework.authentication`) for access control. Use Django's built-in security features, HTTPS, and secure headers.
 
**11. Implement Caching where Appropriate:**
  - Use Django's caching framework (`django.core.cache`). Apply `@cache_page` for entire view responses or use `cache.get`/`cache.set` for specific data or expensive computations.
 
**12. Database Transactions:**
  - Use `django.db.transaction.atomic()` decorator or context manager for methods that perform database write operations to ensure data integrity.
 
**13. Use Django ORM for Repositories:**
  - Leverage Django's powerful ORM for all data access. Utilize custom Model Managers for encapsulating reusable query logic.
 
**14. Write Comprehensive Tests:**
  - Include unit tests for models, serializers, and services. Use `rest_framework.test.APITestCase` for integration tests of views and endpoints. Use Pytest for a more flexible testing framework.
 
**15. Document Your API with OpenAPI/Swagger:**
  - Integrate `drf-spectacular` to automatically generate OpenAPI documentation. Configure it in `settings.py` and `urls.py`.
 
**16. Use Django Debug Toolbar for Monitoring (Dev Only):**
  - Enable `django-debug-toolbar` in development to inspect database queries, cache hits, and request/response cycles. For production, use monitoring tools like Prometheus/Grafana or custom health checks.
 
**17. Avoid Anemic Domain Models:**
  - Add behavior and business logic directly to your Django models where it makes sense, following Domain-Driven Design principles. Models should not just be data containers.
 
**18. Proper Error Response Structure:**
  - Return consistent and informative JSON error responses (e.g., `{"detail": "Error message", "code": "error_code"}`). Avoid exposing internal server details or stack traces.
 
**19. Use a Multi-App Project for Larger Applications:**
  - For complex applications, structure your project into multiple Django applications (e.g., `users`, `products`, `orders`) to promote modularity, clear separation of concerns, and reusability.
 
**20. Utilize Django Signals for Decoupled Logic:**
  - Use Django Signals to allow decoupled applications to be notified when certain actions occur (e.g., a user is created, an order is processed) without tight coupling.
 
### Step 9: Documentation

-   **OpenAPI**: Configure `drf-spectacular` in `settings.py` and include its URLs in `urls.py` to generate API documentation (Swagger UI/Redoc). Add `extend_schema` decorators to views for more detailed schema customization.

-   **Docstrings**: Add Python docstrings to all public classes, methods, and functions in views, serializers, models, and services, explaining their purpose, parameters, and return values.
 
### Step 10: Testing

-   **Unit Tests**: Test models, serializers, and individual functions/methods by mocking dependencies. Place tests in `src/app_name/tests/`. Use Pytest for execution.

-   **Integration Tests**: Use `rest_framework.test.APITestCase` to test the full API flow, from views to serializers and models, ensuring correct HTTP responses and data integrity. Use `pytest-django` for seamless integration with Django.

-   **Coverage**: Aim for a minimum of 80% test coverage using `pytest-cov`.
 
---
 
## 5. 🐳 Containerization & Deployment
 
### Dockerfile

Create a multi-stage `Dockerfile`:

1.  **Build Stage**: Use a Python base image (e.g., `python:3.10-slim-buster`) to install dependencies (Poetry/Pip).

2.  **Runtime Stage**: Use a minimal Python JRE image (e.g., `python:3.10-slim-buster`).

3.  **Security**: Create and run as a non-root user (e.g., `USER django`).

4.  **Health Check**: Add a `HEALTHCHECK` instruction that calls a custom `/health` endpoint or uses `django-health-check`.

5.  **Execution**: Use Gunicorn or Uvicorn to serve the Django application. Example: `ENTRYPOINT ["gunicorn", "--bind", "0.0.0.0:8000", "myproject_name.wsgi:application"]`.
 
### Environment Variables

All secrets (database passwords, API keys, `SECRET_KEY`) **must** be supplied via environment variables, not hardcoded in `settings.py` files.
 
---
 
This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.
 
Sample Agent.md
 
Platform engineer agent can create this..  
 
Here the details and the best practices we need to give link to appropriate confluence, team or github link so that it will be more realistic 
 
then the consumer while building a microservice, he can download this agent.md file from IDP and use his specific development

---

### GitHub Copilot Prompt File Template

```
/generate-django-rest-api
# This slash command template helps generate a Django REST Framework API based on the defined spec.
# 
# Usage:
#   /generate-django-rest-api <description_of_api>
#   e.g., /generate-django-rest-api "Create a user management service with CRUD operations for User model."
#
# Provide details about the API you want to build, and Copilot will assist you by referencing
# the "django.rest.api.md" specification file.

# You can also specify models or specific requirements:
#   /generate-django-rest-api "Implement a product catalog API with Product model, supporting search and filtering."
#   /generate-django-rest-api "Build an order processing service, including Order and OrderItem models, with transactional behavior."
#
# The agent will ensure the generated code adheres to the defined:
# - Technology Stack (Python 3.10+, Django 5.0+, DRF 3.15+, PostgreSQL, OpenAPI, Docker, etc.)
# - Project Structure (project/, apps/app_name/, models.py, serializers.py, views.py, urls.py)
# - Development Workflow (Serializers, Views, Models, Exception Handling, Caching)
# - Coding Standards (DRF Views/Serializers, Model Managers, Service modules, Validation, Global Exception Handling, Logging, Transactions)
# - Testing Best Practices (Unit, Integration, Coverage)
# - Containerization (Dockerfile structure)

# Describe your desired Django REST API:
<YOUR_API_DESCRIPTION_HERE>
```