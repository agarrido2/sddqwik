# 🤖 SDD QWIK: AGENT ORCHESTRATION MANIFEST V4.1

> Project Stack: Qwik + QwikCity + Supabase + Drizzle + Tailwind CSS + Context7 MCP
> Paradigma: Spec-Driven Development (SDD)
> Versión: 2026.4
> Modelo unificado: claude-sonnet-4-6

---

## 🏛️ Jerarquía de Agentes

```text
@QwikOrchestrator  ← Entrada única. Router. No escribe código.
       │
       ├── @QwikBlueprint   FASE -1  PRD → Blueprint técnico + fases
       ├── @QwikSpeccer     FASE 0   Spec formal + Acceptance Criteria
       ├── @QwikArchitect   FASE 1   Plan técnico y diseño de implementación
       │       └── @QwikDBA         Sub-agente de datos: schema, migraciones, RLS
       ├── @QwikBuilder     FASE 2   Implementación según Plan
       ├── @QwikAuditor     FASE 3   Verificación contra Spec + standards
       ├── @QwikPolisher    FASE 4   Production Readiness
       ├── @QwikBugFix      SOPORTE  Gestión estructurada de bugs
       └── @QwikMemory      TRANSV.  Contexto, snapshots, ADRs, índice y archivo
```

**Principio rector:**  
Ningún agente debe invadir el dominio principal de otro.  
El sistema funciona por especialización, handoff explícito, gates y artefactos trazables.

---

## 📋 Registro de Agentes

| Agente | Identificador | Dominio | Fichero |
|---|---|---|---|
| Orchestrator | `@QwikOrchestrator` | Entrada única, routing, anti-loop, health check, contexto mínimo y carga selectiva | `.github/agents/qwik-orchestrator.agent.md` |
| Blueprint | `@QwikBlueprint` | Traduce PRD aprobado a blueprint técnico, módulos, fases y orden de ejecución | `.github/agents/qwik-blueprint.agent.md` |
| Speccer | `@QwikSpeccer` | Define specs formales, acceptance criteria, contratos y límites funcionales | `.github/agents/qwik-speccer.agent.md` |
| Architect | `@QwikArchitect` | Convierte Spec en Plan técnico, fronteras, composición, estrategia Qwik/QwikCity | `.github/agents/qwik-architect.agent.md` |
| DBA | `@QwikDBA` | Diseña schema Drizzle, migraciones, constraints, RLS y decisiones de datos | `.github/agents/qwik-dba.agent.md` |
| Builder | `@QwikBuilder` | Implementa el Plan respetando serialización, arquitectura y standards | `.github/agents/qwik-builder.agent.md` |
| Auditor | `@QwikAuditor` | Audita cumplimiento de Spec, calidad técnica, seguridad, A11Y y standards | `.github/agents/qwik-auditor.agent.md` |
| Polisher | `@QwikPolisher` | Lleva una feature a estado production-ready: limpieza, UX final, perf y consistencia | `.github/agents/qwik-polisher.agent.md` |
| BugFix | `@QwikBugFix` | Gestiona bug lifecycle: registro, diagnóstico, clasificación, fix, verificación y cierre | `.github/agents/qwik-bug-fix.agent.md` |
| Memory | `@QwikMemory` | Gestiona memoria episódica, snapshots, ADRs, índice y compresión de contexto | `.github/agents/qwik-memory.agent.md` |

---

## ⚡ Prompts del Sistema

