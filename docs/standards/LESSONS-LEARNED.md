# 🧠 LESSONS LEARNED — SDD Qwik

> Versión viva. Se actualiza tras cada FAILED de @QwikAuditor o bug recurrente.
> Standard Version: 2026.3 (SDD Qwik V3.0)
>
> **Agentes que leen este fichero:**
> - @QwikBuilder — antes de implementar en el área afectada
> - @QwikAuditor — como checklist adicional en cada auditoría

---

## ⚡ Top Lecciones (leer en cada tarea)

> Este bloque es el ÚNICO que los agentes leen en cada tarea.
> El registro completo solo se consulta al investigar un error específico.

1. **LL-001** `sync$` SOLO para DOM APIs puras (classList, localStorage, scrollTo, focus). Para mutar Signals/Stores usar `onClick$` asíncrono.
2. **LL-002** Nunca re-exportar `routeLoader$`/`routeAction$` entre rutas distintas. Para alias usar `throw requestEvent.redirect(302, '/ruta-canonica')`.
3. **LL-003** En `catch` de `routeAction$`: logging server-side con código `ORCH_XXX` + mensaje genérico al cliente. Nunca `error.message` directo al cliente.
4. **LL-004** En `routeAction$`, los schemas Zod SIEMPRE inline en `zod$(z.object({...}))`. Nunca schemas importados de módulos externos → TS2589.
5. **LL-005** Formularios con tabs: un `<Form action={...}>` independiente por cada tab. Un único Form nunca abarca campos de múltiples tabs si alguno puede estar desmontado.
6. **LL-006** En `onClick$` dentro de `.map()`, pasar solo el ID primitivo al QRL. Leer el objeto completo desde Signal/Store dentro del handler.
7. **LL-007** Toda `server$` con parámetros del cliente DEBE ejecutar `Schema.safeParse(data)` como primera operación. El tipado TS no reemplaza la validación runtime.
8. **LL-008** Todo `role="dialog"` necesita `useVisibleTask$` que ejecute `.focus()` en el primer elemento interactivo al abrirse. `aria-modal="true"` no mueve el foco automáticamente.
9. **LL-009** En CRUD de dashboard, sub-componentes co-localizados en el mismo `index.tsx` salvo reutilización real cross-feature.
10. **LL-010** Supabase `DATABASE_URL` SIEMPRE al Transaction Pooler (puerto 6543, IPv4). La conexión directa es IPv6-only en plan gratuito. Añadir `ssl: 'require'` en postgres-js.
11. **LL-011** En `onRequest`/`onGet`, toda llamada a DB o API externa DEBE tener try-catch. El fallback es el destino más seguro (login/onboarding). Sin try-catch, cualquier excepción rompe el flujo con 500.
12. **LL-012** En flujos de aceptación de tokens/invitaciones: verificar siempre que el usuario autenticado es el destinatario del token (email match). Un token válido sin este check permite que cualquier cuenta autenticada lo consuma.
13. **LL-013** El param `?next=` en redirects post-auth SIEMPRE requiere doble validación: en el loader (al leer query params) y en la action (al consumir el form field). Aceptar solo rutas relativas internas (`startsWith('/') && !includes('://')`). Aplicar en cualquier flow que propague destinos de redirect entre requests.
14. **LL-014** En `routeAction$` handlers usar `requireRole(ev, minRole, 'fail')` para HTTP 403 semántico. El default `'redirect'` se reserva para `onRequest` (navegación GET). Un `member` que POSTea a una action debe recibir 403, no 302.
15. **LL-015** `RequestEventAction` (con `.fail()`) NO está re-exportado desde `@builder.io/qwik-city`. Para acceder a `.fail()` desde funciones con tipo `RequestEventCommon`, declarar interfaz local `RequestEventWithFail { fail(status, data): unknown }` y castear `requestEv as unknown as RequestEventWithFail`. Nunca `as any`.
16. **LL-016** Endpoints HTTP de infraestructura (cron/webhooks) se protegen con `timingSafeEqual` de `node:crypto`, NO con sesiones Supabase. Tokens de servicio-a-servicio son distintos de la autenticación de usuarios. `===` es vulnerable a timing attacks.
17. **LL-017** Drizzle ORM no soporta expresiones computed con tipo `numeric` en cláusulas `WHERE` de forma tipada. Aplicar el filtro de coste normalizado en JS post-query cuando el volumen MVP es <50 filas/día. Migrar a `sql\`\`` solo si el volumen crece.
18. **LL-018** En jobs con múltiples envíos (email, webhook, notificación): usar `Promise.allSettled`, nunca `Promise.all`. Un fallo parcial no debe abortar el job. Acumular `fulfilled`/`rejected` en contadores separados del `JobResult`.
19. **LL-019** El cliente Drizzle/postgres-js con `DATABASE_URL` al Transaction Pooler **no** activa las policies RLS de Supabase (no inyecta `auth.uid()` contextual). El cliente Supabase JS con anon key sí. Para jobs server-only que necesitan leer todas las orgs sin RLS, reutilizar el `db` singleton existente.
20. **LL-020** GitHub Actions `schedule: cron` es el disparador preferido para jobs diarios en MVP. Es observable (logs en GH), sin infra adicional y soporta `workflow_dispatch` para ejecución manual. pg_cron y Edge Functions de Supabase solo si se migra a Supabase hosting completo.
21. **LL-021** En `routeAction$`, usar SIEMPRE la forma factory `zod$((z) => z.object({...}))`. Qwik City bundlea su propia versión interna de Zod — importar `z` externo desde `'zod'` genera TS2769 por conflicto de versiones. Aplica también a schemas vacíos: `zod$((z) => z.object({}))`.
22. **LL-022** La validación `startsWith('/') && !includes('://')` es **insuficiente** para open redirects. URLs protocol-relative `//evil.com` pasan ambos checks y los browsers las resuelven como externas. Patrón correcto: `startsWith('/') && !startsWith('//') && !includes('://')`.
23. **LL-023** Rutas de contenido público (`/mentors`, `/mentors/:id`) jamás deben colocarse bajo el grupo `(public)/`, cuyo layout inyecta `<h1>` propio + `max-w-sm` pensados para auth forms. Patrón correcto: grupo `(content)/` con layout mínimo (sin `<h1>`, sin width constraint, sin guard). Regla: `(public)/` = auth forms, `(content)/` = contenido público, `(app)/` = dashboard autenticado.
24. **LL-024** Tailwind v4 usa sintaxis CSS-native para variables: `text-(--color-primary)`, `bg-(--color-surface)`. La sintaxis legacy `text-[var(--color-primary)]` genera warnings del linter y debe evitarse en código nuevo. Afecta a cualquier clase arbitraria que referencie custom properties CSS.
25. **LL-025** `<input type="datetime-local">` genera valores como `"2026-03-23T10:00"` (sin `Z` ni offset). Zod `.datetime({ offset: true })` rechaza ese formato. Usar `.string().refine((v) => !isNaN(new Date(v).getTime()), { message: 'Fecha inválida' })` en toda `routeAction$` que reciba un campo datetime-local del cliente.
26. **LL-026** Los closures `sync$()` no pueden capturar Signals de Qwik (no son serializables en el scheduler). Para acceder al DOM dentro de `sync$`, usar `document.getElementById('mi-id')`. Para operaciones que necesitan mutar Signals usar `onClick$` (async). Aplica a dialogs nativos, tooltips y cualquier interacción DOM pura que también necesite mutar estado.
27. **LL-027** `alias()` de Drizzle ORM está en `drizzle-orm/pg-core`, **no** en `drizzle-orm`. Cualquier JOIN multi-tabla con alias debe importarlo de la ruta correcta: `import { alias } from 'drizzle-orm/pg-core'`. Importar desde `drizzle-orm` arroja `SyntaxError: Export named 'alias' not found` en runtime (Bun/Node).
28. **LL-028** Tablas de transacciones (`bookings`, `orders`, `payments`): DELETE bloqueado por RLS + soft-delete via campo `status`. Nunca borrar registros de transacciones reales. La trazabilidad es un requisito implícito en cualquier marketplace o SaaS con flujo de dinero.
29. **LL-029** `role="status"` + `aria-live="polite"` para mensajes de empty state **visibles** no debe incluir `class="sr-only"`. Añadir `sr-only` a un mensaje visible es un bug de a11y para usuarios sighted. Reservar `sr-only` para live-regions auxiliares separadas del copy principal.
29. **LL-029** `role="status"` + `aria-live="polite"` para mensajes de estado vacío VISIBLES **no** debe incluir `class="sr-only"`. `sr-only` oculta el contenido a usuarios sighted — añadirlo a un mensaje visible es un bug de a11y, no una mejora. Patrón correcto: `<p role="status" aria-live="polite">Mensaje visible</p>`. Reservar `sr-only` para contenido suplementario destinado exclusivamente a screen readers (ej: una live-region auxiliar separada del texto principal).
30. **LL-030** Clientes de servicios externos (Resend, etc.) en SSR fire-and-forget: crear instancia por llamada (stateless), no singleton módulo-nivel. Evita credenciales rancias cross-request y facilita tests sin orden de ejecución.
31. **LL-031** Tests de `onRequest` en Qwik City: `RequestHandler` está tipado con retorno `void` en TypeScript. En runtime es `Promise<void>` que rechaza con `Response` (redirect). Para capturar el redirect en tests usar `(onRequest(ev) as unknown as Promise<void>)` envuelto en try-catch: si `e instanceof Response` retornarlo, si no re-throw. Evita `.catch()` directo sobre `void` que genera TS2339.
32. **LL-032** `requestEv.redirect()` en Qwik City lanza `RedirectMessage` (no `Response`). El patrón `if (e instanceof Response) throw e` **siempre es false** — el redirect queda atrapado en el catch y el handler continúa retornando `fail()`, bloqueando el `Set-Cookie` de sesión. Regla: `throw requestEv.redirect()` debe ir **siempre fuera del bloque try-catch**. Solo las operaciones async (llamadas a DB, servicios externos) van dentro del try.
33. **LL-033** Qwik City descarta redirects a URLs externas desde `routeAction$` vía `<Form>` — `handleQDataRedirect` elimina cualquier URL que no empiece por `/`. Patrón confirmado para OAuth (Google, y por extensión cualquier proveedor OAuth). Para OAuth usar `onGet` dedicado (`/auth/proveedor`) con `throw requestEv.redirect(302, externalUrl)` y `<a href='/auth/proveedor'>` en lugar de `<Form action={oauthAction}>`. Comportamiento pendiente de confirmar para otros tipos de redirect externo (pagos, etc.).
34. **LL-034** Focus trap en Drawer/Modal con `useVisibleTask$`: mover foco al abrir NO es suficiente. Hay que añadir `document.addEventListener('keydown', handleTab)` para interceptar Tab/Shift+Tab y ciclar el foco entre los elementos focusables del dialog. El listener DEBE eliminarse con el callback `cleanup()` de `useVisibleTask$` cuando el Signal vuelve a `false` — de lo contrario queda activo en toda la app. Selector correcto: `'a[href], button:not([disabled]), [tabindex]:not([tabindex="-1"])'`.
35. **LL-035** El flicker del sidebar (expandido→colapsado visible en cada reload) es consecuencia de usar `localStorage` para persistir la preferencia. `localStorage` solo es accesible en cliente → el SSR siempre renderiza el estado por defecto → layout shift visible antes de la hidratación. Solución: cookie `SameSite=Lax; HttpOnly=false; Max-Age=31536000` leída en `routeLoader$` (SSR) → inicializa el Signal con el valor correcto antes de serializar el HTML → cero flash. Escritura en `onClick$` con `document.cookie = 'sidebar_state=...'`. Para producción añadir `; Secure` si `window.location.protocol === 'https:'`.
34. **LL-034** Logger estructurado obligatorio en handlers — nunca `console.error` directo. Crear `src/lib/logger.ts` con `Logger.error({code, message, context?})` para evitar PII en logs de producción. El objeto `e` de un `catch` puede contener stack traces con rutas de filesystem, tokens parciales o datos de sesión. Patrón correcto: `Logger.error({ code: 'ORCH_XXX', message: 'Descripción', context: { supabaseCode: error.code }, trace: e instanceof Error ? e.stack : String(e) })`. Log estructurado JSON con timestamp; sin `console.log` ni `console.error` en handlers de rutas.
35. **LL-035** `db.execute()` con driver postgres-js **devuelve el array de resultados directamente**, no un objeto con `.rows`. `await db.execute(sql\`...\`)` ya es el array. Pattern correcto: `const rows = await db.execute(sql\`...\`) as unknown as T[]`. Acceder a `.rows` lanza TS2339 en compilación. Aplica a cualquier llamada de función SQL via `db.execute()` con Drizzle ORM + postgres-js.
36. **LL-036** Limitaciones de Zod en Qwik City: El paquete Zod integrado en Qwik City para `routeAction$` es una versión reducida para optimizar el bundle. Métodos como `.startsWith()` no están disponibles en tiempo de ejecución (aunque TypeScript no los detecta como error en compilación). Realizar validaciones estructurales básicas en Zod (tipo, longitud mínima) y mover las validaciones de formato complejas (como verificar prefijos de strings Base64 `data:image/png;base64,`) a la capa de servicios del backend antes del procesamiento.
37. **LL-037** Bun Test Registry Pollution: En Bun v1.3.x, `mock.module()` comparte el registro entre archivos de test en el mismo proceso. Si un mock parcial se registra primero (orden alfabético), afectará a tests posteriores con `SyntaxError: Export named 'X' not found`. Solución: el primer archivo que registre el mock de un módulo compartido debe incluir todos los exports necesarios para la suite completa (como `{}` vacío si no se usan). Alternativa: ejecutar cada archivo en proceso separado.
38. **LL-038** Golden Tests con SHA-256: Para hashes deterministas en tests, la fuente de verdad **siempre** debe ser `node:crypto` aplicado directamente sobre el buffer en runtime — nunca un valor calculado via shell/terminal. `ArrayBuffer(N)` creado en JS no equivale a `printf '\x00%.0s' {1..N}` en shell (diferencias de encoding, piping y plataforma). Patrón correcto: ejecutar el test una vez sin aserción de hash para capturar el valor real producido por `node:crypto`, luego fijarlo como expected. Aplica a cualquier golden test que valide salidas de funciones hash sobre buffers en memoria.
39. **LL-039** Accesibilidad y `sync$` en Drawers Mobile: En listeners globales (`onKeydown$` en nodo raíz o `document:onKeydown$`), `e.currentTarget` referencia al nodo contenedor o al objeto `Document` y provoca crash al invocar métodos de elemento (`setAttribute`, `classList`). Solución: usar selectores directos por ID (`document.getElementById('id')`) dentro del handler `sync$` para garantizar atomicidad y evitar problemas de contexto en el DOM. El parámetro `e` en `sync$` solo es fiable para leer `e.key`, `e.type`, `e.preventDefault()` y `e.stopPropagation()`. Nunca usar `e.currentTarget` para manipular el DOM.
40. **LL-040** Inyección de servicios transaccionales en módulos Qwik/Bun: Al añadir un nuevo servicio (ej. `email.service.ts`) que dependen de env vars a nivel de módulo, el fichero de tests de los servicios consumidores (`signer.service.test.ts`, `signature.service.test.ts`) DEBE registrar un `mock.module('~/lib/services/email.service')` con TODOS los exports antes del import dinámico del servicio bajo test (LL-037 ampliado). Si el mock se omite, el guard de env vars real se activa durante la carga del módulo en tests y falla con `SERV_EMAIL_CONFIG_MISSING`. Patrón canónico: mock todos los deps externos antes del import del módulo a testar.
41. **LL-041** `noSerialize` obligatorio para objetos de terceros (PDF.js/SignaturePad). Evitar `let` de módulo: el optimizador los convierte en `const` dentro de los QRLs.
42. **LL-042** Error `exp` (timestamp) en JWT de Supabase Storage: forzar re-login para refrescar tokens ranciados o desincronizados tras cambios de red.
43. **LL-043** Driver `postgres-js` devuelve `timestamps` como strings ISO 8601. Siempre instanciar con `new Date()` antes de operar con fechas en SSR.
44. **LL-044** Canvas en Retina (Mac): escalar buffer por `devicePixelRatio`. Calibrar mouse/touch con la fórmula: `(e.clientX - rect.left) * (canvas.width / rect.width) / dpr`.
45. **LL-045** Sincronización de Identidad Stripe-DB: Usar `client_reference_id` en `checkout.sessions.create` (no solo `metadata`) para que el webhook `checkout.session.completed` disponga del `profileId` sin depender de la replicación de metadata a `Subscription`. Aplicar "Double Lazy Creation" (verificar perfil antes de insertar suscripción) para soportar entornos con DB volátil (TRUNCATE, nuevos deploys). El UPSERT en `subscriptions` cierra el ciclo incluso si el evento llega antes de que exista la fila.
46. **LL-046** (reservado)
47. **LL-047** En Qwik City, los `routeLoader$` definidos en un layout padre (`(app)/layout.tsx`) son automáticamente heredados y ejecutados en todas las rutas hijas de ese layout. Re-exportarlos desde una sub-ruta hija (`export { useOrgContext } from '~/routes/(app)/layout'`) causa colisión de manifesto de Vite con el error `The same routeLoader$ (…) was exported in multiple modules`. La solución correcta es: importar el loader solo en el componente que lo consume con `useOrgContext()`, o leer los datos directamente del `sharedMap` en el propio `routeLoader$` hijo.
48. **LL-048** `server$` en Qwik expone `RequestEventBase`, no `RequestEventCommon`. Llamar `getSession(this)` dentro de una `server$` genera un type mismatch en runtime. En rutas bajo `(app)/`, donde el `onRequest` del layout ya garantiza la sesión y la puebla en `sharedMap`, usar `this.sharedMap.get('user')` para acceder al usuario. Nunca invocar `getSession(this)` en una `server$` cuando el layout padre ya resolvió la sesión.
50. **LL-050** Supabase Realtime hereda automáticamente las policies RLS SELECT activas para la tabla suscrita. El filtro `user_id=eq.{userId}` en el canal Realtime es defensa en profundidad (segunda barrera), no el mecanismo de seguridad primario — ese es RLS. Nunca omitir el filtro de canal (protección redundante deseable), pero nunca depender de él como única barrera de autorización. La política RLS INSERT debe ser permisiva para `authenticated` para que los servicios server-side puedan insertar notificaciones de otros usuarios.
51. **LL-054** El patrón DI en servicios que exponen `dbClient: typeof db` como primer parámetro obliga a las rutas a importar `db` a nivel de módulo. Qwik City/Rollup procesa las importaciones top-level estáticamente y las incluye en AMBOS grafos (servidor y cliente), filtrando paquetes Node-only (`postgres`, `perf_hooks`) al bundle del navegador. Internalizar **siempre** la conexión `db` dentro del servicio. Para fugas indirectas irresolubles (ej. `layout.tsx → session.ts → db`), usar `build.rollupOptions.external: ['postgres', 'perf_hooks']` en `vite.config.ts` como barrera definitiva.
70. **LL-070** Los `routeLoader$` del layout padre y de la ruta hija se ejecutan **concurrentemente** (`Promise.all`) en Qwik City. Leer `sharedMap.get('profile')` dentro de un loader de ruta cuando ese valor lo escribe `useOrgContext` (layout loader) siempre devuelve `undefined` porque el `await db.select()` del layout no ha completado todavía. Esto produce un redirect silencioso a `/login` → que el layout público redirige a `/dashboard`, dando la apariencia de un fallo de permisos. **Regla:** Cualquier dato que los loaders de ruta necesiten leer de `sharedMap` DEBE escribirse en `onRequest` (middleware, ejecución secuencial y garantizada antes de los loaders), nunca en otro `routeLoader$` del mismo nivel jerárquico.
71. **LL-075** Patrón canónico SSR para rutas protegidas: `createSupabaseAdminClient()` (service role key) en TODOS los `routeLoader$` y `routeAction$` bajo `(app)/`. El cliente user-scoped (`createSupabaseServerClient`) **no** inyecta el JWT en las queries PostgREST bajo `@supabase/ssr` v0.10+ con `skipAutoInitialize: true`, causando error 42501. La seguridad de tenant se delega a `requireRole()` + `.eq('organization_id', orgId)` explícito en cada query. Reservar el client user-scoped para: auth callbacks, signed URLs y realtime.
72. **LL-076** Antes de usar en código cualquier `routeLoader$` cuya tabla depende de una migración pendiente, verificar que la migración está aplicada con `bun run db:migrate` y que Supabase retorna los datos sin error PGRST200. Un FK a una tabla aún no existente en producción genera error 42703 en runtime aunque el schema Drizzle compile sin errores.
73. **LL-077** Supabase JS PostgREST soporta **Embedded Select** para relaciones: `.select('*, case_items(importance)')` retorna `case_items: Array<{importance:number}> | null` inlineado en cada fila — sin JOIN explícito, sin RPC. Usar para campos agregados simples (flags booleanos, counts de sub-items). Registrar el campo en la interfaz de Row DB como `Array<{campo: tipo}> | null`. Nunca usar para columnas de alto volumen (`content`, `description`) — solo campos selectivos. Ver ADR-004.
74. **LL-078** Para COUNT-only en Supabase JS (sin datos): `.select('*', { count: 'exact', head: true })` emite un HEAD request — costo de red mínimo, solo retorna el header `Content-Range`. Usar para sugerencias de secuencia, verificaciones de existencia, contadores de paginación previos al SELECT. No usar para casos donde el offset/cursor ya esté disponible.
75. **LL-079** En Qwik JSX, los atributos de elementos SVG DEBEN estar en **kebab-case** (`stroke-width`, `stroke-linecap`, `stroke-linejoin`), no camelCase. `strokeWidth={1.5}` genera TS2322 porque `LenientSVGProps<SVGSVGElement>` no incluye camelCase. Aplica a todos los SVGs inline en TSX — usar `stroke-width="1.5"` siempre. El atributo `list` de `<input>` tampoco está en los tipos de Qwik: usar spread `{...({ list: 'id' } as Record<string, string>)}`.

