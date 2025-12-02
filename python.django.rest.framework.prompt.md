---
agent: agent
model: Claude Sonnet 4.5 (copilot)
name: python.django.rest.framework
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
| **Framework** | Django, Django REST Framework | **4.2+, 3.15+** |
| **Package Manager** | pip, poetry | Latest |
| **Database** | PostgreSQL (Prod), SQLite (Dev/Test) | Latest |
| **API Spec** | OpenAPI (drf-spectacular) | 3.0 |
| **Container** | Docker | Latest |
| **Caching** | Django Cache (Redis for distributed) | Latest |
| **Validation** | Django's built-in, DRF Serializer validation | Latest |
| **Testing** | Pytest, Django's TestCase, pytest-django | Latest |
 
---
 
## 3. 🏗️ Project Initialization & Structure
 
### Initialization

Generate new projects using `django-admin startproject` and `manage.py startapp` with the following default dependencies (in `requirements.txt`):

- `Django`
- `djangorestframework`
- `psycopg2-binary` (for PostgreSQL)
- `drf-spectacular` (for OpenAPI docs)
- `django-filter` (for filtering API results)
- `djang-cors-headers` (for CORS handling)
- `django-environ` (for environment variable management)
 
### Standard Package Structure

Organize all source code under a root project (e.g., `myproject`).
 
```
myproject/
├── myproject/                      # Root project settings, URLs, WSGI
│   ├── __init__.py
│   ├── settings/                   # Split settings for dev/prod
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── dev.py
│   │   └── prod.py
│   ├── urls.py                     # Global URL configurations
│   ├── wsgi.py
│   └── asgi.py
├── manage.py                       # Django's command-line utility
├── myapp/                          # Django application (e.g., 'parcels')
│   ├── migrations/                 # Database schema migrations
│   ├── __init__.py
│   ├── admin.py                    # Django Admin configurations
│   ├── apps.py                     # Application configuration
│   ├── models.py                   # Django ORM models (JPA Entities equivalent)
│   ├── views.py                    # DRF API Views or ViewSets (REST controllers equivalent)
│   ├── serializers.py              # DRF Serializers (DTOs equivalent)
│   ├── urls.py                     # Application-specific URL patterns
│   ├── permissions.py              # Custom DRF permissions
│   ├── filters.py                  # Custom DRF filters
│   └── tests/                      # Unit and integration tests
│       ├── __init__.py
│       ├── test_models.py
│       ├── test_serializers.py
│       └── test_views.py
├── config/                         # Global configurations (e.g., custom middlewares, logging)
├── utils/                          # Common utility functions
├── exceptions/                     # Custom exceptions and error handlers
├── requirements.txt                # Python package dependencies
└── Dockerfile                      # Containerization definition
```
 
---
 
## 4. 📝 Development Workflow & Best Practices
 
### Step 1: API & Serializer Design (View Layer)

1.  **Define Serializers**: Create separate `rest_framework.serializers.Serializer` or `ModelSerializer` classes for API requests and responses in the `serializers.py` file of the respective app. Use DRF validation fields (`fields.CharField`, `fields.EmailField`, etc.) and custom validators. **Never expose Django Model instances directly in the API responses without proper serialization.**

2.  **Create View/ViewSet**: In the `views.py` file, create a `rest_framework.views.APIView`, `generics.GenericAPIView`, or `viewsets.ModelViewSet`.

3.  **Map Endpoints**: Define URL patterns in `urls.py` files. Use Django's `path()` or `re_path()` for `APIView`s, and `rest_framework.routers.DefaultRouter` for `ViewSet`s.

4.  **API Versioning**: Prefix all routes with `/api/v1/`. Example: `path('api/v1/parcels/', ...)` in `app_urls.py` and `path('', include('myapp.urls'))` in `project_urls.py` within a versioned path.

5.  **Input/Output**: Accept serialized data from `request.data` and return `rest_framework.response.Response` containing serialized data or error messages.
 
### Step 2: Business Logic (Service/Manager Layer)

1.  **Implement Logic**: Write core business logic within dedicated service classes (e.g., `myapp/services.py`) or manager methods (custom methods on Django Managers `models.Manager`). For simpler cases, logic can reside directly in `views.py`. This layer is responsible for coordinating data access, calling external services, and enforcing business rules.

2.  **Use Transactions**: Use Django's `transaction.atomic()` decorator or context manager for methods that modify data across multiple models to ensure atomicity.

3.  **Map Serializers to Models**: The view/service layer is responsible for converting incoming serialized data to Django Model instances before saving, and converting Model instances to serialized data before returning them to the controller.
 
### Step 3: Data Persistence (Models Layer)

1.  **Create Model**: In the `models.py` file, define a Django `models.Model` class. Use appropriate `models.Field` types (`CharField`, `IntegerField`, `ForeignKey`, `DateTimeField`, etc.).

