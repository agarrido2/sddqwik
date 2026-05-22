---
# EXTERNAL_AGENT_PATH: "github/prompts/setup.prompt.md"
name: setup
description: >
  Inicializa o verifica el workspace SDD Qwik para flujo spec-first. Comprueba estructura operativa,
  agentes, prompts, standards, templates, memoria operativa e índice del proyecto,
  y genera un health report accionable sin exigir comandos adicionales al usuario.
tools: ["edit", "execute/runInTerminal", "read"]
argument-hint: "example: /setup"
---

# 🧱 WORKSPACE SETUP / HEALTH CHECK — SDD Qwik

## Objetivo

`/setup` es el punto único de inicialización, diagnóstico y verificación global del sistema SDD Qwik spec-first.

No añade carga cognitiva al usuario.
No requiere comandos auxiliares.
No debe convertirse en un barrido masivo del repositorio.

Su responsabilidad es:

1. verificar estructura operativa mínima;
2. crear estructura documental si falta;
3. verificar que el sistema agéntico es operable;
4. verificar standards canónicos y templates si aplica;
5. asegurar memoria inicial;
6. revisar estado del workspace desde `docs/sessions/INDEX.md`;
7. emitir un informe claro con siguiente paso recomendado.

---

## Regla operativa crítica

`/setup` NO debe ejecutar exploraciones masivas.

Prohibido por defecto:

- `find .`
- `ls -R`
- listar directorios completos de `src/`
- listar todas las specs
- listar todos los plans
- leer sesiones archivadas
- leer specs de features `Done`
- cargar código de aplicación salvo que el INDEX indique WIP concreto

Fuente inicial de verdad operativa:

```text
docs/sessions/INDEX.md
```

Si el INDEX no existe, `/setup` debe crearlo.

El setup operativo prepara el camino hacia `/spec [feature]`. PRD y Blueprint no forman parte del setup operativo principal ni son gates para considerar el sistema listo.

---

## Paso 1 — Verificar estructura agéntica

Ejecutar:

```bash
echo "SDD QWIK SETUP — SYSTEM CHECK"

if [ -d "github" ]; then
  SDD_GITHUB_DIR="github"
  echo "Estructura agéntica: OK github/"
elif [ -d ".github" ]; then
  SDD_GITHUB_DIR=".github"
  echo "Estructura agéntica: OK .github/"
else
  SDD_GITHUB_DIR="github"
  echo "Estructura agéntica: FALTA github/ o .github/"
fi
```

---

## Paso 2 — Crear estructura documental mínima

Crear solo directorios canónicos.
No crear código de aplicación.

```bash
mkdir -p docs/templates
mkdir -p docs/specs
mkdir -p docs/plans
mkdir -p docs/audits
mkdir -p docs/bugs
mkdir -p docs/adr
mkdir -p docs/sessions
mkdir -p docs/sessions/archive
mkdir -p docs/standards
mkdir -p .scratch
mkdir -p scripts/db
```

---

## Paso 3 — Verificar `.gitignore`

Asegurar que `.gitignore` existe.

```bash
if [ ! -f ".gitignore" ]; then
  touch .gitignore
  echo "CREADO .gitignore"
fi
```

Verificar entradas mínimas.

```bash
ensure_gitignore_entry() {
  entry="$1"
  if ! grep -qxF "$entry" .gitignore 2>/dev/null; then
    echo "$entry" >> .gitignore
    echo "AÑADIDO .gitignore: $entry"
  else
    echo "OK .gitignore: $entry"
  fi
}

ensure_gitignore_entry ".scratch"
ensure_gitignore_entry ".env"
ensure_gitignore_entry ".env.local"
ensure_gitignore_entry ".env.*.local"
ensure_gitignore_entry "dist"
ensure_gitignore_entry "node_modules"
ensure_gitignore_entry ".qwik"
ensure_gitignore_entry ".drizzle"
ensure_gitignore_entry "docs/sessions/archive/"
ensure_gitignore_entry "docs/sessions/*.tmp.md"
```

### Regla crítica sobre memoria

No ignorar por defecto:

```text
docs/sessions/INDEX.md
```

`docs/sessions/INDEX.md` es un artefacto estructural del sistema.
Debe poder versionarse si el proyecto quiere mantener memoria operativa entre sesiones, clones o colaboradores.

