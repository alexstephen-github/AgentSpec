Here is a detailed spec file for building Django REST Framework microservices using Python, referencing `Agent-template.md` for overall structure and principles, and `python-rules.txt` for coding standards.

---

# 🤖 Agent Protocol: Building Enterprise-Grade Python Microservices with Django REST Framework

This document provides a comprehensive guide for AI coding agents to build secure, scalable, and maintainable microservices using Python and Django REST Framework. Adherence to these principles is mandatory for all generated projects.

---

## 1. 🎯 Core Principles

-   **Security First**: Implement security controls at every layer. Assume a zero-trust environment. Sanitize all inputs and protect sensitive data.
-   **Production Ready**: All code must be suitable for production, with robust configuration, logging, and error handling.
-   **Developer Experience**: Ensure the project is easy to set up, run, and understand. Prioritize clear documentation and clean code.
-   **Performance by Design**: Build for low latency and high throughput. Implement caching and efficient data access patterns using the Django ORM effectively.
-   **Observability**: The system's internal state must be inferable from its external outputs (logs, metrics, traces).

---

## 2. 🛠️ Standard Technology Stack

| Component | Technology | Version/Standard |
| :-------- | :--------- | :--------------- |
| **Language** | Python | **3.10+** |
| **Framework** | Django | 4.2+ |
| **API Framework** | Django REST Framework | 3.14+ |
| **Dependency Mgmt.** | pip (with `requirements.txt`) | Latest |
| **Database** | PostgreSQL (Prod), SQLite (Dev/Test) | Latest |
| **API Spec** | OpenAPI | 3.0 (via `drf-spectacular`) |
| **Container** | Docker | Latest |
| **Caching** | Redis | Latest |
| **Validation** | DRF Serializers, Django Forms | Latest |
| **Testing** | pytest, pytest-django, requests | Latest |
| **ASGI Server** | Gunicorn/Uvicorn | Latest |
| **Linter** | Ruff/Flake8 | Latest |
| **Formatter** | Black | Latest |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using `django-admin startproject` and `python manage.py startapp`.

### Standard Project Structure

Organize all source code under a root project (e.g., `core_project`). Each logical service or module should be a separate Django app.

```
core_project/
├── manage.py                   # Django's command-line utility
├── core_project/               # Main project configuration
│   ├── __init__.py
│   ├── settings/               # Split settings for environments
│   │   ├── __init__.py
│   │   ├── base.py             # Base settings
│   │   ├── dev.py              # Development settings (e.g., SQLite)
│   │   └── prod.py             # Production settings (e.g., PostgreSQL)
│   ├── urls.py                 # Main URL routing
│   └── wsgi.py / asgi.py       # WSGI/ASGI entry point
├── user_service/               # Example Django App (e.g., for user management)
│   ├── migrations/             # Database migrations
│   │   └── __init__.py
│   ├── __init__.py
│   ├── admin.py                # Django Admin configuration
│   ├── models.py               # Django ORM Models
│   ├── serializers.py          # DRF Serializers (DTOs)
│   ├── views.py                # DRF Views/ViewSets (Controller layer)
│   ├── services.py             # Business logic (Service layer)
│   ├── urls.py                 # App-specific URL routing
│   ├── permissions.py          # Custom DRF Permissions
│   ├── exceptions.py           # Custom exceptions
│   └── tests/                  # Unit and integration tests
│       ├── __init__.py
│       ├── test_models.py
│       ├── test_serializers.py
│       ├── test_views.py
│       └── test_services.py
├── config/                     # Configuration files (e.g., Celery, OpenAPI schema generation)
├── scripts/                    # Helper scripts (e.g., local setup)
├── .env                        # Environment variables (for local dev)
├── .dockerignore
├── Dockerfile
├── requirements.txt            # Python dependencies
└── catalog-info.yaml           # Backstage component descriptor
```

---

## 4. 📝 Development Workflow & Best Practices

### Step 1: API & DTO Design (Serializer/View Layer)

1.  **Define Serializers**: Create separate `rest_framework.serializers.Serializer` or `ModelSerializer` classes in the `serializers.py` file of your app. These act as Data Transfer Objects (DTOs) for API requests and responses. Use DRF's built-in validation or custom validation methods. **Never expose ORM entities directly in the API responses without proper serialization.**
2.  **Create Views/ViewSets**: In the `views.py` file, create `APIView` classes for custom logic or `ModelViewSet` for standard CRUD operations.
3.  **Map Endpoints**: Use Django's `path` or `re_path` in `urls.py`. For `ModelViewSet`s, use `DefaultRouter` or `SimpleRouter` from DRF for automatic URL generation.
4.  **API Versioning**: Prefix all routes with `/api/v1`. Example: `path('api/v1/users/', UserViewSet.as_view({'get': 'list', 'post': 'create'}))`.
5.  **Input/Output**: Accept serialized data in request bodies and return `Response` objects containing serialized data.

