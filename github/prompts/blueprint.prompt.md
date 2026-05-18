---
# EXTERNAL_AGENT_PATH: ".github/prompts/blueprint.prompt.md"
name: blueprint
description: >
  Genera el Blueprint técnico a partir del PRD aprobado. Invoca a @QwikBlueprint para descomponer la aplicación en módulos, definir fases de entrega y preparar el terreno para el primer /spec.
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "example: /blueprint mieleshuelva"
---

# 🗺️ BLUEPRINT KICKOFF: `${input:projectName}`

**Prerequisito:** Debe existir `docs/prd/${input:projectName}-prd.md` aprobado por el cliente.

---

## Verificación del prerequisito

```bash
ls docs/prd/${input:projectName}-prd.md 2>/dev/null || echo "PRD no encontrado"
```

Si el PRD no existe o no está aprobado:
> "No existe PRD aprobado para `${input:projectName}`.
> Completa primero el PRD usando `docs/templates/PRD-TEMPLATE.md`."

---

## Preparar el directorio

```bash
mkdir -p docs/blueprint
```

---

## Invocar a @QwikBlueprint

> "Genera el Blueprint para el proyecto `${input:projectName}`.
>
> **Tu tarea:**
> 1. Lee `docs/prd/${input:projectName}-prd.md` completo.
> 2. Lee los standards necesarios: `ARQUITECTURA-FOLDER`, `RBAC-ROLES-PERMISSIONS`, `DECISIONS-DATA`.
> 3. Presenta los módulos identificados y las decisiones pendientes antes de generar el Blueprint.
> 4. Haz las preguntas que necesites (una a una) para las decisiones que no
>    puedes inferir del PRD.
> 5. Genera el Blueprint completo en `docs/blueprint/${input:projectName}-blueprint.md`
>    usando `docs/templates/BLUEPRINT-TEMPLATE.md` como base.
> 6. Solicita aprobación al desarrollador.
> 7. Tras la aprobación, devuelve control a `@QwikOrchestrator` para enrutar
>    al primer módulo de Fase 0.
> 8. Si no existe `docs/sessions/INDEX.md`, activa `@QwikMemory` para
>    inicializarlo con el proyecto y su Blueprint antes del handoff."

---

## Salida esperada

- `docs/blueprint/${input:projectName}-blueprint.md` generado y aprobado
- `docs/sessions/INDEX.md` inicializado o actualizado con el proyecto

---

**Nota:** El Blueprint es el último paso antes de entrar en el ciclo SDD Qwik.
Con PRD + Blueprint aprobados, cada `/spec` tiene contexto completo y
cada `/feature` tiene un orden claro de implementación.