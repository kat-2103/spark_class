# Documentación del Proceso ETL - Dataset NYC Taxi

## Introducción
Este proceso de ETL tiene como objetivo analizar y limpiar los datos de los taxis amarillos de Nueva York, asegurando que sean válidos y estén preparados para análisis posteriores.

## 1. Carga de Datos
Cargamos los datos desde la ruta especificada en el sistema de archivos distribuido.

```python
volumen_path = "dbfs:/Volumes/workspace/default/test_volume/"
df = spark.read.option("header", True).option("inferSchema", True).csv(volumen_path + "*.csv")
```

## 2. Inspección de Datos
Mostramos una muestra de los datos y verificamos la estructura del DataFrame.  
```python
display(df.limit(10))
df.printSchema()
``` 
Analizamos valores nulos en cada columna.   

```python
from pyspark.sql.functions import col, sum

total_rows = df.count()

null_df = df.select([
    sum(col(c).isNull().cast("int")).alias(c) for c in df.columns
]).toPandas().transpose().reset_index()

null_df.columns = ["Column", "Null_Count"]
null_df["Null_Percentage"] = (null_df["Null_Count"] / total_rows) * 100

display(null_df)

``` 
## 3. Tratamiento de Valores Nulos
Cada columna se trata con lógica específica:

- passenger_count: Se rellena con la moda.

- RatecodeID: Se asigna el valor 99 para indicar valores desconocidos.

- store_and_fwd_flag: Se rellena con "N" para viajes no almacenados.

- congestion_surcharge: Se utiliza la mediana para imputación.

- airport_fee: Se rellena con 0, pues la mayoría de los viajes no parten del aeropuerto.  

## 4. Filtrado de Datos Inválidos
Se eliminan registros con valores inválidos, asegurando datos confiables:

- Distancia y cantidad de pasajeros deben ser mayores a cero.

- IDs y códigos deben estar dentro de rangos permitidos.

- Montos monetarios deben ser positivos.
```python
df = df.filter((col("trip_distance") > 0) & (col("passenger_count") > 0))
df = df.filter(col("VendorID").isin([1, 2, 6, 7]))
df = df.filter(col("RatecodeID").isin([1, 2, 3, 4, 5, 6, 99]))

``` 

## 5. Conversión de Fechas y Nuevas Columnas
Las fechas se convierten a formato timestamp y se crean nuevas columnas para análisis temporal y de propinas.  
```python
from pyspark.sql.functions import col, to_timestamp, hour, dayofweek, month, when, datediff

df = df.withColumn("tpep_pickup_datetime", to_timestamp(col("tpep_pickup_datetime")))
df = df.withColumn("tpep_dropoff_datetime", to_timestamp(col("tpep_dropoff_datetime")))
df = df.withColumn("pickup_hour", hour(col("tpep_pickup_datetime")))
df = df.withColumn("pickup_day", dayofweek(col("tpep_pickup_datetime")))
df = df.withColumn("pickup_month", month(col("tpep_pickup_datetime")))
df = df.withColumn("trip_duration", datediff(col("tpep_dropoff_datetime"), col("tpep_pickup_datetime")))
df = df.withColumn("day_night_flag", when((col("pickup_hour") >= 6) & (col("pickup_hour") < 18), "Día").otherwise("Noche"))
df = df.withColumn("high_tip_flag", when(col("tip_amount") > (col("total_amount") * 0.2), 1).otherwise(0))
```

## 6. Agregaciones y Almacenamiento
Se agregan los datos por hora, día y mes para análisis estadísticos y se almacenan en Delta Lake.  
```python
agg_hour_df.write.format("delta").mode("append").saveAsTable("nyc_taxi_agg_hourly")
agg_day_df.write.format("delta").mode("append").option("mergeSchema", "true").saveAsTable("nyc_taxi_agg_daily")
agg_month_df.write.format("delta").mode("append").saveAsTable("nyc_taxi_agg_monthly")
``` 