---

## 📋 Registro Completo

<!-- Plantilla para nuevas entradas:
### LL-0XX
- **Fecha:**
- **Feature:**
- **Agente responsable:**
- **Error cometido:**
- **Causa raíz:**
- **Regla derivada:**
- **Standard relacionado:**
- **Reincidencias:** 0
- **Estado:** ⏳ Pendiente propagación / ✅ Propagada
-->

### LL-001
- **Fecha:** 2026-03-14
- **Feature:** organization-profile (formulario 3 tabs)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `sync$(() => { signal.value = x })` para mutar un Signal en el tab switcher.
- **Causa raíz:** Confusión entre "sync para evitar petición HTTP" y "sync para DOM puro". `sync$` no integra con el scheduler de Qwik — la mutación del Signal no dispara re-render.
- **Regla derivada:** `sync$` SOLO para APIs de DOM puro (classList, localStorage, scrollTo, focus). Para cualquier mutación de Signal/Store usar `onClick$` async.
- **Standard relacionado:** `DECISIONS_QWIK.md` §3
- **Reincidencias:** 0
- **Estado:** ⏳ Pendiente propagación

---

### LL-002
- **Fecha:** 2026-03-14
- **Feature:** organization-profile (alias ruta configuracion)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Re-export de `routeLoader$` y `routeAction$` desde una ruta hacia otra.
- **Causa raíz:** El optimizer de Qwik City vincula el ID del loader a la ruta donde está declarado, no donde se re-exporta. La ruta destino puede no activar el loader.
- **Regla derivada:** Nunca re-exportar `routeLoader$`/`routeAction$` entre archivos de rutas distintos. Para alias: `throw requestEvent.redirect(302, '/ruta-canonica')`.
- **Standard relacionado:** `DECISIONS_QWIK.md` §10
- **Reincidencias:** 0
- **Estado:** ⏳ Pendiente propagación

---

### LL-003
- **Fecha:** 2026-03-14
- **Feature:** organization-profile (manejo de errores en action)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `requestEvent.fail(500, { message: error.message })` — expone mensaje técnico interno al cliente.
- **Causa raíz:** Ausencia del patrón de dos niveles. Mensajes de Drizzle/PostgreSQL pueden filtrar nombres de columnas, constraints, etc.
- **Regla derivada:** En todo `catch` de `routeAction$`: (1) `Logger.error({ code: 'ORCH_XXX', trace: error.stack })` server-side, (2) `requestEvent.fail(500, { message: 'mensaje genérico' })` al cliente. NUNCA `error.message` directo al cliente.
- **Standard relacionado:** `QUALITY_STANDARDS.md` §3 (Observabilidad)
- **Reincidencias:** 0
- **Estado:** ⏳ Pendiente propagación

---

