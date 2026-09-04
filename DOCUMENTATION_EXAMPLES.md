# Documentation Agent - Usage Examples

## Quick Reference

The Documentation Agent specializes in Python documentation. Invoke it with a specific scope and path.

## Examples by Package

### 1. Document the Auth Package (Full Directory)

**Goal:** Add comprehensive docstrings to all auth modules

**Command:**
```
/documentation directory zibanu/django/auth
```

**What it does:**
- Analyzes all Python files in `zibanu/django/auth/`
- Adds module-level docstrings
- Documents all public classes and functions
- Adds docstrings for REST API endpoints
- Documents authentication flows
- Creates consistent documentation style

**Expected output:**
- Updated Python files with docstrings
- Module docstrings explaining the auth package structure
- Documented API endpoints (login, logout, refresh, etc.)
- Field descriptions in serializers

---

### 2. Generate Auth Package README

**Goal:** Create comprehensive README.md for the auth package

**Command:**
```
/documentation readme zibanu/django/auth
```

**What it includes:**
- Package overview (JWT authentication, user management)
- Installation instructions
- Configuration requirements
- Quick start examples
- API reference (12 endpoints)
- Usage examples for common scenarios
- Testing instructions
- Configuration options

**Creates file:** `zibanu/django/auth/README.md` (or updates existing)

---

### 3. Document Repository Package

**Goal:** Full documentation of the file repository package

**Command:**
```
/documentation directory zibanu/django/repository
```

**Will document:**
- File versioning system
- File metadata extraction
- Thumbnail generation
- Multi-level directory structures
- File categorization logic
- Controllers (FileController, etc.)
- Models (File, FileVersion, Category)
- Services and API endpoints

---

### 4. Generate Repository Package README

**Goal:** Create detailed README for repository package

**Command:**
```
/documentation readme zibanu/django/repository
```

**Includes:**
- File management features overview
- Installation and configuration
- Settings documentation (`ZB_REPOSITORY_*`)
- Usage examples for common operations
- API reference for main controllers
- Versioning system explanation
- Metadata extraction capabilities
- Thumbnail generation setup

**Creates file:** `zibanu/django/repository/README.md`

---

### 5. Document REST Framework Extensions

**Goal:** Document all DRF extensions in the base package

**Command:**
```
/documentation directory zibanu/django/rest_framework
```

**Will document:**
- ViewSet extensions and customizations
- Serializer base classes
- Decorator usage (@permission_required)
- Custom fields (HybridImageField, CurrentUserDefault)
- Exception handling classes
- Pagination and filtering utilities

---

### 6. Document a Single File

**Goal:** Add detailed docstrings to one specific file

**Command:**
```
/documentation file zibanu/django/rest_framework/viewsets.py
```

**Will produce:**
- Module docstring explaining the file's purpose
- Class docstrings for ModelViewSet and ViewSet
- Method docstrings for list, retrieve, create, update, destroy
- Inline comments for complex logic

---

### 7. Document Database Models Package

**Goal:** Document custom model and manager classes

**Command:**
```
/documentation directory zibanu/django/db
```

**Will document:**
- Base Model class (use_db attribute, set() method)
- DatedModel (auto timestamps)
- Custom Manager (get_by_pk, get_queryset)
- Database operations (Oracle backend)

---

### 8. Generate README for Template Package

**Goal:** Document template utilities and tags

**Command:**
```
/documentation readme zibanu/django/template
```

**Includes:**
- Available template tags (static_uri, string_concat, sum_dict, subtotal_dict)
- Context processors (full_static_uri, site)
- Usage examples for each tag
- Integration guide for Django templates

---

### 9. Document GIS Package

**Goal:** Document geographic data models

**Command:**
```
/documentation directory zibanu/django/gis
```

**Will document:**
- Geometry models
- Layer geometry models
- Coordinate system support
- Geographic query examples

---

### 10. Generate README for Logging Package

**Goal:** Create documentation for logging package

**Command:**
```
/documentation readme zibanu/django/logging
```

**Includes:**
- Logging model structure
- Admin interface features
- Configuration options
- Usage examples
- Querying log data

---

## Sequential Documentation Task

To fully document all packages (recommended order):

### Phase 1: Core Base Package
```
/documentation directory zibanu/django/db
/documentation directory zibanu/django/rest_framework
/documentation directory zibanu/django/api
/documentation readme zibanu/django
```

### Phase 2: Optional Packages
```
/documentation directory zibanu/django/auth
/documentation readme zibanu/django/auth

/documentation directory zibanu/django/repository
/documentation readme zibanu/django/repository

/documentation directory zibanu/django/logging
/documentation readme zibanu/django/logging

/documentation directory zibanu/django/template
/documentation readme zibanu/django/template

/documentation directory zibanu/django/gis
/documentation readme zibanu/django/gis
```

---

## Tips for Best Results

### 1. Provide Context
When invoking the documentation agent, you can add context:

```
/documentation directory zibanu/django/auth

Context: This package provides JWT-based authentication using SimpleJWT.
It handles user login, logout, password changes, and permission management.
It exposes 12 REST API endpoints for a complete auth lifecycle.
```

### 2. Ask for Specific Documentation Style
```
/documentation readme zibanu/django/repository

Style: Include detailed configuration examples for all ZB_REPOSITORY_* settings.
Add code examples for common operations like file upload, versioning, and thumbnail generation.
```

### 3. Request Specific Sections
```
/documentation directory zibanu/django/rest_framework

Focus: Document the permission_required decorator thoroughly with examples
of how to use it in views. Include type hints documentation.
```

---

## Expected Output

After running documentation commands, you should have:

### For File/Directory Scope:
- ✅ Module-level docstrings
- ✅ Class docstrings with attributes
- ✅ Function/method docstrings with Args, Returns, Raises, Examples
- ✅ Inline comments for complex logic
- ✅ Type hints in docstrings

### For README Scope:
- ✅ Overview section
- ✅ Installation instructions
- ✅ Quick start example
- ✅ Features list
- ✅ Detailed usage guide
- ✅ API reference
- ✅ Configuration options
- ✅ Testing information
- ✅ License/copyright

---

## Troubleshooting

### "Agent doesn't understand the scope"
Make sure you specify: `/documentation <scope> <path>`
- `scope`: file, directory, or readme
- `path`: full path from project root

### "Documentation seems incomplete"
Ask the agent to focus on specific aspects:
```
/documentation directory zibanu/django/rest_framework
Please pay special attention to the permission_required decorator and add examples.
```

### "Need to update existing documentation"
The agent can update existing docstrings while preserving your code:
```
/documentation file zibanu/django/db/models/model.py
Update existing docstrings but keep the implementation unchanged.
```

---

## Integration with CLAUDE.md

The Documentation Agent works together with your CLAUDE.md file:
- CLAUDE.md provides architecture overview
- Documentation Agent provides detailed code documentation
- README.md files provide package-specific guides
- Docstrings provide inline implementation details

Together they create comprehensive documentation at all levels!