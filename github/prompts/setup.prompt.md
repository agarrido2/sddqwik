---
# EXTERNAL_AGENT_PATH: ".github/prompts/setup.prompt.md"
name: setup
description: >
  Inicializa o verifica el workspace SDD Qwik. Crea la estructura de directorios,
  verifica los standards y templates canónicos y genera un health report del
  estado actual del proyecto usando el INDEX como fuente de verdad operativa.
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "example: /setup"
---

# 🧱 WORKSPACE SETUP / HEALTH CHECK — SDD Qwik

**Objetivo:** Inicializar el workspace desde cero o verificar que un workspace existente
está correctamente configurado para SDD Qwik.

**Regla operativa:** `/setup` no debe ejecutar `ls` recursivo, `find` sobre `src/`, ni leer artefactos fuera del flujo de pasos definido a continuación. Las acciones prohibidas específicamente son: listar directorios completos de `docs/specs/`, `docs/plans/` o `src/`; leer specs de features ya cerradas; y cargar sesiones archivadas. Debe usar `docs/sessions/INDEX.md` como primera fuente de verdad operativa para inspección controlada del workspace, tal como exige el Orchestrator.

**Orden de ejecución:** Completa cada paso en secuencia. No saltes a pasos posteriores hasta terminar el actual.

---

## Paso 1 — Crear estructura de directorios

```bash
# Documentación del proyecto
mkdir -p docs/prd
mkdir -p docs/blueprint
mkdir -p docs/templates

# Artefactos SDD generados por agentes
mkdir -p docs/specs
mkdir -p docs/plans
mkdir -p docs/audits
mkdir -p docs/bugs
mkdir -p docs/adr

# Memoria episódica
mkdir -p docs/sessions
mkdir -p docs/sessions/archive

# Standards
mkdir -p docs/standards

# Temporal y scripts
mkdir -p .scratch
mkdir -p scripts/db
```

---

## Paso 2 — Verificar `.gitignore`

Asegurar que estas entradas estén en `.gitignore`:

```gitignore
# SDD Qwik
.scratch
docs/sessions
.env
.env.local
.env.*.local
dist
node_modules
.qwik
.drizzle
```

---

## Paso 3 — Inicializar el INDEX de memoria

Si `docs/sessions/INDEX.md` no existe, créalo:

```bash
if [ ! -f docs/sessions/INDEX.md ]; then
  cat > docs/sessions/INDEX.md <<'EOF'
# Project Features Index

| Feature | Módulo | Estado | Depende de | Tablas DB | Expone | Resumen | Fecha |
|---------|--------|--------|------------|-----------|--------|---------|-------|

> Mantenido por @QwikMemory. No editar manualmente.
EOF
  echo "INDEX.md creado"
fi
```

---

## Paso 4 — Health check base del workspace

```bash
echo "SDD QWIK WORKSPACE HEALTH CHECK"

echo ""
echo "Estructura docs/"
for dir in prd blueprint templates specs plans audits bugs sessions adr standards; do
  if [ -d "docs/$dir" ]; then
    echo "OK  docs/$dir"
  else
    echo "FALTA  docs/$dir"
  fi
done

echo ""
echo "INDEX de memoria"
if [ -f "docs/sessions/INDEX.md" ]; then
  echo "OK  docs/sessions/INDEX.md"
else
  echo "FALTA  docs/sessions/INDEX.md"
fi
```

---

## Paso 5 — Verificar standards críticos

Los siguientes standards deben existir en `docs/standards/`:

- `ARQUITECTURA-FOLDER.md`
- `PROJECT-RULES-CORE.md`
- `SDD-WORKFLOW.md`
- `DECISIONS-QWIK.md`
- `DECISIONS-DATA.md`
- `DECISIONS-UI.md`
- `SERIALIZATION-CONTRACTS.md`
- `QUALITY-STANDARDS.md`
- `SECURITY-POLICIES.md`
- `TESTING-POLICY.md`
- `UX-GUIDE.md`
- `RBAC-ROLES-PERMISSIONS.md`
- `CONTEXT7-GUIDE.md`
- `LESSONS-LEARNED.md`

Verificación:

