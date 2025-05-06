# Documentación del Proyecto

## Parte 1: Preparación del Entorno

Primero, realizamos la importación de las librerías necesarias:

```python
import os
import glob

from pyspark.dbutils import DBUtils

from pyspark.sql import SparkSession
from pyspark.sql import DataFrame
from pyspark.sql import functions as F
from pyspark.sql.types import *
from pyspark.sql.functions import col, sum as _sum, when, to_timestamp, hour, col, to_timestamp, hour, desc, count


import pandas as pd
import numpy as np

from functools import reduce
```

## Parte 2: Ingesta de Datos
A continuación hacemos la importación y la carga de arhivos

```python

    # Definimos la ruta donde están los archivos
    ruta_volumen = "/Volumes/workspace/default/data-taxis"
    
    # Listamos los arhivos
    archivos = dbutils.fs.ls(ruta_volumen)

    # Cogemos los arhivos csv en este caso
    archivos_csv = [archivo.path for archivo in archivos if archivo.path.endswith('.csv')]

    # Mostrar los archivos encontrados (en este caso los csv)
    print(f"Encontrados {len(archivos_csv)} archivos csv's:")
    for i, archivo in enumerate(archivos_csv[:5], 1): 
        print(f"{i}. {archivo}")
    if len(archivos_csv) > 5:
        print(f"... y {len(archivos_csv) - 5} archivos más")

    # Leemos los archivos csv en un Dataframe
    if archivos_csv:
        df_taxis = spark.read.parquet(*archivos_csv)  # El asterisco * desempaqueta la lista de rutas
        
    # Leer todos los archivos csv en un solo DataFrame
    if archivos_csv:
        df = spark.read.option("header", True).option("inferSchema", True).csv(ruta_volumen)
        
        # Mostramos la info del DataFrame
        print(f"\nDataFrame cargado con {df.count()} registros")
        print("\nEsquema del DataFrame:")
        df.printSchema()
        
        print("\nPrimeras 5 filas:")
        display(df.limit(5))
    else:
        # Mostrar esta linea en caso de error 
        print("No se encontraron archivos csv en el volumen.")
 ```       
## Parte 3: Transformación de Datos

Ahora despues de cargar los archivos hacemos la limpieza de datos

Primero vemos si hay nulos y cuantos hay

```python

  #Contamos los datos nulos
  null_counts = df.select([
      _sum(when(col(c).isNull(), 1).otherwise(0)).alias(c)
      for c in df.columns
  ])
  display(null_counts)

```
Limpiamos los datos 

```python

  df_clean = df \
    .withColumn("pickup_datetime", to_timestamp(col("tpep_pickup_datetime"))) \
    .withColumn("dropoff_datetime", to_timestamp(col("tpep_dropoff_datetime"))) \
    .filter(col("pickup_datetime").isNotNull()) \
    .filter(col("dropoff_datetime").isNotNull()) \
    .filter(col("passenger_count") > 0) \
    .filter(col("trip_distance") > 0) \
    .filter(col("fare_amount") >= 0) \
    .withColumn("pickup_hour", hour(col("pickup_datetime")))

# Vemos los datos limpios
display(df_clean.select("pickup_datetime", "pickup_hour", "passenger_count", "trip_distance", "fare_amount"))

```
Eliminamos los valores nulos

```python

  # Mostramos el recuento de los datos nulos por columnas
  print("Recuento de valores nulos por columna antes de la limpieza:")
  for columna in df.columns:
      count_nulos = df.filter(col(columna).isNull()).count()
      if count_nulos > 0:
          print(f"- {columna}: {count_nulos} valores nulos")
  
  # Contamos las filas antes de eliminar nada
  filas_antes = df.count()
  print(f"\nNúmero de filas antes de eliminar nulos: {filas_antes}")
  
  # Eliminamos las filas con valores nulos
  df = df.dropna()
  
  # Contamos las filas  despues de eliminar los nulos
  filas_despues = df.count()
  print(f"Número de filas después de eliminar nulos: {filas_despues}")
  print(f"Se eliminaron {filas_antes - filas_despues} filas ({((filas_antes - filas_despues)/filas_antes)*100:.2f}% del total)")
  
  # Comprobamos que ya no hay valores nulos
  nulos_restantes = sum([df_clean.filter(col(columna).isNull()).count() for columna in df_clean.columns])
  if nulos_restantes == 0:
      print("Todos los valores nulos han sido eliminados correctamente.")
  else:
      print(f"Advertencia: Aún quedan {nulos_restantes} valores nulos en el dataframe.")
  
  # Actualizamos el dataframe
  df = df_clean
  
  # 7. Mostrar las primeras filas del dataframe limpio
  # Mostramos las primeras 5 filas ahora que esta limpio
  print("\nPrimeras 5 filas del dataframe limpio:")
  display(df.limit(5))
```

