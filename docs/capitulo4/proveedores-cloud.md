# CAPÍTULO 4: Proveedores cloud (AWS, Azure, Google Cloud)

## Introducción a proveedores cloud

Los tres principales proveedores de cloud público a nivel mundial son:

1. **Amazon Web Services (AWS)** - Líder del mercado (~32% cuota)
2. **Microsoft Azure** - Segundo (~23% cuota)
3. **Google Cloud Platform (GCP)** - Tercero (~10% cuota)

Otros proveedores relevantes: Alibaba Cloud, IBM Cloud, Oracle Cloud, Tencent Cloud.

**Comparativa de infraestructura global:**

| Proveedor | Regiones | Zonas de Disponibilidad (AZs) | Edge Locations | Países/Territorios |
|-----------|----------|-------------------------------|----------------|-------------------|
| **AWS** | 33 | 105 | 400+ (CloudFront) | 245+ |
| **Azure** | 60+ | 140+ zonas | 170+ (Azure CDN) | 140 |
| **GCP** | 40 | 121 | 187 (Cloud CDN) | 200+ |

**Actualizado:** Febrero 2024

**Regiones en España:**
- **AWS**: Región `eu-south-2` (Aragón) - 3 AZs - Disponible desde 2022
- **Azure**: Ninguna región específica en España aún
- **GCP**: Región `europe-southwest1` (Madrid) - 3 zonas - Disponible desde 2023

**Comparativa de servicios principales:**

#### Computación

| Categoría | AWS | Azure | GCP |
|-----------|-----|-------|-----|
| **VMs** | EC2 | Virtual Machines | Compute Engine |
| **Serverless Containers** | Fargate | Container Instances | Cloud Run |
| **Kubernetes** | EKS | AKS | GKE |
| **Serverless Functions** | Lambda | Functions | Cloud Functions |
| **Batch Processing** | Batch | Batch | N/A (usa Compute Engine) |

#### Almacenamiento

| Categoría | AWS | Azure | GCP |
|-----------|-----|-------|-----|
| **Object Storage** | S3 | Blob Storage | Cloud Storage |
| **Block Storage** | EBS | Managed Disks | Persistent Disks |
| **File Storage** | EFS | Files | Filestore |
| **Archive** | Glacier | Archive Storage | Archive Storage |

#### Bases de Datos

| Categoría | AWS | Azure | GCP |
|-----------|-----|-------|-----|
| **Relacional (managed)** | RDS (PostgreSQL, MySQL, Oracle, SQL Server, MariaDB) | SQL Database, Database for PostgreSQL/MySQL | Cloud SQL |
| **NoSQL Documento** | DocumentDB | Cosmos DB | Firestore |
| **NoSQL Key-Value** | DynamoDB | Table Storage, Cosmos DB | Cloud Bigtable |
| **Data Warehouse** | Redshift | Synapse Analytics | BigQuery |
| **Cache** | ElastiCache (Redis, Memcached) | Cache for Redis | Memorystore |

#### Big Data y Analytics

| Categoría | AWS | Azure | GCP |
|-----------|-----|-------|-----|
| **Spark/Hadoop** | EMR | HDInsight | Dataproc |
| **ETL** | Glue | Data Factory, Synapse Pipelines | Dataflow |
| **Streaming** | Kinesis | Event Hubs, Stream Analytics | Dataflow (Apache Beam) |
| **SQL sobre Object Storage** | Athena | Synapse Serverless SQL | BigQuery |
| **Data Catalog** | Glue Data Catalog | Purview | Data Catalog |

#### Machine Learning e IA

| Categoría | AWS | Azure | GCP |
|-----------|-----|-------|-----|
| **ML Platform** | SageMaker | Machine Learning | Vertex AI |
| **AutoML** | SageMaker Autopilot | AutoML | AutoML |
| **Vision API** | Rekognition | Computer Vision | Vision API |
| **Speech** | Transcribe, Polly | Speech Services | Speech-to-Text, Text-to-Speech |
| **NLP** | Comprehend | Text Analytics | Natural Language API |

**Comparativa de pricing (ejemplo: VMs):**

**Instancia de propósito general con 4 vCPUs, 16 GB RAM, Linux:**

| Proveedor | Tipo Instancia | On-Demand ($/hora) | 1-Year Reserved | 3-Year Reserved |
|-----------|----------------|-------------------|-----------------|-----------------|
| **AWS** | m5.xlarge | $0.192 | $0.115 (40% off) | $0.077 (60% off) |
| **Azure** | D4s_v5 | $0.192 | $0.108 (44% off) | $0.069 (64% off) |
| **GCP** | n2-standard-4 | $0.194 | $0.126 (35% off) | $0.089 (54% off) |

**Región:** us-east / us-central (precios varían por región)

**Observación:** Azure y GCP suelen ser ligeramente más competitivos en pricing, pero AWS ofrece mayor variedad de tipos de instancias y opciones de optimización.

**Calculadoras de costes:**

- **AWS**: https://calculator.aws/
- **Azure**: https://azure.microsoft.com/en-us/pricing/calculator/
- **GCP**: https://cloud.google.com/products/calculator

**Calculadoras TCO (Total Cost of Ownership):**

Permiten estimar ahorro al migrar de on-premise a cloud:

- **AWS**: https://aws.amazon.com/tco-calculator/
- **Azure**: https://azure.microsoft.com/en-us/pricing/tco/calculator/
- **GCP**: https://cloud.google.com/products/calculator (incluye comparación)

**Free Tier / Prueba Gratuita:**

Todos los proveedores ofrecen capas gratuitas para probar:

**AWS Free Tier:**
- 750 horas/mes de EC2 t2.micro (1 vCPU, 1 GB RAM) - 12 meses
- 5 GB de S3 Standard Storage - 12 meses
- 750 horas/mes de RDS db.t2.micro - 12 meses
- 1 millón de solicitudes Lambda/mes - Siempre gratis
- 25 GB de DynamoDB - Siempre gratis

**Azure Free Account:**
- $200 crédito para 30 días
- 12 meses de servicios gratuitos:
  - 750 horas/mes de B1S VM (1 vCPU, 1 GB RAM)
  - 64 GB ×  2 de almacenamiento con Managed Disks
  - 5 GB de Blob Storage
- Servicios siempre gratuitos: App Service, Functions, etc.

**GCP Free Tier:**
- $300 crédito para 90 días
- Always Free (limitado):
  - 1 f1-micro VM (0.2 vCPU, 0.6 GB RAM) en us-regions
  - 30 GB de Cloud Storage
  - 1 GB de Cloud Functions invocations
  - BigQuery: 1 TB de consultas/mes

---

## 4.1. Amazon Web Services (AWS)

**Infraestructura global AWS:**

**Datos actualizados (2024):**
- **33 regiones geográficas**
- **105 zonas de disponibilidad (AZs)**
- **400+ ubicaciones de borde (Edge Locations)** para CloudFront CDN
- **245+ países y territorios** servidos

