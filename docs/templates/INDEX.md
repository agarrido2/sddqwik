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

## Templates legacy

No hay templates legacy conservados como parte del sistema actual.

Los artefactos retirados no deben listarse ni recomendarse desde este índice.

---

## Artefactos del flujo spec-first

Los artefactos operativos actuales son producidos por comandos y agentes, no por templates activos en este directorio:

```text
docs/specs/[feature].md
docs/plans/[feature].md
docs/audits/[feature]-audit.md
docs/audits/[feature]-polish.md
docs/sessions/INDEX.md
docs/sessions/[feature]-[timestamp].md
```

La Spec la genera `/spec [feature]`.
El Plan lo genera `@QwikArchitect`.
`Implementation Tasks` viven dentro de `docs/plans/[feature].md`.
Audit, Polish y Memory generan sus propios artefactos.

---

## Regla de uso

- Los templates activos se copian y rellenan — nunca se editan como plantilla base.
- El fichero original permanece vacío y reutilizable.
- Este índice debe listar solo templates que existan y sigan formando parte del sistema.
- Si no hay templates activos, debe decirlo explícitamente.

---

## Añadir un nuevo template

1. Crear el fichero en `docs/templates/nombre-TEMPLATE.md`
2. Añadir fila en este INDEX
3. Indicar si pertenece al flujo spec-first
4. Indicar quién lo consume y dónde se guardan las instancias