### LL-004
- **Fecha:** 2026-03-14
- **Feature:** organization-profile (routeAction$ con schema Zod importado)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `routeAction$(handler, zod$(ImportedSchema))` → error TS2589 ("Type instantiation is excessively deep and possibly infinite").
- **Causa raíz:** La `ActionConstructor` tiene múltiples capas de type-level computation. Con schemas importados TypeScript recomputa la cadena completa superando el límite de recursión (~100 niveles). Con schemas inline el tipo se evalúa en el mismo contexto.
- **Regla derivada:** En `routeAction$`, schemas Zod SIEMPRE inline en `zod$(z.object({...}))`. Nunca schemas importados. Si >~8 campos, dividir en sub-acciones por dominio semántico. Los tipos pueden seguir exportándose mediante `z.infer<>`.
- **Standard relacionado:** `DECISIONS_DATA.md` §3
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-005
- **Fecha:** 2026-03-14
- **Feature:** organization-profile (formulario multi-tab)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Un único `<Form action={updateAction}>` con 3 secciones de tabs renderizadas condicionalmente. Inputs de tabs ocultas desmontados del DOM → no se incluían en el FormData → fallos de validación Zod.
- **Causa raíz:** Renderizado condicional de Qwik desmoronta los controles no activos. El atributo `hidden` de HTML oculta pero no garantiza que los inputs estén en el DOM.
- **Regla derivada:** Formularios con tabs: SIEMPRE un `<Form action={acción}>` independiente por cada tab, envuelto en `<div hidden={activeTab.value !== 'tab'}>`. Un único Form nunca abarca campos de múltiples tabs.
- **Standard relacionado:** `DECISIONS_QWIK.md` §1
- **Referencia bug:** `docs/bugs/organization-form-tabs.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-006
- **Fecha:** 2026-03-16
- **Feature:** usersapp-redesign (grid de tarjetas de usuario)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `onClick$={() => startEdit(member)}` — closure captura el objeto `member` completo en lugar de solo su ID.
- **Causa raíz:** Pasar el objeto completo a un QRL obliga a Qwik a serializarlo N veces en el snapshot HTML (qData), aumentando el payload O(N×campos) innecesariamente.
- **Regla derivada:** En `onClick$` dentro de `.map()`, NUNCA pasar el objeto iterado completo. Pasar solo el ID primitivo: `onClick$={() => handler(item.id)}`. Leer el objeto desde `signal.value.find(x => x.id === id)` dentro del handler.
- **Standard relacionado:** `DECISIONS_QWIK.md` §2 · `SERIALIZATION_CONTRACTS.md`
- **Reincidencias:** 0
- **Estado:** ⏳ Pendiente corrección

---

### LL-007
- **Fecha:** 2026-03-16
- **Feature:** usersapp-redesign (server$ saveMember)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `server$(async function(data: SaveMemberData) { ... })` sin validación Zod runtime.
- **Causa raíz:** Confusión entre seguridad de tipos TypeScript (compile-time) y validación de entrada en servidor (runtime). El estándar obliga validación Zod en toda `routeAction$` Y `server$`.
- **Regla derivada:** Toda `server$` con parámetros del cliente DEBE ejecutar `const parsed = Schema.safeParse(data); if (!parsed.success) return { success: false, error: 'Datos inválidos' };` como primera operación.
- **Standard relacionado:** `QUALITY_STANDARDS.md` §5 (Seguro) · `DECISIONS_DATA.md` §3
- **Reincidencias:** 0
- **Estado:** ⏳ Pendiente corrección

---

### LL-008
- **Fecha:** 2026-03-17
- **Feature:** usersapp-list-actions (modal con role=dialog)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Modal con `role="dialog"` y `aria-modal="true"` correctos, pero sin gestión de foco programático. El foco permanece en el botón trigger fuera del dialog.
- **Causa raíz:** `aria-modal="true"` informa a los AT que el contenido exterior es inerte, pero no mueve el foco automáticamente. Sin `focus()` explícito, usuarios de teclado/AT no perciben el dialog al abrirse.
- **Regla derivada:** Todo `role="dialog"` DEBE incluir `useVisibleTask$` que al abrirse ejecute `.focus()` en el primer elemento interactivo (`button, input, select, [tabindex]:not([tabindex="-1"])`). Caso permitido de `useVisibleTask$` (DOM API pura). Al cerrarse, restaurar foco al trigger.
- **Standard relacionado:** `QUALITY_STANDARDS.md` §4 (Accesible) · WCAG 2.1 SC 2.1.2
- **Reincidencias:** 0
- **Estado:** ⏳ Pendiente corrección

---

### LL-009
- **Fecha:** 2026-03-17
- **Feature:** usersapp-list-actions (arquitectura UI)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Extraer sub-secciones CRUD (Header, Grid, Form/Overlay) a archivos separados sin reutilización real fuera de la ruta.
- **Causa raíz:** Aplicar reglas de modularización de SPA tradicionales sin considerar co-localización de QRLs y coste de fragmentación en Qwik.
- **Regla derivada:** En CRUD de dashboard, usar sub-componentes `component$` co-localizados en el mismo `index.tsx`. Extraer a `src/components/` ÚNICAMENTE si hay reutilización real cross-feature.
- **Standard relacionado:** `DECISIONS_QWIK.md` §5 (Co-localización QRLs)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-010- **Fecha:** 03-04-2026
- **Feature:** crm-contacts (M04)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `requireRole()` en `useContactListLoader` leía `sharedMap.get('profile')` y obtenía siempre `undefined`, causando un redirect a `/login?error=session_expired`. El layout público detectaba la sesión activa y redirigía a `/dashboard`. El síntoma observable — URL cambia a `/contacts` y vuelve a `/dashboard` — parecía un fallo de permisos RBAC.
- **Causa raíz:** Los `routeLoader$` del layout padre (`useOrgContext`) y de la ruta hija (`useContactListLoader`) se ejecutan **concurrentemente** via `Promise.all` en Qwik City (confirmado en `node_modules/@builder.io/qwik-city/lib/middleware/request-handler/index.mjs` línea 1242). El `sharedMap.set('profile', profile)` en `useOrgContext` ocurría DESPUÉS de `await db.select(...)`. Para cuando el loader de la ruta hija ejecutaba `requireRole()` síncronamente, el valor aún no existía en sharedMap.
- **Fix aplicado:** Mover la carga de perfil + org desde `useOrgContext` routeLoader$ al `onRequest` middleware de `(app)/layout.tsx`. El middleware se ejecuta SECUENCIALMENTE y ANTES de todos los loaders. Se simplificó `useOrgContext` para leer `sharedMap.get('profile')` y `sharedMap.get('orgName')` ya puestos por `onRequest`.
- **Regla derivada:** Cualquier dato que los loaders de ruta necesiten leer de `sharedMap` DEBE escribirse en `onRequest` (middleware). Nunca confiar en que un `routeLoader$` de layout padre complete su `await` antes de que un loader de ruta hija lea del `sharedMap`. Son concurrentes por diseño del framework.
- **Standard relacionado:** `DECISIONS_DATA.md` §sharedMap · `DECISIONS_QWIK.md` §routeLoader$
- **Reincidencias:** 0 (afecta potencialmente a todas las rutas bajo `(app)/` que usen `requireRole` en loaders)
- **Estado:** ✅ Propagada- **Fecha:** 2026-03-22
- **Feature:** expense-guard-auth (onboarding + callback)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `DATABASE_URL` configurada con el host de conexión directa de Supabase (`db.[ref].supabase.co:5432`) que solo resuelve a IPv6 (registro AAAA únicamente). Drizzle no podía conectar desde redes IPv4-only.
- **Causa raíz:** Supabase en plan gratuito no incluye IPv4 en la conexión directa. La `DATABASE_URL` del ejemplo del dashboard (Direct Connection) usa IPv6. Las redes domésticas/de desarrollo suelen ser IPv4.
- **Regla derivada:** En proyectos Supabase, `DATABASE_URL` SIEMPRE debe apuntar al **Transaction Pooler** (puerto 6543, formato `postgres.[ref]@aws-X-[region].pooler.supabase.com`) — tiene IPv4 garantizado. La `DIRECT_URL` (solo para migraciones `drizzle-kit`) puede usar el host directo. Añadir siempre `ssl: 'require'` al singleton de postgres-js.
- **Standard relacionado:** `DECISIONS_DATA.md` §2 (Drizzle singleton)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-011
- **Fecha:** 2026-03-22
- **Feature:** expense-guard-auth (OAuth callback)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `getMembership()` llamado sin try-catch en `auth/callback/index.tsx`. Si la DB falla, la excepción no capturada rompe el flujo OAuth con Vite overlay en lugar de un redirect seguro.
- **Causa raíz:** Las llamadas a DB en `onRequest`/`onGet` handlers no tienen el safety net de `request.fail()` que sí tienen las `routeAction$`. Cualquier excepción no capturada sube hasta Vite en dev o genera 500 en producción.
- **Regla derivada:** En `onRequest` y `onGet` handlers, toda llamada a servicios externos (DB, APIs) DEBE estar envuelta en try-catch. El fallback debe ser el destino más seguro posible (en auth: `/login` o `/onboarding`). Esta regla aplica especialmente a rutas de callback OAuth donde el usuario ya está autenticado.
- **Standard relacionado:** `QUALITY_STANDARDS.md` §3 (Observabilidad) · `DECISIONS_QWIK.md` §8
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-014
- **Fecha:** 2026-03-22
- **Feature:** expense-guard-subscriptions-crud (M03-A)
- **Agente responsable:** @QwikBuilder Ciclo 1
- **Error cometido:** `requireRole` en handlers de `routeAction$` lanzaba redirect 302 → La Spec (AC-F06) exigía `requestEvent.fail(403, ...)` para members que intentan escritura.
- **Causa raíz:** `requireRole` tenía un único comportamiento (redirect a `/dashboard?error=forbidden`) sin diferenciar entre contexto de navegación GET (`onRequest`) y contexto de action POST (`routeAction$`).
- **Regla derivada:** Añadir `mode: 'redirect' | 'fail' = 'redirect'` a `requireRole`. Usar `mode='fail'` en todos los handlers de `routeAction$` de escritura. Usar default `'redirect'` en `onRequest`. Esto satisface AC-F06 y mantiene buena UX en navegación.
- **Standard relacionado:** `src/lib/auth/guards.ts`, AC-NF03, AC-F06
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-015
- **Fecha:** 2026-03-22
- **Feature:** expense-guard-subscriptions-crud (M03-A)
- **Agente responsable:** @QwikBuilder Ciclo 2
- **Error cometido:** Al intentar usar `requestEv.fail(403, {...})` en `requireRole` (tipado como `RequestEventCommon`), TypeScript lanza TS2339: "Property 'fail' does not exist on type 'RequestEventCommon'".
- **Causa raíz:** `RequestEventAction` (la interfaz que incluye `.fail()`) está definida en `@builder.io/qwik-city/middleware/request-handler` pero NO está re-exportada desde el entry point público del paquete. No se puede importar directamente sin acceder a la ruta interna del módulo.
- **Regla derivada:** Declarar una interfaz local mínima en el fichero que la necesite: `interface RequestEventWithFail { fail(status: number, data: Record<string, unknown>): unknown }`. Castear con `requestEv as unknown as RequestEventWithFail`. Nunca `as any`. Este patrón es seguro porque `mode='fail'` SOLO se pasa desde `routeAction$` handlers donde `requestEv` es efectivamente `RequestEventAction` en runtime.
- **Standard relacionado:** `src/lib/auth/guards.ts`, `DECISIONS_QWIK.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-016
- **Fecha:** 2026-03-22
- **Feature:** expense-guard-alertas (M05)
- **Agente responsable:** @QwikArchitect / @QwikBuilder
- **Error cometido (hipotético):** Proteger un endpoint cron con `===` para comparar el token en lugar de `timingSafeEqual`.
- **Causa raíz:** La comparación `===` entre strings es vulnerable a timing attacks: un atacante puede medir diferencias de tiempo para adivinar token carácter a carácter. Esto aplica a cualquier token de autenticación servicio-a-servicio.
- **Regla derivada:** Endpoints HTTP protegidos por token (cron, webhook, API interna) SIEMPRE deben usar `timingSafeEqual` de `node:crypto`. Incluir guard de longitud previo (`if (a.length !== b.length) return false`) para evitar leaks de longitud. Los tokens de usuario se delegan a Supabase Auth — `timingSafeEqual` es para tokens de infra/servicio.
- **Standard relacionado:** `DECISIONS_QWIK.md` (endpoints puro-HTTP), OWASP Cryptographic Failures
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-017
- **Fecha:** 2026-03-22
- **Feature:** expense-guard-alertas (M05)
- **Agente responsable:** @QwikArchitect
- **Error cometido (limitación):** Intento de aplicar filtro `WHERE monthly_cost >= 50` en Drizzle con columnas de tipo `numeric` que requieren cálculo inline (`cost / 12` para suscripciones anuales).
- **Causa raíz:** Drizzle ORM no soporta expresiones computed (ej: `cost / 12`) en cláusulas `WHERE` de forma tipada con columnas `numeric`. Alternativa con `sql\`\`` es válida pero rompe el tipado estático. El volumen esperado (< 50 filas/día en MVP) hace el filtro JS post-query aceptable.
- **Regla derivada:** Si necesitas filtrar por un valor derivado de una columna `numeric` en Drizzle: (1) ejecutar la query solo con filtros de fecha/status en SQL, (2) aplicar el filtro computado en JS post-query. Documentar el volumen esperado. Si el volumen supera cientos de filas/ejecución, migrar a `sql\`\`` con tipado manual explícito.
- **Standard relacionado:** `DECISIONS_DATA.md` §3 (Drizzle patterns)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-018
- **Fecha:** 2026-03-22
- **Feature:** expense-guard-alertas (M05)
- **Agente responsable:** @QwikArchitect / @QwikBuilder
- **Error cometido (hipotético):** `await Promise.all(recipients.map(r => sendEmail(r)))` en un job de envío de emails — un fallo de Resend en un destinatario aborta todos los envíos pendientes.
- **Causa raíz:** `Promise.all` rechaza en cuanto cualquier promesa falla (fail-fast). En jobs de notificación, el fallo de un destinatario no debe impedir el envío al resto.
- **Regla derivada:** En jobs con múltiples operaciones independientes (envíos de email, webhooks, notificaciones): SIEMPRE usar `Promise.allSettled`. Iterar los `PromiseSettledResult` acumulando `{ fulfilled → emailsSent++, rejected → emailsFailed++, log error }`. El `JobResult` debe reflejar contadores separados para observabilidad.
- **Standard relacionado:** `QUALITY_STANDARDS.md` §3 (Observabilidad), `DECISIONS_DATA.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-019
- **Fecha:** 2026-03-22
- **Feature:** expense-guard-alertas (M05)
- **Agente responsable:** @QwikArchitect
- **Error cometido (decisión aclarada):** Asumir que se necesita una segunda conexión con `SUPABASE_SERVICE_ROLE_KEY` para bypassear RLS en un job server-side.
- **Causa raíz:** Confusión entre los dos mecanismos de Supabase: (A) cliente `@supabase/supabase-js` con anon key — inyecta `auth.uid()` en el contexto de sesión, activando RLS; (B) cliente Drizzle/postgres-js conectado directamente al Transaction Pooler — no inyecta ningún contexto auth, por lo que RLS **no se activa** aunque esté definido. El singleton `db` del proyecto usa el mecanismo B.
- **Regla derivada:** Para jobs server-only que necesitan leer datos de múltiples organizaciones sin restricción de RLS: reutilizar el `db` singleton existente (Drizzle/postgres-js). No es necesario `SUPABASE_SERVICE_ROLE_KEY` si el proyecto ya usa Drizzle directo. El cliente Supabase JS se reserva para operaciones que requieren contexto de usuario autenticado (auth callbacks, signed URLs).
- **Standard relacionado:** `DECISIONS_DATA.md` §2 (Drizzle singleton), ADR-001
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-020
- **Fecha:** 2026-03-22
- **Feature:** expense-guard-alertas (M05)
- **Agente responsable:** @QwikArchitect
- **Error cometido (decisión arquitectónica):** Considerar pg_cron (extensión PostgreSQL) o Edge Functions de Supabase como disparador del job diario de alertas en MVP.
- **Causa raíz:** pg_cron requiere acceso de superuser a la DB y genera acoplamiento entre la lógica de negocio y la infraestructura de base de datos. Las Edge Functions de Supabase tienen latencia cold-start y requieren despliegue y gestión separados. GitHub Actions `schedule: cron` es un trigger externo con visibilidad inmediata en el repositorio, sin infra adicional.
- **Regla derivada:** Para jobs programados en MVP: usar GitHub Actions `schedule: cron` + endpoint HTTP protegido por `CRON_SECRET`. Añadir `workflow_dispatch` para ejecución manual gratuita. Alternativas (pg_cron, Supabase Edge Functions, cron VPS dedicado) solo si: (1) se necesita precisión de segundos, (2) se migra a Supabase hosting con pg_cron habilitado, o (3) el repo no es el lugar adecuado para el trigger.
- **Standard relacionado:** `ARQUITECTURA_FOLDER.md` (decisiones de infra), ADR pattern
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-021
- **Fecha:** 2026-03-23
- **Feature:** auth-core (M01)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `import { z } from 'zod'; routeAction$(handler, zod$(z.object({...})))` → TS2769 "Argument of type 'ZodObject' is not assignable to parameter of type '(zod: typeof import(...internal...)...)'".
- **Causa raíz:** Qwik City bundlea su propia copia interna de Zod. Importar `z` desde el paquete `'zod'` del proyecto crea un mismatch de tipos entre la versión externa y la interna, distinto al TS2589 de LL-004 (que era por recursión profunda).
- **Regla derivada:** En `routeAction$`, SIEMPRE usar la forma factory `zod$((z) => z.object({...}))`. La `z` que recibe la factory es la interna de Qwik City — no importar `z` desde `'zod'` en archivos de rutas. Aplica también a schemas vacíos: `zod$((z) => z.object({}))`.
- **Standard relacionado:** `DECISIONS_DATA.md` §3, LL-004
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-022
- **Fecha:** 2026-03-23
- **Feature:** auth-core (M01) — callback OAuth
- **Agente responsable:** @QwikAuditor (detectado en auditoría)
- **Error cometido:** Validar `?next=` con `startsWith('/') && !includes('://')` — URL `//evil.com` supera ambos checks y los browsers la resuelven como URL externa (OWASP A01: Open Redirect).
- **Causa raíz:** Las URLs protocol-relative (`//host/path`) son una bypass conocida de este patrón. `//` no contiene `://` pero sí indica un host externo.
- **Regla derivada:** En cualquier validación de redirect destination, usar SIEMPRE la triple comprobación: `rawNext.startsWith('/') && !rawNext.startsWith('//') && !rawNext.includes('://')`. Actualiza y complementa LL-013.
- **Standard relacionado:** `QUALITY_STANDARDS.md` §4 (Seguridad), OWASP A01
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-023
- **Fecha:** 2026-03-23
- **Feature:** profiles-public (M02)
- **Agente responsable:** @QwikBuilder / @QwikAuditor
- **Error cometido:** Rutas de contenido público (`/mentors`, `/mentors/:id`) colocadas bajo el grupo `(public)/`, cuyo layout inyecta `<h1>DevLink</h1>` y aplica `max-w-sm` (384 px). Resultado: doble `<h1>` en el DOM + grid visual aplastado (AC-NF-03 FAILED, ciclo 1).
- **Causa raíz:** El grupo `(public)/` fue diseñado exclusivamente para formularios de auth (login, register, reset-password). Al añadir rutas de contenido sin crear un grupo de layout propio, el layout de auth se aplica a páginas de contenido de ancho completo.
- **Regla derivada:** Separar siempre los grupos de layout por propósito: `(public)/` para auth forms (con su `<h1>`, `max-w-sm`, guard de redirect-si-autenticado); `(content)/` para contenido público (layout mínimo: solo `min-h-screen`, sin `<h1>` propio, sin width constraint, sin `onRequest` guard); `(app)/` para dashboard autenticado (con guard obligatorio). No reutilizar un grupo de layout para propósitos distintos aunque ambos sean "sin login requerido".
- **Standard relacionado:** `ARQUITECTURA_FOLDER.md` (grupos de rutas), `DECISIONS_QWIK.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-024
- **Fecha:** 2026-03-23
- **Feature:** profiles-edit (M02)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Uso de sintaxis legacy `text-[var(--color-text-primary)]` en el `<h1>` de `src/routes/(app)/profile/index.tsx` cuando el componente hermano `profile-edit-form.tsx` ya usaba la sintaxis v4 correcta `text-(--color-text-primary)`. El linter de Tailwind v4 reportó el warning.
- **Causa raíz:** Tailwind v4 introdujo la sintaxis CSS-native shorthand para CSS custom properties. La sintaxis `[var(--nombre)]` sigue siendo válida en CSS pero Tailwind v4 la considera legacy y la marca como reescribible. Al copiar clases del dashboard existente (que usa sintaxis legacy) sin adaptar a v4, se propagó el patrón incorrecto.
- **Regla derivada:** En Tailwind v4, toda clase arbitraria que referencie una CSS custom property DEBE usar la sintaxis `(--nombre)` en lugar de `[var(--nombre)]`. Ejemplos: `text-(--color-primary)` ✅, `bg-(--color-surface)` ✅, `border-(--color-border)/30` ✅. La sintaxis `[var(...)]` queda prohibida en código nuevo.
- **Standard relacionado:** `DECISIONS_UI.md` §1 (Tailwind v4 CSS-First)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-025
- **Fecha:** 2026-03-23
- **Feature:** slots-management (M03)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `zod$((z) => z.object({ startTime: z.string().datetime({ offset: true }) }))` — Zod rechaza el formato que genera `<input type="datetime-local">` (`"2026-03-23T10:00"` — sin `Z` ni offset de timezone).
- **Causa raíz:** `.datetime({ offset: true })` de Zod exige formato ISO 8601 estricto con timezone (`Z` o `+HH:MM`). El input HTML `datetime-local` entrega valores locales sin timezone, que son parseable por `new Date()` pero no pasan la validación estricta de Zod.
- **Regla derivada:** En toda `routeAction$` que reciba un campo `datetime-local` del cliente, usar `.string().refine((v) => !isNaN(new Date(v).getTime()), { message: 'Fecha inválida' })` en lugar de `.datetime({ offset: true })`. En el servidor, convertir el string con `new Date(data.startTime)` que acepta el formato sin timezone.
- **Standard relacionado:** `DECISIONS_DATA.md` §3, LL-004, LL-021
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-026
- **Fecha:** 2026-03-23
- **Feature:** slots-management (M03) — slot-cancel-button
- **Agente responsable:** @QwikBuilder (Ciclo 1) / @QwikAuditor (detectado)
- **Error cometido:** En la primera implementación de `SlotCancelButton`, se usó `sync$` con `document.getElementById()` para mantener el dialog. En el Ciclo 1 el `useVisibleTask$` trackeaba `cancelAction.isRunning` en lugar de una Signal de estado del dialog, por lo que el foco nunca se movía al primer botón al `showModal()`.
- **Causa raíz:** Los closures `sync$()` se ejecutan de forma síncrona en el cliente y no pueden capturar Signals de Qwik (no son serializables en el scheduler). Para acceder al DOM dentro de `sync$`, usar `document.getElementById('mi-id')` en lugar de capturar un Signal con `.value`. Para operaciones que necesitan mutar Signals, usar `onClick$` (async). En el Ciclo 2 se corrigió usando una Signal booleana `isDialogOpen` con `onClick$` para mutarla y `useVisibleTask$` trackeando esa Signal para gestionar el foco.
- **Regla derivada:** (1) `sync$` SOLO para APIs de DOM puro sin mutación de Signals — si necesitas mutar un Signal, usa `onClick$`. (2) Para acceder al DOM dentro de `sync$`, usar `document.getElementById('id')` en lugar de capturar un Signal ref. (3) Los `useVisibleTask$` para gestión de foco en dialogs DEBEN trackear una Signal de estado del dialog (`isDialogOpen`), no la action running state.
- **Standard relacionado:** `DECISIONS_QWIK.md` §3, LL-001, LL-008
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-027
- **Fecha:** 2026-03-23
- **Feature:** bookings-flow (M04)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `import { alias } from 'drizzle-orm'` — primer test del servicio falló con `SyntaxError: Export named 'alias' not found in module 'drizzle-orm/index.js'`.
- **Causa raíz:** Drizzle ORM separa los exports por categoria. El módulo raíz `drizzle-orm` exporta operadores genéricos (`eq`, `and`, `or`, `desc`, `inArray`, `sql`…). Los constructores de esquema específicos de PostgreSQL y utilidades de tabla como `alias` viven en el sub-módulo `drizzle-orm/pg-core` junto con `pgTable`, `pgEnum`, etc.
- **Regla derivada:** `alias()` SIEMPRE se importa desde `drizzle-orm/pg-core`, no desde `drizzle-orm`. Dividir los imports de Drizzle en dos líneas: `import { eq, and, or, desc } from 'drizzle-orm'` para operadores genéricos e `import { alias } from 'drizzle-orm/pg-core'` para constructores de tabla y utilidades PG-específicas.
- **Standard relacionado:** `DECISIONS_DATA.md` §2 (cliente Drizzle), `LESSONS_LEARNED.md` LL-010
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-028
- **Fecha:** 2026-03-23
- **Feature:** bookings-flow (M04) — cierre @QwikPolisher
- **Agente responsable:** @QwikPolisher
- **Regla derivada:** Tablas de transacciones (`bookings`, `orders`, `payments`): DELETE bloqueado por RLS (sin política DELETE = denegado) + soft-delete via campo `status` (`'cancelled'`). Nunca borrar registros de transacciones reales. La trazabilidad es un requisito implícito en cualquier marketplace o SaaS con flujo de dinero. En DevLink: `bookings` con `status='cancelled'` permanece en la tabla — historial completo auditado. En `slots-management`, `cancelSlot` sigue el mismo patrón (`status='cancelled'`, nunca DELETE).
- **Standard relacionado:** `DECISIONS_DATA.md` §5 (RLS), `SECURITY_POLICIES.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-029
- **Fecha:** 2026-03-23
- **Feature:** dashboard-student (M05)
- **Agente responsable:** @QwikSpeccer (sobre-especificación en Spec) / detectado por @QwikAuditor (MINOR-02)
- **Error cometido:** La Spec indicaba usar `class="sr-only"` junto a `role="status"` + `aria-live="polite"` en el mensaje de estado vacío del dashboard. El Builder no aplicó `sr-only` y el Auditor verificó que la implementación era la correcta.
- **Causa raíz:** Confusión entre dos patrones de a11y con `aria-live`: (1) live-regions auxiliares invisibles (`sr-only`) que anuncian cambios de estado sin reemplazar el contenido visible, y (2) mensajes de estado vacío que deben ser legibles tanto por screen readers como por usuarios sighted. `sr-only` está diseñado para ocultar contenido visualmente — aplicarlo a un mensaje principal visible es un bug de a11y para usuarios no ciegos.
- **Regla derivada:** Para mensajes de empty state visibles en pantalla: `<p role="status" aria-live="polite">Mensaje visible</p>` **sin** `class="sr-only"`. Reservar `sr-only` exclusivamente para live-regions auxiliares separadas del copy principal (ej: `<span class="sr-only" aria-live="polite">Cargando resultados</span>` junto a un spinner visible). Si el mensaje vacío es el único contenido de la sección, debe ser visible.
- **Standard relacionado:** `DECISIONS_UI.md` §A11Y · `UX_GUIDE.md` §Empty States
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-030
- **Fecha:** 2026-03-23
- **Feature:** email-transactional (M06)
- **Agente responsable:** @QwikBuilder (diseño inicial) / corregido en Ciclo 2 @QwikBuilder + @QwikAuditor
- **Error cometido:** `email.service.ts` instanciaba `new Resend(apiKey)` como singleton módulo-nivel (`let _resend: Resend | null = null`). En tests de bun:test con `mock.module('resend', ...)`, el singleton se inicializaba en el primer test y persistía en los siguientes — los tests que eliminaban `RESEND_API_KEY` fallaban porque el cliente ya estaba cacheado.
- **Causa raíz:** El singleton módulo-nivel funciona para procesos de larga duración (Node.js server con múltiples requests en el mismo proceso), pero rompe la aislación de tests y puede retener credenciales obsoletas si env vars cambian en runtime.
- **Regla derivada:** Clientes de servicios externos (Resend, Stripe, SendGrid, etc.) en SSR fire-and-forget: crear instancia por llamada (`return new Client(apiKey)`), no como singleton módulo-nivel. El coste de `new Resend(apiKey)` es trivial (setup de headers HTTP, no connection pools). Singleton válido solo si el cliente gestiona connection pools persistentes (ej: Drizzle/postgres-js).
- **Standard relacionado:** `DECISIONS_QWIK.md` §Server-Only Modules · `SERIALIZATION_CONTRACTS.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-031
- **Fecha:** 2026-03-23
- **Feature:** index-redirect (Post-Fase 2)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `onRequest(ev).catch((e: Response) => e)` — TypeScript lanza TS2339 "Property 'catch' does not exist on type 'void'" porque `RequestHandler` está tipado como `() => void`.
- **Causa raíz:** Qwik City tipa `RequestHandler` con retorno `void` para forzar el uso de `throw` en lugar de `return` para redirects. En runtime la função es `async` y retorna `Promise<void>` que sí puede rechazar, pero el tipo TS no lo refleja. Usar `.catch()` directamente sobre el tipo `void` es un error de TypeScript.
- **Regla derivada:** Para capturar el redirect (throw de `Response`) en tests de `onRequest`: envolver la llamada con cast `(onRequest(ev) as unknown as Promise<void>)` y usar try-catch. Si `e instanceof Response` → retornar para assertions. Si no → re-throw (es un error real, no un redirect). **No usar** `.catch()` directamente sobre el resultado sin cast.
- **Standard relacionado:** `DECISIONS_QWIK.md` §8 (onRequest pattern) · `TESTING_POLICY.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-032
- **Fecha:** 2026-03-24
- **Feature:** login-session-not-persisting (Bug crítico Post-Fase 2)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `try { throw requestEv.redirect(302, '/dashboard') } catch(e) { if (e instanceof Response) throw e; return requestEv.fail(500, {...}) }` — el redirect quedaba atrapado en el catch, el handler retornaba `fail(500)` y Supabase SSR nunca podía emitir el `Set-Cookie` de sesión.
- **Causa raíz:** `requestEv.redirect()` en Qwik City retorna `RedirectMessage extends AbortMessage`, **no** un objeto `Response` del browser. El check `instanceof Response` es siempre `false` — el redirect se traga silenciosamente. El síntoma visible: ORCH_AUTH_020 en cada login exitoso + `Auth session missing!` (status 400) en cada request autenticada posterior.
- **Regla derivada:** `throw requestEv.redirect()` debe ir **siempre fuera del bloque try-catch**. Patrón correcto: (1) declarar variable de resultado antes del try, (2) ejecutar solo las operaciones async dentro del try-catch, (3) evaluar el resultado y hacer el throw redirect fuera. Nunca usar `instanceof Response` como guard de redirects de Qwik City.
- **Standard relacionado:** `DECISIONS_QWIK.md` §8 (onRequest pattern)
- **Reincidencias:** 0 (detectado en 10 handlers, corregidos en batch)
- **Estado:** ✅ Propagada

