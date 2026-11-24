# project_m · Planificador de proyectos en un solo HTML

Planificador tipo MS Project / ProjectLibre que vive en **un único archivo**: `planificador.html`.
Se abre con doble clic en Edge o Chrome. **No requiere instalar nada** (ni Node, ni Java, ni Python, ni servidor).

Este README es la guía de mantenimiento: con él y el repositorio se puede retomar y evolucionar el proyecto.

---

## 1. Qué hace

| Vista | Qué tiene |
|---|---|
| **Cronograma** (`#cronograma`) | Tabla editable (EDT con fases, tareas, hitos), predecesoras estilo MS Project, ★ hitos clave, y Gantt en SVG con ruta crítica, % completado, vínculos, fines de semana y festivos de Colombia, línea de hoy. |
| **Avance** (`#avance`) | Fecha de corte; indicadores (avance real del proyecto, planificado, desviación, hitos cumplidos, fin previsto); barras por sección con % real vs. % planificado y estado; tabla de hitos clave; opción "ver como tabla". |
| **Hitos** (`#hitos`) | Gantt solo de hitos (todos o solo ★ clave), con estado cumplido / vencido / próximo / pendiente y línea de corte; opción tabla. |

Menú **Archivo**: Nuevo, Abrir (`.plan.json` o XML), Guardar / Guardar como (`.plan.json`), Importar XML de
ProjectLibre / MS Project, Exportar XML para ProjectLibre / MS Project, Exportar a Excel (CSV), Imprimir / PDF, Cargar ejemplo.
También se puede **arrastrar y soltar** un archivo sobre la ventana. El plan se **autoguarda** en el navegador.

---

## 2. Reglas del proyecto (no romperlas)

1. **Un solo archivo**: HTML + CSS + JS en `planificador.html`. Sin archivos externos, sin CDN, sin dependencias.
2. **Sin build**: el JS está escrito a mano en JavaScript nativo (ES2020+). Lo que está en el archivo *es* el código fuente.
   No usar TypeScript, bundlers, npm ni Node.
3. **Funciona desde `file://`**: por eso **no se usan módulos** (`import`/`export` están bloqueados en `file://` en
   Edge/Chrome). Todo el JS va en un `<script>` clásico dentro de una IIFE con `'use strict'`.
4. **Navegador objetivo**: Edge / Chrome actuales (Chromium). Otros navegadores funcionan con degradación
   (abrir = subir archivo, guardar = descarga).
5. **Interfaz en español** (Colombia). Fechas `dd/mm/aa`, decimales con coma en los indicadores.
6. **Nunca perder datos en silencio**: lo que no se puede importar tal cual se ajusta **y se informa**; un archivo inválido
   se rechaza con mensaje claro; los cambios sin guardar se avisan antes de reemplazar el plan.
7. **Seguridad**: todo texto del usuario se inserta en HTML/SVG con `esc()`. No usar `innerHTML` con datos sin escapar.

---

## 3. Mapa del archivo

El archivo está dividido en secciones marcadas con `/* ===...=== nombre */`. Busca `=== ` para saltar entre ellas.