| Comando | Fichero | Agente Invocado | Cuándo |
|---|---|---|---|
| `/blueprint` | `blueprint.prompt.md` | `@QwikBlueprint` | Después de PRD aprobado, antes del primer `/spec` del proyecto |
| `/spec` | `spec.prompt.md` | `@QwikSpeccer` | Siempre primero para toda nueva feature o módulo |
| `/new-feature` | `new-feature.prompt.md` | `@QwikOrchestrator` | Cuando la Spec ya está aprobada |
| `/legacy-audit` | `legacy-audit.prompt.md` | `@QwikAuditor` | Antes de tocar código heredado o no confiable |
| `/bug-fix` | `bug-fix.prompt.md` | `@QwikBugFix` | Cuando hay un bug reportado o una incidencia abierta |
| `/optimizer-code` | `optimizer-code.prompt.md` | `@QwikBuilder` | Refactor puntual, archivo sobredimensionado o deuda localizada |
| `/memory-compact` | `memory-compact.prompt.md` | `@QwikMemory` | Contexto alto, cierre de sesión o compactación estratégica |
| `/new-session` | `new-session.prompt.md` | `@QwikMemory` → `@QwikOrchestrator` | Al abrir un chat nuevo tras `/memory-compact` |
| `/setup` | `setup.prompt.md` | `@QwikOrchestrator` | Inicializar, verificar o diagnosticar el workspace |

**Regla:**  
Los prompts son puertas de entrada operativas.  
No deben saltarse la jerarquía del sistema ni reasignar dominios de forma arbitraria.

---

## Flujo Canónico (SDD)

```text
PROYECTO NUEVO:
docs/prd/[proyecto]-prd.md (aprobado)
  └─► /blueprint [proyecto]
        └─► @QwikBlueprint → docs/blueprint/[proyecto]-blueprint.md
              └─► /spec [modulo-o-feature]
                    └─► @QwikSpeccer → docs/specs/[feature].md (Approved)
                          └─► /new-feature [feature]
                                └─► @QwikOrchestrator → routing
                                      └─► @QwikArchitect → docs/plans/[feature].md
                                            ├─► (si hay cambios de datos) @QwikDBA
                                            │      └─► schema + migración + RLS
                                            └─► @QwikBuilder
                                                  └─► implementación
                                                        └─► @QwikAuditor
                                                              ├─► FAIL ciclo 1-2 → @QwikBuilder
                                                              ├─► FAIL ciclo 3+ → @QwikArchitect
                                                              └─► PASS → @QwikPolisher
                                                                    └─► PRODUCTION-READY
                                                                          └─► @QwikMemory
                                                                                └─► archive + index + snapshot final
```

### Prerrequisito para código heredado

```text
/legacy-audit [ruta]
  └─► @QwikAuditor
        └─► veredicto
              ├─► saneamiento o contención
              └─► recién entonces entrada al ciclo principal
```

---

## 🐛 Flujo de Bugs

```text
/bug-fix [bug-id]
  └─► @QwikBugFix
        ├─► crea o actualiza docs/bugs/[bug-id].md
        ├─► @QwikAuditor → diagnóstico y causa raíz
        ├─► clasificación:
        │      ├─► fix local → @QwikBuilder
        │      ├─► bug de diseño → @QwikArchitect
        │      └─► bug de datos/RLS → @QwikDBA
        ├─► @QwikAuditor → verificación final
        └─► @QwikMemory → memoria o ADR si aplica
```

**Regla crítica:**  
Un bug no entra directamente a implementación.  
Primero se diagnostica, luego se clasifica y solo después se corrige.

---

## 🧠 Sistema de Memoria (4 Capas)

| Capa | Tipo | Ubicación | Gestiona |
|---|---|---|---|
| L0 | Procedimental | `.github/agents/*.agent.md` | Cómo trabaja cada agente |
| L1 | Semántica | `docs/standards/*.md` | Reglas, decisiones y conocimiento técnico estable |
| L2 | Episódica | `docs/specs/`, `docs/plans/`, `docs/audits/`, `docs/bugs/`, `docs/sessions/`, `docs/adr/`, `docs/blueprint/` | Qué ha pasado, qué se decidió y por qué |
| L3 | Trabajo | Contexto activo de la sesión | Información temporal de ejecución |

