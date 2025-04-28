# 📂 10 - Ejercicios PySpark - Enunciados

---

# 🚀 Objetivo

Practicar los conceptos aprendidos sobre **RDDs** y **DataFrames** usando **PySpark** en local o en Azure Synapse Notebooks.

---

# 📅 Instrucciones generales

- Resolver cada ejercicio en un Notebook de PySpark.
- Usar celdas Markdown para explicar brevemente cada paso.
- Usar buenas prácticas: nombres claros, código organizado.

---

# 📂 Ejercicios

## Ejercicio 1: Crear un RDD

- Crea un **RDD** que contenga los números del **1 al 20**.
- Muestra todos los elementos del RDD.


## Ejercicio 2: Filtrar múltiplos de 3

- A partir del RDD anterior:
  - Crea un nuevo RDD que contenga **solo los números múltiplos de 3**.
  - Muestra el contenido del nuevo RDD.


## Ejercicio 3: Crear un DataFrame sencillo

- Crea una lista de tuplas con la siguiente información:
  - Nombre
  - Ciudad
  - Edad

Ejemplo de datos:

```
("Ana", "Madrid", 30)
("Luis", "Sevilla", 25)
("Carlos", "Valencia", 35)
("Laura", "Barcelona", 28)
("Jorge", "Madrid", 40)
```

- Usa esa lista para crear un **DataFrame**.
- Muestra el contenido del DataFrame.


## Ejercicio 4: Filtrar personas por ciudad

- A partir del DataFrame anterior:
  - Filtra **solo las personas que viven en "Madrid"**.
  - Muestra el resultado.


## Ejercicio 5: Agrupar personas por ciudad

- A partir del DataFrame original:
  - Agrupa las personas **por ciudad**.
  - Cuenta cuántas personas hay en cada ciudad.
  - Muestra el resultado.


---

# 📚 Notas adicionales

- Usa `collect()` sólo si el volumen de datos es pequeño.
- Si trabajas en Azure Synapse, recuerda adjuntar tu Notebook al Spark Pool.


---

# 📚 Una vez terminados...

> Consulta el archivo `11_ejercicios_pyspark_soluciones.md` para ver las soluciones sugeridas.
