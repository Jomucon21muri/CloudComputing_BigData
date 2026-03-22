# CAPÍTULO 6: Arquitectura de Data Lake

## 6.1. ¿Qué es un Data Lake?

> **Data Lake:** Repositorio centralizado que permite almacenar todos los datos estructurados, semiestructurados y no estructurados a cualquier escala, en su formato nativo, con esquema-on-read.

| Característica | Data Warehouse | Data Lake |
|----------------|----------------|-----------|
| **Datos** | Estructurados | Todos los tipos |
| **Esquema** | Schema-on-write | Schema-on-read |
| **Usuarios** | Analistas de negocio | Data scientists, analistas |
| **Procesamiento** | ETL antes de cargar | ELT (cargar primero) |
| **Agilidad** | Menos ágil | Muy ágil |
| **Coste** | Más caro por GB | Más barato |
| **Madurez** | Datos curados | Datos crudos y curados |
| **Propósito** | BI y reporting | Analytics, ML, exploración |

**Esquema-on-write (DW):**
```
Datos Fuente → ETL (Transformar + Validar) → Cargar en DW → Query
    ↓
  Lento al ingestar, rápido en consulta
```

**Esquema-on-read (Data Lake):**
```
Datos Fuente → Ingestar directamente en Data Lake → Aplicar esquema al leer
    ↓
  Rápido al ingestar, flexible en análisis
```

**Características de un Data Lake:**

**1. Almacenamiento de todos los tipos de datos:**

- Estructurados: CSV, Parquet, Avro
- Semi-estructurados: JSON, XML
- No estructurados: PDFs, imágenes, vídeos, logs

**2. Almacenamiento económico:**

- Object storage barato (S3, ADLS, GCS)
- $0.023/GB/mes vs $1-10/GB/mes en DW

**3. Escalabilidad:**

- Petabyte-scale
- Sin límites prácticos

**4. Flexibilidad:**

- Sin esquema predefinidoNo rechaza datos
- Exploración sin restricciones

**5. Integración con herramientas Big Data:**
Compatible con Spark, Hadoop, Presto, etc.

## 6.2. Arquitectura de tres capas (zonas)

```
┌─────────────────────────────────────────────────────┐
│                   RAW ZONE                          │
│  (Datos crudos, tal como llegan)                    │
│                                                     │
│  /raw/covid/2026/02/19/casos.csv                    │
│  /raw/ine/poblacion/pob_2024.xlsx                   │
│  /raw/oms/epidemias/brotes.json                     │
│                                                     │
│  Características:                                   │
│  - Inmutables (append-only)                         │
│  - Formato original                                 │
│  - Sin transformaciones                             │
│  - Particionado por fecha                           │
└────────────────┬────────────────────────────────────┘
                 ↓ ETL/Limpieza/Validación
┌─────────────────────────────────────────────────────┐
│                 TRUSTED ZONE                        │
│  (Datos validados, limpios, formato estándar)       │
│                                                     │
│  /trusted/covid/casos_validados.parquet             │
│  /trusted/ine/poblacion_clean.parquet               │
│  /trusted/oms/brotes_normalized.parquet             │
│                                                     │
│  Características:                                   │
│  - Formatos columnares (Parquet, Avro)              │
│  - Comprimidos                                      │
│  - Esquemas validados                               │
│  - Deduplicados                                     │
│  - Particionados                                    │
└────────────────┬────────────────────────────────────┘
                 ↓ Transformaciones/Agregaciones
┌─────────────────────────────────────────────────────┐
│                 REFINED ZONE                        │
│  (Datos agregados, listos para consumo)             │
│                                                     │
│  /refined/covid_por_ccaa_mes/                       │
│  /refined/poblacion_agregada/                       │
│  /refined/indicadores_epidemiologicos/              │
│                                                     │
│  Características:                                   │
│  - Agregados y optimizados                          │
│  - Modelos dimensionales                            │
│  - Optimizados para queries                         │
│  - Exportables a DW                                 │
└─────────────────────────────────────────────────────┘
```

**Zona RAW (cruda):**

**Propósito:** Almacenar datos tal como llegan, sin modificar.

**Características:**

