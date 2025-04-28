# 📂 08 - Primer Notebook de PySpark en Azure Synapse Analytics

---

# 🚀 Objetivo

Crear y ejecutar un **primer ejercicio práctico de PySpark** en un **Notebook** en Azure Synapse Analytics, usando un **Spark Pool**.

---

# 📅 Requisitos previos

- Tener un Synapse Workspace creado.
- Tener un Spark Pool creado y disponible.
- Haber configurado un Notebook y adjuntado el Spark Pool.


---

# 🔢 Ejercicio paso a paso

## 1. Crear el DataFrame de prueba

En una celda de tu Notebook, asegúrate de que esté seleccionado el lenguaje **PySpark** y ejecuta:

```python
# Crear una lista de datos de ejemplo
datos = [
    ("Ana", "Madrid", 30),
    ("Luis", "Sevilla", 25),
    ("Carlos", "Valencia", 35),
    ("Laura", "Barcelona", 28),
    ("Jorge", "Madrid", 40)
]

# Definir columnas
columnas = ["Nombre", "Ciudad", "Edad"]

# Crear DataFrame
personas_df = spark.createDataFrame(datos, columnas)

# Mostrar el DataFrame
personas_df.show()
```

---

## 2. Filtrar personas mayores de 30 años

```python
# Filtrar personas mayores de 30
df_mayores_30 = personas_df.filter(personas_df.Edad > 30)

# Mostrar el resultado
df_mayores_30.show()
```

**Resultado esperado:**

```
+-------+--------+----+
| Nombre| Ciudad |Edad|
+-------+--------+----+
|Carlos |Valencia|  35|
| Jorge | Madrid |  40|
+-------+--------+----+
```

---

## 3. Agrupar por ciudad y contar personas

```python
# Agrupar por Ciudad y contar
conteo_ciudades = personas_df.groupBy("Ciudad").count()

# Mostrar resultado
conteo_ciudades.show()
```

**Resultado esperado:**

```
+----------+-----+
|   Ciudad |count|
+----------+-----+
| Barcelona|    1|
| Valencia |    1|
|   Madrid |    2|
|  Sevilla |    1|
+----------+-----+
```

---

# 💡 Extra: Ordenar los resultados

```python
# Ordenar por edad descendente
personas_df.orderBy(personas_df.Edad.desc()).show()
```

**Resultado esperado:**

```
+-------+--------+----+
| Nombre| Ciudad |Edad|
+-------+--------+----+
| Jorge | Madrid |  40|
|Carlos |Valencia|  35|
| Ana   | Madrid |  30|
|Laura  |Barcelona|28|
| Luis  |Sevilla | 25|
+-------+--------+----+
```

---

# 📈 Flujo visual del Notebook

```
Crear DataFrame → Filtrar datos → Agrupar datos → Ordenar resultados
```


---

# 🚀 Comentarios importantes

- Cada operación que haces genera un nuevo DataFrame.
- No se modifican los datos originales a menos que lo especifiques.
- Spark maneja los datos de manera distribuida aunque no lo veas en pequeños ejemplos.


---

# 📚 Conclusión

👉 Ya sabes:
- Crear DataFrames en PySpark.
- Aplicar **transformaciones** (`filter()`, `groupBy()`, `orderBy()`).
- Ejecutar **acciones** (`show()`) para visualizar resultados.

> *"Con este primer ejercicio, has ejecutado procesamiento distribuido en la nube usando PySpark y Azure Synapse."*

---

# 👏 ¡Felicidades!

Has completado tu primer proyecto real en PySpark usando Azure Synapse Analytics.

---

# 📂 Siguiente archivo: `09_best_practices_synapse.md` (Buenas prácticas trabajando en Synapse)
