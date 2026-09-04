# Swagger/OpenAPI Documentation with Agent

Guide for generating OpenAPI 3.0 (Swagger) documentation for REST API endpoints using the Documentation Agent.

## Overview

The Documentation Agent now supports generating comprehensive Swagger/OpenAPI documentation for your REST endpoints:

- **Python Docstrings**: NumPy format in ViewSets (English only)
- **OpenAPI YAML Specs**: Complete API specification (Bilingual: EN/ES)
- **Automatic Generation**: Extracts information from code structure
- **Examples**: Request/response examples and error scenarios

## Quick Start

### Generate Swagger Docs for Auth Package Endpoints

```bash
/documentation swagger zibanu/django/auth
```

**Generates:**
- ✅ NumPy docstrings in ViewSet methods (English)
- ✅ `openapi/paths/auth.en.yaml` - English OpenAPI spec
- ✅ `openapi/paths/auth.es.yaml` - Spanish OpenAPI spec
- ✅ Schema components for serializers

### Generate Swagger Docs for Repository Package Endpoints

```bash
/documentation swagger zibanu/django/repository
```

**Generates:**
- ✅ NumPy docstrings in FileController and API views
- ✅ `openapi/paths/files.en.yaml` and `.es.yaml`
- ✅ Schema definitions for File, FileVersion, Category
- ✅ Error response documentation

---

## Generated File Structure

```
zibanu/django/{package}/
├── api/
│   ├── views.py          (← Updated with NumPy docstrings)
│   └── viewsets.py       (← Updated with NumPy docstrings)
├── openapi/
│   ├── openapi.en.yaml   (← Main spec English)
│   ├── openapi.es.yaml   (← Main spec Spanish)
│   ├── paths/
│   │   ├── {endpoint}.en.yaml
│   │   └── {endpoint}.es.yaml
│   └── components/
│       └── schemas/
│           ├── {Model}.en.yaml
│           └── {Model}.es.yaml
```

---

## Documentation Format

### Python Docstrings (in ViewSets)

```python
def list(self, request, *args, **kwargs):
    """List all files in the repository.
    
    Retrieve a paginated list of files with optional filtering,
    sorting, and search capabilities.
    
    Parameters
    ----------
    request : Request
        HTTP request with query parameters:
        - page: Page number (default: 1)
        - page_size: Items per page (default: 20)
        - search: Filter by name/description
        - ordering: Sort field (e.g., -created_at)
    *args : tuple
        Additional positional arguments
    **kwargs : dict
        Additional keyword arguments (e.g., pk for detail)
    
    Returns
    -------
    Response
        200: List of files with pagination
        401: Authentication required
        403: Permission denied
        400: Invalid parameters
    
    Examples
    --------
    List first page:
        GET /api/files/?page=1
    
    Search and filter:
        GET /api/files/?search=invoice&page_size=50
    
    Sort by creation date (descending):
        GET /api/files/?ordering=-created_at
    """
    return super().list(request, *args, **kwargs)
```

### OpenAPI YAML (Bilingual)

#### English Spec (openapi/paths/files.en.yaml)
```yaml
paths:
  /api/files/:
    get:
      operationId: files_list
      tags:
        - Files
      summary: List all files
      description: Retrieve a paginated list of files with optional filters
      parameters:
        - name: page
          in: query
          schema:
            type: integer
          description: Page number for pagination
          example: 1
        - name: search
          in: query
          schema:
            type: string
          description: Search by file name or description
      responses:
        '200':
          description: Successful list retrieval
          content:
            application/json:
              schema:
                type: object
                properties:
                  count:
                    type: integer
                  next:
                    type: string
                    nullable: true
                  previous:
                    type: string
                    nullable: true
                  results:
                    type: array
                    items:
                      $ref: '#/components/schemas/File'
        '401':
          description: Authentication required
        '403':
          description: Permission denied
```

