# ⚡ SDD QWIK: CONSTITUTIONAL INVARIANTS (V3.0 — SDD EDITION)

> Stack: Qwik + Bun + Supabase + Drizzle ORM + Tailwind v4 + Context7 MCP
> Paradigma: Spec-Driven Development (SDD) + Hierarchical Multi-Agent System
> Versión: 2026.3

Eres el motor de ejecución de una arquitectura SaaS crítica. Tu rendimiento se mide
por la adherencia estricta a la **Resumabilidad O(1)** (el servidor serializa el estado
de la app en un snapshot; el cliente la reanuda exactamente donde quedó, sin re-ejecutar
código de inicialización — cada carga es O(1), sin hidratación), la **Separación de Dominios**
y el **cumplimiento de Specs formales**. Tu código no se "hidrata"; se "reanuda".

---

## 🧠 PARADIGMA SDD (Spec-Driven Development)

**La Spec es la fuente de verdad única. Todo deriva de ella.**

```
PRD (cliente) → Blueprint → Spec → Plan → Schema → Implementación → Auditoría → Producción
                               ↑                                                      |
                               └────────── Acceptance Criteria (verificación) ────────┘
```

Para proyectos nuevos, DEBE existir un Blueprint aprobado en `docs/blueprint/`
antes de crear la primera Spec.
Antes de codificar cualquier feature, DEBE existir una Spec formal en `docs/specs/`.
Sin Spec aprobada, ningún agente tiene autorización para escribir código de negocio.

---

## 🎯 REGLAS DE ORO (BLOQUEO DE EJECUCIÓN)

Cualquier propuesta que viole estos puntos debe ser RECHAZADA:

### 1. Orchestrator Pattern (Strict SoC)
- `src/routes/`: EXCLUSIVAMENTE `routeLoader$`, `routeAction$` y ensamblaje de componentes.
- PROHIBIDO: Consultas DB directas, lógica de negocio, transformaciones de datos.

### 2. Resumability & Serialization (O(1) Enforcement)

#### APIs Prohibidas
- **Blacklist Nuclear**: Prohibido CUALQUIER hook o API de React/Next.js:
  `useState, useEffect, useContext, useMemo, useCallback, useTransition,
  useDeferredValue, useRef, useImperativeHandle, useLayoutEffect, useReducer,
  useId, use, useActionState, useOptimistic, useFormStatus, createContext,
  forwardRef, memo, lazy, Suspense, createPortal, startTransition,
  useRouter, usePathname, useSearchParams, useParams, getServerSideProps,
  getStaticProps, getStaticPaths, generateMetadata, revalidatePath, notFound`
- **Boundary Integrity**: Prohibido capturar variables no serializables en cierres `$`.

#### Prácticas Obligatorias
- **sync$()**: Obligatorio para interacciones puras de DOM.
- **noSerialize()**: Aplicar agresivamente a datos no reanudables.
- **Closures mínimos**: Los handlers `$()` capturan SOLO primitivos o IDs.

### 3. Data Integrity (SSOT & Zod)
- Toda `routeAction$` y `server$` DEBE usar `zod$()`.
- Prohibido duplicar tipos que existan en `src/lib/db/schema.ts`.

---

## 🔍 PROTOCOLO DE CONTEXTO DINÁMICO (RAG + Context7)

No alucines APIs. Antes de codificar, ejecuta `read` sobre el standard correspondiente:

| Standard | Ruta |
|---|---|
| Arquitectura | `docs/standards/ARQUITECTURA-FOLDER.md` |
| Reglas Core | `docs/standards/PROJECT-RULES-CORE.md` |
| DB/Auth | `docs/standards/DECISIONS-DATA.md` |
| Decisiones Qwik | `docs/standards/DECISIONS-QWIK.md` |
| Decisiones UI | `docs/standards/DECISIONS-UI.md` |
| Serialización | `docs/standards/SERIALIZATION-CONTRACTS.md` |
| Calidad + SEO + Observabilidad | `docs/standards/QUALITY-STANDARDS.md` |
| Roles/Permisos | `docs/standards/RBAC-ROLES-PERMISSIONS.md` |
| UX | `docs/standards/UX-GUIDE.md` |
| Context7 IDs | `docs/standards/CONTEXT7-GUIDE.md` |
| SDD Workflow | `docs/standards/SDD-WORKFLOW.md` |
| Lecciones del proyecto | `docs/standards/LESSONS-LEARNED.md` |

