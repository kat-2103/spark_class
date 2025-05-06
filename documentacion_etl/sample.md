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
df = spark.read.option("header", True).option("inferSchema", True).csv(ruta_volumen)
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

# Convertir columnas de fecha/hora si es necesario
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
df_transformed.write.format("delta").mode("overwrite").save("/mnt/datalake/taxis_delta")
```