- **Inmutable**: Nunca se modifican los archivos (append-only)
- **Formato original**: CSV si llega CSV, JSON si llega JSON
- **Particionado temporal**: Por fecha de ingesta
- **Retención larga**: Mantenerdatos años (económico)
- **Backup obligatorio**: Son los datos fuente

**Estructura de carpetas:**
```
/raw/
  /covid/
    /year=2024/
      /month=01/
        /day=15/
          casos_20240115_093020.csv
    /year=2024/
      /month=02/
        /day=19/
          casos_20240219_103045.csv
  /ine/
    /poblacion/
      /year=2024/
        poblacion_2024_20240101.xlsx
  /ministerio_sanidad/
    /cmbd/
      /year=2023/
        cmbd_altas_2023.csv
```

**Ejemplo de ingesta:**
```python
import boto3
from datetime import datetime
import requests

s3 = boto3.client('s3')

def ingest_to_raw(source_url, dataset_name):
    """
    Ingestar datos a zona RAW del Data Lake
    """
    # Descargar datos
    response = requests.get(source_url)
    data = response.content
    
    # Generar ruta con timestamp
    now = datetime.now()
    s3_key = f"raw/{dataset_name}/year={now.year}/month={now.month:02d}/day={now.day:02d}/{dataset_name}_{now.strftime('%Y%m%d_%H%M%S')}.csv"
    
    # Subir a S3
    s3.put_object(
        Bucket='mi-datalake-sanitario',
        Key=s3_key,
        Body=data,
        Metadata={
            'source_url': source_url,
            'ingestion_timestamp': now.isoformat(),
            'zone': 'raw'
        }
    )
    
    print(f"Datos ingestados en: s3://mi-datalake-sanitario/{s3_key}")
    return s3_key

# Uso
url_covid = "https://cnecovid.isciii.es/covid19/resources/casos_diarios.csv"
ingest_to_raw(url_covid, "covid_casos")
```

**Zona TRUSTED (confiable):**

**Propósito:** Datos validados, limpios y en formatos optimizados.

**Características:**

- **Formato columnar**: Parquet (preferred) o Avro
- **Comprimido**: Snappy, GZIP
- **Esquema definido**: Schema enforcement
- **Deduplicado**: Sin duplicados
- **Validado**: Reglas de calidad aplicadas
- **Particionado**: Por columnas de consulta frecuente

**Transformaciones típicas:**
```python
import pandas as pd
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *

spark = SparkSession.builder.appName("Raw-to-Trusted").getOrCreate()

# Leer desde RAW (todos los CSVs de un mes)
df_raw = spark.read.csv(
    "s3://mi-datalake-sanitario/raw/covid/year=2024/month=02/",
    header=True,
    inferSchema=False  # Evitar inferencia automática
)

# 1. Definir esquema explícito
schema = StructType([
    StructField("fecha", DateType(), False),
    StructField("provincia_iso", StringType(), False),
    StructField("sexo", StringType(), False),
    StructField("grupo_edad", StringType(), False),
    StructField("casos_confirmados", IntegerType(), True),
    StructField("hospitalizados", IntegerType(), True),
    StructField("uci", IntegerType(), True),
    StructField("fallecidos", IntegerType(), True)
])

# 2. Aplicar esquema y convertir tipos
df_typed = spark.read.csv(
    "s3://mi-datalake-sanitario/raw/covid/year=2024/month=02/",
    header=True,
    schema=schema,
    dateFormat="yyyy-MM-dd"
)

# 3. Limpieza y validación
df_clean = df_typed \
    .dropDuplicates() \
    .filter(col("fecha").isNotNull()) \
    .filter(col("casos_confirmados") >= 0) \
    .fillna(0, subset=["hospitalizados", "uci", "fallecidos"]) \
    .withColumn("ccaa", substring(col("provincia_iso"), 4, 2))  # Extraer CCAA de ISO

# 4. Validaciones de calidad
total_rows = df_clean.count()
null_casos = df_clean.filter(col("casos_confirmados").isNull()).count()
negative_casos = df_clean.filter(col("casos_confirmados") < 0).count()

print(f"Quality Report:")
print(f"  Total rows: {total_rows}")
print(f"  Rows with null casos: {null_casos} ({null_casos/total_rows*100:.2f}%)")
print(f"  Rows with negative casos: {negative_casos}")

# 5. Escribir a TRUSTED en formato Parquet, particionado
df_clean.write.mode("append").partitionBy("fecha").parquet(
    "s3://mi-datalake-sanitario/trusted/covid/casos_validados/",
    compression="snappy"
)

print("Datos escritos en zona TRUSTED")
```

