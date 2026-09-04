# Claude Code Configuration for Zibanu Django

This directory contains configuration files for Claude Code agents and settings for the Zibanu Django project.

## Contents

### 1. **AGENTS.md** — Agent Reference Guide
Complete guide to available custom agents and how to invoke them.
- Documentation Agent overview
- Usage patterns and scopes
- Quick reference for common tasks
- Integration tips

**Start here** if you want to use custom agents.

### 2. **DOCUMENTATION_EXAMPLES.md** — Practical Examples
Real-world examples showing how to document different packages.
- Examples for each package (auth, repository, logging, etc.)
- Sequential documentation workflow
- Tips for best results
- Troubleshooting guide

**Use this** for specific documentation tasks.

### 3. **SWAGGER_DOCUMENTATION.md** — OpenAPI/Swagger Guide
Complete guide for generating Swagger/OpenAPI documentation for REST endpoints.
- How to document endpoints
- Bilingual YAML specification generation
- Format and structure guidelines
- Integration with Django/DRF

**Use this** for API endpoint documentation.

### 3. **agents/** — Agent Definitions
Directory containing custom agent specifications (Markdown files with frontmatter).

**Currently available:**
- `documentation.md` — Python documentation specialist agent (this README's
  focus — see below)
- `qa-review.md`, `qa-audit.md`, `qa-code-audit.md` — the QA review /
  fix pipeline (`/qa-review` → `/qa-audit` → `/qa-code-fixer` →
  `/qa-code-audit`). `qa-code-fixer` has no separate agent file — it runs
  directly in the primary session (see `AGENTS.md` for the full pipeline and
  its `Claude/QA/` output locations).

### 4. **settings.local.json** — Local Settings
Claude Code settings specific to this repository (if configured).

---

## Quick Start

### To Document a Python Package

1. **File documentation** (single Python file):
   ```
   /documentation file path/to/file.py
   ```

2. **Directory documentation** (entire package):
   ```
   /documentation directory path/to/package
   ```

3. **Generate README.md** (comprehensive package guide):
   ```
   /documentation readme path/to/package
   ```

### Example: Document the Auth Package

```bash
# Step 1: Document all Python files in auth
/documentation directory zibanu/django/auth

# Step 2: Generate comprehensive README
/documentation readme zibanu/django/auth
```

---

## Available Custom Agents

### Documentation Agent
**Specialization:** Python code documentation, docstrings, README generation, and API documentation

**Capabilities:**
- Document single files with NumPy-style docstrings (English)
- Document entire packages/directories with consistent style
- Generate comprehensive bilingual README files (EN/ES)
- Generate OpenAPI/Swagger documentation for REST endpoints (EN/ES YAML)
- Generate a short, agent-oriented knowledge base per package (`Claude/KB/<package>/knowledge_base.md`)
- Add inline comments for complex logic
- Use NumPy-style docstring format exclusively

**Supported Scopes:**
- `file` — Single Python file (NumPy docstrings, English)
- `directory` — Package or directory (NumPy docstrings, English)
- `readme` — Generate bilingual README files (EN/ES)
- `swagger` — Generate OpenAPI specification (NumPy docstrings + YAML specs)
- `kb` — Generate `Claude/KB/<package>/knowledge_base.md`, a 60–200 line summary for a coding agent to read before depending on the package

**Invoke with:**
```
/documentation <scope> <path>
```

**Examples:**
```
/documentation readme zibanu/django/auth
/documentation swagger zibanu/django/repository
/documentation directory zibanu/django/db
/documentation kb zibanu/django/auth
```

---

## Documentation Strategy for Zibanu Django

This project uses a **multi-level documentation approach**:

### Level 1: Architecture (CLAUDE.md)
- Project structure overview
- Package distribution model
- Development workflow
- Common commands

📍 Located in: `/CLAUDE.md`

### Level 2: Package Documentation (README.md)
- Package-specific guides
- Installation and configuration
- Usage examples and API reference
- Testing and contribution guidelines

📍 Located in: `zibanu/django/<package>/README.md`

### Level 3: Code Documentation (Docstrings)
- Module-level descriptions
- Class and method documentation
- Function signatures with type hints
- Usage examples in docstrings

📍 Located in: Source code files

### Level 4: Inline Comments
- Complex algorithm explanation
- Non-obvious design decisions
- Important constraints or gotchas

📍 Located in: Source code

---

## Packages to Document

### Core Base Package (Always Installed)
- ✅ `zibanu/django/` — Main utilities
- ✅ `zibanu/django/rest_framework/` — DRF extensions
- ✅ `zibanu/django/db/` — Database models and managers
- ✅ `zibanu/django/api/` — REST API services

### Optional Packages (Install Separately)
- ⬜ `zibanu/django/auth/` — Authentication/JWT
- ⬜ `zibanu/django/repository/` — File management
- ⬜ `zibanu/django/logging/` — Logging storage
- ⬜ `zibanu/django/template/` — Template extensions
- ⬜ `zibanu/django/gis/` — Geographic data

**Legend:** ✅ = Has CLAUDE.md reference | ⬜ = Needs documentation

---

## Documentation Workflow

### Recommended Order

