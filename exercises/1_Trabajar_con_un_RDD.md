# 🧪 Ejercicio Mini: Trabajar con un RDD en PySpark

## 📋 Objetivo del ejercicio

- Crear un **RDD** a partir de una lista de números.
- Realizar una **transformación** para obtener los números al cuadrado.
- Ejecutar una **acción** para recolectar y mostrar los resultados.

---

## 💠 Código paso a paso

### 1. Iniciar una **SparkSession**

```python
from pyspark.sql import SparkSession

# Crear una sesión de Spark
spark = SparkSession.builder \
    .appName("EjercicioRDD") \
    .getOrCreate()

# Acceder al contexto de Spark (SparkContext)
sc = spark.sparkContext
```

---

### 2. **Crear un RDD** a partir de una lista

```python
# Crear un RDD con números del 1 al 5
numeros = sc.parallelize([1, 2, 3, 4, 5])
```

> 🔸 Usamos `parallelize()` para crear un RDD desde una lista local.

---

### 3. **Transformar** el RDD (mapear los números al cuadrado)

```python
# Crear un nuevo RDD transformado: números al cuadrado
numeros_cuadrado = numeros.map(lambda x: x * x)
```

> 🔸 `map()` es una transformación: aplica una función a cada elemento del RDD.

---

### 4. **Ejecutar una Acción** para recolectar los resultados

```python
# Acción: recolectar los datos y mostrarlos
resultado = numeros_cuadrado.collect()

print("Números al cuadrado:", resultado)
```

> 🔸 `collect()` es una acción: trae los datos desde el clúster a tu programa local.

---

### 5. **Cerrar la sesión de Spark**

```python
# Cerrar SparkSession
spark.stop()
```

---

# 🌟 Resultado esperado

Cuando ejecutas todo el script deberías ver en la salida:

```
Números al cuadrado: [1, 4, 9, 16, 25]
```

---

# 🧐 Comentarios importantes

- **parallelize()**: Crea un RDD a partir de una lista local.
- **map()**: Es una transformación; no ejecuta nada hasta que haya una acción.
- **collect()**: Ejecuta las transformaciones y devuelve los datos.
- **SparkSession y SparkContext**: Siempre necesarios para trabajar en PySpark.

---

# 💪 Resumen visual del flujo

```
Crear RDD → Transformarlo (map) → Ejecutar Acción (collect)
