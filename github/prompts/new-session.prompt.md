---
# EXTERNAL_AGENT_PATH: ".github/prompts/new-session.prompt.md"
name: new-session
description: >
  Arranca un chat nuevo reanudando el trabajo desde el snapshot operativo más reciente o desde el snapshot explícitamente indicado por `memory-compact`.
  Punto de entrada obligatorio tras abrir una conversación nueva.
  Si ni el snapshot ni el índice proporcionan información fiable, declara la inconsistencia y solicita confirmación al usuario; no procedan automáticamente hasta recibir respuesta.
tools: ["read"]
argument-hint: "example: /new-session (sin argumento reanuda frente activo)"
---

# 🔁 NEW SESSION

**Objetivo:** Reanudar una feature o frente de trabajo en un chat nuevo cargando únicamente el artefacto principal y las dependencias explícitamente necesarias para el siguiente paso, validando la fuente contra `docs/sessions/INDEX.md` y dejando el sistema listo para handoff a `@QwikOrchestrator`.

---

## Invocar a @QwikMemory

> "Ejecuta la operación New Session para reanudar trabajo en un chat nuevo.
>
> **Tu tarea:**
>
> 1. **Determina la fuente de reentrada** evaluando las condiciones de abajo en orden estricto — detente en la primera que se cumpla:
>
>    - **Condición A:** El usuario ha pegado o indicado un `Prompt de Reanudación` emitido por `/memory-compact` → usa el `Snapshot prioritario` indicado como referencia primaria.
>    - **Condición B:** No hay `Prompt de Reanudación` → lee `docs/sessions/INDEX.md` y busca una fila activa en **Frentes activos** → usa esa fila y localiza su último snapshot.
>    - **Condición C:** No hay frentes activos en el índice → busca entradas en "Sesiones / Snapshots" → usa el snapshot más reciente.
>    - **Condición D:** No hay snapshot explícito → usa la entrada activa o más reciente del índice como referencia mínima.
>    - **Condición E:** Ni snapshot ni índice ofrecen información fiable o son inconsistentes entre sí → declara la inconsistencia, detén la reconstrucción automática y solicita confirmación al usuario; no procedan hasta recibir respuesta.
>
> 2. **Verifica la coherencia de la fuente elegida:**
>    - Si usas un snapshot, confírmalo contra la fila correspondiente en `docs/sessions/INDEX.md` sección **Frentes activos**;
>    - si feature, estado o routing del snapshot contradicen el índice, prioriza el snapshot emitido por `memory-compact` si es más reciente y verificable, y deja constancia de que `INDEX.md` requiere actualización.
>
> 3. **Carga el contexto de reentrada** a partir de la fuente elegida. Carga únicamente:
>    - el artefacto principal (Spec o Plan activo de la feature);
>    - los artefactos que el snapshot o la fila del índice marquen explícitamente como necesarios para retomar;
>    - las dependencias explícitamente referenciadas como necesarias para el siguiente paso.
>    Excluye todo lo que no esté listado: specs o planes de otras features, sesiones archivadas, blueprints cerrados y auditorías no relacionadas.
>
> 5. **Reconstruye el estado operativo** necesario para continuar — a partir de los artefactos cargados, no de la conversación anterior:
>    - qué se estaba haciendo;
>    - por qué;
>    - en qué fase quedó;
>    - qué decisiones siguen vigentes;
>    - qué bloqueo o riesgo sigue abierto;
>    - cuál es el siguiente paso exacto;
>    - qué agente debe tomar el relevo.
>    No reconstruyas toda la historia ni explores el repo a ciegas.
>    El objetivo es situar el sistema en el siguiente paso correcto.
>
> 6. **Haz handoff a `@QwikOrchestrator`** con contexto mínimo y verificable,
>    incluyendo:
>    - fuente de reentrada usada (Frentes activos / Snapshot / Histórico);
>    - snapshot o fila de índice usada;
>    - artefactos cargados;
>    - siguiente paso exacto;
>    - agente recomendado por el snapshot o índice, si aplica;
>    - advertencias o prerequisitos abiertos.
>
> **Regla de sistema:**
> - `docs/sessions/INDEX.md` sección **Frentes activos** es la primera puerta de entrada cuando no hay snapshot explícito.
> - El `Snapshot prioritario` emitido por `memory-compact` es la referencia primaria de reentrada cuando exista.
> - `new-session` no sustituye al Orchestrator; prepara el contexto para su routing correcto."

---

## Criterios obligatorios

- Leer `docs/sessions/INDEX.md` siempre como primera validación estructural.
- Revisar sección **Frentes activos** antes de buscar en histórico.
- No hacer exploración masiva del repositorio.
- No cargar más contexto del necesario.
- No reabrir decisiones ya fijadas en el snapshot sin motivo explícito.
- No mezclar features activas por intuición.
- Si falta snapshot útil, usar el índice para localizar el frente correcto sin inventar contexto.
- Si no hay `INDEX.md`, derivar a `@QwikMemory` para inicialización de memoria colectiva.

---

## Salida esperada

Al terminar, `new-session` debe dejar preparado un handoff limpio hacia `@QwikOrchestrator` con esta información mínima:

- Feature o frente retomado.
- Fuente de reentrada usada (Frentes activos / Snapshot / Histórico).
- Snapshot o fila de índice usada como base.
- Entrada de `docs/sessions/INDEX.md` validada.
- Artefactos efectivamente cargados.
- Siguiente paso exacto.
- Agente que debería actuar ahora.
- Riesgos, bloqueos o prerequisitos abiertos.

---

## Formato recomendado del handoff interno

```json
{
  "handoff_id": "[feature]-[timestamp]",
  "resume_source": "prompt-snapshot | index-active-front | index-snapshot | index-entry",
  "snapshot_used": "docs/sessions/[feature]-[timestamp].md",
  "index_entry": "docs/sessions/INDEX.md#frentes-activos | docs/sessions/INDEX.md#[fila-o-feature]",
  "context_refs": [
    "docs/specs/[feature].md",
    "docs/plans/[feature].md",
    "docs/audits/[feature]-audit.md"
  ],
  "next_step": "[acción concreta y acotada]",
  "routing_decision": "@QwikOrchestrator",
  "suggested_next_agent": "@QwikBuilder | @QwikAuditor | @QwikArchitect | @QwikDBA | @QwikBugFix | @QwikPolisher | @QwikSpeccer",
  "warnings": [
    "[bloqueos, conflictos o prerequisitos abiertos]"
  ]
}
```

---

## Cuándo usar este prompt

- Has abierto un chat nuevo tras ejecutar `/memory-compact`
- Vas a retomar una feature y no quieres depender del historial anterior.
- Existe snapshot de sesión y quieres reconstruir el contexto mínimo correcto.
- El sistema debe volver al punto exacto donde quedó el trabajo, sin exploración masiva.

---

## Regla final

`/new-session` no existe para contar qué pasó.  
Existe para recuperar **el siguiente paso correcto** con el menor contexto posible, respetando el snapshot vigente, validando contra `docs/sessions/INDEX.md` sección **Frentes activos** como primera puerta de entrada, y entregando el routing a `@QwikOrchestrator`.
