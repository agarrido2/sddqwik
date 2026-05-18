# Núcleo de Reglas del Proyecto (V3.0 — SDD Edition)

> Stack: Qwik + Bun + Supabase + Drizzle ORM + Tailwind v4 + Context7 MCP
> Paradigma: Spec-Driven Development (SDD)
> Este archivo está por encima de cualquier prompt puntual.

---

## 1. Stack Oficial

**Frontend:** Qwik + Qwik City (última versión estable)
**Estilos:** Tailwind CSS v4 — configuración CSS-first en `src/assets/css/global.css`
**Runtime:** Bun (dev/build/CI) · Node.js 20+ Alpine (producción)
**Datos:** Supabase (PostgreSQL 15+) · Drizzle ORM · pgvector · pg_trgm
**IA:** Context7 MCP para documentación viva de librerías externas

---

## 2. Paradigma de Desarrollo: SDD

**La Spec es la fuente de verdad única. Sin Spec aprobada, no se escribe código.**

```
/spec → Spec aprobada → /feature → Plan → Schema → Build → Audit → Polish → 🚀
```

Ver `docs/standards/SDD-WORKFLOW.md` para el proceso completo.

---

## 3. Arquitectura Canónica (Vista Rápida)

```
src/
├── routes/       → Orquestación ÚNICAMENTE (routeLoader$, routeAction$, layout)
├── components/   → UI pura, agnóstica. Sin imports de DB ni servicios.
│   ├── icons/    → Componentes SVG con PropsOf<'svg'>
│   ├── layout/   → Shells y layouts globales
│   └── ui/       → Botones, inputs, cards (design system)
├── lib/          → El cerebro. Servicios, DB, auth, schemas, utils.
│   ├── auth/     → Guards, middleware, permisos RBAC
│   ├── db/       → client.ts, schema.ts, migraciones
│   ├── services/ → Lógica de negocio reutilizable
│   ├── supabase/ → server.ts (cliente SSR)
│   └── utils/    → dark-mode.ts, validaciones, helpers
└── features/     → Solo para features complejas (>5 archivos)
    └── [feature]/ → components/, services/, types.ts, constants.ts
```

**Reglas por dominio:**
- `routes/`: Sin lógica de negocio. Sin acceso directo a DB.
- `components/`: Sin imports de Drizzle, Supabase ni servicios de lib.
- `lib/`: No importa desde `components/` ni `routes/`.
- `features/`: Exponer facade en `src/lib/[feature]/index.ts`.

---

## 4. Runtime: Bun (Dev) + Node.js (Producción)

**Comandos del ciclo de vida:**
```bash
bun install          # Instalación (siempre bun, nunca npm/pnpm)
bun dev              # Servidor de desarrollo (puerto 5173)
bun run build        # Build de producción
bun test             # Tests con Vitest
bun run lint         # Linter
bun run db:generate  # Generar migración Drizzle (ÚNICO comando DB permitido)
```

> **🚫 PROHIBICIÓN PERMANENTE — `bun drizzle-kit push` / `bun run db:migrate`**
>
> Estos comandos están **explícitamente prohibidos** en este proyecto.
> `drizzle-kit push` compara el schema TypeScript con el estado real de la DB
> y elimina todo lo que no reconoce, incluyendo las políticas RLS definidas
> via SQL manual (migraciones 0013–0015+). Causa pérdida irreversible de
> seguridad multi-tenant.
>
> **Flujo canónico obligatorio para cambios de DB:**
> 1. Editar `src/lib/db/schema-identity.ts` o `schema-domain.ts`
> 2. `bun run db:generate` → Drizzle genera el SQL delta en `drizzle/`
> 3. Revisar el SQL generado — añadir políticas RLS y GRANTs manualmente
> 4. Aplicar el SQL en **Supabase Dashboard → SQL Editor**
> 5. Verificar existencia de tablas/columnas en **Supabase Table Editor**
>
> **Referencia:** LL-083 en `LESSONS-LEARNED.md`

**Producción (Docker multi-stage):**
- Stage 1 — Builder: `FROM oven/bun:1` → `bun run build`
- Stage 2 — Runner: `FROM node:20-slim` → `node server/entry.node.js`

**Adaptador requerido:** `adapter-node` (no `adapter-bun` en producción).
**Prohibido en este proyecto:** `npm install`, `pnpm install`, `yarn`.
El `bun.lockb` es binario — si hay conflicto de merge, borrar y regenerar con `bun install`.

---

## 5. Sistema de Documentación