**Comparación de formatos:**

| Formato | Tamaño | Tiempo Lectura | Compresión | Esquema |
|---------|--------|----------------|------------|---------|
| **CSV** | 100 MB | 10 s | No | No |
| **JSON** | 120 MB | 12 s | No | Parcial |
| **Parquet** | 15 MB | 2 s | Sí (columnar) | Sí |
| **Avro** | 25 MB | 3 s | Sí (fila) | Sí |

**Parquet** es el estándar de facto para Data Lakes analíticos.

**Zona REFINED (refinada):**

**Propósito:** Datos agregados, transformados y listos para consumo directo.

**Características:**

- **Agregaciones pre-calculadas**: Sumas, promedios, etc.
- **Modelos dimensionales**: Preparados para OLAP
- **Optimizados para queries específicas**: Índices, ordenación
- **Exportables a DW**: Formato compatible con Redshift, BigQuery, etc.
- **Documentados**: Metadatos ricos

**Ejemplo de transformación Trusted → Refined:**

```python
# Leer desde TRUSTED
df_trusted = spark.read.parquet("s3://mi-datalake-sanitario/trusted/covid/casos_validados/")

# Agregar por CCAA y mes
df_refined = df_trusted \
    .withColumn("year", year(col("fecha"))) \
    .withColumn("month", month(col("fecha"))) \
    .groupBy("ccaa", "year", "month") \
    .agg(
        sum("casos_confirmados").alias("total_casos"),
        sum("hospitalizados").alias("total_hospitalizados"),
        sum("uci").alias("total_uci"),
        sum("fallecidos").alias("total_fallecidos"),
        avg("casos_confirmados").alias("media_diaria_casos"),
        max("casos_confirmados").alias("pico_casos_diarios"),
        count("*").alias("dias_con_datos")
    ) \
    .withColumn("tasa_hospitalizacion", col("total_hospitalizados") / col("total_casos")) \
    .withColumn("tasa_letalidad", col("total_fallecidos") / col("total_casos")) \
    .orderBy("year", "month", "ccaa")

# Escribir datos refinados
df_refined.write.mode("overwrite").parquet(
    "s3://mi-datalake-sanitario/refined/covid/agregado_ccaa_mes/",
    compression="snappy"
)

# También escribir en formato optimizado para DW
df_refined.write.mode("overwrite") \
    .format("jdbc") \
    .option("url", "jdbc:postgresql://mi-redshift.amazonaws.com:5439/dw") \
    .option("dbtable", "refined.covid_ccaa_mes") \
    .option("user", "admin") \
    .option("password", "password") \
    .save()

print("Datos escritos en zona REFINED y exportados a DW")
```

## 6.3. Particionado de datos

**¿Por qué particionar?**

**Sin particionado:**
```
Query: SELECT * FROM datos WHERE fecha = '2024-02-19'
→ Escanea TODOS los archivos (1 TB)
→ Tiempo: 5 minutos
→ Coste query: $5
```

**Con particionado:**
```
Query: SELECT * FROM datos WHERE fecha = '2024-02-19'
→ Escanea SOLO /year=2024/month=02/day=19/ (500 MB)
→ Tiempo: 10 segundos
→ Coste query: $0.0025
```

**Estrategias de particionado:**

**1. Particionado por fecha (más común):**

```
/trusted/covid/
  /year=2023/
    /month=01/
      data.parquet
    /month=02/
      data.parquet
  /year=2024/
    /month=01/
      data.parquet
    /month=02/
      data.parquet
```

```python
# Escribir con particionado por año y mes
df.write.partitionBy("year", "month").parquet("s3://bucket/covid/")
```

**2. Particionado por región:**