**Nueva Región en España:**
- **Nombre:** AWS Europe (Spain)
- **Código:** `eu-south-2`
- **Ubicación:** Aragón
- **AZs:** 3 zonas de disponibilidad
- **Disponible desde:** 2022

**Mapa interactivo:** https://aws.amazon.com/about-aws/global-infrastructure/

**Regiones AWS (principales):**

| Región | Código | AZs | Año Lanzamiento |
|--------|--------|-----|-----------------|
| US East (N. Virginia) | us-east-1 | 6 | 2006 |
| US West (Oregon) | us-west-2 | 4 | 2011 |
| Europe (Ireland) | eu-west-1 | 3 | 2007 |
| Europe (Frankfurt) | eu-central-1 | 3 | 2014 |
| Europe (Spain) | eu-south-2 | 3 | 2022 |
| Asia Pacific (Singapore) | ap-southeast-1 | 3 | 2010 |
| Asia Pacific (Tokyo) | ap-northeast-1 | 4 | 2011 |
| South America (São Paulo) | sa-east-1 | 3 | 2011 |

**Lista completa:** https://aws.amazon.com/about-aws/global-infrastructure/regions_az/

**Zonas de disponibilidad (AZs):**

**Definición:** Datacenters físicamente separados dentro de una región, conectados con enlaces de baja latencia (<2 ms típicamente).

**Características:**
- Separación física de varios kilómetros (protección contra desastres locales)
- Alimentación eléctrica independiente
- Redes redundantes
- Nombradas con sufijos: -1a, -1b, -1c, etc.

**Ejemplo:** Región `eu-south-2` (España) tiene 3 AZs:
- `eu-south-2a`
- `eu-south-2b`
- `eu-south-2c`

**Alta Disponibilidad:**
Desplegar aplicaciones en múltiples AZs proporciona:
- Tolerancia a fallos de datacenter completo
- SLA más alto (99.99% vs 99.5% single-AZ)

```python
# Ejemplo: Desplegar en múltiples AZs
import boto3

ec2 = boto3.client('ec2', region_name='eu-south-2')

# Obtener AZs disponibles en la región
azs = ec2.describe_availability_zones()['AvailabilityZones']

for az in azs:
    print(f"AZ: {az['ZoneName']} - Estado: {az['State']}")
    
    # Lanzar instancia en cada AZ
    ec2.run_instances(
        ImageId='ami-12345678',  # AMI de la región
        InstanceType='t3.medium',
        MinCount=1,
        MaxCount=1,
        Placement={'AvailabilityZone': az['ZoneName']},
        Tags=[
            {'Key': 'Name', 'Value': f"WebServer-{az['ZoneName']}"},
            {'Key': 'AZ', 'Value': az['ZoneName']}
        ]
    )

# Resultado: 3 instancias (1 por AZ) con alta disponibilidad
```

**Catálogo de servicios AWS:**

AWS ofrece **200+ servicios** en múltiples categorías. A continuación, los más relevantes:

#### 4.1.1. Servicios de Cóm puto (Compute)

**Amazon EC2 (Elastic Compute Cloud)**

Descripción: Máquinas virtuales escalables en la nube.

**Tipos de Instancias EC2:**

AWS ofrece **400+ tipos de instancias** optimizadas para diferentes casos de uso:

**1. Propósito General (General Purpose)**

**T3 / T2 - Rendimiento Ampliable (Burstable Performance)**
- **vCPUs:** 2 - 8
- **RAM:** 0.5 GB - 32 GB
- **Almacenamiento:** Solo EBS
- **Rendimiento:** Base CPU + créditos para burst
- **Precio:** $0.0052/hora (t3.nano) hasta $0.1664/hora (t3.2xlarge)
- **Casos de uso:**
  - Sitios web y aplicaciones de tráfico moderado
  - Entornos de desarrollo y testing
  - Repositorios de código (GitLab, GitHub Enterprise)
  - Microservicios con carga variable
  - Bases de datos pequeñas

```python
# Ejemplo: Crear instancia T3 para desarrollo
import boto3

ec2 = boto3.resource('ec2', region_name='eu-south-2')

instance = ec2.create_instances(
    ImageId='ami-0c55b159cbfafe1f0',  # Amazon Linux 2
    InstanceType='t3.medium',  # 2 vCPU, 4 GB RAM
    MinCount=1,
    MaxCount=1,
    KeyName='mi-clave-ssh',
    SecurityGroupIds=['sg-12345678'],
    SubnetId='subnet-abcdef',
    TagSpecifications=[{
        'ResourceType': 'instance',
        'Tags': [
            {'Key': 'Name', 'Value': 'Dev-Server'},
            {'Key': 'Environment', 'Value': 'Development'},
            {'Key': 'CostCenter', 'Value': 'IT-Dev'}
        ]
    }],
    # Configurar créditos CPU
    CreditSpecification={'CpuCredits': 'unlimited'}  # Permite burst ilimitado
)

print(f"Instancia creada: {instance[0].id}")
print(f"Coste estimado: ~$35/mes (730 horas × $0.0416/hora)")
```

**M5 / M6i - Última Generación Propósito General**
- **vCPUs:** 2 - 96
- **RAM:** 8 GB - 384 GB
- **Almacenamiento:** EBS o NVMe SSD local (M5d)
- **Procesador:** Intel Xeon Platinum 8000 series (hasta 3.1 GHz)
- **Networking:** Hasta 25 Gbps
- **Precio:** $0.096/hora (m5.large) hasta $4.608/hora (m5.24xlarge)
- **Casos de uso:**
  - Servidores de aplicaciones de alto tráfico
  - Bases de datos medianas/grandes (no in-memory)
  - SAP, Oracle E-Business Suite
  - Clusters de procesamiento (Hadoop workers)
  - Backend de juegos multijugador

```python
# Ejemplo: Servidor web de producción con M5
ec2.create_instances(
    ImageId='ami-nginx-optimized',
    InstanceType='m5.xlarge',  # 4 vCPU, 16 GB RAM
    BlockDeviceMappings=[{
        'DeviceName': '/dev/xvda',
        'Ebs': {
            'VolumeSize': 100,  # 100 GB
            'VolumeType': 'gp3',  # SSD propósito general
            'Iops': 3000,
            'Throughput': 125,
            'DeleteOnTermination': True,
            'Encrypted': True
        }
    }],
    # ... otros parámetros
)
# Coste: ~$140/mes
```

**2. Optimizadas para Cómputo (Compute Optimized)**

**C5 / C6i - Alto Rendimiento Computacional**
- **vCPUs:** 2 - 72
- **RAM:** 4 GB - 144 GB (ratio 1:2 vCPU:RAM)
- **Procesador:** Intel Xeon Platinum con AVX-512 a 3.4 GHz
- **Networking:** Hasta 25 Gbps
- **Precio:** $0.085/hora (c5.large) hasta $3.456/hora (c5.18xlarge)
- **Casos de uso:**
  - Servidores web de muy alto rendimiento (millones de req/día)
  - Procesamiento batch intensivo
  - Modelado científico y simulaciones
  - Machine Learning inference (predicción)
  - Análisis distribuido (Spark, Presto)
  - Codificación de vídeo a gran escala
  - Gaming multijugador con física compleja
  - Ad serving de alta frecuencia