### Principios de memoria
- `@QwikMemory` optimiza L3 y persiste señal útil a L2.
- `docs/sessions/INDEX.md` es la memoria colectiva consultable del proyecto.
- El Orchestrator debe leer primero el `INDEX.md` antes de cargar artefactos concretos.
- Ninguna feature `✅ Done` debe quedar fuera del `INDEX.md`.
- Si el contexto se satura, se compacta; no se improvisa.
- Cuando `@QwikPolisher` emite `PRODUCTION-READY`, `@QwikMemory` debe archivar snapshots, actualizar índice y consolidar memoria histórica.

---

## 🧭 Responsabilidades por Fase

| Fase | Agente dueño | Artefacto principal | Condición de salida |
|---|---|---|---|
| PRD → Blueprint | `@QwikBlueprint` | `docs/blueprint/[proyecto]-blueprint.md` | Blueprint aprobado y modularizado |
| Spec | `@QwikSpeccer` | `docs/specs/[feature].md` | Spec en estado `Approved` |
| Plan | `@QwikArchitect` | `docs/plans/[feature].md` | Plan técnico ejecutable |
| Datos | `@QwikDBA` | schema, migraciones, RLS, notas de datos | Base de datos alineada con el plan |
| Build | `@QwikBuilder` | código en `src/` | Implementación completa según Plan |
| Audit | `@QwikAuditor` | `docs/audits/[feature]-audit.md` | Resultado `PASSED` o plan de corrección |
| Polish | `@QwikPolisher` | ajustes finales y readiness | Estado `PRODUCTION-READY` |
| Memory | `@QwikMemory` | snapshots, ADRs, index | Memoria L2 actualizada |
| Bug lifecycle | `@QwikBugFix` | `docs/bugs/[bug-id].md` | Bug cerrado con verificación |

---

## 🚪 Gates del Sistema

### Gate 1 — Blueprint
No se inicia trabajo modular serio sin PRD aprobado y blueprint generado.

### Gate 2 — Spec
Sin Spec `Approved`, ningún agente puede escribir código de feature.

### Gate 3 — Plan
Sin Plan técnico, `@QwikBuilder` no debe implementar.

### Gate 4 — Data
Si la feature requiere datos, schema/RLS deben quedar definidos antes del grueso de implementación.

### Gate 5 — Audit
Ninguna feature se considera válida sin auditoría.

### Gate 6 — Polish
Ninguna feature auditada se considera lista para entrega sin polishing final.

### Gate 7 — Memory
Ninguna feature `PRODUCTION-READY` se considera cerrada hasta actualizar memoria, índice y archivo histórico.

---
## 🗺️ Routing Oficial del Orchestrator

| Situación | Agente destino | Prerequisito |
|---|---|---|
| Proyecto sin índice | `@QwikMemory` | Inicializar `docs/sessions/INDEX.md` |
| Índice desactualizado | `@QwikMemory` | Features `Done` sin entrada en `INDEX.md` |
| PRD aprobado, sin blueprint | `@QwikBlueprint` | Existe `docs/prd/[proyecto]-prd.md` |
| Nueva feature sin Spec | `@QwikSpeccer` | Input funcional suficiente |
| Spec en Draft o Review | `@QwikSpeccer` | Spec pendiente de ajuste o cierre |
| Spec aprobada, sin Plan | `@QwikArchitect` | `docs/specs/[feature].md` aprobado |
| Plan con cambios de datos | `@QwikDBA` | Plan requiere schema, migración o RLS |
| Plan listo para ejecutar | `@QwikBuilder` | Plan ejecutable y contexto mínimo cargado |
| Build finalizado | `@QwikAuditor` | Código implementado |
| Audit FAIL, ciclos 1-2 | `@QwikBuilder` | Corrección acotada |
| Audit FAIL, ciclo 3+ | `@QwikArchitect` | Problema de diseño o planificación |
| Audit PASS | `@QwikPolisher` | Auditoría aprobada |
| Production-ready emitido | `@QwikMemory` | Archivo, snapshot e index |
| Código heredado dudoso | `@QwikAuditor` | `/legacy-audit` |
| Bug reportado | `@QwikBugFix` | `/bug-fix` |
| Contexto saturado | `@QwikMemory` | `/memory-compact` ejecutado |
| Reanudación de sesión | `@QwikMemory` → `@QwikOrchestrator` | `/new-session` ejecutado |
| Refactor puntual (`/optimizer-code`) | `@QwikBuilder` | Scope acotado, idealmente con audit previo |

