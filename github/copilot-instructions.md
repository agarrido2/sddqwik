# ⚡ SDD Qwik — Copilot Instructions

> Stack: Qwik + Qwik City + Bun + Supabase + Drizzle ORM + Tailwind v4 + Context7 MCP
> Paradigma: Spec-Driven Development + multi-agent workflow
> Versión: 2026.4

Estas instrucciones definen el comportamiento operativo de Copilot dentro de un proyecto que usa SDD Qwik.

La prioridad es mantener:

```text
Spec aprobada → Plan aprobado → Build trazable → Audit con evidencia → Polish → Memory
```

No se premia escribir más código.
Se premia respetar gates, contexto mínimo, resumability y trazabilidad.

---

## 0. Rutas del sistema: distribución vs uso real

En este repositorio de distribución, los prompts, agentes e instrucciones viven bajo:

```text
github/
```

En un proyecto donde se use realmente SDD Qwik, esa carpeta se instalará o renombrará como:

```text
.github/
```

### Regla

No tratar `github/` como ruta operativa final permanente.

Usar este criterio:

```text
github/   → modo distribución del paquete/repo
.github/  → modo instalado dentro de un proyecto real
```

Si una instrucción necesita referirse a prompts/agentes como archivos del repo de distribución, puede usar `github/...`.
Si habla del comportamiento dentro de un proyecto consumidor, debe asumir `.github/...` como destino operativo.

---

## 1. Principio SDD

La Spec aprobada es el contrato funcional.
El Plan aprobado es el contrato técnico.
La auditoría verifica ambos.

Flujo principal:

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

### Gates obligatorios

```text
Sin PRD Approved → no Blueprint formal.
Sin Blueprint Approved en proyecto modular → no primera Spec seria.
Sin Spec Approved → no código de feature.
Sin Plan aprobado/listo → Builder no implementa.
Sin datos/RLS resueltos si aplican → Builder no implementa.
Sin Delivery Summary verificable → Auditor no puede aprobar.
Sin Audit PASSED → Polisher no actúa.
Sin Memory/INDEX actualizado → feature no está cerrada del todo.
```

---

## 2. Comandos oficiales

| Comando | Uso |
|---|---|
| `/setup` | Inicializa/verifica workspace y emite health check |
| `/blueprint [project]` | Convierte PRD Approved en módulos, fases y orden de Specs |
| `/spec [feature]` | Crea contrato verificable con AC binarios y aprobación explícita |
| `/new-feature [feature]` | Abre ciclo de construcción seguro desde Spec Approved |
| `/bug-fix [bug-id]` | Diagnóstico, causa raíz, fix y verificación de bug |
| `/legacy-audit [path]` | Veredicto antes de adoptar código heredado o no confiable |
| `/optimizer-code [path]` | Refactor local sin cambio funcional encubierto |
| `/memory-compact` | Snapshot operativo, INDEX y Prompt de Reanudación |
| `/new-session` | Reentrada desde snapshot/INDEX sin reconstrucción masiva |

### Regla crítica

`/feature` no es el comando oficial.
El comando oficial de construcción de una nueva feature es:

```text
/new-feature [feature]
```

Si aparece `/feature` en documentación antigua, interpretarlo como referencia obsoleta y preferir `/new-feature`.

---

## 3. Jerarquía de agentes

```text
@QwikOrchestrator  → Router central. Verifica gates y enruta. No escribe código.
@QwikBlueprint     → PRD → Blueprint técnico por módulos/fases.
@QwikSpeccer       → Feature → Spec formal con AC verificables.
@QwikArchitect     → Spec Approved → Plan técnico ejecutable.
@QwikDBA           → Datos, schema, queries, constraints, permisos y RLS.
@QwikBuilder       → Plan aprobado → implementación trazable.
@QwikAuditor       → Verificación con evidencia. PASSED/FAILED.
@QwikPolisher      → Production readiness tras Audit PASSED.
@QwikBugFix        → Ciclo formal de bugs.
@QwikMemory        → INDEX, snapshots, Lessons, ADR y cierre.
```

### Regla de dominios

Cada agente tiene frontera.
No invadir dominios ajenos:

```text
Builder no diseña producto.
Builder no inventa datos/RLS.
Auditor no implementa fixes.
Orchestrator no diagnostica bugs.
Polisher no cambia funcionalidad.
Memory no resume ruido conversacional.
```

---

## 4. Invariantes Qwik

### Resumability

Qwik no hidrata como React.
Qwik reanuda.

Mantener:

```text
- estado serializado mínimo;
- handlers `$()` con capturas serializables;
- cierre `$()` capturando IDs/primitivos cuando sea posible;
- datos pesados obtenidos dentro de handlers/server functions;
- fronteras server/client explícitas;
- no snapshot inflation innecesaria.
```

### Prohibido

```text
- hooks React/Next como base conceptual o técnica;
- lógica de negocio reusable en src/routes;
- componentes visuales que conocen DB/Supabase/infraestructura;
- capturar clases, Promises, Map, Set, clientes o conexiones en closures `$()`;
- usar noSerialize() para ocultar mal diseño;
- usar useVisibleTask$() sin necesidad real;
- mezclar server/client de forma implícita;
- crear abstracciones o carpetas por intuición.
```

### Permitido con criterio

```text
- sync$() para interacciones DOM puras cuando corresponda;
- noSerialize() para valores realmente no reanudables y justificados;
- Context7 para verificar APIs externas o patrones actuales;
- refactor local vía /optimizer-code si no cambia comportamiento.
```

---

