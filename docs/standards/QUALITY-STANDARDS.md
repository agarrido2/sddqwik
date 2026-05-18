# QUALITY STANDARDS — SDD Qwik

> **Propósito:** Define los 5 pilares de calidad que todo código debe cumplir,
> el formato de auditoría, las reglas de SEO/A11Y y el sistema de observabilidad.
> Documento de referencia principal para @QwikAuditor.

---

## PARTE 1: LOS 5 PILARES

### 1. PERFORMANTE

| Métrica | Objetivo | Violación |
|---|---|---|
| Bundle JS inicial | < 1KB ideal · < 5KB aceptable | > 5KB sin justificación |
| LCP | < 2.5s | > 2.5s |
| CLS | < 0.1 | > 0.1 |
| FID / INP | < 100ms | > 100ms |
| Hidratación | **CERO** | Cualquier `useVisibleTask$` injustificado |

**Checklist:**
- [ ] `routeLoader$` para todos los datos SSR (no `useVisibleTask$` para fetch)
- [ ] Imágenes con `width`+`height` explícitos o `@unpic/qwik` (previene CLS)
- [ ] Scripts de terceros via Partytown (`type="text/partytown"`)
- [ ] `useVisibleTask$` solo para: DOM directo · browser APIs · librerías 3rd-party
- [ ] `sharedMap` para datos del layout compartidos entre loaders (no repetir queries)
- [ ] `useComputed$` para valores derivados síncronos (no `routeLoader$` para ello)

---

### 2. IDIOMÁTICO (Qwik)

**Checklist:**
- [ ] `component$()` en todos los componentes
- [ ] Sufijo `$` en todos los handlers (`onClick$`, `onInput$`, etc.)
- [ ] Props serializables — sin funciones crudas, clases, referencias circulares
- [ ] `PropFunction<T>` para callbacks entre componentes
- [ ] Cero APIs de React/Next.js (ver Blacklist en `DECISIONS_QWIK.md`)
- [ ] `sync$()` para interacciones puras de DOM
- [ ] `noSerialize()` en objetos de librerías de terceros en stores

- Iconos: `icon-map.ts` para runtime selection; funciones puras `PropsOf<'svg'>` para fijos [DECISIONS-UI.md]
- `useStylesScoped$` solo para `::before/::after`, `nth-child`, `keyframes` complejos [DECISIONS-UI.md]

---

### 3. ROBUSTO

**Checklist:**
- [ ] Toda `routeAction$` y `server$` tiene `zod$()` — sin excepciones
- [ ] Manejo explícito de errores en todos los `catch`  — sin `catch` vacíos
- [ ] Estados de carga definidos para operaciones async
- [ ] Fallbacks para `null`, `undefined` y estados vacíos
- [ ] TypeScript strict — cero `any`
- [ ] Tipos derivados de Drizzle-Zod — sin duplicación de schemas
- `noSerialize()` en stores con objetos 3rd-party (PDF.js, SignaturePad) — evita serialización rota en QRLs
- Query params de filtros: allowlist regex O(1) antes de Drizzle — previene injection via URL no confiable

---

### 4. ACCESIBLE Y SEO

**HTML Semántico:**
- [ ] `<main>` envuelve el contenido principal de cada página
- [ ] Un solo `<h1>` por página
- [ ] Jerarquía de headings lógica (`h1→h2→h3`, sin saltos)
- [ ] `role="status" aria-live="polite"` en empty states visibles (sin `sr-only`) — screen readers anuncian cambios sin ocultar texto visible
- [ ] `roledialog`: `useVisibleTask$` para focus primer interactivo + tab trap (`addEventListener keydown`) — WCAG 2.1 SC 2.1.2, evita foco perdido
- [ ] `<button>` para acciones · `<Link>` para navegación interna · `<a>` para externa
- [ ] `<label>` asociado a cada `<input>` (atributo `for`, no solo placeholder)
- [ ] SVG decorativos con `aria-hidden="true"`
- [ ] Botones/enlaces de solo icono con `aria-label`
- [ ] Navegación por teclado funcional (sin `div` con `onClick$`)

**Imágenes:**
- [ ] `alt` descriptivo en todas las imágenes
- [ ] `alt=""` en imágenes puramente decorativas
- [ ] Imágenes estáticas via `import img from '...?jsx'`
- [ ] Imágenes dinámicas via `<Image>` de `@unpic/qwik`

