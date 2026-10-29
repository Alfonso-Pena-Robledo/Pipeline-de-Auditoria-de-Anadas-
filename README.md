# 🍇 Pipeline de Auditoría de Añadas 

Este proyecto consiste en un pipeline de datos desarrollado en **Python (Google Colab)** para automatizar la auditoría de inventario y reconciliación de productos (añadas de vinos/licores) entre el **Catálogo Web (CMS)** y los **Recuentos Físicos de Tiendas**.

El sistema procesa, valida, normaliza y compara ambos conjuntos de datos para identificar discrepancias de stock y generar automáticamente informes visuales y de acción recomendada.

## 🛠️ Tecnologías y Librerías Utilizadas

El proyecto utiliza el stack estándar de Data Science en Python:

* **Python 3** (Entorno Google Colab)

* **Pandas**: Carga, limpieza, cruce (`merge`) y manipulación de DataFrames.

* **NumPy**: Manejo de valores nulos (`NaN`) y operaciones vectoriales.

* **Matplotlib & Seaborn**: Generación de visualizaciones estáticas y gráficos estadísticos.

* **google.colab.files**: Interfaz interactiva para la subida de archivos CSV en el entorno Colab.

* **datetime & zoneinfo**: Gestión de marcas de tiempo y zonas horarias (`Europe/Madrid`).

## 📋 Requisitos de Entrada (Archivos CSV)

El script solicita interactivamente dos archivos CSV. Cada uno debe cumplir con un esquema de columnas específico:

### 1. Catálogo Web (`df_catalogo`)

Contiene los datos de los productos publicados en la plataforma e-commerce:

* `SKU`: Identificador único del producto (cadena de texto).

* `Producto en la web`: Nombre del producto publicado (incluye el año de la añada al final).

* `PVP`: Precio de Venta al Público (€).

### 2. Recuento en Tiendas (`df_recuento`)

Contiene los datos del inventario físico reportado por las tiendas:

* `Tienda`: Identificador o nombre de la sucursal.

* `Fecha del recuento`: Fecha en la que se realizó la auditoría física.

* `SKU`: Identificador único del producto.

* `Producto`: Nombre del producto registrado en la tienda.

* `Añada declarada`: Añada(s) encontrada(s) en tienda (permite múltiples valores separados por `+`).

## ⚙️ Flujo de Trabajo y Lógica del Pipeline

El proceso sigue una estructura modular paso a paso:

```
[ Subida de CSVs ] ──> [ Validación de Esquema ] ──> [ Merge (Left Join) ]
                                                              │
[ Exportación Gráfica ] <── [ Clasificación ] <── [ Extracción de Añadas ]

```

1. **Carga e Ingesta Interactiva**: Mediante un bucle `while` y `files.upload()`, el script solicita los dos archivos CSV y valida que contengan las columnas requeridas antes de continuar.

2. **Cruce de Datos**: Se realiza un `pd.merge` de tipo **Left Join** usando la columna `SKU` como clave primaria, manteniendo como base el Catálogo Web.

3. **Limpieza y Transformación**:

   * Eliminación de espacios en blanco (`.str.strip()`).

   * Conversión de fechas a tipo `datetime` y extracción del mes en español.

   * Parseo de múltiples añadas en tienda mediante división por el carácter `+`.

4. **Extracción y Comparación de Añadas**:

   * Se extrae el último elemento de los nombres de producto (correspondiente al año/añada).

   * Se evalúa la coherencia entre el año del producto en tienda, el año del producto en la web y la añada declarada.

5. **Clasificación y Reglas de Negocio**:
   Cada registro se clasifica según las siguientes condiciones:

   * **Sin discrepancia**: La añada web, la añada de tienda y la declarada coinciden completamente. $\rightarrow$ *Acción: OK*

   * **Con discrepancia**: Existe un descalce entre la añada publicada en la web y la física encontrada. $\rightarrow$ *Acción: Actualizar añada en CMS Web*

   * **Sin recuento en tienda**: El SKU está en la web pero no fue auditado en tienda. $\rightarrow$ *Acción: Auditar stock físico en almacén*

## 📊 Visualizaciones Generadas

El script genera y guarda automáticamente ocho gráficas estadísticas con estilo `dark` de Seaborn, etiquetadas dinámicamente con la fecha y hora exacta de ejecución (`YYYY-MM-DD_HH-MM`):

| Archivo Generado | Tipo de Gráfico | Descripción y Objetivo Analítico | 
| ----- | ----- | ----- | 
| **Clasificación de Discrepancias** | `sns.barplot` | Muestra el recuento absoluto de registros según su categoría (*Sin discrepancia*, *Con discrepancia*, *Sin recuento en tienda*). | 
| **Proporción de Discrepancias** | `sns.barplot` / `pie` | Muestra la distribución porcentual (%) relativa de cada estado sobre el total de registros procesados. | 
| **Distribución Estado - PVP** | `sns.boxplot` | Diagrama de cajas en escala logarítmica para comparar la distribución de precios (PVP) según el estado de auditoría. | 
| **Top 10 Mayor PVP Con Discrepancia** | `sns.barplot` horizontal | Priorización de los 10 productos de mayor valor económico que requieren actualización urgente de añada en la web. | 
| **Top 10 Mayor PVP Sin Recuento** | `sns.barplot` horizontal | Identificación de los 10 productos más costosos en la web que carecen de auditoría física en tienda. | 
| **Primeros Diez Casos a Arreglar** | Tabla / Ranking visual | Listado enfocado en los primeros 10 SKUs críticos que deben ser intervenidos de forma prioritaria por el equipo de operaciones. | 
| **Distribución del Estado por Tienda** | `sns.countplot` apilado | Comparativa del cumplimiento de catálogo y volumen de discrepancias desglosado por cada sucursal/tienda física. | 
| **Impacto Económico Acumulado** | `sns.lineplot` / `area` | Representación del impacto financiero monetario acumulado (€) asociado a los productos con discrepancias o falta de recuento. | 

## 🚀 Cómo Ejecutar

1. Abre el cuaderno en **Google Colab**.

2. Ejecuta las celdas en orden secuencial.

3. Cuando se te solicite, sube el archivo del **Catálogo Web**.

4. A continuación, sube el archivo del **Recuento en Tiendas**.

5. El script procesará los datos, mostrará las gráficas en pantalla y guardará las imágenes `.png` junto al informe `.txt` y el CSV final en el entorno local de Colab.
