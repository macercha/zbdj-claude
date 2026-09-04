---
description: Construye el instalador de un paquete Zibanu Django — valida la versión contra main, sincroniza installers/ y ejecuta make en el servidor
argument-hint: <paquete> [--local | --publish]
allowed-tools: Read, Bash, Glob, Agent, AskUserQuestion
---

Capa conversacional del agente `create-installer`. Tu trabajo es **decidir con el
usuario** y luego **delegar la ejecución**. El agente no tiene canal con él: no puede
preguntar nada mientras corre, así que todo lo que requiera una decisión humana ocurre
aquí, antes de lanzarlo o después de que vuelva.

Argumentos recibidos: `$ARGUMENTS`

## 1. Resolver la invocación

- **Paquete**: `base`, `auth`, `gis`, `logging`, `mqtt`, `repository`, `state_machine`
  o `template` (acepta `zibanu.django.repository` o `zibanu-django-repository`).
  Si no viene o es ambiguo, **pregunta con `AskUserQuestion`** y para. No lo adivines.
- **Modo**: `--local`, `--publish` o ninguno (solo construir). Si no viene, **no
  asumas**: pregunta cuál de los tres, con "solo construir" como opción recomendada.

## 2. Reunir el estado (sin modificar nada)

Lee `.claude/project.config.json` (`environment`, `subprojects[]`) y resuelve las rutas
según la tabla de la definición del agente. Luego, **solo lectura**:

```bash
git -C <repo> branch --show-current
git -C <repo> status --short --untracked-files=no
git -C <repo> show main:<archivo-de-versión> | grep -E '__version__'
```

más el `__version__` actual de la rama de trabajo. **`__build__` no entra en esta
conversación**: es un contador de commits independiente de `__version__`, el instalador
no lo lee ni lo modifica, y un `__build__` que no case con la versión no es un problema
que haya que plantearle al usuario.

## 3. Presentar el plan y esperar confirmación

Muestra en una tabla compacta: paquete, rama, versión de la rama, versión de `main`,
**bump que correspondería** (o "ninguno"), modo, y si hay cambios sin commitear que
quedarán horneados en el instalador. Espera el visto bueno del usuario.

El bump se calcula comparando la **versión principal** (`x.y.z` sin sufijo) de la rama
contra la de `main`, leídas del `__init__.py` raíz del paquete:

| Rama | Comparación con `main` | Bump |
|---|---|---|
| `main` | — | ninguno |
| otra | principal **igual** | `x.y.(z+1)-<etiqueta>.0` — ej. `1.0.0` → `1.0.1-dev.0` |
| otra | principal **mayor** | `x.y.z-<etiqueta>.(b+1)` — ej. `1.0.1-dev.0` → `1.0.1-dev.1` |
| otra | principal **menor** | ninguno: **frena**, la rama quedó por detrás de `main` |

Fuera de `main` **siempre hay bump**, así que siempre hay commit + push que autorizar:
no presentes "ninguno" salvo en `main` o en el caso de rama atrasada.

Antes de seguir, resuelve **aquí** lo que el agente no podrá preguntar:

- **Bump** (siempre, fuera de `main`): confirma la versión destino según la tabla y que
  se hará commit + push a `<rama>`.
- **`--publish` fuera de `main`**: **frena**. Explica que subiría una pre-release a
  pypi.org de forma irreversible y ofrece `--local` o solo construir. Solo sigue con
  autorización explícita dada ahora, sabiendo la rama.
- **`--publish` en `main`**: pide confirmación explícita igualmente, y revisa antes si
  `dist/` ya tiene artefactos de esa misma versión (pypi.org rechaza republicar).

## 4. Delegar la ejecución

Lanza el agente `create-installer` con la decisión ya tomada. En el prompt incluye:
paquete, modo, y la autorización concreta del usuario (bump sí/no y versión destino;
publish autorizado o no). Indícale explícitamente que **no espere confirmación
interactiva** porque corre sin canal, y que ante cualquier compuerta —residuo en el
destino, versión copiada que no coincide, formato de versión que no corresponde a la
rama, rama desconocida— **se detenga y reporte** en lugar de decidir.

Nunca le autorices por adelantado un `--publish` que el usuario no haya aprobado en el
paso 3 sabiendo la rama.

## 5. Cerrar

Cuando el agente vuelva, **verifica lo esencial por tu cuenta** en vez de dar el
informe por bueno: existencia y versión del artefacto en `dist/` remoto, y que el
`__version__` copiado coincide con la fuente. Luego resume al usuario: versión
antes/después, si hubo commit + push, qué se borró y recopió, comando `make` ejecutado
y artefacto resultante.

Si el agente paró en una compuerta, tradúcele el hallazgo al usuario con lo que haría
falta para desbloquearlo, y espera su decisión. No relances el agente por tu cuenta.
