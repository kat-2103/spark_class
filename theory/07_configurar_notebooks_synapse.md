# 📂 07 - Configurar Notebooks en Azure Synapse Analytics

---

# 🚀 Objetivo

Aprender a crear, configurar y preparar un **Notebook** en **Azure Synapse Studio** para programar con **PySpark**.

---

# 📚 ¿Qué es un Notebook en Synapse?

- Un **Notebook** es un documento interactivo que contiene:
  - Código ejecutable (PySpark, SparkSQL, Scala, C#).
  - Resultados de ejecución.
  - Comentarios en Markdown.
- Permite programar y ver resultados **directamente en la nube**.

> En Synapse, los Notebooks son la forma más rápida y visual de trabajar con Spark.

---

# 🔢 Paso a paso para crear un Notebook

## 1. Entrar a Synapse Studio

- Accede a tu Synapse Workspace.
- Abre el **Synapse Studio** ([https://web.azuresynapse.net/](https://web.azuresynapse.net/)).

## 2. Crear un nuevo Notebook

- En el menú izquierdo, haz clic en **Develop**.
- Clic en **+ Notebook**.

## 3. Asignar un Spark Pool al Notebook

- Arriba del Notebook, verás la opción "**Attach to**".
- Selecciona el Spark Pool que creaste (ej: `sparkpool-demo`).

> 📌 **Importante:** Sin Spark Pool, el Notebook no podrá ejecutar código PySpark.

## 4. Configurar el lenguaje

- En cada celda, puedes elegir el lenguaje:
  - **PySpark** (Python)
  - **Spark SQL** (SQL)
  - **Scala**
  - **.NET for Apache Spark (C#)**

- Selecciona **PySpark** para programar en Python.


## 5. Primeras pruebas en el Notebook

### Crear una sesión

Al escribir tu primer código y darle a "Run", el Spark Pool iniciará una sesión.

Ejemplo de código PySpark para probar:

```python
# Crear un DataFrame sencillo
datos = [("Ana", 30), ("Luis", 25), ("Carlos", 35)]
columnas = ["Nombre", "Edad"]

# Crear DataFrame
df = spark.createDataFrame(datos, columnas)

# Mostrar los datos
df.show()
```

### Resultado esperado:

```
+-------+----+
| Nombre|Edad|
+-------+----+
|    Ana|  30|
|   Luis|  25|
| Carlos|  35|
+-------+----+
```

> 🔸 Si ves este resultado, tu Notebook y Spark Pool están funcionando correctamente.

---

# 📊 Buenas prácticas con Notebooks

- **Pon títulos** claros en cada Notebook.
- **Divide** tu código en varias celdas pequeñas.
- **Anota** tus notebooks usando celdas Markdown para explicar procesos.
- **Detén tu sesión** si no estás trabajando para ahorrar recursos.


---

# 🛠️ Opciones adicionales

- Puedes guardar tu Notebook en el almacenamiento del Workspace.
- Puedes exportarlo como archivo `.ipynb` compatible con Jupyter.


---

# 🌟 Resultado esperado

- Tienes un Notebook conectado a un Spark Pool listo para trabajar.
- Has ejecutado tu primer código PySpark en la nube.

---

# 📂 Siguiente archivo: `08_primer_notebook_synapse.md`
