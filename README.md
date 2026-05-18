# SDD Qwik — Guía de Uso

**Qwik + Bun + Supabase + Drizzle + Tailwind v4**
Sistema agéntico con Spec-Driven Development (SDD). Cada feature parte de una
Spec formal con Acceptance Criteria verificables. Sin Spec, no hay código.

> **Herramienta:** VS Code + GitHub Copilot (modelo Sonnet 4.6)
> **Entrada de cualquier tarea:** `@QwikOrchestrator`

---

## Instalación en un proyecto

```bash
# 1. Copia la estructura en tu proyecto
.github/
├── copilot-instructions.md
├── agents/          ← 10 agentes
└── prompts/         ← 8 prompts

docs/standards/      ← 12 standards (no tocar)
docs/templates/      ← plantillas PRD y Blueprint
AGENTS.md            ← raíz del proyecto
README.md            ← este fichero

# 2. Crea la estructura de docs si no existe
mkdir -p docs/{specs,plans,audits,bugs,sessions,adr,prd,blueprint,templates}
mkdir -p .scratch

# 3. Verifica que todo está en orden
/setup
```

---

## Escenario 1 — Proyecto nuevo desde cero

```
/setup
```
Verifica la estructura y te da un health report del workspace.

### Fase de discovery y planificación (antes de código)

Usa `docs/templates/PRD_TEMPLATE.md` como guión para la reunión con el cliente.
El Discovery Checklist (Parte 0) te guía con las preguntas críticas que necesitas
responder antes de escribir una línea.

Con el PRD completado y aprobado por el cliente:

```
/blueprint nombre-proyecto
```
`@QwikBlueprint` lee el PRD, identifica los módulos, define las fases de entrega,
toma las decisiones técnicas que puede inferir y te pregunta solo lo que
genuinamente necesita que decidas tú (proveedor de pagos, OAuth, multi-idioma...).
Una pregunta a la vez. Su output es el Blueprint aprobado en `docs/blueprint/`.

### Fase de desarrollo (ciclo SDD Qwik)

Con PRD y Blueprint aprobados, por cada módulo del Blueprint en orden de fases:

```
/spec nombre-módulo
```
`@QwikSpeccer` crea `docs/specs/nombre-módulo.md` con Acceptance Criteria
verificables y binarios. **Revisa y aprueba la Spec antes de continuar.**

```
/feature nombre-módulo
```
A partir de aquí el ciclo es automático:

```
@QwikOrchestrator → routing decision
    ↓
@QwikArchitect → Plan técnico en docs/plans/
    ↓ (si hay cambios de DB)
@QwikDBA → schema.ts + migración
    ↓
@QwikBuilder → implementación en src/
    ↓
@QwikAuditor → verifica contra Spec + standards
    ↓ (si FAILED, máx. 2 ciclos con @QwikBuilder)
@QwikPolisher → Core Web Vitals, bundle, limpieza
    ↓
PRODUCTION-READY ✅
```

Repite `/spec` + `/feature` por cada módulo en el orden definido en el Blueprint.

---

## Escenario 2 — Retomar un proyecto existente tuyo

Si ya tienes código propio escrito sin este sistema:

```
/legacy-audit src/features/auth
```
`@QwikAuditor` analiza el código y emite un veredicto:

| Veredicto | Significado | Siguiente paso |
|---|---|---|
| 🟢 APTO | Código limpio | Puedes hacer `/feature` directamente |
| 🟠 CONDICIONADO | Deuda técnica | Sanear primero con `/optimizer-code` |
| 🔴 REFACTOR TOTAL | Problemas estructurales | `@QwikArchitect` rediseña el dominio |

Una vez saneado, el flujo normal con `/spec` + `/feature`.

---

## Escenario 3 — Incorporarse a un proyecto existente con este sistema

El proyecto ya tiene `docs/specs/`, `docs/plans/` y `docs/audits/`. Lo primero es entender el estado actual:

```
/setup
```
Te da el health report: specs activas, features en curso, bugs abiertos, audits pendientes.

