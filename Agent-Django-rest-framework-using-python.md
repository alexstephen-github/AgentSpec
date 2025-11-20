# 🤖 Agent Protocol: Building Enterprise-Grade Django REST Framework Microservices

This document provides a comprehensive guide for AI coding agents to build secure, scalable, and maintainable microservices using Python 3.11+, Django 4.2+, and Django REST Framework 3.14+. Adherence to these principles is mandatory for all generated projects.

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
| **Language** | Python | **3.11+** |
| **Framework** | Django | **4.2+** |
| **API Framework** | Django REST Framework | **3.14+** |
| **Build/Dependency** | Poetry (preferred), pip | Latest |
| **Database** | PostgreSQL (Prod), SQLite (Dev/Test) | Latest |
| **API Spec** | OpenAPI | 3.0 (`drf-spectacular`) |
| **Container** | Docker | Latest |
| **Caching** | Django's `locmem` (local), Redis (distributed) | Latest |
| **Validation** | DRF Serializer Validation | Built-in |
| **Testing** | pytest, pytest-django, pytest-mock | Latest |
| **Environment Mgt.** | django-environ | Latest |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using `django-admin startproject` and then `django-admin startapp` for each logical module.

```bash
django-admin startproject my_project .
python manage.py startapp my_app
```

### Standard Package Structure

Organize all source code under a root project (e.g., `my_project`) and logical Django applications (e.g., `my_app`).

```
my_project/
├── manage.py
├── my_project/                 # Main project configuration
│   ├── asgi.py
│   ├── settings/               # Split settings for base, dev, prod environments
│   │   ├── __init__.py
│   │   ├── base.py             # Common settings
│   │   ├── dev.py              # Development-specific settings
│   │   └── prod.py             # Production-specific settings
│   ├── urls.py                 # Main project URL routing
│   └── wsgi.py
├── my_app/                     # Example Django application (e.g., 'parcels')
│   ├── __init__.py
│   ├── admin.py                # Django Admin configurations
│   ├── apps.py                 # Application configuration
│   ├── models.py               # Django Models (ORM definitions)
│   ├── services.py             # Business logic (analogous to Spring's Service layer)
│   ├── serializers.py          # DRF Serializers (analogous to DTOs)
│   ├── views.py                # DRF Views (analogous to Spring's Controller layer)
│   ├── urls.py                 # App-specific URL routing
│   ├── permissions.py          # DRF Custom permissions
│   ├── exceptions.py           # Custom exceptions for the app
│   └── tests/                  # Test files (unit, integration)
│       ├── __init__.py
│       ├── test_models.py
│       ├── test_services.py
│       └── test_views.py
├── requirements.txt / pyproject.toml # Project dependencies
├── .env                        # Environment variables for local development
└── Dockerfile                  # Containerization definition
```

---

## 4. 📝 Development Workflow & Best Practices

### Step 1: API & Serializer Design (View/Serializer Layer)

1.  **Define Serializers**: Create separate `serializers.ModelSerializer` or `serializers.Serializer` classes in the `serializers.py` file. These map Django models to JSON representations and handle incoming data validation. Use built-in DRF validation and custom `validate` methods as needed. **Never expose raw Django ORM model instances directly in API responses without proper serialization.**
2.  **Create Views**: In the `views.py` file, create `rest_framework.generics` (e.g., `ListCreateAPIView`, `RetrieveUpdateDestroyAPIView`) or `rest_framework.viewsets.ModelViewSet` for CRUD operations.
3.  **Map Endpoints**: Define URL patterns in `my_app/urls.py` and include this file in the main `my_project/urls.py`. Use DRF routers for `ModelViewSet` classes for automatic URL generation.
4.  **API Versioning**: Prefix all API routes with `/api/v1/`. Example: `path('api/v1/parcels/', ParcelViewSet.as_view({'get': 'list', 'post': 'create'}), name='parcels')`.
5.  **Input/Output**: Accept incoming data via `request.data` (which is already parsed by DRF) and return `rest_framework.response.Response` objects.

### Step 2: Business Logic (Service Layer)

1.  **Create Service Modules**: In `my_app/services.py`, create functions or classes (e.g., `ParcelService`) to encapsulate core business logic. This layer should be responsible for orchestrating interactions between models, performing complex calculations, and enforcing business rules, keeping views thin.
2.  **Inject Dependencies**: Pass necessary model instances, serializer data, or other service results explicitly as arguments to service functions/methods.
3.  **Implement Logic**: Write the core business logic here. This separation ensures views focus solely on HTTP request/response handling.
4.  **Use Transactions**: For methods that modify data across multiple database operations or models, use `django.db.transaction.atomic()` to ensure data consistency.

### Step 3: Data Persistence (Model Layer)