Si `.gitignore` contiene una línea exacta:

```gitignore
docs/sessions
```

o:

```gitignore
docs/sessions/
```

reportar advertencia:

```text
⚠️ ADVERTENCIA: .gitignore ignora docs/sessions completo.
Esto puede ocultar docs/sessions/INDEX.md, que SDD Qwik usa como memoria operativa.
Recomendación: sustituir por docs/sessions/archive/ y docs/sessions/*.tmp.md
```

No eliminar automáticamente esa línea salvo instrucción explícita del usuario.

---

## Paso 4 — Inicializar `docs/sessions/INDEX.md`

Si no existe, crearlo.

```bash
if [ ! -f docs/sessions/INDEX.md ]; then
  cat > docs/sessions/INDEX.md <<'EOF'
# Project Features Index

> Mantenido por @QwikMemory.
> Primera fuente de verdad operativa para @QwikOrchestrator.
> No es un resumen decorativo: filtra qué artefactos deben cargarse.

| Feature | Módulo | Estado | Depende de | Tablas DB | Expone | Artefactos | Resumen | Fecha |
|---------|--------|--------|------------|-----------|--------|------------|---------|-------|

EOF
  echo "CREADO docs/sessions/INDEX.md"
else
  echo "OK docs/sessions/INDEX.md"
fi
```

---

## Paso 5 — Verificar estructura base

```bash
echo ""
echo "ESTRUCTURA BASE"

dirs_ok=0
dirs_total=0

check_dir() {
  dirs_total=$((dirs_total + 1))
  if [ -d "$1" ]; then
    echo "OK     $1"
    dirs_ok=$((dirs_ok + 1))
  else
    echo "FALTA  $1"
  fi
}

check_dir "docs/templates"
check_dir "docs/specs"
check_dir "docs/plans"
check_dir "docs/audits"
check_dir "docs/bugs"
check_dir "docs/adr"
check_dir "docs/sessions"
check_dir "docs/sessions/archive"
check_dir "docs/standards"
check_dir ".scratch"
check_dir "scripts/db"

echo "Directorios: ${dirs_ok}/${dirs_total}"
```

---

## Paso 6 — Verificar agentes canónicos

```bash
echo ""
echo "AGENTES SDD QWIK"

agents_ok=0
agents_total=0

check_agent() {
  agents_total=$((agents_total + 1))
  file="${SDD_GITHUB_DIR}/agents/$1"
  if [ -f "$file" ]; then
    echo "OK     $file"
    agents_ok=$((agents_ok + 1))
  else
    echo "FALTA  ${SDD_GITHUB_DIR}/agents/$1"
  fi
}

check_agent "qwik-orchestrator.agent.md"
check_agent "qwik-speccer.agent.md"
check_agent "qwik-architect.agent.md"
check_agent "qwik-dba.agent.md"
check_agent "qwik-builder.agent.md"
check_agent "qwik-auditor.agent.md"
check_agent "qwik-polisher.agent.md"
check_agent "qwik-memory.agent.md"
check_agent "qwik-bug-fix.agent.md"

echo "Agentes: ${agents_ok}/${agents_total}"
```

---

## Paso 7 — Verificar prompts canónicos

```bash
echo ""
echo "PROMPTS SDD QWIK"

prompts_ok=0
prompts_total=0

check_prompt() {
  prompts_total=$((prompts_total + 1))
  file="${SDD_GITHUB_DIR}/prompts/$1"
  if [ -f "$file" ]; then
    echo "OK     $file"
    prompts_ok=$((prompts_ok + 1))
  else
    echo "FALTA  ${SDD_GITHUB_DIR}/prompts/$1"
  fi
}

check_prompt "setup.prompt.md"
check_prompt "spec.prompt.md"
check_prompt "new-feature.prompt.md"
check_prompt "bug-fix.prompt.md"
check_prompt "legacy-audit.prompt.md"
check_prompt "optimizer-code.prompt.md"
check_prompt "memory-compact.prompt.md"
check_prompt "new-session.prompt.md"

echo "Prompts: ${prompts_ok}/${prompts_total}"
```

---

## Paso 8 — Verificar standards canónicos

