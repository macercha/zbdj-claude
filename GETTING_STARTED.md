# Guía de Inicio - Zibanu Django

¡Bienvenido! Esta guía te introduce a todos los comandos y flujos de trabajo disponibles en el proyecto Zibanu Django.

**Última actualización:** 2026-07-28

---

## 📋 ¿Qué Existe en Este Proyecto?

### 7 Comandos Personalizados

El proyecto cuenta con **7 comandos principales** organizados en 3 grupos:

#### **Grupo 1: Testing** 🧪
1. `/test-runner` — Ejecutar tests unitarios
2. `/test-writer` — Generar tests unitarios

#### **Grupo 2: QA Pipeline** 🔍
3. `/qa-review` — Revisar calidad de código
4. `/qa-audit` — Auditar cobertura QA
5. `/qa-code-fixer` — Aplicar fixes
6. `/qa-code-audit` — Verificar y hacer commit

#### **Grupo 3: Documentation** 📚
7. `/documentation` — Generar documentación

### 2 Skills Compartidos

- **`python-django-drf`** — Expertise en Django/DRF genérico
- **`zibanu-django-conventions`** — Convenciones específicas del proyecto

---

## 📁 Estructura de Directorios

```
.claude/
├── README.md                          # Overview general
├── AGENTS.md                          # Referencia completa de agentes
├── GETTING_STARTED.md                 # Esta guía (¡estás aquí!)
├── QUICK_REFERENCE.txt                # Tarjeta de referencia rápida
├── DOCUMENTATION_EXAMPLES.md          # Ejemplos prácticos
├── SWAGGER_DOCUMENTATION.md           # Guía de OpenAPI
│
├── commands/                          # Definiciones de comandos
│   ├── test-runner.md
│   ├── test-writer.md
│   ├── qa-review.md
│   ├── qa-audit.md
│   ├── qa-code-fixer.md
│   ├── qa-code-audit.md
│   └── documentation.md
│
├── agents/                            # Definiciones de agentes
│   ├── test-runner.md
│   ├── test-writer.md
│   ├── qa-review.md
│   ├── qa-audit.md
│   ├── qa-code-audit.md
│   └── documentation.md
│
├── skills/                            # Skills compartidos
│   ├── python-django-drf/
│   └── zibanu-django-conventions/
│
└── settings.local.json                # Configuración local
```

---

## 🚀 Primeros Pasos

### Paso 1: Entender la Arquitectura (15 min)

Lee estos archivos en orden:

1. **`../CLAUDE.md`** — Estructura del proyecto, paquetes, instalación
2. **`README.md`** — Overview de los sistemas disponibles
3. **`AGENTS.md`** — Referencia técnica completa

### Paso 2: Elegir Tu Flujo de Trabajo

Depende de lo que necesites hacer:

#### A) Si quieres **escribir/mejorar código**
```
1. /test-writer <path>     → Genera tests para tu código
2. Tu código + tests       → Haces cambios
3. /test-runner <package>  → Verificas que todo funciona
```

#### B) Si quieres **revisar calidad de código**
```
1. /qa-review <path>              → Revisa el código
2. /qa-audit <path>               → Audita la cobertura QA
3. /qa-code-fixer <package>       → Aplica fixes (requiere aprobación)
4. /qa-code-audit <package>       → Verifica y hace commit
```

#### C) Si quieres **documentar**
```
1. /documentation file <path>         → Documenta un archivo
2. /documentation directory <path>    → Documenta paquete completo
3. /documentation readme <path>       → Genera README.md
4. /documentation swagger <path>      → Genera OpenAPI specs
5. /documentation kb <path>           → Genera knowledge base
```

---

## 📚 Los 7 Comandos Explicados

### 1. `/test-runner` — Ejecutar Tests

**Propósito:** Ejecuta tests unitarios en el servidor remoto

**Sintaxis:**
```bash
/test-runner [target]
```

**Parámetros:**
- Sin parámetro: ejecuta todos los tests
- Nombre de paquete: `repository`, `auth`, etc.
- Ruta de módulo: `unit_tests/repository/test_file_metadata.py`

**Ejemplos:**
```bash
/test-runner                                  # Todos los tests
/test-runner repository                       # Tests del paquete repository
/test-runner unit_tests/auth/test_user.py    # Test específico
```

**Salida:** Reportes en `Claude/Tester/<package>/`

**Importante:** Los tests se ejecutan en el servidor remoto `zibanu-io`, no localmente.

---

### 2. `/test-writer` — Generar Tests

**Propósito:** Genera módulos de test unitarios automáticamente

**Sintaxis:**
```bash
/test-writer [target]
```

