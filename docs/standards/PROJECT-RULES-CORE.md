# Núcleo de Reglas del Proyecto — SDD Qwik

> Stack: Qwik + Qwik City + Bun + Supabase + Drizzle ORM + Tailwind v4 + Context7 MCP
> Paradigma: Spec-Driven Development + workflow multi-agente
> Versión: spec-first-garrido

Este archivo define reglas nucleares del proyecto.
Está por encima de prompts puntuales cuando haya conflicto, salvo instrucciones explícitas del usuario que no rompan la seguridad ni los gates estructurales.

---

## 1. Stack oficial

**Frontend:** Qwik + Qwik City  
**Estilos:** Tailwind CSS v4, configuración CSS-first  
**Runtime:** Bun para desarrollo/build/CI · Node.js 20+ para producción  
**Datos:** Supabase/PostgreSQL · Drizzle ORM  
**IA:** Context7 MCP para documentación viva de librerías externas

---

## 2. Paradigma de desarrollo: SDD

Sin contrato aprobado, no se implementa.

Flujo canónico:

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

Reglas duras:

```text
Sin Spec Approved → no código de feature.
Sin Plan técnico con Implementation Tasks → Builder no implementa.
Sin datos/RLS resueltos si aplican → Builder no implementa.
Sin Delivery Summary verificable → Auditor no puede aprobar.
Sin Audit PASSED → Polisher no actúa.
Sin Memory/INDEX actualizado → cierre incompleto.
```

> Regla central: **Sin Spec Approved, no hay implementación.**
> PRD y Blueprint no gobiernan el flujo operativo principal.

`/feature` es referencia obsoleta.
El comando oficial es:

```text
/new-feature [feature]
```

---

## 3. Arquitectura canónica

La arquitectura detallada la gobierna:

```text
docs/standards/ARQUITECTURA-FOLDER.md
```

Vista rápida:

```text
src/
├── routes/      → orquestación de rutas, loaders, actions y layouts
├── components/  → UI reusable/presentacional
├── lib/         → servicios, auth, datos, utilidades compartidas
└── features/    → dominios de feature cuando la complejidad lo justifica
```

Reglas nucleares:

- `routes/` no concentra lógica de negocio reusable.
- `components/` no conocen Drizzle, Supabase ni infraestructura sensible.
- `lib/` es compartido real, no cajón de sastre.
- `features/` se usa cuando hay dominio suficiente.
- Las rutas exactas de datos/schema las definen `ARQUITECTURA-FOLDER`, `DECISIONS-DATA`, el Plan y DBA.

No asumir una ruta única de schema desde este archivo.

---

## 4. Runtime y comandos

Usar Bun para instalación y ciclo de desarrollo:

```bash
bun install
bun dev
bun run build
bun test
bun run lint
bun run db:generate
```

### Prohibiciones DB

No usar comandos destructivos o no aprobados que puedan desincronizar Drizzle, Supabase o RLS.

Si hay cambios de datos:

```text
@QwikDBA debe resolver schema/migraciones/queries/constraints/permisos/RLS.
Builder no improvisa datos.
```

El flujo concreto de DB lo gobierna:

```text
docs/standards/DECISIONS-DATA.md
```

---

## 5. Sistema documental

```text
docs/
├── templates/  → Plantillas base
├── specs/      → Specs formales: fuente del QUÉ
├── plans/      → Plan Files: fuente del CÓMO
├── audits/     → reportes de auditoría
├── bugs/       → trazabilidad de bugs
├── sessions/   → INDEX y snapshots de contexto
├── adr/        → decisiones arquitectónicas permanentes
└── standards/  → reglas del sistema

.scratch/       → temporal, no versionable
scripts/db/     → SQL puntual o mantenimiento según política del proyecto
```

### Regla de INDEX

`docs/sessions/INDEX.md` debe ser versionable.
Es la primera fuente de navegación operativa para Orchestrator, Memory y reentrada.

