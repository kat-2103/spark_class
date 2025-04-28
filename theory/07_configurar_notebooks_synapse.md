# 📂 06 - Crear un Spark Pool en Azure Synapse Analytics

---

# 🚀 Objetivo

Configurar un **Spark Pool** dentro del Azure Synapse Workspace para ejecutar código **PySpark** en notebooks.

---

# 📅 Requisitos previos

- Haber creado un **Azure Synapse Workspace**.
- Tener acceso al **Synapse Studio** ([https://web.azuresynapse.net/](https://web.azuresynapse.net/)).

---

# 🔢 Paso a paso para crear el Spark Pool

## 1. Acceder al Synapse Studio

- Desde el portal de Azure, entra a tu Synapse Workspace.
- Haz clic en **Abrir Synapse Studio**.

## 2. Ir a "Manage"

- En la parte izquierda, haz clic en el icono de **rueda dentada** llamado **Manage**.

## 3. Crear un nuevo Spark Pool

- Dentro de "Apache Spark Pools", haz clic en **+ New** (Nuevo).

## 4. Configurar el Spark Pool

### Campos importantes a rellenar:

| Campo | Recomendación |
|:---|:---|
| **Nombre del pool** | Ejemplo: `sparkpool-demo` |
| **Node Size** | Small (4 vCores, 32 GB RAM) para pruebas |
| **Node Size Family** | Memory Optimized |
| **Autoscale** | Activado (por ejemplo, entre 3 y 5 nodos) |
| **Auto-pause** | Activado (pausar tras 15 min de inactividad) |


## 5. Opciones de Autoscaling

- **Autoscale**: permite ajustar automáticamente el número de nodos según la carga de trabajo.
- **Auto-pause**: suspende automáticamente el Spark Pool si no hay actividad, ahorrando créditos.

> 📌 Consejo: Activar estas opciones para no gastar recursos innecesariamente.


## 6. Crear el Spark Pool

- Revisa la configuración.
- Haz clic en **Create** (Crear).

---

# 📊 ¿Cuánto tarda?

La creación puede tardar entre **2 y 5 minutos**.

---

# 📚 Recordatorio importante

Cada Notebook que ejecutes deberá estar **vinculado** a este Spark Pool para poder correr código PySpark.

---

# 🌟 Resultado esperado

Tendrás disponible tu **Spark Pool** bajo el nombre que hayas elegido, listo para ejecutar código PySpark.

---

# 📂 Siguiente archivo: `07_configurar_notebooks_synapse.md