| Sección | Contenido principal |
|---|---|
| `estilos` | Variables CSS (modo claro y oscuro), layout, tabla, Gantt (`.g-*`), tableros (`.c-*`, `.tile`, `.status`), impresión. |
| `motor: fechas` | `toDayNumber`, `fromDayNumber`, `addDays`, `weekday`, `isoDate`, `isValidDate`, `todayIso`. Fechas `AAAA-MM-DD` en UTC. |
| `motor: festivos de Colombia` | `easterSunday`, `nextMonday` (Ley Emiliani), `colombiaHolidays(year)`. |
| `motor: calendario laboral` | `class WorkCalendar`: `isWorkingDay`, `isHoliday`, `nextWorkingDay`, `dateAt`, `offsetOf`, `workingDaysBetween`. |
| `motor: cronograma (ruta crítica)` | `class ScheduleError`, `schedule(project, calendar)` → fechas, holguras, ruta crítica. |
| `modelo: plan editable y EDT` | `levelOf`, `hasChildren`, `subtreeEnd`, `ancestorsOf`, `insertTask`, `deleteTask`, `indentTask`, `outdentTask`, `moveTask`, `updateTask`, `wbsCodes`, `samplePlan`. |
| `modelo: predecesoras estilo MS Project` | `parsePredecessors`, `formatPredecessors`, `PredecessorSyntaxError`. |
| `avance a la fecha de corte` | `computeProgress`, `statusOf`, umbrales; `listMilestones`, `milestoneStatus`. |
| `archivos: .plan.json` | `serializePlan`, `parsePlanFile`, `validatePlan`, `PlanFileError`. |
| `archivos: importar XML…` | `importMsProjectXml` (MSPDI), `isoDurationHours`, `MsProjectImportError`. |
| `archivos: exportar XML…` | `exportMsProjectXml` (con calendario "Colombia"). |
| `archivos: CSV para Excel` | `planToCsv`. |
| `archivos: abrir y guardar…` | `openFile`, `saveFileAs`, `writeToHandle`, `download`, `fileNameFor` (File System Access API + respaldo). |
| `vistas: utilidades` | `esc`, `shortDate`, `longDate`, `pct`, `pts`, `statusHtml`. |
| `vistas: tabla de tareas` | `COLUMNS`, `renderTableBody`. |
| `vistas: Gantt del cronograma (SVG)` | `renderGantt` (encabezado + cuerpo + `xOf`), `ROW_H`, `HEAD_H`, `DAY_WIDTH`. |
| `vistas: tablero de avance` | `progressSections`, `renderProgressChart`, `renderProgressTable`. |
| `vistas: Gantt de hitos` | `renderMilestoneTimeline`, `renderMilestoneTable`. |
| `aplicación` | Estado, layout (`#app`), `render()`, vistas, archivos, comandos (`run(cmd)`), eventos, atajos. |

**Dependencias entre capas** (de abajo hacia arriba): motor → modelo → avance → archivos → vistas → aplicación.
El motor y el modelo no tocan el DOM (salvo `importMsProjectXml`, que usa `DOMParser`).

---

## 4. Modelo de datos

### Plan
```js
{
  name: 'Proyecto de ejemplo',   // nombre del proyecto
  startDate: '2026-02-02',       // AAAA-MM-DD; si no es laboral se usa el siguiente día laboral
  statusDate: '2026-02-20',      // opcional: fecha de corte del avance (si falta, se usa hoy)
  nextId: 14,                    // siguiente número para ids t1, t2, ...
  tasks: [Task, ...]             // ORDEN = orden de filas
}
```

### Task
```js
{
  id: 't6',                      // único, estable (no cambia al mover filas)
  name: 'Backend',
  duration: 8,                   // días laborales, entero >= 0; 0 = hito; en tareas resumen se ignora (se calcula)
  parentId: 't5',                // opcional: tarea resumen padre (EDT)
  dependencies: [                // predecesoras
    { predecessorId: 't4', type: 'FS', lag: 0 }   // type: FS | SS | FF | SF; lag en días laborales (negativo = adelanto)
  ],
  startNoEarlierThan: '2026-02-16', // opcional: restricción "no comenzar antes de"
  percentComplete: 75,           // opcional: 0-100 (en hitos, 100 = cumplido)
  key: true                      // opcional: hito clave (★)
}
```

**Invariantes que mantienen todas las operaciones del modelo**
- Los descendientes de una tarea van **justo debajo** de ella (subárbol contiguo). `levelOf` se deriva de `parentId`.
- Una **tarea resumen** (tiene hijas) no tiene dependencias **ni es predecesora** de nadie: `becomeSummary()` las quita
  cuando una tarea pasa a ser resumen (sangría / quitar sangría) y la interfaz avisa cuántas se quitaron.
- Las operaciones son **inmutables**: devuelven un plan nuevo (`clonePlan`), nunca modifican el recibido.
- `outdentTask` sigue a MS Project: las hermanas que estaban debajo pasan a ser hijas de la tarea que sube.

---

## 5. Motor de cronograma (`schedule`)

