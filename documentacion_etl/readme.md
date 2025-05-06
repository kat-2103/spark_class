## Introducción

El objetivo era cargar los datos, limpiarlos, transformarlos y guardarlos de forma optimizada para poder analizarlos mejor y crear visualizaciones.

## 1. Carga de Datos

Primero accedí a los archivos CSV desde el volumen asignado en Databricks. Para eso usé este código:

```python
ruta_volumen = "/Volumes/workspace/default/data_taxi"
archivos = dbutils.fs.ls(ruta_volumen)
archivos_csv = [archivo.path for archivo in archivos if archivo.path.endswith(".csv")]
df = spark.read.option("header", True).option("inferSchema", True).csv(ruta_volumen)
```

Con esto conseguí cargar todos los datos en un solo DataFrame para trabajar con ellos más fácilmente.

## 2. Limpieza y Transformación de Datos

A los datos les hice varias transformaciones para dejar sólo los registros que eran válidos:

```python
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

Con esto:

* Pasé las fechas a formato timestamp.
* Quité los viajes con valores raros o nulos.
* Creé una nueva columna con la hora para poder hacer gráficos por franjas horarias.
* Quité columnas que no eran importantes para el análisis.

## 3. Tratamiento de Valores Nulos

Para evitar que los nulos dieran problemas al analizar, conté cuántos había y los eliminé en las columnas que eran clave:

```python
columnas_clave = ["pickup_datetime", "dropoff_datetime", "fare_amount", "trip_distance"]
df_clean = df_clean.dropna(subset=columnas_clave)
```

Así me aseguro de que los datos importantes estén completos.

## 4. Creación de Agregados

Agrupé los datos por hora para ver patrones diarios:

```python
df_agregados = df_clean.groupBy("pickup_hour").agg(
    {"fare_amount": "avg", "trip_distance": "avg", "passenger_count": "sum"}
).withColumnRenamed("avg(fare_amount)", "avg_fare") \
 .withColumnRenamed("avg(trip_distance)", "avg_distance") \
 .withColumnRenamed("sum(passenger_count)", "total_passengers")
```

Esto me permite analizar cosas como a qué hora se hacen más viajes, cuándo se paga más, etc.

## 5. Almacenamiento en Delta Lake

Guardé los datos en Delta Lake para que no se pierdan y sean más fáciles de consultar:

```python
df_clean.write.format("delta").mode("overwrite").saveAsTable("df_clean_delta")
agg_df.write.format("delta").mode("overwrite").saveAsTable("agg_df_delta")
```

Delta ayuda porque guarda versiones, es más rápido y permite hacer cambios sin cargar todo otra vez.



## 7. Dashboards Interactivos

Para facilitar el acceso a los datos, creé una vista que resume los viajes por hora:

```python
# Calcular total de pasajeros por hora
df_total_pasajeros = df_con_hora.groupBy("hora").count().alias("total_pasajeros")

# Visualizar gráfico de barras
display(df_total_pasajeros)

#-----------------
# Calcular promedio de tarifa por hora
df_promedio_tarifa = df_con_hora.groupBy("hora").agg(avg("total_amount").alias("promedio_tarifa"))
# Visualizar gráfico de barras
display(df_promedio_tarifa)


#---------------------
# Calcular distancia media por hora
df_distancia_media = df_con_hora.groupBy("hora").agg(avg("trip_distance").alias("distancia_media"))

# Visualizar gráfico de barras
display(df_distancia_media)


""")
```


## Conclusiones

Gracias a este proyecto he aprendido a trabajar con PySpark en Databricks, a limpiar y transformar datos reales y a guardarlos en Delta Lake
Ha sido una práctica difícil para mi porque hay conceptos que no entiendo al 100% pero lo he hecho lo mejor que he podido. Aún así estoy aprendiendo mucho.