```python
# Ejemplo: Cluster HPC para simulación científica
for i in range(10):  # 10 nodos de cómputo
    ec2.create_instances(
        ImageId='ami-hpc-optimized',
        InstanceType='c5.9xlarge',  # 36 vCPU, 72 GB RAM
        # Placement Group para baja latencia inter-nodo
        Placement={
            'GroupName': 'hpc-cluster-placement-group',
            'Tenancy': 'default'
        },
        NetworkInterfaces=[{
            'DeviceIndex': 0,
            'SubnetId': 'subnet-hpc',
            'Groups': ['sg-hpc'],
            # Habilitar Enhanced Networking (SR-IOV)
            'InterfaceType': 'efa'  # Elastic Fabric Adapter
        }],
        TagSpecifications=[{
            'ResourceType': 'instance',
            'Tags': [
                {'Key': 'Name', 'Value': f'HPC-Node-{i+1}'},
                {'Key': 'Cluster', 'Value': 'Simulacion-CFD'}
            ]
        }]
    )

# Coste cluster: 10 × $1.53/hora = $15.30/hora = $11,169/mes (24/7)
# Si solo se usa 8 horas/día: $3,672/mes
```

**3. Optimizadas para Memoria (Memory Optimized)**

**R5 / R6i - Alta Memoria**
- **vCPUs:** 2 - 96
- **RAM:** 16 GB - 768 GB (ratio 1:8 vCPU:RAM)
- **Procesador:** Intel Xeon Platinum
- **Networking:** Hasta 25 Gbps
- **Precio:** $0.126/hora (r5.large) hasta $6.048 /hora (r5.24xlarge)
- **Casos de uso:**
  - Bases de datos de alto rendimiento (PostgreSQL, MySQL, Oracle)
  - Motores de procesamiento Big Data en memoria (Spark)
  - Caches distribuidos (Redis, Memcached clusters)
  - Elasticsearch/OpenSearch con índices grandes

**X1 / X1e - Memoria Masiva**
- **vCPUs:** Hasta 128
- **RAM:** Hasta **3,904 GB (3.9 TB)**
- **Precio:** $13.338/hora (x1.32xlarge)
- **Casos de uso específicos:**
  - SAP HANA (requiere TB de RAM)
  - Bases de datos en memoria empresariales
  - Apache Spark con datasets enormes en RAM

```python
# Ejemplo: Base de datos PostgreSQL de alto rendimiento
rds = boto3.client('rds')

rds.create_db_instance(
    DBInstanceIdentifier='prod-postgres',
    DBInstanceClass='db.r5.4xlarge',  # 16 vCPU, 128 GB RAM
    Engine='postgres',
    EngineVersion='15.3',
    MasterUsername='admin',
    MasterUserPassword='SuperSecurePassword123!',
    AllocatedStorage=1000,  # 1 TB
    StorageType='gp3',
    Iops=12000,
    StorageEncrypted=True,
    MultiAZ=True,  # Alta disponibilidad (replica en otra AZ)
    VpcSecurityGroupIds=['sg-database'],
    DBSubnetGroupName='private-subnet-group',
    BackupRetentionPeriod=30,  # Backups automáticos 30 días
    PreferredBackupWindow='03:00-04:00',  # 3-4 AM UTC
    PreferredMaintenanceWindow='sun:04:00-sun:05:00'
)

# Coste: $1.008/hora × 730h = $736/mes (single-AZ)
# Con Multi-AZ: ~$1,472/mes
```

**High Memory Instances (u-*.metal)**
- **RAM:** Hasta **12 TiB (12,288 GB)**
- **vCPUs:** 448 cores lógicos
- **Precio:** Solo bajo solicitud (decenas de miles $/mes)
- **Caso de uso:** SAP HANA para grandes empresas

**4. Aceleradas por GPU (Accelerated Computing)**

**P3 / P4 - GPU para Machine Learning**
- **GPU:** NVIDIA Tesla V100 (P3) o A100 (P4)
- **GPU Memory:** 16 GB - 40 GB por GPU
- **GPUs por instancia:** 1, 4, 8, o 16
- **vCPUs:** 8 - 96
- **RAM:** 61 GB - 1,152 GB
- **Precio:** $3.06/hora (p3.2xlarge, 1 GPU) hasta $32.77/hora (p3dn.24xlarge, 8 GPUs)
- **Casos de uso:**
  - Entrenamiento de modelos Deep Learning (BERT, GPT, CNNs)
  - Inferencia de ML de alta performance
  - HPC con GPU (si mulaciones físicas, química computacional)

```python
# Ejemplo: Entrenar modelo Deep Learning con PyTorch
# script_entrenamiento.py (se ejecuta en instancia P3)

import torch
import torch.nn as nn
from torchvision import models, datasets, transforms

# Detectar GPU
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Usando dispositivo: {device}")
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
    print(f"Memoria GPU disponible: {torch.cuda.get_device_properties(0).total_memory / 1e9:.2f} GB")

# Cargar modelo preentrenado
model = models.resnet50(pretrained=True)
model.fc = nn.Linear(model.fc.in_features, 10)  # 10 clases custom
model = model.to(device)  # Mover modelo a GPU

# Cargar datos
transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

train_dataset = datasets.ImageFolder('s3://mi-bucket/training-data/', transform=transform)
train_loader = torch.utils.data.DataLoader(train_dataset, batch_size=128, shuffle=True, num_workers=8)

# Entrenar
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

model.train()
for epoch in range(10):
    for batch_idx, (data, target) in enumerate(train_loader):
        data, target = data.to(device), target.to(device)  # Mover a GPU
        
        optimizer.zero_grad()
        output = model(data)
        loss = criterion(output, target)
        loss.backward()
        optimizer.step()
        
        if batch_idx % 10 == 0:
            print(f'Epoch: {epoch}, Batch: {batch_idx}, Loss: {loss.item():.4f}')

# Guardar modelo entrenado
torch.save(model.state_dict(), 's3://mi-bucket/models/resnet50_trained.pth')

# Velocidad: Con P3 (GPU): 10 épocas en ~2 horas = $6.12
#           Sin GPU (CPU): 10 épocas en ~48 horas = $9.22 (más lento Y más caro)
```

**G4 / G5 - GPU para Graphics/Inference**
- **GPU:** NVIDIA T4 (G4) o A10G (G5)
- **Precio:** Más económico que P3
- **Casos de uso:**
  - ML Inference (predicción)
  - Rendering 3D
  - Streaming de juegos (cloud gaming)
  - Codificación de vídeo con GPU

**5. Optimizadas para Almacenamiento (Storage Optimized)**