- Trabaja en **índices de día laboral** desde el inicio del proyecto. Cada tarea ocupa `[ES, EF)` con `EF = ES + duración`
  (EF exclusivo). Solo al final se convierten a fechas con `WorkCalendar.dateAt`.
- Restricciones de una dependencia P → S con retraso L:

  | Tipo | Regla (pase hacia adelante) | Pase hacia atrás (fin tardío de P) |
  |---|---|---|
  | FS fin-comienzo | `S.ES >= P.EF + L` | `S.LS - L` |
  | SS comienzo-comienzo | `S.ES >= P.ES + L` | `S.LS - L + P.dur` |
  | FF fin-fin | `S.EF >= P.EF + L` | `S.LF - L` |
  | SF comienzo-fin | `S.EF >= P.ES + L` | `S.LF - L + P.dur` |

- `ES` nunca es menor que 0 (inicio del proyecto) ni que `startNoEarlierThan`.
- Holgura total = `LS - ES`; **crítica** si holgura `<= 0`. `criticalPath` = hojas críticas ordenadas por inicio.
- **Fechas mostradas**: `start = dateAt(ES)`, `finish = dateAt(EF - 1)` (último día de trabajo, incluido).
  Un **hito** se muestra "al final del día anterior": `dateAt(max(ES - 1, 0))`, es decir, el día en que termina su predecesora.
- **Tareas resumen**: fechas, duración y holgura salen de sus subtareas (mínimo inicio, máximo fin, menor holgura).
- Orden topológico con Kahn. Si hay ciclo, se reportan **solo las tareas del ciclo** (se descartan las que solo dependen de él).
- `ScheduleError.code`: `DUPLICATE_ID`, `UNKNOWN_PARENT`, `PARENT_CYCLE`, `SUMMARY_DEPENDENCY`, `INVALID_DURATION`,
  `SELF_DEPENDENCY`, `UNKNOWN_TASK`, `INVALID_LAG`, `CYCLE`; `taskIds` lleva las tareas involucradas (la interfaz las
  marca en rojo y muestra su número de fila).

### Calendario y festivos
- Lunes a viernes, más los **18 festivos de Colombia** calculados por año:
  fijos (1 ene, 1 may, 20 jul, 7 ago, 8 dic, 25 dic); trasladables al lunes por Ley Emiliani (6 ene, 19 mar, 29 jun,
  15 ago, 12 oct, 1 nov, 11 nov); según Pascua (jueves y viernes santo; Ascensión +43, Corpus +64, Sagrado Corazón +71 días).
  Si dos coinciden el mismo lunes (ej. 30 jun 2025) se cuentan una vez.
- `WorkCalendar` acepta `workingWeekdays` y `holidaysForYear` por si algún día se quiere otro país u horario.

### Predecesoras (texto de la tabla)
`2, 3CC+2, 4FF-1d` → número de **fila** + tipo + retraso. Tipos en español (FC, CC, FF, CF) o inglés (FS, SS, FF, SF);
sin tipo = fin-comienzo. Se escriben en español y se omite `FC` cuando no hay retraso (`2`, `3CC+2`, `5FC+1`).
Las filas se traducen a ids internos, así que mover filas no rompe vínculos.

---

## 6. Avance e hitos

- **% real** = `percentComplete` de cada tarea.
- **% planificado a la fecha de corte**: `0` antes de su inicio, `100` desde su fin, en medio
  `días laborales transcurridos (incluido el de corte) / duración`. Un hito vale `100` si su fecha ya pasó.
- **Resúmenes y proyecto**: promedio **ponderado por duración** de sus tareas (como MS Project); los hitos pesan 0.
  Si una sección solo tiene hitos, se promedian los hitos. Clave del total del proyecto: `PROJECT_KEY = '__proyecto'`.
- **Estado** (`statusOf`): completado (real 100) · sin iniciar (0/0) · al día (desviación ≥ −5 pts) ·
  en riesgo (−5 a −15) · atrasado (< −15). Umbrales: `RISK_THRESHOLD`, `LATE_THRESHOLD`.
