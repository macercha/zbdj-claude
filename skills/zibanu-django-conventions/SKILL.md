---
name: zibanu-django-conventions
description: This project's (Zibanu Django) own base-class conventions, decorators, and architectural boundaries — on top of generic Django/DRF knowledge. Use alongside the global `python-django-drf` skill when reviewing, writing, fixing, verifying, or documenting any .py file in this codebase that touches Django models, views, serializers, viewsets, or DRF auth/JWT. This skill covers only what's specific to Zibanu Django; framework-generic ORM/DRF/security patterns live in `python-django-drf`.
---

# Zibanu Django Project Conventions

Project-specific knowledge layered on top of the global `python-django-drf`
skill. Load both together — this skill assumes the generic Django/DRF
checklists from that skill already apply, and only adds what's unique to
this codebase's own base classes, decorators, and package boundaries.

## How to use this skill

1. Load `python-django-drf` first (or alongside) for the generic checklists,
   then this skill for anything specific to `zibanu.django.*`.
2. Treat §1 below ("don't flag these as bugs") as intentional project design
   — deviations from these patterns are findings, not the patterns themselves.
3. Cite specific line/symbol locations, same as the global skill.

---

## 1. Zibanu Django project conventions (don't flag these as bugs)

From this project's own architecture — treat deviations from these as
findings, but the patterns themselves are intentional:

- Models inherit `zibanu.django.db.Model` (adds `use_db` for multi-DB
  routing, `.set()` for bulk field updates) or `DatedModel` (adds
  `created_at`/`modified_at` automatically — don't flag a model for
  "manually" tracking these fields if it doesn't use `DatedModel`, but do
  suggest switching to it).
- Custom `Manager` (`get_by_pk()` helper) is included automatically via the
  `Model` base — a model manually reimplementing "get or None" logic that
  `get_by_pk()` already provides is a suggestion-level dedup opportunity.
- Optional packages (`auth`, `repository`, `logging`, `template`, `gis`,
  `mqtt`, `state_machine`) are independently installable, each its own
  nested git repo — core `zibanu/django/` package code should have **no
  dependency** on an optional package. A base-package file importing from
  `zibanu.django.auth` (or similar) breaks modularity — flag as important.
- Settings live in `mainApp/components/*.py`, wired via `django-split-settings`
  in `mainApp/settings.py`. New settings belong in a new component file, not
  bolted onto an unrelated existing one.
- Core `zibanu/django/` packages should not be modified directly by
  project-specific code — extension happens via inheritance or a separate
  app. If a review is of code *outside* `zibanu/django/`, this doesn't apply;
  if it's *inside* core packages, unusual one-off special-casing is a smell.

---

## 2. ORM: `Model.set()` for bulk mutation

This project's base `zibanu.django.db.Model` provides a `set()` method for
bulk field updates — prefer it over manually looping `setattr()` calls when
reviewing custom model code, per §1.

