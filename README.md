# SDD Qwik

**Qwik + Qwik City + Bun + Supabase + Drizzle + Tailwind v4 + Context7**

SDD Qwik es un sistema agéntico de **Spec-Driven Development** para construir aplicaciones Qwik con control de alcance, trazabilidad y calidad verificable.

La idea central es simple:

```text
Sin Spec Approved, no hay implementación.
```

Versión/concepto operativo: `spec-first-garrido`.

El sistema evita que la IA improvise. Cada feature pasa por Spec, Plan, Implementation Tasks, Build, Audit, Polish y Memory.

---

## Flujo principal

```text
/setup
  ↓
/spec [feature]
  ↓
/new-feature [feature]
  ↓
@QwikOrchestrator
  ↓
@QwikArchitect crea Plan técnico + Implementation Tasks
  ↓
@QwikDBA si aplica
  ↓
@QwikBuilder ejecuta tasks
  ↓
@QwikAuditor
  ↓
@QwikPolisher
  ↓
@QwikMemory
```

---

## Instalación básica en un proyecto

Estructura esperada:

```text
.github/
├── copilot-instructions.md
├── agents/
└── prompts/

AGENTS.md
docs/
├── standards/
├── specs/
├── plans/
├── audits/
├── bugs/
├── sessions/
└── adr/
```

Verificación inicial:

```text
/setup
```

`/setup` comprueba estructura, prompts, agentes, standards, `docs/sessions/INDEX.md` y emite un health report con el siguiente paso recomendado.

`docs/standards/` es la constitución técnica del sistema: arquitectura, datos, Qwik, UI, seguridad, testing, workflow y calidad se gobiernan desde ahí.

---

## Comandos principales

| Comando | Uso |
|---|---|
| `/setup` | Inicializa o verifica el workspace |
| `/spec [feature]` | Crea una Spec con Acceptance Criteria verificables |
| `/new-feature [feature]` | Abre el ciclo de construcción desde una Spec Approved |
| `/bug-fix [bug-id]` | Diagnostica, clasifica, corrige y verifica bugs |
| `/legacy-audit [path]` | Audita código heredado antes de construir encima |
| `/optimizer-code [path]` | Refactor local sin cambio funcional encubierto |
| `/memory-compact` | Guarda snapshot operativo y Prompt de Reanudación |
| `/new-session` | Reanuda desde snapshot, Prompt de Reanudación o INDEX |

---

## Agentes

| Agente | Responsabilidad |
|---|---|
| `@QwikOrchestrator` | Router central, gates, handoffs y contexto mínimo |
| `@QwikSpeccer` | Feature → Spec verificable |
| `@QwikArchitect` | Spec Approved → Plan técnico + Implementation Tasks |
| `@QwikDBA` | Datos, schema, queries, constraints, permisos y RLS |
| `@QwikBuilder` | Implementation Tasks del Plan → código + Delivery Summary |
| `@QwikAuditor` | Verificación con evidencia y veredicto PASSED/FAILED |
| `@QwikPolisher` | Production readiness tras Audit PASSED |
| `@QwikBugFix` | Ciclo formal de bugs |
| `@QwikMemory` | INDEX, snapshots, Lessons, ADR y cierre |

---

## Escenario 1 — Proyecto nuevo o primera feature

1. Verificar el workspace:

```text
/setup
```

2. Crear la Spec de la primera feature:

```text
/spec [feature]
```

3. Aprobar la Spec.

4. Abrir ciclo de construcción:

```text
/new-feature [feature]
```

5. El sistema enruta:

```text
@QwikOrchestrator → @QwikArchitect → @QwikDBA si aplica → @QwikBuilder → @QwikAuditor → @QwikPolisher → @QwikMemory
```

El Plan técnico contiene las `Implementation Tasks`; no existe un comando separado para crearlas.

---

## Escenario 2 — Feature nueva en proyecto existente

```text
/spec member-invite-flow
```

La Spec debe quedar en `Approved` antes de continuar.

Después:

```text
/new-feature member-invite-flow
```

`/new-feature` verifica la Spec, consulta INDEX, revisa dependencias, crea o preserva el Plan File y entrega el trabajo al Orchestrator/Architect.

Architect traduce la Spec Approved en Plan técnico e `Implementation Tasks`. Builder ejecuta esas tasks; no inventa scope ni reordena el trabajo por intuición.

---

## Escenario 3 — Bug

```text
/bug-fix member-can-access-billing
```

El bug lifecycle exige:

```text
observed
expected
reproducción o evidencia
causa raíz
clasificación
fix
verificación
```

No se parchea un bug como si fuera una feature normal.

---

## Escenario 4 — Código heredado

```text
/legacy-audit src/features/auth
```

Veredictos posibles:

```text
APTO
CONDICIONADO
REFACTOR TOTAL
NO INCORPORAR
```

Sin veredicto, no se construye encima de legacy dudoso.

---

## Escenario 5 — Refactor local

```text
/optimizer-code src/features/billing/components/BillingForm.tsx
```

`/optimizer-code` solo permite refactor, limpieza o descomposición local sin cambio funcional.

Si aparece bug, feature nueva, datos o arquitectura, redirige al flujo correcto.

---

## Escenario 6 — Contexto alto o sesión larga

Antes de perder continuidad:

```text
/memory-compact
```

Produce:

```text
snapshot operativo
INDEX actualizado
Prompt de Reanudación
siguiente paso exacto
agente recomendado
```

En un chat nuevo:

```text
/new-session
```

---

## Artefactos del sistema

```text
docs/specs/      → Specs formales
docs/plans/      → Planes técnicos y handoffs
docs/audits/     → Auditorías
docs/bugs/       → Bugs y causa raíz
docs/sessions/   → INDEX y snapshots
docs/adr/        → Architecture Decision Records
docs/standards/  → Reglas del sistema
```

`docs/sessions/INDEX.md` es estructural y debe poder versionarse.

---

## Gates de calidad

```text
Sin Spec Approved → no código de feature.
Sin Plan técnico con Implementation Tasks → Builder no implementa.
Sin datos/RLS resueltos → Builder no implementa si la feature toca datos.
Sin Delivery Summary verificable → Auditor no puede aprobar.
Sin Audit PASSED → Polisher no actúa.
Sin Memory/INDEX actualizado → cierre incompleto.
```

---

## Reglas de oro

- El Orchestrator enruta; no implementa.
- Builder ejecuta las Implementation Tasks del Plan; no decide producto.
- DBA gobierna datos y RLS.
- Auditor verifica con evidencia; no arregla código.
- Polisher no cambia funcionalidad.
- Memory no guarda ruido: guarda continuidad.
- `/new-feature` es el comando oficial para iniciar construcción; `/feature` es una referencia obsoleta.

---

## Gestión de contexto

El contexto debe ser mínimo.

Patrón recomendado:

```text
INDEX → artefacto activo → standards aplicables → agente correcto
```

Evitar:

```text
leer todas las specs
leer todos los plans
cargar sesiones archivadas
explorar src/ sin scope
```

Cuando el contexto esté alto:

```text
/memory-compact
```

Después, continuar en chat nuevo con:

```text
/new-session
```