```bash
echo ""
echo "STANDARDS SDD QWIK"

standards_ok=0
standards_total=0

check_standard() {
  standards_total=$((standards_total + 1))
  file="docs/standards/$1"
  if [ -f "$file" ]; then
    echo "OK     $file"
    standards_ok=$((standards_ok + 1))
  else
    echo "FALTA  $file"
  fi
}

check_standard "ARQUITECTURA-FOLDER.md"
check_standard "PROJECT-RULES-CORE.md"
check_standard "SDD-WORKFLOW.md"
check_standard "DECISIONS-QWIK.md"
check_standard "DECISIONS-DATA.md"
check_standard "DECISIONS-UI.md"
check_standard "SERIALIZATION-CONTRACTS.md"
check_standard "QUALITY-STANDARDS.md"
check_standard "SECURITY-POLICIES.md"
check_standard "TESTING-POLICY.md"
check_standard "UX-GUIDE.md"
check_standard "RBAC-ROLES-PERMISSIONS.md"
check_standard "CONTEXT7-GUIDE.md"
check_standard "LESSONS-LEARNED.md"

echo "Standards: ${standards_ok}/${standards_total}"
```

### Regla

Si falta `ARQUITECTURA-FOLDER.md`, `PROJECT-RULES-CORE.md`, `SDD-WORKFLOW.md`, `DECISIONS-QWIK.md`, `SERIALIZATION-CONTRACTS.md` o `QUALITY-STANDARDS.md`, el sistema no debe considerarse plenamente operativo.

---

## Paso 9 — Verificar templates base

```bash
echo ""
echo "TEMPLATES SDD QWIK"

templates_status="OK"

if [ -d "docs/templates" ]; then
  echo "OK     docs/templates"
else
  templates_status="FALTA"
  echo "FALTA  docs/templates"
fi

echo "Templates: ${templates_status}"
```

---

## Paso 10 — Health report desde INDEX

No listar specs ni plans directamente.
Usar `docs/sessions/INDEX.md`.

```bash
echo ""
echo "WORKSPACE HEALTH DESDE INDEX"

if [ -f "docs/sessions/INDEX.md" ]; then
  total_features=$(tail -n +7 docs/sessions/INDEX.md | grep -E '^\|' | wc -l | tr -d ' ')
  wip_features=$(grep -E '\|\s*(WIP|🚧 WIP|IN-PROGRESS|🟡 WIP)\s*\|' docs/sessions/INDEX.md | wc -l | tr -d ' ')
  failed_features=$(grep -E '\|\s*(FAILED|❌ FAILED|BLOCKED|🔴 BLOCKED)\s*\|' docs/sessions/INDEX.md | wc -l | tr -d ' ')
  done_features=$(grep -E '\|\s*(DONE|✅ DONE|PRODUCTION-READY|✅ PRODUCTION-READY)\s*\|' docs/sessions/INDEX.md | wc -l | tr -d ' ')

  echo "Features totales: ${total_features}"
  echo "En curso WIP:      ${wip_features}"
  echo "Completadas:       ${done_features}"
  echo "Con problemas:     ${failed_features}"
else
  echo "No se pudo generar health report: falta docs/sessions/INDEX.md"
fi
```

---

## Paso 11 — Bugs abiertos

Se permite revisar `docs/bugs/*.md` porque es acotado y forma parte del health check.

```bash
echo ""
echo "BUGS"

if [ -d "docs/bugs" ]; then
  open_bugs=$(grep -rE "Estado:\s*(🔴 Open|Open|OPEN|Abierto|ABIERTO)" docs/bugs/*.md 2>/dev/null | wc -l | tr -d ' ')
  echo "Bugs abiertos: ${open_bugs}"
else
  open_bugs=0
  echo "Bugs abiertos: 0"
fi
```

---

## Paso 12 — Detectar riesgo básico de contexto

No calcular tokens reales.
Clasificar riesgo según señales operativas simples.

