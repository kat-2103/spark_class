
# ✨ Proyecto: Ingesta, Transformación y Visualización de Datos NYC Taxi Trips

## 📄 Objetivo del Proyecto

Este proyecto tiene como fin que aprendáis y praciquéis el ciclo completo de un flujo de datos ETL (Extracción, Transformación y Carga) utilizando PySpark sobre Databricks Community Edition. Se trabajará con datos reales de viajes de taxi en Nueva York, disponibles de forma pública.
### Caso empresarial: 
Una empresa de transporte de taxis en Nueva York ha recopilado durante años datos de sus viajes (fecha, ubicación de recogida y destino, tarifas, número de pasajeros, etc.). Actualmente, dispone de estos archivos en formato CSV y ha solicitado tu ayuda como ingeniero/a de datos para lograr lo siguiente:

1. Implementar un pipeline de datos que:
   * Ingesta automáticamente nuevos archivos mensuales de viajes.
   * Procese y limpie los datos (fechas, ubicaciones, tarifas, pasajeros).
   * Agregue estadísticas útiles por hora o zona.
   * Almacene los resultados en un formato eficiente (Parquet o Delta).

2. Construir un dashboard visual con insights clave del negocio:
   * Horas con más pasajeros.
   * Tarifas promedio.
   * Distancias recorridas.

3. Automatizar este flujo para que se ejecute al detectar nuevos archivos o en intervalos regulares.

🧠 Aprendizajes del proyecto:

Crear y configurar clusters en Databricks.

Usar PySpark para cargar, transformar y agregar datos.

Almacenar resultados con Delta Lake.

Automatizar pipelines ETL incrementales.

Construir dashboards interactivos sobre notebooks.

---

# 🏢 Parte 1: Preparación del Entorno

## ▶ Qué es Databricks y por qué lo usamos

**Databricks** es una plataforma basada en la nube para el análisis de datos y la ingeniería de datos que combina Apache Spark con un entorno colaborativo de notebooks. La versión Community Edition es gratuita y perfecta para fines educativos.

### Ventajas:
- Ejecuta PySpark sin necesidad de configuración local.
- Incluye almacenamiento temporal (DBFS).
- Permite crear dashboards y programar trabajos.

## ✅ Registro y configuración inicial

