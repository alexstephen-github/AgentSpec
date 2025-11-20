```markdown
# 🐍 Python Django REST Framework Agent Protocol: Building Enterprise-Grade Python Microservices

This document provides a comprehensive guide for AI coding agents to build secure, scalable, and maintainable microservices using Python 3.11+, Django 5.x, and Django REST Framework (DRF). Adherence to these principles is mandatory for all generated projects.

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
| **Language** | Python | **3.11+** |
| **Framework** | Django | **5.x** |
| **API Framework** | Django REST Framework | **3.15+** |
| **Build Tool/Package Manager** | pip, venv | Latest |
| **Database** | PostgreSQL (Prod), SQLite (Dev/Test) | Latest |
| **API Spec** | OpenAPI | 3.0 (via `drf-spectacular`) |
| **Container** | Docker | Latest |
| **WSGI Server** | Gunicorn | Latest |
| **Caching** | Redis (distributed) | Latest (`django-redis`) |
| **Validation** | DRF Serializer Validation, Django Model Validation | Built-in |
| **Testing** | pytest, pytest-django, factory_boy | Latest |
| **Linting/Formatting** | Black, Flake8/Ruff | Latest |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using `django-admin startproject` and `django-admin startapp` with the following default dependencies (to be added in `requirements.txt`):

-   djangorestframework
-   drf-spectacular
-   psycopg2-binary
-   django-environ
-   gunicorn
-   django-redis
-   pytest
-   pytest-django
-   factory_boy
-   black
-   flake8 (or ruff)

### Standard Package Structure

Organize all source code under a root project (e.g., `core_project`) and a primary application (e.g., `parcel_service`).

```
core_project/
├── manage.py                   # Django's command-line utility
├── core_project/               # Project-level settings and URLs
│   ├── __init__.py
│   ├── settings/               # Environment-specific settings
│   │   ├── __init__.py         # Imports base settings and potentially env-specific ones
│   │   ├── base.py             # Base settings for all environments
│   │   ├── dev.py              # Development-specific settings (e.g., SQLite)
│   │   └── prod.py             # Production-specific settings (e.g., PostgreSQL, Redis)
│   ├── urls.py                 # Project-wide URL definitions
│   ├── wsgi.py                 # WSGI configuration for production deployment
│   └── asgi.py                 # ASGI configuration (if using async)
├── parcel_service/             # Main application
│   ├── __init__.py
│   ├── admin.py                # Django Admin configurations
│   ├── apps.py                 # Application configuration
│   ├── urls.py                 # App-specific URL definitions
│   ├── views.py                # DRF ViewSets/APIViews (API layer)
│   ├── serializers.py          # Data Transfer Objects (DTOs) for API requests/responses
│   ├── models.py               # Django ORM Models (Entities)
│   ├── services.py             # Business logic (service layer)
│   ├── exceptions.py           # Custom exceptions and global handler utils
│   ├── permissions.py          # Custom DRF permissions
│   ├── authentication.py       # Custom DRF authentication backends
│   └── utils/                  # General utility functions
│       └── __init__.py
└── requirements/               # Dependency management
    ├── base.in                 # Base dependencies for pip-compile
    ├── base.txt                # Pinned base dependencies
    ├── dev.in                  # Dev-specific dependencies
    ├── dev.txt                 # Pinned dev dependencies
    └── prod.txt                # Pinned prod dependencies
