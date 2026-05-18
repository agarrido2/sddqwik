---
# EXTERNAL_AGENT_PATH: ".github/prompts/new-feature.prompt.md"
name: new-feature
description: >
  Inicia el ciclo de construcción de una feature. Prerequisito: Spec aprobada en `docs/specs/${input:featureName}.md`. Si no existe la Spec, detén la ejecución y redirige a `/spec ${input:featureName}` primero; no continúes hasta que la Spec esté aprobada.
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "example: /new-feature voice-agent-configuration"
---

# 🚀 FEATURE KICKOFF: `${input:featureName}`

**Prerequisito SDD:** Antes de ejecutar este prompt, completa los siguientes checks en orden. Si alguno falla, detén la ejecución y sigue la acción indicada:

| # | Check | Acción si falla |
|---|---|---|
| 1 | `docs/specs/${input:featureName}.md` existe con estado `🟢 Approved` | Ejecuta `/spec ${input:featureName}` y obtén aprobación antes de continuar |
| 2 | Las dependencias de la feature en `docs/sessions/INDEX.md` están en estado `✅ Done` | Completa los módulos bloqueantes antes de continuar |

---

## Verificación del SDD Gate

Antes de continuar, verifica:

```bash
cat docs/specs/${input:featureName}.md 2>/dev/null | grep "Estado" | head -1
```

Si el resultado NO es `🟢 Approved`:
> "SDD GATE BLOQUEADO: No existe Spec aprobada para `${input:featureName}`.
> Ejecuta `/spec ${input:featureName}` primero y obtén aprobación del usuario."

---

## Paso 0: Consultar el INDEX

```bash
cat docs/sessions/INDEX.md
```

Antes de crear el Plan File, verifica:
- ¿Hay features relacionadas o dependencias que este módulo necesita?
- ¿Hay tablas DB ya creadas (`Tablas DB`) que esta feature deba reutilizar?
- ¿Hay servicios ya expuestos (`Expone`) que eviten duplicar lógica?

Documenta las dependencias identificadas en el Plan File.

---

## Paso 1: Crear el Plan File

```bash
mkdir -p docs/plans
```

Crea `docs/plans/${input:featureName}.md` con estado inicial `🟡 Planning`
y referencia a la Spec aprobada:
```md
# Plan: ${input:featureName}

> Estado: 🟡 Planning
> Spec: docs/specs/${input:featureName}.md
> Fecha: [YYYY-MM-DD]

## 📨 Handoff Log
```

`@QwikArchitect` completará el Plan File con el diseño técnico completo
en su fase de planificación.

---

## Paso 2: Invocar a @QwikOrchestrator

> "Inicia el ciclo de construcción para la feature `${input:featureName}`.
>
> **Contexto:**
> - Spec aprobada: `docs/specs/${input:featureName}.md`
> - Plan File creado: `docs/plans/${input:featureName}.md`
>
> **Tu tarea:**
> 1. Lee la Spec aprobada y el Plan File recién creado.
> 2. Determina el routing correcto según tu tabla de decisiones.
> 3. Registra el Handoff inicial en el Plan File.
> 4. Invoca a `@QwikArchitect` con el contexto completo.
>
> **Flujo esperado:**
> `@QwikArchitect` → (`@QwikDBA` si hay cambios DB) → `@QwikBuilder` → `@QwikAuditor` → `@QwikPolisher` → `@QwikMemory`"

---

**Nota para el Orchestrator:** El Plan File es la memoria compartida del equipo.
Cada agente debe actualizarlo al inicio y al final de su fase.
Cada handoff debe quedar registrado en `## 📨 Handoff Log`.

El Delivery Summary del Builder (incluidos tests) es obligatorio antes de cada auditoría.