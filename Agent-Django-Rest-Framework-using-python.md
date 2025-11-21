Here's a detailed spec file for Django Rest Framework using Python, adapted from the `Agent-template.md` and incorporating the requested Python coding standards.

I've replaced the `#Coding standards#` section with the content from `python-rules.txt` as it is the most relevant for a Python-based spec. The Java Spring Boot and Angular coding standards, while provided in the prompt, would typically be integrated into a multi-language agent template, or specific language-focused templates, rather than a single Python DRF spec.

---

# 🤖 Agent Protocol: Building Enterprise-Grade Python Microservices with Django Rest Framework

This document provides a comprehensive guide for AI coding agents to build secure, scalable, and maintainable microservices using Python 3.11+ and Django Rest Framework. Adherence to these principles is mandatory for all generated projects.

---

## 1. 🎯 Core Principles

-   **Security First**: Implement security controls at every layer. Assume a zero-trust environment.
-   **Production Ready**: All code must be suitable for production, with robust configuration, logging, and error handling.
-   **Developer Experience**: Ensure the project is easy to set up, run, and understand. Prioritize clear documentation and clean code.
-   **Performance by Design**: Build for low latency and high throughput. Implement caching and efficient data access patterns.
-   **Observability**: The system's internal state must be inferable from its external outputs (logs, metrics, traces).

---

## 2. 🛠️ Standard Technology Stack

| Component            | Technology                     | Version/Standard       |
| :------------------- | :----------------------------- | :--------------------- |
| **Language**         | Python                         | **3.11+**              |
| **Web Framework**    | Django                         | **5.0+**               |
| **API Framework**    | Django Rest Framework          | **3.15+**              |
| **Build/Deps Tool**  | Poetry (preferred), pip        | Latest                 |
| **Database**         | PostgreSQL (Prod), SQLite (Dev/Test) | Latest                 |
| **API Spec**         | OpenAPI (drf-spectacular)      | 3.0                    |
| **Container**        | Docker                         | Latest                 |
| **Caching**          | Django Cache (Redis, Memcached) | Latest                 |
| **Validation**       | Django Model Validators, DRF Serializer Validators | Latest                 |
| **Testing**          | Pytest, Factory Boy            | Latest                 |
| **Linter/Formatter** | Black, Flake8, MyPy            | Latest                 |

---

## 3. 🏗️ Project Initialization & Structure

### Initialization

Generate new projects using `django-admin` and manage dependencies with `Poetry` (preferred) or `pip`.

1.  **Create Django Project**:
    ```bash
    django-admin startproject project_name .
    ```
2.  **Create Django App**:
    ```bash
    python manage.py startapp app_name
    ```
3.  **Initialize Poetry (or pip environment)**:
    ```bash
    poetry init # then poetry add django djangorestframework psycopg2-binary django-environ drf-spectacular ...
    # or pip install -r requirements.txt
    ```
4.  **Add `app_name` to `INSTALLED_APPS` in `settings.py`**.

### Standard Project Structure

Organize all source code under a root project (e.g., `my_service_project`).

```
my_service_project/
├── manage.py
├── project_name/                 # Main Django project settings
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings/                 # Settings directory (base, dev, prod)
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── dev.py
│   │   └── prod.py
│   ├── urls.py
│   └── wsgi.py
├── app_name/                     # Django application (e.g., parcels)
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations/
│   ├── models.py                 # JPA-equivalent Entities
│   ├── views.py                  # REST API views (Controller layer)
│   ├── serializers.py            # Data Transfer Objects (DTOs) for API
│   ├── urls.py
│   ├── services.py               # Business logic (Service layer, optional but recommended)
│   ├── tests/                    # Unit and integration tests
│   │   ├── __init__.py
│   │   ├── test_models.py
│   │   ├── test_serializers.py
│   │   └── test_views.py
│   └── exceptions.py             # Custom exceptions
├── docs/                         # OpenAPI documentation, etc.
├── tests/                        # Top-level integration/e2e tests (optional)
├── .env.example
├── Dockerfile
├── poetry.lock                   # or requirements.txt
└── pyproject.toml                # or setup.py
└── README.md
```

---

## 4. 📝 Development Workflow & Best Practices

### Step 1: API & Serializer Design (View & Serializer Layer)

1.  **Define Serializers**: Create `rest_framework.serializers.Serializer` or `ModelSerializer` classes in `app_name/serializers.py`. Use Django's built-in validators or DRF's `validators` for all incoming data. **Never expose Django Model instances directly in API responses without serialization.**
2.  **Create Views/ViewSets**: In `app_name/views.py`, create `APIView` or `ModelViewSet` classes. `ModelViewSet` is preferred for standard CRUD operations due to its convention over configuration.
3.  **Map Endpoints**:
    *   Define URLs in `app_name/urls.py` using `path()` or `re_path()`.
    *   Include `app_name/urls.py` in `project_name/urls.py`.
    *   For `ModelViewSet`s, use `rest_framework.routers.DefaultRouter` to automatically generate URL patterns.