1.  **Create Model**: In `my_app/models.py`, define `django.db.models.Model` classes. Use appropriate `models.Field` types (e.g., `CharField`, `IntegerField`, `ForeignKey`) and define relationships.
2.  **ORM Usage**: Interact with the database exclusively via the Django ORM (e.g., `MyModel.objects.create()`, `MyModel.objects.filter()`, `MyModel.objects.get()`). Avoid raw SQL queries unless absolutely necessary and ensure proper parameterization if used.

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, application-level exceptions in `my_app/exceptions.py` (e.g., `ParcelNotFoundException extends APIException` from DRF).
2.  **Global Handler**: Configure `REST_FRAMEWORK = {'EXCEPTION_HANDLER': 'my_project.utils.custom_exception_handler'}` in `settings.py`. Implement a custom exception handler that catches application-specific exceptions and maps them to appropriate `HTTP status codes` and `ErrorResponse` payloads.
3.  **Handle Exceptions**: Utilize DRF's `rest_framework.exceptions` for common API errors (e.g., `NotFound`, `ValidationError`). **Never expose raw Python stack traces in production environments.**

### Step 5: Caching

1.  **Enable Caching**: Configure `CACHES` in `my_project/settings/base.py`. Use `django.core.cache.backends.locmem.LocMemCache` for local development and `django.core.cache.backends.redis.RedisCache` (with `django-redis`) for production.
2.  **Configure Cache**: Define cache backends with appropriate settings.
3.  **Apply Caching**: Use `@cache_page` decorator for entire view responses or use `django.core.cache.cache.set()`, `cache.get()`, `cache.delete()` directly within service methods for specific expensive operations.

### Step 6: Configuration

-   **Base Config (`my_project/settings/base.py`)**: Define common properties like `SECRET_KEY`, `INSTALLED_APPS`, `MIDDLEWARE`, `REST_FRAMEWORK` defaults.
-   **Dev Profile (`my_project/settings/dev.py`)**: Configure SQLite database, set `DEBUG = True`, enable `django-debug-toolbar`, and set logging to `DEBUG`.
-   **Prod Profile (`my_project/settings/prod.py`)**: Configure PostgreSQL connection details. Use `django-environ` to load sensitive settings (e.g., `DATABASE_URL`, `SECRET_KEY`) from environment variables or a `.env` file. Set `DEBUG = False`. Disable detailed error reporting.

### Step 7: `catalog-info.yaml` File Structure

The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: # required, unique name (e.g., 'parcel-service')
  description: # required, human-readable description (e.g., 'Manages parcel tracking and delivery information.')
  tags:
    - python
    - django
    - drf
    - microservice
  annotations:
    # key-value pairs for tool integrations (e.g., GitHub repo, CI/CD pipeline link)
    github.com/project-slug: my-org/my-project-repo
    circleci.com/project-slug: github/my-org/my-project-repo
  links:
    - url: https://myproject.atlassian.net/jira/software/projects/PARCEL/boards
      title: Jira Board
      icon: dashboard
    - url: https://myproject.readthedocs.io/en/latest/
      title: Documentation
spec:
  type: service
  lifecycle: production
  owner: team-delivery
  system: logistics
  dependsOn:
    - resource:database:postgres-parcel
    - component:user-auth-service
  providesApis:
    - parcel-api
