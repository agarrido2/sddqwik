---
# EXTERNAL_AGENT_PATH: ".github/prompts/blueprint.prompt.md"
name: blueprint
description: >
  Genera o revisa el Blueprint técnico a partir de un PRD aprobado. Obliga a
  validar PRD, descubrir módulos, fases, dependencias, roles, datos, riesgos,
  orden de Specs y criterios de entrega antes de iniciar /spec. Sin Blueprint
  aprobado, no debe arrancar un proyecto modular grande.
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "example: /blueprint mieleshuelva"
---

# 🗺️ BLUEPRINT KICKOFF — `${input:projectName}`

## Propósito

`/blueprint` transforma un PRD aprobado en un mapa técnico y operativo para construir el proyecto por módulos.

No implementa código.
No crea Specs detalladas.
No aprueba decisiones abiertas por cuenta propia.
No sustituye a Architect para planes técnicos de feature.
No mezcla todo el proyecto en una sola Spec.

Su objetivo es producir una guía verificable para el ciclo SDD:

```text
PRD Approved → Blueprint → Módulos → Fases → Dependencias → Orden de Specs → /spec → /new-feature
```

---

## Regla operativa crítica

Un Blueprint solo queda listo si define:

1. PRD fuente aprobado;
2. visión técnica de alto nivel;
3. módulos funcionales;
4. fases de entrega;
5. dependencias entre módulos;
6. roles y permisos globales;
7. mapa preliminar de datos;
8. mapa de rutas/zonas;
9. riesgos y decisiones abiertas;
10. orden recomendado de Specs;
11. criterios de MVP y post-MVP;
12. estado explícito `🟢 Approved` tras aprobación del usuario.

Si falta alguna pieza esencial, el Blueprint debe quedar en `Review`, no `Approved`.

---

## Prohibiciones

Durante `/blueprint`, no hacer:

- escribir código en `src/`;
- crear Specs detalladas;
- crear planes técnicos de implementación;
- crear schema/migraciones;
- aprobar el Blueprint sin confirmación del usuario;
- resolver dudas críticas inventando alcance;
- leer todo el repositorio;
- listar todas las specs o plans;
- bajar a detalles de Builder;
- convertir un PRD ambiguo en arquitectura cerrada sin preguntas.

---

## Paso 0 — Validar entrada

Usar `${input:projectName}` como identificador canónico del proyecto.

```bash
PROJECT="${input:projectName}"

if [ -z "$PROJECT" ]; then
  echo "BLUEPRINT GATE BLOQUEADO: falta projectName."
  echo "Usa: /blueprint [project-name]"
  exit 1
fi

echo "Blueprint objetivo: $PROJECT"
```

---

## Paso 1 — Verificar PRD aprobado

Ruta esperada:

```text
docs/prd/${input:projectName}-prd.md
```

Comprobación:

```bash
PRD_FILE="docs/prd/${PROJECT}-prd.md"

if [ ! -f "$PRD_FILE" ]; then
  echo "BLUEPRINT GATE BLOQUEADO: no existe PRD para ${PROJECT}."
  echo "Ruta esperada: $PRD_FILE"
  echo "Siguiente paso: crear PRD usando docs/templates/PRD-TEMPLATE.md"
  exit 1
fi

PRD_STATUS=$(grep -E '^>?[[:space:]]*Estado:|^>?[[:space:]]*Status:' "$PRD_FILE" | head -1 || true)

echo "PRD: $PRD_FILE"
echo "Estado detectado: ${PRD_STATUS:-NO ENCONTRADO}"

if ! echo "$PRD_STATUS" | grep -Eiq '(🟢[[:space:]]*)?Approved|Aprobado|Aprobada'; then
  echo "BLUEPRINT GATE BLOQUEADO: el PRD existe pero no está Approved."
  echo "Siguiente paso: revisar/aprobar PRD antes de generar Blueprint."
  exit 1
fi
```

### Regla

No generar Blueprint formal desde PRD en Draft, Review o sin estado explícito.

Si el usuario quiere explorar ideas sin PRD aprobado, usar conversación normal o completar PRD primero.

---

## Paso 2 — Preparar workspace mínimo

```bash
mkdir -p docs/blueprint
mkdir -p docs/sessions
```

Si falta `docs/sessions/INDEX.md`, no bloquear necesariamente; el Blueprint puede inicializar el proyecto, pero debe dejar señal para `@QwikMemory`.

```bash
if [ ! -f docs/sessions/INDEX.md ]; then
  echo "ADVERTENCIA: falta docs/sessions/INDEX.md"
  echo "@QwikMemory deberá inicializarlo tras Blueprint."
fi
```

---

## Paso 3 — Detectar Blueprint existente

Ruta esperada:

```text
docs/blueprint/${input:projectName}-blueprint.md
```

