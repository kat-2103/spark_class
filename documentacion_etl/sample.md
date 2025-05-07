# Proyecto: Ingesta, Transformación y Visualización de Datos NYC Taxi Trips con PySpark en Google Colab

## 📄 Descripción General

Este proyecto demuestra un ciclo completo de un flujo de datos ETL (Extracción, Transformación y Carga) utilizando PySpark en el entorno de Google Colab. El objetivo principal es proporcionar una guía práctica para trabajar con datos de viajes de taxi de Nueva York, desde la ingesta de archivos Parquet hasta la visualización de insights clave. Está diseñado para que los usuarios puedan aprender y practicar conceptos fundamentales de procesamiento de datos a gran escala con Spark.

### 🎯 Objetivos del Proyecto

*   Implementar un pipeline de datos para ingestar, procesar y limpiar datos de viajes de taxi.
*   Realizar agregaciones para obtener estadísticas útiles (ej. viajes por hora, tarifas promedio).
*   Almacenar los datos procesados y agregados en formato Parquet.
*   Construir visualizaciones para extraer insights del negocio directamente en el notebook.
*   Comprender cómo configurar y utilizar PySpark en Google Colab.

### 💼 Caso Empresarial Simulado

Una empresa de transporte de taxis en Nueva York busca modernizar su análisis de datos. Han recopilado datos de viajes durante años y necesitan un sistema para:

1.  **Ingesta Automatizada:** Procesar nuevos archivos mensuales de datos de viajes.
2.  **Limpieza y Transformación:** Estandarizar formatos de fecha, manejar datos faltantes, calcular duraciones de viaje, etc.
3.  **Agregación de Insights:** Identificar horas pico de demanda, tarifas promedio por zona, patrones de distancia, etc.
4.  **Almacenamiento Eficiente:** Guardar los datos limpios y agregados en un formato optimizado como Parquet.
5.  **Visualización:** Crear dashboards o reportes visuales para la toma de decisiones.

Este notebook aborda estos puntos, adaptando la solución al entorno de Google Colab.

### 🧠 Aprendizajes Clave

Al completar este proyecto, se espera que el usuario pueda:

*   Configurar e inicializar una sesión de PySpark en Google Colab.
*   Cargar datos desde múltiples archivos Parquet en un DataFrame de Spark.
*   Aplicar diversas transformaciones de datos: limpieza, conversión de tipos, ingeniería de características (extracción de componentes de fecha, cálculo de duraciones).
*   Realizar agregaciones de datos utilizando operaciones `groupBy`.
*   Guardar DataFrames de Spark en formato Parquet.
*   Convertir DataFrames de Spark a Pandas para visualización.
*   Crear visualizaciones básicas utilizando librerías como Matplotlib, Seaborn o Plotly.
*   Comprender las consideraciones al trabajar con múltiples archivos y esquemas potencialmente diferentes.

## 🛠️ Requisitos y Dependencias

*   **Entorno:** Google Colab (o cualquier entorno Python con Jupyter Notebooks y Spark instalado).
*   **Lenguaje:** Python 3.x
*   **Librerías Principales:**
    *   `pyspark`: Para el procesamiento distribuido de datos.
    *   `findspark`: Para ayudar a localizar Spark en el sistema.
*   **Librerías de Soporte (generalmente preinstaladas en Colab o instaladas con PySpark):**
    *   `os`: Para operaciones del sistema operativo (manejo de rutas).
    *   `urllib.request`: Para descargar archivos desde URLs.
    *   `json`: Para manejar datos en formato JSON (si se usa para listas de archivos).
*   **Librerías de Visualización (ejemplos en el notebook):**
    *   `matplotlib`
    *   `seaborn`
    *   `plotly`

### Instalación en Google Colab

La primera celda de código en el notebook se encarga de instalar `pyspark` y `findspark`:

```python
!pip install -q pyspark findspark
```

## 📂 Estructura del Repositorio (Sugerida)

```
/nyc-taxi-etl-colab
|-- NYC_Taxi_ETL_Colab.ipynb  # El notebook principal del proyecto
|-- sample.md                 # Esta documentación (README)
|-- parquet_links.json        # (Opcional) Archivo JSON con URLs a los datasets Parquet de NYC TLC
|-- output_data/              # (Directorio creado al ejecutar) Para almacenar los Parquet procesados
|   |-- processed_trips.parquet
|   |-- hourly_pickups.parquet
|   |-- ... (otros resultados agregados)
|-- images/                   # (Opcional) Para cualquier imagen utilizada en esta documentación
```

## 🚀 Instrucciones de Uso

1.  **Abrir el Notebook:**
    *   Sube el archivo `NYC_Taxi_ETL_Colab.ipynb` a tu entorno de Google Colab.
    *   Alternativamente, si está en un repositorio Git, puedes abrirlo directamente en Colab usando la URL del archivo en GitHub.

2.  **Preparar el Entorno:**
    *   Ejecuta la primera celda de código para instalar `pyspark` y `findspark`.
    *   Ejecuta la celda que inicializa `findspark` y crea la `SparkSession`. Esto prepara el entorno Spark para su uso.

