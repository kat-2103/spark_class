# 📂 09 - Buenas Prácticas Trabajando en Azure Synapse Analytics

---

# 🚀 Objetivo

Adoptar buenas prácticas al usar Azure Synapse Analytics para:
- Optimizar el rendimiento.
- Evitar costes innecesarios.
- Trabajar de manera ordenada y profesional.

---

# 📊 1. Control de Costes

- **Configura Auto-pause** en Spark Pools (pausar tras 15 minutos de inactividad).
- **Usa Autoscale** para ajustar el número de nodos según carga de trabajo.
- **Detén manualmente Spark Sessions** cuando termines (`Stop session` en Synapse Studio).
- **Revisa periódicamente el coste en Azure Portal**.

> 🔸 **Regla de oro:** No dejes Spark Pools encendidos si no los estás usando.

---

# 📊 2. Organización de Recursos

- **Nombre consistente para Spark Pools** (ej: `sparkpool-desarrollo`, `sparkpool-pruebas`).
- **Nombre claro para Notebooks**:
  - Prefijo de proyecto + descripción breve (ej: `ETL_IngestionVentas`, `Transform_Clientes`).
- **Agrupa los Notebooks** en carpetas temáticas.
- **Versiona Notebooks**: guarda versiones importantes (puedes exportarlos `.ipynb` si quieres).

---

# 📊 3. Mejores Prácticas en Código PySpark

- **Usa nombres claros** de variables y columnas.
- **Divide el código en celdas pequeñas**: facilita la lectura y la depuración.
- **Agrega celdas Markdown** explicando el objetivo de cada parte.
- **Evita `collect()` sobre grandes volúmenes de datos**:
  - Traer muchos datos al driver puede saturar la memoria.
  - Prefiere usar `show()`, `take()`, `limit()`, etc.

---

# 📊 4. Buenas prácticas de Sesiones

- **Adjunta el Notebook al Spark Pool adecuado**.
- **No abras múltiples sesiones en paralelo** si no es necesario.
- **Planifica cargas de trabajo**: agrupa ejecuciones para aprovechar la sesión activa.


---

# 📊 5. Seguridad y Accesos

- Usa **roles y permisos** para controlar quién puede leer/escribir datos.
- No compartas información sensible en Notebooks.
- Configura accesos limitados a Data Lake si trabajas en equipo.


---

# 📊 6. Uso del Almacenamiento

- Utiliza **Data Lake Storage Gen2** como repositorio central de archivos.
- Crea una **estructura de carpetas** lógica: `/landing`, `/raw`, `/curated`.
- Prefiere formatos eficientes:
  - **Parquet** sobre **CSV** o **JSON** (mejor compresión y velocidad).


---

# 📚 Resumen de Buenas Prácticas

| 🔢 Categoría | 🔗 Buenas Prácticas |
|:---|:---|
| Costes | Autoscale, Auto-pause, controlar sesiones |
| Organización | Naming consistente, carpetas temáticas |
| Código | Nombres claros, celdas pequeñas, no usar `collect()` salvo necesario |
| Seguridad | Roles, no compartir info sensible |
| Almacenamiento | Data Lake estructurado, usar formatos Parquet |


---

# 🌟 Conclusión

Aplicando estas buenas prácticas, lograrás:
- Trabajar de forma **profesional**.
- **Optimizar costes**.
- **Mantener tu proyecto ordenado** y escalable.

> *"Trabajar bien en Synapse no solo es programar, sino hacerlo de manera eficiente y responsable."* 🚀

---

# 👏 ¡Felicidades!

Has completado la configuración básica para trabajar con PySpark en Azure Synapse Analytics de forma correcta y profesional.
