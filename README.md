# SDD Qwik

**Qwik + Qwik City + Bun + Supabase + Drizzle + Tailwind v4 + Context7**

SDD Qwik es un sistema agéntico de **Spec-Driven Development** para construir aplicaciones Qwik con control de alcance, trazabilidad y calidad verificable.

La idea central es simple:

```text
Sin contrato aprobado, no hay código.
```

El sistema evita que la IA improvise. Cada feature pasa por Spec, Plan, Build, Audit, Polish y Memory.

---

## Flujo principal

```text
PRD Approved
  ↓
/blueprint [project]
  ↓
/spec [feature]
  ↓
/new-feature [feature]
  ↓
@QwikOrchestrator
  ↓
@QwikArchitect / @QwikDBA
  ↓
@QwikBuilder
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
├── templates/
├── prd/
├── blueprint/
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

`/setup` comprueba estructura, prompts, agentes, standards, templates, `docs/sessions/INDEX.md` y emite un health report con el siguiente paso recomendado.

---

## Comandos principales

| Comando | Uso |
|---|---|
| `/setup` | Inicializa o verifica el workspace |
| `/blueprint [project]` | Convierte un PRD aprobado en módulos, fases y orden de Specs |
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
| `@QwikBlueprint` | PRD → Blueprint técnico |
| `@QwikSpeccer` | Feature → Spec verificable |
| `@QwikArchitect` | Spec Approved → Plan técnico |
| `@QwikDBA` | Datos, schema, queries, constraints, permisos y RLS |
| `@QwikBuilder` | Plan aprobado → código + Delivery Summary |
| `@QwikAuditor` | Verificación con evidencia y veredicto PASSED/FAILED |
| `@QwikPolisher` | Production readiness tras Audit PASSED |
| `@QwikBugFix` | Ciclo formal de bugs |
| `@QwikMemory` | INDEX, snapshots, Lessons, ADR y cierre |

---

## Escenario 1 — Proyecto nuevo

1. Crear o completar PRD:

```text
docs/prd/[project]-prd.md
```

2. Ejecutar Blueprint:

```text
/blueprint [project]
```

3. Aprobar Blueprint.

4. Crear la primera Spec según el orden del Blueprint:

```text
/spec [feature]
```

5. Aprobar Spec.

6. Abrir ciclo de construcción:

```text
/new-feature [feature]
```

7. El sistema enruta:

```text
@QwikOrchestrator → @QwikArchitect → @QwikDBA si aplica → @QwikBuilder → @QwikAuditor → @QwikPolisher → @QwikMemory
```

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
docs/prd/        → PRDs aprobados
docs/blueprint/  → Blueprints técnicos
docs/specs/      → Specs formales
docs/plans/      → Planes técnicos y handoffs
docs/audits/     → Auditorías
docs/bugs/       → Bugs y causa raíz
docs/sessions/   → INDEX y snapshots
docs/adr/        → Architecture Decision Records
docs/standards/  → Reglas del sistema
docs/templates/  → Plantillas
```

`docs/sessions/INDEX.md` es estructural y debe poder versionarse.

---

## Gates de calidad

```text
Sin PRD Approved → no Blueprint formal.
Sin Blueprint Approved → no Specs serias en proyecto modular.
Sin Spec Approved → no código de feature.
Sin Plan aprobado/listo → Builder no implementa.
Sin datos/RLS resueltos → Builder no implementa si la feature toca datos.
Sin Delivery Summary verificable → Auditor no puede aprobar.
Sin Audit PASSED → Polisher no actúa.
Sin Memory/INDEX actualizado → cierre incompleto.
```

---

## Reglas de oro

- El Orchestrator enruta; no implementa.
- Builder implementa el Plan; no decide producto.
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