**H1 - HDD Denso**
- **Almacenamiento:** Hasta 16 TB de HDD por instancia (8 × 2 TB)
- **vCPUs:** Hasta 64
- **RAM:** Hasta 256 GB
- **Precio:** $0.468/hora (h1.2xlarge) hasta $4.752/hora (h1.16xlarge)
- **Throughput:** Hasta 2.8 GBps secuencial
- **Casos de uso:**
  - Nodos de datos Hadoop/HDFS
  - Almacenamiento distribuido (Lustre, GlusterFS)
  - Data warehousing con alta throughput secuencial
  - Procesamiento de logs masivos

**I3 / I3en - NVMe SSD**
- **Almacenamiento:** Hasta 60 TB de NVMe SSD
- **vCPUs:** Hasta 96
- **RAM:** Hasta 768 GB
- **Precio:** $0.156/hora (i3.large) hasta $15.360/hora (i3en.24xlarge)
- **IOPS:** Millones de IOPS random
- **Latencia:** Microsegundos
- **Casos de uso:**
  - Bases de datos NoSQL de alta performance (Cassandra, MongoDB, ScyllaDB)
  - Elasticsearch/OpenSearch con índices SSD
  - Caches distribuidos con persistencia
  - Procesamiento real-time de eventos

```python
# Ejemplo: Cluster Cassandra de alto rendimiento
# 3 nodos I3 en 3 AZs diferentes

nodes = []

for i, az in enumerate(['eu-south-2a', 'eu-south-2b', 'eu-south-2c']):
    instance = ec2.create_instances(
        ImageId='ami-cassandra-optimized',
        InstanceType='i3.4xlarge',  # 16 vCPU, 122 GB RAM, 2× 1.9 TB NVMe
        Placement={'AvailabilityZone': az},
        BlockDeviceMappings=[],  # Usar solo NVMe local (efímero pero rápido)
        UserData=f'''#!/bin/bash
        # Configurar Cassandra
        echo "Nodo {i+1} en {az}"
        # Montar discos NVMe
        mkfs.xfs /dev/nvme0n1
        mkfs.xfs /dev/nvme1n1
        mkdir -p /data/cassandra1 /data/cassandra2
        mount /dev/nvme0n1 /data/cassandra1
        mount /dev/nvme1n1 /data/cassandra2
        # ... configurar Cassandra para usar /data/*
        ''',
        Tags=[{'Key': 'Name', 'Value': f'Cassandra-Node-{i+1}-{az}'}]
    )[0]
    
    nodes.append(instance)
    print(f"Node {i+1}: {instance.id} en {az}")

# Coste: 3 × $2.496/hora = $7.49/hora = $5,467/mes
# Performance: Millones de escrituras/sec, latencia p99 < 5ms
```

**6. Instancias con ARM (Graviton)**

**T4g, M6g, C6g, R6g - AWS Graviton2/3**
- **Arquitectura:** ARM64 (en lugar de x86)
- **Ventajas:**
  - 20-40% más económico que instancias Intel equivalentes
  - Mejor eficiencia energética
  - Rendimiento comparable o superior en muchos workloads
- **Compatibilidad:** Requiere software compilado para ARM (Linux, contenedores, etc.)
- **Casos de uso:**
  - Aplicaciones cloud-native (microservicios, contenedores)
  - Cargas de trabajo web/móvil
  - Análisis de datos con software ARM-compatible

```python
# Ejemplo: Migrar de M5 (Intel) a M6g (Graviton) para ahorro
# ANTES: M5.xlarge (Intel) = $0.192/hora
# DESPUÉS: M6g.xlarge (Graviton) = $0.154/hora
# AHORRO: 20% ($0.038/hora × 730h = $27.74/mes por instancia)

# Para 100 micros ervicios:
ahorro_mensual_100_instancias = 0.038 * 730 * 100
print(f"Ahorro mensual con Graviton: ${ahorro_mensual_100_instancias:,.2f}")
# Output: Ahorro mensual con Graviton: $2,774.00

# Solo requiere rebuild de contenedores para ARM:
# docker buildx build --platform linux/arm64 -t mi-app:arm .
```

**Modelos de pago EC2:**

AWS ofrece 4 formas de pagar por instancias EC2:

**1. On-Demand (Bajo Demanda)**
- Pago por hora (o por segundo para Linux)
- Sin compromiso a largo plazo
- Ideal para: Desarrollo, testing, cargas impredecibles
- Precio: Estándar (ejemplo: m5.xlarge = $0.192/hora)

**2. Reserved Instances (Instancias Reservadas)**
- Compromiso de 1 o 3 años
- Descuento: 40-60% vs On-Demand
- 3 tipos de pago:
  - **All Upfront:** Pago total adelantado (máximo descuento)
  - **Partial Upfront:** Parte adelantado, resto mensual
  - **No Upfront:** Todo mensual (menor descuento)
- 2 clases:
  - **Standard:** No se puede cambiar tipo de instancia (máximo descuento)  
  - **Convertible:** Se puede cambiar tipo (menor descuento)

**Comparativa de Precios m5.xlarge (Linux, us-east-1):**

| Modelo | Precio  | Ahorro |
|--------|---------|--------|
| On-Demand | $0.192/hora | Base |
| Reserved 1yr No Upfront | $0.127/hora | 34% |
| Reserved 1yr Partial Upfront | $0.122/hora | 36% |
| Reserved 1yr All Upfront | $0.115/hora | 40% |
| Reserved 3yr No Upfront | $0.087/hora | 55% |
| Reserved 3yr Partial Upfront | $0.082/hora | 57% |
| Reserved 3yr All Upfront | $0.077/hora | 60% |

**Cálculo ROI:**
```python
# Workload: 10 instancias m5.xlarge corriendo 24/7 durante 3 años

horas_3_años = 24 * 365 * 3  # 26,280 horas

# Escenario 1: On-Demand
coste_on_demand = 10 * 0.192 * horas_3_años
print(f"On-Demand 3 años: ${coste_on_demand:,.2f}")
# Output: On-Demand 3 años: $50,457.60

# Escenario 2: Reserved 3yr All Upfront
coste_reserved = 10 * 0.077 * horas_3_años
print(f"Reserved 3yr All Upfront: ${coste_reserved:,.2f}")
# Output: Reserved 3yr All Upfront: $20,235.60

# AHORRO
ahorro = coste_on_demand - coste_reserved
print(f"Ahorro total: ${ahorro:,.2f} ({ahorro/coste_on_demand*100:.1f}%)")
# Output: Ahorro total: $30,222.00 (59.9%)
```

**3. Savings Plans**
- Alternativa más flexible a Reserved Instances
- Compromiso de gasto por hora (ej: $10/hora) durante 1 o 3 años
- Se aplica automáticamente a EC2, Lambda, Fargate
- Puede cambiar tipo de instancia, región, SO, tenancy
- Descuento similar a Reserved (hasta 66%)

