---
# EXTERNAL_AGENT_PATH: ".github/prompts/blueprint.prompt.md"
name: blueprint
description: >
  LEGACY / DISABLED en spec-first-garrido. /blueprint está fuera del flujo
  operativo principal y no debe ejecutarse. Si se invoca, debe detenerse y
  redirigir a /spec [feature].
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "Legacy desactivado: usar /spec [feature]"
---

# LEGACY NOTICE — /blueprint DESACTIVADO

Estado: Legacy / Disabled / fuera del flujo operativo principal / pendiente de redefinir.

Flujo operativo principal vigente:

Spec -> Plan -> Implementation Tasks -> Build -> Audit -> Polish -> Memory.

PRD y Blueprint no forman parte del sistema operativo principal.

Si alguien invoca /blueprint, este prompt debe detenerse de inmediato y responder solo:

```text
/blueprint está desactivado en spec-first-garrido.
No es un comando activo y no forma parte del flujo operativo principal.
No crea gates, no crea contexto obligatorio y no bloquea Specs.
Entrada principal recomendada: /spec [feature]
```

Reglas obligatorias para esta invocación:

- no ejecutar pasos operativos de Blueprint;
- no recomendar /blueprint como siguiente paso;
- no crear ni actualizar gates de aprobación de Blueprint;
- no imponer Blueprint como prerrequisito de Specs;
- no bloquear /spec por ausencia de Blueprint.

## Referencia histórica (no ejecutar)

Todo lo que sigue se conserva solo como referencia documental legacy.
No debe ejecutarse, no define flujo activo y no redefine todavía el futuro de Blueprint.

## Referencia histórica mínima

Se conserva únicamente como señal histórica de que este comando existió para trabajo previo con PRD y Blueprint.

Resumen histórico no operativo:

- antes se usaba para derivar un mapa técnico desde PRD;
- antes definía módulos, fases y orden de Specs;
- antes pedía estado de aprobación para Blueprint.

Nada de lo anterior es ejecutable en el modo actual.