- **Secciones** del gráfico (`progressSections`): "Fases" = tareas de primer nivel (resúmenes y tareas con duración);
  "Todas" = todas las tareas resumen.
- **Hitos** (`milestoneStatus`): cumplido (100 %) · vencido (fecha < corte) · próximo (dentro de `UPCOMING_DAYS` = 14 días)
  · pendiente. `listMilestones` añade la fase de primer nivel a la que pertenece cada hito.

---

## 7. Formatos de archivo

### `.plan.json` (formato propio)
```json
{ "format": "planificador", "version": 1, "savedAt": "2026-11-23T19:00:00.000Z", "plan": { ... } }
```
- `validatePlan` revisa todo (tipos, fechas reales, ids únicos, padres y predecesoras existentes) y recalcula `nextId`
  si viene mal. **Si se agrega un campo nuevo a Task/Plan, hay que agregarlo en `validatePlan`**, si no se pierde al abrir.
- Si cambia el formato de forma incompatible: subir `FILE_VERSION` y migrar en `parsePlanFile`.

### Navegador (`localStorage`)
- `planificador.plan.v1`: el plan actual (autoguardado en cada cambio).
- `planificador.file.v1`: `{ name, saved }` = nombre del archivo y el plan tal como quedó guardado (para saber si hay
  cambios). El `FileSystemFileHandle` solo vive en la pestaña: tras recargar, "Guardar" pide ubicación.

### XML de MS Project / ProjectLibre (MSPDI)
ProjectLibre exporta e importa este formato (Archivo → Guardar como → XML). Su `.pod` nativo **no** es legible (sin especificación).

| MSPDI | Planificador | Notas |
|---|---|---|
| `Project/Title` (o `Name`), `StartDate`, `StatusDate`, `MinutesPerDay` | `name`, `startDate`, `statusDate` | `MinutesPerDay` por defecto 480 (8 h) |
| `Task/OutlineLevel` | `parentId` | padre = fila anterior con un nivel menos; se ignora UID 0, `IsNull`, inactivas (se avisa) |
| `Task/Duration` (`PT40H0M0S`) | `duration` | horas / (MinutesPerDay/60); fracciones se **redondean y se avisa** |
| `PercentComplete` | `percentComplete` | |
| `ConstraintType = 4` + `ConstraintDate` | `startNoEarlierThan` | otras restricciones se ignoran y se avisa |
| `PredecessorLink/Type` (0 FF, 1 FS, 2 SF, 3 SS) | `type` | |
| `LinkLag` (décimas de minuto) / `LagFormat` | `lag` | `LagFormat` 19/20 = porcentaje de la predecesora; se redondea y avisa |
| vínculo hacia o desde una tarea resumen | — | **se quita y se avisa** (el motor no los admite) |

**Exportar** genera MSPDI con: tarea resumen del proyecto (UID 0), `UID`/`ID` = número de fila, `WBS`/`OutlineNumber`,
`OutlineLevel`, `Start` 08:00 / `Finish` 17:00, `Duration` en horas (8 h/día), `PercentComplete`, `Summary`,
`Milestone`, `Critical`, restricción, vínculos (`LinkLag` = días × 4800, `LagFormat` 7) y un calendario base
**"Colombia"** (L-V 8-12 y 13-17) con los festivos como excepciones desde un año antes del inicio hasta dos después del fin.
El ★ de hito clave **no** viaja en el XML (el formato no tiene campo para eso).
Ida y vuelta (exportar → importar) da el mismo plan y las mismas fechas.

### CSV para Excel
Separador `;` (configuración regional de Colombia), BOM UTF-8 (tildes correctas), `\r\n`. Columnas:
Fila; EDT; Nombre; Tipo; Duración (días); Comienzo; Fin; Predecesoras; % real; % planificado; Holgura (días); Crítica; Hito clave.

---

## 8. Aplicación (interfaz)

### Estado (sección `aplicación`)
- `plan` (fuente de verdad), `selectedId`, `zoom` (`dias`/`semanas`/`meses`), `view` (`cronograma`/`avance`/`hitos`),
  `collapsed` (resúmenes contraídos), `pendingFocus` (celda a enfocar tras re-dibujar), `boardOptions`
  (`level`, `progressTable`, `onlyKey`, `milestoneTable`), `fileHandle`, `fileMeta`.