N+1 loop patterns (see the global skill's §1) are especially common in this
codebase's `lib/managers/` recursive tree/menu-building code.

---

## 3. DRF: `permission_required` decorator and `ModelViewSet` conventions

### `permission_required` / `permission_classes` must be present on every action
`zibanu.django.rest_framework` provides a `permission_required` decorator
that validates permissions without raising (returns a boolean/response
instead of throwing). Every custom `list`/`retrieve`/`create`/`update`/
`destroy` override in a `ModelViewSet` or service class should carry either
this decorator or DRF's standard `permission_classes` — an override that
drops the decorator present on sibling methods is a silent authorization
gap, not a stylistic inconsistency.

### `ModelViewSet` conventions in this project
- Extends DRF's viewset with improved queryset handling.
- Returns **204** (not 200 with an empty list) when a list result is empty —
  this is intentional project behavior, don't flag it as a bug.
- `CurrentUserDefault` (project's own field, compatible with SimpleJWT's
  `TokenUser`) should be used for "current user" defaults in serializers
  instead of DRF's stock `CurrentUserDefault`, which breaks against
  `TokenUser` (no real DB-backed `request.user.pk` guarantee the same way).

### JWT / SimpleJWT specifics
- `djangorestframework-simplejwt` + `token_blacklist` app are installed with
  the auth package.
- Watch for inconsistent `get_user`/user-lookup helpers across sibling
  service modules (e.g. `api/services/*.py`) — different modules importing
  "the current user" from different places is a sign of drift, not
  intentional design; flag it as an important consistency issue.

---

## 4. Security: `HybridImageField` for uploads

This project's `HybridImageField` (image validation with size/format checks)
should be used for user-uploaded images instead of a bare
`ImageField`/`FileField` with no validation.

Also check `mainApp/components/cors.py` allows only intended origins;
`CORS_ALLOW_ALL_ORIGINS = True` in anything but local dev is a finding.

---

## 5. Testing conventions (this project)

This section is the source of truth for how tests are written and run here. The
global `test-writer` / `test-runner` agents carry only generic methodology and
defer to this section for the project specifics.

### Runner: `NoDbTestRunner` → local DB, no alternate test database

`TEST_RUNNER = "mainApp.test_runner.NoDbTestRunner"` (`mainApp/env/dev/base.py`;
verified effective) is a `DiscoverRunner` whose `setup_databases` /
`teardown_databases` are **no-ops** — it **never creates or drops a separate
`test_*` database**. Tests run against the **local/dev database the settings
already point at** (currently MySQL/InnoDB `macercha_zibanu`). This runner is a
**required precondition** for DB tests here: it's precisely what lets them use
the local database instead of Django spinning up an alternate test DB. (You'll
see `Skipping setup of unused database(s): default.` — that line is normal.)

### Which base class: `SimpleTestCase` (DB-free) vs `TestCase` (DB)

Pick the base class by whether the unit touches the ORM/DB:

- **DB-free unit → `django.test.SimpleTestCase`.** It forbids DB queries
  outright — turning "needs no DB" from a hope into an enforced property. Most
  of what's worth testing here (controllers' pure logic, file utilities,
  converters, serializer validation, enums) needs no database; default to this
  and it runs cleanly under `NoDbTestRunner`.
- **DB-accessing unit → `django.test.TestCase`** (NOT `SimpleTestCase`). Under
  `NoDbTestRunner`, a `TestCase` runs against the **local DB** and relies on its
  own **per-test transaction rollback** for isolation — so it exercises real
  ORM/queries **without creating an alternate database**. This is the intended,
  sanctioned path for tests that need the DB in this project. **Say so in the
  report** (module uses `TestCase` → exercises the live local DB).
  - **Create the rows the test needs inside the test** (`setUp` / the body); the
    rollback cleans them up. **Never depend on pre-existing / hardcoded rows** —
    this project already lost a suite that hardcoded `pk=216` and broke when the
    row went away.
  - **Never use `TransactionTestCase`** here: it does **not** roll back (it
    truncates tables between tests), which against the live local DB would wipe
    real data. Use `TestCase` (transactional rollback) only.

### Layout & naming

```
unit_tests/
├── base/test_base.py     BaseTestCase, USER_DATA, FILES_DIR, fixture_path()
├── data_test.py          fixtures/factories
├── files/                shared binary fixtures
└── <package>/test_*.py   one folder per package (repository, auth, gis, …)
```

