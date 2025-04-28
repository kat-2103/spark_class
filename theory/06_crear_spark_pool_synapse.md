# 📂 06 - Crear un Apache Spark Pool en Azure Synapse

---

# 🚀 Objetivo

Crear un **Apache Spark Pool** en tu **Azure Synapse Workspace** para poder ejecutar **notebooks de PySpark**.

---

# 📚 ¿Qué es un Spark Pool?

- Un **Spark Pool** es un conjunto de recursos (CPU, memoria) que Synapse reserva para que puedas **ejecutar código PySpark** en paralelo y a gran escala.
- Puedes tener varios pools configurados para diferentes tipos de cargas de trabajo.

> **En este curso:** Vamos a crear un Spark Pool pequeño, para trabajar sin generar costos altos.

---

# 📅 Requisitos

- Tener un **Workspace Synapse** creado (ver `05_crear_workspace_synapse.md`).

---

# 🔢 Paso a paso para crear el Spark Pool

## 1. Entrar al Azure Portal

- Accede a [https://portal.azure.com](https://portal.azure.com).
- Busca tu **Workspace Synapse**.

## 2. Dentro del Workspace, buscar "Apache Spark pools"

- En el menú izquierdo del workspace, clic en **"Apache Spark pools"**.

## 3. Crear un nuevo Spark Pool

- Clic en **"+ Nuevo"**.

---

# 📊 Configuración recomendada para el Spark Pool

| Campo | Explicación | Ejemplo |
|:---|:---|:---|
| **Nombre del pool** | Nombre identificativo. | `spark-pool-demo` |
| **Version de Apache Spark** | Selecciona la más reciente disponible. | `3.3` o superior |
| **Node Size** | Tamaño de las máquinas virtuales. | `Small (4 vCores / 32 GB RAM)` |
| **Number of Nodes** | Cantidad de nodos iniciales. | `3` |
| **Auto-Scale** | Activar si quieres que Azure ajuste automáticamente el número de nodos. | Activado |
| **Auto-Pause** | Activar para pausar automáticamente cuando no uses el pool. | Activado, 15 minutos |


---

# 🛠️ Opciones avanzadas

En este proyecto no necesitamos tocar configuraciones avanzadas, pero en proyectos reales podrías configurar:

- Librerías adicionales (por ejemplo, mllib para machine learning).
- Entornos personalizados.
- Tags para control de costos y organización.

---

# 📝 Confirmar y crear

- Revisa toda la configuración.
- Haz clic en **"Crear"**.

> 🔸 El Spark Pool puede tardar varios minutos en desplegarse. Paciencia.

---

# 💡 Consejos importantes

- **Nombre simple**: Usa nombres fáciles de recordar (`spark-pool-demo` mejor que `katya-cluster-001`).
- **Auto-pause activado**: Evitará gastos innecesarios cuando no estés trabajando.
- **Auto-scale activado**: Ahorra costos y optimiza rendimiento automáticamente.

---

# 📂 Resultado esperado

- Un Spark Pool disponible dentro de tu Azure Synapse Workspace.
- Ahora puedes usarlo al crear y ejecutar Notebooks de PySpark.

---

# 📚 Resumen visual del flujo

```
Workspace Synapse → Apache Spark Pools → Nuevo Pool → Configurar → Crear
```

---

# 🚀 Siguiente paso

Configurar tu primer **Notebook de PySpark** y asociarlo a tu nuevo **Spark Pool**.

**📂 Sigue con el archivo: `07_configurar_notebooks_synapse.md`**
