# 📂 02 - PySpark en Local: Primeros pasos con RDDs

---

# Objetivo

Crear, transformar y ejecutar acciones sobre un **RDD** en PySpark, trabajando **en local**.

---

# Conceptos clave

- **RDD (Resilient Distributed Dataset)**: estructura básica de Spark, una colección distribuida de objetos.
- **Transformaciones**: operaciones como `map()`, `filter()`, que generan nuevos RDDs.
- **Acciones**: operaciones como `collect()`, `count()`, que disparan la ejecución real.

---

#  Ejercicio paso a paso

## 1. Crear una SparkSession

```python
from pyspark.sql import SparkSession

# Crear la sesión de Spark
spark = SparkSession.builder \
    .appName("PrimerRDD") \
    .getOrCreate()

# Obtener el contexto de Spark
sc = spark.sparkContext
```

---

## 2. Crear un RDD a partir de una lista

```python
# Crear un RDD con números del 1 al 10
numeros = sc.parallelize(range(1, 11))
```

> 🔸 `parallelize()` crea un RDD a partir de una lista o un rango de datos.

---

## 3. Aplicar transformaciones

```python
# Transformación: calcular el cuadrado de cada número
numeros_cuadrado = numeros.map(lambda x: x * x)

# Transformación: filtrar solo los cuadrados mayores que 20
cuadrados_filtrados = numeros_cuadrado.filter(lambda x: x > 20)
```

> 🔸 `map()` y `filter()` son transformaciones: **no ejecutan nada todavía**.

---

## 4. Ejecutar acciones

```python
# Acción: recolectar y mostrar los resultados
resultado = cuadrados_filtrados.collect()

print("Cuadrados mayores que 20:", resultado)
```

> 🔸 `collect()` dispara la ejecución y devuelve los resultados.

---

## 5. Cerrar la sesión de Spark

```python
# Siempre cerrar la sesión cuando termines
spark.stop()
```

---

# 📊 Resultado esperado

```
Cuadrados mayores que 20: [25, 36, 49, 64, 81, 100]
```

---

# 📈 Flujo visual del ejercicio

```
Crear RDD → Transformar (map + filter) → Ejecutar acción (collect)
```

---

#  Comentarios importantes

- Las **transformaciones** son perezosas: no ejecutan inmediatamente.
- Las **acciones** fuerzan la ejecución real de las transformaciones acumuladas.
- Siempre **cierra la sesión** de Spark para liberar recursos.

---

#  Siguiente paso

- Trabajarás con **DataFrames**: estructuras más potentes y optimizadas para datos tabulares en PySpark.

---

#  ¡Siguiente archivo: `03_pyspark_local_dataframes.md`! 📂