4.  **API Versioning**: Prefix all routes with `/api/v1/`. Example: `path('api/v1/parcels/', ...)` or `router.register(r'api/v1/parcels', ...)`.
5.  **Input/Output**: Accept serialized data from `request.data` (which is validated by serializers) and return `rest_framework.response.Response` objects.

### Step 2: Business Logic (Service Layer)

1.  **Create Service Functions/Classes**: In `app_name/services.py` (or a similar dedicated module), define functions or classes that encapsulate business logic. This layer should be decoupled from the views.
2.  **Inject Dependencies**: Pass necessary models or other services as arguments to service functions or use dependency injection patterns for classes.
3.  **Implement Logic**: Write core business logic here. This layer coordinates data access, calls other services, and enforces business rules.
4.  **Use Transactions**: Use `django.db.transaction.atomic()` for methods that modify data to ensure atomicity.
5.  **Map DTOs to Entities**: Serializers primarily handle this. Service functions receive validated data from serializers and interact with Django Models.

### Step 3: Data Persistence (Models Layer)

1.  **Create Model**: In `app_name/models.py`, define Django `models.Model` classes. Use appropriate `Field` types and `verbose_name` / `help_text` for clarity.
2.  **Define Relationships**: Use `ForeignKey`, `OneToOneField`, `ManyToManyField` for relationships.
3.  **Add Custom Managers**: Implement `models.Manager` for reusable query logic.

### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in `app_name/exceptions.py` (e.g., `ResourceNotFoundException(APIException)`).
2.  **Global Handler**: Configure DRF's `EXCEPTION_HANDLER` in `settings.py` to point to a custom exception handler function.
3.  **Handle Exceptions**: In the custom handler, map specific exceptions (e.g., `Http404`, `ValidationError`, custom exceptions) to appropriate HTTP status codes and standardized error responses. **Never expose stack traces in production.**

### Step 5: Caching

1.  **Configure Cache**: Define `CACHES` in `settings.py` (e.g., `locmem` for local, `django-redis` for distributed Redis).
2.  **Apply Caching**:
    *   Use `django.core.cache.cache` directly for programmatic caching.
    *   Use `cache_page` decorator for caching entire view responses.
    *   Use `vary_on_cookie` / `vary_on_header` decorators for conditional caching based on request headers.
    *   Apply caching to expensive service functions or ORM queries.

### Step 6: Configuration

-   **Base Config (`settings/base.py`)**: Define common properties like `INSTALLED_APPS`, `MIDDLEWARE`, `REST_FRAMEWORK` settings.
-   **Dev Profile (`settings/dev.py`)**: Inherit from `base.py`. Configure `DEBUG=True`, `ALLOWED_HOSTS=['*']`, use SQLite database, enable `django-debug-toolbar` (if used).
-   **Prod Profile (`settings/prod.py`)**: Inherit from `base.py`. Configure `DEBUG=False`, specific `ALLOWED_HOSTS`, PostgreSQL connection details using environment variables. Disable detailed error reporting to users.
-   **Environment Variables**: Use `django-environ` or `decouple` to manage environment-specific variables and secrets.

### Step 7: `catalog-info.yaml` File Structure

The `catalog-info.yaml` file should be placed in the root directory of the component's source code repository.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: # required, unique name (e.g., parcel-service)
  description: # required, human-readable description
  tags:
    - # list of tags for discoverability (e.g., python, django, microservice, rest-api)
  annotations:
    # key-value pairs for tool integrations
    # e.g., 'backstage.io/kubernetes-id': 'parcel-service'
    # e.g., 'backstage.io/techdocs-ref': 'url:https://github.com/org/repo/tree/main/docs'
  links:
    - url: # URL to a relevant resource (e.g., 'https://github.com/org/repo')
      title: # Display title for the link (e.g., 'GitHub Repository')
      icon: # Optional: e.g., 'github'
spec:
  type: # required, e.g., 'service', 'library'
  lifecycle: # required, e.g., 'experimental', 'production'
  owner: # required, the owning team or user (e.g., 'team-backend')
  system: # optional, the larger system this component belongs to
  dependsOn:
    - # list of components/resources this component depends on (e.g., 'resource:default/postgresql')
  providesApis:
    - # list of APIs this component provides (e.g., 'parcel-api')
