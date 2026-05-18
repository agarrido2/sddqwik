# PRD — Product Requirements Document

> **Proyecto:** [Nombre del proyecto]
> **Cliente:** [Nombre del cliente / empresa]
> **Fecha:** [fecha]
> **Versión:** 1.0
> **Estado:** 🟡 Draft | 🟠 Review | 🟢 Aprobado

---

## PARTE 0: DISCOVERY CHECKLIST
> Usa esta lista en la reunión con el cliente ANTES de escribir el PRD.
> No empieces a documentar sin tener respuesta a las preguntas marcadas como ⚠️ críticas.
> Las marcadas con 💡 son opcionales pero enriquecen mucho el resultado.

### Negocio y contexto
- [ ] ⚠️ ¿Cuál es el problema concreto que quieres resolver con esta aplicación?
- [ ] ⚠️ ¿Tienes ya una solución (aunque sea manual o parcial) para ese problema? ¿Cómo funciona hoy?
- [ ] ⚠️ ¿Cuál es tu modelo de negocio? ¿Cómo genera ingresos la aplicación?
- [ ] ⚠️ ¿Quién es tu competencia directa? ¿Qué hace mejor o peor que tú?
- [ ] 💡 ¿Tienes alguna aplicación de referencia que te guste, aunque sea de otro sector?
- [ ] 💡 ¿Cuál es el plazo que tienes en mente y cuál es el presupuesto orientativo?

### Usuarios
- [ ] ⚠️ ¿Quién va a usar la aplicación? (tipos de usuario, roles, perfiles)
- [ ] ⚠️ ¿Cuántos usuarios esperas tener al lanzar? ¿Y en 12 meses?
- [ ] ⚠️ ¿Hay usuarios internos (empleados, gestores) y externos (clientes, proveedores)?
- [ ] 💡 ¿Tus usuarios son técnicos o el perfil es no técnico?
- [ ] 💡 ¿Hay usuarios con necesidades especiales (accesibilidad, idioma)?

### Funcionalidades
- [ ] ⚠️ ¿Cuáles son las 3 cosas más importantes que la aplicación debe hacer? (Core features)
- [ ] ⚠️ ¿Qué es lo que si no está, la aplicación no sirve para nada? (Must-have)
- [ ] ⚠️ ¿Qué podría esperar a una segunda fase? (Nice-to-have)
- [ ] ⚠️ ¿Hay algo que explícitamente NO quieres que haga la aplicación?
- [ ] 💡 ¿Necesita integrarse con otras herramientas que ya usas? (CRM, pagos, email, etc.)
- [ ] 💡 ¿Necesita funcionar en móvil? ¿Hay app nativa o solo web responsive?

### Datos y contenido
- [ ] ⚠️ ¿Qué tipo de datos maneja la aplicación? (productos, usuarios, pedidos, etc.)
- [ ] ⚠️ ¿Hay datos sensibles? (datos personales, pagos, datos médicos, etc.)
- [ ] 💡 ¿Tienes datos existentes que haya que migrar o importar?
- [ ] 💡 ¿Quién crea y mantiene el contenido de la aplicación?

### Operación y seguridad
- [ ] ⚠️ ¿Quién administra la aplicación? ¿Necesita un panel de administración?
- [ ] ⚠️ ¿Hay zonas públicas y zonas privadas (login)?
- [ ] 💡 ¿Hay requisitos legales o de cumplimiento? (GDPR, facturación electrónica, etc.)
- [ ] 💡 ¿Necesita múltiples idiomas o monedas?

### Métricas de éxito
- [ ] ⚠️ ¿Cómo sabremos que la aplicación ha sido un éxito a los 6 meses?
- [ ] 💡 ¿Hay KPIs concretos que quieras medir? (conversión, retención, ventas, etc.)

---

## PARTE 1: RESUMEN EJECUTIVO

### 1.1 El problema
*(En 2-3 frases: qué problema existe hoy y para quién)*

### 1.2 La solución
*(En 2-3 frases: qué va a hacer la aplicación para resolver ese problema)*

### 1.3 Propuesta de valor
*(Por qué esta solución es mejor que lo que existe hoy)*