**4. Spot Instances**
- Usar capacidad no utilizada de AWS
- Descuento: Hasta 90% vs On-Demand
- RIESGO: AWS puede interrumpir con 2 minutos de aviso si necesita capacidad
- Ideal para: Workloads tolerantes a fallos, batch processing, CI/CD

```python
# Ejemplo: Spot Instance para procesamiento batch
ec2 = boto3.client('ec2')

response = ec2.request_spot_instances(
    SpotPrice='0.05',  # Máximo que pagas (m5.xlarge On-Demand = $0.192)
    InstanceCount=10,
    LaunchSpecification={
        'ImageId': 'ami-batch-processor',
        'InstanceType': 'm5.xlarge',
        'KeyName': 'mi-clave',
        'SecurityGroupIds': ['sg-batch'],
        'UserData': '''#!/bin/bash
        # Procesar trabajo y guardar resultado en S3
        # Manejar interrupción (shutdown hook)
        '''
    },
    # Comportamiento si interrumpido
    InstanceInterruptionBehavior='terminate'  # o 'stop', 'hibernate'
)

# Si precio spot actual < $0.05: Se lanzan instancias
# Si precio spot sube > $0.05: AWS interrumpe con 2 min aviso
# Ahorro típico: 70-80% ($0.04-$0.06/hora vs $0.192)
```

**Servicios de Almacenamiento AWS:**

**Servicios Clave para Big Data y Analytics:**

#### Almacenamiento

**Amazon S3 (Simple Storage Service)**
- Almacenamiento de objetos
- 99.999999999% (11 noves) de durabilidad
- Base para Data Lakes

```python
# Ejemplo: Subir datos a S3

import boto3

s3 = boto3.client('s3')

# Subir archivo CSV
s3.upload_file(
    'datos_covid_ccaa.csv',
    'mi-datalake-sanitario',
    'raw/covid/2026/02/datos_covid_ccaa.csv'
)

# Subir con metadatos
s3.put_object(
    Bucket='mi-datalake-sanitario',
    Key='raw/covid/2026/02/datos_covid_ccaa.csv',
    Body=open('datos_covid_ccaa.csv', 'rb'),
    Metadata={
        'source': 'ministerio-sanidad',
        'date': '2026-02-19',
        'format': 'csv',
        'classification': 'publico'
    },
    ServerSideEncryption='AES256'
)
```

#### Procesamiento

**AWS Glue**
- ETL serverless
- Data Catalog automático
- Compatible con Spark

```python
# Ejemplo: Job ETL en AWS Glue

glue = boto3.client('glue')

# Crear job
job_name = 'etl-covid-cleaning'

glue.create_job(
    Name=job_name,
    Role='AWSGlueServiceRole',
    Command={
        'Name': 'glueetl',
        'ScriptLocation': 's3://mi-bucket/scripts/etl_covid.py',
        'PythonVersion': '3'
    },
    DefaultArguments={
        '--TempDir': 's3://mi-bucket/temp/',
        '--job-bookmark-option': 'job-bookmark-enable',
        '--enable-metrics': ''
    },
    MaxCapacity=10.0  # DPUs (Data Processing Units)
)

# Ejecutar job
glue.start_job_run(
    JobName=job_name,
    Arguments={
        '--INPUT_PATH': 's3://mi-datalake/raw/covid/',
        '--OUTPUT_PATH': 's3://mi-datalake/trusted/covid/',
        '--DATE': '2026-02-19'
    }
)
```

**Amazon EMR (Elastic MapReduce)**
- Cluster Hadoop/Spark managed
- Auto-scaling
- Integración con S3

```python
# Ya visto anteriormente, ejemplo completo en sección de auto-scaling
```

**AWS Lambda**
- Serverless compute
- Pago por ejecución
- Ideal para transformaciones ligeras

```python
# Ejemplo: Lambda para validar datos al subir a S3

# lambda_function.py
import json
import boto3
import pandas as pd
from io import StringIO

s3 = boto3.client('s3')

def lambda_handler(event, context):
    """
    Se dispara al subir CSV a bucket
    Valida formato y calidad
    """
    # Obtener info del archivo subido
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Descargar y leer CSV
    obj = s3.get_object(Bucket=bucket, Key=key)
    df = pd.read_csv(StringIO(obj['Body'].read().decode('utf-8')))
    
    # Validaciones
    errors = []
    
    if df.isnull().sum().sum() > len(df) * 0.1:
        errors.append("Más del 10% de valores nulos")
    
    if 'fecha' not in df.columns:
        errors.append("Falta columna 'fecha'")
    
    # Registrar resultado
    if errors:
        print(f"ERRORES en {key}: {errors}")
        # Mover a carpeta de errores
        s3.copy_object(
            Bucket=bucket,
            CopySource={'Bucket': bucket, 'Key': key},
            Key=key.replace('raw/', 'errors/')
        )
    else:
        print(f"OK: {key}")
        # Mover a carpeta validada
        s3.copy_object(
            Bucket=bucket,
            CopySource={'Bucket': bucket, 'Key': key},
            Key=key.replace('raw/', 'validated/')
        )
    
    return {
        'statusCode': 200,
        'body': json.dumps(f'Processed {key}')
    }
```

#### Analytics

**Amazon Athena**
- SQL sobre S3
- Serverless
- Pago por query (por TB escaneado)

```sql
-- Ejemplo: Crear tabla externa sobre datos en S3

CREATE EXTERNAL TABLE covid_casos (
    fecha DATE,
    ccaa STRING,
    provincia STRING,
    casos_confirmados INT,
    hospitalizados INT,
    uci INT,
    fallecidos INT
)
PARTITIONED BY (year INT, month INT)
STORED AS PARQUET
LOCATION 's3://mi-datalake/trusted/covid/'
TBLPROPERTIES ('parquet.compress'='SNAPPY');

-- Añadir particiones
MSCK REPAIR TABLE covid_casos;

-- Consulta analítica
SELECT 
    ccaa,
    SUM(casos_confirmados) as total_casos,
    SUM(fallecidos) as total_fallecidos,
    ROUND(SUM(fallecidos) * 100.0 / SUM(casos_confirmados), 2) as letalidad
FROM covid_casos
WHERE year = 2024
GROUP BY ccaa
ORDER BY total_casos DESC;
```

**Amazon Redshift**
- Data Warehouse columnar
- Petabyte-scale
- Compatible con PostgreSQL

```python
# Conectar y consultar Redshift

import psycopg2

conn = psycopg2.connect(
    host='mi-cluster.xxxx.eu-south-2.redshift.amazonaws.com',
    port=5439,
    database='dw_sanitario',
    user='admin',
    password='password'
)

cur = conn.cursor()

# Consulta multidimensional (OLAP)
query = """
SELECT 
    d.year,
    d.month,
    g.ccaa,
    g.provincia,
    SUM(f.casos) as total_casos,
    AVG(f.hospitalizados) as avg_hospitalizados
FROM fact_covid f
JOIN dim_fecha d ON f.fecha_id = d.fecha_id
JOIN dim_geografia g ON f.geografia_id = g.geografia_id
WHERE d.year >= 2023
GROUP BY CUBE(d.year, d.month, g.ccaa, g.provincia)
ORDER BY d.year, d.month;
"""

cur.execute(query)
results = cur.fetchall()
```