```
*(Optionally, `repositories.py` can be added to the `parcel_service` if a stricter repository pattern separate from Django's default `Manager` is desired, mirroring Java's `JpaRepository`)*

---

## 4. 📝 Development Workflow & Best Practices

### Step 1: API & DTO Design (View/Serializer Layer)

1.  **Define Serializers**: Create separate `serializers.py` classes (DRF serializers) for API requests and responses in the `parcel_service/serializers.py` file. Use Django's built-in model validation and DRF's serializer field validation (`validators`, `min_length`, `EmailField`, etc.) for all incoming data. **Never expose Django ORM model instances directly in the API; always use serializers.**
2.  **Create Views**: In `parcel_service/views.py`, create DRF `APIView`, `GenericAPIView`, `ModelViewSet`, or `ViewSet` classes.
3.  **Map Endpoints**: In `parcel_service/urls.py`, define URL patterns using `path()` or `re_path()`. For `ViewSet`s, use `DefaultRouter`.
4.  **API Versioning**: Prefix all routes with `/api/v1`. Example: `path('api/v1/parcels/', ...)` in `urls.py`.
5.  **Input/Output**: Accept `request.data` (which is parsed by serializers) and return `Response` objects containing serialized data.

### Step 2: Business Logic (Service Layer)

1.  **Create Service**: In `parcel_service/services.py`, create a Python class (e.g., `ParcelService`). This class should encapsulate the core business logic.
2.  **Inject Dependencies**: Use constructor injection for dependencies (e.g., pass repository interfaces or other service instances). For simple cases, direct calls are acceptable, but for complex interactions, explicit dependency injection is preferred.
3.  **Implement Logic**: Write the core business logic here. This layer is responsible for coordinating data access, calling other services, and enforcing business rules.
4.  **Use Transactions**: Use `django.db.transaction.atomic()` for methods that modify data to ensure atomicity. Use `transaction.atomic(savepoint=False)` for read operations that require strong consistency across multiple queries, though generally, read operations don't require full transactions.
5.  **Map Serializers to Models**: The service layer is responsible for converting validated serializer data to Django ORM models before saving, and converting models to serializers for output.

### Step 3: Data Persistence (Model Layer)

1.  **Create Model**: In `parcel_service/models.py`, define Django ORM models. Use `models.ForeignKey`, `models.DateTimeField`, `models.CharField`, etc., and ensure appropriate indexing.
2.  **Repository Pattern (Optional but Recommended for Complexity)**: While Django's `Model.objects` manager serves as a default repository, for complex projects, consider creating an explicit `repositories.py` module.
    *   Define classes (e.g., `ParcelRepository`) that abstract data access logic, making them injectable into services.
    *   This provides a clearer separation of concerns and easier testing by allowing repositories to be mocked.

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in `parcel_service/exceptions.py` (e.g., `ResourceNotFoundException(APIException)`).
2.  **Global Handler**: Configure a custom exception handler in `settings.py` (e.g., `REST_FRAMEWORK = {'EXCEPTION_HANDLER': 'parcel_service.exceptions.custom_exception_handler'}`).
3.  **Handle Exceptions**: In `parcel_service/exceptions.py`, implement `custom_exception_handler` that catches DRF's `APIException`, custom exceptions, and a fallback `Exception`. Return a standardized `ErrorResponse` (a dictionary) with a clear message and HTTP status. **Never expose stack traces in production.**

### Step 5: Caching

1.  **Enable Caching**: Configure `CACHES` in `settings/base.py` (or `prod.py` for Redis).
2.  **Configure Cache**: Use `django-redis` for distributed caching.
    ```python
    # core_project/settings/prod.py
    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": env("REDIS_URL", default="redis://127.0.0.1:6379/1"),
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient",
                "COMPRESSOR": "django_redis.compressors.zlib.ZlibCompressor",
            }
        }
    }
    ```
3.  **Apply Caching**: Use `@cache_page` decorator on view methods or `cache.get()`, `cache.set()` directly in services for granular control over expensive operations.

### Step 6: Configuration

-   **Base Config (`core_project/settings/base.py`)**: Define common properties like `SECRET_KEY`, `INSTALLED_APPS`, `MIDDLEWARE`. Use `django-environ` for environment variables.
-   **Dev Profile (`core_project/settings/dev.py`)**: Configure SQLite database, enable debug mode (`DEBUG = True`), and set logging to `DEBUG`.
-   **Prod Profile (`core_project/settings/prod.py`)**: Configure PostgreSQL connection details using environment variables (e.g., `env('DATABASE_URL')`). Disable debug mode (`DEBUG = False`). Configure robust logging, disallow `ALLOWED_HOSTS = ['*']`, and ensure `SECURE_SSL_REDIRECT`, `CSRF_COOKIE_SECURE`, `SESSION_COOKIE_SECURE` are `True`.

### Step 7: Documentation

-   **OpenAPI**: Configure `drf-spectacular` in `settings/base.py` to set the API title, version, and description. Generate schema at a designated endpoint (e.g., `/api/schema/`).
    ```python
    # core_project/settings/base.py
    SPECTACULAR_SETTINGS = {
        'TITLE': 'Parcel Service API',
        'DESCRIPTION': 'API for managing parcels',
        'VERSION': '1.0.0',
        'SERVE_INCLUDE_SCHEMA': False, # Serve UI only, schema endpoint separate
        'SWAGGER_UI_SETTINGS': {
            'deepLinking': True,
            'defaultModelsExpandDepth': -1,
            'defaultModelRendering': 'model',
            'displayRequestDuration': True,
            'filter': True,
            'showExtensions': True,
            'showCommonExtensions': True,
            'tryItOutEnabled': True,
        },
    }
    # core_project/urls.py
    from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView
    urlpatterns = [
        # ...
        path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
        path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    ]
    ```
-   **Docstrings**: Add Python Docstrings (PEP 257 compliant) to all public methods in views and services, explaining their purpose, parameters, and return values.

### Step 8: Testing

1.  **Unit Tests**: Test services and utilities by mocking their dependencies (e.g., Django ORM calls or external API clients). Place tests in `parcel_service/tests/unit/`. Use `unittest.mock` or `pytest-mock`.
2.  **Integration Tests**: Use `pytest-django` and `rest_framework.test.APITestCase` to test the full application flow, from view to database. Use `factory_boy` for creating test data fixtures. Place tests in `parcel_service/tests/integration/`. Use `testcontainers-python` for database integration tests (PostgreSQL).
3.  **Coverage**: Aim for a minimum of 80% test coverage using `pytest-cov` (or `coverage.py`).

---

## 5. 🐳 Containerization & Deployment

### Dockerfile

Create a multi-stage `Dockerfile`:

1.  **Build Stage**: Use a Python base image to install dependencies and collect static files.
2.  **Runtime Stage**: Use a minimal Python base image (e.g., `python:3.11-slim-buster`).
3.  **Security**: Create and run as a non-root user. Do not expose `SECRET_KEY` in the image.
4.  **Health Check**: Add a `HEALTHCHECK` instruction that calls a `/api/v1/health` endpoint or `/admin/healthcheck/`.
5.  **Execution**: Use `ENTRYPOINT` to run Gunicorn. Example: `ENTRYPOINT ["gunicorn", "core_project.wsgi:application", "--bind", "0.0.0.0:8000"]`.

### Environment Variables

All secrets (database passwords, API keys, `SECRET_KEY`) **must** be supplied via environment variables, not hardcoded in settings files. Use `django-environ` for easy access.

---

This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.

---

**Additional Guidance and Links:**

*   **Django Project Structure Best Practices:** [Link to Confluence/Team/GitHub Wiki on Django Structure]
*   **DRF Security Best Practices:** [Link to Confluence/Team/GitHub Wiki on DRF Security]
*   **Logging Standards:** [Link to Confluence/Team/GitHub Wiki on Logging Standards]
*   **API Design Guidelines:** [Link to Confluence/Team/GitHub Wiki on API Design]
*   **CI/CD Pipeline Integration:** [Link to Confluence/Team/GitHub for CI/CD Pipeline Setup]
*   **Code Review Checklist:** [Link to Confluence/Team/GitHub for Code Review Checklist]
```