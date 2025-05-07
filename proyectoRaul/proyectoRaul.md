# Análisis de Viajes en Taxi de Nueva York 🚖 NYC Yellow Taxi Trips

> Este proyecto analiza los datos históricos de los viajes en taxi amarillo de la ciudad de Nueva York con el objetivo de identificar patrones horarios, optimizar recursos y tomar decisiones basadas en datos.

---

## 📌 Descripción del Proyecto

Este proyecto realiza un **pipeline ETL completo** desde la ingesta de datos hasta la generación de métricas clave sobre los viajes en taxi. Los datos provienen de la TLC (Taxi and Limousine Commission), y se procesan utilizando **Apache Spark** en **Databricks Community Edition**, almacenando resultados en **Delta Lake** y exponiendo dashboards interactivos.

---

## 🔧 Tecnologías Utilizadas

- **Databricks Community Edition**
- **Apache Spark / PySpark**
- **Delta Lake**
- **Python (Pandas, PySpark SQL)**
- **SQL (para consultas en Databricks)**

---

## 🗺️ Pipeline ETL

### 1. 💾 Ingesta de Datos

- Se descargan archivos `.csv` mensuales de los viajes en taxi.
- Se convierten de `.csv` a `.parquet` (formato optimizado).
- Se almacenan en una ruta local dentro del workspace de Databricks.

### 2. 🔄 Transformación

- Limpieza de valores nulos:
  - Imputación de medias en columnas numéricas: `fare_amount`, `trip_distance`, etc.
- Conversión de fechas (`tpep_pickup_datetime`, `tpep_dropoff_datetime`) a formato `timestamp`.
- Eliminación de columnas irrelevantes: `airport_fee`, `RatecodeID`, `store_and_fwd_flag`, etc.
- Filtrado de registros inválidos: `passenger_count = 0`, `fare_amount <= 0`.
- Generación de nuevas columnas:
  - Hora de recogida y destino (`hora_int_pickup`, `hora_int_dropoff`)
  - Duración del viaje en minutos (`trip_duration_minutes`)

### 3. 📊 Agregación por hora

Se generan tres tablas Delta con las siguientes métricas agrupadas por hora:

| Tabla | Métricas |
|-------|----------|
| `viajes_por_hora` | Distancia promedio, duración promedio, tarifa base y total promedio |
| `pasajeros_df` | Total de pasajeros, cantidad de viajes, promedio de pasajeros por viaje |
| `ingresos_por_hora` | Ingresos totales, cantidad de viajes, ingreso promedio por viaje |

Además, se categorizan las horas en franjas horarias:
- Mañana (6-11)
- Mediodía (12-14)
- Tarde (15-19)
- Noche (20-23)
- Madrugada (0-5)

### 4. 🗃 Almacenamiento

Los datos transformados se guardan como tablas Delta para permitir versionado, actualización incremental y mejor rendimiento en consultas posteriores.

```python
viajes_por_hora.write.format('delta').mode('overwrite').saveAsTable('workspace.default.viajes_por_hora')
```

### 📊 Dashboards Interactivos

En Databricks, se crearon dashboards interactivos con visualizaciones clave:

- Promedio de tarifas por hora del día
- Total de pasajeros por hora
- Distancia media recorrida por hora
- Ingresos totales por franja horaria

### ⏱ Automatización (Opcional)

El proceso puede ser automatizado usando Jobs en Databricks para ejecutar periódicamente el pipeline y actualizar los datos sin intervención manual.

---

## 🧪 Cómo Ejecutar el Proyecto

1. Acceder a Databricks Community Edition: Databricks Community Edition
2. Crear un notebook en PySpark
3. Ejecutar el código paso a paso:
   - Instalar dependencias (si es necesario): `%pip install pyspark`
   - Convertir `.parquet` a `.csv`
   - Leer los archivos con Spark
   - Aplicar limpieza, transformación y agregaciones
   - Guardar en Delta Lake
   - Visualizar resultados
4. Crear Dashboards:
   - Usar botón + Add to Dashboard en cada gráfico
   - Crear un dashboard llamado NYC Taxi Dashboard

---

## 🧠 Insights Clave

- Las horas pico suelen tener mayor número de viajes y mayores ingresos.
- La franja nocturna genera más ingresos totales debido a tarifas nocturnas o viajes más largos.
- En horas de madrugada hay menos viajes pero mayor tiempo promedio de trayecto.

---

## 👥 Autores

- Raul Agras Basanta

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo LICENSE para más detalles.
