# Estándar de Contratos de Serialización (Qwik Resumability)

Propósito: Garantizar que el estado de la aplicación sea 100% resumible, eliminando fallos en tiempo de ejecución causados por datos no serializables en la frontera Servidor -> Cliente.

## ⚡ Reglas Críticas (leer siempre — 30 segundos)
> El agente lee este bloque en TODAS las tareas.
> El resto del documento solo si la tarea requiere detalle.

1. PROHIBIDO: Usar `class` para objetos de dominio consumidos por componentes — Qwik no puede serializar instancias de clase. Usar siempre POJOs/interfaces puras.
2. OBLIGATORIO: Todo dato que cruce una frontera `$()` debe ser POJO, primitivo o Signal. Prohibido: Promesas activas, Maps, Sets, instancias de clase.
3. PATRÓN: Aplicar `noSerialize()` agresivamente a cualquier dato de librería de terceros (Charts, Mapas, etc.) e inicializar en `useVisibleTask$`.

## 1. El Problema de la Hidratación
Qwik no "hidrata"; "reanuda". Para ello, todo el estado debe ser serializable en el HTML (JSON-like). Si un agente introduce un objeto no serializable, la app se romperá al intentar reanudarse en el navegador.

## 2. Reglas Innegociables de Tipado
1. Data Transfer Objects (DTO): Los servicios en lib/ y features/ deben devolver exclusivamente interfaces puras (type o interface).
2. Prohibición de Clases: Queda terminantemente prohibido el uso de class para objetos de dominio que deban ser consumidos por componentes. Las clases no son serializables por Qwik. Usa POJOs (Plain Old JavaScript Objects).
3. Frontera del Loader: Todo dato devuelto por un routeLoader$ debe ser capaz de pasar por JSON.stringify().
   - ✅ Permitido: Primitivos, Arrays, Objetos Planos, Dates (Qwik las soporta).
   - ❌ Prohibido: Funciones, Mapas, Sets, Instancias de Clases, Promesas (fuera de la resolución del loader).

## 3. Contratos de Interfaz
- Clean API: Los servicios deben mapear los resultados complejos de la base de datos a estructuras simples antes de entregarlos al Orchestrator.
- Tipado en Features: Cada feature debe definir sus contratos en src/features/[feature]/types.ts para asegurar que el Builder sepa qué datos son seguros de pasar a la UI.

## 4. Auditoría de Resumabilidad
El @QwikAuditor rechazará cualquier PR que:
- Intente pasar una instancia de una clase a través de un context$.
- Use un objeto con referencias circulares.
- No defina explícitamente el tipo de retorno de un routeLoader$.