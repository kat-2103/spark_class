# 📂 03 - PySpark en Local: Primeros pasos con DataFrames

---

# Objetivo

Crear, transformar y ejecutar acciones sobre un **DataFrame** en PySpark, trabajando **en local**.

---

# Conceptos clave

- **DataFrame**: tabla de datos distribuida, organizada en columnas, similar a una tabla de base de datos o un DataFrame de Pandas.
- **Transformaciones**: operaciones como `select()`, `filter()`, `groupBy()`.
- **Acciones**: operaciones como `show()`, `count()`, `collect()`, que ejecutan el código realmente.

---

# Ejercicio paso a paso

## 1. Crear una SparkSession

```python
from pyspark.sql import SparkSession

# Crear la sesión de Spark
spark = SparkSession.builder \
    .appName("PrimerDataFrame") \
    .getOrCreate()
```

---

## 2. Crear un DataFrame a partir de una lista de diccionarios

```python
datos = [
    {"nombre": "Ana", "edad": 30},
    {"nombre": "Luis", "edad": 25},
    {"nombre": "Carlos", "edad": 35},
    {"nombre": "Laura", "edad": 28}
]

# Crear el DataFrame
df = spark.createDataFrame(datos)
```

> 🔸 `createDataFrame()` transforma una lista de Python en un DataFrame distribuido.

---

## 3. Aplicar transformaciones

```python
# Seleccionar solo la columna "nombre"
df_nombres = df.select("nombre")

# Filtrar personas mayores de 28 años
df_filtrado = df.filter(df["edad"] > 28)
```

> 🔸 `select()` y `filter()` son transformaciones: definen operaciones pero no ejecutan inmediatamente.

---

## 4. Ejecutar acciones

```python
# Mostrar todos los nombres
df_nombres.show()

# Mostrar personas mayores de 28 años
df_filtrado.show()
```

> 🔸 `show()` es una acción: ejecuta las transformaciones y muestra resultados.

---

## 5. Cerrar la sesión de Spark

```python
# Siempre cerrar la sesión cuando termines
spark.stop()
```

---

# 📊 Resultado esperado

**Salida de `df_nombres.show()`:**

```
+-------+
| nombre|
+-------+
|    Ana|
|   Luis|
| Carlos|
|  Laura|
+-------+
```

**Salida de `df_filtrado.show()`:**

```
+-------+----+
| nombre|edad|
+-------+----+
|    Ana|  30|
| Carlos|  35|
+-------+----+
```

---

# 📈 Flujo visual del ejercicio

```
Crear DataFrame → Transformar (select, filter) → Ejecutar acción (show)
```

---

# Comentarios importantes

- **DataFrames** son mucho más eficientes que trabajar directamente con RDDs.
- Permiten **optimizaciones automáticas** usando motores como Catalyst.
- Usar **DataFrames** es la forma moderna de trabajar con PySpark.

---

# Siguiente paso

- Configurar una **cuenta gratuita de Azure** para trabajar con Spark en la nube.

---

# ¡Siguiente archivo: `04_crear_cuenta_azure_free_tier.md`! 📂