```

---

### Step 8: Coding standards

This document outlines the mandatory coding standards for all Python development projects. Adherence to these standards ensures consistency, maintainability, and quality across our codebase.

### 1. Readability & Formatting

*   **PEP 8 Compliance:** All code must adhere to [PEP 8](https://peps.python.org/pep-0008/) guidelines.
*   **Line Length:** Limit all lines to a maximum of 99 characters.
*   **Imports:**
    *   Imports should be grouped in the following order: standard library, third-party, local application.
    *   Each group should be separated by a blank line.
    *   Alphabetize imports within each group.
    *   Use absolute imports (e.g., `from my_package.sub_module import function`) rather than relative imports (e.g., `from .sub_module import function`) for clarity, especially in larger projects.
*   **Whitespace:**
    *   Use 4 spaces per indentation level.
    *   Avoid trailing whitespace.
    *   Use blank lines to separate logical sections of code within functions and between function/class definitions.
*   **Naming Conventions:**
    *   Modules: `lowercase_with_underscores`
    *   Packages: `lowercase_with_underscores`
    *   Classes: `CamelCase`
    *   Functions/Methods: `lowercase_with_underscores`
    *   Variables: `lowercase_with_underscores`
    *   Constants: `UPPERCASE_WITH_UNDERSCORES`
    *   Private members (intended for internal use only): Prefix with a single underscore (e.g., `_private_method`, `_private_variable`).
    *   Dunder methods (e.g., `__init__`): Only use when overriding Python's built-in behavior.

### 2. Documentation

*   **Docstrings:**
    *   All modules, classes, methods, and functions must have [PEP 257](https://peps.python.org/pep-0257/) compliant docstrings.
    *   Use Google Style Docstrings for consistency.
    *   Explain purpose, arguments, return values, and any exceptions raised.
*   **Comments:**
    *   Use comments sparingly, only when code is not self-explanatory.
    *   Comments should explain *why* something is done, not *what* is done.
    *   Keep comments up-to-date with code changes.
*   **Type Hinting:** Use [PEP 484](https://peps.python.org/pep-0484/) type hints for all function arguments and return values, and for complex variable types.

### 3. Code Quality & Best Practices

*   **No Hardcoding:** Avoid hardcoding configuration values. Use environment variables, configuration files, or Django's settings system.
*   **Error Handling:**
    *   Use specific exceptions instead of broad `except Exception:` clauses.
    *   Log exceptions appropriately with sufficient context.
    *   Handle expected errors gracefully; propagate unexpected errors.
*   **Logging:**
    *   Use Python's standard `logging` module.
    *   Log messages should be clear, concise, and provide enough context for debugging.
    *   Avoid logging sensitive information.
*   **Avoid Global State:** Minimize the use of global variables. Prefer passing data explicitly.
*   **Function Purity & Side Effects:** Strive for pure functions where possible, especially in utility modules. Clearly document functions with side effects.
*   **Database Access:**
    *   Use Django's ORM effectively. Avoid raw SQL queries unless absolutely necessary and ensure they are parameterized to prevent SQL injection.
    *   Optimize database queries (e.g., `select_related`, `prefetch_related`).
*   **Security:**
    *   Sanitize all user inputs.
    *   Protect against common web vulnerabilities (XSS, CSRF, SQL Injection) by leveraging Django's built-in protections.
    *   Never store sensitive information (e.g., API keys, passwords) directly in code. Use environment variables or secure vault solutions.
*   **Performance:**
    *   Profile code to identify bottlenecks before optimizing prematurely.
    *   Use caching for expensive operations.

### 4. Tooling & Automation

*   **Linters:** Integrate `Flake8` (or `Ruff`), `Black`, and `isort` into the CI/CD pipeline and local development setup.
    *   `Black` for uncompromising code formatting.
    *   `isort` for sorting imports.
    *   `Flake8` (or `Ruff`) for checking PEP 8 compliance and detecting common errors.
*   **Static Analysis:** Use `MyPy` for type checking.
*   **Pre-commit Hooks:** Encourage the use of `pre-commit` hooks to automate formatting and linting before commits.

### 5. Testing

*   **Test-Driven Development (TDD):** Embrace TDD principles where appropriate.
*   **Unit Tests:** Write unit tests for all critical functions, classes, and methods.
*   **Integration Tests:** Write integration tests for API endpoints and interactions between components.
*   **Coverage:** Aim for a minimum of 80% code coverage. Use tools like `coverage.py`.
*   **Mocking:** Use `unittest.mock` or `pytest-mock` for isolating components during testing.

---

### Step 9: Documentation

-   **OpenAPI**: Install `drf-spectacular` (`pip install drf-spectacular`). Configure it in `settings.py` by adding `'drf_spectacular'` to `INSTALLED_APPS` and defining `SPECTACULAR_SETTINGS`. Include the schema views in `my_project/urls.py` (e.g., `path('api/schema/', SpectacularAPIView.as_view(), name='schema')`, `path('api/schema/swagger-ui/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui')`).
-   **Docstrings**: Add comprehensive docstrings to all modules, classes, methods, and functions following PEP 257 and Google Style, explaining their purpose, arguments, return values, and any side effects or exceptions.

### Step 10: Testing

-   **Unit Tests**: Write focused tests for serializers, services, custom managers, and isolated model methods. Use `pytest`, `unittest.mock`, or `pytest-mock` to mock dependencies. Place tests in `my_app/tests/`.
-   **Integration Tests**: Use `pytest-django` and `rest_framework.test.APITestCase` to test API endpoints, verifying behavior from the view layer through to the database. Utilize tools like `factoryboy` or `pytest-factoryboy` for efficient test data generation.
-   **Coverage**: Aim for a minimum of 80% test coverage for all application code. Integrate `coverage.py` into your testing workflow.

---

## 5. 🐳 Containerization & Deployment

### Dockerfile

Create a multi-stage `Dockerfile` for efficient image size and security:

1.  **Build Stage**: Use a Python builder image (e.g., `python:3.11-slim-buster`). Copy `pyproject.toml` (or `requirements.txt`) and install dependencies using `poetry install --no-root --only main` (or `pip install -r requirements.txt`).
2.  **Runtime Stage**: Use a minimal Python JRE-equivalent image (e.g., `python:3.11-slim-buster`).
3.  **Security**: Create and run the application as a non-root user (e.g., `USER appuser`).
4.  **Health Check**: Add a `HEALTHCHECK` instruction (e.g., `HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 CMD curl --fail http://localhost:8000/api/v1/health/ || exit 1`). A `/api/v1/health/` endpoint should be implemented.
5.  **Execution**: Use `CMD ["gunicorn", "my_project.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4", "--log-level", "info"]` (or `uvicorn` for ASGI applications).

### Environment Variables

All secrets (Django `SECRET_KEY`, database passwords, API keys, etc.) **must** be supplied via environment variables, not hardcoded in settings files. Use `django-environ` to gracefully handle environment variables in `settings.py`.

---

This protocol ensures consistency, security, and quality across all generated Django REST Framework microservices. The agent must validate its output against these guidelines before completing a task.