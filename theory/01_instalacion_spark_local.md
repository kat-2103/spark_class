# 📂 01 - Instalación de Apache Spark en Local

---

# Objetivo

Aprender a instalar **Apache Spark** y preparar un entorno local para ejecutar código **PySpark** en tu propio ordenador.

---

# Requisitos previos

- Tener **Python 3.x** instalado.
- Tener acceso a la línea de comandos o terminal.
- Opcionalmente, tener instalado **Jupyter Notebook** o **VSCode** para programar.

---

# 💡 Paso 1: Instalar Java

Apache Spark necesita **Java** para funcionar.

## Instalación en Windows o Mac
- Descargar e instalar [OpenJDK 8 o superior](https://adoptium.net/temurin/releases/).

## Verificar instalación
Abre una terminal y escribe:

```bash
java -version
```

Deberías ver algo como:

```bash
openjdk version "11.0.x"
```

> Si no lo detecta, recuerda configurar la variable de entorno `JAVA_HOME`.

---

# 💡 Paso 2: Instalar Apache Spark

1. Ve a la página oficial de descargas: [https://spark.apache.org/downloads.html](https://spark.apache.org/downloads.html)
2. Descarga la última versión estable de Spark.
3. Elige una versión precompilada para Hadoop 3.x (aunque no usarás Hadoop en local).
4. Descomprime el archivo `.tgz` o `.zip` en una carpeta de tu ordenador.

> Guarda la ruta donde descomprimiste Spark, la necesitarás.

---

# 💡 Paso 3: Configurar Variables de Entorno

Configura el acceso a Spark desde cualquier terminal:

## En Windows:
- Crea una variable de entorno llamada `SPARK_HOME` apuntando a la carpeta donde descomprimiste Spark.
- Añade `%SPARK_HOME%\bin` al `PATH`.

## En Linux/Mac:
Edita tu archivo `.bashrc` o `.zshrc`:

```bash
export SPARK_HOME=/ruta/a/tu/spark
export PATH=$SPARK_HOME/bin:$PATH
```

Aplica los cambios:

```bash
source ~/.bashrc
```

---

# 💡 Paso 4: Instalar PySpark

Instala PySpark como un paquete de Python usando `pip`:

```bash
pip install pyspark
```

> Esto instalará todo lo necesario para usar Spark desde Python.

---

# 💡 Paso 5: Probar que Spark funciona

En tu terminal, ejecuta:

```bash
pyspark
```

Si ves algo como:

```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\
      /_/
```

👏 ¡Felicidades! Spark está funcionando.

Para salir, escribe:

```bash
exit()
```

---

# 💡 Paso 6 (Opcional): Instalar Jupyter Notebook

Para trabajar de forma más ordenada puedes usar Jupyter Notebooks:

```bash
pip install notebook
```

Levanta un servidor local:

```bash
jupyter notebook
```

Desde ahí podrás crear Notebooks para programar en PySpark.

---

# Resumen rápido

- Instala Java ✅
- Descarga y configura Spark ✅
- Instala PySpark ✅
- Prueba `pyspark` en terminal ✅
- (Opcional) Usa Jupyter Notebooks para un entorno más amigable ✅

---

> *"Trabajar en local es el primer paso antes de escalar tu conocimiento a la nube."*