### Step 2: Business Logic (Service Layer)

1.  **Create Service Module**: In your app, create a `services.py` module or a dedicated `services` subdirectory. Implement plain Python functions or classes that encapsulate core business logic.
2.  **Inject Dependencies**: Dependencies (e.g., ORM models, other services) should be passed as arguments to functions or constructors. Avoid global state.
3.  **Implement Logic**: This layer is responsible for coordinating data access, calling other services, and enforcing business rules.
4.  **Use Transactions**: Use `django.db.transaction.atomic()` for methods that modify data to ensure atomicity.
5.  **Map DTOs to Entities**: The service layer should receive validated data (often from serializers) and convert it into Django ORM model instances before interacting with the database, and convert model instances back to data suitable for serialization before returning.

### Step 3: Data Persistence (Models & Managers)

1.  **Create Model**: In the `models.py` file, define Django ORM `models.Model` classes. Use appropriate field types (`CharField`, `IntegerField`, `ForeignKey`, etc.).
2.  **Define Managers**: Create custom `Manager` classes for complex queries or to encapsulate reusable query logic (e.g., `UserManager.active_users()`).
3.  **Run Migrations**: Use `python manage.py makemigrations` and `python manage.py migrate` to apply database schema changes.

### Step 4: Exception Handling

1.  **DRF Exceptions**: Utilize DRF's built-in `APIException` classes (`ValidationError`, `NotFound`, `PermissionDenied`, etc.) for common API errors.
2.  **Custom Exceptions**: Create specific, custom exceptions in an `exceptions.py` file (e.g., `UserAlreadyExistsException`, `InvalidInputException`). These should inherit from `Exception` or `APIException` as appropriate.
3.  **Global Handler**: Configure a custom exception handler in your `settings.py` by setting `REST_FRAMEWORK['EXCEPTION_HANDLER']`. This handler should map custom exceptions to appropriate HTTP status codes and return a standardized `ErrorResponse` format (e.g., `{"detail": "Error message"}`). **Never expose stack traces in production.**

### Step 5: Caching

1.  **Configure Cache Backend**: In `settings.py`, configure Django's caching framework. Use `django-redis` for production-grade Redis integration.
    ```python
    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": os.environ.get("REDIS_URL", "redis://127.0.0.1:6379/1"),
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient",
            }
        }
    }
    ```
2.  **Apply Caching**: Use `django.core.cache.cache` directly in services for granular control (`cache.get`, `cache.set`). For view-level caching, use the `@cache_page` decorator. For ORM query caching, consider libraries like `django-cacheops`.

### Step 6: Configuration

-   **Split Settings**: Organize `settings.py` into `base.py`, `dev.py`, and `prod.py` files under a `settings` subdirectory within your project root.
-   **Environment Variables**: All sensitive information (database credentials, API keys, `SECRET_KEY`) **must** be supplied via environment variables. Use `os.environ.get()` to retrieve them. For local development, use `python-dotenv` to load `.env` files.
-   **Base Config (`base.py`)**: Define common properties like `INSTALLED_APPS`, `MIDDLEWARE`, `REST_FRAMEWORK` defaults, and logging configuration.
-   **Dev Profile (`dev.py`)**: Configure SQLite, enable `DEBUG=True`, `ALLOWED_HOSTS = ['*']`, and potentially enable Django Debug Toolbar.
-   **Prod Profile (`prod.py`)**: Configure PostgreSQL connection details, `DEBUG=False`, `ALLOWED_HOSTS` with specific domains, and disable verbose error pages.

### Step 7: `catalog-info.yaml` File Structure

The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: # required, unique name (e.g., user-service-drf)
  description: # required, human-readable description
  tags:
    - python
    - django
    - drf
    - microservice
  annotations:
    # key-value pairs for tool integrations
    # e.g., 'backstage.io/kubernetes-id': 'user-service-drf'
  links:
    - url: https://github.com/your-org/user-service-drf # URL to the repository
      title: GitHub Repo
      icon: github
spec:
  type: service # required, e.g., 'service', 'library'
  lifecycle: production # required, e.g., 'experimental', 'production'
  owner: team-backend # required, the owning team or user
  system: core-platform # optional, the larger system this component belongs to
  dependsOn:
    - resource:database/postgresql-main # Example dependency
    - component:authentication-service # Example dependency
  providesApis:
    - user-api # Example API provided by this component