```bash
BLUEPRINT_FILE="docs/blueprint/${PROJECT}-blueprint.md"

if [ -f "$BLUEPRINT_FILE" ]; then
  echo "Blueprint existente: $BLUEPRINT_FILE"
  grep -E '^>?[[:space:]]*Estado:|^>?[[:space:]]*Status:' "$BLUEPRINT_FILE" | head -1 || true
fi
```

Si ya existe y está `Approved`, no sobrescribir.

Acciones permitidas:

```text
- si el usuario cambia alcance del proyecto: volver a Review;
- si solo ajusta fases o orden: actualizar con historial;
- si afecta Specs ya aprobadas: advertir impacto y pedir confirmación;
- si afecta features ya construidas: derivar a Orchestrator/Architect antes de cambiar.
```

---

## Paso 4 — Carga mínima permitida

Cargar:

```text
docs/prd/${input:projectName}-prd.md
docs/templates/BLUEPRINT-TEMPLATE.md
docs/standards/ARQUITECTURA-FOLDER.md
docs/standards/PROJECT-RULES-CORE.md
docs/standards/DECISIONS-QWIK.md
docs/standards/DECISIONS-DATA.md
docs/standards/RBAC-ROLES-PERMISSIONS.md
docs/standards/SECURITY-POLICIES.md
docs/standards/UX-GUIDE.md
```

Cargar si existe y es relevante:

```text
docs/sessions/INDEX.md
Blueprint existente del mismo proyecto
ADRs citadas por el PRD
```

### Regla

No leer Specs o Plans salvo que el Blueprint existente ya tenga trabajo iniciado y el usuario pida revisión del Blueprint.

---

## Paso 5 — Discovery de decisiones abiertas

Antes de generar Blueprint completo, `@QwikBlueprint` debe identificar:

- objetivos MVP;
- objetivos post-MVP;
- usuarios y roles;
- zonas públicas/privadas/admin;
- módulos funcionales;
- datos principales;
- integraciones externas;
- riesgos;
- dudas bloqueantes;
- decisiones que no pueden inferirse del PRD.

Si hay dudas bloqueantes, preguntar antes de cerrar Blueprint.

### Regla de preguntas

Hacer preguntas agrupadas y mínimas.
No alargar indefinidamente.

Formato recomendado:

```text
BLUEPRINT DISCOVERY — decisiones necesarias

1. [pregunta crítica]
2. [pregunta crítica]
3. [pregunta opcional]

Puedo generar un Blueprint en Review con supuestos marcados, o esperar tus respuestas para cerrarlo mejor.
```

---

## Paso 6 — Invocar a @QwikBlueprint

Mensaje de handoff:

```text
@QwikBlueprint

Genera o revisa el Blueprint técnico para `${input:projectName}`.

Contexto obligatorio:
- PRD aprobado: docs/prd/${input:projectName}-prd.md
- Template: docs/templates/BLUEPRINT-TEMPLATE.md
- Standards base: ARQUITECTURA-FOLDER, PROJECT-RULES-CORE, DECISIONS-QWIK, DECISIONS-DATA
- Standards condicionales: RBAC, SECURITY, UX si aplica

Tarea:
1. Leer el PRD aprobado.
2. Identificar módulos funcionales.
3. Definir zonas de aplicación: pública, privada, admin, API/webhooks si aplica.
4. Definir fases de entrega.
5. Definir dependencias entre módulos.
6. Crear mapa preliminar de datos y ownership de entidades.
7. Identificar roles/permisos globales.
8. Identificar integraciones externas.
9. Definir riesgos y decisiones abiertas.
10. Proponer orden recomendado de Specs.
11. Definir criterios de MVP y post-MVP.
12. Crear o actualizar `docs/blueprint/${input:projectName}-blueprint.md`.
13. Dejar estado `🟡 Review` hasta aprobación explícita.
14. Solo tras aprobación, cambiar a `🟢 Approved`.

Restricciones:
- No escribir código.
- No crear Specs todavía.
- No diseñar schema definitivo.
- No cerrar decisiones ambiguas sin señalarlas.
- No mezclar módulos independientes en una sola fase si dificulta auditoría.
```

---

## Paso 7 — Estructura obligatoria del Blueprint

Crear `docs/blueprint/${input:projectName}-blueprint.md` con esta estructura mínima:

```md
# Blueprint: ${input:projectName}

> Estado: 🟡 Review
> Fecha: [YYYY-MM-DD]
> PRD fuente: docs/prd/${input:projectName}-prd.md
> Owner: [usuario/cliente]

## 1. Visión del proyecto

[Resumen técnico-funcional del proyecto]

## 2. Objetivos MVP

- [objetivo]

## 3. Fuera de alcance inicial

- [scope out global]

## 4. Usuarios, roles y permisos globales

| Rol | Descripción | Permisos globales | Restricciones |
|---|---|---|---|

## 5. Zonas de aplicación

| Zona | Propósito | Acceso | Notas |
|---|---|---|---|
| Pública |  |  |  |
| Privada |  |  |  |
| Admin |  |  |  |
| API/Webhooks |  |  |  |

## 6. Módulos funcionales

| Módulo | Propósito | Prioridad | Fase | Depende de | Produce Specs |
|---|---|---|---|---|---|

## 7. Fases de entrega

### Fase 0 — Base operativa

- [módulo]

### Fase 1 — MVP funcional

- [módulo]

### Fase 2 — Expansión

- [módulo]

### Fase 3 — Post-MVP

- [módulo]

## 8. Mapa preliminar de datos

| Entidad | Descripción | Owner funcional | Módulo | RLS requerida | Notas |
|---|---|---|---|---|---|

## 9. Integraciones externas

| Integración | Uso | Riesgo | Fase | Requiere Spec |
|---|---|---|---|---|

## 10. Orden recomendado de Specs

| Orden | Spec propuesta | Módulo | Depende de | Motivo |
|---|---|---|---|---|

## 11. Riesgos y decisiones abiertas

| Riesgo/Decisión | Impacto | Responsable | Resolver antes de |
|---|---|---|---|

## 12. Criterios de entrada a /spec

Una Spec puede iniciarse cuando:

- el módulo está definido;
- su dependencia anterior está clara;
- el Scope IN/OUT puede separarse;
- no depende de una decisión abierta bloqueante.

## 13. Criterios de MVP completado

- [criterio verificable]

## 14. Historial de aprobación

- [YYYY-MM-DD] — Estado inicial: Review
- [YYYY-MM-DD] — Aprobación usuario: pendiente
```

---

## Paso 8 — Control de tamaño y fases

Antes de pedir aprobación, verificar:

- ¿hay módulos demasiado grandes?
- ¿alguna fase mezcla demasiados dominios?
- ¿alguna Spec propuesta parece una aplicación completa?
- ¿hay dependencias circulares?
- ¿hay decisiones abiertas que bloquean Fase 0 o Fase 1?

Si algo falla, dejar Blueprint en `Review`.

---

## Paso 9 — Aprobación explícita

El Blueprint debe quedar inicialmente en:

```text
> Estado: 🟡 Review
```

Solo cambiar a:

```text
> Estado: 🟢 Approved
```

cuando el usuario apruebe explícitamente.

Frases válidas:

```text
aprobado
approved
sí, adelante
lo apruebo
ok, apruébalo
```

Si el usuario pide cambios, mantener `Review`.

---

## Paso 10 — Actualizar INDEX

Si existe `docs/sessions/INDEX.md`, añadir o actualizar entrada del proyecto.

No duplicar filas.
No marcar como `Approved` si el Blueprint sigue en Review.

Formato recomendado:

```md
| ${input:projectName} | Proyecto | 🟡 Blueprint Review / 🟢 Blueprint Approved | PRD | N/A | Specs pendientes | Blueprint | [fase siguiente] | [YYYY-MM-DD] |
```

Si no existe INDEX, activar `@QwikMemory` tras crear Blueprint para inicializarlo.

---

## Paso 11 — Salida esperada

```text
BLUEPRINT REPORT — ${input:projectName}

Blueprint file: docs/blueprint/${input:projectName}-blueprint.md
Estado: Review / Approved
PRD fuente: docs/prd/${input:projectName}-prd.md
Módulos: N
Fases: N
Specs propuestas: N
Dependencias críticas: N
Decisiones abiertas: N
Riesgos: N
INDEX actualizado: sí / no / requiere @QwikMemory
Tamaño: OK / REVISAR FASES

Siguiente paso:
- revisar y aprobar Blueprint
- resolver decisiones abiertas
- ajustar fases/módulos
- /spec [primer-modulo] cuando Blueprint esté Approved
```

---

## Criterios de calidad antes de aprobar

El usuario debe poder responder sí a todo:

- ¿El MVP está claro?
- ¿Lo que queda fuera del MVP está claro?
- ¿Los módulos son separables?
- ¿El orden de Specs tiene sentido?
- ¿Las dependencias están claras?
- ¿Los roles/permisos globales están definidos si aplica?
- ¿Los datos principales están identificados sin sobrediseñar schema?
- ¿Las decisiones abiertas están visibles?
- ¿La primera Spec a crear es evidente?

---

## Regla final

`/blueprint` no significa “haz una arquitectura bonita”.

`/blueprint` significa:

```text
Convierte un PRD aprobado en un mapa de ejecución por Specs, fases y dependencias.
```

Si no deja claro qué Spec va primero y por qué, el Blueprint no está listo.