1. Crear cuenta en [https://community.cloud.databricks.com](https://community.cloud.databricks.com)
2. Crear un cluster:
   - Nombre: `nyc_taxi_cluster`
   - Runtime: usar el predeterminado.
   - Esperar a que esté en estado RUNNING.

3. Crear un notebook:
   - Nombre: `NYC_Taxi_ETL`
   - Lenguaje: PySpark

---

# 🚜 Parte 2: Ingesta de Datos

## 📃 Dataset: NYC Yellow Taxi Trips

- Fuente oficial: [NYC TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
- Ejemplo de archivo: `yellow_tripdata_2019-01.csv`

### Descarga e Ingesta de datos (código explicado):

```python
# Descargar un archivo y moverlo a DBFS (Databricks File System)
import urllib.request
local_path = "/tmp/yellow_tripdata_2019-01.csv"
url = "https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2019-01.csv"
urllib.request.urlretrieve(url, local_path)

# Copiar al sistema distribuido de Databricks
dbutils.fs.cp("file:/tmp/yellow_tripdata_2019-01.csv", "dbfs:/tmp/nyc_taxi/yellow_tripdata_2019-01.csv")

# Leer CSV con Spark
df = spark.read.option("header", True).option("inferSchema", True).csv("dbfs:/tmp/nyc_taxi/yellow_tripdata_2019-01.csv")
```

**Explicación paso a paso:**
- `urllib.request`: Esta línea descarga el archivo CSV desde la URL pública proporcionada (en este caso, desde los servidores de NYC Taxi Data) y lo guarda localmente en el entorno del notebook de Databricks. Este entorno local es efímero y se reinicia al cerrar el cluster.
- `dbutils.fs.cp`: Copiamos el archivo desde el sistema local (ruta file:/tmp/...) hacia el sistema de archivos distribuido propio de Databricks: DBFS (Databricks File System). Esto permite que el archivo esté disponible de forma persistente en el entorno del workspace y pueda ser utilizado por múltiples notebooks.
                   Puedes ver los archivos que subes a DBFS navegando en el menú lateral de Databricks:
                    Data > DBFS > Buscar dentro de /tmp/nyc_taxi/.
                    También puedes usar %fs ls /tmp/nyc_taxi/ desde una celda del notebook para ver los archivos.
- `spark.read.csv`: Esta línea utiliza PySpark para leer el archivo CSV desde la ubicación en DBFS. Se especifican las opciones header=True (para que tome la primera línea como nombres de columnas) y inferSchema=True (para que intente detectar automáticamente el tipo de dato de cada columna). El resultado se guarda como un DataFrame, que es la estructura principal de datos en PySpark para manipular y transformar datos a gran escala.

---

# 🔧 Parte 3: Transformación de Datos

### Objetivos de la limpieza y transformación:

- Convertir fechas en formato timestamp.
- Filtrar valores inválidos (por ejemplo, pasajeros = 0).
- Crear nuevas columnas como la hora del viaje.
- **Diseñar las transformaciones necesarias según el criterio del equipo de análisis, con el fin de garantizar la calidad y utilidad de los datos para su representación posterior.**

Estas transformaciones deben tener en cuenta que los datos resultantes alimentarán visualizaciones que ayudarán a la toma de decisiones. Por lo tanto, es importante asegurarse de que los datos estén completos, limpios y representen la realidad del negocio de forma comprensible. 

Además, **todas las decisiones de transformación deben documentarse adecuadamente**, incluyendo el motivo de cada limpieza o agregado, para garantizar trazabilidad y transparencia del proceso ETL.

```python
from pyspark.sql.functions import col, to_timestamp, hour

# Limpieza básica
df_clean = df.withColumn("pickup_datetime", to_timestamp(col("tpep_pickup_datetime")))              .withColumn("dropoff_datetime", to_timestamp(col("tpep_dropoff_datetime")))              .filter(col("pickup_datetime").isNotNull())              .filter(col("passenger_count") > 0)

# Nueva columna con la hora de recogida
df_clean = df_clean.withColumn("pickup_hour", hour("pickup_datetime"))
```

---

# 🧰 Parte 4: Agregaciones

### ¿Por qué agregamos métricas por hora?

La empresa cliente quiere comprender el comportamiento de los viajes de taxi a lo largo del día. Agregar los datos por hora permite identificar patrones como:
- Las horas con mayor número de pasajeros.
- En qué momentos del día los viajes son más largos o más costosos.
- Qué franjas horarias generan más ingresos.

Estas métricas son clave para que el negocio tome decisiones operativas (por ejemplo, aumentar conductores en horas pico), estratégicas (campañas de marketing en horarios específicos) o incluso regulatorias (optimizar tarifas).

Este paso también simplifica la visualización de los datos en el dashboard, ya que se reduce la granularidad sin perder información importante.

### Agregación de métricas por hora:
```python
agg_df = df_clean.groupBy("pickup_hour").agg(
    {"fare_amount": "avg", "trip_distance": "avg", "passenger_count": "sum"}
).withColumnRenamed("avg(fare_amount)", "avg_fare")  .withColumnRenamed("avg(trip_distance)", "avg_distance")  .withColumnRenamed("sum(passenger_count)", "total_passengers")
```

---

# 🗂 Parte 5: Almacenamiento en Delta Lake

## ✨ ¿Qué es Delta Lake y por qué usarlo?

**Delta Lake** es un motor de almacenamiento optimizado que se ejecuta sobre Apache Spark. Permite trabajar con datos estructurados y semi-estructurados de forma confiable y escalable. A diferencia de los formatos tradicionales como CSV o Parquet, Delta permite operaciones transaccionales (ACID) y mantenimiento incremental de los datos.

### Ventajas principales:
- Permite hacer `UPDATE`, `DELETE`, `MERGE`, lo que es esencial para mantener la consistencia cuando los datos cambian.
- Versionado de datos ("time travel"): puedes acceder al estado de una tabla en un punto anterior en el tiempo.
- Optimizaciones como `ZORDER` y `OPTIMIZE` para mejorar el rendimiento.
- Ideal para cargas incrementales (añadir solo nuevos datos sin sobrescribir lo anterior).

### ¿Dónde ver las tablas Delta en Databricks?
- Las tablas Delta se guardan como carpetas con archivos `.parquet` más un directorio especial `_delta_log`.
- Puedes explorar estos archivos con `%fs ls /tmp/delta/nyc_taxi_agg`.
- También puedes crear una tabla sobre una carpeta Delta con SQL:
```sql
CREATE TABLE taxi_agg
USING DELTA
LOCATION '/tmp/delta/nyc_taxi_agg';
```
Luego podrás consultar esta tabla con comandos SQL directamente desde el notebook o el panel de SQL en Databricks.

### Guardar como Delta:
```python
agg_df.write.format("delta").mode("overwrite").save("/tmp/delta/nyc_taxi_agg")
```

---

# 🚀 Parte 6: Ingesta de Múltiples Archivos y Pipeline Incremental

## ¿Por qué es importante esta parte?

El cliente ha indicado que genera mensualmente un archivo con los datos de viajes. En lugar de procesar todos los datos desde cero cada vez, lo ideal es tener un **pipeline incremental** que cargue únicamente los nuevos archivos conforme estén disponibles. Esto ahorra tiempo, recursos y permite una automatización más eficiente.

### Relación con los objetivos del cliente:
- Permite actualizar el dashboard automáticamente cuando llegan nuevos datos.
- Mejora la eficiencia operativa y reduce la posibilidad de errores por reprocesamiento completo.
- Sienta las bases para escalabilidad futura con años completos de datos.

### ¿Cómo se ve en Databricks?
- Los nuevos archivos se pueden subir manualmente a `/tmp/nyc_taxi/` o cargar desde una fuente externa (como S3 o Azure).
- Puedes ver los archivos en el explorador lateral (`Data > DBFS`) o ejecutar:
```python
%fs ls /tmp/nyc_taxi/
```
- También puedes crear lógica en Python para detectar automáticamente si hay archivos nuevos que aún no se han procesado.

## Ingesta múltiple:
```python
df_all = spark.read.option("header", True).option("inferSchema", True).csv("dbfs:/tmp/nyc_taxi/yellow_tripdata_2019-*.csv")
```

## ¿Qué es un pipeline incremental y cómo lo crearán los alumnos?

El pipeline incremental es una serie de pasos de procesamiento que los alumnos implementarán en un **notebook de PySpark**, con el objetivo de automatizar la carga de nuevos archivos de datos de taxi.

Este pipeline debe ejecutarse periódicamente o en función de eventos (como la subida de un nuevo archivo) y realizar lo siguiente:

- Leer solo los archivos nuevos.
- Procesar y transformar los datos.
- Insertarlos en la tabla Delta existente sin duplicar ni sobreescribir datos previos.

Los estudiantes deberán diseñar este pipeline como parte de su entregable técnico, integrándolo en el flujo general del proyecto solicitado por el cliente.

## Pipeline incremental:
1. Detectar archivos nuevos.
2. Transformarlos.
3. Hacer `append` si ya existe una tabla Delta.

```python
df_new.write.format("delta").mode("append").save("/tmp/delta/nyc_taxi_agg")
```

---

# 📊 Parte 7: Dashboards Interactivos

## Objetivo de esta sección

El cliente ha solicitado un **dashboard empresarial en Databricks** que resuma de forma visual y clara los indicadores clave de los viajes en taxi. Este dashboard debe permitir tomar decisiones estratégicas y operativas a partir de datos reales, agregados y actualizados.

Los estudiantes deberán construir un **dashboard completo dentro de Databricks**, utilizando las herramientas gráficas de la plataforma (no únicamente `display(df)`), seleccionando los tipos de gráficos apropiados (barras, líneas, tablas) y organizándolos en un panel interactivo.

## Indicadores mínimos a incluir:
- Promedio de tarifa por hora del día.
- Total de pasajeros por hora.
- Distancia media recorrida por hora.

## Cómo crearlo:
1. Seleccionar los resultados transformados desde un notebook.
2. En cada celda con una visualización, usar el botón **"+ Add to Dashboard"**.
3. Crear un nuevo dashboard llamado `NYC Taxi Dashboard`.
4. Agregar y organizar los gráficos de manera clara y visualmente efectiva.
5. (Opcional) Incluir filtros interactivos si se crean tablas Delta con múltiples meses.

Este dashboard formará parte de la entrega al cliente y debe responder a las principales preguntas planteadas por ellos en el caso inicial.

---

# ⏰ Parte 8: Automatización con Triggers

## ¿Qué es un trigger y por qué es útil?

Un **trigger** (o desencadenador) permite ejecutar automáticamente un notebook en Databricks sin intervención manual. Esto es clave para el cliente, que necesita que los datos se procesen de forma periódica o cada vez que haya nueva información disponible.

La automatización permite que el dashboard esté siempre actualizado con los últimos datos sin que nadie tenga que lanzar el proceso manualmente.

## Cómo automatizar la ejecución en Databricks:

1. Ve al menú lateral y abre la sección **Jobs**.
2. Pulsa en **Create Job**.
3. Asigna un nombre al job (por ejemplo: `nyc_taxi_etl_job`).
4. Selecciona el notebook que contiene tu pipeline como tarea principal.
5. Configura el cluster (puedes usar uno existente o permitir que Databricks lo cree temporalmente).

## Tipos de trigger:

- **Manual**: el job se ejecuta cuando tú lo lances manualmente (útil para pruebas).
- **Programado**: el job se ejecuta automáticamente según una frecuencia que definas (cada día, cada hora, etc.).
- **Basado en condiciones**: con lógica avanzada podrías activar el job cuando se detecten nuevos archivos. Para esto necesitarás:
  - Guardar la lista de archivos procesados (por ejemplo en una tabla Delta).
  - Comparar esa lista con los archivos actuales en DBFS.
  - Si hay nuevos, ejecutar el pipeline.

Esto se puede hacer con un notebook de control y condicionales en Python.

Esta automatización es parte del entregable y garantiza que el sistema siga funcionando de forma autónoma después de su implementación.

---

# 🔍 Glosario Rápido

- **Cluster**: Conjunto de máquinas que ejecutan Spark.
- **DBFS**: Sistema de archivos de Databricks.
- **ETL**: Extracción, Transformación y Carga de datos.
- **Delta Lake**: Formato de almacenamiento optimizado con control transaccional.
- **Trigger**: Evento que lanza una tarea programada.

---

# 📈 Anexo: Buenas prácticas para la visualización de dashboards

Un dashboard bien diseñado debe ser claro, funcional y enfocado en las necesidades del usuario final. Aquí tienes algunas recomendaciones para construir un dashboard efectivo para el cliente:

### ✅ Claridad visual
- Usa títulos descriptivos en cada gráfico.
- Agrupa visualizaciones relacionadas (por ejemplo, todas las métricas horarias juntas).
- Usa colores coherentes y evita combinaciones confusas.

### 📊 Elección del gráfico adecuado
- Barras horizontales o verticales: ideales para comparar por horas o zonas.
- Líneas: para mostrar evolución temporal.
- Tablas: para mostrar detalles específicos como top zonas o valores exactos.

### 🔍 Enfoque en los KPIs del cliente
- No sobrecargues el dashboard: enfócate en 3-5 indicadores clave que respondan a preguntas concretas del negocio.
- Usa títulos que respondan a una pregunta: "¿A qué hora hay más pasajeros?", "¿Cuándo son más largos los viajes?"

### 🛠 Personalización técnica en Databricks
- Usa "Add to Dashboard" solo para visualizaciones limpias y relevantes.
- Renombra cada visualización desde el panel del dashboard para facilitar su lectura.
- (Opcional) Añade filtros si has trabajado con múltiples meses o zonas.
