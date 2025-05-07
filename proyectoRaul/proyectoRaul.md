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

- Se descargan archivos .parquet mensuales de los viajes en taxi desde 2020 hasta 2024.
- Se convierten de `.parquet` a `.csv` .
- Se almacenan en una ruta local dentro del workspace de Databricks.

### 2. 🔄 Transformación

- Limpieza de valores nulos:
  - Imputación de medias en columnas numéricas: `fare_amount`, `trip_distance`, etc.
  -  Los valores nulos en columnas numéricas pueden distorsionar los análisis y modelos predictivos. Imputar (rellenar)     estos valores con la media de la columna ayuda a mantener la integridad de los datos sin introducir sesgos significativos.
- Conversión de fechas (`tpep_pickup_datetime`, `tpep_dropoff_datetime`) a formato `timestamp`.
  - Convertir las fechas a formato timestamp permite realizar operaciones de tiempo más precisas y eficientes, como cálculos de duración, filtrado por rangos de fechas y agrupaciones por hora, día, mes, etc.
- Eliminación de columnas irrelevantes: `airport_fee`, `RatecodeID`, `store_and_fwd_flag`, etc.
  -  Algunas columnas pueden no ser relevantes para el análisis o pueden contener datos redundantes. Eliminar estas columnas reduce el tamaño del DataFrame y mejora la eficiencia del procesamiento.
- Filtrado de registros inválidos: `passenger_count = 0`, `fare_amount <= 0`.
  - Los registros con valores inválidos pueden sesgar los resultados del análisis. Filtrar estos registros asegura que solo se analicen datos válidos y significativos.
- Generación de nuevas columnas:
  - Hora de recogida y destino (`hora_int_pickup`, `hora_int_dropoff`)
  - Extraer la hora de las fechas de recogida y destino permite analizar patrones horarios, como identificar las horas pico de demanda.

### 3. 📊 Agregación por hora

Se generan tres tablas Delta con las siguientes métricas agrupadas por hora:

| Tabla | Métricas |
|-------|----------|
| `viajes_por_hora` | Distancia promedio, duración promedio, tarifa base y total promedio |
| `pasajeros_df` | Total de pasajeros, cantidad de viajes, promedio de pasajeros por viaje |
| `ingresos_por_hora` | Ingresos totales, cantidad de viajes, ingreso promedio por viaje |
| `ingresos_por_franja` | Franja horaria, ingresos totales, cantidad de viajes, ingreso promedio por viaje |

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
- 
![image](https://github.com/user-attachments/assets/b4d4b4b5-d7d9-4909-a98a-32482b978ef2)

- Total de pasajeros por hora

  ![image](https://github.com/user-attachments/assets/e4ea880c-4304-43c4-ba82-eabb968f615b)

- Distancia media recorrida por hora

![image](https://github.com/user-attachments/assets/8b899b4c-9ac7-4c23-868c-c068dd89fce2)

- Ingresos totales por franja horaria

![image](https://github.com/user-attachments/assets/5b41b680-7fad-4627-a316-be6575ff2b82)


### ⏱ Automatización (Opcional)

El proceso puede ser automatizado usando Jobs en Databricks para ejecutar periódicamente el pipeline y actualizar los datos sin intervención manual. Se crearon dos tareas distintas:

#### Tarea de Ingesta de Datos

- Esta tarea ejecuta el notebook de ingesta de datos, descargando y almacenando los archivos `.parquet` mensuales de los viajes en taxi.

#### Tarea de Transformación ETL

- Esta tarea ejecuta el notebook de transformación ETL, aplicando las limpiezas, transformaciones y agregaciones necesarias a los datos.

Ambas tareas están configuradas con triggers para ejecutarse de manera periódica, asegurando que los datos se mantengan actualizados sin necesidad de intervención manual.

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

- Las horas nocturnas son las horas en las que hay mas promedio de tarifa.
- La franja de tarde genera más ingresos totales debido a que hay una mayor cantidad de viajes.
- Por la tarde es cuando mas cantidad de pasajeros hay concretamente a las 18:00.
- Cuando mas distancia promedio hay es en la madrugada.
---

## 👥 Autores

- Raul Agras Basanta

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo LICENSE para más detalles.