```bash
standards_ok=0

for std in \
  ARQUITECTURA-FOLDER \
  PROJECT-RULES-CORE \
  SDD-WORKFLOW \
  DECISIONS-QWIK \
  DECISIONS-DATA \
  DECISIONS-UI \
  SERIALIZATION-CONTRACTS \
  QUALITY-STANDARDS \
  SECURITY-POLICIES \
  TESTING-POLICY \
  UX-GUIDE \
  RBAC-ROLES-PERMISSIONS \
  CONTEXT7-GUIDE \
  LESSONS-LEARNED
do
  if [ -f "docs/standards/${std}.md" ]; then
    echo "OK  ${std}.md"
    standards_ok=$((standards_ok + 1))
  else
    echo "FALTA  ${std}.md"
  fi
done
echo "Standards: ${standards_ok}/14 disponibles"
```

---

## Paso 6 — Verificar templates base

Los siguientes templates deben existir en `docs/templates/`:

- `PRD-TEMPLATE.md`
- `BLUEPRINT-TEMPLATE.md`

Verificación:

```bash
templates_ok=0

for tpl in PRD-TEMPLATE BLUEPRINT-TEMPLATE; do
  if [ -f "docs/templates/${tpl}.md" ]; then
    echo "OK  ${tpl}.md"
    templates_ok=$((templates_ok + 1))
  else
    echo "FALTA  ${tpl}.md"
  fi
done
echo "Templates: ${templates_ok}/2 disponibles"
```

---

## Paso 7 — Health report desde INDEX

`/setup` debe usar `docs/sessions/INDEX.md` como fuente de verdad inicial para evitar
barridos masivos del repositorio. Si el INDEX existe, extraer desde ahí el estado del
workspace antes de leer artefactos adicionales, en línea con la política del Orchestrator.

```bash
echo ""
echo "WORKSPACE HEALTH"

if [ -f "docs/sessions/INDEX.md" ]; then
  total_features=$(tail -n +4 docs/sessions/INDEX.md | grep -E '^\|' | wc -l | tr -d ' ')
  wip_features=$(grep -E '\|\s*(WIP|🚧 WIP)\s*\|' docs/sessions/INDEX.md | wc -l | tr -d ' ')
  failed_features=$(grep -E '\|\s*(FAILED|❌ FAILED|BLOCKED)\s*\|' docs/sessions/INDEX.md | wc -l | tr -d ' ')
  done_features=$(grep -E '\|\s*(DONE|✅ DONE|PRODUCTION-READY)\s*\|' docs/sessions/INDEX.md | wc -l | tr -d ' ')

  if [ -d "docs/bugs" ]; then
    open_bugs=$(grep -r "Estado: 🔴 Open" docs/bugs/*.md 2>/dev/null | wc -l | tr -d ' ')
  else
    open_bugs=0
  fi

  echo "Features totales: ${total_features}"
  echo "En curso (WIP): ${wip_features}"
  echo "Completadas: ${done_features}"
  echo "Con problemas: ${failed_features}"
  echo "Bugs abiertos: ${open_bugs}"
else
  echo "No se pudo generar health report desde INDEX porque docs/sessions/INDEX.md no existe"
fi
```

---

## Salida esperada

```text
SDD QWIK SETUP REPORT

Estructura: OK / PARCIAL / FALTA
INDEX memoria: OK / CREADO / FALTA
Standards: N/14 disponibles
Templates: N/2 disponibles
Bugs abiertos: N
Estado: LISTO / REQUIERE ATENCIÓN / CONFIGURACIÓN NECESARIA

Próximo paso recomendado:
- /blueprint [proyecto], si aún no existe PRD aprobado
- /spec [módulo], si ya existe Blueprint aprobado
- /feature [módulo], si ya existe Spec aprobada
```

---

## Criterio de interpretación

- **LISTO**: estructura completa, INDEX presente, standards críticos completos y templates base presentes.
- **REQUIERE ATENCIÓN**: estructura creada pero faltan uno o varios standards/templates o el INDEX está incompleto.
- **CONFIGURACIÓN NECESARIA**: faltan directorios base, no existe INDEX y el workspace no está inicializado.