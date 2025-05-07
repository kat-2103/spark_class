# Descarga y Procesamiento de Datos de Taxis NYC (2020–2024)

## 📌 Descripción del Proyecto

Este proyecto automatiza la descarga, limpieza y agregación de datos de taxis en Nueva York entre 2020 y 2024. Utiliza **PySpark** en un entorno **Databricks** para manejar grandes volúmenes de datos y generar datasets consolidados para análisis posteriores.

---

## 🗃️ Fuentes de Datos

Los datos provienen del portal oficial de la NYC Taxi & Limousine Commission, que publica archivos mensuales de viajes para diferentes tipos de taxis:

- **Yellow Taxis**
- **Green Taxis**
- **FHV (For-Hire Vehicles)**
- **FHVHV (High Volume FHV como Uber y Lyft)**

---

## ⚙️ Proceso ETL (Extract, Transform, Load)

### 1. Descarga de Archivos

Se utilizó `BeautifulSoup` para extraer los enlaces de descarga desde la web oficial. Luego, se descargaron los archivos Parquet correspondientes a los años **2020–2024** y se almacenaron en el volumen:


---

### 2. Unión de Archivos por Tipo

Se agruparon y unieron los archivos por tipo de taxi (`yellow`, `green`, `fhv`, `fhvhv`) en un único archivo Parquet por categoría.

---

### 3. Selección y Limpieza de Columnas

Se seleccionaron columnas clave como:

- `pickup_datetime`, `dropoff_datetime`
- `passenger_count`, `trip_distance`, `fare_amount`, `total_amount`

Y se aplicaron las siguientes transformaciones:

- Conversión a tipo `timestamp`
- Filtrado de registros nulos o inválidos
- Cálculo de columnas derivadas:
- `pickup_hour`
- `pickup_month`
- `pickup_year`
- `pickup_day_of_week`

---

### 4. Análisis Agregado

Se agregaron métricas por **hora** para los taxis amarillos y verdes:

- Promedio de tarifa: `avg_fare`
- Promedio de distancia: `avg_distance`
- Total de pasajeros: `total_passengers`

---

### 5. Exportación de Resultados

Los resultados agregados se guardaron como archivos **Parquet** para su posterior análisis o visualización.

También se unieron los DataFrames limpios de taxis amarillos y verdes para análisis combinados.

---

## 📊 Columnas Derivadas

- `pickup_hour`: Hora de recogida
- `pickup_month`: Mes de recogida (formato `MM_MMMM`)
- `pickup_year`: Año de recogida (filtrado entre 2020 y 2024)
- `pickup_day_of_week`: Día de la semana
