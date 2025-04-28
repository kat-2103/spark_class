# 📊 Comparativa: RDD vs DataFrame en PySpark

---

## Tabla Comparativa

| Aspecto | **RDD** | **DataFrame** |
|:---|:---|:---|
| **¿Qué es?** | Colección distribuida de objetos. | Tabla distribuida (filas y columnas). |
| **Estructura** | No estructurado (lista de objetos). | Estructurado (similar a una tabla SQL o Pandas). |
| **Optimización** | Manual, poco optimizado. | Muy optimizado automáticamente (Catalyst y Tungsten engines). |
| **Lenguaje de uso** | Operaciones funcionales: `map`, `filter`, `reduce`. | Operaciones declarativas: `select`, `filter`, `groupBy`, estilo SQL. |
| **Facilidad de uso** | Más complicado para transformaciones complejas. | Más fácil y expresivo para trabajar con datos. |
| **Uso actual** | Casos específicos de bajo nivel o transformaciones personalizadas. | **Uso recomendado** para la mayoría de proyectos modernos. |

---

##  Resumen sencillo

> "Los RDDs fueron la primera estructura de datos en Spark, pero hoy trabajamos principalmente con DataFrames porque son más fáciles de usar, más rápidos y permiten aprovechar optimizaciones automáticas."

---

##  Diagrama ilustrativo

```
            +---------+            +-------------+
            |   RDD   |            |  DataFrame   |
            +---------+            +-------------+
               ↘                           ↙
     Colección de objetos      Tabla de filas y columnas
               ↘                           ↙
  Más manual, flexible       Más optimizado, estilo SQL
```

---

#  Conclusión

- Si necesitas **trabajar con datos estructurados** (éxcel, bases de datos, CSVs, JSONs), **DataFrames** son el camino natural.
- Si necesitas **operaciones muy personalizadas a nivel de elementos individuales** y control más bajo, podrías usar **RDDs**.

**En proyectos modernos, siempre que puedas, usa DataFrames.**

---

#  Fin de la teoría RDD vs DataFrame en PySpark