**Amazon QuickSight**
- BI y visualización
- Serverless
- ML Insights integrado

#### Machine Learning

**Amazon SageMaker**
- Plataforma completa de ML
- Notebooks Jupyter managed
- Training distribuido
- Deploy de modelos

```python
# Ejemplo: Training de modelo en SageMaker

import sagemaker
from sagemaker.sklearn.estimator import SKLearn

role = 'arn:aws:iam::123456789012:role/SageMakerRole'
session = sagemaker.Session()

# Script de training
sklearn_estimator = SKLearn(
    entry_point='train.py',
    role=role,
    instance_type='ml.m5.xlarge',
    framework_version='1.0-1',
    py_version='py3',
    hyperparameters={
        'n_estimators': 100,
        'max_depth': 10
    }
)

# Ejecutar training
sklearn_estimator.fit({'training': 's3://mi-bucket/data/training/'})

# Deploy modelo
predictor = sklearn_estimator.deploy(
    initial_instance_count=2,
    instance_type='ml.t2.medium'
)

# Predecir
import numpy as np
data = np.array([[25, 1, 0, 75, 120]])  # edad, sexo, diabetes, temp, presion
prediction = predictor.predict(data)
```

**Arquitectura de Referencia AWS para Data Lake:**

```
┌─────────────────────────────────────────────────────────┐
│                    FUENTES DE DATOS                     │
│  datos.gob.es | INE | Ministerio Sanidad | APIs OMS   │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│              INGESTA (AWS Data Pipeline)                │
│  Lambda | Kinesis Firehose | AWS Glue | API Gateway    │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                DATA LAKE (Amazon S3)                    │
│  ┌────────────┐  ┌────────────┐  ┌───────────────┐   │
│  │ Raw Zone   │→ │Trusted Zone│→ │ Refined Zone  │   │
│  │ (CSV/JSON) │  │ (Parquet)  │  │ (Agregado)    │   │
│  └────────────┘  └────────────┘  └───────────────┘   │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│            PROCESAMIENTO Y TRANSFORMACIÓN               │
│  AWS Glue | EMR (Spark) | Lambda | Step Functions      │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                  ANÁLISIS Y CONSUMO                     │
│  Athena (SQL ad-hoc) | Redshift (DW) | QuickSight (BI) │
│  SageMaker (ML) | EMR (Data Mining)                    │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                  GOBERNANZA Y SEGURIDAD                 │
│  Glue Data Catalog | Lake Formation | IAM | KMS        │
│  CloudTrail | Macie                                     │
└─────────────────────────────────────────────────────────┘
```

## 4.2. Microsoft Azure

**Servicios Clave para Big Data y Analytics:**

#### Almacenamiento

**Azure Data Lake Storage Gen2**
- Basado en Azure Blob Storage
- Jerárquico (carpetas reales)
- Compatible con Hadoop

```python
# Ejemplo: Acceder a Azure Data Lake

from azure.storage.filedatalake import DataLakeServiceClient

# Conectar
service_client = DataLakeServiceClient(
    account_url="https://midatalake.dfs.core.windows.net",
    credential="<access_key>"
)

# Crear sistema de archivos (contenedor)
file_system_client = service_client.create_file_system(
    file_system="datalake-sanitario"
)

# Subir archivo
file_client = file_system_client.get_file_client("raw/covid/2026/02/datos.csv")

with open("datos_covid.csv", "rb") as data:
    file_client.upload_data(data, overwrite=True)

# Establecer metadatos
file_client.set_metadata({
    'source': 'ministerio-sanidad',
    'classification': 'publico'
})
```

#### Procesamiento

**Azure Synapse Analytics**
- Sucesor de Azure SQL Data Warehouse
- Integra Data Warehouse + Big Data
- Spark managed + SQL serverless

```python
# Ejemplo: Notebook Spark en Synapse

# PySpark en Synapse Spark Pool

from pyspark.sql import SparkSession
from pyspark.sql.functions import *

spark = SparkSession.builder.appName("Analisis COVID").getOrCreate()

# Leer desde Data Lake
df_covid = spark.read.parquet(
    "abfss://datalake-sanitario@midatalake.dfs.core.windows.net/trusted/covid/"
)

# Transformaciones
df_agg = df_covid.groupBy("ccaa", year("fecha").alias("year")) \
    .agg(
        sum("casos_confirmados").alias("total_casos"),
        avg("hospitalizados").alias("avg_hospitalizados"),
        max("uci").alias("max_uci")
    ) \
    .orderBy("year", "ccaa")

# Escribir resultado
df_agg.write.mode("overwrite").parquet(
    "abfss://datalake-sanitario@midatalake.dfs.core.windows.net/refined/covid_agg/"
)
```

**Azure Data Factory**
- Orquestación de pipelines de datos
- ETL visual
- Integración con >100 conectores

```python
# Ejemplo: Pipeline en Azure Data Factory (definición JSON)

pipeline_definition = {
    "name": "Pipeline-ETL-COVID",
    "properties": {
        "activities": [
            {
                "name": "Copy-From-URL",
                "type": "Copy",
                "inputs": [
                    {
                        "referenceName": "HTTPDataset_COVID",
                        "type": "DatasetReference"
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "ADLSDataset_Raw",
                        "type": "DatasetReference"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "HttpSource",
                        "httpRequestTimeout": "00:01:40"
                    },
                    "sink": {
                        "type": "ParquetSink"
                    }
                }
            },
            {
                "name": "Data-Flow-Transformation",
                "type": "ExecuteDataFlow",
                "dependsOn": [
                    {
                        "activity": "Copy-From-URL",
                        "dependencyConditions": ["Succeeded"]
                    }
                ],
                "typeProperties": {
                    "dataFlow": {
                        "referenceName": "DataFlow_Clean_COVID",
                        "type": "DataFlowReference"
                    }
                }
            }
        ]
    }
}
```

**Azure Databricks**
- Plataforma Spark optimizada
- Notebooks colaborativos
- MLflow integrado

```python
# Ejemplo: Notebook en Databricks

# %python

# Leer datos
df = spark.read.format("delta").load("/mnt/datalake/covid/")

# Análisis exploratorio
display(df.groupBy("ccaa").count())

# Registrar como vista temporal para SQL
df.createOrReplaceTempView("covid")

# %sql
-- Análisis SQL
SELECT 
    ccaa,
    DATE_TRUNC('month', fecha) as mes,
    SUM(casos_confirmados) as casos_mes
FROM covid
WHERE fecha >= '2023-01-01'
GROUP BY ccaa, mes
ORDER BY mes, ccaa;
```

#### Analytics

**Azure Synapse SQL (Serverless)**
- Query SQL sobre Data Lake
- Sin infraestructura