**Parámetros:**
- Archivo fuente: `zibanu/django/repository/lib/classes/file_metadata.py`
- Directorio: `zibanu/django/repository/lib/classes`
- Sin parámetro: pregunta cuál documentar

**Ejemplos:**
```bash
/test-writer zibanu/django/auth/models/user.py
/test-writer zibanu/django/repository/lib
```

**Salida:** Tests en `unit_tests/<package>/`

---

### 3. `/qa-review` — Revisar Calidad

**Propósito:** Ejecuta revisión de calidad de código Python

**Sintaxis:**
```bash
/qa-review <path> [depth]
```

**Parámetros:**
- `<path>`: archivo o directorio a revisar
- `[depth]`: `quick`, `standard` (default), o `thorough`

**Ejemplos:**
```bash
/qa-review zibanu/django/auth/models/user.py
/qa-review zibanu/django/repository thorough
/qa-review zibanu/django quick                # Revisa base + todos los paquetes
```

**Salida:**
- `Claude/QA/reports/<package>/` — reportes detallados
- `Claude/QA/metrics/<package>/` — métricas
- `Claude/QA/code-reviews/<package>/` — issues encontrados

---

### 4. `/qa-audit` — Auditar Cobertura QA

**Propósito:** Verifica que `/qa-review` cubrió todos los archivos y genera backlog de fixes

**Sintaxis:**
```bash
/qa-audit <path> [--include-tests]
```

**Parámetros:**
- `<path>`: archivo o directorio
- `[--include-tests]`: incluir cobertura de tests (excluido por default)

**Ejemplos:**
```bash
/qa-audit zibanu/django/auth
/qa-audit zibanu/django/repository --include-tests
```

**Salida:** `Claude/QA/audits/<package>/fix_backlog.json`

---

### 5. `/qa-code-fixer` — Aplicar Fixes

**Propósito:** Aplica los fixes del backlog, uno por uno, requiriendo aprobación manual

**Sintaxis:**
```bash
/qa-code-fixer [scope]
```

**Parámetros:**
- Sin parámetro: todos los issues `open`
- `auth`: paquete específico
- `AUTH-`: prefijo de issue ID
- `pending`: re-procesa issues rechazados
- `pending auth`: issues pendientes de auth

**Ejemplos:**
```bash
/qa-code-fixer                  # Todos los open
/qa-code-fixer auth             # Solo auth
/qa-code-fixer AUTH-            # Issues que comiencen con AUTH-
/qa-code-fixer pending          # Issues pendientes
```

**Flujo por issue:**
1. Propone el fix
2. Pide aprobación: `Aplicar`, `Rechazar`, o `Ajustar`
3. Si aprueba: aplica el fix y lo marca como `in progress`
4. Si rechaza: lo marca como `pending`

**Importante:** NO hace commit. Eso lo hace `/qa-code-audit`.

---

### 6. `/qa-code-audit` — Verificar y Hacer Commit

**Propósito:** Re-verifica que los fixes aplicados realmente resuelven los problemas, luego hace commit

**Sintaxis:**
```bash
/qa-code-audit [scope]
```

**Parámetros:**
- Sin parámetro: audita todos los paquetes
- `auth`: paquete específico
- `AUTH-`: prefijo de issue ID

**Ejemplos:**
```bash
/qa-code-audit
/qa-code-audit auth
```

**Flujo:**
1. Re-verifica cada fix `in progress`
2. Si está bien: ofrece hacer commit
3. Si lo apruebas:
   - Incrementa `__build__` (si existe)
   - Hace commit con todos los fixes
   - Marca issues como `closed`

**Importante:** Este es el único lugar donde ocurren commits en la pipeline QA.

---

### 7. `/documentation` — Generar Documentación

**Propósito:** Genera documentación completa (docstrings, READMEs, OpenAPI)

**Sintaxis:**
```bash
/documentation <scope> <path>
```

**Scopes:**

#### `file` — Documentar un archivo
```bash
/documentation file zibanu/django/db/models/model.py
```
Genera: docstrings NumPy en el archivo

#### `directory` — Documentar paquete
```bash
/documentation directory zibanu/django/auth
```
Genera: docstrings NumPy en todos los archivos

#### `readme` — Generar README bilingüe
```bash
/documentation readme zibanu/django/repository
```
Genera: `README.md` (inglés) + `README.es.md` (español)

#### `swagger` — Documentación OpenAPI
```bash
/documentation swagger zibanu/django/auth
```
Genera: specs OpenAPI 3.0 bilingües en YAML

#### `kb` — Knowledge Base para agentes
```bash
/documentation kb zibanu/django/auth
```
Genera: `Claude/KB/<package>/knowledge_base.md` (60-200 líneas)

