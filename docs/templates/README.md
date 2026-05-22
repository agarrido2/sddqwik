# Templates — SDD Qwik

Este directorio conserva plantillas documentales, pero el sistema operativo actual es `spec-first-garrido`.

Flujo operativo principal:

```text
Spec → Plan → Implementation Tasks → Build → Audit → Polish → Memory
```

Actualmente no hay templates activos del flujo spec-first en este directorio.

No inventar nuevos templates desde esta guía. Si el sistema necesita plantillas para Specs, Plans, Audits o Memory, deben definirse en un cambio explícito posterior.

---

## Estado actual

| Fichero | Estado | Uso en flujo principal |
|---|---|---|
| `PRD-TEMPLATE.md` | Legacy / fuera del flujo operativo principal / pendiente de redefinir | No activo |
| `BLUEPRINT-TEMPLATE.md` | Legacy / fuera del flujo operativo principal / pendiente de redefinir | No activo |

Estos ficheros no se borran todavía.
Tampoco se reinterpretan todavía.

---

## Qué usar hoy

El flujo actual produce artefactos mediante agentes y comandos, no mediante templates activos en este directorio:

```text
/setup
  ↓
/spec [feature]
  ↓
/new-feature [feature]
  ↓
@QwikArchitect crea Plan técnico + Implementation Tasks
  ↓
@QwikBuilder ejecuta tasks
  ↓
@QwikAuditor
  ↓
@QwikPolisher
  ↓
@QwikMemory
```

Artefactos principales:

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

## Reglas

```text
No usar templates legacy como gate operativo.
No recomendar templates legacy desde el flujo principal.
No crear carpetas o artefactos legacy desde esta guía.
No editar los templates legacy salvo cambio explícito posterior.
No crear templates spec-first sin una decisión explícita.
```

---

## Añadir templates en el futuro

Si se decide crear templates activos para `spec-first-garrido`, actualizar primero `docs/templates/INDEX.md` y esta guía.

Cada template nuevo debe indicar:

```text
flujo al que pertenece
agente que lo consume
artefacto que produce
ruta esperada de las instancias
si es activo o legacy/no activo
```