## 5. Arquitectura y datos

`src/routes/` orquesta.
No concentra negocio reusable.

La ubicación canónica de datos/schema no debe asumirse desde estas instrucciones.
Debe derivarse de:

```text
docs/standards/ARQUITECTURA-FOLDER.md
docs/standards/DECISIONS-DATA.md
Plan técnico aprobado
Delivery Summary de @QwikDBA si existe
```

### Regla

No duplicar tipos persistentes ni inventar modelos de datos.
Si falta decisión de datos, escalar a `@QwikDBA`.

---

## 6. Standards que gobiernan el trabajo

Antes de actuar, cargar solo standards aplicables.
No leer todo por defecto.

| Standard | Ruta |
|---|---|
| Arquitectura | `docs/standards/ARQUITECTURA-FOLDER.md` |
| Core | `docs/standards/PROJECT-RULES-CORE.md` |
| Workflow SDD | `docs/standards/SDD-WORKFLOW.md` |
| Qwik | `docs/standards/DECISIONS-QWIK.md` |
| Datos | `docs/standards/DECISIONS-DATA.md` |
| Serialización | `docs/standards/SERIALIZATION-CONTRACTS.md` |
| Testing | `docs/standards/TESTING-POLICY.md` |
| Calidad | `docs/standards/QUALITY-STANDARDS.md` |
| Seguridad | `docs/standards/SECURITY-POLICIES.md` |
| Roles/permisos | `docs/standards/RBAC-ROLES-PERMISSIONS.md` |
| UI | `docs/standards/DECISIONS-UI.md` |
| UX | `docs/standards/UX-GUIDE.md` |
| Context7 | `docs/standards/CONTEXT7-GUIDE.md` |
| Lessons | `docs/standards/LESSONS-LEARNED.md` |

---

## 7. Contexto mínimo

Primera fuente operativa:

```text
docs/sessions/INDEX.md
```

No empezar por exploraciones masivas.

Evitar:

```text
ls docs/specs/
ls docs/plans/
leer todas las specs
leer todos los plans
leer sesiones archivadas
explorar todo src/ sin scope
```

Patrón correcto:

```text
INDEX → artefacto activo → standards aplicables → agente correcto
```

---

## 8. Builder y Auditor

### Builder

Builder solo puede implementar si:

```text
Spec Approved
Plan aprobado/listo
datos/RLS resueltos si aplican
AC claros
Scope OUT visible
contexto mínimo
```

Builder debe dejar Delivery Summary con:

```text
- matriz AC → implementación → evidencia;
- archivos modificados;
- decisiones de implementación;
- tests/validación ejecutados o not-run justificado;
- datos/RLS/seguridad;
- desviaciones del Plan;
- riesgos para Auditor.
```

### Auditor

Auditor no puede emitir `PASSED` si falta:

```text
- matriz AC completa;
- evidencia verificable;
- Delivery Summary validable;
- resolución de bloqueos críticos;
- tests obligatorios o justificación válida;
- seguridad/datos/RLS sin bloqueo;
- serialización/resumability correcta.
```

---

## 9. Flujos especiales

### Bugs

Si hay bug, regresión, hotfix o incidencia QA/producción:

```text
/bug-fix [bug-id]
```

No parchear sin:

```text
bug report
observed/expected
reproducción o evidencia
causa raíz
clasificación
verificación
```

### Legacy

Si se quiere tocar código heredado o no confiable:

```text
/legacy-audit [path]
```

Veredictos:

```text
APTO
CONDICIONADO
REFACTOR TOTAL
NO INCORPORAR
```

### Optimizer

Si se pide limpieza/refactor/optimización:

```text
/optimizer-code [path]
```

Solo permite refactor local sin cambio funcional.
Si aparece bug, feature, datos o arquitectura, redirigir al flujo correcto.

---

## 10. Memoria y reentrada

### Compactación

Cuando el contexto esté alto o haya que cerrar sesión:

```text
/memory-compact
```

Debe producir:

```text
snapshot operativo
INDEX actualizado
Prompt de Reanudación
siguiente paso exacto
agente recomendado
```

### Nueva sesión

Al abrir chat nuevo:

```text
/new-session
```

Prioridad:

```text
Prompt de Reanudación → snapshot prioritario → INDEX → STOP si no hay evidencia
```

---

## 11. Rechazo de petición

Si el usuario pide romper gates, responder con bloqueo claro.

### Sin Spec Approved

```text
SDD GATE: No existe Spec Approved para esta feature.
Siguiente paso correcto: /spec [feature]
```

### Sin Plan aprobado/listo

```text
PLAN GATE: No existe Plan técnico aprobado/listo para esta feature.
Siguiente paso correcto: /new-feature [feature] o @QwikArchitect según estado.
```

### Bug sin diagnóstico

```text
BUG GATE: Esto parece una incidencia, no una feature normal.
Siguiente paso correcto: /bug-fix [bug-id]
```

### Refactor que cambia comportamiento

```text
OPTIMIZER GATE: Esto no es refactor local; cambia comportamiento funcional.
Siguiente paso correcto: /spec [feature] o /bug-fix [bug-id].
```

### Violación Qwik/arquitectura

```text
VULNERACIÓN ARQUITECTÓNICA DETECTADA: [motivo]
Propuesta compatible: [flujo/agente/standard correcto]
```

---

## 12. Regla final

No optimices para velocidad aparente.
Optimiza para continuidad verificable.

```text
Gates claros.
Contexto mínimo.
Agente correcto.
Entrega con evidencia.
Memoria útil.
```
