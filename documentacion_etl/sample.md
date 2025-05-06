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

Este paso me permitió cargar todos los archivos CSV disponibles en un único DataFrame para su procesamiento unificado.

## 2. Limpieza y Transformación de Datos

Después de cargar los datos, realicé varias transformaciones para limpiarlos:

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

# Eliminar columnas que no se usan
df_clean = df_clean.drop("airport_fee", "store_and_fwd_flag")
```

Las operaciones de limpieza incluyeron:
- Conversión de fechas a formato timestamp
- Eliminación de registros con fechas nulas
- Filtrado de registros con valores inválidos (pasajeros <= 0, distancia <= 0, tarifa < 0)
- Extracción de la hora de recogida para análisis temporal
- Eliminación de columnas innecesarias

## 3. Tratamiento de Valores Nulos

Para manejar los valores nulos, implementé un enfoque que me permitió comparar dos estrategias:
1. Eliminar nulos solo en columnas clave
2. Eliminar nulos en todas las columnas

```python
# Identificar y contar nulos en cada columna
for columna in df_clean.columns:
    count_nulos = df_clean.filter(col(columna).isNull()).count()
    if count_nulos > 0:
        print(f"- {columna}: {count_nulos} valores nulos")

# Eliminar nulos solo en columnas clave
columnas_clave = ["pickup_datetime", "dropoff_datetime", "fare_amount", "trip_distance"]
df_clean = df_clean.dropna(subset=columnas_clave)
```

Decidí mantener el enfoque de eliminar nulos solo en columnas clave para preservar más datos, ya que algunas columnas menos importantes podían tener valores nulos sin afectar significativamente mis análisis.

## 4. Creación de Agregados

Para facilitar los análisis comunes, creé un DataFrame agregado por hora:

```python
df_agregados = df_clean.groupBy("pickup_hour").agg(
    {"fare_amount": "avg", "trip_distance": "avg", "passenger_count": "sum"}
).withColumnRenamed("avg(fare_amount)", "avg_fare")
 .withColumnRenamed("avg(trip_distance)", "avg_distance")
 .withColumnRenamed("sum(passenger_count)", "total_passengers")
```

Este DataFrame agregado me permite analizar rápidamente patrones por hora del día sin necesidad de consultar todo el conjunto de datos.

## 5. Almacenamiento en Delta Lake

Guardé tanto el DataFrame limpio como los agregados en formato Delta:

```python
df_agregados.write.format("delta").mode("overwrite").saveAsTable("df_agregados")
df_clean.write.format("delta").mode("overwrite").saveAsTable("df_clean")
```

Delta Lake me proporciona varias ventajas:
- ACID transactions
- Historial de versiones (Time Travel)
- Optimizaciones de rendimiento
- Esquema aplicado

## 6. Optimización de Tablas Delta

Para mejorar el rendimiento de consultas, apliqué varias optimizaciones:

```python
# Optimizar tabla (compactar archivos pequeños)
spark.sql("OPTIMIZE df_clean")

# Añadir Z-Ordering para mejor rendimiento en consultas por fecha
spark.sql("OPTIMIZE df_clean ZORDER BY (pickup_datetime)")

# Configurar propiedades para mantenimiento automático
spark.sql("""
ALTER TABLE df_clean 
SET TBLPROPERTIES (
  'delta.logRetentionDuration' = '30 days',
  'delta.deletedFileRetentionDuration' = '7 days',
  'delta.autoOptimize.optimizeWrite' = 'true',
  'delta.autoOptimize.autoCompact' = 'true'
)
""")
```

Estas optimizaciones incluyen:
- Compactación de archivos pequeños
- Z-Ordering por fecha para acelerar consultas filtradas por tiempo
- Configuración de retención de logs y archivos eliminados
- Habilitación de optimizaciones automáticas

## 7. Creación de Vistas SQL

Para facilitar el acceso a análisis comunes, creé una vista SQL:

```python
spark.sql("""
CREATE OR REPLACE VIEW taxi_hourly_stats AS
SELECT
  pickup_hour,
  COUNT(*) as trips_count,
  AVG(fare_amount) as avg_fare,
  AVG(trip_distance) as avg_distance
FROM df_clean
GROUP BY pickup_hour
ORDER BY pickup_hour
""")
```

Esta vista permite ejecutar consultas simples sin necesidad de escribir código complejo de agrupación cada vez.

## 8. Preparación para Ingesta Incremental

Finalmente, preparé funciones para la ingesta incremental de nuevos datos:

```python
def procesar_e_ingerir_datos(ruta_origen, tabla_destino):
    # Leer nuevos datos
    nuevos_datos = spark.read.option("header", True).schema(schema).csv(ruta_origen)
    
    # Aplicar las mismas transformaciones
    nuevos_datos_procesados = (
        nuevos_datos.withColumn("pickup_datetime", to_timestamp(col("tpep_pickup_datetime")))
        .withColumn("dropoff_datetime", to_timestamp(col("tpep_dropoff_datetime")))
        .filter(col("pickup_datetime").isNotNull())
        .filter(col("passenger_count") > 0)
        .filter(col("trip_distance") > 0)
        .filter(col("fare_amount") >= 0)
        .withColumn("pickup_hour", hour("pickup_datetime"))
    )
    
    # Guardar en formato Delta (modo append)
    nuevos_datos_procesados.write \
        .format("delta") \
        .mode("append") \
        .saveAsTable(tabla_destino)
    
    return nuevos_datos_procesados.count()
```

También implementé funciones para verificar la calidad de los datos y compactar el historial de logs, proporcionando un pipeline completo para mantenimiento continuo de datos.

## Conclusiones

Este proceso me permitió procesar eficientemente un gran volumen de datos de taxis y almacenarlos en un formato optimizado para análisis. La implementación de Delta Lake proporciona numerosas ventajas para el manejo de datos a escala, incluyendo:

- Control de versiones de datos
- Optimización de rendimiento
- Mantenimiento automático
- Esquema aplicado

Los datos ahora están listos para análisis avanzados, visualizaciones y posible alimentación de modelos de machine learning.subir aquí la docu plz