---

### LL-033
- **Fecha:** 2026-03-25
- **Feature:** google-oauth-not-working (Bug crítico Post-Fase 2)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `useLoginWithGoogle` (routeAction$) hacía `throw requestEv.redirect(302, oauthUrl)` donde `oauthUrl = 'https://accounts.google.com/...'`. El usuario pulsaba el botón `<Form action={googleAction}>`, la acción se ejecutaba en servidor, pero el redirect a Google nunca llegaba al navegador — pantalla parpadeaba y volvía a `/login`.
- **Causa raíz:** Qwik City intercepta los submits de `<Form action={routeAction$}>` como `fetch POST` a `/ruta/q-data.json`. Internamente, `handleQDataRedirect` llama a `makeQDataPath(location)`, que devuelve `undefined` para cualquier URL que no empiece por `/`. Al recibir `undefined`, el middleware elimina el header `Location` y borra el status 302, devolviendo una respuesta 200 vacía. El cliente Qwik, al no recibir JSON válido, ejecuta `location.href = url.href` (la URL actual), devolviendo al usuario al login. El `code_verifier` de PKCE sí se escribe correctamente en el cookie (confirmado con @supabase/ssr source) — el problema es que el navegador nunca llega a Google para recogerlo.
- **Regla derivada:** Para todo flujo OAuth (Google, GitHub, o cualquier proveedor), usar una ruta `onGet` dedicada (`/auth/proveedor/index.tsx`) que genere la OAuth URL y haga `throw requestEv.redirect(302, externalUrl)`. Este es un redirect HTTP real — el navegador lo sigue nativamente, el `Set-Cookie` del `code_verifier` llega al browser y el flujo PKCE funciona. En el componente de login, usar `<a href="/auth/proveedor">` en lugar de `<Form action={oauthAction}>`. Comportamiento de descarte de URLs externas pendiente de confirmar para otros contextos (pagos, webhooks, etc.).
- **Standard relacionado:** `DECISIONS_QWIK.md` §8 (onRequest pattern), LL-032
- **Archivos afectados:** `src/routes/(public)/auth/google/index.tsx` (nuevo), `src/routes/(public)/login/index.tsx` (eliminado useLoginWithGoogle), `src/components/auth/login-form.tsx` (Form → a)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-035
- **Fecha:** 2026-03-25
- **Feature:** notarylink-signers (M03) — `getSignerByToken`
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `const rows = result.rows as SignerRow[]` — TypeScript lanza TS2339 "Property 'rows' does not exist on type 'RowList<Record<string, unknown>[]>'".
- **Causa raíz:** El tipo de retorno de `db.execute()` en Drizzle ORM con el driver postgres-js es directamente el array de resultados (tipo `RowList`), no un objeto wrapper con propiedad `.rows`. A diferencia del driver pg (node-postgres) que sí devuelve `{ rows: [] }`, postgres-js devuelve el array directamente. El tipo TS de `RowList` no expone `.rows`.
- **Regla derivada:** Al usar `db.execute(sql\`...\`)` con el driver postgres-js: el resultado ya es el array. Pattern correcto: `const rows = await db.execute(sql\`...\`) as unknown as T[]`. Nunca acceder a `.rows` sobre el resultado de `db.execute()` con postgres-js. Si el proyecto migra a node-postgres, revisar este cast.
- **Standard relacionado:** `DECISIONS_DATA.md` §2 (cliente Drizzle), LL-019
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-036
- **Fecha:** 2026-03-25
- **Feature:** notarylink-viewer (M04) — `useSubmitSignature` routeAction$
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Se intentó usar `z.string().min(5000).startsWith('data:image/png;base64,')` en el schema `zod$()` de un `routeAction$`. En runtime lanza `TypeError: z.string(...).min(...).startsWith is not a function`.
- **Causa raíz:** El bundle de Zod integrado en Qwik City para validación en `routeAction$` es una versión reducida que omite métodos no críticos (como `.startsWith()`, `.endsWith()`) para reducir el tamaño del bundle del servidor. Aunque TypeScript puede no detectar el error en compilación, el método falla en runtime.
- **Regla derivada:** En `zod$()` dentro de `routeAction$` usar únicamente validaciones básicas: `.string()`, `.uuid()`, `.email()`, `.min(n)`, `.max(n)`, `.optional()`, `.nullable()`, `.enum([...])`. Validaciones de formato avanzadas (prefijo de string, regex complejo, `.startsWith()`, `.endsWith()`) deben moverse a la primera línea del servicio que procesa el dato. Para Base64 de PNG: validar en `uploadSignaturePng()` antes del upload a Storage.
- **Standard relacionado:** `DECISIONS_QWIK.md` §5 (validación en routeAction$), LL-021
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-075
- **Fecha:** 04-04-2026
- **Feature:** crm-contacts (M04) + crm-cases (M05)
- **Agente responsable:** @QwikBuilder / @QwikAuditor
- **Error cometido:** Uso de `createSupabaseServerClient()` (user-scoped) en `routeLoader$` y `routeAction$` bajo `(app)/`. Causaba error PostgreSQL `42501` (violations de RLS) incluso con usuario autenticado.
- **Causa raíz:** `@supabase/ssr` v0.10 introduce `skipAutoInitialize: true`. Las queries PostgREST no reciben el header `Authorization: Bearer <jwt>` — el cliente cae al rol `anon` que no tiene permisos de escritura sobre tablas con RLS. `getUser()` funciona (llama al endpoint REST de Auth directamente) pero las queries de datos fallan.
- **Regla derivada:** Usar `createSupabaseAdminClient()` (service role key) en TODOS los `routeLoader$` y `routeAction$` bajo `(app)/`. Seguridad de tenant garantizada por (1) `requireRole()` en `onRequest` + (2) `.eq('organization_id', orgId)` explícito en cada query. Reservar `createSupabaseServerClient()` para: auth callbacks, signed URLs de Storage, Realtime subscriptions.
- **Standard relacionado:** `DECISIONS_DATA.md` §SSR Client Pattern
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-076
- **Fecha:** 04-04-2026
- **Feature:** crm-cases v1.0 (M05) — bugs post-build
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `routeLoader$` de `/cases` fallaba con error `42703` (columna no encontrada) en producción porque `case_items` (dependency de `cases.service.ts`) aún no existía en la DB.
- **Causa raíz:** La migración `0007_case_items` estaba generada en el filesystem pero pendiente de aplicar. El servicio compilaba sin errores porque Drizzle schema y TypeScript no validan contra la DB real. El error solo aparece en runtime cuando Supabase intenta resolver el embedded select `case_items(importance)`.
- **Regla derivada:** Antes de desplegar cualquier loader/action que referencie una tabla nueva: ejecutar `bun run db:migrate` y verificar con una query de prueba que la tabla existe y la FK resuelve sin errores PGRST200/42703. El schema Drizzle y los tests unitarios (con mocks) no detectan FK inexistentes en la DB real.
- **Standard relacionado:** `DECISIONS_DATA.md` §Migraciones
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-077
- **Fecha:** 04-04-2026
- **Feature:** crm-cases v1.1 (M05) — hasCriticalItems
- **Agente responsable:** @QwikAuditor (detectado) / @QwikBuilder (corregido)
- **Error cometido:** `hasCriticalItems` inicialmente hardcodeado a `false` en `mapListRow` en lugar de computarlo desde los `case_items` de cada expediente.
- **Causa raíz:** Desconocimiento de la sintaxis de Embedded Select de PostgREST. Se asumió erróneamente que era necesaria una sub-query o RPC separada para obtener el agregado.
- **Regla derivada:** Supabase JS PostgREST soporta Embedded Select: `.select('id, title, ..., case_items(importance)')` retorna `case_items: Array<{importance:number}> | null` inlineado por fila. Usar para flags booleanos sin query adicional: `(row.case_items ?? []).some(i => i.importance === 3)`. Declarar en la interfaz de Row DB como `case_items: Array<{importance: number}> | null`. Ver ADR-004.
- **Standard relacionado:** `DECISIONS_DATA.md`, ADR-004
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-078
- **Fecha:** 04-04-2026
- **Feature:** crm-cases v1.1 (M05) — suggestCaseNumber
- **Agente responsable:** @QwikAuditor (detectado) / @QwikBuilder (corregido)
- **Error cometido:** `suggestCaseNumber` usaba ILIKE con prefijo `EXP-${YEAR}-` para contar casos del año actual. Generaba: formato no especificado en spec, colisiones en año nuevo, y overhead de string matching.
- **Causa raíz:** Implementación asumida de un formato no documentado. La spec indica `EXP-NNN` sin year-prefix.
- **Regla derivada:** Para COUNT-only en Supabase JS: `.select('*', { count: 'exact', head: true })` emite HEAD request — sin datos transferidos, solo header `Content-Range`. Patrón para secuencias sugeridas: `EXP-${String((count ?? 0) + 1).padStart(3, '0')}`. Leer la spec literalmente antes de implementar formatos de referencia. Ver ADR-004.
- **Standard relacionado:** `DECISIONS_DATA.md`, ADR-004
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-079
- **Fecha:** 04-04-2026
- **Feature:** crm-cases v1.1 (M05) — @QwikPolisher empty state SVG
- **Agente responsable:** @QwikPolisher
- **Error cometido:** SVG inline con `strokeWidth={1.5}` y `strokeLinecap="round"` (camelCase) — TypeScript lanzaba TS2322 `LenientSVGProps<SVGSVGElement>` no incluye `strokeWidth`.
- **Causa raíz:** Qwik JSX define los atributos SVG en kebab-case siguiendo la especificación SVG/HTML5, no en camelCase como React. `LenientSVGProps` incluye `stroke-width` y `stroke-linecap`, no sus versiones camelCase.
- **Regla derivada:** En Qwik JSX, SVGs SIEMPRE con atributos kebab-case: `stroke-width`, `stroke-linecap`, `stroke-linejoin`, `fill-rule`, `clip-path`, etc. El atributo `list` de `<input>` tampoco está en los tipos de Qwik Input — usar spread assertion: `{...({ list: 'datalist-id' } as Record<string, string>)}`.
- **Standard relacionado:** `DECISIONS_QWIK.md`, `DECISIONS_UI.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

### LL-037
- **Fecha:** 2026-03-25
- **Feature:** notarylink-viewer (M04) — tests de servicios (`ai.service.test.ts`, `signature.service.test.ts`, `signer-session.service.test.ts`)
- **Agente responsable:** @QwikBuilder / @QwikPolisher (detectado en cierre)
- **Error cometido:** `ai.service.test.ts` registró `mock.module('~/lib/db/schema', () => ({ aiAnalysis: {...} }))` con solo 3 exports. Los archivos de test ejecutados después en el mismo proceso (`signature.service.test.ts`, `signer-session.service.test.ts`) fallaron con `SyntaxError: Export named 'documents'/'profiles'/'signatures' not found in module '~/lib/db/schema'`.
- **Causa raíz:** Bun v1.3.x ejecuta todos los archivos de test en el mismo proceso y comparte el registry de `mock.module()`. El primer registro de un módulo gana y no puede ser sobrescrito por archivos posteriores. Un mock parcial (que solo exporta un subset de los módulos reales) contamina todos los test files que se ejecuten a continuación en la misma invocación de `bun test`.
- **Regla derivada:** Cuando varios test files comparten un mock del mismo módulo, el archivo que primero lo registre (orden alfabético en `bun test`) debe incluir todos los exports que cualquier test de la suite necesite. Para módulos grandes como `~/lib/db/schema`, incluir todos los exports del real module como `{}` (objeto vacío) excepto los que el propio test personalice. Alternativa: ejecutar cada test file en proceso separado (`bun test path/to/file.test.ts`). Documentar en el propio test: `// Mock comprehensivo: incluye exports no usados por este archivo para evitar registry pollution en bun 1.3.x`.
- **Standard relacionado:** `TESTING_POLICY.md` §K (cobertura de servicios)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-038
- **Fecha:** 2026-03-26
- **Feature:** notarylink-pdf-consolidation (M05) — `pdf.service.test.ts`
- **Agente responsable:** @QwikAuditor (error en reporte) / @QwikBuilder (corrección)
- **Error cometido:** El reporte de @QwikAuditor incluía el hash SHA-256 esperado `2d2da19605a3e959b3cf4e70f42e81f0ee4a34d8566a0c2a22bb7f02e2ff8e9` calculado offline vía shell. El valor correcto producido por `node:crypto` sobre `Buffer.from(new ArrayBuffer(10))` en runtime es `01d448afd928065458cf670b60f5a594d735af0172c8d67f22a81680132681ca`. El test falló en el primer `bun test` del Ciclo 2.
- **Causa raíz:** `ArrayBuffer(10)` inicializado en JavaScript (10 bytes `0x00`) no produce el mismo hash que comandos shell como `printf '\x00%.0s' {1..10} | sha256sum` o `echo -n ...` debido a diferencias en encoding, nulos de terminal y comportamiento de piping entre plataformas. La fuente de verdad para hashes en tests unitarios debe ser siempre el runtime de Node.js/Bun, no herramientas CLI.
- **Regla derivada:** Para establecer el valor esperado en un golden test de hash: (1) escribir el test primero sin la aserción `toBe`, (2) ejecutar `bun test` y capturar el valor impreso/recibido de `node:crypto` real, (3) fijar ese valor como `EXPECTED_HASH` en el test. Nunca calcular el valor esperado offline con herramientas shell ni calculadoras online. Aplica a SHA-256, SHA-1, MD5 y cualquier función hash sobre buffers en memoria.
- **Standard relacionado:** `TESTING_POLICY.md` §K (cobertura de servicios)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-039
- **Fecha:** 2026-03-26
- **Feature:** notarylink-dashboard (M06) — `layout.tsx` sidebar mobile toggle
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `(e.currentTarget as HTMLButtonElement).setAttribute('aria-expanded', 'true')` dentro del handler `openMenu` declarado con `sync$`. En runtime: `TypeError: e.currentTarget.setAttribute is not a function`.
- **Causa raíz:** `sync$` en Qwik se serializa como un string literal de función JS que se ejecuta como listener nativo del DOM. El valor de `e.currentTarget` en ese contexto depende de cómo Qwik re-despacha el evento internamente — puede ser `null` o el nodo raíz del documento cuando el evento burbujea, no el elemento concreto que tenía el listener en el momento del click. El cast TypeScript `as HTMLButtonElement` solo oculta el problema en compilación.
- **Regla derivada:** En `sync$`, NUNCA usar `e.currentTarget` para manipular atributos o clases. Acceder siempre al elemento destino mediante `document.getElementById('id-estable')`. El ID del elemento es un primitivo estable capturado en el string serializado; `currentTarget` es contextual y frágil. El parámetro `e: Event` en `sync$` solo es fiable para leer `e.key`, `e.type` o invocar `e.preventDefault()` / `e.stopPropagation()`.
- **Standard relacionado:** `DECISIONS_QWIK.md` §3, `SERIALIZATION_CONTRACTS.md`, LL-026 (Signals en sync$)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-040
- **Fecha:** 2026-03-26
- **Feature:** notarylink-email-transactional (M07)
- **Agente responsable:** @QwikBuilder / @QwikPolisher
- **Error cometido:** Al integrar `email.service.ts` (con guard de env vars a nivel de módulo) en `signer.service.test.ts`, el test fallaba con `Error: SERV_EMAIL_CONFIG_MISSING` porque el import dinámico de `signer.service` cargaba transitivamente `email.service`, activando el guard antes de que las env vars de test estuviesen configuradas.
- **Causa raíz:** El guard module-level de env vars se ejecuta en el momento de la carga del módulo (`import()`). Si un test no mockea las deps del módulo bajo test antes de su import dinámico, cualquier dep con guard lanzará en cold-load.
- **Regla derivada:** Antes del `await import('./mi.service')` en cualquier test, registrar un `mock.module` para TODOS los módulos importados por `mi.service` que tengan guards o side-effects de inicialización. Esta regla amplía LL-037 al escenario de deps transitivas con guards de módulo.
- **Standard relacionado:** `SERIALIZATION_CONTRACTS.md` §Guard Pattern, `DECISIONS_QWIK.md` §Testing
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

