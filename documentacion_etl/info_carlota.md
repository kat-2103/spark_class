# 🚖 Análisis de Datos de Taxis Amarillos en NYC (2020–2024)

## 📌 Descripción del Proyecto

Este proyecto analiza los datos de los taxis amarillos en Nueva York entre los años 2020 y 2024. A través de ETL desarrollado con Apache Spark y visualizaciones en Lightdash, se estudian los patrones de uso, ingresos y comportamiento horario del servicio de taxis.

## 🗃️ Fuentes de Datos

Los datos provienen del portal oficial de taxis de NYC: [NYC Taxi & Limousine Commission](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

- Los archivos fueron guardados en **Google Drive** y posteriormente montados en un volumen manual llamado `test_volume` dentro del entorno de Databricks.

## ⚙️ Proceso ETL (Extract, Transform, Load)

El proceso se llevó a cabo en un notebook de Jupyter usando PySpark para manejar grandes volúmenes de datos.

### 1. Lectura de Datos

Inicialmente intenté automatizar la descarga de los archivos desde la web oficial de NYC Taxi y copiarlos a una carpeta temporal (`/tmp`) en el entorno de Databricks, pero me encontré con problemas de permisos.

Como solución alternativa:

- Subieron los archivos a [**Google Drive**](https://drive.google.com/file/d/1KH8jxRvZ9Z5AesszU1ZA0HHiA7RsxaWR/view).
- Desde allí, los transferí manualmente al volumen montado en Databricks llamado `test_volume`.
- Finalmente, los leí directamente desde este volumen usando PySpark y los almacené en un **DataFrame** para continuar con el proceso de transformación.
### 2. Limpieza y Manejo de Nulos

Durante la limpieza, se identificaron varias columnas con valores nulos:

- `passenger_count`
- `ratecodeID`
- `store_and_fwd_flag`
- `congestion_surcharge`
- `airport_fee`

Las decisiones fueron:

- ✅ **`passenger_count`**: Se filtraron los registros con valores inválidos (`passenger_count = 0`) y se eliminaron las filas con valores nulos.
- ❌ **Otras columnas**: Se eliminaron las columnas `ratecodeID`, `store_and_fwd_flag`, `congestion_surcharge` y `airport_fee` ya que no aportaban valor para el análisis planteado y contenían muchos nulos. Esta decisión también se basó en los requisitos y preguntas de negocio definidos.

Además, al realizar un `display()` para visualizar el número de pasajeros por **año, mes y hora**, detecté que existían registros fuera del rango de años 2020–2024. Para asegurar la coherencia temporal del análisis, **eliminé todos los registros correspondientes a años fuera de este intervalo.**

```python
df = df.filter(col("passenger_count") > 0)
columnas_a_eliminar = ["airport_fee", "congestion_surcharge", "RatecodeID", "store_and_fwd_flag"] 
df = df.drop(*columnas_a_eliminar)
````

### 3. Conversión de Timestamps

Se transformaron las columnas de tiempo:

- `tpep_pickup_datetime`
- `tpep_dropoff_datetime`

Estas se convirtieron a tipo `datetime` para permitir operaciones de agregación temporal.
```python
from pyspark.sql.functions import to_timestamp

df = df.withColumn("tpep_pickup_datetime", to_timestamp(df["tpep_pickup_datetime"], "yyyy-MM-dd HH:mm:ss"))
df = df.withColumn("tpep_dropoff_datetime", to_timestamp(df["tpep_dropoff_datetime"], "yyyy-MM-dd HH:mm:ss"))
```

### 4. Particionamiento de Datos

Para optimizar el rendimiento de las consultas, los datos se particionaron por:

- `hour` (hora de recogida): muchas de las consultas agregadas se hacen por hora.
- `year`, `month`, `day`: para permitir filtrado rápido por periodos de tiempo.

Esta decisión mejora significativamente los tiempos de respuesta en entornos distribuidos como Spark o cuando se consulta desde herramientas como Lightdash.
```python
from pyspark.sql.functions import hour

df_con_hora = df.withColumn("hour", hour("tpep_pickup_datetime"))

from pyspark.sql.functions import year, month, dayofmonth

df = df_con_hora \
    .withColumn("year", year("tpep_pickup_datetime")) \
    .withColumn("month", month("tpep_pickup_datetime")) \
    .withColumn("day", dayofmonth("tpep_pickup_datetime"))
```

## 📊 Visualización (Lightdash)

La visualización se desarrolló en Lightdash, conectando directamente con las tablas creadas en Databricks. Se construyeron dashboards interactivos con insights como:

- Pasajeros por hora y mes, año a año (2020–2024).
- Ingresos totales por franjas horarias.
- Distancia media y tarifa media según la hora del día.

Estas visualizaciones permiten detectar patrones de demanda y comportamiento del servicio, como las **horas pico**, o las franjas horarias que generan más ingresos.

## 🧪 Análisis Exploratorio

Durante el análisis, se realizaron varias agregaciones clave para entender los patrones de comportamiento de los viajes en NYC.

### 🔹 Horas con mayor número de pasajeros

```python
from pyspark.sql.functions import sum

hour_passengers = df.select("hour", "passenger_count") \
  .groupBy("hour") \
  .agg(sum("passenger_count").alias("total_passengers")) \
  .orderBy("total_passengers", ascending=False)

display(hour_passengers)
```

### 🔹 Horas con mayor número de pasajeros

En este análisis agrupamos por **hora del día** y sumamos la cantidad total de pasajeros para identificar las **horas pico con mayor demanda**.

```python
from pyspark.sql.functions import sum

hour_passengers = df.select("hour", "passenger_count") \
  .groupBy("hour") \
  .agg(sum("passenger_count").alias("total_passengers")) \
  .orderBy("total_passengers", ascending=False)

display(hour_passengers)
```
![Horas con mayor número de pasajeros 2020-2024](https://github.com/user-attachments/assets/31c0f2dd-ff1a-485f-a153-4b6d9b5bf6e0)


### 🔹  Momentos del día con viajes más largos o caros

Aquí calculamos la **distancia media** y la **tarifa media** por hora. Esto nos permite identificar qué horas concentran los **viajes más largos y costosos**.

```python
from pyspark.sql.functions import avg

df_largos_caros = df.select("hour", "trip_distance", "fare_amount") \
  .groupBy("hour") \
  .agg(
      avg("trip_distance").alias("avg_distance"),
      avg("fare_amount").alias("avg_fare")
  ) \
  .orderBy("avg_distance", ascending=False)

display(df_largos_caros)
```
![Tarifa media de cada hora](https://github.com/user-attachments/assets/0f99a173-a9eb-459a-b655-2ea5cc241e05)
![Distancia media por hora](https://github.com/user-attachments/assets/4ee82f73-edbf-4fd5-bb8b-1191d21b58fa)


### 🔹   Franjas horarias que generan más ingresos

Sumamos el **total de ingresos por hora** para detectar en qué momentos del día se genera **mayor rentabilidad para los taxistas**.


```python
from pyspark.sql.functions import sum

df_ingresos = df.select("hour", "fare_amount") \
  .groupBy("hour") \
  .agg(sum("fare_amount").alias("total_revenue")) \
  .orderBy("total_revenue", ascending=False)

display(df_ingresos)
```
![Franjas horarias que general más ingresos 2020-2024](https://github.com/user-attachments/assets/834a182a-d72b-4a80-afe5-436b7b9f1755)

