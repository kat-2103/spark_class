# 📂 16 - Ejercicio PySpark Súper Avanzado - Análisis Complejo de Datos de E-commerce (Enunciado)

---

# Objetivo

Realizar un procesamiento avanzado de datos usando PySpark, simulando un escenario real de **E-commerce**:

- Limpieza de datos.
- Transformaciones complejas.
- Joins entre DataFrames.
- Agregaciones avanzadas.
- Análisis temporal.

Este ejercicio está diseñado para tomarse entre **20-30 minutos** y poner a prueba un nivel intermedio-alto de PySpark.

---

# Descripción del ejercicio

Dispones de **dos datasets**:

### Dataset 1: `pedidos`

| id_pedido | id_cliente | fecha_pedido | importe_total |
|:---------:|:----------:|:------------:|:-------------:|
| 1         | 101        | 2023-01-15    | 150.0         |
| 2         | 102        | 2023-01-17    | 200.0         |
| 3         | 101        | 2023-02-01    | 300.0         |
| 4         | 103        | 2023-02-10    | 80.0          |
| 5         | 104        | 2023-03-05    | 50.0          |
| 6         | 102        | 2023-03-08    | 120.0         |


### Dataset 2: `clientes`

| id_cliente | nombre_cliente | ciudad       |
|:----------:|:--------------:|:------------:|
| 101        | Ana García      | Madrid       |
| 102        | Luis Pérez      | Barcelona    |
| 103        | Marta Sánchez   | Valencia     |
| 104        | Jorge López     | Sevilla      |

> *Puedes crear estos datasets manualmente en el Notebook usando listas de tuplas.*

---

# 🛠Tareas a realizar

1. Crear los **DataFrames** de `pedidos` y `clientes`.
2. Asegúrate de que las columnas de fecha estén en tipo `date`.
3. **Agregar una nueva columna** en `pedidos` llamada `mes_pedido` extrayendo el mes de `fecha_pedido`.
4. **Filtrar** solo los pedidos con `importe_total > 100`.
5. **Hacer un Join** entre `pedidos` y `clientes` usando `id_cliente`.
6. **Calcular por ciudad**:
   - El número total de pedidos.
   - El importe total acumulado.
   - El ticket medio (importe_total medio).
7. **Mostrar el Top 3 ciudades** con mayor importe total.
8. **Crear una tabla resumen** con:
   - ciudad
   - mes_pedido
   - número de pedidos
   - importe total del mes
9. **Guardar el resultado** final como archivo Parquet.

---

# Notas y ayudas

- Usa `from pyspark.sql.functions import col, month, sum, avg, count`.
- El Join puede ser de tipo `inner`.
- Para extraer el mes de una fecha, usa `month(fecha_pedido)`.
- Para ordenar y limitar resultados, usa `orderBy()` y `limit()`.

---

# Resultado esperado

Obtener insights reales de los datos de ventas por ciudad y mes, aplicando transformaciones complejas.


---

# Habilidades que trabajas en este ejercicio

- Gestión de múltiples DataFrames.
- Transformaciones sobre fechas.
- Join entre tablas.
- Agregaciones por grupo.
- Escritura de resultados en almacenamiento.

---
