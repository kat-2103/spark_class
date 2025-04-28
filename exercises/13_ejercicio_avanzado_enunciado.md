# 📂 13 - Ejercicio PySpark Avanzado - Análisis de Ventas (Enunciado)

---

# Objetivo

Aplicar transformaciones y acciones en PySpark sobre un conjunto de datos de ventas para:

- Limpieza de datos
- Filtrado de registros
- Agrupación y agregación
- Ordenación de resultados

---

# Descripción del ejercicio

Tienes la siguiente información de ventas:

| ID Venta | Producto | Categoría | Precio | Cantidad | Ciudad     |
|:--------:|:--------:|:---------:|:------:|:--------:|:----------:|
| 1        | Camiseta | Ropa      | 15.0   | 2        | Madrid     |
| 2        | Zapatos  | Calzado   | 50.0   | 1        | Barcelona  |
| 3        | Pantalón | Ropa      | 25.0   | 3        | Madrid     |
| 4        | Bufanda  | Accesorios| 10.0   | 5        | Valencia   |
| 5        | Zapatos  | Calzado   | 55.0   | 1        | Sevilla    |
| 6        | Camiseta | Ropa      | 15.0   | 4        | Valencia   |
| 7        | Gorro    | Accesorios| 12.0   | 2        | Barcelona  |

**El dataset puede crearse manualmente en el notebook usando una lista de tuplas.**

---

# Tareas a realizar

1. **Crear un DataFrame** a partir de estos datos.
2. **Agregar una columna** llamada `TotalVenta` que sea `Precio * Cantidad`.
3. **Filtrar** las ventas donde `TotalVenta > 50`.
4. **Agrupar** los datos por `Categoría` y calcular:
   - El **total de ventas** (`sum(TotalVenta)`)
   - El **número de ventas** (`count(ID Venta)`)
5. **Ordenar** el resultado del agrupamiento por `Total de Ventas` de mayor a menor.
6. **Mostrar** los resultados finales usando `.show()`.
7. (Opcional) **Guardar** el resultado en un archivo Parquet (solo si trabajas en local).

---

# Notas

- Utiliza funciones de PySpark como:
  - `.withColumn()`
  - `.filter()`
  - `.groupBy()`
  - `.agg()`
  - `.orderBy()`
- Puedes usar:

```python
from pyspark.sql.functions import col, sum, count
```

---

# Objetivo final

Obtener una tabla resumen que te permita saber:

| Categoría | Total de Ventas | Número de Ventas |
|:---------:|:---------------:|:----------------:|
| Ropa      | (suma)          | (conteo)          |
| Accesorios| (suma)          | (conteo)          |
| Calzado   | (suma)          | (conteo)          |

*(Los valores deberán calcularse como parte del ejercicio.)*

---

# Resultado esperado

Aplicar correctamente transformaciones avanzadas sobre un DataFrame en PySpark.

---
