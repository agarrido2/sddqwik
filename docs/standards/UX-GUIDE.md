# UX PRINCIPLES & PRODUCT DESIGN STRATEGY

> **Propósito:** Este documento define el "Cerebro de Diseño" del proyecto. No contiene reglas de CSS ni instrucciones de librerías; contiene leyes de comportamiento humano, psicología cognitiva y estrategia de producto para construir productos digitales de alta complejidad y uso recurrente (B2B o profesionales).
> **Rol:** Actúa como un Jefe de Producto (CPO) obsesionado con la eficiencia, la claridad y la conversión.

---

## 1. MANIFIESTO: "LA HERRAMIENTA INVISIBLE"


Nuestro objetivo no es que el usuario admire la interfaz, sino que **se olvide de ella** para concentrarse en su trabajo.

* **Principio de Doble Velocidad (Expertos vs Nuevos Usuarios):** El producto debe optimizarse para usuarios expertos y recurrentes, pero nunca a costa de bloquear a usuarios nuevos.
La eficiencia máxima se alcanza cuando:
- El flujo principal es directo y rápido para expertos.
- El sistema es tolerante con principiantes mediante ayudas discretas, valores por defecto inteligentes y aprendizaje progresivo.
- El onboarding no debe interrumpir el trabajo, sino desaparecer cuando deja de ser necesario.
* **Velocidad de Percepción > Velocidad de Carga:** La interfaz debe sentirse instantánea. Si un dato tarda en llegar, la estructura (Shell) ya debe estar ahí. La percepción de fluidez es más importante que los milisegundos reales.
* **Respeto por el Experto:** Diseñamos para usuarios que usarán la herramienta de forma recurrente. Priorizamos la densidad de información y la eficiencia sobre las animaciones decorativas o los espacios vacíos excesivos ("White space" decorativo).
* **Aburrido es Bueno:** En B2B, "predecible" significa "confiable". La creatividad se reserva para el marketing; el producto debe ser estrictamente estandarizado para reducir la curva de aprendizaje. **NOTA**: 'Aburrido' no significa visualmente pobre ni sin jerarquía. Significa predecible, consistente y cognitivamente estable.
La interfaz puede ser moderna y cuidada, pero nunca sorprendente en su comportamiento.

---

## 2. PSICOLOGÍA COGNITIVA Y CARGA MENTAL

### 2.1 Ley de Jakob (Familiaridad)
No reinventes patrones de navegación. Los usuarios pasan la mayor parte de su tiempo en *otros* sitios (Linear, Google, Slack, Stripe).
* **Mandato:** Si existe un estándar industrial para una interacción (ej: "Cmd+K" para buscar, "Esc" para cerrar modal, tres puntos "..." para menús de contexto), úsalo estrictamente. La innovación en la estructura de navegación suele generar fricción innecesaria.

### 2.2 Principio de Proximidad y Región Común (Gestalt)
* **Agrupación:** Los elementos relacionados deben estar visualmente conectados sin necesidad de líneas divisorias explícitas ("Borders"), simplemente por espaciado.
* **Contenedores:** Usa "Tarjetas" o contenedores con fondo solo cuando la información deba aislarse drásticamente del resto. Si no, el espacio negativo es el mejor separador para mantener la interfaz limpia.

### 2.3 Carga Cognitiva y Toma de Decisiones
* **Opciones Binarias:** Evita la parálisis por análisis. En lugar de preguntar "¿Qué configuración técnica avanzada deseas?", ofrece una **opción por defecto inteligente** y un botón de "Personalizar" discreto.
* **Divulgación Progresiva:** Muestra solo lo necesario para el paso actual. En formularios complejos, usa patrones de "Drill-down" (profundizar) o pasos secuenciales en lugar de mostrar todos los campos de golpe.

### 2.4 Performance Percibida y Expectativas del Usuario
El usuario no percibe milisegundos; percibe **estructura, continuidad y respuesta inmediata**
* **Estructura Primero**: La interfaz debe mostrar su esqueleto y jerarquía antes de que los datos estén disponibles. Un layout incompleto se percibe como error; un layout vacío pero estructurado se percibe como “cargando”.
* **Continuidad Visual**: Los cambios de estado deben ser progresivos y predecibles. Evita pantallas en blanco, saltos bruscos o bloqueos totales de la interfaz.
* **Respuesta Inmediata**: Toda acción del usuario debe generar una reacción visual instantánea, incluso si el resultado final depende de procesos asíncronos.

**Principio General**: La interfaz nunca debe "desaparecer” mientras el sistema trabaja.
---