### LL-041
- **Fecha:** 2026-03-27
- **Feature:** notarylink-viewer (M04) — `pdf-viewer.tsx`
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Intentar reasignar un `let` de módulo dentro de un `useVisibleTask$` y no usar `noSerialize` para el proxy de PDF.js.
- **Causa raíz:** Qwik optimiza los QRLs moviéndolos a ficheros independientes donde las variables capturadas se tratan como constantes. Objetos complejos no-serializables (PDF.js) rompen el snapshot si no se marcan explícitamente.
- **Regla derivada:** Objetos de terceros van en `useStore({ doc: noSerialize(obj) })`. Nunca usar variables mutables fuera del componente para lógica reactiva.
- **Standard relacionado:** `DECISIONS_QWIK.md` §Serialization
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-042
- **Fecha:** 2026-03-27
- **Feature:** notarylink-services (M02) — `generateSignedUrl`
- **Agente responsable:** @QwikAuditor
- **Error cometido:** El enlace de descarga de Supabase fallaba con `"exp" claim timestamp check failed`.
- **Causa raíz:** Tokens de sesión persistentes en el navegador que no coinciden con la ventana de tiempo del servidor o el TTL del JWT tras periodos de inactividad o cambios de red.
- **Regla derivada:** Ante errores de "InvalidJWT" en Storage, el primer paso de debug es un re-login completo para garantizar que el `access_token` inyectado en el cliente de Supabase esté fresco.
- **Standard relacionado:** `SECURITY_POLICIES.md` §Token Management
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-043
- **Fecha:** 2026-03-27
- **Feature:** notarylink-signer-session (M04)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `r.expires_at.toISOString is not a function`.
- **Causa raíz:** `postgres-js` (Drizzle) devuelve las columnas `timestamptz` como Strings ISO 8601 en el entorno SSR de Bun, no como objetos Date nativos.
- **Regla derivada:** Toda fecha proveniente de DB debe pasar por un helper de conversión: `new Date(value)`. No asumir que el tipado de Drizzle garantiza un objeto Date en el runtime del servidor.
- **Standard relacionado:** `DECISIONS_DATA.md` §Type Conversion
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-044
- **Fecha:** 2026-03-27
- **Feature:** notarylink-signature-canvas (M05)
- **Agente responsable:** @QwikPolisher
- **Error cometido:** Trazo de firma descalibrado (offset) respecto a la posición del ratón.
- **Causa raíz:** Desajuste entre píxeles lógicos (CSS) y píxeles físicos del buffer en pantallas HiDPI/Retina. El canvas se estiraba por CSS sin ajustar su resolución interna.
- **Regla derivada:** Inicializar canvas multiplicando `offsetWidth` por `window.devicePixelRatio`. Calibrar coordenadas restando el `rect.left` y aplicando el ratio de escala: `x = (clientX - rect.left) * (canvas.width / rect.width) / dpr`.
- **Standard relacionado:** `DECISIONS_UI.md` §Canvas Precision
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-045
- **Fecha:** 2026-03-27
- **Feature:** notarylink-payments-stripe (M08)
- **Agente responsable:** @QwikBuilder / @QwikAuditor
- **Error cometido:** El usuario pagaba en Stripe pero permanecía en el plan Starter. La suscripción nunca se sincronizaba en la DB.
- **Causa raíz:** Triple fallo encadenado:
  1. `checkout.session.metadata.profileId` **no se replica** al objeto `Stripe.Subscription`. Leer `sub.metadata['profileId']` en `checkout.session.completed` siempre devuelve `null`.
  2. Sin `profileId`, el webhook no podía hacer UPSERT en `subscriptions`.
  3. Si la tabla `profiles` estaba vacía (TRUNCATE en dev/staging), el INSERT en `subscriptions` fallaba por FK violation antes de llegar al problema anterior.
