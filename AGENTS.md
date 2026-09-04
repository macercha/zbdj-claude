# Claude Code Agents for Zibanu Django

This directory contains custom agents and commands configured for this project.

**Last verified:** 2026-07-28

## Shared Skills

### python-django-drf
**Location:** `.claude/skills/python-django-drf/SKILL.md`

Domain knowledge shared across agents — Django ORM pitfalls, DRF
serializer/viewset/JWT patterns, Django-specific security/performance
checks. Currently loaded by **qa-review** at the start of every review. 
Any agent that reviews, writes, or fixes Django/DRF code (e.g. `/qa-code-fixer` 
consuming `Claude/QA/audits/<package>/fix_backlog.json`) should load it the same way —
add `Skill` to that agent's `tools:` list and invoke
`Skill(skill: "python-django-drf")` before analysis.

### zibanu-django-conventions
**Location:** `.claude/skills/zibanu-django-conventions/SKILL.md`

This project's own base-class conventions, decorators, and architectural boundaries 
on top of generic Django/DRF knowledge. Loaded together with **python-django-drf** 
by agents that review, write, or fix code touching Django models, views, serializers, 
viewsets, or DRF auth/JWT. This covers only what's specific to Zibanu Django; 
framework-generic ORM/DRF/security patterns live in **python-django-drf**.

## Available Agents and Commands

### Test Runner Command
**Purpose:** Execute unit tests on the remote server and generate test reports.

**How to invoke:**
```
/test-runner [target]
```

**Parameters:**
- `[target]` (optional): 
  - Omitted: runs all tests in `unit_tests/`
  - Package name (e.g., `repository`): runs all tests in that package
  - Module path (e.g., `unit_tests/repository/test_file_metadata.py`)
  - Django test label (e.g., `unit_tests.repository.test_file_metadata`)

**Examples:**
```bash
/test-runner
/test-runner repository
/test-runner unit_tests/repository/test_file_metadata.py
```

**Output:** Test reports in `Claude/Tester/<package>/`

**See also:** `.claude/commands/test-runner.md`

---

### Test Writer Command
**Purpose:** Generate unit test modules for Python source files or packages.

**How to invoke:**
```
/test-writer [target]
```

**Parameters:**
- `[target]` (required if not specified):
  - Source file (e.g., `zibanu/django/repository/lib/classes/file_metadata.py`)
  - Directory/package (e.g., `zibanu/django/repository/lib/classes`)
  - Omitted: asks which file or package to cover

**Package routing:**
- `zibanu/django/repository/...` → `unit_tests/repository/`
- `zibanu/django/auth/...` → `unit_tests/auth/`
- `zibanu/django/{gis,logging,mqtt,state_machine,template}/...` → `unit_tests/<package>/`
- `zibanu/django/` (base) → `unit_tests/zibanu/`

**Examples:**
```bash
/test-writer zibanu/django/repository/lib/controllers/file_controller.py
/test-writer zibanu/django/repository/lib/classes
/test-writer zibanu/django/auth/lib/backends.py
```

**See also:** `.claude/commands/test-writer.md`

---

### QA Review Command
**Purpose:** Execute code quality review on Python files or directories and generate QA reports.

**How to invoke:**
```
/qa-review <path> [depth]
```

**Parameters:**
- `<path>` (required): file or directory to review
- `[depth]` (optional): `quick`, `standard` (default), or `thorough`

**Examples:**
```bash
/qa-review zibanu/django/auth/models/user.py
/qa-review zibanu/django/repository thorough
/qa-review zibanu/django/gis quick
/qa-review zibanu/django thorough  # reviews base + all installed packages
```

**Output:** 
- `Claude/QA/reports/<package>/`
- `Claude/QA/metrics/<package>/`
- `Claude/QA/code-reviews/<package>/`

**See also:** `.claude/commands/qa-review.md`

---

### QA Auditor Command
**Purpose:** Audit the output of `/qa-review` — confirm every file under a path was 
actually reviewed, validate that each report is complete (not stale, not a stub), 
reconcile `Claude/QA/metrics/<package>/summary.json` and `Claude/QA/code-reviews/<package>/issues.md` 
against the individual reports, and emit a per-package `Claude/QA/audits/<package>/fix_backlog.json` 
that `/qa-code-fixer` can consume directly.

**How to invoke:**
```
/qa-audit <path> [--include-tests]
```

**Parameters:**
- `<path>` (required): file or directory whose QA coverage to audit
- `[--include-tests]` (optional): also require coverage for `test_*.py` files (excluded by default)

