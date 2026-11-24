# project_m

Planificador de proyectos en **un solo archivo HTML**: no requiere instalar nada.
Ábrelo con doble clic en Edge o Chrome (`planificador.html`).

## Qué hace

- **Cronograma:** tareas, fases (EDT), hitos y predecesoras al estilo de MS Project (`2`, `3CC+2`, `4FF-1`).
  Gantt con ruta crítica, % completado y festivos de Colombia (lunes a viernes).
- **Avance:** % real contra % planificado a una fecha de corte, por fase, con estado (al día / en riesgo / atrasado).
- **Hitos:** Gantt solo de hitos, con hitos clave (★) y estado (cumplido / vencido / próximo / pendiente).
- **Archivos:** guarda y abre `.plan.json`; importa y exporta XML de ProjectLibre / MS Project; exporta a Excel (CSV);
  imprime o guarda en PDF.

Enlaces directos a cada vista: `planificador.html#cronograma`, `#avance`, `#hitos`.

## Notas

- ProjectLibre guarda en `.pod` (formato sin especificación pública): para traer un proyecto, en ProjectLibre usa
  **Archivo → Guardar como → XML** y ábrelo aquí.
- El plan se autoguarda en el navegador; para compartirlo, guárdalo como `.plan.json` y envía ese archivo junto con el HTML.
