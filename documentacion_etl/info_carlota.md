# Análisis de Datos de Taxis Amarillos en NYC (2020–2024)

## 📌 Descripción del Proyecto

Este proyecto analiza los datos de los taxis amarillos en Nueva York entre los años 2020 y 2024. A través de ETL desarrollado con Apache Spark y visualizaciones en Lightdash, se estudian los patrones de uso, ingresos y comportamiento horario del servicio de taxis.

## 🗃️ Fuentes de Datos

Los datos provienen del portal oficial de taxis de NYC: [NYC Taxi & Limousine Commission](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

- Se descargaron manualmente los datasets **Yellow Taxi** correspondientes a los años 2020–2024.
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

### 3. Conversión de Timestamps

Se transformaron las columnas de tiempo:

- `tpep_pickup_datetime`
- `tpep_dropoff_datetime`

Estas se convirtieron a tipo `datetime` para permitir operaciones de agregación temporal.

### 4. Particionamiento de Datos

Para optimizar el rendimiento de las consultas, los datos se particionaron por:

- `hour` (hora de recogida): muchas de las consultas agregadas se hacen por hora.
- `year`, `month`, `day`: para permitir filtrado rápido por periodos de tiempo.

Esta decisión mejora significativamente los tiempos de respuesta en entornos distribuidos como Spark o cuando se consulta desde herramientas como Lightdash.

## 📊 Visualización (Lightdash)

La visualización se desarrolló en Lightdash, conectando directamente con las tablas creadas en Databricks. Se construyeron dashboards interactivos con insights como:

- Pasajeros por hora y mes, año a año (2020–2024).
- Ingresos totales por franjas horarias.
- Distancia media y tarifa media según la hora del día.

Estas visualizaciones permiten detectar patrones de demanda y comportamiento del servicio, como las **horas pico**, o las franjas horarias que generan más ingresos.

## 🧰 Tecnologías Utilizadas

- **Apache Spark** para procesamiento de datos.
- **Databricks** como entorno de ejecución.
- **Python (PySpark)** para el desarrollo del ETL.
- **Lightdash** para la visualización de datos.

## 🎯 Motivación

El análisis busca responder a preguntas clave sobre la operación de taxis en la ciudad de Nueva York, como:

- ¿En qué horarios se concentra la mayor demanda?
- ¿Cuáles son las franjas más rentables?
- ¿Cómo evolucionan los patrones de uso a lo largo de los años?

---