```
docs/
├── prd/        → PRDs aprobados por el cliente (uno por proyecto)
├── blueprint/  → Blueprints técnicos aprobados (uno por proyecto)
├── templates/  → Plantillas PRD y Blueprint (no modificar)
├── specs/      → Specs formales (fuente de verdad del QUÉ)
├── plans/      → Plan Files (fuente de verdad del CÓMO)
├── audits/     → Reportes de auditoría
├── bugs/       → Trazabilidad de bugs
├── sessions/   → Snapshots de contexto (@QwikMemory)
├── adr/        → Architecture Decision Records
└── standards/  → LA BIBLIA (este directorio)

.scratch/       → Temporal, gitignored
scripts/db/     → SQL one-time, gitignored
```

---

## 6. Jerarquía de Agentes (V3.0)

```
@QwikOrchestrator → @QwikBlueprint (proyecto nuevo: PRD → Blueprint)
                 → @QwikSpeccer → @QwikArchitect → @QwikDBA
                                                 → @QwikBuilder → @QwikAuditor → @QwikPolisher
                                                                ↕ (ciclo máx. 2)
                    @QwikMemory (transversal)
```

Ver `AGENTS.md` para el manifest completo y los prompts disponibles.

---

## 7. Resolución de Conflictos entre Standards

Si dos standards se contradicen, este es el orden de prioridad:

1. `copilot-instructions.md` (La Constitución)
2. `docs/standards/DECISIONS-DATA.md` (Para capa de datos)
3. `docs/standards/ARQUITECTURA-FOLDER.md` (Para estructura)
4. `docs/standards/PROJECT-RULES-CORE.md` (Este archivo)
5. Resto de standards
6. Instrucciones del prompt actual
7. Instrucciones ad-hoc del usuario

**Regla dura:** Si el usuario pide algo que viola los puntos 1-4, rechazar con explicación técnica.
Un prompt puntual nunca supera una regla de la Constitución.

---

## 8. TDD — Qué Tiene Test Obligatorio

**Test obligatorio:**
- Lógica en `lib/services/`, `lib/auth/`, `lib/utils/`
- Funciones que calculan, transforman o validan datos
- Casos de borde críticos (errores, datos vacíos, autorizaciones)

**Sin test (excepciones):**
- Componentes UI puramente presentacionales
- Wiring trivial de rutas que solo montan componentes
- Estilos y clases Tailwind

---

## 9. Reglas Anti-Alucinación

**Librerías:** Antes de usar una librería nueva, verificar que:
1. Existe en npm (no es un typo)
2. Tiene actividad reciente
3. No duplica funcionalidad ya presente en el proyecto
4. Si hay duda sobre su API — usar Context7 (`CONTEXT7-GUIDE.md`)

**Nunca inventar APIs de librerías.** Context7 existe para esto.

---

## 10. El Checklist de un PR

- [ ] Sigue la arquitectura (`docs/standards/ARQUITECTURA-FOLDER.md`)
- [ ] Tiene Spec aprobada en `docs/specs/`
- [ ] Plan File actualizado en `docs/plans/`
- [ ] Audit report PASSED en `docs/audits/`
- [ ] No añade dependencias sin verificar
- [ ] Tests incluidos donde aplica
- [ ] `@QwikPolisher` ha emitido `PRODUCTION-READY`

---

## 11. Control de Versiones — `.gitignore`

Estas rutas deben estar en `.gitignore`. El sistema SDD Qwik genera contenido
temporal y volátil que no debe versionarse:

```gitignore
# SDD Qwik — no versionar
.scratch/
scripts/db/

# Snapshots de sesión — volátiles, se regeneran con /memory-compact
docs/sessions/

# Variables de entorno
.env
.env.local
.env.*.local

# Build y dependencias
dist/
node_modules/
.qwik/

# Drizzle — el SQL generado sí se versiona, las migraciones aplicadas también
# Solo excluir el studio temporal
.drizzle/
```

**Lo que SÍ debe versionarse:**
- `docs/specs/` — contratos formales del sistema
- `docs/plans/` — decisiones técnicas tomadas
- `docs/audits/` — historial de calidad
- `docs/bugs/` — trazabilidad de incidencias
- `docs/adr/` — decisiones arquitectónicas permanentes
- `docs/prd/` — requisitos aprobados por el cliente
- `docs/blueprint/` — planos técnicos aprobados
- `drizzle/` — migraciones SQL generadas

**Lo que NO debe versionarse:**
- `docs/sessions/` — snapshots volátiles de contexto de sesión
- `.scratch/` — notas y borradores temporales
- `scripts/db/` — SQL one-time de mantenimiento ya ejecutado