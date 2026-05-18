# DECISIONS_UI — Estándar de Interfaz

> **Estado:** 🟢 Approved | **Versión:** 3.0 | **Fecha:** 2026-04-21
> **Propósito:** Especificación autoritaria de las leyes de construcción visual para el stack SDD Qwik.
> Todo el código UI del proyecto debe ser conforme con este documento. Las desviaciones son rechazadas en auditoría.

---

## 1. Sistema de Diseño — Tailwind v4 CSS-First

### 1.1 Fuente de Verdad Única

Todos los tokens de diseño (colores, tipografías, espaciados, radios) se declaran
exclusivamente dentro de la directiva `@theme` en el archivo CSS global del proyecto
(`src/assets/css/global.css`).

**Prohibición absoluta:** No existe ni existirá `tailwind.config.js` ni `tailwind.config.ts`
en este proyecto. Cualquier PR que los introduzca debe ser rechazado.

### 1.2 Contrato de Variables HSL

Los colores semánticos (background, foreground, primary, etc.) se definen mediante
**variables CSS de canal HSL**, no como valores de color completos. Este patrón es
el único que permite cambios de tema dinámicos sin regenerar el CSS ni producir
parpadeos.

Contrato de nomenclatura obligatorio:

```
--[token-semantico]: [H] [S%] [L%];        ← variable de canal HSL en :root / .dark
--color-[token]: hsl(var(--[token]));      ← token de Tailwind que consuma la variable
```

Ejemplo de estructura correcta en `@layer base`:

```
:root  → valores light (canales HSL)
.dark  → override de canales HSL (mismas propiedades, valores distintos)
```

### 1.3 Regla de Clases Estáticas

El scanner de Tailwind opera en build-time sobre texto literal. Las clases construidas
por interpolación o concatenación dinámica no aparecen en el bundle final.

- **Obligatorio:** Strings literales completos en todos los condicionales de clase.
- **Prohibido:** Interpolación de fragmentos de clase: `` `bg-${color}-500` ``, `` `text-${size}` ``.
- **Correcto:** `condition ? 'bg-primary text-white' : 'bg-muted text-foreground'`

### 1.4 Uso de `useStylesScoped$`

Reservado exclusivamente para estos casos. Todo lo demás se resuelve con Tailwind:

1. Selectores que Tailwind no puede expresar: `::before`, `::after`, `nth-child`,
   `@keyframes` con curvas complejas.
2. Widgets embebibles que requieren aislamiento CSS total.
3. Uso explícitamente autorizado con comentario de justificación en el código.

---

## 2. Dark Mode — Política Zero-Flicker

### 2.1 Prohibición de `useVisibleTask$` para Temas

Usar `useVisibleTask$` para inicializar el tema es una vulneración arquitectónica.
Provoca un ciclo de hidratación innecesario y un parpadeo visible (flash of unstyled
content) en el primer render.

**Esta práctica queda terminantemente prohibida.**

### 2.2 Implementación Canónica: Script Síncrono en `<head>`

El tema se aplica mediante un script inline **síncrono** insertado en el `<head>` de
`root.tsx`, antes de cualquier renderizado del árbol. El navegador lo ejecuta de forma
bloqueante antes de pintar el primer frame.

Responsabilidades del script:

1. Leer la preferencia almacenada (`localStorage` o cookie).
2. Si no hay preferencia, consultar `window.matchMedia('(prefers-color-scheme: dark)')`.
3. Aplicar la clase `.dark` al elemento `<html>` de forma síncrona.

El script debe ser un string literal mínimo, sin dependencias de módulos externos.
No debe superar 300 bytes minificados.

### 2.3 Storage Key

Clave canónica: `'theme'` — Valores válidos: `'light'` | `'dark'` | `'system'`.

### 2.4 Separación de Responsabilidades

Los componentes UI (Button, Card, Input, etc.) **no contienen lógica de tema**.
Los cambios visuales entre light y dark los gestionan automáticamente las variables
CSS definidas en `@layer base` al alternar la clase `.dark` en `<html>`.

---

## 3. Estándar Dual de Iconografía — SVG Contract

Existen exactamente dos patrones válidos para iconos. La elección entre ellos depende
del contexto de uso. **No se inventarán patrones alternativos.**