```sql
-- Ejemplo: Query OPENROWSET sobre CSV en ADLS

SELECT 
    ccaa,
    COUNT(*) as num_registros,
    SUM(CAST(casos AS INT)) as total_casos
FROM OPENROWSET(
    BULK 'https://midatalake.dfs.core.windows.net/datalake/raw/covid/*.csv',
    FORMAT = 'CSV',
    PARSER_VERSION = '2.0',
    HEADER_ROW = TRUE
) AS covid_data
GROUP BY ccaa;
```

**Power BI**
- Líder en BI
- Integración nativa con Azure
- Actualización automática

#### Machine Learning

**Azure Machine Learning**
- Plataforma ML completa
- AutoML
- MLOps pipelines

```python
# Ejemplo: Training en Azure ML

from azureml.core import Workspace, Experiment, ScriptRunConfig, Environment
from azureml.core.compute import AmlCompute

# Conectar a workspace
ws = Workspace.from_config()

# Crear compute cluster
compute_target = AmlCompute(ws, 'ml-cluster')

# Definir entorno
env = Environment.from_conda_specification(
    name='sklearn-env',
    file_path='conda.yml'
)

# Configurar training
config = ScriptRunConfig(
    source_directory='./src',
    script='train.py',
    compute_target=compute_target,
    environment=env,
    arguments=[
        '--data-folder', ws.get_default_datastore(),
        '--n-estimators', 100,
        '--max-depth', 10
    ]
)

# Ejecutar experimento
experiment = Experiment(ws, 'prediccion-hospitalizacion')
run = experiment.submit(config)
run.wait_for_completion(show_output=True)

# Registrar modelo
model = run.register_model(
    model_name='modelo-hospitalizacion',
    model_path='outputs/model.pkl'
)
```

**Arquitectura de Referencia Azure para Data Lake:**

```
┌─────────────────────────────────────────────────────────┐
│                    FUENTES DE DATOS                     │
│  datos.gob.es | INE | Ministerio Sanidad | APIs OMS   │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                INGESTA (Azure Data Factory)             │
│  Copy Activity | Data Flows | Event Hubs | Functions   │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│       DATA LAKE (Azure Data Lake Storage Gen2)          │
│  ┌────────────┐  ┌────────────┐  ┌───────────────┐   │
│  │ Raw Zone   │→ │Trusted Zone│→ │ Refined Zone  │   │
│  │ (CSV/JSON) │  │ (Parquet)  │  │ (Delta Lake)  │   │
│  └────────────┘  └────────────┘  └───────────────┘   │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│            PROCESAMIENTO Y TRANSFORMACIÓN               │
│  Synapse Spark | Databricks | Data Factory | Functions │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                  ANÁLISIS Y CONSUMO                     │
│  Synapse SQL | Synapse Pipelines | Power BI            │
│  Azure ML | Databricks (Data Mining)                   │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                  GOBERNANZA Y SEGURIDAD                 │
│  Purview | Azure AD | Key Vault | Defender for Cloud   │
└─────────────────────────────────────────────────────────┘
```

## 4.3. Google Cloud Platform (GCP)

**Servicios Clave para Big Data y Analytics:**

#### Almacenamiento

**Google Cloud Storage**
- Object storage
- Clases de almacenamiento automáticas
- Base para Data Lake

```python
# Ejemplo: Trabajar con GCS

from google.cloud import storage

# Conectar
client = storage.Client()

# Crear bucket
bucket = client.create_bucket('mi-datalake-sanitario', location='europe-southwest1')  # Madrid

# Subir archivo
blob = bucket.blob('raw/covid/2026/02/datos.csv')
blob.upload_from_filename('datos_covid.csv')

# Establecer metadatos
metadata = {
    'source': 'ministerio-sanidad',
    'classification': 'publico',
    'format': 'csvdatos'
}
blob.metadata = metadata
blob.patch()

# Configurar lifecycle
bucket.lifecycle_rules = [{
    'action': {'type': 'SetStorageClass', 'storageClass': 'NEARLINE'},
    'condition': {'age': 30, 'matchesPrefix': ['raw/']}
}]
bucket.patch()
```

#### Procesamiento

**Google Cloud Dataproc**
- Hadoop/Spark managed
- Clusters efímeros (crear/destruir rápido)
- Integración con GCS

```python
# Ejemplo: Cluster Dataproc

from google.cloud import dataproc_v1

# Cliente
client = dataproc_v1.ClusterControllerClient(
    client_options={'api_endpoint': 'europe-southwest1-dataproc.googleapis.com'}
)

# Configuración del cluster
cluster_config = {
    'project_id': 'mi-proyecto',
    'cluster_name': 'cluster-analisis-covid',
    'config': {
        'master_config': {
            'num_instances': 1,
            'machine_type_uri': 'n1-standard-4'
        },
        'worker_config': {
            'num_instances': 4,
            'machine_type_uri': 'n1-standard-4'
        },
        'software_config': {
            'image_version': '2.0-debian10',
            'properties': {
                'spark:spark.executor.memory': '4g'
            }
        }
    }
}

# Crear cluster
operation = client.create_cluster(
    request={'projectId': 'mi-proyecto', 'region': 'europe-southwest1', 'cluster': cluster_config}
)
cluster = operation.result()

# Ejecutar job Spark
job_client = dataproc_v1.JobControllerClient(
    client_options={'api_endpoint': 'europe-southwest1-dataproc.googleapis.com'}
)

job_config = {
    'placement': {'cluster_name': 'cluster-analisis-covid'},
    'pyspark_job': {
        'main_python_file_uri': 'gs://mi-bucket/scripts/analisis_covid.py',
        'args': ['--input', 'gs://mi-datalake/raw/covid/', '--output', 'gs://mi-datalake/trusted/covid/']
    }
}

job = job_client.submit_job(request={'project_id': 'mi-proyecto', 'region': 'europe-southwest1', 'job': job_config})
```

**Google Cloud Dataflow**
- Apache Beam managed
- Streaming y batch unificado
- Auto-scaling

```python
# Ejemplo: Pipeline Dataflow

import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions

# Opciones
options = PipelineOptions(
    project='mi-proyecto',
    runner='DataflowRunner',
    region='europe-southwest1',
    temp_location='gs://mi-bucket/temp/',
    staging_location='gs://mi-bucket/staging/'
)

# Pipeline
with beam.Pipeline(options=options) as pipeline:
    (pipeline
     | 'Leer CSV' >> beam.io.ReadFromText('gs://mi-datalake/raw/covid/*.csv', skip_header_lines=1)
     | 'Parsear' >> beam.Map(lambda line: line.split(','))
     | 'Filtrar' >> beam.Filter(lambda row: row[2] != '' and int(row[2]) > 0)  # casos > 0
     | 'Transformar' >> beam.Map(lambda row: {
         'fecha': row[0],
         'ccaa': row[1],
         'casos': int(row[2]),
         'hospitalizados': int(row[3]) if row[3] else 0
     })
     | 'Escribir JSON' >> beam.io.WriteToText('gs://mi-datalake/trusted/covid/processed', file_name_suffix='.json')
    )
```