## Parte 3: Transformación de Datos

Ahora transformamos los datos

```python

# Hacemos una limpieza basica
df_clean = df.withColumn("pickup_datetime", to_timestamp(col("tpep_pickup_datetime"))).withColumn("dropoff_datetime", to_timestamp(col("tpep_dropoff_datetime")))              .filter(col("pickup_datetime").isNotNull())              .filter(col("passenger_count") > 0)

# Insertamos otra columna conla hora de recogida
df_clean = df_clean.withColumn("pickup_hour", hour("pickup_datetime"))

```

##  Parte 4: Agregaciones

Ahora agregamos columnas necesarias

```python
  agg_df = df_clean.groupBy("pickup_hour").agg(
    {"fare_amount": "avg", 
     "trip_distance": "avg", 
     "passenger_count": "sum"}
).withColumnRenamed("avg(fare_amount)", 
                    "avg_fare").withColumnRenamed("avg(trip_distance)", 
                                                  "avg_distance")  .withColumnRenamed("sum(passenger_count)", 
                                                                                      "total_passengers")
```

##  Parte 5: Almacenamiento en Delta Lake

Hacemos el Delta Lake

```python

  df_clean.write.format("delta").mode("overwrite").saveAsTable("df_limpio_delta")

```

## Parte 6: Dashboards Interactivos

Hacemos la visualización de datos

Primero hacemos los imports

```python

  from pyspark.sql.functions import month, when, col, sum, avg

  from pyspark.sql.functions import unix_timestamp, hour, month, year, avg
  
```

Ahora les ponemos nombre a los meses

```python

  df_con_mes = df.withColumn("mes_num", month("tpep_pickup_datetime")) \
    .withColumn("mes_nombre", 
        when(col("mes_num") == 1, "1_Enero")
        .when(col("mes_num") == 2, "2_Febrero")
        .when(col("mes_num") == 3, "3_Marzo")
        .when(col("mes_num") == 4, "4_Abril")
        .when(col("mes_num") == 5, "5_Mayo")
        .when(col("mes_num") == 6, "6_Junio")
        .when(col("mes_num") == 7, "7_Julio")
        .when(col("mes_num") == 8, "8_Agosto")
        .when(col("mes_num") == 9, "9_Septiembre")
        .when(col("mes_num") == 10, "10_Octubre")
        .when(col("mes_num") == 11, "11_Noviembre")
        .when(col("mes_num") == 12, "12_Diciembre")
    )

```

Agrupamos los datos por el nombre de los meses

```python

  df_por_mes = df_con_mes.groupBy("mes_nombre").agg(
    sum("passenger_count").alias("total_passengers"),
    avg("trip_distance").alias("avg_distance")
    ).orderBy("mes_nombre")

```

Visualizamos Promedio de tarifa por hora del día.

```python

display(
    agg_df.select("pickup_hour", "total_passengers").orderBy("total_passengers", ascending=False)
)
  
```

Visualizamos el Total de pasajeros por hora.

```python
  
  def tarifa_promedio_por_hora(df):
      # Asegurar que la columna de fecha sea tipo timestamp
      df = df.withColumn("pickup_datetime", F.col("tpep_pickup_datetime").cast("timestamp"))
      
      # Extraer la hora y calcular promedio de tarifa
      df = df.withColumn("hora", F.hour("pickup_datetime"))
      resultado = df.groupBy("hora").agg(F.avg("fare_amount").alias("tarifa_promedio"))
      
      # Ordenar por tarifa promedio descendente
      return resultado.orderBy(F.col("tarifa_promedio").desc())
  
  # Ejemplo de uso:
  tarifa_prom = tarifa_promedio_por_hora(df)
  display(tarifa_prom)
  
```

Visualizamos la Distancia media recorrida por hora.

```python

# Agrupar por zona de recogida y calcular distancia media
df_distancia = df.groupBy("PULocationID").agg(avg("trip_distance").alias("distancia_media"))

# Ordenar de mayor a menor por distancia media
df_distancia = df_distancia.orderBy(desc("distancia_media"))

# Convertir PULocationID a string para mejorar la visualización
df_distancia = df_distancia.withColumn("PULocationID", col("PULocationID").cast("string"))

# Mostrar resultados
display(df_distancia)

```

## Conclusiones 

Este proceso ETL desarrollado con PySpark y respaldado por Delta Lake permite gestionar datos de forma eficiente dentro de un entorno Data Lake. Entre sus principales ventajas destacan:

✨ Carga de datos rápida y optimizada, incluso con grandes volúmenes

🧪 Transformaciones limpias y estructuradas, listas para ser analizadas

📊 Resultados visibles al instante, facilitando el análisis exploratorio

📚 Base robusta para futuras integraciones con soluciones de inteligencia de negocios (BI)

Autor:
Zayra Fernández Miramontes
