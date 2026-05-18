---
# EXTERNAL_AGENT_PATH: ".github/prompts/spec.prompt.md"
name: spec
description: >
  Punto de entrada del ciclo SDD. Invoca a @QwikSpeccer para crear una
  Especificación Formal antes de cualquier feature. Prerequisito obligatorio
  para /feature.
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "example: /spec mieleshuelva-configuration"
---

# 📐 SPEC KICKOFF: `${input:featureName}`

**Objetivo:** Crear la Especificación Formal para `${input:featureName}` antes de iniciar ningún trabajo de implementación.

> ⚠️ **SDD Gate:** Sin Spec aprobada, `/feature` no puede ejecutarse.

---

## Paso 1: Contexto previo

Antes de invocar al Speccer, recopila el contexto necesario:

```bash
# 1. ¿Existe Blueprint del proyecto?
ls docs/blueprint/*.md 2>/dev/null | head -3 || echo "No hay blueprints"

# 2. Features relacionadas en INDEX
grep -i "${input:featureName}" docs/sessions/INDEX.md 2>/dev/null || echo "No hay features relacionadas"
```

Si existe Blueprint, la Spec debe alinearse con el módulo/fase en términos de estructura, contenido y nomenclatura.
Si INDEX tiene features del mismo dominio, evitar duplicación.

## Paso 2: Preparar el Workspace

```bash
mkdir -p docs/specs docs/sessions
```

## Paso 3: Registrar en INDEX

```bash
# Añadir nueva spec al INDEX
echo "| ${input:featureName} | | 🟡 Spec | | | | Creada $(date '+%Y-%m-%d') |" >> docs/sessions/INDEX.md
echo "Spec registrada en INDEX"
```

## Paso 4: Invocar @QwikSpeccer

Llama a **@QwikSpeccer** con este contexto:

> "Crea la Spec formal para la feature `${input:featureName}`.
>
> **Tu tarea:**
> 1. Crea `docs/specs/${input:featureName}.md` usando la plantilla canónica de @QwikSpeccer.
> 2. Antes de escribir, lee:
>    - `docs/standards/ARQUITECTURA-FOLDER.md` (para entender las capas disponibles)
>    - `docs/standards/RBAC-ROLES-PERMISSIONS.md` (si la feature involucra usuarios/roles)
>    - Cualquier spec del mismo dominio en `docs/specs/` (para consistencia)
> 3. Define la Spec siguiendo estos sub-pasos en orden:
>    - **3a. Acceptance Criteria funcionales:** mínimo 3, formulados como condiciones binarias verificables (pasa/no pasa).
>    - **3b. Acceptance Criteria no-funcionales:** mínimo 3, cubriendo al menos performance, seguridad y a11y.
>    - **3c. Contratos de datos:** DTOs completos de entrada y salida, sin tipos no serializables.
>    - **3d. Scope OUT:** lista explícita de qué NO se construye en esta Spec.
>    - **3e. Análisis de impacto:** listado de archivos nuevos y modificados estimados.
> 4. Presenta al usuario el resumen de la Spec y solicita aprobación.
> 5. Solo tras aprobación explícita: cambia estado a 🟢 Approved y handoff a @QwikArchitect.
>
> **SDD Principle:** La calidad de la Spec determina la calidad de todo lo que viene después.
> Mejor invertir 20 minutos en una Spec clara que 2 horas corrigiendo malentendidos."

---

## Criterios de Calidad de la Spec

Antes de aprobar, el usuario debe verificar:
- [ ] ¿Entiendo exactamente qué se va a construir y qué NO?
- [ ] ¿Los Acceptance Criteria son verificables sin ambigüedad?
- [ ] ¿Los contratos de datos son completos y sin tipos no serializables?
- [ ] ¿El análisis de impacto identifica todos los archivos afectados?

---

**Nota:** La Spec es un documento vivo durante el Planning. Una vez que @QwikBuilder empieza a implementar, los cambios en la Spec requieren un nuevo ciclo de aprobación del Plan.