---
# EXTERNAL_AGENT_PATH: ".github/prompts/optimizer-code.prompt.md"
name: optimizer-code
description: >
  Auditoría y Refactorización Quirúrgica. Desmonta monolitos y aplica Clean
  Code (DI, SoC, Portabilidad) en Qwik.
tools: ["read", "edit", "upstash/context7/*"]
argument-hint: "example: /optimizer-code src/features/auth/components/LoginForm.tsx"
---

# 🔬 PROTOCOLO DE DESCOMPOSICIÓN ANTIMONOLITO — QWIK

**Objetivo:** Erradicar el _bulky code_ del archivo `${input:filePath}`. No refinamos — **segmentamos y purificamos**.

---

## 🩻 Fase 1 — Auditoría de Deuda Técnica

Antes de generar código, detecta y reporta:

1. **Fugas de Lógica (SoC):** ¿Hay lógica de negocio, validaciones o transformaciones de datos directamente implementadas dentro de componentes de UI?
2. **Dependencias Rígidas (DI):** ¿Hay importaciones directas de Supabase/Drizzle dentro de un componente visual?
3. **Hardcoded Junk:** ¿Hay arrays de datos, configuraciones JSON o estilos complejos definidos inline dentro del componente?
4. **Violaciones de Prosa:** ¿Hay funciones de más de 10 líneas, nombres genéricos (`data`, `item`, `handle`) o anidamiento excesivo?
5. **Fronteras `$()` rotas:** ¿Hay closures que capturan objetos no serializables (instancias de clase, Maps, Sets, Promesas)?
6. **Estado sobredimensionado:** ¿El `useStore` o `useSignal` contiene datos que no son necesarios para reanudar la interactividad?

---

## 🗺️ Fase 2 — Estrategia de Segmentación

Diseña la nueva estructura bajo estos mandatos:

- **Lógica Portátil:** Extrae TODO el estado y side-effects a un Custom Hook `use[DomainLogic]$`.
- **Externalización de Datos:** Mueve arrays y configuraciones a `constants.ts` o un `.json` externo.
- **Inyección de Dependencias:** El componente solo recibe piezas (`props` o `signals`), nunca sabe cómo se procesan los datos.
- **Atomic Design:** Descompone bloques TSX repetitivos en micro-componentes `component$` locales o compartidos.
- **Stores mínimos:** Redefine el estado para incluir SOLO lo necesario para reanudar la interactividad. Aplica `noSerialize()` agresivamente al resto.
- **Co-localización de QRLs:** Agrupa los handlers `$()` que se invocan juntos en el mismo archivo/chunk.

---

## ✍️ Fase 3 — Refactorización a Código Prosa

Genera el nuevo código siguiendo estas leyes:

1. **Nombres Semánticos:** Si el nombre no describe el _qué_ y el _por qué_, cámbialo.
2. **Single Purpose Functions:** Cada función hace UNA sola cosa. Si tiene un `if/else` complejo, divídela.
3. **Pureza del Orquestador:** El archivo original en `src/routes/` debe quedar reducido a un índice visual: consume el hook, ensambla piezas, no declara lógica.
4. **Resumabilidad QRL:** Asegura que cada `$()` es independiente y captura solo primitivos o IDs.
5. **sync$():** Usa `sync$()` para todas las interacciones puras de DOM (toggle de modales, clases CSS).

---

## ✅ Fase 4 — Validación de Invariantes

Cruza el resultado contra los siguientes grupos de validación en orden:

**Grupo A — Resumabilidad Qwik:**
- **Blacklist Absoluta:** Cero rastro de hooks de React o Next.js (ver `docs/standards/DECISIONS-QWIK.md` — hooks de React/Next.js).
- **Snapshot Size:** ¿El estado serializado es mínimo? ¿Se aplica `noSerialize()` donde corresponde?
- **Bundle Safety:** ¿Sin barrel exports (`export *`) en `src/features/`? ¿Sin librerías >10kB importadas completas? (ver `docs/standards/DECISIONS-QWIK.md` — bundle safety).

**Grupo B — Contratos y Datos:**
- **Contratos de Datos:** ¿Todos los retornos son interfaces puras/DTOs según `docs/standards/SERIALIZATION-CONTRACTS.md`?

**Grupo C — Observabilidad y Calidad:**
- **Observabilidad:** ¿Se usan prefijos `ORCH_`, `SERV_`, `DATA_` en las capturas de excepciones?
- **Lessons Learned:** ¿Se ha consultado el bloque Top Lecciones de `docs/standards/LESSONS-LEARNED.md`?
- **Tests:** ¿Los servicios refactorizados tienen sus tests en `src/tests/unit/`? (ver `docs/standards/TESTING-POLICY.md`)

---

## 📤 Salida Esperada

1. **Reporte de Auditoría:** Qué principios se violaban y cómo se han resuelto.
2. **Nuevos Artefactos:** Código para Custom Hooks, Services o Constants (archivos independientes).
3. **Componente Refactorizado:** La versión delgada y orquestadora.
4. **Sub-componentes Atómicos:** Desglose de piezas extraídas.

---

## Cuándo usar este prompt

- Un archivo mezcla responsabilidades visibles (UI + lógica de negocio + datos)
- El Auditor ha detectado violaciones de SoC o closures rotos
- Quieres refactorizar código existente antes de añadir nueva lógica
- Un componente no es portable sin arrastrar dependencias

---

> 💡 **Regla de oro:** Un componente que no puedes mover a otro proyecto sin
> tocar nada más **no es portátil**. Si no es portátil, no está terminado.