#### Spanish Spec (openapi/paths/files.es.yaml)
```yaml
paths:
  /api/files/:
    get:
      operationId: files_list
      tags:
        - Archivos
      summary: Listar todos los archivos
      description: Obtener una lista paginada de archivos con filtros opcionales
      parameters:
        - name: page
          in: query
          schema:
            type: integer
          description: Número de página para paginación
          example: 1
        - name: search
          in: query
          schema:
            type: string
          description: Buscar por nombre o descripción del archivo
      responses:
        '200':
          description: Recuperación exitosa de lista
          # ... rest identical to English
```

---

## Supported Endpoint Methods

The agent documents all standard REST operations:

| Method | Operation | Example |
|--------|-----------|---------|
| **GET** | List/Retrieve | `/api/files/` or `/api/files/{id}/` |
| **POST** | Create | `/api/files/` with request body |
| **PUT** | Replace | `/api/files/{id}/` with full object |
| **PATCH** | Partial update | `/api/files/{id}/` with partial data |
| **DELETE** | Delete | `/api/files/{id}/` |

---

## Documentation Components

### Request Parameters

**Query Parameters**
```yaml
parameters:
  - name: page
    in: query
    description: Page number
    schema:
      type: integer
  - name: ordering
    in: query
    description: Sort field (e.g., -created_at)
    schema:
      type: string
```

**Path Parameters**
```yaml
parameters:
  - name: id
    in: path
    required: true
    description: Unique file identifier
    schema:
      type: integer
```

**Request Body**
```yaml
requestBody:
  required: true
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/FileCreate'
      example:
        name: "invoice.pdf"
        category: 1
```

### Responses

**Success Response**
```yaml
responses:
  '200':
    description: Successful operation
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/File'
        example:
          id: 123
          name: "invoice.pdf"
          uuid: "550e8400-e29b-41d4-a716-446655440000"
          created_at: "2024-01-15T10:30:00Z"
```

**Error Responses**
```yaml
responses:
  '400':
    description: Invalid request parameters
  '401':
    description: Authentication required
  '403':
    description: Permission denied
  '404':
    description: Resource not found
  '500':
    description: Server error
```

---

## Authentication Documentation

Each endpoint documents required authentication:

```yaml
security:
  - bearerAuth: []
  - cookieAuth: []

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT authentication token
    cookieAuth:
      type: apiKey
      in: cookie
      name: sessionid
```

---

## Common Use Cases

### 1. Document Auth Package Endpoints

```bash
/documentation swagger zibanu/django/auth
```

Generates documentation for:
- Login endpoint (POST)
- Logout endpoint (POST)
- Refresh token (POST)
- Change password (POST)
- List users (GET)
- User detail (GET)
- Create user (POST)
- Update user (PATCH)
- Delete user (DELETE)

### 2. Document Repository Package Endpoints

```bash
/documentation swagger zibanu/django/repository
```

Generates documentation for:
- List files (GET)
- File detail (GET)
- Upload file (POST)
- Update file (PATCH)
- Delete file (DELETE)
- Download file (GET with attachment)
- List categories (GET)
- Create category (POST)
- Get file versions (GET)

### 3. Document Single ViewSet

```bash
/documentation swagger zibanu/django/repository/api/viewsets.py
```

Focuses on specific ViewSet and documents all its methods.

---

## YAML File Organization

### Main OpenAPI Spec (openapi.en.yaml / openapi.es.yaml)

```yaml
openapi: 3.0.0
info:
  title: Zibanu Django API
  description: REST API for Zibanu Django platform
  version: 2.1.2-dev.0
servers:
  - url: https://api.example.com
    description: Production server
  - url: http://localhost:8000
    description: Development server
paths:
  $ref: ./paths/index.yaml
components:
  schemas:
    $ref: ./components/schemas/index.yaml
  securitySchemes:
    $ref: ./components/security.yaml
```

### Paths Index (paths/index.en.yaml)

