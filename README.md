
# Databricks Semiconductor Yield Pipeline

Proyecto de Data Engineering usando **Databricks** y arquitectura **Medallion** aplicando a datos reales de fabricación de semiconductores (dataset SECOM).

## Objetivo 
Demostrar el manejo profesional de datos en databricks: ingestar, limpieza, transformación y análisis de calidad de datos (Yield Analysis) en un proceso de fabricación de chips.

## Arquitectura Medallion
| Capa       | Tabla                               Descripción
|
|-------------|-------------------------------------|---------------|
| Bronze      | `bronze_signal_data`             | Datos crudos del proceso de fabricación
| Silver      | `silver_signal_data`             | Datos limpios + columnas renombrados
| Gold        | `gold_nulls_summary`             | Resumen de valores nulos por columna
| Gold        | `gold_sensor_stats`              | Estadísticas descriptivas de cada sensor

# Explicación y soluciones

## Aprendizaje y errores solucionado

Durante el desarrollo del proyecto me encontré con varios errores que al principio no los conocías pero que a medida que desarrollaba se hallaba más común en Databricks y Delta Lake. Proceso principales y cómo los solucioné: 

### 1. Ingesta de datos (Capa Bronze)

- Descargué el dataset **SECOM Semiconductor Manufacturing** desde Kaggle (`signal-data.csv`).
- Subí el archivo a Databricks Free Edition.
- Creé el shema `semiconductor_project` dentro del catálogo `devsquad_bt`
- Cree la tabla:  SQL
         devsquad_bt.semiconductor_project.bronze_signal_data
- Esta capa contiene los **datos crudos** tal como llegaron (1,567 registros y 592 columnas).
- En esta capa guarda los datos originales sin modificar, para poder volver a ellos en cualquier momento.

### 2. Limpieza y transformación (Capa Silver)
A partir de la tabla Bronze realicé las siguientes transformaciones:

1. Eliminé las columnas que tenían más del 50% de valores nulos (pasé de 592 a 564 columnas).
2. Renombré las columnas numéricas (0, 1, 2...) a nombres más claros y profesionales:
         - Sensor_000, Sensor_001, Sensor_002, sucesivamente
3.  Guardé el resultado en la tabla:
        'devsquad_bt.semiconductor_project.silver_signal_data'

Tener datos limpios, con nombres entendibles y listos para análisis. Es la capa de calidad.


### 3. Análisis y agregaciones (Capa Gold)
Creé dos tablas de valor de negocio:


| Tabla Gold         | Qué contiene                                                | Para qué sirve                                    |
| ------------------ | ----------------------------------------------------------- | ------------------------------------------------- |
| gold_nulls_summary | Cantidad y porcentaje de nulos por columna                  | Evaluar la calidad de los datos                   |
| gold_sensor_stats  | Promedio, mínimo, máximo, desviación y nulos de cada sensor | Análisis exploratorio de los sensores del proceso |
Son tablas listas para consumo (reportes, dashboards modelos de Machine Learning).

### 4. Tecnologías y conceptos 
- Delta Lake: Todas las tablas son Delta (historial de versiones, Time Travel, ACID).
- Unity Catalog: Organización con Catalog -> Schema -> Table.
- PySpark + Spark SQL: Mezcla de Python y SQL en el mismo notebook.
- Medallion Arquitecture: Separación clara de responsabilidades por capa.

### Errores y aprendizaje

### 1. Error: 'overwriteShema' (typo)
Escribía '.option('overwriteshema,.....')' en lugar de 'overwriteschema' que era corregir la ortografía agregando la consonante 'c'

### 2. Error: 'DELTA_METADATA_MISMATH'
Al intentar renombrar muchas columnas de golpe, Delta Lake detectaba incompatibilidad de esquema. como solución elimine la tabla Silver con 'DROP TABLE' y volver a crearla desde Bronze con los nuevos nombres.

### 3. Error: Columnas no se renombraban permanentemente
El renombrado parecía funcionar en memoria, pero al leer la tabla nuevo volvían los nombre originales (0, 1, 2 …).
Al eliminar la tabla volví a reescribir completa de la tabla y verificar inmediatamente después de guardar.

### 4. Error: 'CANNOT_INFER_EMPTY_SCHEMA'
Al calcular estadísticas, la lista de sensores quedaba vacía porque las columnas todavía no tenían el prefijo 'Sensor_'.
En esto tuve que volver asegurar de que los renombres se haya guardado correctamente antes de ejecutar el código de estadísticas.

Y también olvidaba que no se podía acceder a versiones antiguas ya que después de 7 días se eliminaba y solo tenia que volver a ejecutar el código o la celda que necesitara. 

# Conclusión
Con este proyecto demostré el flujo completo de un pipeline de datos profesional en Databricks:
1. Ingestar datos crudos (Bronze)
2. Limpieza                      (Silver)
3. Tablas analíticas (Gold)
## Tecnologías utilizadas
- Databricks Free Edition
- Apache Spark
- Delta Lake
- PySpark + Spark SQL
- Unity Catalog

## Dataset
- **Fuente**: SECOM Semiconductor Manufacturing Dataset (Kaggle / UCI)
- **Registros**: 1,567
- **Columnas originales**: 592 (sensores + Time)

## Estructura del proyecto
notebooks/
____01_Architecture_Semiconductor.ipynb


## Autor
Ericka A 