```

---

### Step 8: #Coding standards#

The following Python coding standards are mandatory. Use tools like `ruff` or `flake8` for linting and `black` for auto-formatting to ensure compliance.

1.  **PEP 8 Adherence**: All Python code must strictly follow PEP 8 style guide. Use linters like `flake8` or `ruff` to enforce this.
    *   Maximum line length: 120 characters (flexible for long URLs, specific data structures).
    *   Use 4 spaces for indentation.
    *   Imports should be grouped: standard library, third-party, local application. Each group separated by a blank line.
    *   Avoid wildcard imports (`from module import *`).
    *   Follow naming conventions: `lowercase_with_underscores` for functions, methods, variables; `CamelCase` for classes; `CONSTANT_CASE` for global constants.

2.  **Docstrings**: All modules, classes, methods, and public functions must have comprehensive docstrings following reStructuredText or Google style.
    *   Describe purpose, arguments, return values, and any raised exceptions.

3.  **Type Hinting**: Utilize type hints (`annotations`) for all function arguments, return values, and class attributes to improve code readability, maintainability, and enable static analysis.

4.  **Error Handling**:
    *   Use specific exception types rather than broad `except Exception:` catches.
    *   Log errors and exceptions appropriately with context.
    *   Raise custom exceptions for business-logic errors.

5.  **Logging**: Use Python's standard `logging` module.
    *   Configure log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL).
    *   Avoid printing to `stdout` for production logs.

6.  **Dependency Management**: Use `pip` with `requirements.txt` or `Poetry` for managing project dependencies. Pin exact versions of direct dependencies.

7.  **Testing**:
    *   Write unit tests for individual components (functions, classes).
    *   Write integration tests for interactions between components.
    *   Aim for high test coverage (minimum 80%).
    *   Use a testing framework like `pytest`.

8.  **Security**:
    *   Sanitize all user inputs to prevent injection attacks (SQL, XSS).
    *   Protect sensitive data (e.g., passwords, API keys) using environment variables, not hardcoding.
    *   Use secure defaults provided by frameworks (e.g., Django's CSRF protection).
    *   Regularly update dependencies to patch known vulnerabilities.

9.  **Performance**:
    *   Optimize database queries (e.g., `select_related`, `prefetch_related` in Django ORM).
    *   Implement caching strategies for frequently accessed data.
    *   Profile critical sections of code to identify bottlenecks.

10. **Concurrency/Asynchronicity**:
    *   Understand GIL implications for CPU-bound tasks.
    *   Use `asyncio` for I/O-bound tasks where appropriate (e.g., with `ASGI` servers like `Uvicorn`).

11. **Code Duplication**: Avoid "Don't Repeat Yourself" (DRY) principle. Refactor common logic into reusable functions, classes, or modules.

12. **Modularity and Abstraction**: Design code to be modular, with clear separation of concerns. Use abstraction to hide implementation details.

---

### Step 9: Documentation

-   **OpenAPI**: Integrate `drf-spectacular` to automatically generate OpenAPI 3.0 schemas from your DRF views and serializers. Configure it in `settings.py` and include its URLs in `urls.py`.
-   **Docstrings**: Add comprehensive docstrings (refer to #Coding standards#) to all modules, classes, methods, and public functions in controllers (views), serializers, and services, explaining their purpose, parameters, and return values.

### Step 10: Testing

-   **Unit Tests**: Test serializers, services, models, and custom managers in isolation. Mock dependencies (e.g., external API calls, database interactions where appropriate for unit tests). Place tests in `app_name/tests/`. Use `pytest`.
-   **Integration Tests**: Use `pytest-django` and DRF's `APIClient` to test the full application flow, from API endpoint to database. For complex scenarios, `requests` can be used to hit the actual server. Use Testcontainers for database integration tests if possible.
-   **Coverage**: Aim for a minimum of 80% test coverage using `pytest-cov`.

---

## 5. 🐳 Containerization & Deployment

### Dockerfile

Create a multi-stage `Dockerfile`:

1.  **Build Stage**: Use a Python base image (e.g., `python:3.10-slim-buster`) to install dependencies.
2.  **Runtime Stage**: Use a minimal Python runtime image (e.g., `python:3.10-slim-buster`).
3.  **Security**: Create and run as a non-root user.
4.  **Health Check**: Add a `HEALTHCHECK` instruction that calls a `/health` or `/api/v1/health` endpoint.
5.  **Execution**: Use `CMD ["gunicorn", "core_project.wsgi:application", "--bind", "0.0.0.0:8000"]` (or `uvicorn` for ASGI applications).

### Environment Variables

All secrets (database passwords, Django `SECRET_KEY`, API keys) **must** be supplied via environment variables at runtime, not hardcoded in settings files or committed to version control.

---

This protocol ensures consistency, security, and quality across all generated Python microservices using Django REST Framework. The agent must validate its output against these guidelines before completing a task.