- **Regla derivada:**
  - Usar siempre `client_reference_id: profileId` en `stripe.checkout.sessions.create`. Este campo SÍ está disponible en el objeto `checkout.session.completed` del webhook.
  - Pasar `client_reference_id` como `knownProfileId` a `syncSubscription(subId, profileId)` para saltarse la resolución por metadata.
  - Aplicar **Double Lazy Creation**: antes de insertar en `subscriptions`, verificar que el perfil existe en `profiles`. Si no, `INSERT ... ON CONFLICT DO NOTHING` (idempotente ante concurrencia). Log `SERV_PAY_PROFILE_RECOVERED`.
  - Usar UPSERT (`onConflictDoUpdate`) en `subscriptions` en lugar de UPDATE puro: cubre el caso en que la fila aún no existe cuando llega el primer webhook.
- **Standard relacionado:** `DECISIONS_DATA.md` §Lazy Creation · `SERIALIZATION_CONTRACTS.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-047
- **Fecha:** 2026-03-27
- **Feature:** notarylink-team-multiuser (M09)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Se re-exportó `useOrgContext` desde `src/routes/(app)/dashboard/team/index.tsx` usando `export { useOrgContext } from '~/routes/(app)/layout'`. Esto causó el error en runtime: `The same routeLoader$ (useOrgContext_routeLoader_At0oi3qAstw) was exported in multiple modules`.
- **Causa raíz:** Qwik City identifica cada `routeLoader$` por un ID estable derivado del módulo donde se **define**. Cuando ese mismo export aparece en un segundo módulo (aunque sea solo un re-export), el bundler de Vite registra el mismo loader en dos entradas del manifiesto → colisión de IDs en tiempo de ejecución.
- **Regla derivada:**
  - Los `routeLoader$` definidos en un layout padre están disponibles en todas las rutas hijas **por herencia automática de Qwik City**. No es necesario, ni correcto, re-exportarlos.
  - Para consumir un loader del layout padre en un componente hijo: llamar `useOrgContext()` directamente (solo invocación, sin re-export).
  - Para acceder a los datos en un `routeLoader$` hijo: leer directamente de `sharedMap` (el middleware padre ya lo populó en `onRequest`).
  - **Nunca** hacer `export { useXxx } from '~/routes/(app)/layout'` en una sub-ruta.
- **Standard relacionado:** `DECISIONS_QWIK.md` §routeLoader$ — `ARQUITECTURA_FOLDER.md` §Orchestrator Pattern
- **Reincidencias:** 0
- **Estado:** ✅ Propagada
- **Estado:** ✅ Propagada

---

### LL-048
- **Fecha:** 2026-03-28
- **Feature:** notarylink-notifications (M11)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Se llamó `getSession(this)` dentro de una `server$` en `NotificationBell.tsx`, generando un type mismatch entre `RequestEventBase` (lo que expone `server$`) y `RequestEventCommon` (lo que espera `getSession`).
- **Causa raíz:** `server$` en Qwik usa `RequestEventBase`, que no incluye el contrato completo de `RequestEventCommon`. El `onRequest` del layout `(app)/` ya resuelve la sesión y la escribe en `sharedMap` — llamar `getSession(this)` es redundante y provoca el error.
- **Regla derivada:** En toda `server$` bajo el grupo `(app)/`, leer el usuario autenticado con `this.sharedMap.get('user')`. Reservar `getSession(this)` para contextos donde no existe middleware previo que garantice la sesión.
- **Standard relacionado:** `DECISIONS_QWIK.md` — `SERIALIZATION_CONTRACTS.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-050
- **Fecha:** 2026-03-28
- **Feature:** notarylink-notifications (M11)
- **Agente responsable:** @QwikArchitect / @QwikDBA
- **Error cometido:** Diseño inicial del canal Realtime sin filtro `user_id=eq.{userId}`, asumiendo que RLS era suficiente.
- **Causa raíz:** Supabase Realtime hereda las policies RLS SELECT activas, pero añadir el filtro en el canal es una capa adicional de defensa en profundidad que reduce tráfico innecesario al cliente y protege ante posibles cambios de configuración de RLS.
- **Regla derivada:** En canales Realtime de Supabase siempre incluir `filter: 'user_id=eq.{userId}'`. La RLS es el mecanismo de seguridad primario; el filtro de canal es la segunda barrera. No es correcto omitirlo aunque RLS esté activo. La policy INSERT debe ser permisiva para `authenticated` para que los servicios server-side puedan insertar notificaciones en nombre de otros usuarios (fire-and-forget cross-user).
- **Standard relacionado:** `DECISIONS_DATA.md` §RLS — `RBAC_ROLES_PERMISSIONS.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-051
- **Fecha:** 2026-03-28
- **Feature:** notarylink-public-api (M12)
- **Agente responsable:** @QwikSpeccer / @QwikBuilder
- **Error cometido:** Primer borrador del handler `DELETE /api/v1/api-keys/:id` respondía `{ revoked: true }` — campo booleano sin identidad del recurso ni estado resultante.
- **Causa raíz:** El contrato de respuesta no estaba especificado en la Spec antes de codificar. El boolean `revoked: true` es ambiguo: ¿confirma la acción o el estado actual del recurso?
- **Regla derivada:** En APIs REST, los endpoints DELETE deben responder con el objeto resultante mínimo: `{ id: string, status: 'revoked' | 'deleted' }`. Identifica el recurso afectado y expresa el estado resultante sin ambigüedad. Documentar el contrato de respuesta en la Spec (sección "Contratos de API") antes de codificar el handler.
- **Standard relacionado:** `DECISIONS_DATA.md` §API Contracts
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-052
- **Fecha:** 2026-03-28
- **Feature:** notarylink-public-api (M12)
- **Agente responsable:** @QwikArchitect / @QwikBuilder
- **Error cometido:** Intento de capturar el resultado de `createTenantDb` directamente en un closure `$()` para reutilizar la conexión entre llamadas.
- **Causa raíz:** `createTenantDb` devuelve una instancia de Drizzle ORM que contiene la conexión activa (objeto no serializable). Capturarla en un closure `$()` viola la frontera de serialización de Qwik: el objeto no puede cruzar de servidor a cliente ni persistirse en el manifiesto.
- **Regla derivada:** Los objetos devueltos por `createTenantDb` (y cualquier instancia de ORM/DB) son **no serializables**. Instanciarlos siempre dentro del cuerpo de `server$`, `routeLoader$` o `routeAction$` — nunca capturarlos en variables de módulo ni en cierres `$()`. La factory está marcada SERVER-ONLY pero el guard está en la disciplina de uso.
- **Standard relacionado:** `SERIALIZATION_CONTRACTS.md` §Closures · `DECISIONS_DATA.md` §Tenant Isolation
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-053
- **Fecha:** 2026-03-28
- **Feature:** notarylink-public-api (M12)
- **Agente responsable:** @QwikBuilder / @QwikAuditor
- **Error cometido:** Uso de `await import('~/lib/services/...')` (dynamic import) dentro de un handler en `src/routes/api/v1/documents/[id]/send/index.ts`, causando que Vite incluyera el módulo en el grafo del bundle cliente.
- **Causa raíz:** Los archivos bajo `src/routes/api/v1/` son endpoints HTTP puros (no rutas Qwik City con QRLs). Vite analiza el grafo de imports desde el punto de entrada — los dynamic imports `await import()` dentro de estos handlers son procesados por Vite como posibles splits de cliente, filtrando dependencias server-only al bundle del navegador.
- **Regla derivada:** En handlers bajo `src/routes/api/v1/` (y cualquier endpoint server-only sin componentes Qwik), usar **static imports** en la cabecera del archivo. El `dynamic import` solo tiene sentido en componentes `.tsx` para code-splitting controlado por Qwik. Los handlers REST no tienen contratos de serialización QRL — no se benefician del splitting dinámico y sí asumen el riesgo de filtrar módulos server al cliente.
- **Standard relacionado:** `DECISIONS_QWIK.md` §Bundle Boundaries · `ARQUITECTURA_FOLDER.md` §Orchestrator Pattern
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-054
- **Fecha:** 2026-03-28
- **Feature:** notarylink-templates (M10) — detectado en cierre M12
- **Agente responsable:** @QwikAuditor / @QwikBuilder
- **Error cometido:** `template.service.ts` exponía `dbClient: typeof db` como primer parámetro en 10 funciones (DI pattern). Las rutas `templates/index.tsx` y `templates/new/index.tsx` importaban `import { db } from '~/lib/db/client'` a nivel de módulo para satisfacer el parámetro. Esto filtró `postgres/src/connection.js` (con uso de `perf_hooks`, paquete Node-only) al bundle cliente, causando error de build.
- **Causa raíz:** Qwik City/Rollup analiza las importaciones top-level de forma estática durante el bundling. Las importaciones a nivel de módulo aparecen en el grafo de dependencias de AMBOS chunks (servidor y cliente), independientemente de si se usan solo dentro de `routeLoader$` o `routeAction$`. El patrón DI forzaba a las rutas a depender del módulo `db/client.ts` → `drizzle-orm/postgres-js` → `postgres` → `perf_hooks`. Una fuga indirecta adicional existía en `layout.tsx` → `session.ts` → `db/client.ts`.
- **Regla derivada:** Los servicios de base de datos **nunca** deben exponer la conexión `db` como parámetro a sus callers. La instancia `db` debe internalizarse dentro del módulo del servicio con su propio `import { db }`. Para fugas indirectas irresolubles por cadenas de imports en layout/auth, añadir `build.rollupOptions.external: ['postgres', 'perf_hooks']` a `vite.config.ts` como barrera definitiva. Verificar en `dist/build/` que los chunks con `"interactivity": 0` en el q-manifest son los únicos que contienen referencias a paquetes server-only.
- **Standard relacionado:** `SERIALIZATION_CONTRACTS.md` §Bundle Boundaries · `DECISIONS_DATA.md` §Service Layer · `ARQUITECTURA_FOLDER.md` §Orchestrator Pattern
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-055
- **Fecha:** 2026-03-29
- **Feature:** notarylink-resiliencia-industrial (M13)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** `mock.module('@upstash/qstash', ...)` aplicado en una suite de tests contaminó el module registry global de Bun, causando que otras suites que no necesitaban el mock se ejecutaran con el módulo reemplazado.
- **Causa raíz:** Bun 1.3.11 usa un module registry compartido entre todas las suites ejecutadas en el mismo proceso. `mock.module()` es persistente dentro del proceso — no se resetea entre archivos de test automáticamente.
- **Regla derivada:** Nunca usar `mock.module()` para dependencias externas en tests de servicios de infraestructura. En su lugar, implementar las funciones directamente en el test usando `node:crypto` u otros built-ins, o aislar el módulo con un wrapper inyectable. Si `mock.module()` es imprescindible, aislar la suite en un proceso separado (`bun test --watch` individual o `bun test [file]`).
- **Standard relacionado:** `PROJECT_RULES_CORE.md` §Testing
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-056
- **Fecha:** 2026-03-29
- **Feature:** notarylink-resiliencia-industrial (M13)
- **Agente responsable:** @QwikAuditor
- **Error cometido:** Primera implementación de HMAC en `qstash.service.ts` usaba comparación directa de strings (`hmac === expected`) en lugar de `timingSafeEqual`.
- **Causa raíz:** El AC de seguridad requería explícitamente `timingSafeEqual` (timing-safe comparison). La comparación de strings en JavaScript es vulnerable a timing attacks: el intérprete cortocircuita la comparación en el primer byte diferente, filtrando información sobre el secreto.
- **Regla derivada:** Toda comparación de secretos criptográficos (HMAC, API keys, tokens) **debe** usar `crypto.timingSafeEqual()`. Es un AC binario en auditorías de seguridad — no puede satisfacerse con alternativas ni mocks. Incluirlo en la checklist de cualquier servicio que verifique firmas o tokens.
- **Standard relacionado:** `QUALITY_STANDARDS.md` §Security · `DECISIONS_DATA.md` §API Keys
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-057
- **Fecha:** 2026-03-29
- **Feature:** notarylink-resiliencia-industrial (M13)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** El receptor de webhooks QStash respondía 200 con `{ success: false }` cuando ocurría un error de base de datos durante el procesamiento.
- **Causa raíz:** El sistema de reintentos de QStash (y del dispatcher local) interpreta cualquier respuesta 2xx como "entrega exitosa" y no reintenta. Responder 200 ante un error DB silencia el fallo y pierde el evento permanentemente.
- **Regla derivada:** Los receptores de webhooks con reintentos deben responder **500** ante errores transitorios (DB no disponible, timeout de red) y **200** solo cuando el evento ha sido procesado correctamente. Los errores 4xx (payload inválido, recurso no encontrado) son no-reintentables y deben rechazarse con el código apropiado. Nunca usar 200 como "ack silencioso" de errores.
- **Standard relacionado:** `DECISIONS_DATA.md` §Webhook Reliability
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-058
- **Fecha:** 2026-03-29
- **Feature:** notarylink-resiliencia-industrial (M13)
- **Agente responsable:** @QwikBuilder / @QwikAuditor
- **Error cometido:** `encryption.service.ts` llamaba `loadEncryptionKey()` al nivel de módulo (top-level), haciendo que el servidor de desarrollo lanzara un error fatal al importar el módulo si `ENCRYPTION_KEY` no estaba configurada en `.env`.
- **Causa raíz:** Node.js/Bun ejecuta el código top-level de un módulo en el momento de la primera importación. Una variable de entorno requerida que no existe en desarrollo bloquea completamente cualquier ruta que importe el servicio, aunque no vaya a cifrar nada en esa request.
- **Regla derivada:** Los servicios que dependen de variables de entorno opcionales en desarrollo **deben** usar **Lazy Validation**: resolver la variable dentro del método (`encrypt()`, `decrypt()`) que la necesita, no al importar el módulo. Esto permite que el servidor arranque normalmente y lanza el error solo cuando la funcionalidad se usa realmente. El servidor de desarrollo no debe requerir todas las variables de producción para arrancar.
- **Standard relacionado:** `PROJECT_RULES_CORE.md` §Environment · `DECISIONS_QWIK.md` §Module Initialization
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-059
- **Fecha:** 2026-03-29
- **Feature:** notarylink-resiliencia-industrial (M13)
- **Agente responsable:** @QwikArchitect
- **Error cometido:** El diseño inicial de M13 asumía Upstash QStash como dependencia de infraestructura obligatoria para la cola de webhooks, añadiendo complejidad operacional (cuenta Upstash, credenciales, endpoint externo, latencia de red).
- **Causa raíz:** Para MVP, la durabilidad extrema de una cola externa (persistencia multi-región, dead-letter queue, rate limiting cloud) es sobreingeniería. Los webhooks de notarylink tienen volumen bajo y toleran reintentos en memoria con backoff.
- **Regla derivada:** Para MVPs con volumen de webhooks bajo-medio (<1000/día), un **dispatcher local** con `setTimeout` + backoff exponencial es suficiente. Implementar la interfaz abstracta (`IQStashService`) primero y la implementación concreta después — esto permite migrar a Upstash real en el futuro sin cambiar los callers. Solo escalar a infraestructura de colas externa cuando el volumen o los requisitos de durabilidad (reintentos post-crash, dead-letter queue) lo justifiquen.
- **Standard relacionado:** `DECISIONS_DATA.md` §Infrastructure Choices · `ARQUITECTURA_FOLDER.md` §Service Interfaces
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-060
- **Fecha:** 2026-03-29
- **Feature:** notarylink-ui-audit (M15)
- **Agente responsable:** @QwikBuilder / @QwikAuditor
- **Error cometido:** Primera implementación de diálogos de confirmación usaba el atributo `open` de `<dialog>` gestionado manualmente, sin llamar a `.showModal()`. Esto rompía scroll-lock, focus-trap y la tecla Escape nativos del navegador.
- **Causa raíz:** El atributo `open` abre el diálogo en modo no-modal (sin backdrop, sin focus-trap). `.showModal()` es el único método que activa el comportamiento modal completo: bloquea el scroll del body, atrapa el foco dentro del diálogo y cierra con Escape vía evento `close` nativo.
- **Regla derivada:** Siempre abrir diálogos con `el.showModal()` (nunca con `el.setAttribute('open', '')`). Usar `sync$` para el handler que llama a `.showModal()` (interacción síncrona de DOM). Unificar los tres paths de cierre (Escape, clic en backdrop, botón cancelar) en un único handler `onClose$` que reacciona al evento nativo `close` del `<dialog>`. Para clic en backdrop: `sync$` que compara `event.target === dialogEl` y llama `.close()`. Registrar ref con `useSignal<HTMLDialogElement>()`.
- **Standard relacionado:** `DECISIONS_QWIK.md` §sync$ · `UX_GUIDE.md` §Modals · `DECISIONS_UI.md` §Dialog
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-061
- **Fecha:** 2026-03-29
- **Feature:** notarylink-ui-audit (M15)
- **Agente responsable:** @QwikAuditor
- **Error cometido:** El routeLoader$ de la vista de auditoría reflejaba el valor crudo del parámetro de filtro de URL (`searchParams.get('action')`) directamente como argumento de query Drizzle sin validación de allowlist.
- **Causa raíz:** Los parámetros de URL son input no confiable. Reflejar un valor crudo en una cláusula `WHERE` (incluso si Drizzle usa parámetros preparados) expone la posibilidad de que valores inesperados pasen a la lógica de negocio y generen resultados incorrectos o inesperados.
- **Regla derivada:** Todo parámetro de filtro procedente de URL **debe** validarse contra una allowlist explícita antes de usarse en queries. Patrón: `const VALID_FILTER_ACTIONS = new Set(['action_a', 'action_b', ...])`. En el loader: `const action = VALID_FILTER_ACTIONS.has(raw) ? raw : undefined`. El valor crudo nunca llega a la capa de datos. Esta validación cierra el vector de input injection vía URL incluso cuando el ORM usa parámetros preparados.
- **Standard relacionado:** `QUALITY_STANDARDS.md` §Security (OWASP A03 Injection) · `PROJECT_RULES_CORE.md` §Input Validation
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-062
- **Fecha:** 2026-03-29
- **Feature:** notarylink-m16-comm-engine (M16)
- **Agente responsable:** @QwikBuilder
- **Error cometido:** El primer diseño del receptor `/api/resend/webhook` procesaba el payload JSON antes de verificar la firma SVIX, y respondía 200 incluso si la verificación fallaba posteriormente.
- **Causa raíz:** Verificar la firma SVIX después de parsear el body es semánticamente incorrecto: cualquier actor que conozca la URL puede inyectar eventos forjados (`email.bounced`, `email.complained`) para degradar la trazabilidad de comunicaciones antes de ser detectado.
- **Regla derivada:** La verificación de firma webhook **debe ser el primer paso** antes de cualquier parsing o lógica de negocio. Patrón canónico: (1) leer body como string crudo, (2) extraer headers `svix-id`/`svix-timestamp`/`svix-signature`, (3) `new Webhook(secret).verify(body, svixHeaders)` — sincrónico, lanza si inválido, (4) si lanza → `json(401, { error: 'Invalid signature' })` inmediato, (5) solo entonces parsear JSON y procesar. Mockear en tests: `svix-signature: 'valid'` → acepta; cualquier otro valor → lanza.
- **Standard relacionado:** `QUALITY_STANDARDS.md` §Security (OWASP A07 Auth Failures) · `ADR-012-webhook-secret-storage.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-063
- **Fecha:** 2026-03-29
- **Feature:** notarylink-m16-comm-engine (M16)
- **Agente responsable:** @QwikBuilder / @QwikAuditor
- **Error cometido:** El loader de `/sign/[token]` intentaba inferir el tipo de token (hex-64 session vs UUID-v4 legacy) mediante try/catch: primero llamaba a `validateSigningSession()` y si lanzaba asumía UUID legacy. Esto provocaba un roundtrip DB innecesario en el path más común.
- **Causa raíz:** Los dos formatos de token son mutuamente excluyentes por construcción geométrica: `SESSION_TOKEN_RE = /^[0-9a-f]{64}$/` (64 hex chars) vs UUID-v4 (36 chars con guiones). Usar try/catch como discriminador de tipo cuando existe un test O(1) es un error de diseño: añade latencia, oscurece la intención y mezcla control flow de errores con lógica de routing.
- **Regla derivada:** Cuando un loader recibe un parámetro de URL con múltiples formatos semánticos posibles, aplicar la distinción con un regex O(1) como **primer guard** — antes de cualquier llamada a DB. `if (SESSION_TOKEN_RE.test(token)) { /* path hex-64 */ } else { /* path UUID legacy */ }`. No usar try/catch como discriminador de formato. Los dos formatos deben ser estructuralmente distinguibles (longitud y charset distintos) — si no lo son, rediseñar el esquema de tokens.
- **Standard relacionado:** `DECISIONS_QWIK.md` §RouteLoader · `SERIALIZATION_CONTRACTS.md` §Token Formats
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-064
- **Fecha:** 2026-03-29
- **Feature:** M16.5 UX/UI Audit — Done page
- **Agente responsable:** @QwikAuditor
- **Error cometido:** El loader de `/sign/[token]/done` solo manejaba UUID-v4 (M03). Tras introducir session tokens hex-64 en M16, el guard `if (!isValidUUID(token)) throw redirect(302, '/')` enviaba al firmante a la homepage en lugar de a la pantalla de confirmación. Bug silencioso sin error visible en consola.
- **Causa raíz:** Cuando se añade un segundo formato de token a una ruta existente, **todos los loaders de rutas derivadas** (`/done`, `/retry`, etc.) deben actualizarse simultáneamente. El Done page quedó fuera del scope de M16.
- **Regla derivada:** Al introducir un nuevo formato de token/identificador en un flujo multi-step, crear un checklist explícito de todas las sub-rutas del mismo flujo (`/done`, sub-páginas, etc.) y verificar que cada loader cubre el nuevo formato antes de cerrar el PR. La discriminación hex-64 usa `validateSigningSession()` + `getSignerDataById()` — incluso para sesiones ya consumidas (`already_used`), `signerId` sigue disponible en el result.
- **Standard relacionado:** `DECISIONS_QWIK.md` §RouteLoader token formats
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-065
- **Fecha:** 2026-03-29
- **Feature:** M16.5 UX/UI Audit — Action error feedback
- **Agente responsable:** @QwikAuditor / @QwikBuilder
- **Error cometido:** El `useTask$` que observaba `resendAction.value` solo manejaba el caso `success` (`sent: true`). Los casos de error de `.fail()` nunca llegaban al usuario — el botón simplemente dejaba de estar en `isRunning` sin ningún mensaje.
- **Causa raíz:** Pattern incompleto: al implementar feedback de action, siempre deben cubrirse 3 estados: `isRunning` (spinner), `success` (mensaje verde), `error` (mensaje rojo con `role="alert"`). Omitir el tercer estado es un bug silencioso de UX.
- **Regla derivada:** Todo `useTask$` que trackea `action.value` debe incluir la rama `else if (v && 'error' in v)` para materializar el error en un Signal visible. La estructura mínima: `if (v?.sent) { /* success */ } else if (v?.error) { /* error visible en UI */ }`.
- **Standard relacionado:** `UX_GUIDE.md` §4 (Marco de Estados del Sistema) — Error state es obligatorio
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-066
- **Fecha:** 2026-03-29
- **Feature:** M16.5 UX/UI Audit — Dark mode consistency
- **Agente responsable:** @QwikAuditor
- **Error cometido:** `documents/[id]/index.tsx` tenía `STATUS_COLORS` sin variantes dark, contenedor `bg-white` sin dark mode, y fechas en formato ISO `YYYY-MM-DD` en lugar de `DD/MM/YYYY`.
- **Causa raíz:** La vista de detalle fue implementada en un sprint diferente al listado del dashboard. Al no tener una referencia compartida de `STATUS_COLORS`, cada archivo definió su propia versión. La inconsistencia de fechas surgió de usar `.slice(0, 10)` (formato ISO) vs la función `formatDate()` local del dashboard.
- **Regla derivada:** (1) `STATUS_COLORS` y `STATUS_LABELS` son shared constants candidatas a extracción a `~/lib/utils/document-status.ts` si existen en ≥2 archivos. (2) Nunca formatear fechas con `.slice()` — usar siempre una función `formatDate()` locale-aware. (3) En la auditoría M16.5, verificar dark mode en CADA contenedor de ruta nueva antes de LGTM.
- **Standard relacionado:** `DECISIONS_UI.md` §2 (Dark Mode) · `UX_GUIDE.md` §1 (Respeto por el Experto)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-067
- **Fecha:** 2026-03-29
- **Feature:** M16.5 UX/UI Audit — Shared action isRunning state
- **Agente responsable:** @QwikAuditor
- **Error cometido:** `consolidatedAction.isRunning` (boolean global) deshabilitaba simultáneamente TODOS los botones "Descargar PDF" de la tabla del dashboard cuando cualquier descarga estaba en progreso.
- **Causa raíz:** En Qwik, `routeAction$.isRunning` es un Signal global al componente — no está scoped por fila. En tablas con N filas que comparten la misma action, usar `isRunning` directamente como `disabled` y como selector del texto del botón produce un estado visual incorrecto.
- **Regla derivada:** En tablas con múltiples filas que usan la misma `routeAction$`, usar un Signal auxiliar (`pendingDocId`) para trackear qué fila inició la acción. El `onClick$` captura el ID como primitivo (serializable). El guard de disabled es `isRunning && pendingDocId.value === rowId`. Resetear `pendingDocId` en un `useVisibleTask$` separado que observe `isRunning → false`.
- **Standard relacionado:** `DECISIONS_QWIK.md` §Signals · `SERIALIZATION_CONTRACTS.md` §Closures mínimos
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-068
- **Fecha:** 2026-03-29
- **Feature:** M16.5 — API Keys & Webhooks (Fix de persistencia)
- **Agente responsable:** @QwikAuditor / @QwikBuilder
- **Error cometido:** `<Form onSubmitCompleted$={...}>` cerraba el modal de creación de API Keys y Webhooks después de **cualquier** round-trip server (exitoso o fallido). Si Zod rechazaba el form (0 scopes seleccionados, URL inválida), el modal cerraba sin feedback y sin escritura en DB. El usuario percibía "éxito silencioso" pero la lista permanecía vacía.
- **Causa raíz (parte 1):** `onSubmitCompleted$` en Qwik City se dispara tras **todo** completed round-trip, no solo en casos de éxito. Nunca usarlo para cerrar un modal o ejecutar lógica de éxito. La lógica de post-acción pertenece exclusivamente al `useTask$` que trackea `action.value`.
- **Causa raíz (parte 2):** El `useTask$` detectaba `'message' in result` (errores de `fail()`) y `'id' in result` (éxito), pero no tenía rama para errores de validación Zod, que devuelven `{ failed: true, fieldErrors: {...} }` — sin `message`, sin `id`. La validación fallaba en silencio total: sin toast, sin modal abierto.
- **Regla derivada:** Patrón canónico para `routeAction$` + modal en Qwik City: (1) Quitar `onSubmitCompleted$` del `<Form>`. (2) En el `useTask$`: `'id' in result && !('failed' in result)` → éxito → toast + cerrar modal. `'message' in result` → error servidor → toast. Errores Zod (`fieldErrors`) → dejar modal abierto + error inline. (3) El inline form error maneja `message` (fail explícito) y `fieldErrors` (Zod) via `(value as unknown) as { fieldErrors?: Record<string, string | string[]> }` + `.flatMap(v => Array.isArray(v) ? v : [v])`.
- **Bono técnico:** Los errores de TypeScript al castear `fieldErrors` revelaron el shape exacto de Zod failures en Qwik City: los valores son `string | string[]` según si el campo tiene error único o múltiple. Siempre usar `as unknown` intermediario para evitar "conversion may be a mistake".
- **Standard relacionado:** `DECISIONS_QWIK.md` §routeAction$ · `UX_GUIDE.md` §4 Marco de Estados
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-069
- **Fecha:** 02-04-2026
- **Feature:** dashboard-home (M03-02) — `head` export con `resolveValue`
- **Agente responsable:** @QwikBuilder / @QwikMemory (clarificación)
- **Error cometido (confusión):** Ambigüedad en torno a LL-047: si estaba prohibido **importar** `useOrgContext` desde `~/routes/(app)/layout` en una ruta hija para usarlo con `resolveValue()` en la función `head`.
- **Causa raíz:** LL-047 prohíbe el **re-export** (`export { useOrgContext } from '~/routes/(app)/layout'`), que duplica la entrada del loader en el manifiesto de Vite y genera colisión de IDs en runtime. El **import** del loader para invocarlo localmente (`useOrgContext()` dentro del componente o `resolveValue(useOrgContext)` dentro de `head`) es el patrón correcto documentado en la propia LL-047. Son operaciones con consecuencias completamente distintas.
- **Regla derivada:** Importar `useOrgContext` (o cualquier `routeLoader$` del layout padre) en una ruta hija para consumo local — ya sea en el `component$` o en la función `head` con `resolveValue()` — es **válido y correcto**. La restricción de LL-047 aplica única y exclusivamente al re-export explícito (`export { useLoader } from '../layout'`). Regla mnemónica: **import = consumir (✅) · re-export = duplicar en manifiesto (❌)**.
- **Standard relacionado:** `DECISIONS_QWIK.md` §routeLoader$ · LL-047
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-070
- **Fecha:** 03-04-2026
- **Feature:** crm-contacts (M04) — race condition layout/ruta
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Redirect a `/dashboard` inmediato al navegar a `/contacts`. `requireRole()` en el loader de la ruta hija leía `sharedMap.get('profile')` y obtenía `undefined`, provocando el redirect por "sesión expirada".
- **Causa raíz:** Los `routeLoader$` del layout padre y de la ruta hija se ejecutan en **paralelo** (`Promise.all` — confirmado en código fuente de Qwik City). El `sharedMap.set('profile')` del layout padre ocurría después de su `await db.select(...)`. Para cuando el loader hija ejecutaba `requireRole()` síncronamente, el valor aún no existía.
- **Fix aplicado:** Mover la carga de perfil + org del `routeLoader$` del layout al `onRequest` middleware de `(app)/layout.tsx`. El middleware se ejecuta **secuencialmente** y **antes** de todos los loaders.
- **Regla derivada:** Cualquier dato que los loaders de ruta necesiten leer de `sharedMap` DEBE escribirse en `onRequest`. Nunca confiar en que un `routeLoader$` de layout padre complete antes de que un loader hija lea `sharedMap`. Son concurrentes por diseño del framework.
- **Standard relacionado:** `DECISIONS_DATA.md` §sharedMap · `DECISIONS_QWIK.md` §routeLoader$
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-074
- **Fecha:** 03-04-2026
- **Feature:** crm-contacts (M04) — modal de creación de contacto congelado
- **Agente responsable:** @QwikBuilder
- **Error cometido:** El modal nativo `<dialog>` se abría (`.showModal()` ejecutado) pero parecía congelado: sin estilos activos, sin foco, sin respuesta a inputs. El `useVisibleTask$` que inicializaba el formulario nunca disparó.
- **Causa raíz:** `useVisibleTask$` usa por defecto la estrategia `intersection-observer`. Un elemento `<dialog>` cerrado tiene `display:none` — sin caja de layout — y el IntersectionObserver nunca detecta el elemento como visible. La tarea nunca se ejecuta aunque `.showModal()` haya cambiado el estado visual del diálogo.
- **Regla derivada:** `useVisibleTask$` con la estrategia `intersection-observer` (por defecto) falla en elementos que empiezan con `display:none` como `<dialog>`. **Obligatorio usar `{ strategy: 'document-ready' }` para cualquier lógica de inicialización en modales nativos** (focus, reset form, carga lazy de datos). Alternativa: migrar la lógica de inicialización al handler que dispara `.showModal()`.
- **Standard relacionado:** `DECISIONS_QWIK.md` §useVisibleTask$ · LL-008 (gestión de foco en dialogs)
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-075
- **Fecha:** 03-04-2026
- **Feature:** crm-contacts (M04) — error 42501 RLS en INSERT/UPDATE/DELETE
- **Agente responsable:** @QwikBuilder / @QwikAuditor
- **Error cometido:** `routeAction$` de creación de contacto retornaba error 42501 (RLS violation) aunque el usuario estaba autenticado, las políticas RLS estaban correctas y el orgId era válido. `getUser()` devolvía el usuario sin error.
- **Causa raíz:** `@supabase/ssr` v0.10 introduce `skipAutoInitialize: true`. El cliente se crea sin cargar la sesión desde las cookies en memoria. `getUser()` funciona porque llama directamente a la Auth API REST. Pero las queries a PostgREST se envían **sin el header `Authorization: Bearer <jwt>`** — el cliente cae al rol `anon` y RLS rechaza todas las escrituras. Añadir `await supabase.auth.getSession()` antes de las queries no resuelve el problema de forma consistente en Qwik City SSR.
- **Fix aplicado (temporal canónico):** Usar `createSupabaseAdminClient()` (service role) en todos los loaders y actions de rutas protegidas por `requireRole()`. Seguridad de tenant garantizada por: (1) `requireRole()` valida la sesión antes de cualquier loader/action, (2) `.eq('organization_id', orgId)` explícito en **cada query**.
- **Regla derivada:** En rutas SSR protegidas por `requireRole()` bajo `(app)/`: usar `createSupabaseAdminClient()` con filtro `orgId` explícito en cada query. Reservar `createSupabaseServerClient()` para operaciones que requieren el contexto de usuario (auth callbacks, signed URLs, Realtime). Revisar al actualizar `@supabase/ssr` si el comportamiento de `skipAutoInitialize` cambia.
- **Standard relacionado:** `DECISIONS_DATA.md` §Supabase Client · `SECURITY_POLICIES.md` §Tenant Isolation
- **Reincidencias:** 0 (afecta potencialmente a todas las rutas de escritura bajo `(app)/`)
- **Estado:** ✅ Propagada

