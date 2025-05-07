# Análisis de Datos de Taxis - Databricks

Este proyecto realiza un análisis exploratorio de datos de taxis utilizando Apache Spark en un entorno Databricks utilizando Delta Lake. El proyecto incluye carga de datos, limpieza, transformación, análisis y visualización de patrones en los viajes de taxis.


## Estructura de Datos

El dataset contiene información sobre viajes de taxis, incluyendo:

- Fechas y horas de recogida y entrega (`tpep_pickup_datetime`, `tpep_dropoff_datetime`)
- Número de pasajeros (`passenger_count`)
- Distancia del viaje (`trip_distance`)
- Información de tarifa (`fare_amount`, `total_amount`)
- Información adicional relacionada con el viaje

### 1. Carga de Datos

Los datos se cargan desde archivos CSV almacenados en un volumen de Databricks:

```python
# Definir la ruta del volumen donde están los archivos
ruta_volumen = "/Volumes/workspace/default/data-taxi"

# Leer todos los archivos CSV en un DataFrame
df_taxis = spark.read.option("header", True).option("inferSchema", True).csv(ruta_volumen)
```

### 2. Exploración y Limpieza de Datos

Se analizan los valores nulos y se realiza una limpieza exhaustiva de los datos:

```python
# Identificación de valores nulos
null_counts = df_taxis.select([
    _sum(when(col(c).isNull(), 1).otherwise(0)).alias(c)
    for c in df_taxis.columns
])

# Limpieza de datos
df_taxis_limpio = df_taxis.dropna()

# Transformaciones básicas
df_clean = df_taxis.withColumn("pickup_datetime", to_timestamp(col("tpep_pickup_datetime")))
                   .withColumn("dropoff_datetime", to_timestamp(col("tpep_dropoff_datetime")))
                   .filter(col("pickup_datetime").isNotNull())
                   .filter(col("passenger_count") > 0)
                   .filter(col("trip_distance") > 0)
                   .filter(col("fare_amount") >= 0)
                   .withColumn("pickup_hour", hour(col("pickup_datetime")))

# Eliminación de columnas
df_clean = df_clean.drop("airport_fee", "store_and_fwd_flag")
```
Transformaciones:

- Conversión de fechas
- Eliminación de nulos
- Filtrado de registros con valores inválidos
- Extracción de hora de recogida
- Eliminación de columnas no usadas


### 3. Transformación de Datos

Se crean nuevas columnas para facilitar el análisis:

```python
# Añadir columnas de mes (número y nombre)
df_con_mes = df_taxis.withColumn("mes_num", month("tpep_pickup_datetime"))
                     .withColumn("mes_nombre", 
                                 when(col("mes_num") == 1, "1_Enero")
                                 .when(col("mes_num") == 2, "2_Febrero")
                                 # ... y así sucesivamente para todos los meses
                     )

# Añadir columna de hora
df_con_hora = df_con_mes.withColumn("hora", hour("tpep_pickup_datetime"))
```

### 4. Análisis y Agregaciones

Se realizan diversas agregaciones para analizar patrones:

```python
# Análisis por hora
agg_df = df_clean.groupBy("pickup_hour").agg(
    {"fare_amount": "avg", "trip_distance": "avg", "passenger_count": "sum"}
)

# Promedio de tarifa por hora
df_promedio_tarifa = df_con_hora.groupBy("hora").agg(avg("total_amount").alias("promedio_tarifa"))

# Total de viajes por hora
df_total_pasajeros = df_con_hora.groupBy("hora").count().alias("total_pasajeros")

# Distancia media por hora
df_distancia_media = df_con_hora.groupBy("hora").agg(avg("trip_distance").alias("distancia_media"))
```

### 5. Persistencia de Datos

Los datos procesados se guardan como tablas Delta para su reutilización:

```python
# Guardar DataFrame limpio como tabla Delta
df_taxis_limpio.write.format("delta").mode("overwrite").saveAsTable("df_limpio_delta")

# Guardar DataFrame agregado como tabla Delta
agg_df.write.format("delta").mode("overwrite").saveAsTable("agg_df")
```

## Visualizaciones

El proyecto incluye visualizaciones para identificar patrones temporales en:

1. Tarifa promedio por hora del día
2. Volumen de viajes por hora del día
3. Distancia media de viaje por hora del día

## Análisis

El análisis permite descubrir patrones como:

- Horas pico con mayor demanda de taxis
- Variaciones en las tarifas según la hora del día
- Relación entre distancia de viaje y hora del día
- Patrones estacionales (por mes) en el uso de taxis