**Examples:**
```bash
/qa-audit zibanu/django/auth
/qa-audit zibanu/django/repository --include-tests
/qa-audit zibanu/django/auth/lib/backends.py
/qa-audit zibanu/django  # audits base + all installed packages independently
```

**Output:** `Claude/QA/audits/<package>/fix_backlog.json` per package

Does not review code and does not fix anything — see
`.claude/commands/qa-audit.md` for the full audit procedure and output schema.

**Package subdirectory routing (all pipeline stages use this same,
exhaustive, exclusive list — confirmed against the filesystem 2026-07-28):**

| Path | Package name |
|---|---|
| `zibanu/django/auth/` | `auth` |
| `zibanu/django/gis/` | `gis` |
| `zibanu/django/logging/` | `logging` |
| `zibanu/django/mqtt/` | `mqtt` |
| `zibanu/django/repository/` | `repository` |
| `zibanu/django/state_machine/` | `state_machine` |
| `zibanu/django/template/` | `template` |
| `zibanu/django` itself, or any base-package file directly under it (not inside one of the seven above) | `zibanu` |

Anything outside `zibanu/django/` entirely isn't in this recognized list,
and every pipeline stage should stop and say so rather than invent a
package subdirectory. A single invocation can span multiple *recognized*
packages (e.g. reviewing/auditing `zibanu/django` as a whole) — every stage
of this pipeline then processes each one **independently**, with its own
reports, backlog, verdict, and (for `/qa-code-audit`) commit gate. Nothing
is ever merged across packages.

---

### QA Code Fixer Command
**Purpose:** Apply fixes from a package's `Claude/QA/audits/<package>/fix_backlog.json` 
to source files, one issue at a time, each requiring explicit human approval before editing.

**Important:** This command never commits anything. It ends after applying (or 
skipping/rejecting) fixes. Committing only happens through `/qa-code-audit`, after 
independent re-verification. Use `/qa-code-audit` after fixing to verify and commit.

**How to invoke:**
```
/qa-code-fixer [scope]
```

**Parameters:**
- `[scope]` (optional):
  - Normal mode: file path, directory, package name (e.g., `auth`), or issue-id prefix (e.g., `AUTH-`)
  - `pending [scope]`: re-process issues with `status: pending` (previously rejected or unfixable)
  - Omitted: processes all `open` issues across every package's backlog

**Examples:**
```bash
/qa-code-fixer                    # All open issues in all packages
/qa-code-fixer zibanu/django/auth # Only auth package's open issues
/qa-code-fixer auth               # By package name
/qa-code-fixer AUTH-              # By issue-id prefix
/qa-code-fixer pending            # Re-process pending issues
/qa-code-fixer pending auth       # Pending issues in auth only
```

**Per-issue approval gate:**
For each issue, the command:
1. Reads the target file and confirms the defect is present
2. Drafts the fix (never applies yet)
3. Outputs the proposal and calls `AskUserQuestion`
4. Options: `Aplicar` (apply), `Rechazar` (reject), `Ajustar` (adjust/propose different fix)
5. If approved: applies, runs any related tests, updates `status` to `in progress`
6. If rejected/unfixable: updates `status` to `pending` for later review

**Output:** 
- Updates `Claude/QA/audits/<package>/fix_backlog.json` (status field only)
- Appends to `Claude/QA/audits/<package>/fix_log.json` (detailed history)

**See also:** `.claude/commands/qa-code-fixer.md`

---

### QA Code Audit Command
**Purpose:** Independently re-verify "in progress" fixes against the original issues, 
then offer to commit the verified fixes and close them in the backlog.

**How to invoke:**
```
/qa-code-audit [scope]
```

**Parameters:**
- `[scope]` (optional):
  - File path, directory, package name (e.g., `auth`), or issue-id prefix (e.g., `AUTH-`)
  - Omitted: audits all packages with `fix_backlog.json`

**Examples:**
```bash
/qa-code-audit
/qa-code-audit zibanu/django/auth
/qa-code-audit auth
/qa-code-audit AUTH-
```

**Workflow:**
1. Re-verifies each `in progress` issue using the **python-django-drf** skill's checklist
2. Downgrades issues that didn't fully resolve to `pending` or `not-resolved`
3. For each package with fully verified files (all issues `code-reviewed`):
   - Asks human approval before committing
   - Increments `__build__` in `__init__.py` (if package defines it)
   - Commits with detailed message including issue IDs
   - Updates `status` to `closed` in backlog

**Output:**
- Updates `Claude/QA/audits/<package>/fix_backlog.json` (status field)
- Appends to `Claude/QA/audits/<package>/code_audit_log.json`
- Creates git commits per package repo

**See also:** `.claude/commands/qa-code-audit.md`

---

## QA Pipeline Lifecycle