### Protocolo de diagnóstico inicial del Orchestrator

Orden exacto:

```text
1. Leer docs/sessions/INDEX.md
   Si no existe → activar @QwikMemory para inicializarlo

2. Identificar la feature actual en el INDEX
   → leer dependencias y relaciones desde el índice

3. Carga selectiva mínima de artefactos
   - docs/specs/[feature].md
   - docs/plans/[feature].md
   - docs/audits/[feature]-audit.md (solo si aplica)

4. Checks de estado
   - estado del Plan File
   - contexto estimado
   - índice desactualizado o no
   - si hay código heredado
```

### Routing Decision recomendado

```json
{
  "handoff_id": "[feature]-[timestamp]",
  "routing_decision": "[agente]",
  "reason": "[por qué]",
  "context_refs": ["docs/specs/...", "docs/plans/..."],
  "related_features": ["[del INDEX si aplica]"],
  "warnings": ["[prerequisitos incumplidos]"],
  "estimated_cycles": "[N]"
}
```

Esto refuerza el comportamiento operativo del Orchestrator como router con carga selectiva y control de contexto.

---

## 🧹 Política de Contexto


### Contexto mínimo para Builder
Antes de handoff a `@QwikBuilder`, el contexto activo debería reducirse a:


- `docs/specs/[feature].md`
- `docs/plans/[feature].md`
- `src/lib/schemas/db/schema.ts` y submódulos relacionados (`schema-domain.ts`, `schema-identity.ts`, `schema-relations.ts`) si aplica
- `docs/standards/LESSONS-LEARNED.md` o su bloque equivalente vigente
- artefacto de auditoría previo solo si corrige un ciclo fallido


### Contexto a expulsar
- blueprints no necesarios en ejecución
- planes de otras features
- auditorías antiguas no relacionadas
- sesiones archivadas
- specs de features `✅ Done` sin dependencia confirmada


### Umbrales de contexto
- **Umbral de advertencia pre-Builder:** >50% — Context Eviction obligatorio antes del handoff
- **Umbral de compactación global:** ~60% — activar `/memory-compact`


### Regla
Más contexto no implica mejor resultado.  
En SDD Qwik, el contexto debe ser **suficiente, no masivo**.


### Context Eviction pre-Builder
Antes de cada handoff a `@QwikBuilder`, el Orchestrator debe forzar limpieza del contexto activo y evitar arrastrar blueprint, planes ajenos o histórico irrelevante. Esa política ya está explicitada en `QwikOrchestrator` y debe considerarse normativa del sistema.

---

## 🔄 Protocolo Anti-loop

- Los ciclos `Auditor ↔ Builder` tienen máximo **2 iteraciones correctivas normales**.
- Si un tercer ciclo mantiene errores críticos, el problema deja de considerarse “de implementación” y escala a `@QwikArchitect`.
- El Orchestrator debe registrar los ciclos en el Plan File.
- Un bug recurrente o un fail repetido de auditoría debe considerarse señal de diseño, contrato o arquitectura.

### Registro mínimo en el Plan File

```md
## 🔄 Ciclos de Auditoría
- Ciclo 1: [fecha] — [issues]
- Ciclo 2: [fecha] — [issues]
```

A partir de ciclo 3 con errores críticos, el sistema escala a diseño. Esto ya está definido en el protocolo del Orchestrator.

---

## 📊 Health Check del Workspace

Cuando el usuario ejecuta `/setup`, `@QwikOrchestrator` debe:

1. leer `docs/sessions/INDEX.md`
2. extraer totales por estado, WIP y Failed
3. leer solo los `docs/plans/[feature].md` necesarios para features `🚧 WIP`
4. evitar exploraciones masivas de directorios

