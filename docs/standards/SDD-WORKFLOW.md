# SDD WORKFLOW: Spec-Driven Development en SDD Qwik

> **Propósito:** Definir el proceso completo de Spec-Driven Development adoptado por el equipo
> Este documento es el "por qué" detrás del sistema agéntico.
> **Audiencia:** Desarrolladores, agentes IA y nuevos colaboradores.

---

## 🎯 La Tesis Central

> **La especificación es el código más importante que escribes.**

En 2026, los agentes IA pueden generar código correcto rápidamente. El cuello de botella no es la velocidad de implementación — es la claridad de lo que se debe implementar.

Un agente IA con una Spec vaga producirá código correcto para el problema equivocado.
Un agente IA con una Spec precisa producirá código correcto para el problema correcto.

---

## 🏗️ El Sistema de Memoria en 4 Capas

Los agentes del proyecto trabajan con 4 tipos de memoria, cada uno con una función distinta:

```
L0 — MEMORIA PROCEDIMENTAL (Cómo trabajar)
     .github/agents/*.agent.md
     .github/copilot-instructions.md
     → Quién soy, qué puedo hacer, cómo tomo decisiones

L1 — MEMORIA SEMÁNTICA (Conocimiento del dominio)
     docs/standards/*.md
     → Qwik, Drizzle, Supabase, UX, RBAC...
     → Se lee bajo demanda, no se carga todo siempre

L2 — MEMORIA EPISÓDICA (Qué ha pasado)
     docs/specs/          → Contratos de qué se debe construir
     docs/plans/          → Cómo se decidió construirlo
     docs/audits/         → Qué se verificó y con qué resultado
     docs/bugs/           → Qué se rompió y cómo se reparó
     docs/sessions/       → Snapshots para retomar trabajo
     docs/adr/            → Por qué se tomaron decisiones clave
     docs/prd/            → Requisitos del cliente (aprobados)
     docs/blueprint/      → Plano técnico por proyecto

L3 — MEMORIA DE TRABAJO (Contexto activo)
     La ventana de contexto del modelo en la sesión actual
     → Volátil, limitada, el recurso más valioso
     → @QwikMemory la optimiza cuando se satura
```

**Principio de diseño:** Los agentes siempre deben poder responder:
"¿Por qué está esto así?" mirando L2. Nunca debe ser un misterio.

---

## 🔄 El Ciclo Completo SDD

### Fase -1: Blueprint (`/blueprint`)
**Agente:** @QwikBlueprint
**Input:** PRD aprobado por el cliente en `docs/prd/[proyecto]-prd.md`
**Output:** `docs/blueprint/[proyecto]-blueprint.md` aprobado

El Blueprint traduce el PRD en un plano técnico ejecutable antes de que
cualquier Spec se escriba. Define los módulos, sus dependencias, las fases
de entrega y las decisiones arquitectónicas globales.

El agente toma las decisiones que puede inferir del PRD (estructura de rutas, orden de fases, lib/ vs features/) y pregunta al desarrollador solo lo que genuinamente necesita decidir (proveedor de pagos, auth,multi-idioma).

**Gate:** Sin Blueprint aprobado → `/spec` no debería ejecutarse en proyectos nuevos.

---

### Fase 0: Spec (`/spec`)
**Agente:** @QwikSpeccer
**Input:** Requisito humano (lenguaje natural)
**Output:** `docs/specs/[feature].md` con estado 🟢 Approved

El Speccer no adivina. Hace preguntas y define:
- QUÉ problema resuelve (contexto)
- PARA QUIÉN (usuarios afectados)
- QUÉ se construye exactamente (scope IN)
- QUÉ NO se construye (scope OUT)
- CÓMO se verifica (Acceptance Criteria)
- QUÉ datos maneja (contratos)

**Gate:** Sin Spec Approved → `/feature` no puede ejecutarse.

---

### Fase 1: Plan (`/feature` → @QwikArchitect)
**Agente:** @QwikArchitect (+ @QwikDBA si hay cambios DB)
**Input:** Spec aprobada
**Output:** `docs/plans/[feature].md` completo

El Arquitecto traduce el WHAT en HOW:
- Qué archivos crear/modificar
- Qué fronteras `$()` diseñar
- Qué estado serializar (mínimo)
- Qué handlers co-localizar

**Principio:** Si el Arquitecto no puede planificar sin ambigüedad, la Spec está incompleta.
Debe volver a @QwikSpeccer, no inventar lo que falta.

---

### Fase 2: Construcción (@QwikBuilder)
**Input:** Plan aprobado + Schema migrado (si aplica)
**Output:** Código en `src/`

El Builder implementa el Plan, no interpreta la Spec directamente.
Si el Plan tiene un gap, escala a @QwikArchitect, no improvisa.

---

### Fase 3: Auditoría (@QwikAuditor)
**Input:** Código + Spec (para AC) + Plan (para decisiones)
**Output:** `docs/audits/[feature]-audit.md`

La auditoría tiene DOS dimensiones:
1. **Spec Compliance:** ¿El código cumple los AC de la Spec?
2. **Technical Quality:** ¿El código cumple los standards técnicos?

Ambas deben ser PASSED. Una sin la otra no es suficiente.

**Límite de ciclos:** 2 ciclos Builder↔Auditor máximo.
Si en el ciclo 3 hay errores críticos: problema de diseño → @QwikArchitect.

---

### Fase 4: Production (@QwikPolisher)
**Input:** Audit PASSED + Plan File
**Output:** Plan File cerrado con métricas + PRODUCTION-READY

El Polisher no puede actuar sin Audit PASSED. Es la última línea de defensa
antes de que el código llegue a producción.

---

## 📊 Trazabilidad Completa

Cada feature puede reconstruirse completamente desde los artefactos:

```
¿Qué quería el cliente?    → docs/prd/[proyecto]-prd.md
¿Cómo se planificó?        → docs/blueprint/[proyecto]-blueprint.md
¿Qué se construyó?         → docs/specs/[feature].md (los AC)
¿Cómo se decidió?          → docs/plans/[feature].md (las decisiones)
¿Por qué así?              → docs/adr/ADR-NNN-*.md (las razones)
¿Pasó la calidad?          → docs/audits/[feature]-audit.md
¿Cómo quedó en prod?       → docs/plans/[feature].md (Estado Final)
```

Esto no es burocracia — es la diferencia entre un sistema mantenible en 6 meses
y uno que nadie entiende por qué está como está.

---

## 🚫 Anti-Patrones SDD

| Anti-patrón | Síntoma | Consecuencia |
|---|---|---|
| Blueprint ausente | Specs sin orden ni dependencias claras | Features construidas en el orden equivocado, retrabajos |
| Spec Vaga | "Hacer que funcione el login" | El Builder improvisa, el Auditor no puede verificar |
| Spec Post-hoc | Escribir la Spec después del código | Pierde el propósito; los AC son descripción, no contrato |
| Plan sin Spec | @QwikArchitect inventa el WHAT | Código correcto para el problema equivocado |
| Auditor sin AC | Solo verifica calidad técnica | Feature técnicamente correcta pero funcionalmente incorrecta |
| Memoria no persistida | No usar @QwikMemory | Decisiones perdidas, ciclos repetidos, deuda de contexto |

---

## ✅ La Pregunta de Oro

Antes de empezar cualquier tarea, pregúntate:

> "¿Tengo una Spec aprobada que define exactamente qué debo verificar
> cuando termine?"

Si la respuesta es No → `/spec` primero.
Si la respuesta es Sí → continúa con confianza.