```bash
echo ""
echo "RIESGO DE CONTEXTO"

context_risk="bajo"

if [ -f "docs/sessions/INDEX.md" ]; then
  wip_count=$(grep -E '\|\s*(WIP|🚧 WIP|IN-PROGRESS|🟡 WIP)\s*\|' docs/sessions/INDEX.md | wc -l | tr -d ' ')
  failed_count=$(grep -E '\|\s*(FAILED|❌ FAILED|BLOCKED|🔴 BLOCKED)\s*\|' docs/sessions/INDEX.md | wc -l | tr -d ' ')

  if [ "$wip_count" -gt 2 ] || [ "$failed_count" -gt 0 ]; then
    context_risk="medio"
  fi

  if [ "$wip_count" -gt 4 ]; then
    context_risk="alto"
  fi
fi

echo "Riesgo estimado de contexto: ${context_risk}"
```

---

## Paso 13 — Determinar estado general

```bash
echo ""
echo "ESTADO GENERAL"

status="LISTO"

if [ ! -d "$SDD_GITHUB_DIR" ]; then
  status="CONFIGURACIÓN NECESARIA"
fi

if [ "$status" != "CONFIGURACIÓN NECESARIA" ] && { [ "$agents_ok" -lt "$agents_total" ] || [ "$prompts_ok" -lt "$prompts_total" ]; }; then
  status="REQUIERE ATENCIÓN"
fi

if [ "$status" != "CONFIGURACIÓN NECESARIA" ] && { [ "$standards_ok" -lt "$standards_total" ] || [ "$templates_status" != "OK" ]; }; then
  status="REQUIERE ATENCIÓN"
fi

if [ "$status" != "CONFIGURACIÓN NECESARIA" ] && [ "$dirs_ok" -lt "$dirs_total" ]; then
  status="REQUIERE ATENCIÓN"
fi

echo "Estado general: ${status}"
```

---

## Paso 14 — Siguiente paso recomendado

Emitir una recomendación operativa, no una lista larga.

```bash
echo ""
echo "SIGUIENTE PASO RECOMENDADO"

if [ "$status" = "CONFIGURACIÓN NECESARIA" ]; then
  echo "- Completar instalación del sistema SDD Qwik."
  echo "- Debe existir github/ o .github/ con agents/, prompts/ y copilot-instructions.md."
elif [ "$status" = "REQUIERE ATENCIÓN" ]; then
  echo "- Corregir los elementos marcados como FALTA antes de iniciar features nuevas."
elif [ -f "docs/sessions/INDEX.md" ] && grep -qE '\|\s*(WIP|🚧 WIP|IN-PROGRESS|🟡 WIP)\s*\|' docs/sessions/INDEX.md; then
  echo "- Retomar la feature WIP desde @QwikOrchestrator."
  echo "- El Orchestrator debe leer docs/sessions/INDEX.md y cargar solo artefactos mínimos."
else
  echo "- Ejecutar /spec [feature] para crear el primer contrato de la siguiente feature."
fi
```

---

## Salida esperada

```text
SDD QWIK SETUP REPORT

Estructura agéntica: OK github/ / OK .github/ / FALTA
Estructura base: N/N
Agentes: N/9
Prompts: N/8
Standards: N/14
Templates: OK / FALTA
INDEX memoria: OK / CREADO / FALTA
Features totales: N
WIP: N
Done: N
Failed/Bloqueadas: N
Bugs abiertos: N
Riesgo contexto: bajo / medio / alto
Estado general: LISTO / REQUIERE ATENCIÓN / CONFIGURACIÓN NECESARIA

Siguiente paso recomendado:
[una acción clara]
```

---

## Criterios de interpretación

### LISTO

El sistema tiene:

- `github/` o `.github/` disponible;
- estructura base presente;
- agentes presentes;
- prompts presentes;
- standards canónicos presentes;
- `docs/templates/` disponible si aplica;
- `docs/sessions/INDEX.md` disponible.

### REQUIERE ATENCIÓN

El sistema existe, pero falta algún agente, prompt, standard, template o carpeta estructural.

No iniciar features nuevas hasta revisar los elementos marcados como `FALTA`.

### CONFIGURACIÓN NECESARIA

No existe estructura suficiente para operar SDD Qwik.

Falta `github/` o `.github/`, o la base documental está incompleta.

---

## Regla final

`/setup` no debe resolver features.
`/setup` no debe escribir código de aplicación.
`/setup` no debe auditar implementaciones.
`/setup` no debe leer el repo masivamente.

`/setup` solo debe dejar claro:

```text
¿El sistema está listo?
¿Qué falta?
¿Cuál es el siguiente paso correcto?
```
