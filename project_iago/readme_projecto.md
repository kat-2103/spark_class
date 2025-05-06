# 📊 Análisis de Datos de Taxis - Proyecto Spark

Este repositorio contiene un notebook de Databricks que realiza una **limpieza, transformación y análisis exploratorio de datos** sobre un conjunto de archivos CSV relacionados con viajes en taxi amarillo (Yellow Taxi Trip Records).

## 📁 Estructura del Proyecto

- `notebook.ipynb` - El notebook principal donde se ejecuta todo el procesamiento.
- `data/` - Carpeta donde se encuentran los archivos de entrada y salida (`csv/yellow_*.csv`, `csv/clean/`).
- `README.md` - Documentación actual.

---

## 🧾 Descripción del Código

### 1. 🔍 Carga de Datos
- Se cargan todos los archivos CSV (`yellow_*.csv`) desde la ruta especificada.
- Se infiere automáticamente el esquema y se lee el encabezado.

### 2. 🧹 Limpieza de Datos
- Se eliminan columnas irrelevantes o poco útiles como: `airport_fee`, `RatecodeID`, `congestion_surcharge`, etc.
- Se identifican y manejan valores nulos reemplazándolos por la media de cada columna numérica:
  - Columnas afectadas: `passenger_count`, `trip_distance`, `fare_amount`, `total_amount`, `tip_amount`.
- Se convierten tipos de datos a formatos adecuados (ej. `DoubleType`, `IntegerType`).

## ⚠️ Nota sobre Outliers

En este análisis **no se ha realizado una limpieza explícita de outliers**, por las siguientes razones:

- **Representatividad de casos reales**: muchos valores extremos (como viajes muy largos o tarifas altas) pueden representar situaciones legítimas dentro del servicio de taxis.
- **Objetivo exploratorio del proyecto**: el enfoque está puesto en obtener una visión general de los patrones de uso, sin necesidad de eliminar datos para modelado predictivo.
- **Escalabilidad**: dado que los datos pueden ser muy grandes, la detección y tratamiento de outliers puede implicar un costo computacional elevado si no se justifica por el objetivo del análisis.
- **Posibilidad de análisis posterior**: el tratamiento de outliers se deja pendiente para etapas posteriores, en caso de requerirse para modelos estadísticos o de machine learning.

Si en una fase futura se requiere un análisis más riguroso o entrenamiento de modelos, se recomienda aplicar técnicas como el rango intercuartílico (IQR), z-score u otros métodos de detección de anomalías.

---
### 3. 🕒 Transformaciones Temporales
- Se convierten las columnas de fecha (`tpep_pickup_datetime`, `tpep_dropoff_datetime`) a tipo `Timestamp`.
- Se extrae la hora de ambas fechas (`hora_pickup`, `hora_dropoff`) y se convierten a enteros.
- Se eliminan las columnas temporales redundantes (`hora_pickup`, `hora_dropoff`).

### 4. ⏱️ Duración del Viaje
- Se crea una nueva columna `trip_duration_minutes` calculando la diferencia entre las horas de recogida y entrega.

### 5. 💳 Mapeo de Tipos de Pago
- Se mapea el campo `payment_type` a nombres descriptivos (ej. "Tarjeta de crédito", "Efectivo").
- Se une esta información al DataFrame original mediante un `join`.

### 6. 📅 Fechas y Horarios
- Se extraen nuevas columnas:
  - `dia_semana`: día de la semana (1-7).
  - `nombre_dia`: nombre del día (ej. "Monday").
  - `mes`: número del mes (1-12).
  - `nombre_mes`: nombre del mes (ej. "January").

### 7. 📈 Agregaciones
Se generan varios DataFrames agrupados por hora de inicio del viaje (`hora_int_pickup`):
- **Pasajeros por hora**: total y promedio de pasajeros.
- **Viajes por hora**: distancia, duración, tarifas promedio.
- **Ingresos por hora**: ingresos totales, cantidad de viajes y promedio por viaje.
- **Franjas horarias**: categorización en Mañana, Tarde, Noche, etc., con agregaciones adicionales.

### 8. 📥 Exportación
- Los DataFrames limpios y agregados se exportan como archivos `.csv` en la carpeta `/Volumes/workspace/default/prueba/csv/clean/`.
- El DataFrame final también se guarda como tabla Delta en Databricks (`workspace.default.df_final`).

---


## 🗂️ ¿Qué es Delta Lake?

