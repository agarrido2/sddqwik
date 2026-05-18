---
# EXTERNAL_AGENT_PATH: ".github/prompts/memory-compact.prompt.md"
name: memory-compact
description: >
  Comprime el contexto de trabajo activo y guarda un snapshot de sesión
  para poder retomar sin perder información. Invocar cuando el contexto
  está cerca del límite o al terminar una sesión de trabajo larga.
tools: ["read", "edit", "execute/runInTerminal"]
argument-hint: "example: /memory-compact voice-agent-feature"
---

# 🧠 MEMORY COMPACT: `${input:featureName}`

> **Prerequisito:** Si `${input:featureName}` no se proporciona o no existe ningún artefacto activo con ese nombre en `docs/specs/` o `docs/plans/`, responde con:
> `"No se encontraron artefactos activos para '${input:featureName}'. Verifica el nombre e inténtalo de nuevo."` y detén la operación.

**Objetivo:** Comprimir el contexto activo de la feature `${input:featureName}`
y guardar un snapshot operativo de reentrada, actualizado en `docs/sessions/INDEX.md`,
para poder retomar en un chat nuevo sin pérdida de señal útil.

---

## Invocar a @QwikMemory

> "Ejecuta la operación Memory Compact para la feature `${input:featureName}`.
>
> **Tu tarea:**
>
> 1. Lee únicamente los artefactos activos y necesarios de la feature:
>    - `docs/specs/${input:featureName}.md`
>    - `docs/plans/${input:featureName}.md`
>    - `docs/audits/${input:featureName}-audit.md` (si existe y aplica)
>
> 2. Identifica y preserva solo la señal útil de continuidad:
>    - Estado exacto del trabajo y de la fase actual
>    - Estado verificable del Plan File y checklist relevante
>    - Decisiones técnicas vigentes que no deben reabrirse sin motivo
>    - Problemas encontrados y su resolución, si afectan a la reentrada
>    - Riesgos, bloqueos o incertidumbres aún abiertos
>    - Próximo paso exacto, concreto y ejecutable
>    - Agente sugerido para retomar
>
> 3. Crea un snapshot operativo en:
>    - `docs/sessions/${input:featureName}-[timestamp].md`
>
>    El snapshot debe cumplir el contrato de `QwikMemory` y responder con claridad:
>    - qué se estaba haciendo;
>    - por qué se estaba haciendo;
>    - en qué fase quedó;
>    - qué artefactos gobiernan ese trabajo;
>    - qué decisiones siguen vigentes;
>    - qué bloqueo o riesgo queda abierto;
>    - cuál es el siguiente paso exacto;
>    - qué agente debe tomar el relevo.
>
>    **Reglas del snapshot (contenido):**
>    - No guardes conversación literal.
>    - No uses placeholders.
>    - No dejes secciones vacías: elimínalas o marca `N/A` de forma explícita.
>
>    **Reglas del snapshot (alineación):**
>    - El snapshot debe estar alineado con la Spec, el Plan y el INDEX.
>    - Si algo no aplica al estado actual, elimínalo o marca `N/A`.
>
> 4. Actualiza `docs/sessions/INDEX.md` como fuente de verdad colectiva,
>    siguiendo esta tabla según el estado de la feature:
>
>    | Estado de la feature | Acción en INDEX.md |
>    |---|---|
>    | `WIP` | Añade o actualiza la fila con el snapshot recién creado |
>    | `Blocked` | Añade o actualiza la fila indicando el bloqueo en la nota breve |
>    | `Ready for Handoff` | Añade o actualiza la fila indicando el agente de relevo |
>    | `Done` o `Archived` | Verifica que la fila existe y sigue siendo localizable; no la elimines |
>
>    La fila debe dejar visible, como mínimo:
>      - feature o frente;
>      - estado actual;
>      - fase actual;
>      - artefacto principal;
>      - último snapshot útil;
>      - siguiente agente;
>      - dependencias;
>      - nota breve de routing
>
> 5. Si durante la compactación detectas una decisión estructural no formalizada,
>    crea o propone el ADR correspondiente en `docs/adr/`.
>
> 6. Si detectas una señal reusable real, márcala en el snapshot para promoción posterior:
>    - Lessons learned
>    - ADR candidate
>    - Bug formalizable
>
> 7. Genera al final del snapshot un bloque obligatorio llamado `## Prompt de Reanudación`
>    con este propósito:
>    - servir como puente explícito entre `/memory-compact` y `/new-session`;
>    - apuntar al snapshot recién creado como contexto prioritario de reentrada;
>    - evitar que la reanudación dependa de memoria humana.
>
>    Usa este formato exacto:
>
>    ```md
>    ## Prompt de Reanudación
>
>    /new-session
>
>    Reanudar feature: `${input:featureName}`
>    Snapshot prioritario: `docs/sessions/${input:featureName}-[timestamp].md`
>    Verificar contra índice: `docs/sessions/INDEX.md`
>    Objetivo inmediato: [siguiente paso exacto]
>    Agente sugerido tras reentrada: [@QwikOrchestrator | @QwikBuilder | @QwikAuditor | @QwikArchitect | @QwikDBA | @QwikBugFix | @QwikPolisher]
>    ```
>
>    Regla: el snapshot recién creado es la referencia prioritaria de reentrada;
>    `docs/sessions/INDEX.md` sigue siendo la fuente de verdad colectiva y debe
>    usarse para validar contexto, estado y routing.
>
> 8. Confirma al usuario que el contexto quedó guardado y devuelve:
>    - la ruta exacta del snapshot creado;
>    - confirmación de actualización del `INDEX.md`;
>    - el `Prompt de Reanudación` listo para copiar;
>    - recordatorio explícito de abrir un chat nuevo antes de continuar."

---

## Criterios obligatorios

- No explores el repositorio completo ni archivos no relacionados con la feature actual.
- No cargues artefactos ajenos a la feature, a menos que estén explícitamente referenciados como dependencias en los artefactos de la feature actual.
- No conviertas el snapshot en un resumen narrativo.
- No reemplaces `docs/sessions/INDEX.md` por el prompt de reanudación.
- El prompt de reanudación no sustituye al índice: lo complementa.
- El objetivo no es conservar toda la conversación, sino permitir el siguiente paso correcto con contexto mínimo y fiable.

---

## Cuándo usar este prompt

- El contexto de la sesión supera ~60% del límite estimado
- Vas a cerrar el IDE o terminar la sesión de trabajo
- Quieres hacer un checkpoint antes de una operación arriesgada
- Cambias de feature y quieres guardar el estado de la actual
- Se ha producido una decisión relevante que no debe perderse aunque aún no haya cierre de fase

---

## Resultado esperado

Al terminar, deben existir y estar alineados:

1. `docs/sessions/${input:featureName}-[timestamp].md`
2. `docs/sessions/INDEX.md` actualizado
3. un `Prompt de Reanudación` copiable, vinculado al snapshot recién creado

La compactación solo está bien hecha si un chat nuevo puede reanudar la feature
sin explorar el repositorio a ciegas y sin depender de la conversación anterior.