## 3. ARQUITECTURA DE INFORMACIÓN (IA)

### 3.1 El Modelo Mental "Objeto-Acción"
El usuario piensa en sustantivos, no en verbos.
* **Correcto:** Navegar a lista de "Llamadas" -> Clic en una llamada -> Botón "Archivar".
* **Incorrecto:** Página de "Archivar cosas" -> Buscar la llamada.
* **Directriz:** La navegación principal siempre debe reflejar los **Objetos del Negocio** (Agentes, Llamadas, Facturas, Miembros), no las acciones.

### 3.2 Wayfinding (Orientación)
El usuario nunca debe preguntarse "¿Dónde estoy?".
* **Breadcrumbs (Migas de pan):** Obligatorios en cualquier vista que tenga más de 2 niveles de profundidad jerárquica.
* **Estado Activo:** El ítem del menú de navegación debe marcarse inequívocamente (cambio de peso tipográfico o fondo).
* **Consistencia de Títulos:** El título (H1) de la página debe coincidir semánticamente con el enlace que se clicó para llegar allí.

---

## 4. DISEÑO DE INTERACCIÓN Y FLUJOS (PRODUCTO)
---
**Marco de Estados del Sistema**
Todo elemento interactivo del producto existe siempre en uno de varios estados reconocibles.
Diseñar explícitamente estos estados reduce errores, ambigüedad y carga cognitiva:

- Reposo (Idle)
- Interacción (Hover / Focus)
- Procesando (Loading)
- Éxito (Success)
- Error (Error)
- No disponible (Disabled)

Ningún componente debe existir sin que sus estados estén definidos y sean visualmente distinguibles.
---

### 4.1 Estados Vacíos (The Blank Canvas Paradox)
Un dashboard vacío es intimidante y parece roto.
* **Nunca digas "No hay datos":** Di *"Aún no has recibido llamadas"* o *"Tu equipo está listo para empezar"*.
* **Llamada a la Acción (CTA):** El estado vacío es la mejor oportunidad de educación. Incluye siempre el botón principal para crear el primer registro.
* **Ilustración:** Usa metáforas visuales sutiles para indicar que el sistema está listo y esperando inputs, no fallando.

### 4.2 Optimistic UI (Mentiras Piadosas)
La interfaz debe ser más rápida que el servidor.
* **Acción Inmediata:** Cuando el usuario pulsa "Guardar", "Borrar" o "Favorito", la UI debe reflejar el cambio **instantáneamente** (cambiar estado visual, desaparecer fila), asumiendo que el servidor responderá OK.
* **Gestión de Error:** Solo si el servidor falla, revertimos el cambio y mostramos un error no intrusivo. Esto genera una sensación de fluidez extrema y confianza en la herramienta.

### 4.3 Modos de Edición y Contexto
* **Edición en Contexto (Inline):** Para cambios rápidos (ej: renombrar un agente, cambiar un estado), permite editar directamente el texto o usar un dropdown sin navegar a otra página.
* **Drawer (Panel Lateral) vs Modal:**
    * Usa **Modal** para decisiones de bloqueo, confirmaciones "Sí/No" o tareas muy breves.
    * Usa **Drawer (Slide-over)** para formularios largos o configuraciones complejas. Esto permite al usuario mantener el contexto visual de la página "padre" visible detrás mientras trabaja.

---

## 5. UX PARA DATOS E INTELIGENCIA ARTIFICIAL

### 5.1 Confianza y Transparencia
En sistemas automatizados o de IA, el usuario siente ansiedad de control ("¿Qué hizo la IA?", "¿Por qué?").
* **Logs y Auditoría:** Muestra siempre el "por qué". Si la IA tomó una decisión, muestra el fragmento de lógica o texto que la motivó.
* **Simulación (Sandbox):** Permite al usuario "probar" la configuración en un entorno seguro antes de publicar cambios en producción.

### 5.2 Visualización de Datos
* **Métricas con Contexto:** Un número aislado ("50 llamadas") no aporta valor. Acompáñalo siempre de una tendencia ("+10% vs semana pasada") o una meta ("50/100").
* **Escaneabilidad:** En tablas y listas, los usuarios escanean horizontalmente. Alinea textos a la izquierda, números a la derecha (fuentes tabulares) y estados/badges al centro.

### 5.3 Feedback de Sistema (System Status)
* **Visibilidad de Estado:** Si el sistema está procesando una tarea larga (ej: "Entrenando modelo"), muestra un indicador claro pero **no bloqueante**. El usuario debe poder seguir navegando por otras partes de la app mientras la tarea ocurre en segundo plano.