[Delta Lake](https://delta.io/) es un proyecto open-source construido sobre Apache Spark que permite trabajar con datos estructurados y versionados en entornos de Big Data. Algunas ventajas clave de Delta Lake son:

- **Soporte ACID Transactions**: garantiza consistencia e integridad en escrituras concurrentes.
- **Metadatos mejorados**: facilita la gestión eficiente de grandes volúmenes de datos.
- **Versionado de datos**: permite retroceder en el tiempo a versiones anteriores del dataset.
- **Compatibilidad con Spark SQL**: integración nativa con herramientas de consulta y procesamiento.

En este proyecto, se ha utilizado Delta Lake para almacenar el dataset final de forma estructurada y escalable, permitiendo su fácil acceso futuro para análisis o integración en pipelines.

---

## 📊 Dashboards y Visualizaciones

Como parte del análisis, se han creado **dashboards interactivos** para visualizar los siguientes **insights clave**:

| Insight | Descripción |
|--------|-------------|
| **Promedio de tarifa por hora del día** | Permite observar cómo varía la tarifa base según la hora, lo cual puede estar influenciado por demanda o zonas de mayor actividad. |
| **Total de pasajeros por hora** | Muestra picos de uso del servicio en diferentes momentos del día, útil para planificación operativa. |
| **Distancia media recorrida por hora** | Ayuda a entender patrones de movilidad urbana y destinos típicos según la hora. |
| **Total de tarifa por día de la semana** | Identifica días con mayores ingresos y comportamientos estacionales en el uso del servicio. |

Estos dashboards pueden implementarse utilizando herramientas como **Power BI**, **Tableau**, **Databricks SQL**, o incluso **Dash + Plotly** si se trabaja en un entorno Python.

---
## 📋 Requisitos

Este proyecto está desarrollado en **Databricks** y utiliza **Apache Spark** con Python (PySpark). Para replicarlo necesitas:

- Acceso a un entorno Databricks.
- Archivos CSV de Yellow Taxi disponibles en la ruta especificada.
- Permisos para escribir en rutas de almacenamiento y crear tablas Delta.

---

## 🔄 Automatización con Triggers

Para hacer el proceso más robusto y operativo, se han creado **dos triggers** dentro del entorno de Databricks:

### ✅ Trigger de Ingestión de Datos
- **Objetivo:** Ejecutar automáticamente la carga de nuevos archivos CSV cuando estos lleguen a la carpeta de origen.
- **Tipo:** *Event-based trigger* (por ejemplo, basado en la llegada de nuevos archivos a una ubicación en DBFS o Cloud Storage).
- **Uso:** Ideal para mantener los datos actualizados sin intervención manual cada vez que hay nuevos registros.

### ✅ Trigger de Transformación de Datos
- **Objetivo:** Ejecutar automáticamente el proceso de limpieza, transformación y agregación una vez que se completa la ingestión.
- **Tipo:** *Scheduled trigger* (programado diariamente, semanalmente o bajo demanda).
- **Uso:** Permite refrescar los datasets limpios y las agregaciones para alimentar dashboards o reportes periódicos.

---
## ⚠️ Nota sobre Pipelines

En este proyecto **no se han implementado pipelines formales**, ya que:

- **Es una fase exploratoria**: el objetivo principal ha sido analizar y entender los datos, no crear una solución operativa a largo plazo.
- **El volumen de datos es moderado**: aún siendo Big Data, el alcance actual no requiere una arquitectura compleja ni orquestación avanzada.
- **Se priorizó la simplicidad**: el uso de notebooks y triggers permite un desarrollo rápido y comprensible sin necesidad de múltiples herramientas intermedias.
- **No se requiere escalabilidad inmediata**: si el proyecto crece o se industrializa, entonces sí sería recomendable migrar a un sistema de pipelines.
---

## 📌 Resultados Generados

| DataFrame              | Descripción                                     |
|-----------------------|-------------------------------------------------|
| `df`                  | Dataset limpio y transformado listo para análisis |
| `pasajeros_df`        | Total de pasajeros y viajes por hora             |
| `viajes_por_hora`     | Estadísticas promedio por hora                   |
| `ingresos_por_hora`   | Ingresos por hora                                |
| `franjas_horarias`    | Ingresos divididos en franjas horarias          |
| `ingresos_por_franja` | Resumen de ingresos por franja horaria           |

---

## ✅ Notas Adicionales

- Todas las transformaciones están pensadas para ser escalables usando PySpark.
- Las estadísticas son ideales para visualizar patrones de uso del servicio de taxis durante diferentes horas del día y días de la semana.
- Este proceso puede adaptarse fácilmente para incluir más datasets o integrarse en pipelines automatizados.

---