### Salida esperada

```text
🏥 WORKSPACE HEALTH — [fecha]

📋 Features totales:     [N]
🏗️ En curso (WIP):       [N] — [nombres]
✅ Completadas:          [N]
❌ Con problemas:        [N] — [nombres]
🐛 Bugs abiertos:        [N]
💾 Contexto estimado:    [bajo/medio/alto/crítico]
📑 INDEX:                [✅ actualizado / ⚠️ N features sin indexar]

Recomendación: [acción prioritaria]
```

Esto convierte `/setup` en una inspección controlada y no en un barrido indiscriminado del repo.

---

## 🎯 Carga Selectiva de Contexto

**Regla:** Nunca cargar más contexto del necesario.

| Tipo | Cuándo cargar |
|---|---|
| `docs/sessions/INDEX.md` | Siempre — primera operación |
| `docs/specs/[feature].md` | Siempre — feature en curso |
| `docs/plans/[feature].md` | Siempre — feature en curso |
| `docs/audits/[feature]-audit.md` | Solo en fase de auditoría o corrección de auditoría |
| `docs/specs/[otra-feature].md` | Solo si hay dependencia directa confirmada en INDEX |
| `docs/bugs/[bug-id].md` | Solo cuando routing es hacia un bug |
| snapshots de sesión | Solo para resume, compactación o reconstrucción contextual |

**Nunca cargar por defecto:**
- `ls docs/specs/`
- `ls docs/plans/`
- specs de features `✅ Done`
- sesiones archivadas
- blueprints ya cerrados si no son necesarios para la ejecución

**Principio:** el índice filtra; los artefactos amplían.  
No se explora primero y se piensa después. Se enruta primero y se carga lo mínimo necesario.

---

## 📐 Standards Reference

**Alcance:** esta tabla recoge los standards operativos del ciclo SDD diario.  
No incluye artefactos de evaluación metodológica del sistema.

| Standard | Agente principal | Cuándo leerlo |
|---|---|---|
| `ARQUITECTURA-FOLDER.md` | `@QwikArchitect` | Toda nueva feature |
| `PROJECT-RULES-CORE.md` | `@QwikArchitect` | Toda nueva feature |
| `SDD-WORKFLOW.md` | `@QwikSpeccer`, `@QwikOrchestrator` | Onboarding y dudas de proceso |
| `CONTEXT7-GUIDE.md` | Todos | Antes de usar Context7 MCP |
| `DECISIONS-QWIK.md` | `@QwikBuilder`, `@QwikArchitect` | Toda implementación Qwik/QwikCity |
| `DECISIONS-DATA.md` | `@QwikDBA`, `@QwikBuilder` | Todo cambio de schema, consultas o datos |
| `DECISIONS-UI.md` | `@QwikBuilder`, `@QwikPolisher` | Todo componente o flujo UI |
| `SERIALIZATION-CONTRACTS.md` | `@QwikBuilder`, `@QwikAuditor` | Toda frontera `$()` |
| `QUALITY-STANDARDS.md` | `@QwikAuditor`, `@QwikPolisher` | Auditoría y readiness final |
| `SECURITY-POLICIES.md` | `@QwikDBA`, `@QwikAuditor` | RLS, seguridad de datos y auditoría de seguridad |
| `TESTING-POLICY.md` | `@QwikBuilder`, `@QwikAuditor` | Tests obligatorios, cobertura y validación técnica |
| `RBAC-ROLES-PERMISSIONS.md` | `@QwikArchitect`, `@QwikDBA` | Features con usuarios, roles o permisos |
| `UX-GUIDE.md` | `@QwikBuilder`, `@QwikPolisher` | Todo trabajo de interfaz |
| `LESSONS-LEARNED.md` | `@QwikBuilder`, `@QwikAuditor`, `@QwikBugFix` | Antes de implementar y al resolver bugs |

**Regla de naming:**  
Los standards deben mantener naming consistente.  
Si el repositorio usa guiones, no mezclar con underscores ni variantes legacy.