```yaml
paths:
  /api/auth/login:
    $ref: ./auth.en.yaml#/paths/~1api~1auth~1login
  /api/auth/logout:
    $ref: ./auth.en.yaml#/paths/~1api~1auth~1logout
  /api/files:
    $ref: ./files.en.yaml#/paths/~1api~1files
  /api/files/{id}:
    $ref: ./files.en.yaml#/paths/~1api~1files~1{id}
```

---

## Quality Checklist

When documenting endpoints with Swagger scope:

- ✅ All HTTP methods documented (GET, POST, PUT, PATCH, DELETE)
- ✅ All query/path parameters described
- ✅ Request and response schemas defined
- ✅ All status codes documented (200, 201, 400, 401, 403, 404, 500)
- ✅ Authentication requirements specified
- ✅ Permission requirements noted
- ✅ Example requests and responses included
- ✅ Bilingual YAML files generated (EN/ES)
- ✅ Consistent terminology across files
- ✅ OpenAPI 3.0 spec valid and complete

---

## Integration with Django

### Settings Configuration

```python
# settings.py
INSTALLED_APPS = [
    # ... other apps
    'drf_spectacular',  # For Swagger/OpenAPI generation
]

SPECTACULAR_SETTINGS = {
    'SCHEMA_PATH_PREFIX': '/api/',
    'VERSION': '2.1.2-dev.0',
    'TITLE': 'Zibanu Django API',
    'DESCRIPTION': 'REST API for Zibanu Django platform',
}
```

### URL Configuration

```python
# urls.py
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView

urlpatterns = [
    # ... your endpoints
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema')),
]
```

---

## Viewing Generated Documentation

### 1. Local Swagger UI
```
http://localhost:8000/api/docs/
```

### 2. ReDoc Alternative
```
http://localhost:8000/api/redoc/
```

### 3. OpenAPI JSON
```
http://localhost:8000/api/schema/
```

---

## Examples by Package

### Auth Package Endpoints

```bash
/documentation swagger zibanu/django/auth
```

Endpoints documented:
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `POST /api/auth/refresh` - Token refresh
- `POST /api/auth/password/change` - Change password
- `GET /api/users/` - List users
- `GET /api/users/{id}/` - User detail
- `POST /api/users/` - Create user
- `PATCH /api/users/{id}/` - Update user
- `DELETE /api/users/{id}/` - Delete user

### Repository Package Endpoints

```bash
/documentation swagger zibanu/django/repository
```

Endpoints documented:
- `GET /api/files/` - List files
- `POST /api/files/` - Upload file
- `GET /api/files/{id}/` - File detail
- `GET /api/files/{id}/download/` - Download file
- `PATCH /api/files/{id}/` - Update file
- `DELETE /api/files/{id}/` - Delete file
- `GET /api/files/{id}/versions/` - File versions
- `GET /api/categories/` - List categories
- `POST /api/categories/` - Create category

---

## Tips & Best Practices

### 1. Document Permission Classes
```python
def create(self, request, *args, **kwargs):
    """Create a new file.
    
    Requires: FileUpload permission or admin role
    
    Parameters
    ----------
    request : Request
        Must include authentication token
    """
```

### 2. Include Rate Limiting Info
```yaml
x-rate-limit:
  calls: 1000
  period: 3600  # per hour
```

### 3. Add Custom Headers
```yaml
parameters:
  - name: X-Custom-Header
    in: header
    description: Custom header value
    schema:
      type: string
```

### 4. Document Filtering Options
```
GET /api/files/?category=2&created_after=2024-01-01&ordering=-created_at
```

---

## Troubleshooting

**Issue**: YAML syntax errors
**Solution**: Validate at https://www.yamllint.com/

**Issue**: Bilingual files not in sync
**Solution**: Agent automatically maintains parallel structure

**Issue**: Missing endpoint documentation
**Solution**: Ensure ViewSet has proper docstrings before running

---

¡Listo para documentar tus endpoints! 🚀