Los snapshots pueden ser volátiles, pero el INDEX no debe perderse por un `.gitignore` demasiado amplio.

---

## 6. Jerarquía de agentes

```text
@QwikOrchestrator → router central, gates spec-first, handoffs, contexto mínimo
@QwikSpeccer      → Feature → Spec verificable
@QwikArchitect    → Spec Approved → Plan técnico + Implementation Tasks
@QwikDBA          → datos, schema, queries, constraints, permisos, RLS
@QwikBuilder      → Implementation Tasks → código + Delivery Summary
@QwikAuditor      → verificación con evidencia
@QwikPolisher     → production readiness
@QwikBugFix       → ciclo formal de bugs
@QwikMemory       → INDEX, snapshots, Lessons, ADR, cierre
```

Cada agente respeta su dominio.
Coordinar no significa invadir.

---

## 7. Resolución de conflictos

Orden recomendado:

1. Instrucción explícita del usuario, si no rompe gates/seguridad.
2. `AGENTS.md`.
3. `.github/copilot-instructions.md`.
4. Standard aplicable al dominio.
5. Artefacto aprobado más cercano: Spec, Plan, Audit, Bug report.
6. Prompt actual.
7. Contexto conversacional.

Si una fuente antigua contradice el flujo reforzado, señalar obsolescencia y preferir el flujo actual.

---

## 8. Testing obligatorio

Gobernado por:

```text
docs/standards/TESTING-POLICY.md
```

Reglas base:

- servicio nuevo o modificado → test obligatorio;
- helper crítico → test o justificación;
- lógica de datos/permisos → test si es viable;
- bugfix → test de regresión si es viable;
- UI puramente presentacional → test no obligatorio salvo lógica relevante.

No inventar resultados de tests.
Si no se ejecutan, documentar motivo.

---

## 9. Anti-alucinación

Antes de usar librerías o APIs dudosas:

- verificar standard interno aplicable;
- usar Context7 si hay riesgo de versión, API o deprecación;
- no inventar nombres de funciones, paquetes ni patrones;
- no añadir dependencias si duplican funcionalidad existente.

---

## 10. Checklist de PR/feature

Una feature solo debería considerarse lista si:

- [ ] existe Spec Approved;
- [ ] existe Plan aprobado/listo con Implementation Tasks;
- [ ] datos/RLS están resueltos si aplican;
- [ ] Builder dejó Delivery Summary verificable;
- [ ] tests obligatorios están presentes o justificados;
- [ ] Audit report está `PASSED`;
- [ ] Polisher emitió `PRODUCTION-READY`;
- [ ] Memory actualizó INDEX/snapshot si aplica;
- [ ] no hay desviaciones no aprobadas.

---

## 11. Control de versiones y `.gitignore`

Debe versionarse:

```text
docs/specs/
docs/plans/
docs/audits/
docs/bugs/
docs/adr/
docs/standards/
docs/sessions/INDEX.md
```

Puede ignorarse:

```text
.scratch/
scripts/db/ temporales si la política del proyecto lo define
docs/sessions/archive/
docs/sessions/*.tmp.md
snapshots puramente locales o temporales
.env
.env.local
.env.*.local
dist/
node_modules/
.qwik/
.drizzle/
```

No añadir esta línea sin excepción:

```gitignore
docs/sessions/
```

porque ocultaría `docs/sessions/INDEX.md`, que es estructural para el sistema.

---

## 12. Flujos especiales

```text
/bug-fix [bug-id]       → diagnóstico, causa raíz, clasificación y verificación
/legacy-audit [path]    → veredicto antes de construir encima
/optimizer-code [path]  → refactor local sin cambio funcional encubierto
/memory-compact         → snapshot operativo y Prompt de Reanudación
/new-session            → reentrada sin reconstrucción masiva
```

---

## 13. Regla final

El sistema no existe para que la IA escriba código rápido.

Existe para que la IA escriba código controlado:

```text
contrato claro
plan claro
build trazable
audit verificable
cierre con memoria útil
```