**Formato:**
- Docstrings: **NumPy style** (inglés)
- README: **Bilingüe** (EN/ES)
- OpenAPI: **Bilingual** (EN/ES)

---

## 🔄 Pipeline QA Completa (El Flujo Real)

Este es el flujo que realmente ocurre en un proyecto de calidad:

### Fase 1: Revisión Inicial
```bash
/qa-review zibanu/django/auth thorough
```
→ Genera reportes de calidad en `Claude/QA/reports/auth/`

### Fase 2: Auditoría
```bash
/qa-audit zibanu/django/auth
```
→ Genera backlog de fixes en `Claude/QA/audits/auth/fix_backlog.json`

### Fase 3: Aplicación de Fixes
```bash
/qa-code-fixer auth
```
→ Por cada issue:
  - Propone el fix
  - Espera tu aprobación (`Aplicar`/`Rechazar`/`Ajustar`)
  - Aplica o rechaza según tu decisión
  - Actualiza estado a `in progress` o `pending`

### Fase 4: Verificación y Commit
```bash
/qa-code-audit auth
```
→ Por cada issue `in progress`:
  - Re-verifica independientemente
  - Si está bien: ofrece hacer commit
  - Haces commit con todos los fixes verificados
  - Actualiza estado a `closed`

### Ciclo de Vida de un Issue

```
planning (initial) 
  ↓
open (validated by qa-audit)
  ↓
in progress (applied by qa-code-fixer)
  ↓ ← aquí verificas independientemente
closed (committed by qa-code-audit)

o bien:
pending (rejected o needs-follow-up)
  ↓
[back to qa-code-fixer pending mode]
```

---

## 🧪 Testing Workflow

### Escribir Tests para Código Nuevo

```bash
# 1. Escribir tests para tu código
/test-writer zibanu/django/repository/lib/controllers/file_controller.py

# 2. Revisar los tests generados
cat unit_tests/repository/test_file_controller.py

# 3. Ejecutar tests para verificar
/test-runner repository
```

### Verificar Tests Después de Cambios

```bash
# Después de hacer cambios en un módulo:
/test-runner repository  # Ejecuta todos los tests del paquete

# O un test específico:
/test-runner unit_tests/repository/test_file_metadata.py
```

---

## 📚 Documentación Workflow

### Documentar un Paquete Completo

Recomendado: seguir este orden para cada paquete

```bash
# 1. Docstrings en archivos
/documentation directory zibanu/django/auth

# 2. README bilingüe
/documentation readme zibanu/django/auth

# 3. OpenAPI specs
/documentation swagger zibanu/django/auth

# 4. Knowledge base para agentes
/documentation kb zibanu/django/auth
```

### Documentar un Archivo Específico

```bash
/documentation file zibanu/django/db/models/model.py
```

### Actualizar Knowledge Base

Cuando cambias la API pública:
```bash
/documentation kb zibanu/django/auth
```

Es seguro ejecutar varias veces — se regenera completamente cada vez.

---

## 🎯 Plan Recomendado de Primeras Acciones

### Semana 1: Familiarización

**Día 1:**
- [ ] Lee `CLAUDE.md` (15 min)
- [ ] Lee `AGENTS.md` (20 min)
- [ ] Lee esta guía (15 min)

**Día 2-3:**
- [ ] Ejecuta `/test-runner` para ver tests existentes (5 min)
- [ ] Ejecuta `/documentation file zibanu/django/api/languages.py` (5 min)
- [ ] Revisa el resultado en el archivo

**Día 4-5:**
- [ ] Ejecuta `/test-writer zibanu/django/api/languages.py` (5 min)
- [ ] Revisa los tests generados
- [ ] Ejecuta `/test-runner unit_tests/zibanu/test_languages.py` (2 min)

### Semana 2: Pipeline QA

**Día 6-7:**
- [ ] Ejecuta `/qa-review zibanu/django/db quick` (5 min)
- [ ] Ejecuta `/qa-audit zibanu/django/db` (5 min)
- [ ] Revisa los reportes en `Claude/QA/`

**Día 8-10:**
- [ ] Ejecuta `/qa-code-fixer zibanu/django/db` (10 min aprox)
- [ ] Aprueba algunos fixes (requiere tu aprobación)
- [ ] Ejecuta `/qa-code-audit zibanu/django/db` (5 min)
- [ ] Revisa los commits

### Semana 3: Documentación a Escala

- [ ] Documenta paquetes pequeños primero (api, lib)
- [ ] Luego paquetes medianos (db, rest_framework)
- [ ] Finalmente paquetes grandes (auth, repository)

---

## 📖 Referencia Rápida