---

## 🚫 Restricciones Globales

- **Stack obligatorio:** Qwik + QwikCity; no React como base del sistema.
- **SDD Gate:** sin Spec `Approved`, no se escribe código de feature.
- **Orchestrator Pattern:** sin lógica de negocio en `src/routes/`.
- **Serialización:** ningún objeto no serializable cruza una frontera `$()`.
- **RLS:** toda tabla con datos de usuario debe tener policy documentada.
- **Auditoría obligatoria:** no hay `done` sin audit.
- **Polish obligatorio:** no hay `production-ready` sin polish.
- **Memory obligatoria al cierre:** no hay feature cerrada sin indexar o archivar.
- **Anti-alucinación:** APIs externas se verifican con Context7 MCP cuando corresponda.
- **Logging:** evitar `console.log`; usar logging estructurado según standards del proyecto.
- **No exploración masiva del repo:** el sistema debe priorizar `INDEX.md`, carga selectiva y handoffs explícitos sobre inspección indiscriminada.

---

## 🧩 Artefactos Canónicos del Sistema

| Tipo | Ruta |
|---|---|
| PRD | `docs/prd/[proyecto]-prd.md` |
| Blueprint | `docs/blueprint/[proyecto]-blueprint.md` |
| Spec | `docs/specs/[feature].md` |
| Plan | `docs/plans/[feature].md` |
| Audit | `docs/audits/[feature]-audit.md` |
| Bug | `docs/bugs/[bug-id].md` |
| Session Snapshot | `docs/sessions/[feature]-[timestamp].md` |
| Sessions Index | `docs/sessions/INDEX.md` |
| ADR | `docs/adr/ADR-[NNN]-[slug].md` |

---

## 🤝 Handoff Contract

Todo handoff relevante debe dejar constancia en el artefacto principal correspondiente.

### Formato recomendado

```md
### [timestamp] — @QwikOrchestrator → @[Agente]
- **Contexto:** [refs]
- **Tarea:** [descripción]
- **Scope:** [qué sí]
- **No tocar:** [qué no]
- **Condición de salida:** [criterio verificable]
- **Riesgos conocidos:** [si aplica]
```

### Requisitos mínimos del handoff
- referencia a artefactos concretos
- objetivo claro
- límites de scope
- condición de salida verificable
- advertencias o prerequisitos si existen

### Regla
Un handoff sin scope, sin condición de salida o sin artefacto de referencia es incompleto.

Esto está alineado con el handoff estructurado que el Orchestrator ya exige en el Plan File.

---

## 🔑 Resolución de Conflictos

Si dos fuentes se contradicen, usar este orden de prioridad:

1. `copilot-instructions.md`
2. `AGENTS.md`
3. Standards del dominio aplicable:
   - `DECISIONS-DATA.md` para datos
   - `ARQUITECTURA-FOLDER.md` para estructura
   - `SERIALIZATION-CONTRACTS.md` para fronteras `$()`
   - `QUALITY-STANDARDS.md` para calidad y readiness
4. Artefacto aprobado más cercano al trabajo actual:
   - Spec aprobada
   - Plan aprobado
   - Audit vigente
5. Instrucción explícita del usuario
6. Resto de prompts y contexto operativo

**Regla crítica:**  
Si una instrucción del usuario contradice restricciones estructurales del sistema, no debe ejecutarse sin explicitar el conflicto y proponer alternativa.

---

## ✅ Definición de “Perfecto” en este sistema

El sistema solo considera una feature bien resuelta cuando se cumplen todas estas condiciones:

- hay PRD y Blueprint si el alcance lo requiere
- hay Spec aprobada
- hay Plan ejecutable
- el código sigue arquitectura Qwik/QwikCity
- los cambios de datos están resueltos y documentados
- la auditoría pasa
- el polish final pasa
- la memoria queda archivada e indexada
- no quedan decisiones críticas sin trazabilidad

**Ese es el estándar operativo del sistema.**