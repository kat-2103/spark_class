# 📂 15 - Funciones Comunes en Azure Synapse Analytics (PySpark)

---

# Objetivo

Tener una referencia rápida de las funciones **más comunes** que usarás al trabajar con **PySpark** en **Azure Synapse Analytics**.

---

# Funciones de PySpark más usadas en Synapse

| Función | Uso | Ejemplo |
|:---|:---|:---|
| `read.format()` | Leer datos de diferentes formatos (Parquet, CSV, JSON) | `spark.read.format("parquet").load("ruta")` |
| `write.format()` | Guardar datos en diferentes formatos | `df.write.format("parquet").save("ruta")` |
| `select()` | Seleccionar columnas específicas | `df.select("columna1", "columna2")` |
| `filter()` | Filtrar filas basadas en condiciones | `df.filter(df.edad > 30)` |
| `withColumn()` | Crear o modificar columnas | `df.withColumn("nueva_col", df.edad + 5)` |
| `groupBy().agg()` | Agrupar datos y aplicar funciones de agregación | `df.groupBy("ciudad").agg(sum("ventas"))` |
| `orderBy()` | Ordenar los datos por una o más columnas | `df.orderBy("edad")` |
| `drop()` | Eliminar columnas | `df.drop("columna_a_eliminar")` |
| `dropDuplicates()` | Eliminar filas duplicadas | `df.dropDuplicates(["columna1"])` |
| `distinct()` | Eliminar duplicados en todo el DataFrame | `df.distinct()` |
| `limit()` | Limitar el número de filas | `df.limit(10)` |
| `cache()` | Cachear un DataFrame en memoria para acelerar procesos repetidos | `df.cache()` |
| `count()` | Contar el número de filas | `df.count()` |
| `show()` | Mostrar resultados en pantalla | `df.show()` |
| `printSchema()` | Mostrar la estructura del DataFrame | `df.printSchema()` |


---

# Funciones útiles de `pyspark.sql.functions`

```python
from pyspark.sql.functions import col, sum, avg, count, max, min, when, lit
```

| Función | Uso |
|:---|:---|
| `col("nombre_columna")` | Referenciar columnas de manera segura. |
| `sum("columna")` | Sumar valores de una columna. |
| `avg("columna")` | Calcular promedio. |
| `count("columna")` | Contar valores. |
| `max("columna")` | Obtener el valor máximo. |
| `min("columna")` | Obtener el valor mínimo. |
| `when(condición, valor).otherwise(valor)` | Equivalente a `IF-THEN-ELSE`. |
| `lit(valor)` | Crear una columna con un valor literal. |

---

# Notas importantes para Synapse

- Siempre adjunta el **Notebook** a un **Spark Pool** antes de ejecutar código.
- Si trabajas sobre un **Data Lake**, usa rutas tipo `"abfss://<filesystem>@<storageaccount>.dfs.core.windows.net/ruta"`.
- Cuando guardes datos, **usa `overwrite` o `append`** explícitamente para controlar cómo se escriben:

```python
df.write.mode("overwrite").parquet("ruta")
```

- **Evita usar `collect()`** sobre datasets muy grandes (puede saturar la memoria del driver).

---

# Consejo final

> *"Conocer estas funciones te permitirá construir pipelines de datos eficientes en PySpark dentro de Synapse Analytics."*
