# 📁 Templates Index — SDD Qwik

> Todos los templates viven en `docs/templates/`.
> Son documentos base para rellenar — no standards ni reglas.
> Sistema actual: `spec-first-garrido`.

Flujo operativo principal:

```text
Spec → Plan → Implementation Tasks → Build → Audit → Polish → Memory
```

---

## Templates activos del flujo principal

Actualmente no hay templates activos del flujo `spec-first-garrido` en este directorio.

No inventar templates nuevos desde este índice.
Si el sistema necesita plantillas para Specs, Plans, Audits o Memory, deben definirse en un cambio explícito posterior.

---

## Templates legacy / no activos

Estos ficheros se conservan, pero no forman parte del flujo operativo principal y quedan pendientes de redefinir.

| Fichero | Estado | Uso en flujo principal |
|---|---|---|
| `PRD-TEMPLATE.md` | Legacy / pendiente de redefinir | No activo |
| `BLUEPRINT-TEMPLATE.md` | Legacy / pendiente de redefinir | No activo |

Reglas:

```text
No usarlos como gate operativo.
No recomendarlos desde el flujo principal.
No reinterpretarlos todavía.
No borrarlos todavía.
```

---

## Artefactos del flujo spec-first

Los artefactos operativos actuales son producidos por agentes y no por templates activos en este directorio:

```text
docs/specs/[feature].md
docs/plans/[feature].md
docs/audits/[feature]-audit.md
docs/audits/[feature]-polish.md
docs/sessions/INDEX.md
docs/sessions/[feature]-[timestamp].md
```

`Implementation Tasks` viven dentro de `docs/plans/[feature].md`.

---

## Regla de uso

- Los templates activos se copian y rellenan — nunca se editan como plantilla base.
- El fichero original permanece vacío y reutilizable.
- Este índice debe separar templates activos de templates legacy/no activos.
- Si no hay templates activos, debe decirlo explícitamente.

---

## Añadir un nuevo template

1. Crear el fichero en `docs/templates/nombre-TEMPLATE.md`
2. Añadir fila en este INDEX
3. Indicar si pertenece al flujo spec-first o si es legacy/no activo
4. Indicar quién lo consume y dónde se guardan las instancias