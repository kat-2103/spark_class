# 🚗 Proyecto ETL: Análisis de Viajes en Taxi de Nueva York con PySpark

## 📌 Introducción

Este proyecto tiene como objetivo realizar un proceso completo de ETL (Extracción, Transformación y Carga) usando PySpark sobre la plataforma Databricks. Se trabajan datos reales de taxis en Nueva York para analizarlos y visualizarlos de forma interactiva.

## 📂 1. Carga de Datos

Primero accedí a los archivos CSV almacenados en el volumen de trabajo de Databricks:

```python
ruta_volumen = "/Volumes/workspace/default/data_taxi"
archivos = dbutils.fs.ls(ruta_volumen)
archivos_csv = [archivo.path for archivo in archivos if archivo.path.endswith(".csv")]
df = spark.read.option("header", True).option("inferSchema", True).csv(ruta_volumen)

```

Con esto pude cargar todos los archivos disponibles en un solo DataFrame para trabajar de forma unificada.

## 🧹 2. Limpieza y Transformación de Datos

Transformé el DataFrame original para dejar solo los datos válidos:

```python
from pyspark.sql.functions import to_timestamp, hour, col

df_clean = (
    df.withColumn("pickup_datetime", to_timestamp(col("tpep_pickup_datetime")))
      .withColumn("dropoff_datetime", to_timestamp(col("tpep_dropoff_datetime")))
      .filter(col("pickup_datetime").isNotNull())
      .filter(col("dropoff_datetime").isNotNull())
      .filter(col("passenger_count") > 0)
      .filter(col("trip_distance") > 0)
      .filter(col("fare_amount") >= 0)
      .withColumn("pickup_hour", hour(col("pickup_datetime")))
)

df_clean = df_clean.drop("airport_fee", "store_and_fwd_flag")
```

Esto me permitió:

Convertir las fechas a formato timestamp

Eliminar registros con valores erróneos o vacíos

Crear una columna para la hora de recogida

Quitar columnas innecesarias

## ❌ 3. Tratamiento de Valores Nulos

Para asegurarme de trabajar con datos completos, eliminé los registros con valores nulos en las columnas clave:

from pyspark.sql.functions import to_timestamp, hour, col
```python
# Limpieza y transformación
df_clean = (
    df.withColumn("pickup_datetime", to_timestamp(col("tpep_pickup_datetime")))
    .withColumn("dropoff_datetime", to_timestamp(col("tpep_dropoff_datetime")))
    .filter(col("pickup_datetime").isNotNull())
    .filter(col("dropoff_datetime").isNotNull())
    .filter(col("passenger_count") > 0)
    .filter(col("trip_distance") > 0)
    .filter(col("fare_amount") >= 0)
    .withColumn("pickup_hour", hour(col("pickup_datetime")))
)

# Ver algunos registros limpios
display(
    df_clean.select(
        "pickup_datetime",
        "pickup_hour",
        "passenger_count",
        "trip_distance",
        "fare_amount",
    )
)
```
## 📊 4. Agregación de Datos por Hora

Agrupé los viajes por hora para obtener métricas importantes del negocio:
```python
agg_df = (
    df_clean.groupBy("pickup_hour")
    .agg({
        "fare_amount": "avg",
        "trip_distance": "avg",
        "passenger_count": "sum"
    })
    .withColumnRenamed("avg(fare_amount)", "avg_fare")
    .withColumnRenamed("avg(trip_distance)", "avg_distance")
    .withColumnRenamed("sum(passenger_count)", "total_passengers")
)
```
Esto me permitió analizar qué horas concentran más actividad, cuándo los trayectos son más largos o cuándo se generan mayores ingresos.

## 📀 5. Almacenamiento en Delta Lake

Para almacenar los resultados de forma eficiente, utilicé formato Delta:
```python
df_clean.write.format("delta").mode("overwrite").saveAsTable("df_clean_delta")
agg_df.write.format("delta").mode("overwrite").saveAsTable("agg_df_delta")
```

## 📈 6. Visualización de Datos en Dashboards

Creé dashboards con las siguientes visualizaciones:

**📆 a. Total de pasajeros por hora**

```python
from pyspark.sql.functions import hour

df_con_hora = df.withColumn("hora", hour("tpep_pickup_datetime"))
df_total_pasajeros = df_con_hora.groupBy("hora").count().alias("total_pasajeros")
display(df_total_pasajeros)
```

**💸 b. Tarifa promedio por hora**

```python
from pyspark.sql.functions import avg

df_promedio_tarifa = df_con_hora.groupBy("hora").agg(avg("total_amount").alias("promedio_tarifa"))
display(df_promedio_tarifa)
```

**🌍 c. Distancia media por hora**

```python
df_distancia_media = df_con_hora.groupBy("hora").agg(avg("trip_distance").alias("distancia_media"))
display(df_distancia_media)
```

## 📊 Conclusión

Este proyecto me ha ayudado a aprender a usar PySpark para analizar datos reales en Databricks. He practicado el ciclo completo de ETL y he creado dashboards con KPIs que podrían servir para tomar decisiones en una empresa de taxis. Aunque algunas partes han sido desafiantes, he aprendido un montón y me siento mucho más segura trabajando con datos ✨💻
