# Carga de datos

Los datos están separados por tipo de vehículos, los cuales tienen distintas columnas así que los he tratado por separado.

```
yellow_files = []
green_files = []
fhv_files = []
fhvhv_files = []

for file in os.listdir(path_volume):
    if "yellow" in file:
        yellow_files.append(os.path.join(path_volume, file))
    elif "green" in file:
        green_files.append(os.path.join(path_volume, file))
    elif "fhv" in file and "fhvhv" not in file:
        fhv_files.append(os.path.join(path_volume, file))
    elif "fhvhv" in file:
        fhvhv_files.append(os.path.join(path_volume, file))</code>
```

## Estandarización de los datos

Para agrupar en un único dataframe los ficheros de cada vehículo primero hay que definir un esquema con los tipos ya que un merge provocará conflictos porque algunos ficheros guardan columnas con un tipo distinto.

```
def standardize_yellow_schema(df):
    return df.selectExpr([
        "CAST(VendorID AS BIGINT) AS VendorID",
        "tpep_pickup_datetime",
        "tpep_dropoff_datetime",
        "CAST(passenger_count AS DOUBLE) AS passenger_count",
        "trip_distance",
        "CAST(RatecodeID AS DOUBLE) AS RatecodeID",
        "store_and_fwd_flag",
        "CAST(PULocationID AS BIGINT) AS PULocationID",
        "CAST(DOLocationID AS BIGINT) AS DOLocationID",
        "CAST(payment_type AS BIGINT) AS payment_type",
        "fare_amount", "extra", "mta_tax",
        "tip_amount", "tolls_amount",
        "improvement_surcharge", "total_amount",
        "congestion_surcharge",
        "COALESCE(airport_fee, Airport_fee) AS airport_fee"
    ])
```

# Limpieza de datos

Tras visualizar los datos he comprobado que están bien formateados así que no hay necesidad evaluar valores erróneos. Sin embargos hay columnas que tienen todos los valores nulos. La función get_null_summary me muestra un porcentaje de valores nulos y efectivamente compruebo que hay dataframes con valores nulos.

```
+---------------------+----------+----------+--------+
|column               |null_count|total_rows|null_pct|
+---------------------+----------+----------+--------+
|ehail_fee            |4517720   |4517720   |100.0   |
|trip_type            |1073648   |4517720   |23.77   |
|store_and_fwd_flag   |1073536   |4517720   |23.76   |
|RatecodeID           |1073536   |4517720   |23.76   |
|passenger_count      |1073536   |4517720   |23.76   |
|payment_type         |1073536   |4517720   |23.76   |
|congestion_surcharge |1073536   |4517720   |23.76   |
|VendorID             |0         |4517720   |0.0     |
```

Asumiré que los valores nulos son datos que no se han registrado pero no necesariamente el resto del registro es inválido. Mantendré los registros nulos debido a la cantidad de ellos, sin embargo voy a borrar las columnas en las que el 100% de sus valores son nulos

```
green_df = green_df.drop("ehail_fee")
fhv_df = fhv_df.drop("SR_Flag")
```

# Agregación

Por cada dataset he extraído columnas de hora y mes con el motivo de poder analizar temporalmente los viajes.

```
yellow_df = yellow_df.withColumn("month", month("tpep_pickup_datetime")).withColumn("hour", hour("tpep_pickup_datetime"))
green_df = green_df.withColumn("month", month("lpep_pickup_datetime")).withColumn("hour", hour("lpep_pickup_datetime"))
fhv_df = fhv_df.withColumn("month", month("pickup_datetime")).withColumn("hour", hour("pickup_datetime"))
fhvhv_df = fhvhv_df.withColumn("month", month("request_datetime")).withColumn("hour", hour("request_datetime"))
```

# Preparación para dashboard

Creo un dataframe para analizar los datos por año y mes, y otro para analizarlos por horas. Para ello tengo que unificar los dataframes de cada vehículo.

## Extracción de datos por mes y año
```
def agregar_por_mes(df, col_fecha, col_ganancia):
    if isinstance(col_ganancia, str):
        ganancia_col = col(col_ganancia)
    else:
        ganancia_col = col_ganancia

    return df.withColumn("año", year(col(col_fecha))) \
             .withColumn("mes", month(col(col_fecha))) \
             .groupBy("año", "mes") \
             .agg(
                 spark_sum(ganancia_col).alias("ganancia_total"),
                 count("*").alias("numero_viajes")
             ) \
             .orderBy("año", "mes")
```

## Extracción de datos por hora
```
def agregar_horario(df, col_fecha, col_ganancia):
    return df.withColumn("hora", hour(col(col_fecha))) \
             .groupBy("hora") \
             .agg(
                 spark_sum(col_ganancia).alias("ganancia_total"),
                 count("*").alias("numero_viajes")
             )
```

Creación de dataframes por vehículo
```
# Para Yellow
mensual_yellow = agregar_por_mes(yellow_df, "tpep_pickup_datetime", "total_amount")
horario_yellow = agregar_horario(yellow_df, "tpep_pickup_datetime", "total_amount")

# Para Green
mensual_green = agregar_por_mes(green_df, "lpep_pickup_datetime", "total_amount")
horario_green = agregar_horario(green_df, "lpep_pickup_datetime", "total_amount")

# Para FHV (sin ganancias, solo viajes)
mensual_fhv = agregar_por_mes(fhv_df, "pickup_datetime", functions.lit(0))
horario_fhv = agregar_horario(fhv_df, "pickup_datetime", functions.lit(0))


# Para FHvhv (usando driver_pay como ganancias)
mensual_fhvh = agregar_por_mes(fhvh_df, "pickup_datetime", "driver_pay")
horario_fhvh = agregar_horario(fhvh_df, "pickup_datetime", "driver_pay")
```
Unificación de dataframes

```
# Unir todos los DataFrames mensuales
mensual_df = mensual_yellow.withColumn("source", F.lit("yellow")) \
    .union(mensual_green.withColumn("source", F.lit("green"))) \
    .union(mensual_fhv.withColumn("source", F.lit("fhv"))) \
    .union(mensual_fhvh.withColumn("source", F.lit("fhvh")))

horario_df = horario_yellow.withColumn("source", F.lit("yellow")) \
    .union(horario_green.withColumn("source", F.lit("green"))) \
    .union(horario_fhv.withColumn("source", F.lit("fhv"))) \
    .union(horario_fhvh.withColumn("source", F.lit("fhvh")))
```

## Guardado de tablas para visualización

```
mensual_df.write.mode("overwrite").saveAsTable("tabla_metricas_mensuales")
horario_df.write.mode("overwrite").saveAsTable("tabla_metricas_horarias")
```
