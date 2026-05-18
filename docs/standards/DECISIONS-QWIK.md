# DECISIONES QWIK — SDD Qwik

> **Propósito:** Decisiones técnicas del proyecto sobre cómo usamos Qwik.
> NO documenta cómo funciona Qwik internamente — para eso existe Context7.
> Documenta QUÉ decidimos usar, cómo y por qué en este proyecto.
>
> Context7 ID: `/qwikdev/qwik` — usar para sintaxis, APIs, ejemplos detallados.

---

## 1. Selección de Primitivas (Decisión de Proyecto)

La elección incorrecta de primitiva degrada la resumabilidad. Esta tabla es la ley:

| Necesidad | Primitiva correcta | Primitiva prohibida |
|---|---|---|
| Valor reactivo único | `useSignal` | `useStore` para un valor |
| Objeto con múltiples props | `useStore` | `useSignal` por cada prop |
| Valor derivado síncrono | `useComputed$` | `useTask$` para cálculos puros |
| Fetch reactivo al input del usuario | `useResource$` | `useVisibleTask$` para fetches |
| Side-effects reactivos (server+client) | `useTask$` | `useEffect` (React, prohibido) |
| DOM / browser APIs / librerías 3rd-party | `useVisibleTask$` | Solo para estos 3 casos |

**Regla de oro:** `useVisibleTask$` requiere justificación explícita en comentario.
Cualquier uso fuera de DOM/browser APIs/librerías externas es violación de performance.

---

## 2. Serialización de Closures `$()` (Regla Crítica)

Los closures `$()` solo capturan **primitivos o IDs**. Nunca objetos pesados.

```typescript
// ✅ CORRECTO — captura solo el ID (primitivo)
const handleDelete = $(() => {
  const item = itemStore.items.find(i => i.id === itemId); // lee del store dentro
  deleteItem(item);
});

// ❌ INCORRECTO — captura el objeto completo
const handleDelete = $(() => {
  deleteItem(fullItemObject); // fullItemObject capturado en closure
});
```

**¿Por qué?** Los closures se serializan en el HTML. Un objeto grande = snapshot HTML pesado = peor TTI.

---

## 3. `sync$()` para Interacciones Puras de DOM

Para toggle de modales, clases CSS, scroll — operaciones que NO necesitan servidor:

```typescript
// ✅ CORRECTO — sync$ no genera petición HTTP
const toggleModal = sync$((e: MouseEvent) => {
  (e.target as HTMLElement).closest('[data-modal]')?.classList.toggle('open');
});

// ❌ INCORRECTO — onClick$ genera petición HTTP para toggle de clase
const toggleModal = $(() => {
  isOpen.value = !isOpen.value; // si solo afecta al DOM, usar sync$
});
```

**Regla:** Si la operación es pura DOM (sin state de servidor, sin señales reactivas complejas) → `sync$`.

---

## 4. `noSerialize()` — Aplicación Agresiva

Para librerías de terceros que necesitan el DOM (Charts, Mapas, Editors):

```typescript
// ✅ CORRECTO — noSerialize evita que Qwik intente serializar el objeto
useVisibleTask$(({ cleanup }) => {
  const chart = new Chart(canvasRef.value, config);
  store.chart = noSerialize(chart); // Chart no es serializable
  cleanup(() => chart.destroy());
});
```

**Regla:** Todo objeto de librería de terceros que vive en un store DEBE usar `noSerialize()`.
Si no lo tiene, la app rompe al intentar reanudarse.

---

## 5. Co-localización de QRLs (Performance)

Los handlers que se usan juntos en el mismo componente van en el mismo archivo.
Separarlos crea waterfalls HTTP — múltiples peticiones en cascada al primer click.

```typescript
// ✅ CORRECTO — onClick$ y onInput$ del mismo form en el mismo archivo
export const SearchForm = component$(() => {
  const handleInput = $((e: InputEvent) => { /* ... */ });
  const handleSubmit = $(() => { /* ... */ });
  // Qwik los agrupa en el mismo chunk automáticamente
});

// ❌ INCORRECTO — importar handlers desde archivos separados para el mismo componente
import { handleInput } from './handlers/input.ts';   // chunk A
import { handleSubmit } from './handlers/submit.ts'; // chunk B
// Dos peticiones HTTP en lugar de una
```

---

## 6. PropFunction para Callbacks entre Componentes

El tipo `PropFunction<T>` es obligatorio para pasar callbacks a componentes hijos.
Las funciones crudas no son serializables.

```typescript
// ✅ CORRECTO
interface CardProps {
  onDelete$: PropFunction<(id: string) => void>;
}

// ❌ INCORRECTO
interface CardProps {
  onDelete: (id: string) => void; // función cruda, no serializable
}
```

---