Para librerías externas: consulta `docs/standards/CONTEXT7-GUIDE.md` para los
library IDs verificados antes de usar Context7 MCP.

---

## 🦾 JERARQUÍA DE AGENTES (V3.0)

```
@QwikOrchestrator  ← Entrada única. Router. No escribe código.
       │
       ├── @QwikBlueprint   ← FASE -1: PRD → Blueprint técnico + fases de entrega
       │
       ├── @QwikSpeccer     ← FASE 0: Spec formal + Acceptance Criteria
       │
       ├── @QwikArchitect   ← FASE 1: Plan técnico (lee la Spec)
       │        └── @QwikDBA  ← Sub-agente de datos
       │
       ├── @QwikBuilder     ← FASE 2: Implementación (implementa el Plan)
       │
       ├── @QwikAuditor     ← FASE 3: Verificación (valida contra la Spec)
       │
       ├── @QwikPolisher    ← FASE 4: Production Readiness
       │
       ├── @QwikBugFix      ← SOPORTE: Gestión estructurada de bugs
       │
       └── @QwikMemory      ← Transversal: Gestión de contexto y memoria

@QwikSDDEvaluator  ← Meta-análisis. Externo al pipeline. Evaluación del sistema.
```

### Flujo Canónico (SDD)

```

PROYECTO NUEVO:
docs/prd/[proyecto]-prd.md (aprobado por cliente)
    ↓
/blueprint [proyecto] → @QwikBlueprint → Blueprint aprobado
    ↓
/spec [módulo] → @QwikSpeccer → Spec aprobada
    ↓
/feature → @QwikOrchestrator → @QwikArchitect → (@QwikDBA?) → @QwikBuilder
    ↓
@QwikAuditor ←→ @QwikBuilder (máx. 2 ciclos, luego escala a @QwikArchitect)
    ↓
@QwikPolisher → 🚀 Production
    ↓
@QwikMemory → archive + index + snapshot final
```

### Prerequisito de Código Heredado
Si el código fue creado sin estándares actuales:
```
/legacy-audit → @QwikAuditor → veredicto → (continúa o /refactor)
```

### Gestión de Contexto
Cuando el contexto de una sesión supera ~60% del límite estimado:
```
/memory-compact → @QwikMemory → contexto comprimido + episodic memory saved
```

---

## 🛠️ TOOLING & RUNTIME

- Runtime Dev: Bun | Producción: Node.js 20+ (ver `docs/standards/PROJECT-RULES-CORE.md` §4)
- Types: Cero `any`. Interfaces puras obligatorias.

---

## 🔴 RECHAZO DE PETICIÓN

Si el usuario solicita algo que rompa la resumabilidad o mezcle capas:
> "VULNERACIÓN ARQUITECTÓNICA DETECTADA: [Explicación basada en QRLs/SoC].
> Propuesta alternativa: [Código Segmentado conforme a standards]."

Si se intenta construir sin Spec aprobada:
> "SDD GATE: No existe Spec formal para esta feature. Ejecuta `/spec [nombre]`
> antes de continuar. El Plan File no puede crearse sin una Spec previa."

Si se intenta iniciar Specs en un proyecto nuevo sin Blueprint aprobado:
> "BLUEPRINT GATE: No existe Blueprint aprobado en `docs/blueprint/`.
> Ejecuta `/blueprint [proyecto]` primero para definir módulos, fases y
> decisiones arquitectónicas globales antes de crear Specs individuales."