2.  **Use ORM**: Interact with the database using Django's Object-Relational Mapper (ORM) (e.g., `Model.objects.create()`, `Model.objects.filter()`, `Model.objects.get()`). Avoid raw SQL queries unless absolutely necessary.
 
### Step 4: Exception Handling

1.  **Custom Exceptions**: Create specific, unchecked exceptions in the `exceptions/` directory (e.g., `ResourceNotFoundException(APIException)` extending DRF's `APIException`).

2.  **Global Handler**: Configure a custom `EXCEPTION_HANDLER` in `settings.py` pointing to a function (e.g., `exceptions.handlers.custom_exception_handler`) that handles DRF's `APIException`s and Django's `Http404`, `PermissionDenied`, etc.

3.  **Handle Exceptions**: Map custom exceptions to appropriate HTTP status codes and return a standardized JSON error response (e.g., `{"detail": "Error message"}`). **Never expose stack traces directly in API responses.**
 
### Step 5: Caching

1.  **Enable Caching**: Configure `CACHES` in your `settings.py` file.

2.  **Configure Cache**: Define cache backends (e.g., `django.core.cache.backends.locmem.LocMemCache` for local development, `django.core.cache.backends.redis.RedisCache` for distributed production caching).

3.  **Apply Caching**: Use Django's low-level cache API (`cache.set()`, `cache.get()`), `cache_page` decorator for entire views, or `cache_per_user` for authenticated views.
 
### Step 6: Configuration

- **Base Config (`myproject/settings/base.py`)**: Define common properties like `SECRET_KEY` (from env), `DEBUG` (from env), `ALLOWED_HOSTS`, `INSTALLED_APPS`, `MIDDLEWARE`, `DATABASES` (defaulting to SQLite), `REST_FRAMEWORK` settings, `SPECTACULAR_SETTINGS`.

- **Dev Profile (`myproject/settings/dev.py`)**: Inherit from `base.py`, configure SQLite database, enable Django Debug Toolbar (if used), set logging to `DEBUG`.

- **Prod Profile (`myproject/settings/prod.py`)**: Inherit from `base.py`, configure PostgreSQL connection details using `django-environ` to read from environment variables (e.g., `DATABASES = {"default": env.db("DATABASE_URL")}`). Set `DEBUG = False`. Disable detailed health info.

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
  type: # required, e.g., 'service', 'library'
  lifecycle: # required, e.g., 'experimental', 'production'
  owner: # required, the owning team or user
  system: # optional, the larger system this component belongs to
  dependsOn:
    - # list of components/resources this component depends on
  providesApis:
    - # list of APIs this component provides
```
 
---
 ### Step 8: #Coding standards#

**Python/Django/DRF Coding Standards**
 
**1. General Python Standards (PEP 8 Adherence):**
*   **Indentation:** Use 4 spaces per indentation level.
*   **Line Length:** Max 79 characters for code, 72 for docstrings and comments.
*   **Blank Lines:**
    *   Two blank lines between top-level function/class definitions.
    *   One blank line between method definitions within a class.
    *   One blank line around logical sections within functions/methods.
*   **Imports:**
    *   Imports should be on separate lines.
    *   Group imports: standard library, third-party, local application imports.
    *   Alphabetical order within each group.
    *   Use absolute imports.
*   **Naming Conventions:**
    *   Modules: `lowercase_with_underscores`.
    *   Packages: `lowercase_with_underscores`.
    *   Classes: `CamelCase`.
    *   Functions/Methods: `lowercase_with_underscores`.
    *   Variables: `lowercase_with_underscores`.
    *   Constants: `ALL_CAPS_WITH_UNDERSCORES`.
    *   Private members (internal use only): Prefix with a single underscore (`_`).
    *   Name mangling: Double underscore (`__`) should be avoided unless strictly necessary for inheritance.
*   **Comments:**
    *   Use clear, concise comments where code isn't self-explanatory.
    *   Inline comments should be separated by at least two spaces from the statement.
    *   Block comments apply to some (or all) code that follows them, and are indented to the same level as that code.
*   **Docstrings (PEP 257):**
    *   Use triple double-quotes (`"""Docstring"""`).
    *   All public modules, functions, classes, and methods should have docstrings.
    *   Multi-line docstrings should summarize on the first line, then describe arguments, return values, and exceptions. Use Sphinx or Google style.
*   **White Space:**
    *   No extraneous whitespace immediately inside parentheses, brackets, or braces.
    *   No whitespace immediately before a comma, semicolon, or colon.
    *   One whitespace around operators and assignments.
    *   No trailing whitespace.
*   **Conditional Statements:**
    *   Prefer `is None` over `== None`.
    *   Use `if not x:` instead of `if x is None:` or `if x == False:`.
    *   `if condition:` for truthiness, `if not condition:` for falsiness.

**2. Django Specific Standards:**
*   **Models:**
    *   Define `__str__` methods for all models for better readability in admin and debugging.
    *   Use verbose names for fields (e.g., `CharField(max_length=100, verbose_name="Customer Name")`).
    *   Use `related_name` for `ForeignKey` and `ManyToManyField` relationships to simplify reverse lookups.
    *   Avoid using `null=True` on `CharField` and `TextField`; use `blank=True` and a `default=''` instead.
    *   Use `db_index=True` for frequently queried fields.
    *   Custom Managers: Implement custom querysets/methods on a `Manager` when complex query logic is reusable across views/services.
*   **Views:**
    *   For API development, prefer Django REST Framework's `GenericAPIView` or `ModelViewSet` over `APIView` for common CRUD operations.
    *   Keep views thin; delegate complex business logic to service layers or model managers.
    *   Avoid direct database access in views unless it's a very simple lookup.
*   **URLs:**
    *   Use explicit `name` arguments for URL patterns to allow easy referencing (e.g., `{% url 'app_name:view_name' %}`).
    *   Organize URLs into app-specific `urls.py` files and include them in the project's root `urls.py`.
    *   Use API versioning (e.g., `/api/v1/`).
*   **Settings:**
    *   Follow the best practice of splitting `settings.py` into `base.py`, `dev.py`, and `prod.py` using `django-split-settings` or similar.
    *   Use `django-environ` or similar library to manage environment variables for sensitive data and configuration differences between environments.
    *   Do not hardcode sensitive information (e.g., database credentials, secret keys) in settings files.
*   **Templates (if used for non-API views):**
    *   Separate HTML, CSS, and JS.
    *   Avoid business logic in templates; pass pre-processed data from views.

**3. Django REST Framework (DRF) Specific Standards:**
*   **Serializers:**
    *   Use `ModelSerializer` for creating serializers that map directly to Django models.
    *   Define explicit fields for `ModelSerializer` using `fields = '__all__'` or `fields = ('field1', 'field2')`. Avoid `exclude = ...` unless absolutely necessary.
    *   Implement custom validation logic within `validate_<field_name>` or `validate` methods.
    *   Use `SerializerMethodField` for custom read-only fields that depend on other fields or complex logic.
    *   Avoid putting complex business logic inside serializers; they should focus on data validation and representation.
*   **Permissions:**
    *   Use DRF's built-in permissions (e.g., `IsAuthenticated`, `AllowAny`, `IsAdminUser`) where possible.
    *   Create custom permissions by inheriting from `BasePermission` for fine-grained access control.
    *   Apply permissions at the `APIView` or `ViewSet` level using `permission_classes`.
*   **Authentication:**
    *   Use DRF's authentication classes (e.g., `TokenAuthentication`, `SessionAuthentication`, `JWTAuthentication`).
    *   For API tokens, avoid exposing them in URLs.
*   **Filtering, Searching, Ordering:**
    *   Use `django-filter` integration for robust filtering capabilities.
    *   Implement `SearchFilter` and `OrderingFilter` from DRF for common search and sort needs.
    *   Document available filters, search fields, and ordering options in API documentation.
*   **API Versioning:**
    *   Consistently apply an API versioning strategy (e.g., URLPathVersioning, NamespaceVersioning). URLPathVersioning (`/api/v1/`) is generally recommended for clarity.
*   **Error Handling:**
    *   Use DRF's default exception handling or a custom exception handler (as per Step 4) for consistent error responses.
    *   Return meaningful error messages and appropriate HTTP status codes (e.g., 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error).
*   **Pagination:**
    *   Implement pagination for list views to prevent large data transfers and improve performance. Use `LimitOffsetPagination` or `PageNumberPagination`.
    *   Configure default pagination settings in `REST_FRAMEWORK` settings.

**4. Security Best Practices:**
*   **SQL Injection:** Use Django ORM methods; avoid raw SQL or sanitize user input if raw SQL is unavoidable.
*   **XSS:** Django's templating engine automatically escapes output, but be careful with raw HTML in templates or user-submitted content. DRF serializers handle input validation, preventing XSS in API.
*   **CSRF:** Django provides built-in CSRF protection for session-based authentication. For token-based APIs, CSRF is less of a concern but ensure token storage is secure.
*   **Authentication & Authorization:** Implement strong authentication mechanisms and granular permissions.
*   **Sensitive Data:** Encrypt sensitive data at rest and in transit. Never log sensitive user data.
*   **Dependencies:** Regularly update dependencies to patch known vulnerabilities. Use tools like `pip-audit`.
*   **CORS:** Configure `django-cors-headers` explicitly in `settings.py` for `CORS_ALLOWED_ORIGINS` or `CORS_ALLOW_ALL_ORIGINS = True` (only for development/testing).

**5. Testing:**
*   **Unit Tests:** Write tests for models, serializers, utility functions.
*   **Integration Tests:** Write tests for API endpoints using DRF's `APITestCase` to verify end-to-end functionality.
*   **Test Coverage:** Aim for high test coverage (e.g., 80%+) using tools like `pytest-cov`.
*   **Fixtures/Factories:** Use `pytest-factoryboy` or Django fixtures for test data generation.

**6. Linting & Formatting:**
*   Use `Black` for uncompromised code formatting.
*   Use `Flake8` for linting (combines Pyflakes, pycodestyle, and McCabe complexity).
*   Integrate linters/formatters into CI/CD pipelines and pre-commit hooks.

**7. Logging:**
*   Configure Django's logging in `settings.py`.
*   Log meaningful events (info, warnings, errors) with appropriate severity levels.
*   Avoid logging sensitive information.

### Step 9: Documentation

- **OpenAPI**: Configure `drf-spectacular` in `settings.py` to generate OpenAPI schema. Use `SPECTACULAR_SETTINGS` to set the API title, version, and description. Document API endpoints with docstrings on views/serializers for better schema generation.

- **Docstrings**: Add PEP 257 compliant docstrings to all public modules, classes, methods, and functions, explaining their purpose, parameters, and return values.

- **README.md**: Maintain a comprehensive `README.md` file at the project root with setup instructions, environment variables, API endpoints, and deployment steps.
 
### Step 10: Testing

- **Unit Tests**: Test models, serializers, and helper functions by mocking dependencies. Place tests in `app_name/tests/` (e.g., `myapp/tests/test_models.py`). Use `pytest` for running tests.

- **Integration Tests**: Use `rest_framework.test.APITestCase` or `pytest-django` to test the full application flow, from API endpoint to database. Use Testcontainers for database integration tests to ensure isolation and consistent environments.

- **Coverage**: Aim for a minimum of 80% test coverage using `pytest-cov`.
 
---
 
## 5. 🐳 Containerization & Deployment
 
### Dockerfile

Create a multi-stage `Dockerfile`:

1.  **Build Stage**: Use a Python base image (e.g., `python:3.10-slim-buster`). Install project dependencies from `requirements.txt`.

2.  **Runtime Stage**: Use a minimal Python base image (e.g., `python:3.10-slim-buster`). Copy only necessary application code and static files.

3.  **Security**: Create and run as a non-root user.

4.  **Health Check**: Add a `HEALTHCHECK` instruction that calls a custom health check endpoint (e.g., `/health/` or `/api/v1/health/`) to verify application responsiveness.

5.  **Execution**: Use `ENTRYPOINT` and `CMD` instructions to run the application with a production-ready WSGI server like Gunicorn (e.g., `CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myproject.wsgi:application"]`).
 
### Environment Variables

All secrets (database URLs, API keys, `SECRET_KEY`, etc.) **must** be supplied via environment variables, not hardcoded in settings files. Use `django-environ` to easily load these variables into your Django settings.
 
---
 
This protocol ensures consistency, security, and quality across all generated microservices. The agent must validate its output against these guidelines before completing a task.
 
---
 
### GitHub Copilot Prompt File Template (slash command)

```
/create-django-rest-component
Generate a {component_type} for a Django REST Framework application.

**Component Type:** {Model/Serializer/ViewSet/APIView/Permission/Filter/Service}
**Model Name (if applicable):** {e.g., Parcel, Product}
**Fields (for Model/Serializer):**
- {field_name}: {field_type} {constraints, e.g., CharField(max_length=255), ForeignKey(User)}
- ...
**Actions (for ViewSet/APIView):** {e.g., CRUD operations, list, retrieve, create, update, delete, custom_action}
**Permissions (for ViewSet/APIView):** {e.g., IsAuthenticated, IsAdminUser, CustomPermission}
**Relationships (for Model/Serializer):** {e.g., One-to-many with User, Many-to-many with Tag}
**Description/Context:** {Provide a brief description of the component's purpose and any specific requirements, e.g., "The Parcel model should have a foreign key to a User and include sender/receiver addresses."}

---
Example:
/create-django-rest-component
Generate a ViewSet for a Django REST Framework application.

**Component Type:** ViewSet
**Model Name (if applicable):** Order
**Actions (for ViewSet/APIView):** list, retrieve, create, update, delete, mark_as_shipped (custom action)
**Permissions (for ViewSet/APIView):** IsAuthenticated, IsAdminUser
**Description/Context:** The OrderViewSet should allow users to view their own orders and admins to manage all orders. The custom action `mark_as_shipped` should only be accessible by admins.
```