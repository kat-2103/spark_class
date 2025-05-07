# Documentación del Procesamiento de Datos de Taxis en Databricks

## Introducción

Este documento describe el proceso de limpieza, transformación y almacenamiento de datos de taxis en Databricks utilizando Delta Lake. El proyecto consistió en cargar datos de taxis desde archivos CSV, limpiarlos y transformarlos, y finalmente guardarlos en formato Delta para su posterior análisis.

## 1. Carga de Datos

Primero, cargué los datos desde el volumen de Databricks que contenía los archivos CSV de taxis:

```python
# Definir la ruta del volumen específico donde están los archivos
ruta_volumen = "/Volumes/workspace/default/data_taxi"

# Listar y filtrar los archivos CSV
archivos = dbutils.fs.ls(ruta_volumen)
archivos_csv = [archivo.path for archivo in archivos if archivo.path.endswith(".csv")]

# Cargar todos los archivos CSV en un DataFrame
df_taxis = spark.read.option("header", True).option("inferSchema", True).csv(ruta_volumen)
```

## 2. Limpieza de Datos

Se realizó un proceso de limpieza para eliminar registros nulos, corregir tipos de datos y filtrar valores inválidos, asegurando así la calidad de la información:

```python
from pyspark.sql.functions import col

# Eliminar filas con valores nulos en columnas clave
df_clean = df.dropna(subset=["tpep_pickup_datetime", "tpep_dropoff_datetime", "fare_amount"])

# Filtrar registros con valores negativos o inconsistentes
df_clean = df_clean.filter(
    (col("fare_amount") > 0) & 
    (col("trip_distance") > 0) & 
    (col("passenger_count") > 0)
)
```

## 3. Transformación de Datos

Se agregaron columnas adicionales para facilitar el análisis posterior, como la hora de recogida y la duración del viaje:

```python
from pyspark.sql.functions import hour, unix_timestamp

# Extraer la hora de recogida
df_transformed = df_clean.withColumn("pickup_hour", hour(col("tpep_pickup_datetime")))

# Calcular duración del viaje en minutos
df_transformed = df_transformed.withColumn(
    "trip_duration_min",
    (unix_timestamp("tpep_dropoff_datetime") - unix_timestamp("tpep_pickup_datetime")) / 60
)
```

## 4. Almacenamiento en Formato Delta

Finalmente, los datos procesados se almacenaron en formato Delta Lake, lo que permite un manejo eficiente y consultas optimizadas:

```python
# Guardar el DataFrame transformado en formato Delta
df_taxis_limpio.write.format("delta").mode("overwrite").saveAsTable("df_limpio_delta")
```

## 5. Análisis Visual Interactivo

Con el objetivo de generar un dashboard empresarial en Databricks, se realizaron tres análisis clave que permiten evaluar el comportamiento de los viajes en taxi a lo largo del día:

### 5.1 Promedio de Tarifa por Hora del Día

Se agruparon los datos por la hora de recogida (`pickup_hour`) y se calculó el promedio del monto de la tarifa (`fare_amount`), lo que permite visualizar en qué horas del día se registran tarifas más altas o bajas.

```python
from pyspark.sql.functions import avg

df_avg_fare = df_transformed.groupBy("pickup_hour").agg(
    avg("fare_amount").alias("avg_fare")
).orderBy("pickup_hour")
```

### 5.2 Total de Pasajeros por Hora

Se sumó la cantidad de pasajeros por hora para observar los momentos del día con mayor volumen de pasajeros transportados.

```python
from pyspark.sql.functions import sum

df_total_passengers = df_transformed.groupBy("pickup_hour").agg(
    sum("passenger_count").alias("total_passengers")
).orderBy("pickup_hour")
```

### 5.3 Distancia Media Recorrida por Hora

Se calculó la distancia promedio recorrida por hora, utilizando la columna `trip_distance`. Esto permite detectar cuándo se realizan trayectos más largos o más cortos.

```python
df_avg_distance = df_transformed.groupBy("pickup_hour").agg(
    avg("trip_distance").alias("avg_distance")
).orderBy("pickup_hour")
```

Estas tres métricas fueron visualizadas usando gráficos de líneas y barras dentro de Databricks, lo que permitió construir un **dashboard interactivo** que facilita la toma de decisiones operativas y estratégicas.

## 6.Conclusion
Este proceso de limpieza, transformación y análisis de los datos de taxis en Databricks ha creado un flujo de trabajo eficiente que garantiza la calidad de los datos y proporciona información útil para tomar decisiones. Al almacenar los datos en formato Delta Lake, se mejora la gestión y se pueden realizar consultas de manera más rápida.

El análisis de métricas como el promedio de tarifa por hora, el total de pasajeros por hora y la distancia recorrida por hora permite entender cómo varían los viajes a lo largo del día. Esto ayuda a identificar los momentos de mayor o menor demanda y ajustar las operaciones de los taxis.

El dashboard interactivo en Databricks facilita ver estos resultados y ayuda a tomar decisiones de forma más informada. Este tipo de herramientas son clave para optimizar el funcionamiento de los taxis y mejorar la experiencia de los pasajeros y conductores.

En resumen, este proceso ha mejorado el análisis y la toma de decisiones en tiempo real, sentando una buena base para seguir mejorando la gestión de los datos y las operaciones de taxis.
