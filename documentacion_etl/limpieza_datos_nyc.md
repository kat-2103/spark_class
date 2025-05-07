# Limpieza de Datos con PySpark en Databricks - NYC Taxi Dataset

Este documento describe el proceso de limpieza de datos aplicado al dataset de taxis de la ciudad de Nueva York, utilizando PySpark dentro de un entorno Databricks.

## 1. Preparación del Entorno

Se instalaron las dependencias necesarias para el procesamiento, aunque en este caso solo se requería `beautifulsoup4` (posiblemente para un paso posterior no incluido).

```python
%pip install beautifulsoup4
```

## 2. Exploración Inicial del Volumen de Datos

Se validó la existencia de los datos en el volumen montado:

```python
display(dbutils.fs.ls('dbfs:/Volumes/workspace/default/nyc_taxis/'))
```

## 3. Carga del Dataset

Se cargó el archivo CSV utilizando `spark.read.option(...).csv(...)`, aplicando opciones para inferencia de esquema y uso de cabeceras.

```python
df = spark.read \
    .option("inferSchema", "true") \
    .option("header", "true") \
    .csv("dbfs:/Volumes/workspace/default/nyc_taxis/green_tripdata_2020-10.csv")
```

## 4. Revisión del Esquema y Datos Nulos

Se examinó el esquema y se identificaron columnas con valores nulos mediante:

```python
df.printSchema()
df.select([F.count(F.when(F.col(c).isNull(), c)).alias(c) for c in df.columns]).show()
```

## 5. Renombrado de Columnas

Se renombraron columnas para estandarizar el formato a `snake_case` y facilitar la manipulación posterior:

```python
df_clean = df.withColumnRenamed("VendorID", "vendor_id") \
             .withColumnRenamed("lpepPickupDatetime", "pickup_datetime") \
             .withColumnRenamed("lpepDropoffDatetime", "dropoff_datetime")
```

## 6. Eliminación de Registros Inválidos

Se aplicaron filtros para limpiar los registros con datos incorrectos o no plausibles:

- Fechas futuras o inconsistentes.
- Coordenadas geográficas fuera de los rangos esperados.
- Valores negativos o nulos en campos de cantidad o precio.

Ejemplo:

```python
df_clean = df_clean.filter(
    (F.col("pickup_datetime") < F.col("dropoff_datetime")) &
    (F.col("trip_distance") > 0) &
    (F.col("fare_amount") > 0)
)
```

## 7. Conversión de Tipos

Se aseguraron los tipos adecuados para columnas de fecha y numéricas:

```python
df_clean = df_clean.withColumn("pickup_datetime", F.to_timestamp("pickup_datetime")) \
                   .withColumn("dropoff_datetime", F.to_timestamp("dropoff_datetime"))
```

## 8. Resultados Finales

Se verificó la cantidad de registros tras la limpieza:

```python
print(f"Registros limpios: {df_clean.count()}")
```

---

## Conclusión

La limpieza de datos en este pipeline incluyó validación de integridad, estandarización de nombres de columnas, tratamiento de valores nulos y eliminación de registros erróneos. Este conjunto de datos limpio puede usarse para análisis exploratorio, visualización o entrenamiento de modelos de machine learning.
