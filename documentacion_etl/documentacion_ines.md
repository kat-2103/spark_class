# Proceso ETL - NYC Taxi Data 🚖  

##  Introducción  

Este documento describe el proceso de extracción, transformación y carga (**ETL**) aplicado a los datos de **NYC Taxi Data**, almacenados en un volumen de **DataBricks**, con el objetivo de generar **tablas finales para dashboards**.  

## 1️⃣ Carga de Datos  

Los datos se cargan desde múltiples archivos **CSV** almacenados en un volumen de **DataBricks**.  

## 2️⃣ Transformaciones  

Se aplican las siguientes transformaciones para garantizar la calidad y utilidad de los datos:  

- **Eliminación de viajes sin pasajeros**.  
- **Imputación de distancia igual a 0** mediante la mediana.  
- **Transformación de fechas** a tipo `Timestamp`.  
- **Creación de variables nueva a partir del campo 'tpep_pickup_datetime
'**: `pickup_year`, `pickup_month`, `pickup_day`, `pickup_hour`.  
- **Filtrado de años inválidos** en `pickup_year`, eliminando registros fuera del rango 2020-2024 (aproximadamente 1000 registros).  
- **Transformación de día y mes numéricos** a palabra en español.  
- **congestion_surcharge, fare_amount y tips_amount negativos** pasados a positivo.  
- **Enriquecimiento con datos de ubicación**: Se incorpora información de distrito y zona (`PU_Borough`, `PU_Zone`, `DO_Borough`, `DO_Zone`) mediante un `JOIN` con un dataset de ubicaciones (`PULocationID`, `DOLocationID`). Además creación de un campo 'Borough_concat', que une el PU_Borough con el DO_Borough, para tener los desplazamientos entre distritos.  

## 3️⃣ Resultado Final   

Las transformaciones generan **tres tablas en formato Delta**, con los siguientes esquemas:  

### Tabla 1: `generalInsights`  
| Campo | Descripción |  
|-----------------|--------------------------------------|  
| `pickup_year`   | Año de recogida del viaje  |  
| `pickup_month`  | Mes de recogida del viaje  |  
| `pickup_hour`   | Hora de recogida del viaje  |  
| `avg_fare`  | Tarifa promedio  |  

| `avg_distance`  | Distancia promedio de viajes  |  
| `total_passengers` | Número total de pasajeros  |  

### Tabla 2: `passengersBehaviour`  
| Campo | Descripción |  
|-----------------|--------------------------------------|  
| `total_passengers` | Número total de pasajeros  |  
| `avg_tips`      | Propinas promedio  |  
| `pickup_hour`   | Hora de recogida del viaje  |  
| `pickup_day`    | Día de recogida del viaje  |  

### Tabla 3: `tripsLocation`  
| Campo | Descripción |  
|-----------------|--------------------------------------|  
| `Borough_Concat` | Concatenación de distritos |  
| `DO_Zone`      | Zona de destino |  
| `PU_Zone`      | Zona de origen |  
| `DO_Borough`   | Distrito de destino |  
| `PU_Borough`   | Distrito de origen |  
| `pickup_day`   | Día de recogida del viaje |  

## 4️⃣ Pipeline de Ejecución 

El proceso se automatiza mediante un **pipeline** que ejecuta el **notebook con el ETL** con las siguientes características:  

 **Frecuencia:** Mensual (5 de cada mes), a las **3:00 AM**.  
 **Notificaciones:** Se envían un correo electrónico notificando éxito ✅ o error ❌ en la ejecución.  

---


