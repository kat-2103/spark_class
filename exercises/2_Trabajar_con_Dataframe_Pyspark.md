# 🧪 Ejercicio Mini: Trabajar con un DataFrame en PySpark

## 📋 Objetivo del ejercicio

- Crear un **DataFrame** en PySpark a partir de una lista de diccionarios.
- Realizar **transformaciones** para seleccionar y filtrar datos.
- Ejecutar **acciones** para visualizar resultados.

---

## 💠 Código paso a paso

### 1. Iniciar una **SparkSession**

```python
from pyspark.sql import SparkSession

# Crear una sesión de Spark
spark = SparkSession.builder \
    .appName("EjercicioDataFrame") \
    .getOrCreate()
```

---

### 2. **Crear un DataFrame** a partir de una lista de diccionarios

```python
datos = [
    {"nombre": "Ana", "edad": 30},
    {"nombre": "Luis", "edad": 25},
    {"nombre": "Carlos", "edad": 35},
    {"nombre": "Laura", "edad": 28}
]

# Crear DataFrame
df = spark.createDataFrame(datos)
```

> 🔸 `createDataFrame()` transforma una lista de diccionarios en un DataFrame.

---

### 3. **Transformar** el DataFrame (seleccionar y filtrar)

```python
# Seleccionar solo la columna "nombre"
df_nombres = df.select("nombre")

# Filtrar personas mayores de 28 años
df_filtrado = df.filter(df["edad"] > 28)
```

> 🔸 `select()` y `filter()` son transformaciones: no ejecutan inmediatamente.

---

### 4. **Ejecutar acciones** para ver resultados

```python
# Mostrar todos los nombres
df_nombres.show()

# Mostrar personas mayores de 28 años
df_filtrado.show()
```

> 🔸 `show()` es una acción: ejecuta las transformaciones y muestra resultados en pantalla.

---

### 5. **Cerrar la sesión de Spark**

```python
# Cerrar SparkSession
spark.stop()
```

---

# 🌟 Resultado esperado

Primera salida (`df_nombres.show()`):

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

Segunda salida (`df_filtrado.show()`):

```
+-------+----+
| nombre|edad|
+-------+----+
|    Ana|  30|
| Carlos|  35|
+-------+----+
```

---

# 🧐 Comentarios importantes

- **createDataFrame()**: crea DataFrames a partir de listas o estructuras.
- **select()**: elige columnas.
- **filter()**: filtra filas según condiciones.
- **show()**: acción para visualizar resultados.

---

# 💪 Resumen visual del flujo

```
Crear DataFrame → Transformarlo (select, filter) → Ejecutar Acción (show)