### 1.4 Métricas de éxito
| Métrica | Valor objetivo | Plazo |
|---|---|---|
| [ej: usuarios registrados] | [ej: 500] | [ej: 3 meses] |

---

## PARTE 2: USUARIOS

### 2.1 Roles y perfiles

| Rol | Descripción | Frecuencia de uso |
|---|---|---|
| [ej: Cliente] | [ej: Compra productos en la tienda] | [ej: Semanal] |
| [ej: Admin] | [ej: Gestiona catálogo y pedidos] | [ej: Diaria] |

### 2.2 User stories principales
*(Las más importantes — las que definen el core de la aplicación)*

**Como [rol], quiero [acción] para [beneficio].**

- Como [rol], quiero [acción] para [beneficio].
- Como [rol], quiero [acción] para [beneficio].
- Como [rol], quiero [acción] para [beneficio].

---

## PARTE 3: ALCANCE

### 3.1 IN SCOPE — Fase 1 (lo que construimos ahora)
- [módulo o funcionalidad 1]
- [módulo o funcionalidad 2]

### 3.2 OUT OF SCOPE — Fase 2+ (explícitamente excluido ahora)
- [funcionalidad A] — *(razón: complejidad / prioridad / presupuesto)*
- [funcionalidad B] — *(razón)*

### 3.3 Dependencias externas
| Servicio / Integración | Propósito | Obligatorio |
|---|---|---|
| [ej: Stripe] | [ej: Pagos online] | ✅ Sí / 🔲 No |

---

## PARTE 4: MÓDULOS FUNCIONALES

> Describe cada módulo principal de la aplicación.
> Sé específico — esto es lo que @QwikSpeccer convertirá en Specs técnicas.

### Módulo: [Nombre]
**Descripción:** *(qué hace este módulo)*

**Funcionalidades:**
- [funcionalidad 1]
- [funcionalidad 2]

**Usuarios que lo usan:** [roles]

**Criterios de aceptación de negocio:**
- [ ] [criterio 1 — verificable y binario]
- [ ] [criterio 2]

---

### Módulo: [Nombre]
*(repetir estructura para cada módulo)*

---

## PARTE 5: REQUISITOS NO FUNCIONALES

### Rendimiento
- Tiempo de carga máximo: [ej: < 3s en conexión media]
- Usuarios concurrentes esperados: [N]

### Seguridad
- [ ] Autenticación requerida
- [ ] Datos personales (GDPR)
- [ ] Pagos online (PCI DSS)
- [ ] Otros: [especificar]

### Accesibilidad
- Nivel requerido: [ej: WCAG 2.1 AA]
- Idiomas: [ej: Español, Inglés]

### Dispositivos
- [ ] Desktop (prioritario)
- [ ] Mobile responsive
- [ ] App nativa (fuera de scope)

---

## PARTE 6: DISEÑO Y UX

### 6.1 Tono y estilo
*(Describe brevemente la personalidad visual: moderno/clásico, minimalista/rico, etc.)*

### 6.2 Referencias visuales
*(URLs o nombres de aplicaciones que el cliente admira como referencia)*

### 6.3 Branding
- [ ] Logo y branding del cliente
- [ ] Favicon definido
- [ ] Paleta de colores definida: [sí/no — si sí, cuáles]
- [ ] Tipografía definida: [sí/no]
- [ ] Dark mode: [requerido / no requerido]

---

## PARTE 7: RESTRICCIONES Y RIESGOS

### 7.1 Restricciones
| Tipo | Descripción |
|---|---|
| Presupuesto | [orientativo] |
| Tiempo | [fecha límite si existe] |
| Tecnología | Stack: Qwik + QwikCity + Supabase + Drizzle + Tailwind |

### 7.2 Riesgos identificados
| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| [ej: Scope creep] | Alta | Alto | Fijar alcance en contrato |

---

## PARTE 8: HISTORIAL Y APROBACIONES

| Versión | Fecha | Autor | Cambio |
|---|---|---|---|
| 1.0 | [fecha] | [nombre] | Draft inicial |

**Aprobado por el cliente:** [ ] Sí — Fecha: ___________

---

> **Siguiente paso tras la aprobación:**
> Llevar este PRD al `BLUEPRINT.md` para descomponer en módulos técnicos,
> fases de desarrollo y orden de implementación.