# ✨ Proyecto: Ingesta, Transformación y Visualización de Datos NYC Taxi Trips

## 📄 Objetivo del Proyecto
Este proyecto tiene como fin practicar el ciclo completo de un flujo de datos ETL (Extracción, Transformación y Carga) utilizando PySpark sobre Databricks. Se trabajará con datos de viajes de taxi en Nueva York, aplicando una arquitectura Medallion (Bronze, Silver, Gold) y gestionando los datos con Unity Catalog.

## 🏢 Parte 1: Preparación del Entorno en Databricks
* **Databricks:** Plataforma en la nube para análisis e ingeniería de datos con Apache Spark.
* **Unity Catalog:** Se utiliza para la gestión centralizada de catálogos, esquemas y tablas.
    * Catálogo Principal: `taxi_project_catalog`
* **Cluster:** Se requiere un clúster de Databricks configurado.
* **Notebooks del Proyecto:**
    * `00_Setup_and_Load_Bronze`: Carga de datos crudos a la capa Bronze.
    * `01_Exploratory_Data_Analysis`: Análisis exploratorio.
    * `02_Process_Silver`: Limpieza y transformación a la capa Silver.
    * `03_Create_Gold_Aggregates`: Creación de agregados en la capa Gold.

## 🚜 Parte 2: Ingesta de Datos Crudos (Capa Bronze)
* **Notebook:** `00_Setup_and_Load_Bronze`
* **Dataset:** Datos de NYC Taxi Trips en formato Parquet.
    * Ruta de ejemplo: `/Volumes/mydata/nyc_taxi_data/raw_files/taxi_data_2020_2024.parquet`
* **Proceso:**
    1.  **Configurar Parámetros:** Nombres para catálogo, esquema Bronze (`bronze_layer`) y tabla Bronze (`raw_taxi_trips`).
    2.  **Inicializar Spark con Delta y Unity Catalog:**
        ```python
        from pyspark.sql import SparkSession

        spark = SparkSession.builder \
            .appName("NYC Taxi - Bronze Load") \
            .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
            .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
            .getOrCreate()

        CATALOG_NAME = "taxi_project_catalog"
        BRONZE_SCHEMA_NAME = "bronze_layer"
        spark.sql(f"CREATE CATALOG IF NOT EXISTS {CATALOG_NAME}")
        spark.sql(f"CREATE SCHEMA IF NOT EXISTS {CATALOG_NAME}.{BRONZE_SCHEMA_NAME}")
        ```
    3.  **Cargar Datos desde Parquet:**
        ```python
        RAW_DATA_PATH = "/Volumes/mydata/nyc_taxi_data/raw_files/taxi_data_2020_2024.parquet"
        df_raw = spark.read.format("parquet").load(RAW_DATA_PATH)
        ```
    4.  **Guardar en Tabla Delta Bronze:**
        ```python
        BRONZE_TABLE_NAME = "raw_taxi_trips"
        bronze_table_full_name = f"{CATALOG_NAME}.{BRONZE_SCHEMA_NAME}.{BRONZE_TABLE_NAME}"
        df_raw.write \
            .format("delta") \
            .mode("overwrite") \
            .option("overwriteSchema", "true") \
            .saveAsTable(bronze_table_full_name)
        ```
* **Explicación:** Los datos crudos se cargan tal cual desde el Parquet y se almacenan en una tabla Delta en la capa Bronze para preservarlos y permitir reprocesamientos.

## 🔍 Parte Intermedia: Análisis Exploratorio de Datos (EDA)
* **Notebook:** `01_Exploratory_Data_Analysis`
* **Objetivo:** Analizar los datos de la capa Bronze para identificar patrones, anomalías, outliers y definir las reglas de limpieza que se aplicarán en la capa Silver. Se generan estadísticas descriptivas, conteos de nulos, histogramas, box plots y se crean características temporales para el análisis.
* **Importancia:** Este paso es fundamental para asegurar la calidad de los datos en las capas subsiguientes. Los umbrales y criterios de limpieza (ej. rangos válidos para `passenger_count`, `trip_distance`, `fare_amount`) se definen aquí.