**DocumentHead (obligatorio en todas las rutas públicas):**
- [ ] `title` único por página
- [ ] `meta description` (150-160 caracteres)
- [ ] Open Graph: `og:title`, `og:description`, `og:image` (1200x630px), `og:url`, `og:type`
- [ ] Twitter Card: `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- [ ] URL canónica declarada
- [ ] Datos estructurados Schema.org para landing pages (FAQPage, Product, Organization)

**Para landing pages públicas (adicional):**
- [ ] `robots.txt` en `public/`
- [ ] Endpoint `sitemap.xml` implementado en `src/routes/sitemap.xml/index.ts`

---

### 5. SEGURO

**Checklist:**
- [ ] Validación server-side obligatoria — nunca solo cliente
- [ ] Sin secrets en variables `PUBLIC_*` (solo `PUBLIC_SUPABASE_URL` y `PUBLIC_SUPABASE_ANON_KEY`)
- [ ] RLS habilitado y documentado en schema para tablas con datos de usuario
- [ ] Autorización verificada (no solo autenticación) — `sharedMap.get('user')` no basta
- [ ] Sin SQL raw — solo queries via Drizzle
- [ ] Sin datos sensibles en logs de producción (no PII en `console.*`)
- [ ] Multi-tenant safe: ninguna query sin filtro `organizationId`

---

## PARTE 2: FORMATO DE AUDITORÍA

Para código crítico (formularios, auth, loaders/actions, estado complejo):

```
🔍 QUALITY AUDIT — [feature/componente]

Pilar                | Estado | Notas
---------------------|--------|------------------
Performante          | ✅/❌  | [evidencia]
Idiomático (Qwik/UI) | ✅/❌  | [QRLs, sync$, iconos duales]
Robusto              | ✅/❌  | [Zod inline, noSerialize]
Accesible/SEO/UI     | ✅/❌  | [h1 único, focus trap, empty states]
Seguro               | ✅/❌  | [RLS, orgId filter]

UI Checklist (si aplica):
- [ ] Dark mode zero-flicker (script head) [DECISIONS-UI]
- [ ] Estados vacíos: CTA + ilustración + microcopy humano [UX-GUIDE]
- [ ] Optimistic UI en tablas/forms (acción inmediata, revert si error) [UX-GUIDE]

RESULTADO: ✅ PASSED / ❌ FAILED
Issues críticos: [N] — [descripción con archivo:línea]
```

Para código no crítico (componentes visuales simples, sin estado ni acciones):
aplicar estándares implícitamente, reportar solo si hay violaciones claras.

---

## PARTE 3: OBSERVABILIDAD Y LOGGING

### Taxonomía de Errores (Obligatoria)

| Prefijo | Capa | Cuándo |
|---|---|---|
| `ORCH_XXX` | `src/routes/` | Errores en Loaders, Actions, validación Zod |
| `SERV_XXX` | `src/lib/` | Errores en servicios, integraciones externas |
| `DATA_XXX` | `src/lib/db/` | Errores de persistencia, RLS, conectividad |

### Logging Estructurado (Prohibido `console.log`)

```typescript
// ✅ CORRECTO
Logger.error({
  code: 'SERV_001',
  message: 'Fallo al procesar el pago con Stripe',
  context: { organizationId: org.id, planId },
  trace: error.stack,
});

// ❌ INCORRECTO
console.log('Error en el pago', error); // sin código, sin contexto, sin trazabilidad
console.log(user); // puede exponer PII en producción
```

### Estrategia de Captura por Capa

```
src/lib/ (Servicios):
  - Captura errores técnicos
  - Transforma en código SERV_XXX
  - Lanza excepción controlada

src/routes/ (Orchestrator):
  - Captura excepción del servicio
  - Decide qué mostrar al usuario
  - Devuelve { success: false, message: "mensaje amigable" }

src/components/ (UI):
  - Solo renderiza el estado de error
  - Nunca maneja lógica de errores
```

### @QwikAuditor verifica:
- [ ] Sin `catch` vacíos en ninguna capa
- [ ] Toda `routeAction$` maneja explícitamente el caso de error
- [ ] Prefijos `ORCH_/SERV_/DATA_` en todas las capturas de excepciones
- [ ] Sin PII en logs de producción (userId sí · email/nombre no en logs)

---

## PARTE 4: TDD — REGLAS DE TESTING

> **Fuente canónica:** `docs/standards/TESTING-POLICY.md`
>
> `TESTING-POLICY.md` define qué código necesita test, el protocolo del Builder,
> la estructura de tests, los coverage targets y el BLOQUE K de verificación del
> Auditor. Es la referencia única y obligatoria para todo lo relativo a testing.
> @QwikAuditor verifica el cumplimiento de `TESTING-POLICY.md` en cada auditoría.

#