Para entender una feature específica en curso:
```
@QwikOrchestrator  ← describe la feature que quieres retomar
```
El Orchestrator lee el Plan File y te indica exactamente en qué fase está y qué agente debe continuar.

Si la sesión anterior fue larga y no hay snapshot:
```
/memory-compact
```
Antes de continuar, guarda el estado actual para tener un punto de referencia limpio.

---

## Escenario 4 — Corregir un bug

```
/bug-fix nombre-del-bug
```
El sistema crea `docs/bugs/nombre-del-bug.md` y sigue este flujo:

```
@QwikAuditor (diagnóstico + causa raíz)
    ↓
@QwikBuilder (fix quirúrgico)
    ↓
@QwikAuditor (verificación — sin regresiones)
    ↓
Bug cerrado 🟢
```

Si el diagnóstico revela un problema de arquitectura → escala automáticamente a `@QwikArchitect`.
Si el diagnóstico revela un problema de schema → escala a `@QwikDBA`.

---

## Escenario 5 — Refactorizar código con deuda técnica

Cuando un fichero supera 100 líneas, mezcla capas o tiene código difícil de mantener:

```
/optimizer-code src/features/billing/components/BillingForm.tsx
```
`@QwikBuilder` audita y refactoriza siguiendo 4 fases:
1. Diagnóstico de deuda (SoC, DI, closures, estado sobredimensionado)
2. Estrategia de segmentación
3. Refactorización a código prosa
4. Validación de invariantes

---

## Escenario 6 — Sesión larga, contexto saturado

Cuando notes que el modelo pierde el hilo o antes de cerrar una sesión larga:

```
/memory-compact
```
`@QwikMemory` guarda un snapshot en `docs/sessions/` con el estado exacto
y genera un **Prompt de Reanudación** listo para copiar. La próxima sesión
arranca exactamente donde lo dejaste.

Para retomar en una sesión nueva:
```
@QwikOrchestrator Retoma la feature [nombre].
Contexto en docs/sessions/[feature]-[timestamp].md
```

---

## Referencia rápida de comandos

| Comando | Cuándo |
|---|---|
| `/setup` | Iniciar workspace o verificar estado general |
| `/blueprint [proyecto]` | Con PRD aprobado — antes del primer `/spec` |
| `/spec [nombre]` | Por cada módulo del Blueprint, en orden de fases |
| `/feature [nombre]` | Después de tener Spec aprobada |
| `/bug-fix [id]` | Cuando hay un bug |
| `/legacy-audit [ruta]` | Antes de tocar código heredado |
| `/optimizer-code [ruta]` | Fichero con deuda técnica o >100 líneas |
| `/memory-compact` | Contexto alto o al cerrar sesión |

---

## Referencia rápida de agentes

| Agente | Rol | Escribe en |
|---|---|---|
| `@QwikOrchestrator` | Router, coordinador, health check | — |
| `@QwikBlueprint` | PRD → Blueprint técnico, módulos y fases | `docs/blueprint/` |
| `@QwikSpeccer` | Specs formales + Acceptance Criteria | `docs/specs/` |
| `@QwikArchitect` | Plan técnico, fronteras $(), QRL strategy | `docs/plans/` |
| `@QwikDBA` | Schema Drizzle, migraciones, RLS | `src/lib/db/` |
| `@QwikBuilder` | Implementación completa | `src/` |
| `@QwikAuditor` | Verificación Spec + calidad técnica | `docs/audits/` |
| `@QwikPolisher` | Core Web Vitals, bundle, hygiene | `docs/plans/` (cierre) |
| `@QwikMemory` | Snapshots, índice del proyecto, ADRs, compresión de contexto | `docs/sessions/` |

---

## Artefactos generados por el sistema

