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

No hay templates activos propios del flujo `spec-first-garrido` en este directorio.

No hay templates legacy conservados como parte del sistema actual.

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

La Spec la genera `/spec [feature]`.
El Plan lo genera `@QwikArchitect`.
Audit, Polish y Memory generan sus propios artefactos.

---

## Reglas

```text
No crear templates spec-first sin una decisión explícita.
No listar templates que no existan o que hayan sido retirados del sistema.
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
si reemplaza o retira otro artefacto previo
```