- **File name must match `test*.py`** (Django's discovery pattern); a file like
  `foo_test.py` is silently invisible to `manage.py test`. Name modules after the
  unit under test (`test_file_metadata.py`).
- **Module docstring must open with** ``Unit tests for :class:`<dotted.path.Name>`.``
  — the `test-runner` agent parses that exact line for its `Unit under test`
  field. Keep the `:class:` role even for a plain function (the dotted path just
  ends in the function name).
- Reach shared fixtures via the helper, never by recomputing paths:
  `from ..base.test_base import fixture_path`.
- Notable fixtures in `unit_tests/files/` (via `fixture_path("<name>")`): an MP4
  with **no audio** (exercises `in_audio is None`), a rich-EXIF JPEG, PDFs,
  DOCX/XLSX (office extraction), and a legacy `.doc` (unsupported-type negative
  case). Prefer a real fixture over a synthetic one; build a `SimpleUploadedFile`
  over crafted bytes only for cases no real file covers.

### Coverage as a number

A file with **zero** corresponding test coverage (no `test_<name>.py` and no
factory in `data_test.py` referencing it) is a real, quantifiable gap — report
it as `0%`, not a vague "could use more tests."

### Endpoint tests (this project's REST surface)

The generic technique is in `python-django-drf` §5.1; this section is what is
true **here**. Endpoint tests are the only thing that exercises
`api/services/*.py`, which is where this codebase's measured coverage gap lives.

**Name them `test_api_<subject>.py`**, in the same `unit_tests/<pkg>/` folder as
the unit modules, so the runner's report separates the two kinds at a glance.

**Base class: `BaseTestCase`**, from `unit_tests/zb_base/test_base.py` — not
`APITestCase` directly. It is a `django.test.TestCase` (so the per-test rollback
of §5 applies) and carries the three helpers this project's auth needs:

```python
from ..zb_base.test_base import BaseTestCase, USER_DATA

class MenuListTest(BaseTestCase):
    def test_authenticated_caller_gets_the_menu(self):
        token = self.do_login()                     # creates its user, returns a sliding token
        response = self.request("/auth/menu/", token=token, data={"app_id": appid})
        self.assertEqual(response.status_code, 200)
```

- `create_test_user(**overrides)` — creates the user **and its profile** inside
  the test transaction. The profile is not optional: nothing in the package
  creates one automatically, and login rejects a non-superuser without one
  (`invalid_profile`). Pass `is_superuser=False, is_staff=False` for an
  unprivileged caller.
- `do_login(user=None, password=None)` — POSTs to `/auth/login/` and returns the
  sliding token, creating the default user when called bare. It **asserts** the
  200 instead of returning `None`, so a broken login fails where it happens.
- `request(path, token=None, data=None)` — POSTs JSON, adding
  `Authorization: JWT <token>` when a token is given. Note the scheme is **JWT**,
  not `Bearer` (`SIMPLE_JWT["AUTH_HEADER_TYPES"] = ("JWT",)`).

**Never hardcode a user, an id or any other row.** `USER_DATA` is a *template*
the helper instantiates, not a row that must exist; its address uses the
reserved `.invalid` TLD so it can never collide with a real account. This is the
same rule as §5's "create the rows the test needs inside the test" — it exists
because this project already lost a suite that hardcoded `pk=216`.

**Three traps specific to this environment**, none of which the transaction
rollback protects you from:

1. **The database is the live development one** (`NoDbTestRunner` creates no test
   DB). `TestCase` rollback contains the writes; `TransactionTestCase` would
   truncate real tables — never use it, here least of all.
2. **The cache is a shared Memcached instance** (`127.0.0.1:11211`), the same one
   the running dev server uses. **Never call `cache.clear()`** — it would wipe
   that server's cache. The single-session logic writes a key per login, which is
   why `create_test_user` builds the profile with `multiple_login=True`: with the
   flag on, login writes no cache entry at all. Overriding to a non-staff user
   flips the flag back to `False` (`UserProfile.clean`, unless
   `ZB_AUTH_ALLOW_MULTIPLE_LOGIN` is set), and such a test must delete its own
   key — `get_cache_key(request, user)` builds it as `<username>.<md5 of origin>`.
3. **`POST user/add/` sends a real welcome e-mail** over SMTP (`UserService.create`
   → `_send_mail`). Any test touching the user-creation endpoints must wrap them
   in `override_settings(EMAIL_BACKEND="django.core.mail.backends.locmem.EmailBackend")`.

**Where the case list comes from.** `zibanu/django/auth/urls.py` publishes the
routes (all POST, mounted by the project under `auth/` in `mainApp/urls.py`), and
`zibanu/django/auth/openapi/paths/*.yaml` documents each operation's status
codes; the service docstrings repeat them in their `Returns` block. Cross those:
one case per documented code, and report any mismatch between the three rather
than encoding it as correct.

### Module template (header + shape)

```python
# -*- coding: utf-8 -*-

#  Developed by CQ Inversiones SAS. Copyright ©. 2019-2026. All rights reserved.
#  Desarrollado por CQ Inversiones SAS. Copyright ©. 2019-2026. Todos los derechos reservados.

# ****************************************************************
# IDE:          PyCharm
# Developed by: macercha
# Date:         <dd/mm/yy>
# Project:      Zibanu Django
# Module Name:  <module_name>
# Description:
# ****************************************************************
"""Unit tests for :class:`<dotted.path.ClassName>`.

<Why DB-free or not; which code paths are covered separately; where fixtures come from.>

Run with::

    python manage.py test unit_tests.<package>.<module_name>
"""
```

Then imports (stdlib, third-party, django, zibanu, then
`from ..base.test_base import ...`), module-level fixture constants, an optional
small builder helper, then the test classes. Give each test class a one-line
docstring naming the branch it covers; give any regression guard a docstring
saying what breaks if the guard is removed.

---

## 6. Applying fixes (for agents with Edit/Write access)

Project-specific fix patterns, on top of the generic ones in the global
skill. Apply the smallest change that resolves the defect; don't use a fix
as an excuse to refactor the surrounding code (see project-wide "no
unrequested refactors" convention).

### Missing `permission_required` / `permission_classes`
```python
# Before
def destroy(self, request, *args, **kwargs):
    ...
# After
from zibanu.django.rest_framework.decorators import permission_required

@permission_required("app.delete_model")
def destroy(self, request, *args, **kwargs):
    ...
```
Match the permission string convention already used by sibling
actions/methods in the same viewset — don't invent a new one.

### Manual "get or None" reimplementing `get_by_pk()`
```python
# Before
try:
    obj = Model.objects.get(pk=pk)
except Model.DoesNotExist:
    obj = None
# After
obj = Model.objects.get_by_pk(pk)
```
Only apply when the base `zibanu.django.db.Manager` is actually in the
model's MRO — verify before assuming `get_by_pk()` is available.

### Settings read at class-definition time (project settings example)
```python
# Before
class MySerializer(serializers.ModelSerializer):
    max_size = settings.ZB_REPOSITORY_MAX_SIZE  # evaluated at import

# After
class MySerializer(serializers.ModelSerializer):
    def get_max_size(self, obj):
        return settings.ZB_REPOSITORY_MAX_SIZE
```

### After applying any fix
- Re-run the specific test file covering the touched module if one exists
  (`python manage.py test unit_tests.test_<module>`), per §5.
- Don't fix unrelated findings from the same report in the same edit unless
  they're on the same line/block — keep changes traceable to one issue id.

---

## 7. Verifying a fix landed (for audit/verification agents)

Read-only re-verification against the *current* file on disk. Never trust a
prior log's "corregido"/"fixed" claim without re-reading the actual lines:
this project has had a real incident where an applied, verified fix was
silently reverted (suspected IDE buffer overwrite) before the next check.

| Defect | Resolved when the current file shows... |
|---|---|
| Missing `permission_required`/`permission_classes` | The decorator or DRF's `permission_classes` is present on the method, using a permission string consistent with sibling methods |
| Manual "get or None" | Uses `Manager.get_by_pk()` (only expected where the base `Manager` is actually in the model's MRO) |
| Base-package importing an optional package | The import is gone from `zibanu/django/` core files, or the dependency was restructured to not require it |

If the audited file no longer matches the "resolved" state above for an
issue a prior run claimed to have fixed, that is a **regression finding**,
not a formatting nitpick — report it with the same severity the original
issue carried, and flag explicitly that a previously-fixed defect is back.
Don't assume malice or a specific cause (e.g. don't assert "the IDE
overwrote it" as fact) — just report what the file actually contains now
vs. what was claimed.

---

## Quick reference: severity defaults (project-specific additions)

| Finding | Default severity |
|---|---|
| Base-package file depending on an optional package | Important |
| Missing `permission_required`/`permission_classes` on an override | Important |
| Manual "get or None" reimplementing `get_by_pk()` | Suggestion |