---

## 6. UX WRITING (MICROCOPY)

Las palabras son parte fundamental de la interfaz.

* **Sé Humano, no Robot:**
    * *Mal:* "Autenticación fallida. Credenciales inválidas."
    * *Bien:* "El correo o la contraseña no son correctos."
* **Verbos de Acción Específicos:** En botones, usa verbos que describan exactamente lo que pasará.
    * *Mal:* "Aceptar", "Sí", "OK".
    * *Bien:* "Crear Agente", "Publicar Cambios", "Borrar Definitivamente".
* **Voz Activa:** Siempre usa voz activa. "La IA llamó al cliente" es mejor que "El cliente fue llamado por la IA".

---

## 7. PREVENCIÓN DE ERRORES (POKA-YOKE)

El mejor mensaje de error es el que nunca tiene que aparecer.

* **Principio del Costo del Error:** No todos los errores tienen el mismo impacto. La fricción de la interfaz debe ser proporcional al daño potencial:
  - Errores reversibles → feedback ligero y rápido.
  - Errores persistentes → confirmación explícita.
  - Errores irreversibles o financieros → fricción alta y consciente.

* **Validación Temprana:** Indica si un campo es válido o inválido **mientras** el usuario escribe (o al perder el foco), no esperes a que pulse el botón "Enviar".
* **Fricción Intencional:** Para acciones destructivas e irreversibles, obliga a una confirmación consciente (ej: "Escribe BORRAR para confirmar") en lugar de un simple clic que puede ser accidental.
* **Defaults Inteligentes:** Pre-rellena formularios con los valores más probables (ej: Prefijo de país, configuración típica del sector). Reduce la fricción de entrada al mínimo.

---

## 8. ESTRATEGIA DE CRECIMIENTO (LANDING PAGE UX)

Mientras que la aplicación busca la eficiencia, la web pública busca la **persuasión ética y la conversión**.

### 8.1 La Regla de los 3 Segundos (The Hero Section)
El usuario llega escéptico y con el dedo en el botón "Atrás". El Hero debe responder tres preguntas antes de que parpadee:
1.  **¿Qué es esto?** (Claridad radical. "Agentes de Voz IA" > "Revoluciona tu comunicación").
2.  **¿Es para mí?** (Cualificación inmediata: "Para concesionarios de vehículos").
3.  **¿Qué gano yo?** (Beneficio directo: "Automatiza el 100% de tus llamadas").

### 8.2 Arquitectura de la Confianza
En B2B, nadie compra por impulso; compran por reducción de riesgo.
* **Prueba Social Técnica:** En lugar de citas genéricas ("Me encanta"), usa métricas de éxito cuantificables ("Recuperamos 40 horas/mes", "ROI de 3x").
* **Integraciones:** Mostrar logos de herramientas que el usuario ya utiliza (Salesforce, Google Calendar, WhatsApp) reduce drásticamente la ansiedad de implementación ("¿Funcionará con lo que ya tengo?").

### 8.3 Anatomía del "Feature" (Show, Don't Tell)
No listes funcionalidades abstractas. Vende superpoderes tangibles.
* **Visualización:** Cada beneficio debe ir acompañado de una **captura real de la interfaz** o una representación abstracta fiel que demuestre que el producto existe, es moderno y usable. Evita ilustraciones genéricas de stock que griten "software falso".

### 8.4 El Camino a la Conversión (CTA)
* **Un único objetivo:** Una página, una meta (ej: "Empezar Demo"). Cualquier otro enlace que saque al usuario del túnel (Blog, Redes Sociales en el header) es una fuga de conversión.
* **Fricción Cero:**
    * Si pides el email, no pidas la tarjeta de crédito.
    * Si pides la tarjeta, ofrece un trial sin compromiso y garantía clara.
* **Lenguaje de Beneficio:** Cambia el texto del botón de "Enviar" o "Registrarse" a algo que implique valor: "Ver a mi Agente en Acción" o "Empezar Prueba Gratuita".

## ACCESIBILIDAD FUNCIONAL (NO NEGOCIABLE)
La accesibilidad no es una “feature”; es un requisito base de calidad.
* **Contraste suficiente**: El contenido debe ser legible en cualquier condición de luz.
* **Navegación por teclado**: Todas las acciones críticas deben ser accesibles sin ratón.
* **Focus visible**: El usuario siempre debe saber dónde está interactuando.
* **Texto comprensible**: Evita tecnicismos innecesarios y ambigüedades.

```text
Un producto inaccesible genera fricción, errores y abandono, incluso en usuarios expertos.
```