#### Analytics

**Google BigQuery**
- Data Warehouse serverless
- Petabyte-scale
- SQL estándar
- ML integrado (BigQuery ML)

```sql
-- Ejemplo: Análisis en BigQuery

-- Crear tabla externa sobre GCS
CREATE EXTERNAL TABLE `mi-proyecto.datalake.covid_raw`
OPTIONS (
  format = 'CSV',
  uris = ['gs://mi-datalake/raw/covid/*.csv'],
  skip_leading_rows = 1
);

-- Query analítica
SELECT 
    ccaa,
    EXTRACT(YEAR FROM fecha) as year,
    EXTRACT(MONTH FROM fecha) as month,
    SUM(casos_confirmados) as total_casos,
    AVG(hospitalizados) as avg_hospitalizados,
    MAX(uci) as max_uci
FROM `mi-proyecto.datalake.covid_raw`
WHERE fecha >= '2023-01-01'
GROUP BY ccaa, year, month
ORDER BY year, month, ccaa;

-- BigQuery ML: Entrenar modelo directamente con SQL
CREATE OR REPLACE MODEL `mi-proyecto.models.predict_hospitalizacion`
OPTIONS(
  model_type='logistic_reg',
  input_label_cols=['hospitalized']
) AS
SELECT
    edad,
    sexo,
    comorbilidades,
    sintomas,
    hospitalized
FROM `mi-proyecto.datalake.pacientes_clean`
WHERE fecha >= '2023-01-01';

-- Predecir
SELECT
    paciente_id,
    predicted_hospitalized,
    predicted_hospitalized_probs[OFFSET(1)].prob as probabilidad
FROM ML.PREDICT(
    MODEL `mi-proyecto.models.predict_hospitalizacion`,
    (SELECT * FROM `mi-proyecto.datalake.pacientes_nuevos`)
);
```

**Looker / Data Studio**
- BI y visualización
- Integración nativa con BigQuery

#### Machine Learning

**Vertex AI**
- Plataforma ML unificada
- AutoML
- Notebooks managed
- Deploy de modelos

```python
# Ejemplo: Training en Vertex AI

from google.cloud import aiplatform

aiplatform.init(project='mi-proyecto', location='europe-southwest1')

# Crear dataset
dataset = aiplatform.TabularDataset.create(
    display_name='covid-hospitalizacion',
    gcs_source='gs://mi-bucket/data/training_data.csv'
)

# Training con AutoML
job = aiplatform.AutoMLTabularTrainingJob(
    display_name='automl-hospitalizacion',
    optimization_prediction_type='classification',
    optimization_objective='maximize-au-roc'
)

model = job.run(
    dataset=dataset,
    target_column='hospitalized',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    budget_milli_node_hours=1000,
    model_display_name='modelo-hospitalizacion'
)

# Deploy
endpoint = model.deploy(
    deployed_model_display_name='hospitalizacion-endpoint',
    machine_type='n1-standard-4',
    min_replica_count=1,
    max_replica_count=3
)

# Predecir
prediction = endpoint.predict(instances=[
    {'edad': 65, 'sexo': 'M', 'comorbilidades': 2, 'sintomas_graves': 1}
])
```

**Arquitectura de Referencia GCP para Data Lake:**

```
┌─────────────────────────────────────────────────────────┐
│                    FUENTES DE DATOS                     │
│  datos.gob.es | INE | Ministerio Sanidad | APIs OMS   │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                 INGESTA (Cloud Composer)                │
│  Pub/Sub | Cloud Functions | Dataflow | Workflows      │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│          DATA LAKE (Google Cloud Storage)               │
│  ┌────────────┐  ┌────────────┐  ┌───────────────┐   │
│  │ Raw Zone   │→ │Trusted Zone│→ │ Refined Zone  │   │
│  │ (CSV/JSON) │  │ (Avro)     │  │ (Parquet)     │   │
│  └────────────┘  └────────────┘  └───────────────┘   │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│            PROCESAMIENTO Y TRANSFORMACIÓN               │
│  Dataproc (Spark) | Dataflow | Cloud Functions         │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                  ANÁLISIS Y CONSUMO                     │
│  BigQuery (DW) | Looker/Data Studio (BI)               │
│  Vertex AI (ML) | Dataproc (Data Mining)               │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                  GOBERNANZA Y SEGURIDAD                 │
│  Data Catalog | IAM | Cloud KMS | Security Command     │
│  Center | DLP API                                       │
└─────────────────────────────────────────────────────────┘
```

## 4.4. Comparación de Proveedores

**Servicios Equivalentes:**

| Función | AWS | Azure | GCP |
|---------|-----|-------|-----|
| **Object Storage** | S3 | Blob Storage / ADLS Gen2 | Cloud Storage |
| **Data Warehouse** | Redshift | Synapse SQL | BigQuery |
| **Spark Managed** | EMR | Databricks / Synapse | Dataproc |
| **ETL Serverless** | Glue | Data Factory | Dataflow |
| **SQL Serverless** | Athena | Synapse Serverless | BigQuery |
| **ML Platform** | SageMaker | Azure ML | Vertex AI |
| **BI** | QuickSight | Power BI | Looker / Data Studio |
| **Data Catalog** | Glue Catalog | Purview | Data Catalog |
| **NoSQL** | DynamoDB | Cosmos DB | Firestore / Bigtable |
| **Streaming** | Kinesis | Event Hubs | Pub/Sub |

**Fortalezas por Proveedor:**

**AWS:**
✅ Catálogo de servicios más amplio
✅ Madurez y ecosistema
✅ Regiones globales (incluida España)
✅ Flexibilidad máxima

**Azure:**
✅ Integración con Microsoft (Office, AD, etc.)
✅ Power BI (líder en BI)
✅ Híbrido cloud (Azure Arc)
✅ Enterprise focus

**GCP:**
✅ BigQuery (mejor DW serverless)
✅ Precios competitivos
✅ Innovación en ML/AI
✅ K8s nativo (GKE)

**Criterios de Selección:**

**Elegir AWS si:**
- Necesitas máxima flexibilidad
- Ecosistema de partners importante
- Multi-región crítica

**Elegir Azure si:**
- Ya usas Microsoft (Windows, Office 365, AD)
- Power BI es crítico
- Entorno híbrido on-premise + cloud

**Elegir GCP si:**
- BigQuery es ideal para tu caso (DW serverless)
- Foco en ML/AI avanzado
- Contenedores y Kubernetes

**Para el proyecto final del curso (datos abiertos sanitarios):**
Recomendación: **AWS** o **Azure**
- Ambos tienen región en España (cumplimiento RGPD facilitado)
- Documentación extensa en español
- Comunidad activa en España
- Azure si usas Power BI para visualización final

---

# BLOQUE 3: DATA LAKES Y ALMACENAMIENTO ANALÍTICO

---