## 7. Context API para Dependency Injection

Usamos Context en lugar de prop drilling para estado compartido.
El OrganizationContext es el contexto principal del dashboard:

```typescript
// Patrón canónico del proyecto
export const OrganizationContext = createContextId<OrganizationState>('app.organization');

// En layout.tsx (proveedor)
useContextProvider(OrganizationContext, orgState);

// En componentes hijos (consumidor)
const org = useContext(OrganizationContext);
```

**Regla:** El contexto se provee en el layout más cercano que englobe todos los consumidores.
No crear contextos globales innecesarios — aumentan el snapshot size.

---

## 8. `usePreventNavigate$()` — Solo para Formularios con Datos Sin Guardar

Único caso de uso autorizado: evitar pérdida de datos en formularios.

```typescript
usePreventNavigate$(() => {
  if (hasUnsavedChanges.value) {
    return !confirm('¿Salir sin guardar?');
  }
  return true;
});
```

---

## 9. Blacklist Nuclear (Prohibición Absoluta)

NINGÚN agente puede usar estas APIs bajo ninguna circunstancia:

```
useState, useEffect, useContext, useMemo, useCallback, useTransition,
useDeferredValue, useRef, useImperativeHandle, useLayoutEffect, useReducer,
useId, use, useActionState, useOptimistic, useFormStatus, createContext,
forwardRef, memo, lazy, Suspense, createPortal, startTransition,
useRouter, usePathname, useSearchParams, useParams,
getServerSideProps, getStaticProps, getStaticPaths,
generateMetadata, revalidatePath, revalidateTag, notFound
```

Si el modelo produce cualquiera de estos → rechazar y reescribir.

---

## 10. Reglas de Arquitectura Interna

- `src/routes/`: SOLO `routeLoader$`, `routeAction$` y ensamblaje. Prohibida lógica de negocio.
- `src/components/`: UI pura. Sin imports de Supabase, Drizzle ni servicios.
- `src/lib/`: Todo el cerebro. Servicios, DB, auth, utils, schemas.
- `src/features/[feature]/`: Solo para features con más de 5 archivos. Exponer facade en `src/lib/`.

> Para detalles de sintaxis, ejemplos avanzados y APIs: usar Context7 con `/qwikdev/qwik`

---

## 11. Bundle Safety — Reglas de Importación (Impacto Directo en Resumabilidad)

Estas reglas protegen el modelo de ejecución O(1) de Qwik. Una importación mal
hecha no es un warning de linter — es código que se descarga cuando no debería.

### 11.1 Prohibición de Barrel Exports en `src/features/`

```typescript
// ❌ PROHIBIDO — src/features/billing/index.ts que re-exporta todo
export * from './components/BillingForm';
export * from './components/InvoiceList';
export * from './services/billing.service';
export * from './hooks/useBilling';
// Fuerza la descarga de TODOS los módulos aunque solo se use uno

// ✅ CORRECTO — importar solo lo necesario, directamente
import { BillingForm } from '~/features/billing/components/BillingForm';
import { BillingService } from '~/lib/billing'; // solo el facade público
```

**¿Por qué?** Los barrel exports anulan el tree shaking de QRLs. Qwik divide el
código en chunks por QRL — si un barrel importa 10 módulos, los 10 se incluyen
en el mismo chunk aunque solo se use 1.

**Excepción:** `src/lib/[feature]/index.ts` — el facade público sí puede re-exportar,
pero solo las APIs que `routes/` necesita, no la implementación interna.

### 11.2 Importaciones Selectivas de Librerías Externas

```typescript
// ❌ PROHIBIDO — importar la librería completa para usar una función
import _ from 'lodash';           // +70kB por usar _.debounce
import * as R from 'ramda';       // +40kB por usar R.pipe
import dayjs from 'dayjs';        // aceptable si se usa extensamente

// ✅ CORRECTO — importar solo lo necesario
import { debounce } from 'lodash-es';     // tree-shakeable
import { formatDate } from '~/lib/utils/date'; // helper propio, 0kB extra
```

**Umbral de decisión:** Si una librería externa aporta más de 10kB al bundle
y se usa en un único punto → crear un helper en `src/lib/utils/` en su lugar.

### 11.3 Aislamiento Servidor/Cliente

```typescript
// ❌ PROHIBIDO — importar módulo de servidor en componente cliente
import { db } from '~/lib/db/client';          // Drizzle en el cliente
import { createSupabaseServerClient } from '~/lib/supabase/server'; // en componente

// ✅ CORRECTO — la DB solo existe en routeLoader$, routeAction$ y server$
export const useData = routeLoader$(async () => {
  return await DataService.getAll(); // el servicio usa db internamente
});
```

**@QwikAuditor verifica estos tres puntos en BLOQUE J.**