
 # 🚕 Análisis de Datos de Taxis de Nueva York con PySpark en Databricks 🧱
 
Este proyecto implementa un workflow ETL para procesar datos de viajes en taxi en Nueva York utilizando PySpark con Databricks. El objetivo es construir un flujo de datos escalable y eficiente que permita el analizar en tiempo real (en fase incompleta) a través de dashboards.
 
**Partes del ETL**
 
🕸️ Web Scrapping de datos.
 
🧼 Limpieza y transformación de datos con PySpark
 
💾 Guardado de datos en parquet y extracción a tablas.
 
📊 Visualizaciones de dashboards.
 
 
 
---
 
1. 🕸️ Web Scrapping de datos
 
Se utilizó la librería BeautifulSoup (bs4) para recoger todos los archivos paquet de taxis desde el año 2020 hasta el año 2024 (https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).
 
```python

  local_dir = "/Volumes/workspace/default/test_volume"
  
  headers = headers = {
  "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36",
      "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
      "Accept-Language": "en-US,en;q=0.9",
  "Referer": "https://www.google.com/",
      "Connection": "keep-alive",
      "DNT": "1",  # Do Not Track
      "Upgrade-Insecure-Requests": "1"
  }
  
  
  session = requests.Session()
  for i, file_url in enumerate(data_links):
      file_name = file_url.split("/")[-1]
      file_path = os.path.join(local_dir, file_name)
  
      try:
          # Descargar el archivo 
          response = session.get(file_url, headers=headers, allow_redirects=True)
  
          if response.status_code == 200:
              with open(file_path, "wb") as file:
                  for chunk in response.iter_content(chunk_size=8192):
                      file.write(chunk)
              print(f"✅ Archivo guardado en: {file_path}")
  
              # Agregar una pausa estratégica cada 10 archivos
              if i % 10 == 0:
                  espera = random.uniform(15, 40)
                  print(f"⏳ Pausa de {espera} segundos para evitar bloqueos...")
                  time.sleep(espera)
  
          else:
              print(f"❌ Error {response.status_code}: No se pudo descargar {file_name}")
  
      except Exception as e:
          print(f"❌ Error al descargar {file_name}: {str(e)}")
```
 
 
---
 
2. 🧹 Limpieza y Validación de Datos
 
El proceso de limpieza se ejecutan las siguientes fases:

  1. Visualización inicial de los datos:
     
       ```python
          from pyspark.sql import SparkSession

          # Crear una sesión de Spark
          spark = SparkSession.builder \
              .appName("Leer Parquet desde Managed Volume") \
              .config("spark.executor.memory", "16g") \
              .config("spark.driver.memory", "16g") \
              .config("spark.executor.cores", "4") \
              .config("spark.sql.shuffle.partitions", "200") \
              .getOrCreate()
          
          archive= "fhvhv_tripdata_2019-03.parquet"
          # Leer el archivo Parquet desde el managed volume
          df= spark.read.parquet(f'dbfs:/Volumes/workspace/default/test_volume/{archive}')

          df.columns
          df.display()
       ```
       
---

  2. Fusión de todos los archivos de un mismo tipo (yellow, green, fhv, fhvhv). Para poder manejar todos los datos de un mismo tipo de vehículo se crea una función que fusione todos los archivos de un mismo tipo de vehículo en un dataframe de spark.
     
     ```python
        # Función para leer y mergear
        def merge_files(file_list, output_name):
            df_list = [spark.read.parquet(os.path.join(local_dir, f), header=True, inferSchema=True) for f in file_list]
            merged_df = df_list[0]
            for df in df_list[1:]:
                merged_df = merged_df.unionByName(df)
            merged_df.write.mode('overwrite').parquet(os.path.join(local_dir, output_name))
     ```
---
 
  3. Transformaciones y limpieza de datos.
     
       Columnas de fechas deben ser transformadas a datetime(), las de hora a tipo hour(). Además, también se manejan nulos y valores no consistentes como viajes con 0 pasajeros.
     

     ```python
       df_clean = df.withColumn("tpep_pickup_datetime", to_timestamp(col("tpep_pickup_datetime"))) \
                             .withColumn("tpep_dropoff_datetime", to_timestamp(col("tpep_dropoff_datetime"))) \
                             .filter(col("tpep_pickup_datetime").isNotNull()) \
                             .filter(col("passenger_count") > 0) \
                             .withColumn("pickup_hour", hour("tpep_pickup_datetime"))
     ```
     

      También se añaden columnas con valor adicional, como el día de semana del viaje, día de mes y mes:

     ```python
        df_clean = yellow_df_clean.withColumn(
          "pickup_month", 
          concat_ws("_", date_format("tpep_pickup_datetime", "M"), date_format("tpep_pickup_datetime", "MMMM")))

        df_clean = df_clean.withColumn("pickup_day_of_week", date_format("tpep_pickup_datetime", "EEEE"))
     ```
 
---

  4. Agregaciones de datos.

     ```python
        def aggregate_data(df_clean, type_value):
          agg_df = df_clean.groupBy("pickup_hour").agg({
      "total_amount": "avg", "trip_distance": "avg","fare_amount": "avg" ,"passenger_count": "sum"}
          ).orderBy(asc("pickup_hour")).withColumnRenamed("avg(total_amount)", "avg_amount") \
           .withColumnRenamed("avg(trip_distance)", "avg_distance") \
           .withColumnRenamed("avg(fare_amount)", "avg_fare") \
           .withColumnRenamed("sum(passenger_count)", "total_passengers") \
           .withColumn("type", lit(type_value))
          return agg_df
     ```

---
 
  5. 💾 Almacenamiento en parquet
   
    Los datos transformados se almacenaron como parquet para poder generar tablas y dashboards.

    ```python
      file_path_parquet = '/Volumes/workspace/default/test_volume/yellow_green.parquet'
    
      yellow_green_dataframe = yellow_df_clean.union(green_df_clean)
      
      yellow_green_df = yellow_green_dataframe.coalesce(1)
      
      yellow_green_df.write \
          .mode("overwrite") \
          .parquet(file_path_parquet)
    ```
 
 
---
 
6. 📈 Visualizaciones de Dashboards


 
 