The complete QA workflow flows through these stages:

```
1. /qa-review <path> [depth]
   ↓ Generates: Claude/QA/reports/<package>/
   
2. /qa-audit <path> [--include-tests]
   ↓ Generates: Claude/QA/audits/<package>/fix_backlog.json
   
3. /qa-code-fixer [scope]
   ↓ Applies fixes (requires per-issue approval)
   ↓ Updates status: open → in progress (or pending)
   
4. /qa-code-audit [scope]
   ↓ Re-verifies fixes independently
   ↓ If fully verified: offers commit
   ↓ Updates status: in progress → closed (after commit)
```

### Issue Status Lifecycle

Each package's `Claude/QA/audits/<package>/fix_backlog.json` carries one
`status` field per issue that travels through the whole pipeline:

| Status | Set by | Meaning |
|---|---|---|
| `planning` | `qa-audit` (first entry) | Logged, not yet cross-validated |
| `open` | `qa-audit` (after validation) | Cross-validated, ready for fixing |
| `in progress` | `/qa-code-fixer` (after approval) | Fix approved & applied, awaiting re-verification |
| `pending` | `/qa-code-fixer` or `/qa-code-audit` | Rejected by human, unfixable, or re-verification failed — needs another look |
| `closed` | `/qa-code-audit` (after commit) | Independently verified and committed |

**Key point:** An issue only closes after `/qa-code-audit` independently 
re-verifies it against the original defect and the fix lands in a git commit. 
No auto-closing — every stage validates what the previous stage did.

### Documentation Command
**Purpose:** Generate comprehensive documentation — docstrings (NumPy format), bilingual READMEs (EN/ES), 
OpenAPI specs, and agent-oriented knowledge bases.

**How to invoke:**
```
/documentation <scope> <path>
```

**Scopes available:**

#### `file` — Document a single Python file
Generate NumPy-style docstrings for module, classes, and functions.

**Example:**
```bash
/documentation file zibanu/django/rest_framework/decorators.py
```

**Generates:**
- Module docstring
- Class/function docstrings (NumPy format)
- Inline comments for complex logic

#### `directory` — Document all files in a package
Generate consistent NumPy-style docstrings across all Python files.

**Example:**
```bash
/documentation directory zibanu/django/auth
```

**Generates:**
- Package-level documentation
- Docstrings for all public classes and functions
- Cross-module consistency

#### `readme` — Generate bilingual README files
Create comprehensive `README.md` (English) and `README.es.md` (Spanish).

**Example:**
```bash
/documentation readme zibanu/django/repository
```

**Generates:**
- `README.md` (English) with 12-section structure
- `README.es.md` (Spanish) — parallel structure
- Installation, usage, API reference, examples

#### `swagger` — Generate OpenAPI/Swagger documentation
Create bilingual (EN/ES) OpenAPI 3.0 YAML specs with NumPy docstrings on ViewSet methods.

**Example:**
```bash
/documentation swagger zibanu/django/auth
```

**Generates:**
- NumPy docstrings in ViewSet methods
- `openapi/paths/{endpoint}.{en,es}.yaml`
- `openapi/components/schemas/` — component definitions
- Security definitions and request/response examples

#### `kb` — Generate agent-oriented knowledge base
Create a short, dense functional summary (60–200 lines) for coding agents to consume 
before depending on the package from new code.

**Example:**
```bash
/documentation kb zibanu/django/auth
```

**Generates:**
- `Claude/KB/<package>/knowledge_base.md` — single file, always fully regenerated
- Fixed sections: Purpose, When to depend on it, Install & wire-up, Public API, 
  Models, Settings, Minimal usage example, Conventions & gotchas, See also
- Dense and scannable for LLM readers (no narrative padding)

**See also:** `.claude/commands/documentation.md`

---

## Documentable Packages

### Core Base Package (Always Installed)
- `zibanu/django/api/` — REST API services (languages, timezones)
- `zibanu/django/db/` — Database models and managers
- `zibanu/django/lib/` — Shared utilities
- `zibanu/django/rest_framework/` — DRF extensions
- `zibanu/django/` itself — base package utilities

### Optional Packages (Install Separately)
- `zibanu/django/auth/` — Authentication and JWT
- `zibanu/django/gis/` — Geographic information system
- `zibanu/django/logging/` — Logging storage
- `zibanu/django/mqtt/` — MQTT integration
- `zibanu/django/repository/` — File management and versioning
- `zibanu/django/state_machine/` — State machine models
- `zibanu/django/template/` — Template utilities

## Quick Start Examples

### Document a single file
```bash
/documentation file zibanu/django/db/models/model.py
```

