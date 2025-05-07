# Proyecto de Análisis de Datos de NYC Taxi con PySpark en Databricks

Este proyecto realiza el scraping, limpieza, unificación y análisis mensual de datos de taxi de la ciudad de Nueva York desde Databricks, utilizando PySpark. El pipeline está diseñado para ser modular, eficiente y apto para big data, con almacenamiento en formato Delta Lake.

---

## Estructura del Proyecto

El flujo completo se divide en notebooks de Databricks modulares:

1. **`descarga_datos_nyc`**

   * Scrapea el portal oficial de NYC TLC.
   * Descarga archivos nuevos (solo los que no se encuentran en la carpeta de trabajo).

2. **`lectura_union_datasets`**

   * Lee todos los archivos locales (Parquet).
   * Une los archivos por tipo (yellow, green, fhv, fhvhv).
   * Escribe en formato Delta Lake. (Versión en desarrollo)

3. **`limpieza_transformaciones`**

   * Convierte columnas de fecha.
   * Agrega columnas derivadas como hora, mes, día de la semana y año.
   * Filtra datos nulos y viajes con pasajeros igual a 0.
   * Une yellow + green para análisis conjunto.

4. **`agregaciones_exportacion`**

   * Realiza agregaciones por hora, mes y tipo de servicio.
   * Exporta en formato Delta en una carpeta separada. (Versión en desarrollo)

---

## Tecnologías y Herramientas

* **Databricks / Apache Spark**
* **PySpark (DataFrames)**
* **Delta Lake** para almacenamiento eficiente (Versión en desarrollo)
* **Python para scraping (requests, BeautifulSoup)**
* **Programación modular y orientada a Big Data**
* **Visualizaciones con Databricks SQL o Power BI**

---

## Consideraciones de Big Data

* Uso de operaciones `unionByName` con `allowMissingColumns=True` para robustez.
* Procesamiento basado en particiones temporales por tipo y fecha. (Versión en desarrollo)
* Escritura en Delta Lake para consultas rápidas y versionado. (Versión en desarrollo)
* Lectura perezosa y carga incremental en futuras versiones.

---

## Automatización

El pipeline está pensado para ejecutarse **una vez al mes** como job en Databricks. Esto garantiza que siempre se recojan archivos nuevos sin reprocesar los anteriores.

---

## Pendientes y Extensiones Futuras

* Incluir FHV y FHVHV en la unificación final.
* Agregar almacenamiento en S3 o Blob Storage.
* Generar estadísticas avanzadas y series temporales.

---

## Autores

* Ricardo Vilariño Gestal

---

¡Listo para escalar y automatizar el análisis de transporte público en NYC! 🌆🚕

