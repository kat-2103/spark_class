# Documentación del Procesamiento de Datos de Taxis en Databricks

## Introducción

Este documento describe el proceso de ingesta, limpieza, transformación, almacenamiento y análisis de datos de taxis de la ciudad de Nueva York en Databricks utilizando Auto Loader y Delta Lake. El objetivo del proyecto fue crear un flujo ETL eficiente que garantice la calidad de los datos y permita su análisis en tiempo real mediante visualizaciones interactivas.

## 1. Ingesta de Datos con Auto Loader

Se utilizó Auto Loader de Databricks para detectar y cargar archivos CSV de forma incremental desde un volumen de almacenamiento:

```python
from pyspark.sql.types import *

# Definición del esquema
schema = StructType([
    StructField("VendorID", IntegerType(), True),
    StructField("tpep_pickup_datetime", StringType(), True),
    StructField("tpep_dropoff_datetime", StringType(), True),
    StructField("passenger_count", IntegerType(), True),
    StructField("trip_distance", DoubleType(), True),
    StructField("RatecodeID", IntegerType(), True),
    StructField("store_and_fwd_flag", StringType(), True),
    StructField("PULocationID", IntegerType(), True),
    StructField("DOLocationID", IntegerType(), True),
    StructField("payment_type", IntegerType(), True),
    StructField("fare_amount", DoubleType(), True),
    StructField("extra", DoubleType(), True),
    StructField("mta_tax", DoubleType(), True),
    StructField("tip_amount", DoubleType(), True),
    StructField("tolls_amount", DoubleType(), True),
    StructField("improvement_surcharge", DoubleType(), True),
    StructField("total_amount", DoubleType(), True),
    StructField("congestion_surcharge", DoubleType(), True)
])

# Ruta de los datos
ruta = "/Volumes/workspace/default/data_taxi"

# Carga incremental con Auto Loader
raw_df = spark.readStream.format("cloudFiles") \
    .option("cloudFiles.format", "csv") \
    .option("header", "true") \
    .schema(schema) \
    .load(ruta)
```

## 2. Limpieza de Datos

Se limpiaron los datos eliminando valores nulos, duplicados y registros inválidos:

```python
from pyspark.sql.functions import col, to_timestamp

# Conversión de columnas de fecha/hora
clean_df = raw_df.withColumn("tpep_pickup_datetime", to_timestamp("tpep_pickup_datetime")) \
                 .withColumn("tpep_dropoff_datetime", to_timestamp("tpep_dropoff_datetime"))

# Eliminación de duplicados y valores nulos en columnas clave
clean_df = clean_df.dropna(subset=["tpep_pickup_datetime", "tpep_dropoff_datetime", "fare_amount"])
clean_df = clean_df.dropDuplicates()

# Filtro de valores inválidos o inconsistentes
clean_df = clean_df.filter(
    (col("fare_amount") > 0) &
    (col("trip_distance") > 0) &
    (col("passenger_count") > 0)
)
```

## 3. Transformación de Datos

Se añadieron columnas para análisis posterior:

```python
from pyspark.sql.functions import hour, unix_timestamp

# Agregar columna con la hora de recogida
transformed_df = clean_df.withColumn("pickup_hour", hour("tpep_pickup_datetime"))

# Calcular duración del viaje en minutos
transformed_df = transformed_df.withColumn(
    "trip_duration_min",
    (unix_timestamp("tpep_dropoff_datetime") - unix_timestamp("tpep_pickup_datetime")) / 60
)
```

## 4. Almacenamiento en Delta Lake

Los datos procesados fueron almacenados como tabla Delta administrada dentro del metastore de Databricks:

```python
# Guardar como tabla Delta en modo sobreescritura
transformed_df.write.format("delta").mode("overwrite").saveAsTable("taxi_data_bronze")
```

## 5. Análisis Interactivo y Visualizaciones

Se realizaron varias visualizaciones clave en el notebook para generar un dashboard empresarial interactivo.

### 5.1 Promedio de Tarifa por Hora del Día

```python
from pyspark.sql.functions import avg

df_avg_fare = transformed_df.groupBy("pickup_hour").agg(
    avg("fare_amount").alias("avg_fare")
).orderBy("pickup_hour")
```

### 5.2 Total de Pasajeros por Hora

```python
from pyspark.sql.functions import sum

df_total_passengers = transformed_df.groupBy("pickup_hour").agg(
    sum("passenger_count").alias("total_passengers")
).orderBy("pickup_hour")
```

### 5.3 Distancia Media Recorrida por Hora

```python
df_avg_distance = transformed_df.groupBy("pickup_hour").agg(
    avg("trip_distance").alias("avg_distance")
).orderBy("pickup_hour")
```

Estas métricas fueron graficadas directamente en el notebook usando visualizaciones nativas de Databricks.

## 6. Conclusión

Este flujo de trabajo ETL utilizando Auto Loader, transformaciones en PySpark y almacenamiento en Delta Lake permite un manejo escalable y confiable de datos de taxis. Las visualizaciones interactivas habilitan el monitoreo en tiempo real y el soporte a la toma de decisiones operativas. El uso de tablas gestionadas mejora la integración con herramientas de BI y facilita el gobierno del dato dentro del ecosistema de Databricks.