```
/trusted/covid/
  /ccaa=Andalucía/
    data.parquet
  /ccaa=Cataluña/
    data.parquet
  /ccaa=Madrid/
    data.parquet
```

**3. Particionado mixto (fecha + región):**

```
/trusted/covid/
  /year=2024/
    /month=02/
      /ccaa=Madrid/
        data.parquet
      /ccaa=Cataluña/
        data.parquet
```

```python
# Particionado mixto
df.write.partitionBy("year", "month", "ccaa").parquet("s3://bucket/covid/")
```

**Best practices de particionado:**

**✅ Hacer:**

- Particionar por columnas de filtro frecuente (fecha, región)
- Tamaño de partición: 128 MB - 1 GB (óptimo para Spark)
- Usar tipos de dato apropiados (int para year, no string)

**❌ Evitar:**

- Particionar por columnas de alta cardinalidad (ej: usuario_id con millones de valores)
- Particiones muy pequeñas (< 10 MB) → muchos archivos pequeños
- Demasiados niveles de particionado (> 4 niveles)

**Ejemplo de problemas:**

```python
# MAL: Particionar por usuario (millones de particiones)
df.write.partitionBy("usuario_id").parquet("s3://bucket/datos/")
# Resultado: 5,000,000 carpetas con 1 archivo cada una → performance horrible

# BIEN: Particionar por fecha
df.write.partitionBy("year", "month").parquet("s3://bucket/datos/")
# Resultado: 24 carpetas para 2 años → performance óptima
```

## Formatos de almacenamiento

**Parquet (recomendado):**

**Características:**

- **Columnar**: Almacena por columnas, no por filas
- **Comprimido**: Snappy, GZIP, LZO
- **Esquema incluido**: Autodescriptivo
- **Optimizado para análisis**: Lee solo columnas necesarias

**Ventajas:**

| Ventaja | Descripción |
|---------|-------------|
| ✅ Compresión excelente | 5-10x vs CSV |
| ✅ Queries rápidas | Lee solo columnas necesarias con SELECT |
| ✅ Compatible con todo | Spark, Hive, Athena, Redshift, etc. |
| ✅ Soporta tipos complejos | Tipos de dato complejos soportados |

**Ejemplo:**

```python
# Escribir Parquet
df.write.parquet("s3://bucket/datos.parquet", compression="snappy")

# Leer Parquet (solo columnas necesarias)
df = spark.read.parquet("s3://bucket/datos.parquet").select("fecha", "casos")
# → Lee SOLO las columnas fecha y casos (no todo el archivo)
```

**Comparación row-based vs columnar:**

```
Row-based (CSV):
ROW1: fecha,ccaa,casos,hospitalizados,uci
ROW2: 2024-02-01,Madrid,100,5,2
ROW3: 2024-02-02,Madrid,110,6,2

Columnar (Parquet):
COL fecha:          [2024-02-01, 2024-02-02, ...]
COL ccaa:           [Madrid, Madrid, ...]
COL casos:          [100, 110, ...]
COL hospitalizados: [5, 6, ...]
COL uci:            [2, 2, ...]

Query: SELECT fecha, casos WHERE ccaa='Madrid'
→ CSV: Lee TODAS las columnas
→ Parquet: Lee SOLO fecha, casos, ccaa
```

**Avro:**

**Características:**

- **Row-based**: Almacena por filas
- **Esquema incluido**: en JSON
- **Evolución de esquema**: Soporta cambios de esquema

**Cuándo usar Avro:**

- Streaming de datos
- Aplicaciones que necesitan escribir filas completas frecuentemente
- Kafka con Schema Registry

**Delta Lake (evolución de Parquet):**

**Características:**

- **Parquet con transacciones ACID**
- **Time Travel**: Versiones de datos
- **Schema Evolution**: Cambiar esquema sin reescribir
- **UPSERT/DELETE**: Actualizaciones y borrados eficientes

```python
# Escribir Delta
df.write.format("delta").save("s3://bucket/covid_delta/")

# UPSERT (merge)
from delta.tables import *

deltaTable = DeltaTable.forPath(spark, "s3://bucket/covid_delta/")

deltaTable.alias("target").merge(
    df_new.alias("source"),
    "target.fecha = source.fecha AND target.provincia = source.provincia"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()

# Time Travel
df_yesterday = spark.read.format("delta") \
    .option("versionAsOf", 1) \
    .load("s3://bucket/covid_delta/")

df_last_week = spark.read.format("delta") \
    .option("timestampAsOf", "2024-02-12") \
    .load("s3://bucket/covid_delta/")
```