```

---

### Step 8: Coding standards

# Python Coding Standards (PEP 8 Compliant)

1.  **Indentation:**
    *   Use 4 spaces per indentation level. Never mix spaces and tabs.

2.  **Line Length:**
    *   Limit all lines to a maximum of 79 characters for code and 72 characters for docstrings and comments.
    *   For longer lines, use parentheses, brackets, or braces for implicit line continuation.

3.  **Blank Lines:**
    *   Surround top-level function and class definitions with two blank lines.
    *   Method definitions inside a class are surrounded by a single blank line.
    *   Use blank lines sparingly to separate logical sections of code.

4.  **Imports:**
    *   Imports should usually be on separate lines.
    *   Imports should be grouped in the following order:
        1.  Standard library imports.
        2.  Third-party related imports.
        3.  Local application/library specific imports.
    *   Each group should be separated by a blank line.
    *   Absolute imports are recommended over relative imports.

5.  **Whitespace in Expressions and Statements:**
    *   Avoid extraneous whitespace in the following situations:
        *   Immediately inside parentheses, brackets or braces.
        *   Immediately before a comma, semicolon, or colon.
        *   Immediately before the open parenthesis that starts the argument list of a function call.
        *   Immediately before the open parenthesis that starts an indexing or slicing.
    *   Always surround binary operators with a single space on either side.

6.  **Naming Conventions:**
    *   **Modules:** short, all-lowercase names. Underscores can be used if it improves readability.
    *   **Packages:** short, all-lowercase names.
    *   **Classes:** CapWords (PascalCase) convention.
    *   **Type variables:** CapWords.
    *   **Functions & Methods:** lowercase_with_underscores (snake_case).
    *   **Variables:** lowercase_with_underscores (snake_case).
    *   **Constants:** ALL_CAPS_WITH_UNDERSCORES.
    *   **Instance methods:** `self` as the first parameter.
    *   **Class methods:** `cls` as the first parameter.
    *   **Private members:** Prefix with a single underscore (`_private_method`).
    *   **Name mangling:** Prefix with double underscore (`__mangled_attribute`).

7.  **Docstrings:**
    *   Write docstrings for all public modules, functions, classes, and methods.
    *   Use triple double quotes (`"""Docstring goes here."""`).
    *   For one-liner docstrings, keep the closing quotes on the same line.
    *   For multi-line docstrings, the opening quotes are on a line by themselves, followed by the summary, a blank line, the rest of the description, and the closing quotes on a line by themselves.

8.  **Comments:**
    *   Comments should be complete sentences and start with a capital letter.
    *   Update comments when the code changes.
    *   Use `#` for inline comments.
    *   Block comments should be indented to the same level as the code they describe.

9.  **Type Hinting:**
    *   Use type hints for function arguments and return values for improved readability and maintainability, especially in larger codebases.
    *   Example: `def greet(name: str) -> str:`.

10. **Error Handling:**
    *   Use specific exceptions.
    *   Avoid bare `except:` clauses. Always specify the exception type.
    *   Use `finally` for cleanup code.

11. **Tooling:**
    *   Use `Black` for automatic code formatting.
    *   Use `Flake8` or `Pylint` for linting and style checking.
    *   Use `MyPy` for static type checking.

12. **DRY Principle:**
    *   Don't Repeat Yourself. Refactor common logic into reusable functions or classes.

13. **Magic Numbers/Strings:**
    *   Avoid using magic numbers or strings directly in the code. Define them as named constants.

---

### Step 9: Documentation

-   **OpenAPI**: Integrate `drf-spectacular` for automatic OpenAPI (Swagger) documentation generation. Configure schema title, version, and description in `settings.py`.
-   **Docstrings**: Add PEP 257 compliant docstrings to all public modules, classes, and methods in views, serializers, models, and services.
-   **README.md**: Maintain a comprehensive `README.md` in the project root with setup instructions, environment variables, how to run tests, and API endpoints.

### Step 10: Testing

-   **Unit Tests**: Test individual components (serializers, service functions, model methods) in isolation, mocking dependencies. Place tests in `app_name/tests/`. Use `pytest`.
-   **Integration Tests**: Use Django's `TestCase` or `APITestCase` (from `rest_framework.test`) to test API endpoints, verifying full request-response cycles including database interactions.
-   **Coverage**: Aim for a minimum of 80% test coverage using `pytest-cov`.
-   **Test Data**: Use `Factory Boy` to generate realistic test data efficiently for models and serializers.

---

## 5. 🐳 Containerization & Deployment

### Dockerfile

Create a multi-stage `Dockerfile`:

1.  **Build Stage**: Use a Python base image (e.g., `python:3.11-slim-buster`) to install dependencies and collect static files.
2.  **Runtime Stage**: Use a minimal Python runtime image (e.g., `python:3.11-slim-buster`).
3.  **Application Server**: Use Gunicorn or uWSGI as the production application server.
    ```dockerfile
    # Example for Gunicorn
    CMD ["gunicorn", "--bind", "0.0.0.0:8000", "project_name.wsgi:application"]
    ```
4.  **Security**: Create and run as a non-root user.
    ```dockerfile
    RUN adduser --system --group appuser
    USER appuser
    ```
5.  **Health Check**: Add a `HEALTHCHECK` instruction that calls a simple `/health` or `/ready` endpoint provided by the application.
6.  **Execution**: Use `CMD` to start the Gunicorn/uWSGI server.

### Environment Variables

All secrets (database passwords, API keys, `SECRET_KEY`) **must** be supplied via environment variables, not hardcoded in `settings.py` files. Use `django-environ` or `decouple` to manage this.

---

This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.