### Document entire package
```bash
/documentation directory zibanu/django/auth
```

### Generate bilingual README
```bash
/documentation readme zibanu/django/repository
```

### Generate API documentation
```bash
/documentation swagger zibanu/django/auth
```

### Generate knowledge base for agents
```bash
/documentation kb zibanu/django/auth
```

### Full package documentation (recommended order)
```bash
/documentation directory zibanu/django/gis
/documentation readme zibanu/django/gis
/documentation swagger zibanu/django/gis
/documentation kb zibanu/django/gis
```

## Documentation Format

The agent uses **NumPy-style docstrings** (English only):

```python
"""Module description.

Longer explanation with usage examples and key information.
"""

def function(param1: str, param2: int) -> bool:
    """Brief function description.
    
    Detailed explanation of what it does and why.
    
    Parameters
    ----------
    param1 : str
        Description of param1
    param2 : int
        Description of param2
        
    Returns
    -------
    bool
        Description of return value
        
    Examples
    --------
    >>> result = function("test", 42)
    >>> result
    True
    """
```

### README Format
- **Bilingual** (EN/ES) — `README.md` + `README.es.md`
- **12-section structure** — overview, installation, usage, API, examples, etc.
- Professional technical tone
- Code examples and tables

### OpenAPI Format
- **OpenAPI 3.0** specification
- **Bilingual** (EN/ES) — separate YAML files
- Request/response examples
- Complete HTTP method documentation

### Knowledge Base Format
- **English only**, single file: `Claude/KB/<package>/knowledge_base.md`
- **60–200 line hard budget** — dense and scannable
- Fixed section order, written for LLM readers
- Fully regenerated from source on every run (never hand-patched)

## Tips and Best Practices

### Documentation Command
1. **Be Specific**: Always provide full path from project root
2. **One Scope at a Time**: Document one scope per invocation for best results
3. **Regenerate KB Often**: `/documentation kb` is cheap to regenerate and always fully 
   overwritten, so it's safe to run after any public API changes
4. **Review Output**: Check generated documentation for accuracy before committing
5. **Complete Package Documentation**: Follow the recommended order: file → directory → readme → swagger → kb

### QA Pipeline
1. **Run in Order**: Always follow the pipeline sequence (review → audit → code-fixer → code-audit)
2. **No Silent Fixes**: Every issue in `/qa-code-fixer` requires explicit human approval per issue
3. **Independent Verification**: `/qa-code-audit` always re-verifies fixes before committing
4. **Package Isolation**: Each package is audited and committed independently, never merged
5. **Backlog as Source of Truth**: Use `fix_backlog.json` as the authoritative state throughout the pipeline

### Testing Command
1. **Run Remote**: Tests always execute on the remote server at `zibanu-io`
2. **No Local Environment**: There is no Django environment on this Mac
3. **Use Package Names**: Reference tests by package name for clarity
4. **Review Test Reports**: Check `Claude/Tester/<package>/` for detailed results

## Common Tasks

| Task | Command |
|---|---|
| **Test** |  |
| Run all unit tests | `/test-runner` |
| Run tests in a package | `/test-runner repository` |
| Generate tests for a file | `/test-writer zibanu/django/auth/models/user.py` |
| **Code Quality** |  |
| Review Python code | `/qa-review zibanu/django/auth thorough` |
| Audit QA coverage | `/qa-audit zibanu/django/auth` |
| Apply fixes to code | `/qa-code-fixer auth` |
| Verify and commit fixes | `/qa-code-audit auth` |
| **Documentation** |  |
| Document single file | `/documentation file zibanu/django/db/models/model.py` |
| Document entire package | `/documentation directory zibanu/django/auth` |
| Generate bilingual README | `/documentation readme zibanu/django/repository` |
| Generate OpenAPI specs | `/documentation swagger zibanu/django/auth` |
| Update knowledge base | `/documentation kb zibanu/django/auth` |

## Integration with Project

These agents and commands work with:
- **CLAUDE.md** — project architecture documentation
- **Shared Skills** — `python-django-drf` and `zibanu-django-conventions`
- **Modular package structure** — each package is independently routed and processed
- **NumPy-style docstrings** (English) + bilingual documentation (EN/ES)
- **Remote server execution** — tests and large operations run on `zibanu-io`
- **Git repository structure** — nested per-package repos, careful handling of commits

---

**For detailed command documentation**, see:
- `.claude/commands/test-runner.md`
- `.claude/commands/test-writer.md`
- `.claude/commands/qa-review.md`
- `.claude/commands/qa-audit.md`
- `.claude/commands/qa-code-fixer.md`
- `.claude/commands/qa-code-audit.md`
- `.claude/commands/documentation.md`