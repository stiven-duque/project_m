# Ejemplo: línea base, reprogramación, cortes y curva S

Proyecto corto de dos fases (cada una con una tarea y un hito) para practicar el ciclo completo:
crear el plan con la fecha de inicio correcta → fijar la línea base → registrar cortes semanales →
reprogramar una tarea atrasada → revisar avance, hitos y curva S.

**Archivo listo:** [`prueba-linea-base.plan.json`](prueba-linea-base.plan.json) es el resultado de seguir los pasos 1 a 9.
Ábrelo en `planificador.html` con **Archivo → Abrir…** (o arrastrándolo sobre la ventana) para ver el resultado final,
o sigue los pasos desde cero para construirlo tú.

Las fechas ya consideran los festivos de Colombia (el lunes 16/11/2026 es festivo).

---

## Paso 1. Plan nuevo con la fecha de inicio correcta
1. Abre `planificador.html` → **Archivo → Nuevo plan**.
2. Arriba, en el nombre, escribe **Prueba línea base**.
3. **Comienzo: 03/11/2026.** Ninguna tarea puede empezar antes de esta fecha.

## Paso 2. Fases, tareas e hitos (pestaña Cronograma, columnas Planificación)

| Acción | Fila | Qué escribir |
|---|---|---|
| **＋ Tarea** | 1 | Nombre: `Plan 1` |
| **＋ Tarea** y luego **⇥** (sangría) | 2 | `task 1`, Duración **5** |
| **◆ Hito** | 3 | `Hito 1 - Requisitos aprobados`, Predecesoras **2**, clic en ☆ para marcarlo ★ |
| Selecciona la fila 1 y pulsa **＋ Tarea** | 4 | `plan 2` |
| **＋ Tarea** y luego **⇥** | 5 | `task 2`, Duración **8**, Predecesoras **3** |
| **◆ Hito** | 6 | `Hito 2 - Entrega`, Predecesoras **5**, ★ |

**No escribas nada en la columna Comienzo**: las predecesoras ya ubican las tareas (Comienzo solo sirve para
*atrasar* una tarea a propósito).

Fechas que deberías ver: task 1 **mar 03/11 → lun 09/11** · Hito 1 **09/11** ·
task 2 **mar 10/11 → vie 20/11** (salta el festivo del 16/11) · Hito 2 **20/11**.

## Paso 3. Línea base
1. Pulsa **Crear línea base**. Las columnas cambian a **Línea base** y aparece "✎ LB 1 en borrador".
2. Revisa *Inicio LB* / *Fin LB*. Si quieres margen, este es el momento (por ejemplo, *Fin LB* de task 2 al 23/11).
   En el archivo de ejemplo se dejó igual.
3. Pulsa **Fijar LB 1** y confirma. Queda "🔒 LB 1 fijada": desde aquí la línea base ya no se mueve.

## Paso 4. Primer corte (semana 1)
1. Pestaña **Avance** → **Fecha de corte: 06/11/2026**.
2. Vuelve a **Cronograma** y en **% real** de `task 1` escribe **60**.
3. En **Avance** pulsa **📌 Registrar corte**.

Se esperaba 80 % (4 de 5 días): la verás atrasada.

## Paso 5. Segundo corte (semana 2)
1. Fecha de corte: **13/11/2026**.
2. `task 1` → **100** (terminada) · `Hito 1` → **100** (cumplido) · `task 2` → **30**.
3. **📌 Registrar corte**.

## Paso 6. Reprogramar (task 2 se atrasa)
1. **Cronograma**, columnas **Línea base**: cambia el **Fin** de `task 2` a **25/11/2026**.
2. Pide motivo: escribe `Proveedor entregó tarde`.
3. Resultado: el **Hito 2 se mueve solo** al 25/11 · **Var. fin `+3 d`** en task 2 y en el Hito 2 ·
   historial **↻ 1** en task 2 (clic para ver el cambio y el motivo) · la **línea base sigue en el 20/11**.

## Paso 7. Tercer corte (semana 3)
1. Fecha de corte: **20/11/2026**.
2. `task 2` → **70**.
3. **📌 Registrar corte**.

## Paso 8. Revisar los resultados
- **Cronograma**: barra gris (línea base) debajo de la barra actual; se ve el corrimiento de task 2.
  Columnas **Avance**: % real, % esperado según LB, % esperado con fechas actuales y desviación por tarea.
- **Avance**: real del proyecto, esperado según LB 1, esperado con fechas actuales y desviación.
- **Hitos**: el Hito 2 con el rombo hueco en el 20/11 (línea base) unido al rombo actual del 25/11 y `+3 d`.
- **Curva S**: tres líneas (esperado LB, esperado actual, real) con un punto real por corte; pasa el mouse para ver
  los valores.

## Paso 9. Cerrar el proyecto
1. Fecha de corte: **27/11/2026**.
2. `task 2` y `Hito 2` → **100**.
3. **📌 Registrar corte**: la curva real llega al 100 %.

Valores del archivo de ejemplo (todo el proyecto):

| Corte | Esperado LB 1 | Real |
|---|---|---|
| 06/11/2026 | 30,8 % | 18,8 % |
| 13/11/2026 | 69,2 % | 51,9 % |
| 20/11/2026 | 100 % | 79,4 % |
| 27/11/2026 | 100 % | 100 % |

## Paso 10 (opcional). Nueva línea base
Si aprueban un cambio de alcance: **Nueva línea base…** (con motivo) → ajusta → **Fijar LB 2**.
En **Comparar con** puedes alternar entre LB 1 y LB 2. *(No está incluido en el archivo de ejemplo.)*

## Paso 11. Guardar
**Archivo → Guardar como…** → `prueba-linea-base.plan.json` (guarda línea base, cortes e historial).

---

## Errores comunes
- **Tareas antes del Comienzo del proyecto**: no se puede; cambia primero el Comienzo del proyecto (arriba, junto al nombre).
- **Escribir un Comienzo en tareas con predecesora**: no las adelanta, solo puede atrasarlas.
- **Fijar la línea base antes de revisar las fechas**: una vez fijada no se corrige; tocaría crear la LB 2.
- **Olvidar registrar el corte**: la curva real solo tiene puntos en los cortes registrados.