---

### LL-076
- **Fecha:** 03-04-2026
- **Feature:** crm-contacts (M04) — diagnóstico RLS bloqueado por migración fantasma
- **Agente responsable:** @QwikDBA / @QwikBuilder
- **Error cometido:** La política `contacts_select_own_org` (SELECT) no existía en producción, causando que las queries de listado devolvieran 0 filas. La migración `0005_fix_contacts_rls.sql` aparecía marcada como aplicada en `__drizzle_migrations` pero el SQL nunca se había ejecutado realmente en Supabase.
- **Causa raíz:** Drizzle marca una migración como aplicada en su journal interno sin verificar si el SQL tuvo efecto real. En Supabase, el historial de migraciones de Drizzle (`__drizzle_migrations`) y el catálogo de políticas de PostgreSQL (`pg_policies`) son fuentes de verdad independientes. Una discrepancia de timing entre ambos puede dejar el journal "ahead" del estado real de la DB.
- **Regla derivada:** **No confiar en `__drizzle_migrations` para confirmar que las políticas RLS están activas.** Tras cada migración crítica que contenga `CREATE POLICY` / `DROP POLICY`, verificar con consulta directa al catálogo:
  ```sql
  SELECT policyname, cmd FROM pg_policies WHERE tablename = 'nombre_tabla';
  ```
  La verificación debe hacerse sobre la `DIRECT_URL` (conexión directa), no a través del Transaction Pooler, para asegurar que refleja el estado real del schema. Añadir esta verificación al checklist de cierre de cada PR con cambios de RLS.
