# 📂 11 - Ejercicios PySpark - Soluciones

---

# 📅 Instrucciones

- Estas son las soluciones sugeridas a los ejercicios planteados en `10_ejercicios_pyspark_enunciados.md`.
- Puedes ejecutarlas en local o en Azure Synapse Notebooks.

---

# 💠 Soluciones paso a paso

## Ejercicio 1: Crear un RDD

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("EjerciciosRDD").getOrCreate()
sc = spark.sparkContext

# Crear RDD
numeros = sc.parallelize(range(1, 21))

# Mostrar todos los elementos
print(numeros.collect())
```

---

## Ejercicio 2: Filtrar múltiplos de 3

```python
# Filtrar números múltiplos de 3
multiplos_de_3 = numeros.filter(lambda x: x % 3 == 0)

# Mostrar resultado
print(multiplos_de_3.collect())
```

---

## Ejercicio 3: Crear un DataFrame sencillo

```python
# Datos de ejemplo
datos = [
    ("Ana", "Madrid", 30),
    ("Luis", "Sevilla", 25),
    ("Carlos", "Valencia", 35),
    ("Laura", "Barcelona", 28),
    ("Jorge", "Madrid", 40)
]

# Definir columnas
columnas = ["Nombre", "Ciudad", "Edad"]

# Crear DataFrame
personas_df = spark.createDataFrame(datos, columnas)

# Mostrar contenido
personas_df.show()
```

---

## Ejercicio 4: Filtrar personas por ciudad

```python
# Filtrar personas de Madrid
personas_madrid = personas_df.filter(personas_df.Ciudad == "Madrid")

# Mostrar resultado
personas_madrid.show()
```

---

## Ejercicio 5: Agrupar personas por ciudad

```python
# Agrupar y contar personas por ciudad
conteo_ciudades = personas_df.groupBy("Ciudad").count()

# Mostrar resultado
conteo_ciudades.show()
```

---

# 📚 Recordatorio

- No olvides siempre cerrar la sesión de Spark si estás trabajando en local:

```python
spark.stop()
```

- Si estás trabajando en Synapse Notebooks, simplemente cierra la sesión del Notebook para liberar el Spark Pool.

---