- **Ciclo**: toda edición crea un plan nuevo y llama `commit(next)` → guarda en `localStorage` → `render()`.
  `render()` recalcula con `schedule()` y dibuja la vista activa, el encabezado, el aviso de error y la barra de estado.
- Las vistas se generan como **strings HTML/SVG** y se asignan con `innerHTML` (todo texto pasa por `esc()`).

### Eventos (delegados en `document`)
- `click` sobre `[data-cmd]` → `run(cmd)`; `[data-view]` → `setView`; `[data-action="toggle"|"key"]` en filas.
- `change` en celdas `input[data-field]` (`name`, `duration`, `pred`, `pct`) y en controles (`#project-name`,
  `#project-start`, `#status-date`, `#zoom`, `#level`, `#progress-table`, `#only-key`, `#milestone-table`).
- Comandos (`run`): `add`, `milestone`, `delete`, `indent`, `outdent`, `up`, `down`, `new`, `open`, `import`, `save`,
  `save-as`, `export-xml`, `export-csv`, `print`, `sample`, `cut-today`, `goto-start`, `goto-today`.
- Atajos: Enter (confirma y baja), Shift+Enter (sube), Esc (cancela), Ctrl/Cmd+S, Ctrl/Cmd+Shift+S, Ctrl/Cmd+O,
  Ctrl/Cmd+Enter (nueva tarea), Supr (eliminar fila seleccionada), Alt+Shift+→ ← ↑ ↓ (sangría, quitar, subir, bajar).
- `#cronograma`, `#avance`, `#hitos` en la URL abren esa vista (`hashchange`).
- Tabla y Gantt sincronizan el scroll vertical; filas de **30 px** en ambos (`ROW_H`, alto de `td`) y encabezados de **48 px**:
  si cambias uno, cambia el otro.

### Ejemplo
`samplePlan(sampleStart())` arranca 15 días laborales antes de hoy, para que el tablero de avance muestre estados reales.

---

## 9. Estilo y gráficos

- Colores como **variables CSS** en `:root`, con bloque `@media (prefers-color-scheme: dark)` para modo oscuro
  (automático según el sistema).
- Paleta validada (daltonismo y contraste, claro y oscuro): serie `#2a78d6` (oscuro `#3987e5`), ruta crítica /
  crítico `#d03b3b`; estados fijos: bien `#0ca30c`, alerta `#fab219`, serio `#ec835a`, crítico `#d03b3b`.
  **Si agregas colores de series, valida la paleta** antes (distancia para daltonismo y contraste ≥ 3:1 sobre la superficie).
- Reglas que siguen los tableros: un solo número grande (hero) por vista; barras delgadas (14 px) con extremo redondeado
  de 4 px y base recta; líneas de referencia finas y **continuas** (no punteadas); leyenda siempre que hay ≥ 2 elementos;
  **estado con ícono + texto, nunca solo color**; textos en tinta (no en el color de la serie); toda vista tiene versión tabla;
  tooltips (`<title>`) como complemento, nunca como única forma de ver un dato.
- Impresión: `@media print` oculta barras y controles; "Imprimir / PDF" imprime la vista activa.

---

## 10. Cómo validar cambios (sin Node)

Todo se valida con **Edge en modo headless**, abriendo archivos locales. Ruta en macOS:
`/Applications/Microsoft Edge.app/Contents/MacOS/Microsoft Edge` (en Windows, `msedge.exe`).

1. **Lógica (motor, modelo, archivos)**: crear una página temporal (fuera del repo) con el JS de las secciones de
   `motor` a `vistas` copiado dentro de un `<script>` y, al final, un script de verificaciones que escriba los resultados
   en `document.body` (OK / FALLA por caso). Ejecutar:
   ```bash
   "$EDGE" --headless=new --disable-gpu --user-data-dir=/tmp/perfil-prueba --virtual-time-budget=5000 \
     --dump-dom "file:///ruta/validar.html"
   ```
   Casos mínimos a cubrir: festivos 2025/2026, los 4 tipos de vínculo con retraso, holgura y ruta crítica, ciclos,
   resúmenes, restricción, hitos, predecesoras (leer/escribir/errores), operaciones de EDT, `.plan.json` ida y vuelta y
   archivos inválidos, importar XML y **exportar→importar ida y vuelta**, CSV, avance y estados de hitos.
   Romper algo a propósito una vez para confirmar que la verificación lo detecta.