- **Standard relacionado:** `DECISIONS_DATA.md` §Migrations · `DECISIONS_DATA.md` §RLS Verification
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-080
- **Fecha:** 21-04-2026
- **Feature:** S001 UX Notifications — migración de `contact-form-sheet.tsx` a NotificationProvider global
- **Agente responsable:** @QwikBuilder
- **Error cometido:** El formulario mezclaba feedback local inline con un `ToastStore` local para errores de `routeAction$`, generando duplicidad de señales visuales y riesgo de solapamiento con la capa global.
- **Causa raíz:** Acoplamiento histórico entre validación por pasos del formulario y transporte de errores del servidor en el mismo componente. El canal de notificaciones local no distinguía errores de validación (`422`) frente a errores operativos.
- **Regla derivada:** En formularios migrados al sistema global, el feedback debe dividirse en dos canales estrictos: (1) validación local/422 permanece inline en el formulario, (2) errores operativos y éxitos se enrutan a `notifyFromActionResult$`. Si el payload contiene `status=422` o `fieldErrors/errors`, el canal global debe permanecer en silencio.
- **Comportamiento observado (escenario real):** Tras reemplazar `addToast` por `notifyFromActionResult$(activeAction.value)`, los errores de paso (`stepErrors`) siguen visibles en los campos sin emitir toast global, mientras que un `success` de edición dispara toast y cierra el sheet.
- **Standard relacionado:** `DECISIONS_UI.md` §2 · `SPEC-001` AC-15/AC-16 · `SERIALIZATION_CONTRACTS.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-081
- **Fecha:** 2026-03-XX
- **Feature:** ui-tabs-wrapper
- **Agente responsable:** @QwikBuilder
- **Error cometido:** Crear wrappers custom para `@qwik-ui/headless Tabs` propagando props de forma incorrecta: `TabsPanel` sin `{...rest}` y/o `TabsList` propagando `{...rest}` sobre `<Tabs.List>`.
- **Causa raíz:** El sistema de `HTabs` inyecta props internas en wrappers registrados (`_tabId` en panels y manipulación de `children` en list). Si el wrapper no respeta exactamente esa interfaz implícita, Qwik rompe la proyección esperada: panels ocultos o triggers invisibles.
- **Regla derivada:** En wrappers de `@qwik-ui/headless Tabs`, `TabsPanel` DEBE propagar `{...rest}` para recibir `_tabId`; `TabsList` NO debe propagar `{...rest}` porque entra en conflicto con `<Slot />`; `TabsTrigger` sí puede propagar `{...rest}`. Además, `TabsRoot` debe usar `selectedIndex={0}` para SSR estable y definirse como función inline cuando el sistema necesite introspección de hijos.
- **Standard relacionado:** `DECISIONS_UI.md` · `DECISIONS_QWIK.md`
- **Referencia origen:** antiguo `LESSONS-TABS.md`
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-082
- **Fecha:** 2026-04-25
- **Feature:** F06 — case-contacts (migración 0010)
- **Agente responsable:** @QwikDBA / @QwikBuilder
- **Error cometido:** `bun db:migrate` reportó ✅ y registró la migración en `__drizzle_migrations`, pero el DDL no se ejecutó realmente contra Supabase. Resultado: error `PGRST205` en runtime al intentar acceder a `case_contacts`.
- **Causa raíz:** El cliente Drizzle puede considerar una migración como "aplicada" (insertando el registro en la tabla de control) sin haber ejecutado con éxito el DDL subyacente contra Supabase, especialmente en entornos con conexión pooled o timeouts silenciosos.
- **Regla derivada:** Tras cualquier migración que incluya nuevas tablas, verificar su existencia en **Supabase Table Editor** antes de probar en desarrollo. Si la tabla no aparece, aplicar el SQL manualmente desde **Supabase SQL Editor**. El registro en `__drizzle_migrations` no es garantía suficiente de ejecución DDL.
- **Standard relacionado:** `DECISIONS-DATA.md` §migraciones
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-083
- **Fecha:** 2026-05-01
- **Feature:** schema-identity / migración 0016 (profiles: position, tax_id, is_active, deactivated_at)
- **Agente responsable:** @QwikDBA
- **Error cometido:** Se ejecutó `bun drizzle-kit push` para aplicar los nuevos campos de `profiles`. El comando aplicó los 4 campos correctamente pero **eliminó 16 políticas RLS** aplicadas via SQL manual en migraciones 0013/0014/0015 (tablas: `profiles`, `hub_teams`, `hub_team_members`, `case_contacts`, `case_items`, `notifications`).
- **Causa raíz:** `drizzle-kit push` compara el schema Drizzle TypeScript con el estado real de la DB y **destruye todo lo que no reconoce**, incluyendo políticas RLS definidas via SQL puro. Drizzle no rastrea RLS en su modelo de schema. Al detectar divergencia entre el schema TS y el estado en DB (policies creadas manualmente), las eliminó para "sincronizar".
- **Efecto real:** La seguridad multi-tenant via Supabase SDK quedó desprotegida hasta la aplicación del script de recuperación `0016b_rls_recovery_after_push.sql`. El servidor Drizzle/adminClient (BYPASSRLS) no se vio afectado — sin impacto en producción server-side.
- **Regla derivada:** `bun drizzle-kit push` está **PROHIBIDO** en este proyecto. El flujo canónico es: (1) editar `schema-identity.ts` / `schema-domain.ts`, (2) generar la migración con `bun run db:generate`, (3) revisar el SQL generado en `drizzle/`, (4) añadir manualmente las sentencias RLS/GRANTs necesarias en el mismo archivo SQL, (5) aplicar el SQL en **Supabase Dashboard → SQL Editor**. Nunca usar `push`.
- **Standard relacionado:** `PROJECT-RULES-CORE.md` §4 · `DECISIONS-DATA.md` §migraciones
- **Reincidencias:** 0
- **Estado:** ✅ Propagada

---

### LL-084
- **Fecha:** 2026-05-07
- **Feature:** M11 — case-item-mentions
- **Agente responsable:** @QwikBuilder / @QwikAuditor
- **Error cometido:** En `useCreateCaseItemAction`, la llamada a `syncMentions()` no capturaba ni propagaba su resultado. Si el servicio fallaba (error de elegibilidad server-side, error de DB), la action retornaba `{ ok: true }` sin advertir al usuario. El ítem quedaba creado sin menciones y sin feedback.
- **Causa raíz:** Patrón de "llamada fire-and-forget implícita" aplicado a un servicio que sí produce un resultado relevante para el usuario. La diferencia con el patrón fire-and-forget explícito de `_insertNotifications` (que usa `Promise.allSettled` conscientemente) es que en ese caso la decisión está documentada y es intencional (fallo de notificación no debe bloquear ni el sync ni el ítem).
- **Regla derivada:** En `routeAction$`, toda llamada a un servicio que retorne `ServiceResult<T>` DEBE capturarse y propagarse al cliente. Si el fallo es no bloqueante (el flujo principal tuvo éxito), retornar un campo de advertencia como `mentionsSyncWarning` en el resultado de la action. Nunca ignorar silenciosamente un `ServiceResult` que indica error. El patrón correcto es: `const result = await service(...); if (!result.ok) return { ok: true, mainId, warningField: result.error };`.
- **Standard relacionado:** `DECISIONS-QWIK.md` §routeAction$ · `QUALITY-STANDARDS.md` §feedback-usuario
- **Reincidencias:** 0
- **Estado:** ✅ Propagada