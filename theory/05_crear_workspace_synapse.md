# 📂 05 - Crear un Workspace de Azure Synapse Analytics

---

# Objetivo

Configurar un **Azure Synapse Workspace** desde el portal de Azure, con explicaciones claras de cada servicio que vamos a usar.

---

# ¿Qué es Azure Synapse Analytics?

- **Azure Synapse Analytics** es una plataforma unificada para **análisis de datos** a gran escala.
- Combina:
  - **Data Warehousing** (almacenamiento estructurado de datos)
  - **Big Data Analytics** (procesamiento de grandes volúmenes de datos)
  - **Integración de datos** (pipelines de ingestón tipo ETL)
- Permite trabajar con:
  - **Spark Pools** (para procesamiento distribuido de datos en memoria)
  - **SQL Pools** (para consultas tipo base de datos)
  - **Pipelines** (para orquestar flujos de datos)

> En este curso nos centraremos sobre todo en **Spark Pools** para programar en **PySpark**.

---

# 📅 Servicios que vamos a usar dentro de Synapse

| Servicio | Propósito |
|:---|:---|
| **Apache Spark Pools** | Ejecutar código PySpark de forma distribuida en la nube. |
| **Synapse Notebooks** | Crear y ejecutar notebooks interactivos de PySpark. |
| **SQL Serverless Pools** | Opcional: consultas SQL ligeras sobre archivos de almacenamiento. |
| **Data Lake Storage Gen2** | Almacén de archivos en la nube que Synapse usará como "disco duro" para los datos. |


---

# Paso a paso para crear el Workspace

## 1. Ir al Portal de Azure

- Accede a [https://portal.azure.com](https://portal.azure.com)

## 2. Buscar "Synapse Analytics"

- En la barra de búsqueda, escribe **"Synapse Analytics"**.
- Clic en **"Synapse Analytics"** en los resultados.

## 3. Crear un nuevo Workspace

- Clic en **"Crear"**.

## 4. Completar la configuración básica

| Campo | Explicación |
|:---|:---|
| **Subscription** | Selecciona la suscripción gratuita que creaste. |
| **Resource Group** | Crea uno nuevo, por ejemplo `rg-synapse-demo`. |
| **Workspace Name** | Nombre único, ejemplo: `synapse-workspace-demo`. |
| **Region** | Elige la región más cercana a ti. Ej: `West Europe` si estás en España. |
| **Data Lake Storage Gen2** | Crea uno nuevo (Azure lo guiará automáticamente). |
| **File System Name** | Nombre del contenedor de datos, ejemplo `rawdata`. |

> **Importante:** Recuerda bien el nombre del storage y el filesystem.


---

## 5. Seguridad

- Puedes usar la configuración por defecto.
- No necesitas habilitar Managed Virtual Networks ni configurar firewalls complicados por ahora.

## 6. Revisar y crear

- Revisa toda la configuración.
- Pulsa **Crear**.

> El proceso de creación puede tardar unos minutos. Ten paciencia.

---

# ¿Qué recursos se crean automáticamente?

Cuando creas un Workspace Synapse, Azure también crea:

- Una cuenta de **Azure Data Lake Storage Gen2**.
- Varios roles y permisos de acceso.
- Un portal específico de **Synapse Studio** para trabajar.


---

# 🌐 Acceso al Synapse Studio

Una vez que el workspace esté listo:

1. Accede al recurso creado.
2. Clic en **Abrir Synapse Studio**.

Esto te llevará a [https://web.azuresynapse.net/](https://web.azuresynapse.net/) donde trabajarás en un entorno web exclusivo para tu Synapse Workspace.

---

# 🛠️ Siguientes pasos

Ahora que tienes tu Workspace, lo próximo será:

- Crear un **Apache Spark Pool** para ejecutar PySpark.
- Configurar Notebooks para programar directamente en la nube.

---

# Done!

Tu Azure Synapse Workspace está listo para empezar a trabajar con PySpark.

---

# 📂 Siguiente archivo: `06_crear_spark_pool_synapse.md`