```
docs/
├── prd/       → docs/prd/[proyecto]-prd.md          ← aprobado por el cliente
├── blueprint/ → docs/blueprint/[proyecto]-blueprint.md ← plano técnico aprobado
├── specs/     → docs/specs/[feature].md              ← QUÉ se construye
├── plans/     → docs/plans/[feature].md              ← CÓMO se construye
├── audits/    → docs/audits/[feature]-audit.md       ← calidad verificada
├── bugs/      → docs/bugs/[bug-id].md                ← trazabilidad de bugs
├── sessions/  → docs/sessions/INDEX.md               ← índice consultable del proyecto
│              → docs/sessions/[feature]-*.md         ← snapshots de contexto
├── adr/       → docs/adr/ADR-NNN-*.md                ← decisiones arquitectónicas
├── templates/ → plantillas PRD y Blueprint (no tocar)
└── standards/ → reglas del proyecto (no tocar)
```

---

## Reglas de oro

> **Sin PRD aprobado por el cliente, no hay Blueprint.**
> Sin Blueprint aprobado, no hay `/spec`.
> Sin Spec aprobada, ningún agente escribe código de negocio.

> **Máximo 2 ciclos Auditor↔Builder.** Si en el tercero hay errores críticos,
> es un problema de diseño — `@QwikArchitect` debe revisar el Plan.

> **Cuando el contexto se satura, `/memory-compact` antes de continuar.**
> Un snapshot de 5 minutos evita perder una hora de decisiones.

---

## Estructura del sistema SDD Qwik

```
proyecto/
│
├── AGENTS.md                          ← Manifest y referencia rápida del sistema
├── README.md                          ← Este fichero
│
├── .github/
│   ├── copilot-instructions.md        ← La Constitución — cargada en cada sesión
│   │
│   ├── agents/
│   │   ├── qwik-orchestrator.agent.md ← Entrada única. Router.
│   │   ├── qwik-blueprint.agent.md    ← Fase -1: PRD → Blueprint técnico
│   │   ├── qwik-speccer.agent.md      ← Fase 0: Specs + Acceptance Criteria
│   │   ├── qwik-architect.agent.md    ← Fase 1: Plan técnico
│   │   ├── qwik-dba.agent.md          ← Fase 1b: Schema + migraciones
│   │   ├── qwik-builder.agent.md      ← Fase 2: Implementación
│   │   ├── qwik-auditor.agent.md      ← Fase 3: Verificación
│   │   ├── qwik-polisher.agent.md     ← Fase 4: Production readiness
│   │   ├── qwik-memory.agent.md       ← Transversal: contexto y snapshots
│   │   └── qwik-bug-fix.agent.md      ← Diagnóstico y corrección de bugs
│   │
│   └── prompts/
│       ├── blueprint.prompt.md        ← /blueprint
│       ├── spec.prompt.md             ← /spec
│       ├── new-feature.prompt.md      ← /feature
│       ├── bug-fix.prompt.md          ← /bug-fix
│       ├── legacy-audit.prompt.md     ← /legacy-audit
│       ├── optimizer-code.prompt.md   ← /optimizer-code
│       ├── memory-compact.prompt.md   ← /memory-compact
│       └── setup.prompt.md            ← /setup
│
└── docs/
    ├── templates/                     ← Plantillas de trabajo por proyecto
    │   ├── PRD_TEMPLATE.md            ← Discovery + PRD
    │   ├── BLUEPRINT_TEMPLATE.md      ← Plano técnico
    │   └── README.md                  ← Guía del proceso PRD → Blueprint → /spec
    └── standards/                     ← No tocar — reglas del proyecto
        ├── INDEX.md
        ├── ARQUITECTURA_FOLDER.md
        ├── PROJECT_RULES_CORE.md
        ├── SDD_WORKFLOW.md
        ├── DECISIONS_QWIK.md
        ├── DECISIONS_DATA.md
        ├── DECISIONS_UI.md
        ├── SERIALIZATION_CONTRACTS.md
        ├── QUALITY_STANDARDS.md
        ├── RBAC_ROLES_PERMISSIONS.md
        ├── UX_GUIDE.md
        ├── CONTEXT7_GUIDE.md
        └── LESSONS_LEARNED.md         ← Vivo — se actualiza con cada error
```

Los directorios `docs/prd/`, `docs/blueprint/`, `docs/specs/`, `docs/plans/`,
`docs/audits/`, `docs/bugs/`, `docs/sessions/` y `docs/adr/` los crea el sistema
automáticamente la primera vez que se invocan los agentes correspondientes.