3.  **Configurar Fuentes de Datos (Ingesta):
    *   Localiza la celda de código bajo la sección "🚜 Parte 2: Ingesta de Datos".
    *   Dentro de esta celda, encontrarás una lista llamada `parquet_files_to_download` (o similar).
    *   **Modifica esta lista** para incluir las URLs de los archivos Parquet de NYC Taxi que deseas procesar. Puedes añadir o quitar diccionarios de la lista. Cada diccionario debe tener al menos una clave `"href"` con la URL del archivo Parquet.
        ```python
        # Ejemplo de cómo modificar la lista:
        parquet_files_to_download = [
            {"text": "Yellow Taxi Trip Records 2023-01", "href": "https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet"},
            {"text": "Yellow Taxi Trip Records 2023-02", "href": "https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-02.parquet"},
            # Añade más archivos aquí si es necesario
        ]
        ```
    *   (Opcional) Si tienes un archivo `parquet_links.json` con una lista completa de URLs, puedes adaptar el código para leer este JSON y seleccionar los archivos desde allí.

4.  **Ejecutar el Pipeline ETL:**
    *   Ejecuta las celdas del notebook en secuencia.
    *   **Ingesta:** Los archivos Parquet especificados se descargarán al directorio `/content/data/` en Colab y luego se cargarán en un DataFrame de Spark.
    *   **Transformación:** Se aplicarán diversas operaciones de limpieza y transformación de datos.
    *   **Agregación:** Se calcularán estadísticas y se agruparán los datos.
    *   **Almacenamiento:** Los DataFrames resultantes (procesados y agregados) se guardarán en formato Parquet en el directorio `/content/output_data/` (o el especificado en el código).
    *   **Visualización:** Se generarán gráficos directamente en las celdas de salida del notebook.

5.  **Revisar Resultados:**
    *   **Archivos Parquet:** Puedes encontrar los archivos Parquet generados en el panel de archivos de Colab, dentro de la carpeta `output_data`. Puedes descargarlos a tu máquina local.
    *   **Visualizaciones:** Observa los gráficos generados en el notebook para entender los insights de los datos.
    *   **Logs y Salidas:** Revisa las salidas de las celdas para ver esquemas de DataFrames, conteos de filas y cualquier mensaje de log.

## ⚙️ Flujo de Trabajo Detallado (Resumen del Notebook)

El notebook está estructurado en varias partes lógicas:

### Parte 1: Preparación del Entorno
*   Instalación de dependencias (`pyspark`, `findspark`).
*   Inicialización de la `SparkSession`:
    ```python
    import findspark
    findspark.init()
    from pyspark.sql import SparkSession

    spark = SparkSession.builder \
        .appName("NYCTaxiETLColab") \
        .master("local[*]") \
        .getOrCreate()
    sc = spark.sparkContext
    ```

### Parte 2: Ingesta de Datos
*   Definición de las URLs de los archivos Parquet a procesar.
*   Descarga de los archivos al entorno local de Colab (`/content/data/`).
*   Lectura de múltiples archivos Parquet en un único DataFrame de Spark. Spark puede manejar la lectura de múltiples archivos y, si los esquemas son compatibles, los unirá. Si los esquemas difieren (ej. al mezclar datos de taxis amarillos y verdes), Spark intentará una "fusión de esquemas".
    ```python
    # `downloaded_file_paths` es una lista de rutas locales a los archivos descargados
    # ej: ["/content/data/yellow_tripdata_2023-01.parquet", ...]
    uri_file_paths = [f"file://{path}" for path in downloaded_file_paths]
    df = spark.read.parquet(*uri_file_paths)
    
    df.printSchema()
    df.show(5)
    ```

### Parte 3: Limpieza y Transformación de Datos
Esta sección incluye varias sub-tareas comunes en un ETL:
*   **Conversión de Tipos de Datos:** Asegurar que las columnas tengan los tipos correctos (ej. fechas a `TimestampType`, numéricos a `DoubleType` o `IntegerType`).
*   **Manejo de Nulos:** Identificar y tratar valores faltantes (ej. eliminar filas o imputar valores).
*   **Renombrar Columnas:** Para mayor claridad o consistencia.
*   **Ingeniería de Características:** Crear nuevas columnas útiles para el análisis. Ejemplos:
    *   Extraer componentes de fecha/hora (año, mes, día, hora, día de la semana) de las columnas de timestamp.
        ```python
        from pyspark.sql.functions import year, month, dayofmonth, hour, dayofweek
        df = df.withColumn("pickup_hour", hour("tpep_pickup_datetime"))
        ```
    *   Calcular la duración del viaje.
        ```python
        from pyspark.sql.functions import col
        df = df.withColumn("trip_duration_seconds", 
                           (col("tpep_dropoff_datetime").cast("long") - col("tpep_pickup_datetime").cast("long")))
        df = df.withColumn("trip_duration_minutes", col("trip_duration_seconds") / 60)
        ```
