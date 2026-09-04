# Reglas del paquete `zb_auth`

Reglas de diseño propias de `zibanu.django.auth`, decididas por el dueño del
proyecto. Son deliberadas: parecen defectos vistas desde fuera, y por eso las
revisiones estáticas las reportan una y otra vez.

**Alcance:** solo este paquete (`zibanu/django/auth/`). Lo que aplica a todo
Zibanu Django vive en el skill `zibanu-django-conventions`; lo genérico de
Django/DRF, en el skill global `python-django-drf`.

**Para agentes de QA (`qa-review`, `qa-audit`, `qa-code-fixer`, `qa-code-audit`)
y de desarrollo:** lo que esté aquí **no es un hallazgo**. Proponer lo contrario
es un falso positivo, y un fix que lo contradiga debe rechazarse.

---

## Las relaciones de usuario apuntan al proxy del paquete, nunca a `AUTH_USER_MODEL`

Toda relación a usuario declarada **dentro de este paquete** —`ForeignKey`,
`OneToOneField`, `ManyToManyField`— apunta al proxy propio:

```python
from zibanu.django.auth.models.user import User as ZbUser

users = models.ManyToManyField(ZbUser, ...)   # models/application.py
```

**Nunca `settings.AUTH_USER_MODEL`.** Recorrer la relación tiene que entregar
objetos que lleven las reglas del paquete (`User.level`,
`User.get_entity_permissions()`); la referencia intercambiable entrega
instancias planas que no las tienen. Es una garantía de compatibilidad hacia
adelante, así que **«hoy ningún código recorre esa relación» no es argumento en
contra**.

### Precio aceptado

Como el target es un proxy, `add()` rechaza una instancia obtenida de
`get_user_model()` aunque sea exactamente la misma fila:

```python
application.users.add(request.user)                              # TypeError
application.users.add(ZbUser.objects.get(pk=request.user.pk))    # OK
```

Hay que enlazar por PK — por eso el form del admin sí funciona. La suite lo fija
a propósito en `unit_tests/zb_auth/`:
`test_adding_a_plain_user_instance_is_rejected` y
`test_the_users_field_targets_the_zb_auth_proxy_user_model`. **Las dos
aserciones son load bearing: no las "modernices".**

### Hacer que el proxy sea `AUTH_USER_MODEL` no es alternativa

Django lo prohíbe. Designar un proxy como modelo de usuario **swappea su propia
base**, así que quedaría proxyando justo aquello que lo sustituye. Falla en
tiempo de definición de clase:

```
TypeError: ProxyUser cannot proxy the swapped model 'probeapp.ProxyUser'
    django/db/models/base.py:189      (verificado en Django 5.2.13)
```

Con el paquete tal como está da además `ImproperlyConfigured: AUTH_USER_MODEL
refers to model 'zb_auth.User' that has not been installed`, porque el proxy se
define con `get_user_model()` y por tanto se referenciaría a sí mismo. Se
comprobaron las dos variantes, incluida una app de laboratorio con la base fija:
no es un artefacto de cómo está escrito el paquete, es estructural.

Definirlo desde `apps.py` tampoco sirve: `AppConfig.ready()` corre **después** de
que todos los modelos hayan capturado su referencia de FK.

### Excepción que sí es válida

Cambiar un `get_user_model()` **eager** en cuerpo de clase por el string lazy
`settings.AUTH_USER_MODEL` está bien y es otro asunto: ambos resuelven al modelo
base —nunca al proxy—, así que solo cambia la resolución en tiempo de import.
Fueron los hallazgos `AUTH-R83` y `AUTH-R85`, legítimos y aplicados.

### Historia

Apuntar a `settings.AUTH_USER_MODEL` **es** la buena práctica genérica de Django,
y por eso la revisión estática la propone. Se reportó como `AUTH-R76`
(`ZBDJ-92`), se arregló, se commiteó, y en el camino **se reescribieron los dos
tests que lo contradecían** para que el fix pasara. El dueño lo **rechazó** el
2026-09-04: se revirtió el modelo, se borró la migración
`0020_alter_application_users` (estaba sin aplicar) y se restauraron los tests.