| Quiero... | Comando | Archivo |
|-----------|---------|---------|
| Ver tests existentes | `/test-runner` | `Claude/Tester/<package>/` |
| Generar tests | `/test-writer <path>` | `unit_tests/<package>/` |
| Revisar código | `/qa-review <path>` | `Claude/QA/reports/` |
| Auditar QA | `/qa-audit <path>` | `Claude/QA/audits/*/fix_backlog.json` |
| Aplicar fixes | `/qa-code-fixer <scope>` | `Claude/QA/audits/*/fix_backlog.json` |
| Verificar y commit | `/qa-code-audit <scope>` | Git commits + backlog actualizado |
| Documentar archivo | `/documentation file <path>` | Mismo archivo con docstrings |
| Documentar paquete | `/documentation directory <path>` | Todos los archivos documentados |
| README | `/documentation readme <path>` | `README.md` + `README.es.md` |
| OpenAPI | `/documentation swagger <path>` | `openapi/paths/*.yaml` |
| Knowledge Base | `/documentation kb <path>` | `Claude/KB/<package>/knowledge_base.md` |

---

## ⚠️ Cosas Importantes

### Testing
- ✅ Los tests se ejecutan en el servidor remoto `zibanu-io`
- ✅ No hay Django local en tu Mac
- ✅ Los cambios se sincronizan automáticamente

### QA Pipeline
- ✅ Cada issue requiere aprobación manual en `/qa-code-fixer`
- ✅ Solo `/qa-code-audit` hace commits (nunca `/qa-code-fixer`)
- ✅ Cada paquete se procesa independientemente
- ✅ Si rechazas un fix, va a `pending` para revisarlo después

### Documentación
- ✅ Formato: NumPy (English) + Bilingual (EN/ES)
- ✅ KB se regenera completamente cada vez (seguro)
- ✅ Es barato actualizar after API changes

### Git
- ✅ Base package (`zibanu/django/`) usa `zibanu/.git`
- ✅ Cada paquete opcional (`auth`, `repository`, etc.) tiene su propio `.git`
- ✅ Commits ocurren en su respectivo repo

---

## 🤔 Preguntas Frecuentes

**P: ¿Por dónde empiezo?**
R: Lee CLAUDE.md → AGENTS.md → ejecuta `/test-runner` para familiarizarte

**P: ¿Cuál es la diferencia entre qa-review y qa-audit?**
R: `qa-review` encuentra issues, `qa-audit` verifica que todos fueron revisados

**P: ¿Por qué `/qa-code-fixer` no hace commit?**
R: Porque `/qa-code-audit` re-verifica independientemente primero

**P: ¿Qué formato de docstrings debo usar?**
R: NumPy style (no Google-style)

**P: ¿Puedo ejecutar tests localmente?**
R: No, todo se ejecuta en `zibanu-io`. Tus cambios se sincronizan automáticamente

**P: ¿Qué pasa si rechazó un fix en qa-code-fixer?**
R: Va a `pending`. Puedes revisarlo después con `/qa-code-fixer pending`

---

## 📖 Archivos de Referencia

| Archivo | Para Qué | Cuándo Leer |
|---------|----------|-----------|
| `../CLAUDE.md` | Arquitectura del proyecto | Primera vez |
| `README.md` | Overview de sistemas | Segunda lectura |
| `AGENTS.md` | Referencia técnica completa | Necesito detalles |
| `QUICK_REFERENCE.txt` | Sintaxis rápida | Recordar comandos |
| `DOCUMENTATION_EXAMPLES.md` | Ejemplos por paquete | Documentar algo |
| `SWAGGER_DOCUMENTATION.md` | Guía OpenAPI | Documentar endpoints |
| `commands/*.md` | Detalles de cada comando | Debug o profundizar |
| `agents/*.md` | Definición técnica de agentes | Curiosidad técnica |

---

## ✅ Checklist: Listo para Empezar

- [ ] He leído CLAUDE.md
- [ ] He leído AGENTS.md
- [ ] He ejecutado `/test-runner` una vez
- [ ] He visto cómo se ven los reportes
- [ ] Entiendo la pipeline QA (review → audit → fixer → code-audit)
- [ ] Sé dónde están los 7 comandos
- [ ] Estoy listo para trabajar

---

## 🚀 Próximos Pasos

Elige uno:

### Opción A: Aprender Testeando
```bash
/test-runner
# Luego
/test-writer zibanu/django/api/languages.py
```

### Opción B: Aprender Revisando
```bash
/qa-review zibanu/django/db quick
/qa-audit zibanu/django/db
```

### Opción C: Aprender Documentando
```bash
/documentation directory zibanu/django/api
/documentation readme zibanu/django/api
```

---

**¡Que disfrutes trabajando con Zibanu Django! 🚀**