## Data Lake vs Data Swamp

**Riesgo: Data Swamp (pantano de datos):**

**Data Swamp**: Data Lake sin gobierno, metadatos ni organización → inútil.

**Síntomas:**

- ❌ Nadie sabe qué datos hay
- ❌ Duplicación masiva
- ❌ Sin documentación
- ❌ Datos obsoletos mezclados con actuales
- ❌ Sin control de calidad
- ❌ Sin control de acceso

**Cómo evitar Data Swamp:**

**1. Catálogo de datos obligatorio:**

```python
# Registrar dataset en catálogo (AWS Glue)

glue = boto3.client('glue')

glue.create_table(
    DatabaseName='datalake_sanitario',
    TableInput={
        'Name': 'covid_casos_trusted',
        'StorageDescriptor': {
            'Columns': [
                {'Name': 'fecha', 'Type': 'date'},
                {'Name': 'provincia_iso', 'Type': 'string'},
                {'Name': 'casos_confirmados', 'Type': 'int'}
            ],
            'Location': 's3://mi-datalake/trusted/covid/',
            'InputFormat': 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat',
            'SerdeInfo': {
                'SerializationLibrary': 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe'
            }
        },
        'PartitionKeys': [
            {'Name': 'year', 'Type': 'int'},
            {'Name': 'month', 'Type': 'int'}
        ],
        'Parameters': {
            'classification': 'parquet',
            'owner': 'equipo-salud-publica',
            'source': 'ministerio-sanidad',
            'update_frequency': 'daily',
            'quality_score': '92'
        }
    }
)
```

**2. Documentación automática:**

```python
# Generar metadatos automáticamente

def document_dataset(s3_path):
    df = spark.read.parquet(s3_path)
    
    metadata = {
        'path': s3_path,
        'row_count': df.count(),
        'columns': df.columns,
        'schema': df.schema.json(),
        'file_size_mb': get_s3_size(s3_path) / 1024 / 1024,
        'last_modified': get_s3_last_modified(s3_path),
        'partitions': get_partitions(s3_path),
        'data_quality': {
            'completeness': calculate_completeness(df),
            'duplicates': df.count() - df.dropDuplicates().count()
        }
    }
    
    # Guardar metadatos
    save_metadata(metadata)
    
    return metadata
```

**3. Retención y limpieza:**

```python
# S3 Lifecycle para limpieza automática

lifecycle_config = {
    'Rules': [
        {
            'ID': 'Limpiar RAW antiguo',
            'Status': 'Enabled',
            'Filter': {'Prefix': 'raw/'},
            'Transitions': [
                {'Days': 90, 'StorageClass': 'GLACIER'}
            ],
            'Expiration': {'Days': 730}  # Borrar después de 2 años
        },
        {
            'ID': 'Limpiar logs',
            'Status': 'Enabled',
            'Filter': {'Prefix': 'logs/'},
            'Expiration': {'Days': 90}
        }
    ]
}
```

**4. Control de calidad continuo:**

```python
# Monitorización de calidad

class DataQualityMonitor:
    def __init__(self, dataset_path):
        self.path = dataset_path
        self.df = spark.read.parquet(dataset_path)
    
    def check_quality(self):
        checks = {
            'row_count': self.df.count(),
            'null_percentage': self.calculate_nulls(),
            'duplicates': self.check_duplicates(),
            'schema_drift': self.check_schema_changes(),
            'freshness': self.check_freshness()
        }
        
        # Alertar si problemas
        if checks['null_percentage'] > 0.1:
            send_alert("Más del 10% de nulos en " + self.path)
        
        if checks['duplicates'] > 100:
            send_alert("Duplicados detectados en " + self.path)
        
        return checks

# Ejecutar diariamente
monitor = DataQualityMonitor("s3://mi-datalake/trusted/covid/")
quality_report = monitor.check_quality()
```

---