## 🔧 Parte 3: Transformación y Limpieza de Datos (Capa Silver)
* **Notebook:** `02_Process_Silver`
* **Objetivo:** Aplicar las reglas de limpieza y transformaciones definidas en el EDA a los datos de la capa Bronze, y almacenar los datos limpios y enriquecidos en la capa Silver.
* **Proceso:**
    1.  **Cargar de Bronze y Definir Umbrales de Calidad (basados en EDA):**
        ```python
        df_bronze = spark.read.table(f"{CATALOG_NAME}.{BRONZE_SCHEMA_NAME}.{BRONZE_TABLE_NAME}")
        
        MIN_PASSENGERS = 1     
        MAX_PASSENGERS = 6
        MIN_TRIP_DISTANCE = 0.0
        MAX_TRIP_DISTANCE_DOMAIN = 100 
        # ... más umbrales para fechas, tarifas, duración, etc.
        ```
    2.  **Calcular Duración y Aplicar Filtros de Calidad:**
        ```python
        from pyspark.sql.functions import col, year, unix_timestamp
        
        df_with_duration = df_bronze.withColumn(
            "trip_duration_seconds",(unix_timestamp(col("tpep_dropoff_datetime")) - unix_timestamp(col("tpep_pickup_datetime")))
        )
        # Ejemplo de filtro
        df_filtered = df_with_duration.filter(
            (year(col("tpep_pickup_datetime")) >= 2020) & (col("trip_duration_seconds") >= 60) &
            (col("passenger_count").between(MIN_PASSENGERS, MAX_PASSENGERS)) 
            # ... muchos más filtros
        )
        ```
    3.  **Ingeniería de Características Adicionales:** Se crean columnas como `pickup_hour`, `pickup_day_name`, `avg_speed_mph`, `tip_percentage`.
        ```python
        from pyspark.sql.functions import hour, date_format, when
        df_featured = df_filtered \
            .withColumn("pickup_hour", hour(col("tpep_pickup_datetime"))) \
            .withColumn("avg_speed_mph", when(col("trip_duration_seconds") > 0, (col("trip_distance") / (col("trip_duration_seconds") / 3600.0))).otherwise(0.0))
        # ... más características
        ```
    4.  **Guardar en Tabla Delta Silver (Particionada y Optimizada):**
        ```python
        SILVER_SCHEMA_NAME = "silver_layer"
        SILVER_TABLE_NAME = "cleaned_enriched_taxi_trips"
        silver_table_full_name = f"{CATALOG_NAME}.{SILVER_SCHEMA_NAME}.{SILVER_TABLE_NAME}"
        
        df_silver_final.write \
            .format("delta") \
            .mode("overwrite") \
            .option("overwriteSchema", "true") \
            .partitionBy("pickup_year", "pickup_month") \
            .saveAsTable(silver_table_full_name)
        
        spark.sql(f"OPTIMIZE {silver_table_full_name} ZORDER BY (tpep_pickup_datetime)")
        ```

## 🧰 Parte 4: Agregaciones (Capa Gold)
* **Notebook:** `03_Create_Gold_Aggregates`
* **Objetivo:** Crear tablas agregadas a partir de los datos limpios de la capa Silver, optimizadas para consultas de BI y dashboards.
* **Proceso:**
    1.  **Cargar de Silver:**
        ```python
        df_silver = spark.read.table(f"{CATALOG_NAME}.{SILVER_SCHEMA_NAME}.{SILVER_TABLE_NAME}")
        ```
    2.  **Definir Agregaciones:** Se calculan métricas como total de viajes, pasajeros, ingresos promedio, distancia promedio, etc., agrupadas por diversas dimensiones.
        * **Agregación Horaria (ejemplo):**
        ```python
        from pyspark.sql.functions import avg, sum, count as F_count
        
        GOLD_SCHEMA_NAME = "gold_layer"
        HOURLY_AGG_TABLE_NAME = "agg_hourly_trip_metrics"
        
        df_hourly_agg = df_silver.groupBy("pickup_hour").agg(
            F_count("*").alias("total_trips"),
            sum("passenger_count").alias("total_passengers"),
            avg("fare_amount").alias("avg_fare_amount")
            # ... más agregaciones horarias ...
        ).orderBy("pickup_hour")
        
        # Guardar usando la función helper save_gold_table
        # save_gold_table(df_hourly_agg, HOURLY_AGG_TABLE_NAME)
        ```
        Se crean múltiples tablas agregadas: `agg_daily_trip_metrics`, `agg_monthly_trip_metrics`, `agg_pickup_location_metrics`, `agg_payment_type_metrics`, `agg_overall_performance_summary`.
    3.  **Guardar Tablas Agregadas en Delta Lake (Gold):** Cada DataFrame agregado se guarda como una tabla Delta en el esquema `gold_layer`.

## 🗂 Parte 5: Almacenamiento en Delta Lake
* **Delta Lake:** Utilizado en todas las capas (Bronze, Silver, Gold) por sus ventajas:
    * Transacciones ACID.
    * Versionado de datos (Time Travel).
    * Manejo y evolución de esquemas.
    * Optimizaciones (`OPTIMIZE`, `ZORDER BY`).
    * Ideal para cargas incrementales (aunque este ejemplo usa `overwrite` para simplificar, se puede adaptar a `append` o `merge`).
* **Tablas Delta en Unity Catalog:** Visibles y gestionables a través del Data Explorer de Databricks y consultables vía SQL.

## 🚀 Parte 6: Pipeline Incremental y Automatización (Consideraciones Futuras)
* **Ingesta Incremental:** Aunque este proyecto usa `mode("overwrite")` para las tablas, en un escenario real se implementaría una lógica para procesar solo datos nuevos (ej. nuevos archivos Parquet mensuales).