---

## Gestión de Contexto — Guía Operativa

El contexto no es gratis. Se paga en tokens, en latencia y — lo más grave — en degradación de la inteligencia del modelo cuando la ventana está saturada. Esta sección define cómo usar SDD Qwik de forma que el contexto nunca sea el problema.

### Las tres reglas de oro

**Regla 1 — Cierre de pestañas agresivo**

Antes de invocar cualquier agente, cierra en el editor todo lo que no sea estrictamente necesario para esa fase. Copilot considera "contexto" todos los ficheros abiertos aunque no los menciones.

```
Fase /spec    → abierto: solo la plantilla PRD o Blueprint
Fase /feature → abierto: docs/plans/[feature].md + docs/specs/[feature].md
Fase Builder  → abierto: plan + schema.ts + los ficheros src/ que toca
Fase Auditor  → abierto: plan + audit report + el código a revisar
```

**Regla 2 — Usa `#mentions` explícitos en lugar de contexto abierto**

No dejes que Copilot decida qué estándares leer. En el prompt inicial de cada tarea, especifícalo:

```
# En lugar de dejar todo abierto:
@QwikBuilder implementa el módulo de carrito

# Mejor así:
@QwikBuilder implementa el módulo de carrito.
Usa #DECISIONS_QWIK y #LESSONS_LEARNED. Plan en #docs/plans/cart.md
```

Esto garantiza que el agente lee exactamente lo que necesita — ni más ni menos.

**Regla 3 — Nuevo chat tras `/memory-compact`**

El historial de mensajes es el mayor consumidor de tokens del sistema, y es invisible. Un chat de 50 mensajes con outputs de agentes puede acumular 40,000-60,000 tokens de historial aunque el contexto estimado diga "bajo".

```
Flujo correcto:
1. /memory-compact → @QwikMemory guarda snapshot + Prompt de Reanudación
2. Cierra el chat actual
3. Abre chat nuevo
4. Copia el Prompt de Reanudación del snapshot
5. Continúa desde ahí — sin el peso del historial anterior
```

---

### Análisis de coste por feature

Estimación real de tokens para una feature de complejidad media (1 ciclo de auditoría):

| Fase | Agente activo | Coste estimado |
|---|---|---|
| Carga fija Copilot | `copilot-instructions` + Orchestrator | ~3,000 tokens |
| `/spec` | @QwikSpeccer + standards base | ~19,000 tokens |
| Plan | @QwikArchitect + standards | ~23,000 tokens |
| Schema | @QwikDBA + standards datos | ~19,000 tokens |
| Build | @QwikBuilder + standards Qwik | ~28,000 tokens |
| Audit | @QwikAuditor + standards calidad | ~36,000 tokens |
| **Total** | **Feature completa, 1 ciclo** | **~128,000 tokens** |

Con Copilot Pro (ventana de 200K): **margen de ~72,000 tokens** antes de saturar.
**Conclusión práctica: 1 feature compleja por chat, 2 features simples si aplicas las 3 reglas.**

---

### Puntos ciegos del sistema

Estos son los riesgos reales de contexto que debes conocer:

**1. `ARQUITECTURA_FOLDER.md` es el fichero más pesado del sistema (~8,700 tokens)**
Es el 6.8% de la ventana de 200K por sí solo. Y se carga dos veces: en `/spec` (Speccer) y en `/feature` (Architect). Aplica la Regla 1 para cerrarlo entre fases.

**2. El historial de chat es invisible pero devastador**
Cada output de cada agente — plans, specs, código generado, audit reports — queda en el historial y viaja en cada siguiente petición. Es el mayor consumidor de tokens y el que más degrada la inteligencia del modelo. La Regla 3 es la única solución.

**3. `LESSONS_LEARNED.md` crece sin techo**
Actualmente 9 lecciones. Con 50 features puede tener 40-60 lecciones y se carga en Builder Y Auditor. Rotar periódicamente: cuando una lección lleva mucho tiempo sin reincidencias y está bien consolidada en los standards, sacarla del bloque "Top Lecciones".