### 3.1 Patrón A — IconMap (para menús dinámicos y sidebars)

**Cuándo usar:** Cuando los iconos se seleccionan en runtime a partir de un identificador
de string (menú de navegación, sidebar configurable, lista de acciones dinámica).

**Estructura:** Un archivo `icon-map.ts` exporta una constante tipada
`Record<string, JSXOutput>` con instancias SVG estáticas inlineadas. El componente
consumidor indexa el mapa con la clave string y renderiza el valor directamente.

**Ventaja:** Evita la creación de QRLs por cada icono. Al ser una constante del módulo,
no cruza ninguna frontera de serialización.

**Regla:** Este mapa es de sólo lectura en runtime. No se modifica dinámicamente.

### 3.2 Patrón B — Función Pura (para iconos individuales y de composición)

**Cuándo usar:** Cuando el icono se usa directamente en un componente concreto y no
necesita selección dinámica por string.

**Estructura:** Una función TypeScript (no un componente Qwik con `component$`) que
acepta `PropsOf<'svg'>` y retorna el JSX del SVG.

Contrato de firma obligatorio:

```typescript
import type { PropsOf } from '@builder.io/qwik';
export function IconName(props: PropsOf<'svg'>): JSXOutput
```

### 3.3 Reglas de Oro (aplican a ambos patrones)

| Regla | Especificación |
|---|---|
| Color | `stroke="currentColor"` o `fill="currentColor"`. Nunca colores hardcodeados. |
| Dimensiones | Sin `width` ni `height` en el SVG. Se controlan externamente con clases TW (`h-4 w-4`). |
| Accesibilidad | `aria-hidden="true"` si el icono es decorativo. Si tiene significado, el `aria-label` va en el elemento padre. |
| `viewBox` | Obligatorio. Valor estándar: `"0 0 24 24"`. |
| SVG inline | Prohibido en componentes reutilizables. Solo permitido en prototipos desechables. |
| Prop `key` | Nunca se añade `key` como parámetro explícito de la función. |

### 3.4 Ubicación

```
src/components/icons/       ← Patrón B: una función por archivo
src/lib/icon-map.ts         ← Patrón A: mapa centralizado
```

---

## 4. Animaciones y Micro-interacciones

### 4.1 Jerarquía de Implementación

| Prioridad | Herramienta | Caso de uso |
|---|---|---|
| 1 (preferida) | Tailwind `transition-*` / `animate-*` | Transiciones de estado: hover, focus, active, dark mode |
| 2 (cuando JS es obligatorio) | Motion One `animate()` | Entradas secuenciadas, gestos, scroll-triggered |
| — (prohibida) | Mezcla CSS+JS en la misma propiedad | Conflicto garantizado de transformaciones |

**Regla de separación:** Si un elemento tiene `animate()` de Motion One actuando sobre
`opacity` o `transform`, esas mismas propiedades no deben tener `transition-*` de
Tailwind. Usar `will-change-[transform,opacity]` como optimización de capa.

---

## 5. Accesibilidad — Contratos No Negociables

| Contrato | Especificación |
|---|---|
| Elementos decorativos | `aria-hidden="true"` obligatorio (iconos, separadores, imágenes decorativas) |
| Elementos interactivos no nativos | `role="button"` + manejador de teclado (`onKeyDown$` con `Enter`/`Space`) |
| Imágenes con significado | `alt` descriptivo obligatorio |
| Contraste de color | Mínimo WCAG AA (4.5:1 para texto normal, 3:1 para texto grande) |
| Focus visible | Nunca `outline: none` sin un reemplazo de focus visible |

---

## 6. Registro de Decisiones

| Decisión | Motivo | Fecha |
|---|---|---|
| Prohibir `tailwind.config.*` | CSS-First es el paradigma de Tailwind v4; la config JS es legacy | 2026-04-21 |
| Prohibir `useVisibleTask$` para temas | Provoca flicker + hidratación innecesaria | 2026-04-21 |
| Patrón dual de iconos (A+B) | Evita QRL overhead en menús; mantiene composabilidad en iconos individuales | 2026-04-21 |
| Variables HSL de canal | Único patrón que permite theming dinámico sin regenerar CSS | 2026-04-21 |