2. **Flujos de usuario**: copia temporal del HTML con un `<script>` al final que simule acciones (fijar `value` y
   disparar `change`, `click` en `[data-cmd]`, `DragEvent('drop')` con un `File` para importar) y escriba resultados en un
   `<pre>`; leerlo con `--dump-dom`. Sobrescribir `window.confirm = () => true` (si no, la página queda bloqueada) y
   **esperar por condición** (lectura de archivos y guardado son asíncronos; `--virtual-time-budget` no los acelera).
3. **Visual**: capturas de cada vista y en modo oscuro:
   ```bash
   "$EDGE" --headless=new --disable-gpu --user-data-dir=/tmp/perfil-ui --window-size=1600,1000 \
     --virtual-time-budget=6000 --screenshot=/tmp/avance.png "file:///ruta/planificador.html#avance"
   # modo oscuro: agregar --force-dark-mode --blink-settings=preferredColorScheme=0
   ```
   Revisar solapes de texto, alineación tabla/Gantt, etiquetas cortadas, y que la consola no tenga errores de la página
   (`--enable-logging=stderr --v=0`; los mensajes de `cv_display_link_mac` son del modo headless, no de la página).

Las páginas y perfiles temporales se borran al terminar; **el repo solo contiene `planificador.html` y este README**.

---

## 11. Limitaciones conocidas

- Duraciones y retrasos en **días enteros** (sin horas ni medios días).
- Un solo calendario (L-V + festivos de Colombia); sin calendarios por tarea ni recursos con calendario propio.
- No se arrastran barras en el Gantt: se edita desde la tabla.
- Sin recursos ni costos, sin línea base, sin deshacer/rehacer.
- El ★ hito clave se pierde al exportar a MSPDI (sí se conserva en `.plan.json`).
- Sin vínculos hacia/desde tareas resumen (se ponen en subtareas; al importar se quitan con aviso).
- ProjectLibre `.pod` no se lee directo (usar XML).

## 12. Ideas para evolucionar

- **Línea base** (guardar fechas planeadas y comparar en Gantt y tablero).
- **Recursos** (asignación, carga, sobreasignación) y costos.
- **Deshacer / rehacer** (pila de planes, ya que las operaciones son inmutables).
- Arrastrar barras para cambiar fechas/duración y crear vínculos desde el Gantt.
- Columna "No comenzar antes de" editable en la tabla y más restricciones de MS Project.
- Curva S de avance (planificado vs. real en el tiempo) y avance por responsable.
- Exportar el ★ hito clave en el XML (por ejemplo en un campo de texto personalizado `Text1`).

---

## 13. Publicar cambios

Repositorio: `https://github.com/stiven-duque/project_m` (público), rama `main`.

- La identidad de git es **solo de este repositorio** (`git config user.name/user.email` locales):
  `Fabian Stiven Duque <87889181+stiven-duque@users.noreply.github.com>`. No usar el correo corporativo.
- En este equipo `gh` tiene dos cuentas; el push debe salir con **`stiven-duque`**:
  ```bash
  gh auth switch --hostname github.com --user stiven-duque
  git add planificador.html README.md
  git commit -m "describe el cambio"
  git push
  ```
- Mantener este README al día cuando cambie el modelo de datos, el formato de archivo, las secciones o las reglas.

## 14. Para pedir un cambio a Claude

Pásale el repositorio (o `planificador.html` + este README) y describe el cambio. El flujo esperado:
leer este README → ubicar la sección por su marcador `===` → hacer el cambio respetando las reglas de la sección 2 →
validar con la sección 10 (lógica, flujos y capturas) → actualizar este README si aplica → commit y push con la sección 13.
