# CAPÍTULO 7: Tecnologías de almacenamiento distribuido

## 7.1. HDFS (Hadoop Distributed File System)

**Arquitectura HDFS:**

```
┌──────────────────────────────────────────────────────┐
│              NameNode (Master)                       │
│  - Mantiene metadatos (nombre, ubicación archivos)  │
│  - Gestiona namespace                                │
│  - Coordina acceso                                   │
└─────────────────────┬────────────────────────────────┘
                      ↓
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
┌───────────┐  ┌───────────┐  ┌───────────┐
│ DataNode1 │  │ DataNode2 │  │ DataNode3 │
│           │  │           │  │           │
│ Bloque A1 │  │ Bloque A2 │  │ Bloque A3 │
│ Bloque B1 │  │ Bloque B2 │  │ Bloque C1 │
└───────────┘  └───────────┘  └───────────┘
```

**Características clave:**

**1. Bloques grandes:**
- Tamaño típico: 128 MB o 256 MB
- Por qué: minimiza seeks de disco, optimiza throughput

**2. Replicación:**
- Replication factor: 3 (por defecto)
- Fault tolerance: puede perder 2 nodos sin pérdida de datos

**3. Write-once-read-many:**
- Archivos inmutables
- Optimizado para análisis, no para actualizaciones

**Ejemplo:**

```
Archivo de 1 GB guardado en HDFS:
→ Se divide en 8 bloques de 128 MB cada uno
→ Cada blocks se replica 3 veces
→ Total almacenado: 1 GB × 3 = 3 GB

Bloque 1: DataNode 1, DataNode 3, DataNode 5
Bloque 2: DataNode 2, DataNode 4, DataNode 6
...

Si DataNode 3 falla:
→ HDFS detecta automáticamente
→ Re-replica Bloque 1 a otro DataNode
→ Siempre mantiene 3 copias
```

**Cuándo usar HDFS:**

✅ **Usar HDFS si:**
- Cluster Hadoop on-premise
- Procesamiento MapReduce/Spark batch
- Archivos grandes (GBs)
- Alta throughput más importante que latencia

❌ **NO usar HDFS si:**
- Cloud-native (usar S3/ADLS/GCS en su lugar)
- Archivos pequeños (< 1 MB)
- Actualizaciones frecuentes
- Baja latencia crítica

**En cloud:**
```
On-premise: HDFS
     ↓
Cloud: S3 / ADLS / GCS

Spark on EMR/Dataproc lee directamente de S3/GCS
→ No necesitas HDFS en cloud
```

## 7.2. Amazon S3 (Simple Storage Service)

**Características:**

**1. Object storage:**
- No es un filesystem tradicional
- Almacena objetos (archivos) con metadata
- Claves (keys) simulan carpetas

**2. Durabilidad y disponibilidad:**
- Durabilidad: 99.999999999% (11 noves)
- Disponibilidad: 99.99% (SLA)

**3. Escalabilidad ilimitada:**
- Sin límite de almacenamiento
- Sin límite de objetos

**4. Clases de almacenamiento:**

| Clase | Uso | Latencia | Coste |
|-------|-----|----------|-------|
| **Standard** | Datos frecuentes | ms | $0.023/GB/mes |
| **Intelligent-Tiering** | Acceso impredecible | ms | $0.023/GB + monitoring |
| **Standard-IA** | Acceso infrecuente | ms | $0.0125/GB + retrieval |
| **Glacier Instant** | Archivo con acceso inmediato | ms | $0.004/GB + retrieval |
| **Glacier Flexible** | Archivo (1-5 min retrieval) | min | $0.0036/GB |
| **Glacier Deep Archive** | Archivo (12h retrieval) | horas | $0.00099/GB |

```python
# Ejemplo: Lifecycle para optimizar costes

lifecycle_policy = {
    'Rules': [
        {
            'ID': 'Optimizar costes Data Lake',
            'Status': 'Enabled',
            'Filter': {'Prefix': 'trusted/'},
            'Transitions': [
                {
                    'Days': 30,
                    'StorageClass': 'INTELLIGENT_TIERING'  # Después de 30 días
                },
                {
                    'Days': 180,
                    'StorageClass': 'GLACIER_IR'  # Después de 180 días
                }
            ]
        }
    ]
}

s3.put_bucket_lifecycle_configuration(
    Bucket='mi-datalake',
    LifecycleConfiguration=lifecycle_policy
)

# Ahorro estimado:
# 1 TB en Standard: $23/mes
# → Después de 30 días en Intelligent-Tiering: $23/mes (auto-optimiza)
# → Después de 180 días en Glacier IR: $4/mes
# Ahorro: $19/mes = 82%
```

**S3 como Data Lake:**

**Ventajas:**

| Ventaja | Descripción |
|---------|-------------|
| ✅ Económico | Almacenamiento de bajo coste |
| ✅ Escalable sin límites | Capacidad ilimitada |
| ✅ Integrado con todo | Spark, Athena, Redshift, etc. |
| ✅ Serverless | Sin gestión de infraestructura |

**Best Practices:**

```python
# 1. Particionado
s3://mi-datalake/trusted/covid/year=2024/month=02/data.parquet

# 2. Formato Parquet
df.write.parquet("s3://mi-datalake/trusted/covid/", compression="snappy")

# 3. Encriptación
s3.put_object(
    Bucket='mi-datalake',
    Key='datos_sensibles.parquet',
    Body=data,
    ServerSideEncryption='AES256'  # O 'aws:kms'
)

# 4. Versionado (para datos críticos)
s3.put_bucket_versioning(
    Bucket='mi-datalake',
    VersioningConfiguration={'Status': 'Enabled'}
)

# 5. Replicación cross-region (DR)
replication_config = {
    'Role': 'arn:aws:iam::123456789012:role/s3-replication',
    'Rules': [{
        'Status': 'Enabled',
        'Priority': 1,
        'DeleteMarkerReplication': {'Status': 'Enabled'},
        'Filter': {'Prefix': 'critical/'},
        'Destination': {
            'Bucket': 'arn:aws:s3:::mi-datalake-replica',
            'ReplicationTime': {
                'Status': 'Enabled',
                'Time': {'Minutes': 15}
            }
        }
    }]
}
```

---

(Continuaré con los siguientes bloques en el archivo...)