1. **Read CLAUDE.md** — Understand overall architecture
2. **For each package:**
   - Run `documentation directory` to add docstrings
   - Run `documentation readme` to create package README
   - Review generated documentation for accuracy

3. **Verify completeness:**
   - All public APIs documented
   - Examples in docstrings work correctly
   - README has installation and usage instructions

---

## Integration with Project

### How These Files Work Together

```
CLAUDE.md
  ↓
  Provides architecture overview and lists packages
  ↓
zibanu/django/<package>/README.md
  ↓
  Provides package-specific guide
  ↓
Source code docstrings
  ↓
  Provides implementation details
```

### Agent Workflow

```
User invokes /documentation <scope> <path>
  ↓
Documentation Agent analyzes code
  ↓
Generates/updates docstrings
  ↓
Creates/updates README.md
  ↓
Reports completion and coverage
```

---

## Key Files Reference

| File/Folder | Purpose |
|------------|---------|
| `CLAUDE.md` | Project architecture and development guide |
| `AGENTS.md` | Custom agents reference |
| `DOCUMENTATION_EXAMPLES.md` | Practical examples for documentation tasks |
| `.claude/agents/documentation.md` | Documentation Agent definition |
| `.claude/README.md` | This file |

---

## Tips & Best Practices

### When Using the Documentation Agent

1. **Be Specific:** Always provide full paths
   ```bash
   # Good
   /documentation directory zibanu/django/auth
   
   # Avoid
   /documentation directory auth
   ```

2. **One Scope at a Time:** Document file → directory → README sequentially

3. **Verify Output:** Always review generated docstrings for accuracy

4. **Add Context:** Include package purpose if invoking the agent programmatically
   ```python
   Agent(
       prompt="""Document zibanu/django/repository package.
       Purpose: File versioning, storage, and metadata extraction.
       Focus on explaining FileController and versioning models."""
   )
   ```

5. **Check Existing Docs:** Reference any existing README.md when generating new ones

---

## Common Commands

```bash
# Full documentation for auth package (code + bilingual README + Swagger)
/documentation directory zibanu/django/auth
/documentation readme zibanu/django/auth
/documentation swagger zibanu/django/auth

# Full documentation for repository package
/documentation directory zibanu/django/repository
/documentation readme zibanu/django/repository
/documentation swagger zibanu/django/repository

# Document REST framework extensions
/documentation directory zibanu/django/rest_framework

# Document database utilities
/documentation directory zibanu/django/db

# Generate OpenAPI/Swagger specs for endpoints
/documentation swagger zibanu/django/auth
/documentation swagger zibanu/django/repository

# Document specific file
/documentation file zibanu/django/rest_framework/decorators.py
```

---

## Questions?

- **How do I use agents?** → See `AGENTS.md`
- **What should I document?** → See `DOCUMENTATION_EXAMPLES.md`
- **What's the project structure?** → See `../CLAUDE.md`
- **How do I run tests?** → See `../CLAUDE.md` → Common Development Tasks
- **How are packages distributed?** → See `../CLAUDE.md` → Architecture & Code Organization

---

## Project Information

- **Project:** Zibanu Django
- **Version:** 2.1.2-dev.0
- **Developer:** CQ Inversiones SAS
- **Documentation Tools:** Claude Code Agents
- **Configuration Location:** `./.claude/`
---

## Archivos de estado (añadidos después de la redacción original)

Dos archivos de este directorio **no son documentación**: son estado vivo que los agentes
leen y escriben en cada corrida.

### `project.config.json`

Contrato de configuración que consumen **todos** los agentes globales del toolkit
(`~/.claude/agents/`). Declara la topología (`multi`, 8 subproyectos con su `id`,
`sourceRoot` y `stack`), las rutas de salida de cada familia de artefactos
(`qa.paths.*`, `tests.paths.*`, `pathTemplate.*`), el entorno de ejecución —**remoto por
SSH: este proyecto no tiene intérprete local**— y la asociación con YouTrack.

Los agentes **fallan en vez de asumir rutas por defecto** cuando les falta una clave. Es
deliberado: producir artefactos en el sitio equivocado es peor que no producirlos.

### `youtrack_sync.json`

Ledger de sincronización: el mapeo entre cada issue local del `fix_backlog.json` y su
incidencia en YouTrack, con el hash del contenido publicado y el estado con que se publicó.

**Es caché, no fuente de verdad.** La verdad es el tablero; esto existe para no tener que
preguntársela entera en cada corrida y para saber si el *contenido* cambió. Si se pierde,
el mapeo se reconstruye paginando el proyecto y leyendo el prefijo `[<id local>]` del
resumen de cada incidencia — cuesta una pasada de adopción, no los datos.

Se versiona precisamente por eso: **evita duplicados**. Si viviera solo en una máquina, otro
equipo republicaría el backlog entero, y el MCP de YouTrack **no tiene borrado ni upsert**,
así que cada duplicado sería permanente.

## Qué no se versiona

`settings.local.json` — fija permisos con rutas absolutas de esta máquina, así que
sincronizarlo haría que un path de macOS pisara el de Windows. Misma regla que en el
toolkit global.