**4. Sin disciplina de `#mentions`, Copilot lee de más**
Si tienes 10 pestañas de `docs/standards/` abiertas, todas viajan en cada petición. Sin las 3 reglas, el sistema funciona pero quema el doble de tokens del necesario.

---

### Cuándo ejecutar `/memory-compact`

```
🟢 Contexto <40%    → continúa normalmente
🟡 Contexto 40-60%  → considera compact al terminar la fase actual
🔴 Contexto >60%    → compact antes de continuar + nuevo chat tras el compact
```

El semáforo está en `@QwikMemory` → Métricas de Contexto. Si está en rojo, no invoques al Builder — compact primero.

---

mis propias sheetchets:
A) Caso 1: Refinamiento puramente Visual / UX (No cambia la funcionalidad)
Si solo vas a ajustar Tailwind, márgenes, dark mode, accesibilidad o componentes visuales, el agente ideal es @QwikPolisher o directamente el @QwikBuilder en modo optimización. No necesitas tocar la Spec.

Tu prompt de inicio para la conversación:
    /optimizer-code src/features/contacts/components/[archivo-ui].tsx

    ## Objetivo de refinamiento
    La feature 'contacts' ya es funcional, pero vamos a refinar la UI.
    Quiero que actuemos en modo iterativo. Te iré pidiendo ajustes visuales
    y de UX paso a paso.

    ## Primer ajuste
    [Explica el cambio de Tailwind, alineación, color o feedback visual que necesitas]
    Basate en los tokens de DECISIONS-UI.md.

A partir de ahí, puedes conversar libremente: "Haz el gap más grande", "Ese botón debería ser ghost", etc.

B) Caso 2: Refinamiento de Backend / Lógica de Negocio
Si el ajuste implica que la base de datos devuelve un campo nuevo, un cambio en Drizzle, una validación Zod diferente o un flujo distinto en el routeAction, la Spec debe enterarse primero. Si dejas que el Builder programe sin actualizar la Spec, el Auditor fallará más adelante.

Tu prompt de inicio para la conversación:
    ## Refinamiento de Feature: Contacts

    La feature está implementada pero necesitamos iterar el backend.
    @QwikSpeccer: Necesito modificar la funcionalidad actual.
    Los cambios son:
    1. [Ej: El contacto ahora debe incluir un campo 'empresa']
    2. [Ej: La validación al guardar debe impedir emails duplicados]

    Actualiza el archivo docs/specs/contacts.md (o el que corresponda) añadiendo 
    estos nuevos Acceptance Criteria. 

    Cuando termines, pasa a @QwikBuilder para que implemente estos ajustes 
    en el loader/action y en el esquema Zod.

Aquí la conversación es: "Añade este requisito", el agente actualiza el documento, y luego te da el código.

El secreto para "conversar" sin perder el control
La clave en SDD Qwik no es evitar conversar con el agente, sino darle un marco estricto a la conversación.

1 nicia siempre delimitando los archivos: Si vas a charlar sobre código, usa /optimizer-code [archivo1] [archivo2] para que el agente sepa que la conversación solo afecta a esos archivos.

2 Ciclos cortos: Pídele un cambio, revísalo en tu navegador, y pídele el siguiente. No le pidas 8 ajustes de UI y 3 de backend en el mismo prompt.

3 Evita el "hazlo tú" genérico: Sé específico. En vez de "mejora la tabla", dile "refina la tabla: aplica borders alpha-blended, tabular-nums en las cifras y sticky header, según DECISIONS-UI".

---

## Fastify Server

This app has a minimal [Fastify server](https://fastify.dev/) implementation. After running a full build, you can preview the build using the command:

```
bun serve
```

Then visit [http://localhost:3000/](http://localhost:3000/)

## Bun Server

This app has a minimal [Bun server](https://bun.sh/docs/api/http) implementation. After running a full build, you can preview the build using the command:

```
bun run serve
```

Then visit [http://localhost:3000/](http://localhost:3000/)
