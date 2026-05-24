# Sessions

Este directorio contiene la memoria operativa de SDD Qwik.

`docs/sessions/INDEX.md` es la primera fuente de verdad operativa para
@QwikOrchestrator, @QwikMemory y `/new-session`.

En proyectos consumidores, `/setup` debe crear `docs/sessions/INDEX.md`
si no existe.

El repositorio base SDD Qwik puede mantener este directorio sin un INDEX real
para evitar confundir memoria del kit con memoria de una aplicación concreta.

Versionar:
- `README.md`
- snapshots relevantes si el proyecto lo decide
- `INDEX.md` en proyectos reales

No versionar por defecto:
- `archive/`
- `*.tmp.md`