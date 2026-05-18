# De la idea del cliente al código — Guía de proceso

Antes de escribir una sola línea de código, necesitas dos documentos aprobados.
Este directorio contiene las plantillas para crearlos.

```text
docs/templates/
├── PRD-TEMPLATE.md        ← Conversación con el cliente
└── BLUEPRINT-TEMPLATE.md  ← Plano técnico de implementación
```

---

## El flujo completo

```text
Reunión con el cliente
        ↓
Discovery Checklist          ← dentro de docs/templates/PRD-TEMPLATE.md
        ↓
PRD completado
        ↓
Cliente aprueba el PRD
        ↓
Blueprint completado
        ↓
Equipo técnico aprueba el Blueprint
        ↓
/setup                       ← workspace SDD Qwik operativo
        ↓
/spec [módulo]               ← por cada módulo de la Fase 0
        ↓
/feature [módulo]            ← ciclo SDD Qwik completo
```

Nada en este flujo puede saltarse.
Un PRD sin aprobar produce un Blueprint con huecos.
Un Blueprint sin aprobar produce Specs inconsistentes entre sí.

---

## Paso 1 — Reunión de Discovery

Antes de escribir el PRD, usa el **Discovery Checklist** de `docs/templates/PRD-TEMPLATE.md`
como guión de la reunión con el cliente.

Las preguntas marcadas con ⚠️ son críticas: sin respuesta no puedes escribir el PRD.
Las marcadas con 💡 son opcionales, pero enriquecen el resultado.

No salgas de la reunión sin tener respondidas todas las ⚠️.

---

## Paso 2 — Escribir el PRD

Copia `docs/templates/PRD-TEMPLATE.md` a `docs/prd/[nombre-proyecto]-prd.md` y complétalo.

El PRD responde:

- **Qué** problema existe y para quién
- **Qué** va a hacer la aplicación
- **Qué no** va a hacer en esta fase
- **Cómo** sabremos que ha sido un éxito

Cuando esté completo, el cliente lo revisa y lo aprueba formalmente.
Sin aprobación escrita del cliente, no avances.

---

## Paso 3 — Generar el Blueprint con @QwikBlueprint

Con el PRD aprobado, ejecuta:

```bash
/blueprint [nombre-proyecto]
```

`@QwikBlueprint` lee el PRD automáticamente y se encarga de todo:

- Identifica los módulos y sus dependencias
- Define las fases de entrega con criterios de salida claros
- Toma las decisiones técnicas que puede inferir del PRD
- Te pregunta una a una solo las decisiones que realmente no puede inferir
- Genera el Blueprint completo en `docs/blueprint/[nombre-proyecto]-blueprint.md`
- Usa `docs/templates/BLUEPRINT-TEMPLATE.md` como base
- Solicita tu aprobación antes de continuar

No copies ni rellenes `docs/templates/BLUEPRINT-TEMPLATE.md` manualmente.
El agente la usa como plantilla base para generar el documento final.

Cuando apruebes el Blueprint, `@QwikBlueprint` hace handoff a `@QwikSpeccer`
con el primer módulo de Fase 0.

---

## Paso 4 — Arrancar SDD Qwik

Con PRD y Blueprint aprobados, el proceso de desarrollo está listo para empezar.

```bash
# 1. Verificar workspace
/setup

# 2. Por cada módulo de la Fase 0, en el orden del Blueprint
/spec [nombre-módulo]
# → @QwikSpeccer crea la Spec formal con Acceptance Criteria

# 3. Con la Spec aprobada
/feature [nombre-módulo]
# → Ciclo completo: Architect → DBA → Builder → Auditor → Polisher
```

Repite para cada módulo en el orden de fases definido en el Blueprint.

---

## Dónde guardar los documentos

```text
docs/
├── prd/
│   └── [proyecto]-prd.md
├── blueprint/
│   └── [proyecto]-blueprint.md
├── specs/                         ← generado por @QwikSpeccer
├── plans/                         ← generado por @QwikArchitect
└── templates/
    ├── PRD-TEMPLATE.md
    └── BLUEPRINT-TEMPLATE.md
```

---

## Reglas de oro

> **El PRD define el QUÉ. El Blueprint define el CÓMO. Las Specs definen el CÓMO VERIFICARLO.**
> Los tres deben ser coherentes entre sí.

> **Sin PRD aprobado por el cliente, no hay Blueprint.**
> Sin Blueprint aprobado, no hay `/spec`.
> Sin Spec aprobada, no hay `/feature`.

> **El scope OUT es tan importante como el scope IN.**
> Lo que explícitamente no se construye en esta fase evita el principal riesgo
> del desarrollo de software: el scope creep.