*   **Filtrado de Datos:** Eliminar registros anómalos o irrelevantes (ej. viajes con distancia o tarifa cero/negativa, número de pasajeros inválido).
    ```python
    df_filtered = df.filter((col("trip_distance") > 0) & 
                            (col("fare_amount") > 0) & 
                            (col("passenger_count") > 0))
    ```

### Parte 4: Agregación de Datos
*   Agrupar datos para obtener estadísticas resumidas. Ejemplo: contar viajes por hora.
    ```python
    from pyspark.sql.functions import desc
    hourly_pickups = df_filtered.groupBy("pickup_hour").count().orderBy(desc("count"))
    hourly_pickups.show()
    ```
*   Otros ejemplos pueden incluir tarifas promedio por zona, distancia promedio por tipo de pasajero, etc.

### Parte 5: Almacenamiento de Datos
*   Guardar los DataFrames procesados y/o agregados en formato Parquet para uso futuro o análisis más profundo.
    ```python
    df_filtered.write.mode("overwrite").parquet("/content/output_data/processed_taxi_trips.parquet")
    hourly_pickups.write.mode("overwrite").parquet("/content/output_data/hourly_pickups_summary.parquet")
    ```
    *   `mode("overwrite")` reemplazará los archivos si ya existen. Otras opciones son `append`, `ignore`, `errorifexists`.

### Parte 6: Visualización de Datos
*   Convertir los DataFrames de Spark (que suelen ser agregados y de tamaño manejable) a DataFrames de Pandas para facilitar la visualización con librerías estándar de Python.
    ```python
    pandas_hourly_pickups = hourly_pickups.toPandas()
    ```
*   Crear gráficos utilizando `matplotlib.pyplot`, `seaborn`, o `plotly.express`.
    *   Ejemplo con Matplotlib/Seaborn para un gráfico de barras:
        ```python
        import matplotlib.pyplot as plt
        import seaborn as sns

        plt.figure(figsize=(12, 6))
        sns.barplot(x="pickup_hour", y="count", data=pandas_hourly_pickups, palette="viridis")
        plt.title("Número de Viajes de Taxi por Hora del Día")
        plt.xlabel("Hora de Recogida")
        plt.ylabel("Número de Viajes")
        plt.xticks(rotation=45)
        plt.tight_layout()
        plt.show()
        ```

## 💡 Consideraciones Adicionales

*   **Manejo de Esquemas al Cargar Múltiples Archivos:**
    *   Si cargas archivos Parquet de diferentes tipos de datos (ej. Yellow Taxi, Green Taxi, FHV) simultáneamente, sus esquemas pueden diferir (diferentes nombres de columna, tipos de datos, columnas adicionales/faltantes).
    *   PySpark intentará fusionar estos esquemas si lees desde un directorio (`spark.read.option("mergeSchema", "true").parquet(directorio)`). Si lees una lista de archivos, el comportamiento puede variar o requerir unión manual si los esquemas son muy dispares.
    *   **Es crucial inspeccionar el esquema del DataFrame combinado (`df.printSchema()`)** después de la carga y adaptar las transformaciones subsiguientes. Es posible que necesites manejar columnas que solo existen en algunos de los datasets fuente (resultando en nulos para otros) o estandarizar nombres de columnas.

*   **Recursos en Google Colab:**
    *   Las sesiones gratuitas de Google Colab tienen limitaciones en cuanto a RAM, disco y tiempo de ejecución. El procesamiento de volúmenes muy grandes de datos puede exceder estos límites.
    *   Para datasets más grandes, considera ejecutar el notebook en lotes más pequeños, muestrear los datos o utilizar una plataforma Spark más robusta.

*   **Persistencia de Datos:**
    *   Los archivos guardados en el sistema de archivos de Colab (`/content/`) son temporales y se eliminan cuando la sesión termina.
    *   Para persistir tus datos de entrada, salida o el propio notebook, monta tu Google Drive y ajusta las rutas de lectura/escritura para que apunten a tu Drive.
        ```python
        from google.colab import drive
        drive.mount('/content/drive')
        # Ejemplo de ruta en Drive: "/content/drive/MyDrive/Colab Notebooks/NYCTaxi/output_data/"
        ```

*   **Automatización:**
    *   La automatización completa de pipelines ETL como se haría en un entorno de producción (ej. con Apache Airflow, cron jobs en un servidor, o Databricks Jobs) tiene limitaciones en Colab.
    *   Colab no está diseñado para ser un orquestador de flujos de trabajo productivos. Sin embargo, puedes:
        *   Ejecutar el notebook manualmente cuando sea necesario.
        *   Explorar opciones limitadas de ejecución programada si Google ofrece alguna funcionalidad para ello (esto puede cambiar).
        *   Utilizar APIs de Colab si deseas activar la ejecución desde un script externo, aunque esto es más complejo.

## 🤝 Contribuciones

Las contribuciones a este proyecto son bienvenidas. Por favor, abre un *issue* para discutir cambios importantes o envía un *pull request* con tus mejoras.

## 📜 Licencia

Este proyecto se distribuye bajo la Licencia MIT. Consulta el archivo `LICENSE` para más detalles (si aplica).

