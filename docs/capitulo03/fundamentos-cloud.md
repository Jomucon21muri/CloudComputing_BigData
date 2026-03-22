# CAPÍTULO 3: Fundamentos de Cloud Computing

## 3.1. Definición y características esenciales

**¿Qué es Cloud Computing?:**

**Definiciones fundamentales:**

**Definición NIST (National Institute of Standards and Technology):**
> "Cloud computing is a model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction."

**En español:**
> "Cloud computing es un modelo que permite el acceso conveniente y ubicuo bajo demanda a través de la red a un conjunto compartido de recursos computacionales configurables (ej.: redes, servidores, almacenamiento, aplicaciones y servicios) que pueden ser rápidamente aprovisionados y liberados con un mínimo de esfuerzo de gestión o interacción con el proveedor del servicio."

**Definición práctica:**

- Es un paradigma que posibilita el acceso ubicuo bajo demanda a servicios TIC accesibles a través de Internet
- **Computación como servicio:** Externalización de recursos IT
- **Economía de escala:** Grandes proveedores optimizan costes
- **Foco en el negocio:** Permite centrarse en la lógica de negocio y no en la infraestructura informática

**El Modelo Cloud del NIST: 5-3-4:**

El modelo cloud del NIST es conocido popularmente como **5-3-4** y sirve como referencia también a nivel europeo:

```
┌───────────────────────────────────────────────────┐
│           MODELO CLOUD NIST (5-3-4)               │
│                                                   │
│   5 CARACTERÍSTICAS ESENCIALES                    │
│   ├─ Autoservicio bajo demanda                    │
│   ├─ Amplio acceso a la red                       │
│   ├─ Agrupación de recursos (pooling)             │
│   ├─ Elasticidad rápida                           │
│   └─ Servicio medido                              │
│                                                   │
│   3 MODELOS DE SERVICIO                           │
│   ├─ SaaS (Software as a Service)                 │
│   ├─ PaaS (Platform as a Service)                 │
│   └─ IaaS (Infrastructure as a Service)           │
│                                                   │
│   4 MODELOS DE DESPLIEGUE                         │
│   ├─ Nube Pública                                 │
│   ├─ Nube Privada                                 │
│   ├─ Nube Comunitaria                             │
│   └─ Nube Híbrida                                 │
└───────────────────────────────────────────────────┘
```

**Cloud Computing en la nueva economía digital:**

**Piezas claves de la economía digital:**

```
Mobile         Social       Cloud         Business        Digital
Technology  →  Media    →  Computing  →  Intelligence  →  Economy
```

**Impacto:**

- ✅ Transformación digital de industrias tradicionales
- ✅ Fomento de start-ups tecnológicas
- ✅ Democratización en el acceso a recursos TIC
- ✅ Base en el desarrollo de nuevas tecnologías: Big Data, IoT, Machine Learning, AI

**El Cloud Supone un Cambio de Paradigma:**

**En la gestión de recursos por ISPs:**

**Antes (Infraestructura Tradicional):**

- Servidores físicos dimensionados para pico de demanda
- Recursos frecuentemente infrautilizados
- Escalabilidad lenta (compra de hardware)
- Alto CAPEX (inversión inicial)

**Ahora (Cloud Computing):**

- **Plataformas flexibles y "elásticas"**
  - Adaptación a la demanda real
  - Soporta mejor los picos de demanda
  - Escalabilidad automática
  - Tolerancia a fallas integrada

- **Virtualización como base**
  - Recursos virtualizados bajo la forma de máquinas virtuales (VMs)
  - VMs se despliegan dinámicamente bajo demanda
  - Recursos expuestos como servicios a través de interfaces estandarizadas

**5 Características Esenciales (Detalladas):**

**1. Autoservicio Bajo Demanda (On-demand Self-Service):**

> **Definición:** El consumidor puede aprovisionar capacidades de cómputo (tiempo de servidor, almacenamiento en red) de forma unilateral según lo necesite, sin requerir interacción humana con el proveedor del servicio.

**Implicaciones:**

- **Cuándo (When):** Consumes el servicio cuando quieras
- **Sin esperas:** No hay proceso de aprobación manual
- **Interfaz self-service:** Portal web, API, CLI

**Ejemplo práctico:**

```python
# Aprovisionar recursos en AWS sin intervención humana
import boto3

ec2 = boto3.client('ec2')

# Crear instancia en 2 minutos
response = ec2.run_instances(
    ImageId='ami-0c55b159cbfafe1f0',  # Ubuntu 20.04
    InstanceType='t3.medium',
    MinCount=1,
    MaxCount=1,
    KeyName='my-key',
    TagSpecifications=[{
        'ResourceType': 'instance',
        'Tags': [{'Key': 'Name', 'Value': 'Mi-Servidor-Analisis'}]
    }]
)

print("Servidor creado con ID:", response['Instances'][0]['InstanceId'])
# En ~2 minutos, el servidor está operativo
```

**Contraste con Infraestructura Tradicional:**

| Infraestructura Local | Cloud (Autoservicio) |
|----------------------|----------------------|
| Solicitud via ticket IT | Portal web / API |
| Días o semanas de espera | Minutos |
| Aprobaciones múltiples | Inmediato (dentro de límites) |
| Dimensionamiento rígido | Ajuste dinámico |

**Problema resuelto: ¿Cómo dimensionar la capacidad?**

```
ANTES:
Servidor físico dimensionado para pico máximo
→ 80% del tiempo infrautilizado
→ Alto coste, baja eficiencia

AHORA:
Aprovisionamiento bajo demanda
→ Consume lo que se necesita
→ Pago por lo que se consume
→ Elasticidad automática
```

**2. Amplio Acceso a la Red (Broad Network Access):**

> **Definición:** Las capacidades están disponibles a través de la red y se accede mediante mecanismos estándar que promueven su uso por plataformas heterogéneas thick o thin client (ej.: teléfonos móviles, tablets, laptops, workstations).

**Implicaciones:**

- **Dónde (Where):** Consumes el servicio desde cualquier lugar
- **Cualquier dispositivo:** Smartphone, tablet, laptop, desktop
- **Protocolos estándar:** HTTP/HTTPS, APIs REST
- **Ubicuidad:** Trabajo remoto, movilidad

**Ejemplo de acceso:**

```python
# Acceso desde cualquier dispositivo con Python
import requests

# Desde laptop en la oficina
# Desde tablet en casa
# Desde smartphone en un café
# Mismo código, misma API

API_ENDPOINT = "https://api.miservicioencloud.com/v1/datos"

headers = {
    'Authorization': 'Bearer mi-token-de-acceso',
    'Content-Type': 'application/json'
}

response = requests.get(API_ENDPOINT + "/pacientes/12345", headers=headers)
datos_paciente = response.json()

# Funciona desde cualquier lugar con Internet
```

**Beneficio para organizaciones:**

- Trabajo remoto sin VPN compleja
- Colaboración global
- Acceso 24/7
- Disaster Recovery: Si oficina inaccesible, seguir trabajando desde casa

**3. Agrupación de recursos (Resource Pooling):**

>**Definición:** Los recursos de cómputo del proveedor se agrupan en un pool para servir a múltiples consumidores usando un modelo multi-tenant, con diferentes recursos físicos y virtuales asignados dinámicamente según la demanda del consumidor.

**Características:**

- **Pool de infraestructuras compartidas**
- **Virtualización:** Múltiples VMs en mismo hardware físico
- **Capacidades como servicio (*aaS):** IaaS, PaaS, SaaS
- **Independencia de localización:** Usuario generalmente no sabe ubicación exacta

**Multitenancy:**

```
┌────────────────────────────────────────────────┐
│       SERVIDOR FÍSICO (Multi-tenancy)          │
│                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ VM       │  │ VM       │  │ VM       │      │
│  │ Cliente A│  │ Cliente B│  │ Cliente C│      │
│  │          │  │          │  │          │      │
│  │ 2 vCPU   │  │ 4 vCPU   │  │ 1 vCPU   │      │
│  │ 4 GB RAM │  │ 8 GB RAM │  │ 2 GB RAM │      │
│  └──────────┘  └──────────┘  └──────────┘      │
│                                                │
│         Servidor: 16 vCPU, 32 GB RAM           │
│         (Recursos compartidos eficientemente)  │
└────────────────────────────────────────────────┘
```

**Ventajas del pooling:**

- **Eficiencia:** Mejor utilización de recursos físicos
- **Coste reducido:** Economía de escala
- **Aislamiento:** Cada tenant independientemente seguro

**Ejemplo:** Cómo (How) se proveen los recursos

```python
# AWS provee recursos de un pool compartido

# Solicitas 1 VM
ec2.run_instances(
    InstanceType='t3.medium',  # 2 vCPU, 4 GB RAM
    MinCount=1,
    MaxCount=1
)

# AWS:
# 1. Busca en pool de servidores físicos con capacidad disponible
# 2. Asigna recursos virtualizados de ese servidor físico
# 3. Tu VM puede estar en el mismo hardware que VMs de otros clientes
# 4. Aislamiento garantizado mediante virtualización
# 5. No sabes (ni necesitas saber) el servidor físico exacto
```

**4. Elasticidad Rápida (Rapid Elasticity):**

>**Definición:** Las capacidades pueden ser elástica y rápidamente aprovisionadas (a veces automáticamente) para escalar rápidamente hacia afuera (scale out) y liberadas para escalar hacia adentro (scale in). Para el consumidor, las capacidades disponibles frecuentemente parecen ilimitadas.

**Características:**

- **Scale out/in:** Crecer o decrecer según demanda
- **Automático o manual:** Auto-scaling rules
- **Rápido:** Minutos, no días/semanas
- **Prácticamente ilimitado:** Pool de recursos masivo del proveedor

**Tipos de escalado:**

| Tipo | Descripción | Ejemplo |
|------|-------------|---------|
| **Scaling Out (Horizontal)** | Agregar más instancias | 2 VMs → 10 VMs |
| **Scaling In (Horizontal)** | Reducir instancias | 10 VMs → 2 VMs |
| **Scaling Up (Vertical)** | Aumentar capacidad instancia | t3.medium → t3.2xlarge |
| **Scaling Down (Vertical)** | Reducir capacidad instancia | t3.2xlarge → t3.medium |

**Ejemplo: Auto-scaling basado en CPU**

```python
import boto3

autoscaling = boto3.client('autoscaling')
cloudwatch = boto3.client('cloudwatch')

# Crear grupo de auto-scaling
autoscaling.create_auto_scaling_group(
    AutoScalingGroupName='mi-app-asg',
    LaunchTemplate={
        'LaunchTemplateId': 'lt-xyz',
        'Version': '$Latest'
    },
    MinSize=2,      # Mínimo 2 instancias siempre
    MaxSize=20,     # Máximo 20 instancias
    DesiredCapacity=2,
    TargetGroupARNs=['arn:aws:elasticloadbalancing:...'],
    VPCZoneIdentifier='subnet-abc123,subnet-def456'
)

# Política de escalado: Scale out cuando CPU > 70%
autoscaling.put_scaling_policy(
    AutoScalingGroupName='mi-app-asg',
    PolicyName='scale-out-on-high-cpu',
    PolicyType='TargetTrackingScaling',
    TargetTrackingConfiguration={
        'PredefinedMetricSpecification': {
            'PredefinedMetricType': 'ASGAverageCPUUtilization'
        },
        'TargetValue': 70.0  # Mantener CPU ~70%
    }
)

# Resultado:
# - Si CPU > 70%: AWS añade automáticamente más VMs
# - Si CPU < 70%: AWS reduce VMs (hasta mínimo de 2)
# - Respuesta en 2-5 minutos
```

**Caso de uso real: E-commerce en Black Friday**

```
Tráfico normal (lunes-jueves):
→ 2 servidores web (t3.medium)
→ Coste: $0.0416/hora × 2 × 24 = ~$2/día

Black Friday (viernes):
→ Auto-scaling detecta pico de tráfico
→ Escala a 20 servidores (t3.medium)
→ Coste: $0.0416/hora × 20 × 24 = ~$20/día
→ Solo ese día, reduces a 2 al día siguiente

Alternativa tradicional:
→ Comprar 20 servidores físicos para 1 día al año
→ 19 servidores infrautilizados 364 días
→ CAPEX: ~$50,000
→ Cloud saving: ~99.8% del coste total
```

**5. Servicio Medido (Measured Service):**


>**Definición:** Los sistemas cloud automáticamente controlan y optimizan el uso de recursos mediante una capacidad de medición en algún nivel de abstracción apropiado al tipo de servicio (ej.: almacenamiento, procesamiento, ancho de banda, cuentas de usuario activas). El uso de recursos puede ser monitoreado, controlado y reportado, proveyendo transparencia tanto al proveedor como al consumidor.

**Características:**

- **Pago por uso:** Pay-as-you-go
- **Cuánto (How much):** Mides exactamente lo que consumes
- **Optimización:** Identifica y elimina desperdicios
- **Transparencia:** Dashboards de coste en tiempo real

**Métricas medidas según servicio:**

| Servicio | Métricas Medidas |
|----------|------------------|
| **Cómputo (EC2)** | Horas de instancia, tipo de instancia |
| **Almacenamiento (S3)** | GB almacenados, requests (GET/PUT) |
| **Base de Datos (RDS)** | Horas DB, storage, I/O operations, backup storage |
| **Red** | GB transferidos (egress), requests |
| **Lambda** | Número de invocaciones, GB-seconds de cómputo |

**Ejemplo: Monitorización de uso y costes**

```python
import boto3
from datetime import datetime, timedelta

ce = boto3.client('ce')  # Cost Explorer

# Obtener coste del último mes
end = datetime.now().strftime('%Y-%m-%d')
start = (datetime.now() - timedelta(days=30)).strftime('%Y-%m-%d')

response = ce.get_cost_and_usage(
    TimePeriod={
        'Start': start,
        'End': end
    },
    Granularity='DAILY',
    Metrics=['UnblendedCost'],
    GroupBy=[
        {'Type': 'DIMENSION', 'Key': 'SERVICE'},
    ]
)

# Analizar costes por servicio
for result in response['ResultsByTime']:
    date = result['TimePeriod']['Start']
    print(f"\nFecha: {date}")
    for group in result['Groups']:
        service = group['Keys'][0]
        cost = group['Metrics']['UnblendedCost']['Amount']
        print(f"  {service}: ${float(cost):.2f}")

# Output ejemplo:
# Fecha: 2026-02-19
#   Amazon EC2: $245.32
#   Amazon S3: $12.45
#   Amazon RDS: $89.12
#   AWS Lambda: $3.21
#   TOTAL: $350.10
```

**Beneficios de la medición:**

1. **Control de costes:**
   ```python
   # Configurar alarma si coste > $500/mes
   cloudwatch = boto3.client('cloudwatch')
   
   cloudwatch.put_metric_alarm(
       AlarmName='budget-alert',
       ComparisonOperator='GreaterThanThreshold',
       EvaluationPeriods=1,
       MetricName='EstimatedCharges',
       Namespace='AWS/Billing',
       Period=86400,  # 1 día
       Statistic='Maximum',
       Threshold=500.0,
       ActionsEnabled=True,
       AlarmActions=['arn:aws:sns:us-east-1:123456789012:billing-alerts']
   )
   ```

2. **Optimización:**
   - Identifica recursos no utilizados
   - Cambia a instancias más eficientes
   - Elimina snapshots antiguos

3. **Presupuestos:**
   - Planificación financiera precisa
   - Asignación de costes por proyecto/departamento
   - Análisis de ROI

## 3.2. Modelos de servicio (IaaS, PaaS, SaaS)

**Visión General de los Modelos:**

Los servicios cloud se modelan en varias categorías conocidas como "XaaS" (Anything as a Service):

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│    Aplicaciones          SaaS    ]                      │
│    Datos                         ]  Software            │
│    Runtime                       ]  as a Service        │
│    Middleware           PaaS     ]                      │
│    SO                            ]  Platform            │
│    Virtualización                ]  as a Service        │
│    Servers              IaaS     ]                      │
│    Storage                       ]  Infrastructure      │
│    Networking                    ]  as a Service        │
│                                                         │
│    ◄──────► Gestionas tú                                │
│    ████████ Gestiona el proveedor                       │
└─────────────────────────────────────────────────────────┘
```

**Comparativa de responsabilidades:**

| Capa | On-Premise | IaaS | PaaS | SaaS |
|------|------------|------|------|------|
| **Aplicaciones** | 👤 | 👤 | 👤 | ☁️ |
| **Datos** | 👤 | 👤 | 👤 | ☁️ |
| **Runtime** | 👤 | 👤 | ☁️ | ☁️ |
| **Middleware** | 👤 | 👤 | ☁️ | ☁️ |
| **Sistema Operativo** | 👤 | 👤 | ☁️ | ☁️ |
| **Virtualización** | 👤 | ☁️ | ☁️ | ☁️ |
| **Servidores** | 👤 | ☁️ | ☁️ | ☁️ |
| **Almacenamiento** | 👤 | ☁️ | ☁️ | ☁️ |
| **Redes** | 👤 | ☁️ | ☁️ | ☁️ |

👤 = Tú gestionas | ☁️ = Proveedor gestiona

**3.2.1. IaaS (Infrastructure as a Service):**

**Definición:**

**Infraestructura como Servicio:** El consumidor aprovisiona recursos de computación (CPU, almacenamiento, red) en los que ejecuta su software (incluidas aplicaciones y sistemas operativos).

**Control:**

- ✅ El consumidor **SÍ controla** los sistemas operativos, el almacenamiento y las aplicaciones desplegadas
- ✅ Posiblemente control limitado sobre componentes de red (ej.: firewalls, balanceadores)
- ❌ El consumidor **NO controla** la infraestructura cloud subyacente

**Características Clave:**

**Lo que ofrece IaaS:**

- Servicios de hosting mediante virtualización
- Redimensión sencilla y rápida de las "máquinas virtuales"
  - Agregar o quitar procesadores (vCPUs)
  - Incrementar el almacenamiento
  - Incrementar la memoria RAM
  - Arranque y parada de instancias
- **Se paga por la capacidad que se necesita**

**Lo que debes gestionar tú:**

- Máquinas virtuales (creación, configuración, eliminación)
- Sistemas operativos (instalación, patches, updates)
- Middleware (frameworks, librerías)
- Aplicaciones y datos
- **NO necesitas** mantener ni actualizar la infraestructura del datacenter

**Ventajas y Desventajas de IaaS:**

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| **Control total** sobre el SO y aplicaciones | **Debes gestionar** el SO y actualizaciones de seguridad |
| **Flexibilidad máxima** en configuración | **Requiere conocimientos técnicos** (administración de sistemas) |
| **No necesitas comprar hardware** físico | **Responsabilidad** de backup, patching, monitoring |
| **Escalabilidad rápida** (minutos vs meses) | **Vendor lock-in** potencial (APIs propietarias) |
| **Pago por uso** (sin CAPEX masivo) | |
| **Alta disponibilidad** mediante replicación en múltiples zonas | |

**Proveedores Principales de IaaS:**

| Proveedor | Servicio Principal | Características |
|-----------|-------------------|-----------------|
| **AWS** | Amazon EC2 (Elastic Compute Cloud) | Líder de mercado, mayor variedad de tipos de instancia |
| **Microsoft Azure** | Azure Virtual Machines | Integración con ecosistema Microsoft |
| **Google Cloud** | Google Compute Engine (GCE) | Alta performance networking, GPUs potentes |
| **IBM Cloud** | IBM Cloud Virtual Servers | Enfoque enterprise, mainframe integration |
| **Oracle Cloud** | Oracle Cloud Infrastructure (OCI) | Optimizado para workloads Oracle Database |
| **Alibaba Cloud** | Elastic Compute Service (ECS) | Líder en Asia, crecimiento global |

**Fuente de comparación:** https://www.g2crowd.com/categories/infrastructure-as-a-service-iaas

**Caso de Uso Detallado: IaaS para Análisis de Datos:**

```python
# Ejemplo: Levantar cluster de análisis con Apache Spark en AWS EC2
import boto3

ec2 = boto3.resource('ec2')
client = boto3.client('ec2')

# Configuración del cluster
INSTANCE_TYPE_MASTER = 'm5.2xlarge'  # 8 vCPU, 32 GB RAM
INSTANCE_TYPE_WORKER = 'r5.4xlarge'  # 16 vCPU, 128 GB RAM (optimized for memory)
NUM_WORKERS = 5

# Script de inicialización (User Data)
user_data_master = '''#!/bin/bash
# Actualizar sistema
apt-get update && apt-get upgrade -y

# Instalar Java
apt-get install -y openjdk-11-jdk

# Instalar Spark
wget https://dlcdn.apache.org/spark/spark-3.5.0/spark-3.5.0-bin-hadoop3.tgz
tar xvf spark-3.5.0-bin-hadoop3.tgz
mv spark-3.5.0-bin-hadoop3 /opt/spark

# Configurar como maestro
echo "export SPARK_MASTER_HOST=$(hostname -I | awk '{print $1}')" >> /opt/spark/conf/spark-env.sh
/opt/spark/sbin/start-master.sh
'''

user_data_worker = '''#!/bin/bash
apt-get update && apt-get upgrade -y
apt-get install -y openjdk-11-jdk
wget https://dlcdn.apache.org/spark/spark-3.5.0/spark-3.5.0-bin-hadoop3.tgz
tar xvf spark-3.5.0-bin-hadoop3.tgz
mv spark-3.5.0-bin-hadoop3 /opt/spark

# Conectar al maestro (IP se pasa como parámetro)
MASTER_IP="PLACEHOLDER"
/opt/spark/sbin/start-worker.sh spark://$MASTER_IP:7077
'''

# 1. Crear instancia MASTER
master_instance = ec2.create_instances(
    ImageId='ami-0c55b159cbfafe1f0',  # Ubuntu 20.04
    InstanceType=INSTANCE_TYPE_MASTER,
    MinCount=1,
    MaxCount=1,
    KeyName='my-spark-key',
    UserData=user_data_master,
    SecurityGroupIds=['sg-spark-cluster'],  # Puertos: 7077, 8080, 4040
    TagSpecifications=[{
        'ResourceType': 'instance',
        'Tags': [
            {'Key': 'Name', 'Value': 'Spark-Master'},
            {'Key': 'Role', 'Value': 'Master'},
            {'Key': 'Project', 'Value': 'BigData-Analysis'}
        ]
    }]
)[0]

print(f"Master creado: {master_instance.id}")

# Esperar a que master tenga IP
master_instance.wait_until_running()
master_instance.load()
master_ip = master_instance.private_ip_address
print(f"Master IP: {master_ip}")

# 2. Crear instancias WORKER
worker_user_data = user_data_worker.replace("PLACEHOLDER", master_ip)

worker_instances = ec2.create_instances(
    ImageId='ami-0c55b159cbfafe1f0',
    InstanceType=INSTANCE_TYPE_WORKER,
    MinCount=NUM_WORKERS,
    MaxCount=NUM_WORKERS,
    KeyName='my-spark-key',
    UserData=worker_user_data,
    SecurityGroupIds=['sg-spark-cluster'],
    TagSpecifications=[{
        'ResourceType': 'instance',
        'Tags': [
            {'Key': 'Name', 'Value': f'Spark-Worker'},
            {'Key': 'Role', 'Value': 'Worker'},
            {'Key': 'Project', 'Value': 'BigData-Analysis'}
        ]
    }]
)

print(f"{NUM_WORKERS} workers creados")

# 3. Estimación de coste
cost_master = 0.384  # $/hora m5.2xlarge (US East)
cost_worker = 1.008  # $/hora r5.4xlarge
cost_per_hour = cost_master + (cost_worker * NUM_WORKERS)
cost_per_day = cost_per_hour * 24
cost_per_month = cost_per_day * 30

print(f"\nEstimación de costes:")
print(f"  Por hora: ${cost_per_hour:.2f}")
print(f"  Por día (24h): ${cost_per_day:.2f}")
print(f"  Por mes (30 días 24/7): ${cost_per_month:.2f}")

# Si solo usas 8 horas/día, 22 días/mes:
cost_real = cost_per_hour * 8 * 22
print(f"  Real (8h/día, 22 días/mes): ${cost_real:.2f}")

# Resultado:
# Master: m5.2xlarge   → $0.384/h
# 5 Workers: r5.4xlarge → $5.04/h
# TOTAL: $5.424/hora
#
# 24/7 durante 30 días: $3,905/mes
# Solo cuando se usa (8h/día, 22 días): $954/mes
#
# ¡Clave: Apagar cuando no se usa!
```

**Optimización de costes:**

```python
# Script para apagar cluster al final del día laboral
import boto3
from datetime import datetime

ec2 = boto3.client('ec2')

def stop_spark_cluster():
    """Detener cluster Spark fuera de horario laboral"""
    # Buscar instancias del proyecto
    response = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Project', 'Values': ['BigData-Analysis']},
            {'Name': 'instance-state-name', 'Values': ['running']}
        ]
    )
    
    instance_ids = []
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instance_ids.append(instance['InstanceId'])
    
    if instance_ids:
        ec2.stop_instances(InstanceIds=instance_ids)
        print(f"Detenidas {len(instance_ids)} instancias")
        # Ahorro: De ~$130/día a ~$43/día (67% ahorro)
    else:
        print("No hay instancias en ejecución")

def start_spark_cluster():
    """Iniciar cluster al comenzar jornada laboral"""
    response = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Project', 'Values': ['BigData-Analysis']},
            {'Name': 'instance-state-name', 'Values': ['stopped']}
        ]
    )
    
    instance_ids = []
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instance_ids.append(instance['InstanceId'])
    
    if instance_ids:
        ec2.start_instances(InstanceIds=instance_ids)
        print(f"Iniciadas {len(instance_ids)} instancias")
    else:
        print("No hay instancias detenidas")

# Programar con cron o AWS EventBridge:
# 09:00 AM: start_spark_cluster()
# 06:00 PM: stop_spark_cluster()
```

**3.2.2. PaaS (Platform as a Service):**

**Definición:**

**Plataforma como Servicio:** El consumidor despliega aplicaciones (propias o adquiridas) desarrolladas usando lenguajes de programación, librerías, servicios y herramientas soportados por el proveedor, en la infraestructura cloud del proveedor.

**Control:**

- ✅ El consumidor **SÍ controla** las aplicaciones desplegadas
- ✅ Posiblemente configuración del entorno de despliegue
- ❌ El consumidor **NO controla** la infraestructura cloud subyacente (red, servidores, SO, almacenamiento)

**Características Clave:**

**Lo que ofrece PaaS:**

- Herramientas completas de desarrollo y despliegue
- Incluye todo lo que los desarrolladores necesitan:
  - Servidores y sistemas operativos
  - Redes y almacenamiento
  - Middleware (frameworks, libraries)
  - Entornos de ejecución (runtime)
  - Herramientas de desarrollo
  - Servicios adicionales (BD, colas, cache, etc.)

**Los desarrolladores:**

- Crean y despliegan aplicaciones sobre la plataforma
- Se centran en el código de negocio
- **No tienen que** gestionar la plataforma de desarrollo ni mantenerla

**Soporte de Plataformas Completas:**

**Las plataformas PaaS más completas soportan:**

- ✅ **Diferentes lenguajes:** Python, Java, Node.js, .NET, PHP, Ruby, Go
- ✅ **Entornos de desarrollo:** IDEs integrados, CI/CD pipelines
- ✅ **Servidores de aplicaciones:** Tomcat, Jetty, IIS
- ✅ **SGBD:** PostgreSQL, MySQL, SQL Server, Oracle
- ✅ **Almacenamiento NoSQL:** MongoDB, Redis, Cassandra
- ✅ **Proveedores IaaS:** Pueden ejecutarse sobre diferentes clouds

**Gestión automatizada del ciclo de vida:**

- Despliegue automático (CI/CD)
- Escalado automático
- Monitorización integrada
- Logs centralizados
- Backup automático
- Rollback en caso de errores

**Servicios de aplicación:**

- Integración entre servicios
- Orquestación de microservicios
- Configuración centralizada
- Service discovery
- Load balancing automático

**Ventajas y Desventajas de PaaS:**

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| **No gestionas SO ni runtime:** El proveedor se encarga | **Menos control:** No accedes al SO subyacente |
| **Despliegue rápidísimo:** De código a producción en minutos | **Vendor lock-in:** APIs propietarias, difícil migración |
| **Auto-scaling integrado:** Escalado automático según carga | **Limitaciones:** Solo lenguajes/frameworks soportados |
| **Servicios adicionales listos:** BD, colas, storage, APIs | **Dependencia:** Cambios del proveedor afectan tu app |
| **Actualizaciones automáticas:** De platform, no afectan tu app | **Coste:** Puede ser más caro que IaaS (pagas conveniencia) |
| **Ideal para desarrolladores:** Foco 100% en código de negocio | **Debugging complejo:** Sin acceso bajo nivel |
| **Multi-lenguaje:** Soporte de múltiples stacks tecnológicos | |

**Proveedores Principales de PaaS:**

| Proveedor | Servicio | Lenguajes Soportados | Características |
|-----------|----------|---------------------|-----------------|
| **Heroku** | Heroku Platform | Ruby, Node.js, Java, Python, PHP, Go, Scala, Clojure | Pionero PaaS, muy fácil de usar, add-ons ricos |
| **AWS** | Elastic Beanstalk | Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker | Integración total AWS, multi-container |
| **Microsoft Azure** | Azure App Service | .NET, Java, Node.js, PHP, Python | Integración Visual Studio, Windows/Linux |
| **Google Cloud** | Google App Engine | Python, Java, Node.js, PHP, Ruby, Go | Escalado automático agresivo, Standard/Flexible |
| **Red Hat** | OpenShift | Cualquiera (containers) | Basado en Kubernetes, híbrido/multi-cloud |
| **Salesforce** | Heroku, Force.com | Apex, Ruby, Java, Python, Node | Ecosistema Salesforce, enfoque CRM |

**Fuente de comparación:** https://www.g2crowd.com/categories/cloud-platform-as-a-service-paas

**Caso de Uso Detallado: PaaS para API de ML:**

```python
# Ejemplo: Desplegar API de predicción en Azure App Service (PaaS)

# -----------------------------
# 1. CÓDIGO DE LA APLICACIÓN
# -----------------------------

# app.py - Flask API para predicción de riesgo cardiovascular
from flask import Flask, request, jsonify
import pandas as pd
import joblib
import logging

app = Flask(__name__)

# Configurar logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Cargar modelo ML entrenado previamente
model = joblib.load('modelo_riesgo_cardiovascular.pkl')

@app.route('/health', methods=['GET'])
def health_check():
    """Endpoint de salud para monitoreo"""
    return jsonify({'status': 'healthy', 'service': 'CardioRisk API'}), 200

@app.route('/predict', methods=['POST'])
def predict():
    """
    Predecir riesgo cardiovascular
    
    Input JSON:
    {
        "edad": 55,
        "sexo": "M",
        "colesterol": 240,
        "tension_sistolica": 140,
        "fumador": true,
        "diabetes": false
    }
    """
    try:
        # Obtener datos del request
        data = request.json
        logger.info(f"Received prediction request: {data}")
        
        # Validar datos
        required_fields = ['edad', 'sexo', 'colesterol', 'tension_sistolica', 'fumador', 'diabetes']
        if not all(field in data for field in required_fields):
            return jsonify({'error': 'Missing required fields'}), 400
        
        # Preparar datos para el modelo
        df = pd.DataFrame([{
            'edad': data['edad'],
            'sexo': 1 if data['sexo'] == 'M' else 0,
            'colesterol': data['colesterol'],
            'tension_sistolica': data['tension_sistolica'],
            'fumador': 1 if data['fumador'] else 0,
            'diabetes': 1 if data['diabetes'] else 0
        }])
        
        # Predecir
        prediction = model.predict(df)[0]
        probability = model.predict_proba(df)[0][1]
        
        # Clasificar riesgo
        if probability < 0.2:
            risk_level = 'Bajo'
        elif probability < 0.5:
            risk_level = 'Moderado'
        else:
            risk_level = 'Alto'
        
        result = {
            'prediccion': int(prediction),
            'probabilidad': round(float(probability), 3),
            'nivel_riesgo': risk_level,
            'recomendacion': get_recommendation(risk_level)
        }
        
        logger.info(f"Prediction result: {result}")
        return jsonify(result), 200
        
    except Exception as e:
        logger.error(f"Error in prediction: {str(e)}")
        return jsonify({'error': 'Internal server error', 'message': str(e)}), 500

def get_recommendation(risk_level):
    """Generar recomendación según nivel de riesgo"""
    recommendations = {
        'Bajo': 'Mantener hábitos saludables. Revisión anual.',
        'Moderado': 'Control médico cada 6 meses. Considerardieta y ejercicio.',
        'Alto': 'Consulta médica urgente. Posible tratamiento farmacológico.'
    }
    return recommendations.get(risk_level, 'Consultar médico')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)

# -----------------------------
# 2. ARCHIVOS DE CONFIGURACIÓN
# -----------------------------

# requirements.txt
"""
flask==2.3.0
pandas==2.0.0
scikit-learn==1.2.2
joblib==1.2.0
gunicorn==20.1.0
"""

# Procfile (para Heroku)
"""
web: gunicorn app:app
"""

# -----------------------------
# 3. DESPLIEGUE EN AZURE APP SERVICE
# -----------------------------

# Usando Azure CLI
"""
# Login
az login

# Crear grupo de recursos
az group create --name rg-cardiorisk-api --location westeurope

# Crear App Service Plan (Linux)
az appservice plan create \\
    --name plan-cardiorisk \\
    --resource-group rg-cardiorisk-api \\
    --sku B1 \\
    --is-linux

# Crear Web App
az webapp create \\
    --resource-group rg-cardiorisk-api \\
    --plan plan-cardiorisk \\
    --name cardiorisk-api-prod \\
    --runtime "PYTHON:3.9"

# Configurar auto-scaling
az monitor autoscale create \\
    --resource-group rg-cardiorisk-api \\
    --resource cardiorisk-api-prod \\
    --resource-type Microsoft.Web/serverfarms \\
    --name autoscale-cardiorisk \\
    --min-count 1 \\
    --max-count 10 \\
    --count 2

# Regla: Scale out cuando CPU > 70%
az monitor autoscale rule create \\
    --resource-group rg-cardiorisk-api \\
    --autoscale-name autoscale-cardiorisk \\
    --condition "Percentage CPU > 70 avg 5m" \\
    --scale out 2

# Desplegar código
az webapp up \\
    --name cardiorisk-api-prod \\
    --resource-group rg-cardiorisk-api \\
    --runtime "PYTHON:3.9"

# Configurar variables de entorno
az webapp config appsettings set \\
    --resource-group rg-cardiorisk-api \\
    --name cardiorisk-api-prod \\
    --settings \\
        MODEL_PATH="/home/site/wwwroot/modelo_riesgo_cardiovascular.pkl" \\
        LOG_LEVEL="INFO" \\
        FLASK_ENV="production"

# Habilitar logs
az webapp log config \\
    --name cardiorisk-api-prod \\
    --resource-group rg-cardiorisk-api \\
    --application-logging filesystem \\
    --level information

# Ver logs en tiempo real
az webapp log tail \\
    --name cardiorisk-api-prod \\
    --resource-group rg-cardiorisk-api
"""

# -----------------------------
# 4. TESTING DE LA API
# -----------------------------

import requests
import json

# URL de la API desplegada
API_URL = "https://cardiorisk-api-prod.azurewebsites.net"

# Test 1: Health check
response = requests.get(f"{API_URL}/health")
print("Health check:", response.json())
# Output: {'status': 'healthy', 'service': 'CardioRisk API'}

# Test 2: Predicción - Riesgo bajo
paciente_riesgo_bajo = {
    "edad": 30,
    "sexo": "F",
    "colesterol": 180,
    "tension_sistolica": 110,
    "fumador": False,
    "diabetes": False
}

response = requests.post(
    f"{API_URL}/predict",
    json=paciente_riesgo_bajo,
    headers={'Content-Type': 'application/json'}
)

print("\nPaciente riesgo bajo:")
print(json.dumps(response.json(), indent=2))
# Output:
# {
#   "prediccion": 0,
#   "probabilidad": 0.087,
#   "nivel_riesgo": "Bajo",
#   "recomendacion": "Mantener hábitos saludables. Revisión anual."
# }

# Test 3: Predicción - Riesgo alto
paciente_riesgo_alto = {
    "edad": 65,
    "sexo": "M",
    "colesterol": 280,
    "tension_sistolica": 165,
    "fumador": True,
    "diabetes": True
}

response = requests.post(
    f"{API_URL}/predict",
    json=paciente_riesgo_alto,
    headers={'Content-Type': 'application/json'}
)

print("\nPaciente riesgo alto:")
print(json.dumps(response.json(), indent=2))
# Output:
# {
#   "prediccion": 1,
#   "probabilidad": 0.782,
#   "nivel_riesgo": "Alto",
#   "recomendacion": "Consulta médica urgente..."
# }

# -----------------------------
# 5. MONITORIZACIÓN Y COSTES
# -----------------------------

# Azure App Service B1 (Basic): ~$13/mes
# Auto-scaling 1-10 instancias:
#   - Normal: 2 instancias = $26/mes
#   - Pico: 10 instancias = $130/mes (solo durante pico)
#   - Promedio: ~$40/mes

# Ventajas PaaS vs IaaS en este caso:
# ✅ Despliegue en 5 minutos (vs 2 horas configurando VM)
# ✅ Auto-scaling automático (vs scripting manual)
# ✅ SSL/HTTPS gratis incluido
# ✅ Logs centralizados
# ✅ Backup automático
# ✅ Rollback fácil a versiones previas
# ✅ CI/CD integrado con GitHub
# ✅ Monitoreo con Application Insights incluido
```

**Ventaja clave de PaaS en este ejemplo:**

- IaaS: 2 horas setup + mantenimiento continuo + scripts auto-scaling
- PaaS: 5 minutos setup + cero mantenimiento + auto-scaling nativo

**3.2.3. SaaS (Software as a Service):**

**Definición:**

**Software como Servicio:** El consumidor utiliza las aplicaciones del proveedor que se ejecutan en la infraestructura cloud. Las aplicaciones son accesibles desde varios dispositivos cliente a través de interfaces thin client (ej.: navegador web) o interfaces de programa.

**Control:**

- ✅ El consumidor **SÍ controla** configuraciones personales de la aplicación (settings de usuario)
- ❌ El consumidor **NO controla** la infraestructura cloud subyacente
- ❌ El consumidor **NO controla** capacidades de la aplicación (excepto config limitada)

**Características Clave:**

**Lo que ofrece SaaS:**

- Los clientes acceden al software que contratan por Internet
- Interfaz web o aplicación móvil
- **Se paga por** versión y por número de usuarios (no por "licencia" perpetua tradicional)

- Todo es responsabilidad del proveedor:
  - Hardware y servidores
  - Sistema operativo
  - Middleware y runtime
  - Aplicación y datos
  - Actualizaciones y patches
  - Seguridad
  - Disponibilidad y backups

**El proveedor:**

- Se encarga de gestionar todas las actualizaciones
- Corrige errores (bugs)
- Realiza pasos de mantenimiento generales
- Garantiza disponibilidad (SLA)

**El usuario:**

- Solo tiene que conectarse a la aplicación (navegador web típicamente)
- Consumir el servicio
- Pagar suscripción mensual/anual

**Ventajas y Desventajas de SaaS:**

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| **Cero gestión técnica:** Nada que instalar, configurar o mantener | **Nula personalización técnica:** Solo configuración limitada UI |
| **Acceso inmediato:** Signup y empiezas a usar en minutos | **Dependencia total del proveedor:** Si cae el servicio, estás bloqueado |
| **Actualizaciones automáticas:** Siempre última versión sin esfuerzo | **Vendor lock-in extremo:** Datos en formato propietario, difícil migración |
| **Acceso multi-dispositivo:** Web, móvil, tablet | **Privacidad preocupante:** Tus datos en servidores del proveedor |
| **Colaboración fácil:** Múltiples usuarios simultáneos | **Conectividad requerida:** Sin Internet = sin servicio |
| **Costes predecibles:** Suscripción mensual fija por usuario | **Compliance:** Puede no cumplir regulaciones específicas |
| **Escalado fácil:** Añadir/quitar usuarios según necesidad | **Integraciones limitadas:** APIs pueden no cubrir tus necesidades |
| **No requiere IT:** Departamentos no técnicos pueden adoptarlo | |

**Ejemplos de SaaS Populares:**

**Productividad y Colaboración:**

| SaaS | Proveedor | Funcionalidad | Usuarios (aprox) |
|------|-----------|---------------|------------------|
| **Gmail / Google Workspace** | Google | Email, Drive, Docs, Sheets, Meet | 3+ billones |
| **Microsoft 365** | Microsoft | Email, OneDrive, Word, Excel, Teams | 400+ millones |
| **Slack** | Salesforce | Mensajería y colaboración | 18+ millones |
| **Zoom** | Zoom Video Communications | Videoconferencias | 300+ millones |
| **Dropbox** | Dropbox Inc. | Almacenamiento cloud | 700+ millones |

**CRM y Ventas:**

| SaaS | Funcionalidad | Precio típico |
|------|---------------|---------------|
| **Salesforce** | CRM completo, automatización ventas | $25-300/usuario/mes |
| **HubSpot** | Marketing, ventas, CRM | Freemium, $50-3,200/mes |
| **Zoho CRM** | CRM para PYMEs | $14-52/usuario/mes |

**Marketing y Automatización:**

| SaaS | Funcionalidad |
|------|---------------|
| **HubSpot Marketing** | Inbound marketing, automatización |
| **Mailchimp** | Email marketing |
| **Marketo** (Adobe) | Marketing automation enterprise |

**BI y Analytics:**

| SaaS | Funcionalidad | Casos de uso |
|------|---------------|--------------|
| **Tableau Online** | Visualización datos, dashboards | Business Intelligence |
| **Power BI** (Microsoft) | Análisis y visualización datos | Reportes ejecutivos |
| **Looker** (Google) | Plataforma BI moderna | Data exploration |
| **Qlik Sense** | BI self-service | Análisis ad-hoc |

**Data Warehousing:**

| SaaS | Características | Precio |
|------|-----------------|--------|
| **Snowflake** | DW cloud-native, separación compute/storage | Pay-per-use, $2-4/crédito |
| **BigQuery** (Google) | DW serverless, petabyte-scale | $5/TB query, $20/TB storage/mes |
| **Amazon Redshift** | DW columnar, integración AWS | From $0.25/hora/node |

**Herramientas de Desarrollo:**

| SaaS | Funcionalidad |
|------|---------------|
| **GitHub** | Control de versiones, CI/CD |
| **GitLab** | DevOps platform completo |
| **Jira** (Atlassian) | Gestión proyectos ágiles |
| **Trello** (Atlassian) | Gestión tareas Kanban |

**Caso de Uso: Análisis de Datos con Tableau Online (SaaS):**

**Escenario:** Hospital necesita dashboards de KPIs sin infraestructura propia.

**Paso 1: Signup**
```
1. Ir a https://www.tableau.com/
2. Sign up Tableau Online
3. Elegir plan (Creator: $70/usuario/mes)
4. Listo para usar en 5 minutos
```

**Paso 2: Conectar a fuentes de datos**
```python
# Aunque Tableau es SaaS (web), puedes preparar datos con Python

import pandas as pd
import tableauhyperapi as hapi

# Cargar datos de hospitalizaciones
df = pd.read_csv('hospitalizaciones_2024.csv')

# Limpiar y preparar
df['fecha_ingreso'] = pd.to_datetime(df['fecha_ingreso'])
df['edad'] = df['edad'].fillna(df['edad'].median())

# Publicar a Tableau Online vía Hyper API
with hapi.HyperProcess(telemetry=hapi.Telemetry.SEND_USAGE_DATA_TO_TABLEAU) as hyper:
    with hapi.Connection(
        endpoint=hyper.endpoint,
        database='hospitalizaciones.hyper',
        create_mode=hapi.CreateMode.CREATE_AND_REPLACE
    ) as connection:
        
        # Definir schema
        schema = hapi.TableDefinition(
            table_name=hapi.TableName('Extract', 'Hospitalizaciones'),
            columns=[
                hapi.TableDefinition.Column('fecha_ingreso', hapi.SqlType.date()),
                hapi.TableDefinition.Column('edad', hapi.SqlType.int()),
                hapi.TableDefinition.Column('diagnostico', hapi.SqlType.text()),
                hapi.TableDefinition.Column('dias_estancia', hapi.SqlType.int()),
                hapi.TableDefinition.Column('coste_total', hapi.SqlType.double())
            ]
        )
        
        connection.catalog.create_table(schema)
        
        # Insertar datos
        with hapi.Inserter(connection, schema) as inserter:
            for _, row in df.iterrows():
                inserter.add_row([
                    row['fecha_ingreso'],
                    int(row['edad']),
                    row['diagnostico'],
                    int(row['dias_estancia']),
                    float(row['coste_total'])
                ])
            inserter.execute()

# Subir a Tableau Online
import tableauserverclient as TSC

tableau_auth = TSC.TableauAuth('usuario@hospital.com', 'password', site_id='mi-hospital')
server = TSC.Server('https://10ay.online.tableau.com')

with server.auth.sign_in(tableau_auth):
    # Publicar datasource
    datasource = TSC.DatasourceItem('mi-hospital', 'Hospitalizaciones2024')
    datasource = server.datasources.publish(
        datasource,
        'hospitalizaciones.hyper',
        mode=TSC.Server.PublishMode.Overwrite
    )
    print(f"Datasource publicado: {datasource.id}")
```

**Paso 3: Crear dashboards (UI web)**

- Arrastrar y soltar dimensiones/métricas
- Crear visualizaciones (gráficos, mapas, tablas)
- Configurar filtros interactivos
- Diseñar dashboard completo
- Publicar a usuarios

**Paso 4: Compartir**
```
1. Click "Share"
2. Añadir usuarios (emails)
3. Configurar permisos (view/edit)
4. Enviar link

Usuarios acceden:
- Via web: https://10ay.online.tableau.com
- Via móvil: App Tableau Mobile
- Sin instalar nada
```

**Ventajas en este caso:**

- ✅ Cero infraestructura (ni servidor, ni BD, ni software instalado)
- ✅ Actualizaciones automáticas de Tableau
- ✅ Alta disponibilidad gestionada por Tableau
- ✅ Backups automát icos
- ✅ Colaboración inmediata
- ✅ Acceso móvil nativo

**Coste:**
```
- 5 usuarios Creator (crear dashboards): 5 × $70/mes = $350/mes
- 20 usuarios Viewer (solo ver): 20 × $15/mes = $300/mes
- TOTAL: $650/mes = $7,800/año

Alternativa on-premise:
- Servidores: $10,000 (one-time)
- Licencias Tableau Server: $35,000/año
- Admin IT: $50,000/año
- TOTAL primer año: $95,000

Ahorro SaaS: 91% primer año, 85% años subsecuentes
```

**3.2.4. Otros Modelos XaaS:**

**DaaS (Data as a Service):**

**Definición:** Datos como servicio, acceso a datasets via APIs o interfaces web.

**Ejemplos:**

- **AWS Data Exchange:** Marketplace de datasets de terceros
- **Snowflake Data Marketplace:** Compartición de datos entre organizaciones
- **factual.com:** Datos de localización y POIs
- **Xignite:** Datos financieros en tiempo real

**Caso de uso:**
```python
# Consumir datos financieros via API (DaaS)
import requests

API_KEY = "tu-api-key"
symbol = "AAPL"  # Apple

# Datos de Xignite (DaaS)
url = f"https://globalquote.xignite.com/v3/xGlobalQuote.json/GetGlobalDelayedQuote"
params = {
    '_token': API_KEY,
    'IdentifierType': 'Symbol',
    'Identifier': symbol
}

response = requests.get(url, params=params)
data = response.json()

print(f"Precio actual {symbol}: ${data['Last']}")
print(f"Volumen: {data['Volume']}")

# Pagas por:
# - Número de requests
# - Típicamente: $50-500/mes según volumen
#
# Alternativa:
# - Crear scraper propio: Ilegal, bloqueado
# - Comprar feed directo bolsa: $10,000+/mes
# - DaaS: $50-500/mes, legal, confiable
```

**FaaS (Function as a Service) / Serverless:**

**Definición:** Ejecutar código sin aprovisionar servidores. El proveedor gestiona la infraestructura completamente.

**Características principales:**

- **Sin servidor:** No contamos con infraestructura donde alojar aplicaciones
- **Pequeñas funciones:** Los programadores escriben código que se ejecuta en "entorn os de ejecución" proporcionados por proveedores
- **Efímero:** Cada vez que se ejecuta el código, el entorno se crea, ejecuta el código y luego se destruye

**Ventajas y Desventajas:**

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| **Mantenimiento zero** de infraestructura física o lógica | **Vendor lock-in:** Código puede quedar acoplado al proveedor cloud (programadores deben tener cuidado) |
| **Escalable horizontalmente:** Automáticamente a millones de invocaciones | **Monitorización difícil:** Debugging complicado en entornos distribuidos efímeros |
| **Pago por ejecución:** No pagas cuando no se ejecuta (vs VMs que cobran 24/7) | **Testing limitado:** Las pruebas no pueden hacerse "en casa", siempre en proveedor |
| **Integración fácil:** Con otros servicios cloud del mismo proveedor | **Lenguajes limitados:** Cada proveedor soporta unos lenguajes específicos (no todos) |
| | **Cold start:** Primera invocación tarda más (segundos) |
| | **Timeouts:** Límites de tiempo de ejecución (ej.: AWS Lambda max 15 min) |

**Ejemplos de FaaS:**

| Proveedor | Servicio | Lenguajes | Precio |
|-----------|----------|-----------|--------|
| **AWS** | AWS Lambda | Python, Node.js, Java, Go, C#, Ruby | $0.20 por 1M requests + $0.0000166667 por GB-segundo |
| **Azure** | Azure Functions | C#, Java, JavaScript, Python, PowerShell | Similar a AWS Lambda |
| **Google** | Cloud Functions | Node.js, Python, Go, Java |  $0.40 por 1M invocaciones |
| **Cloudflare** | Cloudflare Workers | JavaScript, Rust, C, C++ (WASM) | $5/mes 10M requests |

**Ejemplo: Lambda para procesar uploads a S3**

```python
# lambda_function.py
# Se ejecuta automáticamente cuando se sube un archivo CSV a S3

import json
import boto3
import pandas as pd
from io import BytesIO

s3 = boto3.client('s3')
sns = boto3.client('sns')

def lambda_handler(event, context):
    """
    Trigger: S3 Put Object
    Acción: Validar CSV y notificar resultados
    """
    
    # Obtener info del archivo subido
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    print(f"Procesando: s3://{bucket}/{key}")
    
    try:
        # Descargar archivo
        obj = s3.get_object(Bucket=bucket, Key=key)
        df = pd.read_csv(BytesIO(obj['Body'].read()))
        
        # Validaciones
        errors = []
        warnings = []
        
        # Validación 1: Columnas requeridas
        required_cols = ['paciente_id', 'fecha', 'diagnostico']
        missing = [col for col in required_cols if col not in df.columns]
        if missing:
            errors.append(f"Faltan columnas: {missing}")
        
        # Validación 2: Nulos
        null_pct = (df.isnull().sum() / len(df) * 100).to_dict()
        high_nulls = {col: pct for col, pct in null_pct.items() if pct > 10}
        if high_nulls:
            warnings.append(f"Columnas con >10% nulos: {high_nulls}")
        
        # Validación 3: Duplicados
        duplicates = df.duplicated().sum()
        if duplicates > 0:
            warnings.append(f"Registros duplicados: {duplicates}")
        
        # Generar reporte
        report = {
            'archivo': key,
            'filas': len(df),
            'columnas': len(df.columns),
            'errores': errors,
            'warnings': warnings,
            'estado': 'ERROR' if errors else ('WARNING' if warnings else 'OK')
        }
        
        # Notificar via SNS
        message = json.dumps(report, indent=2)
        sns.publish(
            TopicArn='arn:aws:sns:eu-west-1:123456789012:data-validation',
            Subject=f"Validación: {key} - {report['estado']}",
            Message=message
        )
        
        # Si OK, mover a carpeta "validated"
        if report['estado'] == 'OK':
            new_key = key.replace('uploads/', 'validated/')
            s3.copy_object(
                Bucket=bucket,
                CopySource={'Bucket': bucket, 'Key': key},
                Key=new_key
            )
            s3.delete_object(Bucket=bucket, Key=key)
            print(f"Movido a: {new_key}")
        
        return {
            'statusCode': 200,
            'body': json.dumps(report)
        }
        
    except Exception as e:
        error_msg = f"Error procesando {key}: {str(e)}"
        print(error_msg)
        
        sns.publish(
            TopicArn='arn:aws:sns:eu-west-1:123456789012:data-validation',
            Subject=f"ERROR: {key}",
            Message=error_msg
        )
        
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }

# Configuración Lambda:
# - Runtime: Python 3.11
# - Memory: 512 MB
# - Timeout: 30 segundos
# - Trigger: S3 bucket "mi-datalake", prefix "uploads/"
# 
# Coest o estimado:
# - 10,000 archivos/mes
# - Promedio 2 segundos/archivo, 512 MB
# - 10,000 requests × $0.20/1M = $0.002
# - 10,000 × 2s × 0.5GB × $0.0000166667 = $0.17
# - TOTAL: ~$0.17/mes
#
# Alternativa EC2 t3.medium 24/7: ~$30/mes
# Ahorro: 99.4%
```

**Cuándo usar Serverless/FaaS:**

- ✅ Tareas event-driven (responder a eventos)
- ✅ Procesamiento intermitente (no 24/7)
- ✅ Microservicios ligeros
- ✅ APIs con tráfico variable
- ✅ ETL y procesamiento de datos
- ✅ Scheduled jobs (cron)
- ✅ IoT backends

**Cuándo NO usar Serverless:**

- ❌ Procesamiento de larga duración (>15 min en AWS Lambda)
- ❌ Aplicaciones stateful
- ❌ Alto rendimiento computing (HPC)
- ❌ Requisitos de latencia ultra-baja (<10ms)

**Resumen Comparativo de Modelos de Servicio:**

| Dimensión | IaaS | PaaS | SaaS | FaaS |
|-----------|------|------|------|------|
| **Control** | Alto | Medio | Bajo | Muy Bajo |
| **Flexibilidad** | Máxima | Alta | Baja | Media |
| **Gestión Requerida** | Alta | Media | Nula | Baja |
| **Velocidad Despliegue** | Media | Alta | Inmediata | Alta |
| **Vendor Lock-in** | Bajo | Medio-Alto | Muy Alto | Alto |
| **Coste** | Medio | Medio-Alto | Depende | Muy Bajo (pay-per-use) |
| **Casos de Uso** | Infraestructura custom | Desarrollo apps | Software estándar | Event-driven, microservicios |
| **Ejemplo** | AWS EC2 | Heroku | Salesforce | AWS Lambda |
| **Usuarios Típicos** | DevOps, SysAdmins | Desarrolladores | Usuarios finales | Desarrolladores |

```
instances = ec2.create_instances(
    ImageId='ami-0c55b159cbfafe1f0',  # Ubuntu 20.04
    MinCount=1,
    MaxCount=1,
    InstanceType='t3.2xlarge',  # 8 vCPUs, 32 GB RAM
    KeyName='my-key-pair',
    UserData='''#!/bin/bash
                apt-get update
                apt-get install -y python3-pip
                pip3 install pandas numpy scikit-learn
                '''
)
```

**PaaS (Platform as a Service):**

> **Definición**: Plataforma completa para desarrollar, ejecutar y gestionar aplicaciones.

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| No gestionas SO ni runtime | Menos control |
| Despliegue rápido | Posible vendor lock-in |
| Auto-scaling integrado | |
| Servicios adicionales (BD, colas, etc.) | |

**Ejemplos:**

- **AWS Elastic Beanstalk**
- **Azure App Service**
- **Google App Engine**
- **Heroku**

**Caso de uso:**
```python
# Ejemplo: Desplegar API de análisis en Azure App Service

# requirements.txt
# flask
# pandas
# scikit-learn

# app.py
from flask import Flask, request, jsonify
import pandas as pd
import joblib

app = Flask(__name__)

# Cargar modelo entrenado
model = joblib.load('model.pkl')

@app.route('/predict', methods=['POST'])
def predict():
    data = request.json
    df = pd.DataFrame([data])
    prediction = model.predict(df)
    return jsonify({'prediction': int(prediction[0])})

if __name__ == '__main__':
    app.run()

# Desplegar con:
# az webapp up --name mi-api-prediccion --runtime "PYTHON:3.9"
```

**SaaS (Software as a Service):**

**Definición**: Aplicación completa accesible vía web.

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| Cero gestión técnica | Nula personalización técnica |
| Acceso inmediato | Dependencia del proveedor |
| Actualizaciones automáticas | |

**Ejemplos:**

- **Gmail**: Email
- **Salesforce**: CRM
- **Tableau Online**: BI
- **Snowflake**: Data Warehouse

**Caso de uso:** Análisis de datos con Tableau Online

- Conectar a fuentes de datos
- Crear dashboards
- Compartir con equipo
- Sin infraestructura propia

## 3.3. Modelos de despliegue

**Nube Pública:**

**Características:**

- Recursos compartidos con otros usuarios
- Gestionado por proveedor (AWS, Azure, GCP)
- Pago por uso
- Internet-accessible

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| Sin inversión inicial (CAPEX = 0) | Menor control |
| Escalabilidad ilimitada | Preocupaciones de seguridad/privacidad |
| Mantenimiento del proveedor | Dependencia de internet |

**Cuándo usar:**

- Startups y PYMEs
- Desarrollo y pruebas
- Cargas de trabajo variables
- Proyectos con presupuesto limitado

**Nube Privada:**

**Características:**

- Infraestructura dedicada a una organización
- On-premise o en datacenter dedicado
- Mayor control y seguridad

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| Control total | Alto CAPEX (inversión inicial) |
| Cumplimiento normativo más fácil | Requiere equipo de gestión |
| Seguridad personalizada | Escalabilidad limitada |

**Cuándo usar:**

- Datos muy sensibles (sanidad, defensa, banca)
- Requisitos regulatorios estrictos
- Grandes empresas

**Nube Híbrida:**

**Características:**

- Combinación de pública y privada
- Workloads distribuidos según necesidades
- Interconexión segura

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| Flexibilidad máxima | Complejidad de gestión |
| Datos sensibles en privada, el resto en pública | Requiere integración y orquestación |
| Cloud bursting (escalar a pública cuando necesario) | |

**Caso de uso en sanidad:**
```
Nube Privada (On-premise):
- Datos clínicos identificables
- Historia clínica electrónica
- PACS (imágenes médicas)

Nube Pública (AWS/Azure):
- Análisis de datos anonimizados
- Entrenamiento de modelos ML
- Data Lake de datos epidemiológicos agregados
- Dashboards públicos
```

**Multi-cloud:**

**Características:**

- Uso de múltiples proveedores cloud (AWS + Azure + GCP)
- Evita vendor lock-in
- Best-of-breed approach

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| Sin dependencia de un proveedor | Complejidad de gestión |
| Aprovechar fortalezas de cada uno | Múltiples herramientas y APIs |
| Negociación de precios | Integración compleja |

---

## 3.4. Beneficios y desventajas de Cloud Computing

**Beneficios principales:**

**1. Integración probada de servicios de red:**

**Ventaja:** La tecnología cloud computing se integra con mayor facilidad y rapidez con el resto de aplicaciones empresariales.

```python
# Ejemplo: Integración fácil de servicios AWS
import boto3

# Integrar múltiples servicios en mismo ecosistema
s3 = boto3.client('s3')           # Almacenamiento
lambda_client = boto3.client('lambda')  # Computación serverless
sns = boto3.client('sns')         # Notificaciones
sqs = boto3.client('sqs')         # Colas de mensajes

# Todo integrado nativamente, APIs consistentes
# Sin necesidad de middleware complejo
```

**2. Prestación de Servicios a Nivel Mundial:**

**Ventaja:** Infraestructuras que proporcionan:

- ✅ Mayor capacidad de adaptación
- ✅ Recuperación completa de pérdida de datos (backups automáticos)
- ✅ Reducción al mínimo de tiempos de inactividad
- ✅ Alta disponibilidad global

**Mecanismo:**
Ante eventualidades de fallas, el cloud re-asigna automáticamente nuevas VMs para las aplicaciones.

```
Ejemplo: Aplicación web en 3 regiones

       Usuario España          Usuario USA          Usuario Asia
            ↓                      ↓                    ↓
    EU-West-1 (Irlanda)    US-East-1 (Virginia)   AP-Southeast-1 (Singapur)
         Server 1               Server 2              Server 3
         
Si Server 1 falla:
→ Route 53 (DNS) redirige automáticamente a Server 2
→ Auto-scaling lanza nuevo Server 1'
→ Downtime: < 30 segundos
```

**3. Implementación Más Rápida y con Menos Riesgos:**

**Ventajas:**

- ✅ Se comienza a trabajar más rápido
- ✅ No es necesaria una gran inversión inicial (CAPEX)
- ✅ Reducción de riesgo de inversión en hardware

**Comparativa:**
```
Datacenter Tradicional:
1. Aprobar presupuesto: 2 meses
2. Comprar hardware: 1 mes
3. Instalar y configurar: 1 mes
4. Testing: 2 semanas
TOTAL: ~4.5 meses

Cloud:
1. Crear cuenta: 10 minutos
2. Provisionar recursos: 5 minutos  
3. Desplegar aplicación: 30 minutos
TOTAL: ~45 minutos

Velocidad: 99.5% más rápido
```

**4. Escalamiento Dinámico (Elastic Scaling):**

**Ventaja:** Permite scaling out y scaling in de acuerdo a los requerimientos cambiantes de las aplicaciones.

**Tipos:**

- **Scaling Out (Horizontal):** Añadir más instancias
- **Scaling Back (Horizontal):** Reducir instancias
- **Scaling Up (Vertical):** Aumentar capacidad instancia
- **Scaling Down (Vertical):** Reducir capacidad instancia

```python
# Auto-scaling elástico en AWS
import boto3

autoscaling = boto3.client('autoscaling')

# Configurar escalado automático
autoscaling.put_scaling_policy(
    AutoScalingGroupName='mi-aplicacion',
    PolicyName='scale-dynamically',
    PolicyType='TargetTrackingScaling',
    TargetTrackingConfiguration={
        'PredefinedMetricSpecification': {
            'PredefinedMetricType': 'ASGAverageCPUUtilization'
        },
        'TargetValue': 70.0  # Mantener CPU en 70%
    }
)

# Resultado:
# - Tráfico bajo: 2 instancias (ahorro)
# - Tráfico medio: 5 instancias (óptimo)
# - Pico tráfico: 20 instancias (soporta carga)
# - Automático, sin intervención manual
```

**5. Actualizaciones Automáticas:**

**Ventajas:**

- ✅ No afectan negativamente a los recursos de TIC
- ✅ Ahorro de tiempo y recursos para personalizar e integrar actualizaciones
- ✅ Siempre última versión de servicios
- ✅ Patches de seguridad aplicados automáticamente

**Ejemplo SaaS:**

- Salesforce: Actualización 3 veces/año, transparente
- Office 365: Actualizaciones continuas
- Sin downtime, sin esfuerzo del usuario

**6. Alta Disponibilidad y Durabilidad:**

**Ventaja:** El cloud ofrece:

- ✅ Plataforma para aplicaciones con acceso y almacenamiento confiable
- ✅ Alta disponibilidad de los recursos computacionales
- ✅ SLAs típicos: 99.9% - 99.99% uptime

**Ejemplo Amazon S3:**

- Durabilidad: 99.999999999% (11 noves)
- Disponibilidad: 99.99%
- Replicación automática en 3 zonas

```python
#Alta disponibilidad configurada automáticamente
import boto3

s3 = boto3.client('s3')
s3.create_bucket(
    Bucket='mi-aplicacion-datos',
    CreateBucketConfiguration={
        'LocationConstraint': 'eu-west-1'
    }
)

# AWS automáticamente:
# - Replica datos en 3 datacenters separados
# - Monitorea integridad continuamente
# - Repara automáticamente si detecta corrupción
# - Garantiza disponibilidad 99.99%
```

**7. Contribuye al Uso Eficiente de la Energía:**

**Ventaja:** Las organizaciones no requieren mantener datacenters que consumen gran energía.

**Eficiencia energética:**
```
Datacenter tradicional empresa:
- PUE (Power Usage Effectiveness): 2.0
- Por cada 1W en servidores, 2W totales (1W cooling/infraestructura)
- Utilización típica: 20-30%

Datacenter AWS/Google/Azure:
- PUE: 1.2
- Utilización: 70-80% (multi-tenancy)
- Energías renovables: 50-100%

Ahorro energético: ~75% por workload
```

**Desventajas y Limitaciones:**

**1. Centralización y Dependencia del Proveedor:**

**Problema:** La centralización de aplicaciones y almacenamiento origina interdependencia de los proveedores de servicios.

❌ **Vendor Lock-in:** Difícil migrar a otro proveedor

❌ **Dependencia total:** Si el proveedor quiebra, problema grave

❌ **Cambios de precios:** Provider puede subir precios

**Mitigación:**
```python
# Usar abstracciones para reducir lock-in
# En lugar de APIs propietarias, usar estándares

# ❌ MAL: Específico de AWS
import boto3
s3 = boto3.client('s3')
s3.put_object(Bucket='mybucket', Key='file.txt', Body=data)

# ✅ MEJOR: Abstracción con Apache Libcloud
from libcloud.storage.types import Provider
from libcloud.storage.providers import get_driver

cls = get_driver(Provider.S3)  # Cambiar fácilmente a GCS u otro
driver = cls('api_key', 'api_secret')
container = driver.get_container(container_name='mybucket')
container.upload_object('file.txt', object_name='file.txt')

# Ahora puedes cambiar Provider.S3 por Provider.GOOGLE_STORAGE
# Sin cambiar resto del código
```

**2. Disponibilidad Sujeta a Internet:**

**Problema:** La disponibilidad de los servicios está sujeta a la disponibilidad de acceso a Internet.

❌ **Sin Internet = Sin servicio**

❌ **Latencia de red** puede afectar performance

❌ **Costes de ancho de banda** al transferir datos

**Casos problemáticos:**

- Zonas rurales con conectividad pobre
- Aplicaciones de ultra-baja latencia (<1ms)
- Transferencias masivas de datos (100s de TB)

**Solución: Arquitectura híbrida/edge computing**

**3. Vulnerabilidad de Datos Sensibles:**

**Problema:** Los datos "sensibles" del negocio no residen en las instalaciones de las empresas, lo que podría generar un contexto de vulnerabilidad.

❌ **Privacidad:** Terceros tienen acceso físico a servidores

❌ **Compliance:** Puede no cumplir regulaciones (HIPAA, GDPR)

❌ **Soberanía de datos:** Datos pueden estar en otro país

**Mitigación:**
```python
# Encriptación end-to-end
import boto3
from cryptography.fernet import Fernet

# 1. Encriptar ANTES de subir a cloud
key = Fernet.generate_key()
cipher = Fernet(key)

data_sensible = "Datos médicos del paciente..."
data_encriptada = cipher.encrypt(data_sensible.encode())

# 2. Subir datos encriptados
s3 = boto3.client('s3')
s3.put_object(
    Bucket='datos-sensibles',
    Key='paciente_12345_encrypted.bin',
    Body=data_encriptada,
    ServerSideEncryption='AES256'  # Doble encriptación
)

# 3. Solo tú tienes la clave de desencriptación
# Ni AWS puede leer los datos
```

**4. Confiabilidad Dependiente del Proveedor:**

**Problema:** La confiabilidad de los servicios depende de la "salud" tecnológica y financiera de los proveedores.

❌ **Outages:** Fallos del proveedor afectan a miles de clientes

❌ **SLA** no siempre cumplido al 100%

❌ **Compensación limitada:** Créditos, no compensación real de pérdidas

**Ejemplo de outages famosos:**

- AWS US-East-1 outage (2017): Afectó a miles de sitios
- Google Cloud outage (2019): YouTube, Snapchat caídos
- Azure outage (2020): Microsoft Teams inaccesible

**Mitigación: Multi-cloud o multi-región**

**5. Escalabilidad a Largo Plazo (Riesgo de Sobrecarga):**

**Problema:** A medida que más usuarios comparten la infraestructura de la nube, la sobrecarga en los servidores de los proveedores aumentará.

❌ Si el proveedor no dispone de un esquema de crecimiento óptimo puede llevar a:

- Degradaciones en el servicio
- Altos niveles de jitter (variabilidad de latencia)
- Rendimiento impredecible

**Realidad:** Los grandes proveedores (AWS, Azure, GCP) invierten billones en infraestructura, este problema es mínimo.

**6. Complejidad de los Servicios Cloud:**

**Problema:** Los servicios del cloud son más complejos que los servicios tradicionales.

❌ **Curva de aprendizaje:** Cientos de servicios, cada uno con configuraciones

❌ **Configuración incorrecta:** Puede llevar a brechas de seguridad

❌ **Costes inesperados:** Fácil gastar más de lo planeado

**Ejemplo de complejidad:**

AWS tiene 200+ servicios:
> EC2, S3, RDS, Lambda, DynamoDB, Kinesis, Redshift, EMR, SageMaker, ECS, EKS, CloudFormation, CloudWatch, IAM, VPC, Route 53, CloudFront, SNS, SQS, Step Functions...

Aprender todos: Años de experiencia

**7. Privacidad de la Información:**

**Problema:** La información queda expuesta a terceros.

❌ **Acceso del proveedor:** Empleados del proveedor pueden tener acceso

❌ **Subpoenas legales:** Gobiernos pueden solicitar datos

❌ **Ubicación física:** Datos pueden estar en jurisdicción que no controlas

**8. Seguridad de la Información:**

**Problema:** La información debe recorrer nodos para llegar a su destino, cada uno (y sus canales) son un foco de inseguridad.

❌ **Man-in-the-middle attacks**

❌ **Protocolos seguros (HTTPS)** disminuyen velocidad (overhead)

❌ **Responsabilidad compartida:** Proveedor y cliente comparten responsabilidad

**Modelo de responsabilidad compartida (AWS):**
```
┌──────────────────────────────────────┐
│  CLIENTE RESPONSABLE DE:             │
│  - Datos                             │
│  - Configuración SO                  │
│  - Aplicaciones                      │
│  - IAM (accesos)                     │
│  - Encriptación client-side          │
│  - Configuración Firewall            │
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│  AWS RESPONSABLE DE:                  │
│  - Hardware / Infraestructura física │
│  - Virtualización                    │
│  - Seguridad física datacenters      │
│  - Redes globales                    │
│  - Regiones / Zonas disponibilidad   │
└──────────────────────────────────────┘
```

❗ **Importante:** La mayoría de brechas en cloud son por **mala configuración del cliente**, no fallos del proveedor.

**9. Manejo de Estándares Aún Inmaduro:**

**Problema:** El manejo de estándares para el despliegue de nubes aún no está del todo establecido.

❌ **Falta de estandarización** entre proveedores

❌ **APIs propietarias** diferentes

❌ **Portabilidad limitada** entre clouds

**Estandarización en proceso:**

- **Cloud Standards**: https://cloud-standards.org/
- **TOSCA** (Topology and Orchestration Specification for Cloud Applications)
- **OCCI** (Open Cloud Computing Interface)
- **CIMI** (Cloud Infrastructure Management Interface)

Pero aún lejos de ser universal.

**10. Requiere Buenas Prácticas:**

**Problema:** Se requieren fuertes fundamentos de "buenas prácticas" en:

- Desarrollo de software
- Arquitectura de software
- Gestión de servicios

❌ Sin buenas prácticas:

- Arquitecturas monolíticas que no escalan
- Aplicaciones stateful que no toleran fallas
- Seguridad débil (credenciales hardcodeadas)
- Costes descontrolados

**Solución: Adoptar metodologías como:**

- 12-Factor App
- Microservicios
- DevOps / CI/CD
- Infrastructure as Code
- Observabilidad (logging, monitoring, tracing)

**11. Consumo Energético de Datacenters:**

**Paradoja:** Aunque el cloud puede ser más eficiente, los clouds están conformados por grandes datacenters que consumen gran energía en total.

**Realidad:**

- Datacenter de AWS/Google: 50-100 MW (potencia de ciudad pequeña)
- Refrigeración masiva necesaria
- Impacto ambiental significativo

**Contraargumento:**
Despite el consumo absoluto, el cloud es más eficiente que miles de pequeños datacenters empresariales.

Además, proveedores invierten en energías renovables:

- Google Cloud: 100% energía renovable (2017+)
- AWS: Objetivo 100% renovable para 2025
- Azure: 100% renovable para 2025

**Balance: ¿Cuándo Usar Cloud?:**

**SÍ usar cloud si:**

- ✅ Startup o PYME con poco CAPEX
- ✅ Carga de trabajo variable/impredecible
- ✅ Necesitas escalar rápido (time-to-market)
- ✅ Desarrollo y testing (entornos efímeros)
- ✅ Innovación y experimentación
- ✅ No tienes equipo IT especializado
- ✅ Cumplimiento legal permite datos en cloud

**NO usar cloud (o usar híbrido) si:**

- ❌ Datos extremadamente sensibles (defensa, inteligencia)
- ❌ Regulaciones prohíben datos fuera de instalaciones
- ❌ Carga de trabajo 100% constante y predecible a largo plazo
- ❌ Latencia ultra-baja crítica (<1ms)
- ❌ Transferencias masivas continuas de datos


## 3.5. Retos y desafíos en Cloud Computing

**Visión general de retos:**


La construcción y operación de clouds enfrenta múltiples desafíos técnicos y organizacionales:
```text
┌───────────────────────────────────────────────────────┐
│           RETOS PRINCIPALES EN CLOUD                  │
│                                                       │
│  ┌───────────────┐    ┌──────────────┐                │
│  │ Virtualización│    │ Servicios Web│                │
│  │ - Rendimiento │    │ - APIs       │                │
│  │ - Control     │    │ - Estándares │                │
│  └───────────────┘    └──────────────┘                │
│                                                       │
│  ┌──────────────┐    ┌──────────────┐                 │
│  │  Seguridad y │    │ Contabilidad │                 │
│  │  Privacidad  │    │ - Medición   │                 │
│  │              │    │ - Costes     │                 │
│  └──────────────┘    └──────────────┘                 │
│                                                       │
│            ┌──────────────────┐                       │
│            │ Calidad Servicio │                       │
│            │ (QoS)            │                       │
│            └──────────────────┘                       │
└───────────────────────────────────────────────────────┘
```

**Reto 1: Calidad de Servicio (QoS):**

**Problema:** La ausencia de QoS puede generar que las organizaciones declinen del uso del cloud.

**Aspectos críticos:**

1. **Ancho de Banda Suficiente**
   - El compartimiento de recursos y datos complejos requiere suficiente ancho de banda sin generar más costos
   - Transferencia de grandes volúmenes de datos (TBs) puede ser costosa y lenta

2. **Operaciones Libres de Fallas**
   - Debe ofrecer operaciones libres de fallas
   - Realidad: Un cloud puede experimentar fallas regulares
   - Necesidad de mecanismos de tolerancia a fallos

**Desafíos QoS:**

| Aspecto | Desafío | Solución |
|---------|---------|----------|
| **Latencia** | Variable según carga | Multi-región, edge computing |
| **Throughput** | Compartido entre tenants | QoS policies, reserva de recursos |
| **Disponibilidad** | SLA 99.9% = 8.76h downtime/año | Multi-AZ, auto-failover |
| **Jitter** | Variabilidad impredecible | Networking dedicado, Direct Connect |

**Ejemplo: SLA de AWS EC2**
```text
AWS EC2 SLA:
- Single AZ: 99.5% uptime
- Multi-AZ (misma región): 99.99% uptime
- Multi-Región: 99.999% uptime (teórico)

99.99% uptime = 52.56 minutos downtime/año
99.5% uptime = 1.83 días downtime/año

Diferencia: Multi-AZ es 20x más confiable
```
---
**Reto 2: Seguridad y Privacidad:**

**Problema:** De primordial importancia, dado que los datos se comparten en el cloud y están susceptibles a ataques cibernéticos.

**Vectores de Amenaza:**

1. **Ataques externos:**
   - DDoS (Distributed Denial of Service)
   - Inyección SQL
   - Cross-Site Scripting (XSS)
   - Man-in-the-Middle

2. **Amenazas internas:**
   - Empleados maliciosos del proveedor
   - Configuración incorrecta (más común)
   - Credenciales comprometidas

3. **Multi-tenancy:**
   - Aislamiento entre clientes
   - Side-channel attacks
   - Resource exhaustion

**Mejores Prácticas de Seguridad:**

```python
# 1. Principio de mínimo privilegio (Least Privilege)
import boto3

iam = boto3.client('iam')

# ❌ MAL: Permisos demasiado amplios
policy_too_broad = {
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": "*",  # Permite TODO
        "Resource": "*"  # En TODOS los recursos
    }]
}

# ✅ BIEN: Permisos específicos
policy_least_privilege = {
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": [
            "s3:GetObject",
            "s3:PutObject"
        ],
        "Resource": "arn:aws:s3:::mi-bucket-especifico/*"
    }]
}

# 2. Encriptación en reposo y en tránsito
s3 = boto3.client('s3')

s3.put_object(
    Bucket='datos-sensibles',
    Key='archivo.txt',
    Body=data,
    ServerSideEncryption='AES256',  # Encriptación en reposo
    # En tránsito: Usar HTTPS (boto3 por defecto)
)

# 3. Multi-Factor Authentication (MFA)
# Para acciones críticas (ej: eliminar bucket)
bucket_policy = {
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:DeleteBucket",
        "Resource": "arn:aws:s3:::mi-bucket-critico",
        "Condition": {
            "BoolIfExists": {"aws:MultiFactorAuthPresent": "false"}
        }
    }]
}

# 4. Auditoría continua con CloudTrail
cloudtrail = boto3.client('cloudtrail')

cloudtrail.create_trail(
    Name='audit-trail-completo',
    S3BucketName='logs-auditoria',
    IncludeGlobalServiceEvents=True,
    IsMultiRegionTrail=True,
    EnableLogFileValidation=True  # Detecta manipulación de logs
)

# Registra TODA actividad:
# - Quién hizo qué
# - Cuándo
# - Desde dónde (IP)
# - Resultado (éxito/fallo)
```

**Framework de Seguridad:**

```
CIS (Center for Internet Security) Benchmarks para Cloud:
1. Identity and Access Management
2. Logging and Monitoring
3. Networking
4. Compute (VMs)
5. Storage
6. Databases

Cada benchmark con 100+ controles específicos
```

**Reto 3: Virtualización:**

**Problema:** Los accesos simultáneos de múltiples clientes a los servicios del cloud demandan más protección y control.

**Desafíos:**

1. **Rendimiento**
   - Overhead de virtualización (5-10%)
   - "Noisy neighbor" problem: Un tenant consume recursos excesivos, afecta a otros

2. **Control de Recursos**
   - Optimizar desempeño de aplicaciones
   - Mejorar utilización de recursos
   - Reducir consumo de energía

3. **Aislamiento**
   - Garantizar que tenants no puedan acceder a datos de otros
   - VM escape attacks

**Tecnologías de Virtualización:**

| Tecnología | Tipo | Overhead | Aislamiento | Uso |
|------------|------|----------|-------------|-----|
| **KVM** | Hypervisor Type-1 | Bajo (2-5%) | Alto | AWS, GCP |
| **Xen** | Hypervisor Type-1 | Bajo (3-7%) | Alto | AWS (antiguo) |
| **VMware ESXi** | Hypervisor Type-1 | Medio (5-10%) | Muy Alto | Enterprise |
| **Docker** | Contenedores | Muy Bajo (<1%) | Medio | Microservicios |
| **Firecracker** | MicroVM | Muy Bajo (~1%) | Alto | AWS Lambda |

**Evolución hacia contenedores:**
```
Virtualización Tradicional:
App → SO Guest → Hypervisor → SO Host → Hardware

Contenedores:
App → Container Runtime → SO Host → Hardware

Beneficios contenedores:
- Menor overhead
- Start time: segundos vs minutos
- Mayor densidad (más contenedores por host)
- Pero: Menor aislamiento
```

---
**Reto 4: Servicios Web y APIs:**

**Problema:** Ofrecer interfaces sencillas que escondan la heterogeneidad de hardware y software en el cloud.

**Desafíos:**

1. **Estandarización de APIs**
   - Cada proveedor tiene APIs propietarias
   - Falta de estándar universal
   - Dificulta portabilidad

2. **Complejidad**
   - AWS tiene 200+ servicios, cada uno con su API
   - Azure similar
   - Curva de aprendizaje alta

3. **Versionado**
   - APIs evolucionan
   - Breaking changes pueden romper aplicaciones
   - Necesidad de versioning (v1, v2, v3)

**Ejemplo de diferencias entre proveedores:**

```python
# AWS S3
import boto3
s3 = boto3.client('s3')
s3.put_object(Bucket='mybucket', Key='file.txt', Body=data)

# Google Cloud Storage
from google.cloud import storage
client = storage.Client()
bucket = client.get_bucket('mybucket')
blob = bucket.blob('file.txt')
blob.upload_from_string(data)

# Azure Blob Storage  
from azure.storage.blob import BlobServiceClient
blob_service = BlobServiceClient.from_connection_string(conn_str)
blob_client = blob_service.get_blob_client(container='mybucket', blob='file.txt')
blob_client.upload_blob(data)

# APIs completamente diferentes para misma funcionalidad
# Solución: Abstracciones (Apache Libcloud, etc.)
```

**Iniciativas de estandarización:**

- **OCCI** (Open Cloud Computing Interface)
- **CIMI** (Cloud Infrastructure Management Interface)
- **TOSCA** (Topology and Orchestration Specification)
- **OpenAPI** (Swagger) para documentación

Pero adopción limitada, proveedores prefieren APIs propietarias (lock-in).

**Reto 5: Contabilidad y Control de Costes:**

**Problema:** Se requiere control de costos para no desbordar a las organizaciones con incrementos de pagos.

**Desafíos:**

1. **Complejidad de Pricing**
   - Cientos de dimensiones de coste
   - Diferentes precios por región
   - Descuentos por volumen, reservas
   - Difícil estimar costes futuros

**Ejemplo de complejidad AWS EC2:**
```text
Factores de coste:
- Tipo de instancia (t3, m5, c5, r5, etc.) = 400+ tipos
- Región (us-east-1, eu-west-1, etc.) = 30+ regiones
- Sistema operativo (Linux, Windows, RHEL)
- Forma de pago (On-Demand, Reserved, Spot)
- Duración de reserva (1 año, 3 años)
- Tipo de pago reserva (All Upfront, Partial, No Upfront)
- Uso de red (salida de datos)
- Volúmenes EBS (gp3, io2, st1, sc1)

Combinaciones posibles: Millones
Resultado: Difícil optimizar sin herramientas
```

2. **Costes inesperados**
   - Recursos olvidados ("zombie resources")  
   - Snapshots acumulados
   - Logs no rotados
   - Auto-scaling mal configurado

**Herramientas de FinOps (Financial Operations):**

```python
# 1. AWS Cost Explorer - Análisis histórico y forecast
import boto3
from datetime import datetime, timedelta

ce = boto3.client('ce')

response = ce.get_cost_forecast(
    TimePeriod={
        'Start': datetime.now().strftime('%Y-%m-%d'),
        'End': (datetime.now() + timedelta(days=30)).strftime('%Y-%m-%d')
    },
    Metric='UNBLENDED_COST',
    Granularity='MONTHLY'
)

forecast = float(response['Total']['Amount'])
print(f"Forecast próximo mes: ${forecast:.2f}")

# 2. AWS Budgets - Alertas proactivas
budgets = boto3.client('budgets')

budgets.create_budget(
    AccountId='123456789012',
    Budget={
        'BudgetName': 'Presupuesto-Mensual',
        'BudgetLimit': {
            'Amount': '10000',  # $10,000/mes
            'Unit': 'USD'
        },
        'TimeUnit': 'MONTHLY',
        'BudgetType': 'COST'
    },
    NotificationsWithSubscribers=[
        {
            'Notification': {
                'NotificationType': 'ACTUAL',
                'ComparisonOperator': 'GREATER_THAN',
                'Threshold': 80,  # Alertar al 80%
                'ThresholdType': 'PERCENTAGE'
            },
            'Subscribers': [
                {
                    'SubscriptionType': 'EMAIL',
                    'Address': 'finanzas@empresa.com'
                }
            ]
        }
    ]
)

# Ahora recibirás email cuando gastes >$8,000 (80% de $10K)
```

**Reto 6: Estandarización:**

**Problema:** El manejo de estándares para el despliegue de nubes aún no está del todo establecido.

**Organizaciones trabajando en estandarización:**

- **Cloud Standards Customer Council**: https://cloud-standards.org/
- **NIST** (National Institute of Standards and Technology)
- **ISO/IEC 17788** y **ISO/IEC 17789**: Cloud Computing Standards
- **DMTF** (Distributed Management Task Force)
- **OASIS** (Organization for the Advancement of Structured Information Standards)

**Áreas de estandarización:**

```text
┌────────────────────────────────────────────────────┐
│      ÁREAS DE ESTANDARIZACIÓN CLOUD                │
│                                                    │
│  1. Terminología y definiciones                    │
│  2. Arquitectura de referencia                     │
│  3. Interfaces y APIs                              │
│  4. Seguridad y privacidad                         │
│  5. Portabilidad e interoperabilidad               │
│  6. SLA (Service Level Agreements)                 │
│  7. Governance y compliance                        │
│  8. Auditoría y certificación                      │
└────────────────────────────────────────────────────┘
```

**Problema:** A pesar de esfuerzos, los grandes proveedores (AWS, Azure, GCP) siguen usando APIs propietarias → Su interés es lock-in de clientes.

**Solución parcial:** Abstracciones y multi-cloud tools:
- Terraform (HashiCorp)
- Pulumi
- Apache Libcloud
- Kubernetes (abstracción de cómputo)

---

## 3.6. Estrategias De Adopción De Cloud Computing

**Ciclo de Vida de Adopción:**

La adopción de cloud computing requiere un enfoque metodológico estructurado en fases:

```
┌──────────────────────────────────────────────────────┐
│      CICLO DE VIDA DE ADOPCIÓN DE CLOUD              │
│                                                      │
│  Fase 1: EVALUACIÓN (Assessment)                     │
│       ↓                                              │
│  Fase 2: PRUEBA DE CONCEPTO (PoC)                    │
│       ↓                                              │
│  Fase 3: MIGRACIÓN PILOTO                            │
│       ↓                                              │
│  Fase 4: PRUEBAS                                     │
│       ↓                                              │
│  Fase 5: GO-LIVE (Producción)                        │
│       ↓                                              │
│  Fase 6: AUDITORÍA Y OPTIMIZACIÓN                    │
└──────────────────────────────────────────────────────┘
```

**Fase 1: Evaluación (Assessment):**

**Objetivo:** Evaluar desafíos, perspectivas e impacto en el mercado del cloud.

**Actividades:**

1. **Evaluación de criticidad de datos**
2. **Funcionalidades del proveedor de servicios cloud**
3. **Seguridad** (certificaciones, modelo responsabilidad compartida)
4. **SLA** (garantías de uptime)
5. **Evaluación de características técnicas**
6. **Análisis de costo-beneficio estimado**

**Fase 2: Prueba de Concepto (PoC):**

**Objetivo:** Evaluar las funcionalidades de diferentes vendedores con pruebas reales.

**Actividades:**

1. Acceso temporal a proveedores (Free Tier)
2. Pruebas técnicas con caso de uso real
3. Evaluación de equipo IT

**Fase 3: Migración Piloto:**

**Objetivo:** Migrar un subconjunto pequeño y no crítico al cloud.

**Actividades:**

1. Seleccionar aplicación piloto
2. Migrar unos pocos usuarios (beta testers)
3. Análisis de beneficios reales
4. Migración de datos (Lift and Shift, Replatforming, Refactoring)

**Fase 4: Pruebas:**

**Objetivo:** Validar que el sistema migrado cumple requisitos.

**Actividades:**

1. Pruebas de aceptación de usuarios (UAT)
2. Análisis de beneficios reales (métricas)
3. Cálculo de ROI
4. Problemas y riesgos identificados
5. Pruebas de seguridad
6. Monitoreo de uptime

**Fase 5: Go-Live (En Producción):**

**Objetivo:** Desplegar en producción de manera gradual y controlada.

**Actividades:**

1. Despliegue en fases (Blue-Green Deployment)
2. Entrenamiento de los usuarios
3. Documentación completa
4. Prueba de aceptación de usuarios final

**Fase 6: Auditoría y Optimización:**

**Objetivo:** Seguimiento continuo y optimización post-despliegue.

**Actividades:**

1. Seguimiento del proceso de despliegue
2. Monitoreo durante producción
3. Optimización continua de costes
4. Optimización de arquitectura
5. Auditoría de seguridad continua

**Cuatro Estrategias Principales de Adopción:**

**1. Scalability-Driven Strategy (Orientada a la Escalabilidad):**

**Definición:** Uso de los recursos del cloud para soportar cargas adicionales o como respaldo.

**Casos de uso:**

- **Cloud Bursting:** Workload normal en on-premise, picos en cloud
- **Auto-scaling horizontal:** Añadir instancias automáticamente

**Ejemplo:**
```
E-commerce durante Black Friday:
- Capacidad base (on-premise): 10,000 req/sec
- Pico Black Friday: 100,000 req/sec
- Burst a cloud: +90,000 req/sec adicionales
- Solo durante el evento (1 día)
```

**2. Availability-Driven Strategy (Orientada a la Disponibilidad):**

**Definición:** Uso de recursos del cloud balanceados y localizados para incrementar la disponibilidad y reducir los tiempos de respuesta.

**Casos de uso:**

- **Multi-región deployment:** Aplicación en múltiples regiones geográficas
- **Disaster Recovery:** Cloud como backup de datacenter principal
- **Edge locations:** Content Delivery Networks (CDNs)

**3. Market-Driven Strategy (Orientada al Negocio):**

>**Definición:** Usuarios y proveedores de los recursos del cloud toman decisiones basadas en el potencial ahorro y ganancias.

**Enfoque:** Puramente económico/financiero

**Casos de uso:**

- **Startups:** Evitar CAPEX masivo, usar OPEX
- **Empresas en crecimiento:** Escalar según demanda
- **Proyectos temporales:** Solo pagar durante el proyecto

**4. Convenience-Driven Strategy (Orientada a la Conveniencia):**

>**Definición:** Uso de los recursos del cloud para no mantener recursos locales.

**Motivación:** Simplicidad operativa

**Casos de uso:**

- **Empresas sin equipo IT:** No tienen capacidad de gestionar datacenter
- **Focus en core business:** Empresa de salud quiere centrarse en medicina, no en IT
- **Eliminar complejidad:** No gestionar hardware, actualizaciones, seguridad física

**Selección de Estrategia: Factores Clave:**

Para elegir la estrategia apropiada de adopción, considerar:

**Preguntas estratégicas:**

1. **¿Dónde aportará valor al negocio actual?**
   - Reducción de costes
   - Mejora de agilidad
   - Nuevas capacidades (ML, analytics)
   - Mejor experiencia de usuario

2. **¿Puede ser implementado solo en áreas seleccionadas del negocio?**
   - Comenzar con workloads no críticos
   - Enfoque híbrido: Crítico on-premise, resto cloud

3. **¿Cuánto se puede controlar de la migración?**
   - Migración gradual (meses/años)
   - Big bang (todo de una vez) - ⚠️ Alto riesgo

**Las respuestas a estas preguntas deben conducir a las decisiones de la organización de implementar el cloud sobre la base de:**

- ✅ Escalabilidad
- ✅ Disponibilidad
- ✅ Coste
- ✅ Conveniencia

---

## 3.7. Regiones Y Disponibilidad

**Regiones (Regions):**

**Definición**: Ubicación geográfica con múltiples datacenters.

**Ejemplos AWS:**

- `eu-west-1`: Irlanda
- `eu-south-2`: España (Aragón) ← Nuevo 2022
- `us-east-1`: Virginia del Norte
- `ap-southeast-1`: Singapur

**Factores para elegir región:**

1. **Latencia**
   - Más cerca del usuario = Menor latencia
   - Ejemplo: Usuarios en España → `eu-south-2` o `eu-west-1`

2. **Cumplimiento legal**
   - RGPD: Los datos de residentes UE deben estar en UE
   - Datos sanitarios españoles → Preferible región España

3. **Coste**
   - Varía por región
   - España suele ser ~5-10% más cara que Irlanda

4. **Servicios disponibles**
   - No todos los servicios en todas las regiones
   - Regiones nuevas tienen menos servicios

**Zonas de Disponibilidad (Availability Zones):**

**Definición**: Datacenters aislados dentro de una región.

```
Región eu-south-2 (España)
    ├── AZ-1 (Datacenter A)
    ├── AZ-2 (Datacenter B)
    └── AZ-3 (Datacenter C)
```

**Alta Disponibilidad:**
```python
# Ejemplo: Desplegar en múltiples AZs

# Base de datos replicada en 3 AZs
rds = boto3.client('rds')

rds.create_db_instance(
    DBInstanceIdentifier='mi-db-sanitaria',
    DBInstanceClass='db.r5.xlarge',
    Engine='postgres',
    MultiAZ=True,  # ← Replica automática en otra AZ
    AllocatedStorage=100,
    StorageEncrypted=True
)

# Load Balancer distribuyendo entre AZs
elbv2 = boto3.client('elbv2')

lb = elbv2.create_load_balancer(
    Name='mi-load-balancer',
    Subnets=[
        'subnet-az1',  # AZ-1
        'subnet-az2',  # AZ-2
        'subnet-az3'   # AZ-3
    ],
    Scheme='internet-facing'
)
```

**SLA (Service Level Agreement):**

**Definición**: Garantía de disponibilidad del proveedor.

| Servicio | SLA | Downtime anual permitido |
|----------|-----|--------------------------|
| **EC2 (1 AZ)** | 99.5% | 1.8 días |
| **EC2 (Multi-AZ)** | 99.99% | 52 minutos |
| **S3** | 99.99% | 52 minutos |
| **RDS (Multi-AZ)** | 99.95% | 4.4 horas |
| **DynamoDB** | 99.999% | 5.3 minutos |

**Cálculo de disponibilidad compuesta:**

```
Disponibilidad total = Disp(Componente1) × Disp(Componente2) × ...

Ejemplo:
Load Balancer (99.99%) → App Server (99.95%) → Database (99.99%)
= 0.9999 × 0.9995 × 0.9999
= 0.9993 = 99.93%
```

## 3.8. Escalabilidad

**Escalado Vertical (Scale Up):**

**Definición**: Aumentar recursos de la misma máquina.

```
Antes: t3.medium (2 vCPU, 4 GB RAM)
         ↓
Después: t3.2xlarge (8 vCPU, 32 GB RAM)
```

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| Simple (sin cambios en arquitectura) | Límite físico |
| No requiere distribución de carga | Requiere downtime (normalmente) |
| | Single point of failure |

**Cuándo usar:**

- Aplicaciones legacy no diseñadas para escalar horizontalmente
- Bases de datos tradicionales
- Fases iniciales

**Escalado Horizontal (Scale Out):**

**Definición**: Aumentar número de máquinas.

```
Antes: 2 servidores
         ↓
Después: 10 servidores
```

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| Sin límite práctico | Requiere arquitectura distribuida |
| Alta disponibilidad | Complejidad de sincronización |
| Sin downtime | Load balancing necesario |

**Auto-scaling:**

**Definición**: Ajuste automático de recursos según demanda.

```python
# Ejemplo: Auto-scaling en AWS

import boto3

autoscaling = boto3.client('autoscaling')

# Crear configuración de lanzamiento
autoscaling.create_launch_configuration(
    LaunchConfigurationName='mi-config-web',
    ImageId='ami-xxxxx',
    InstanceType='t3.medium',
    SecurityGroups=['sg-xxxxx']
)

# Crear grupo de auto-scaling
autoscaling.create_auto_scaling_group(
    AutoScalingGroupName='mi-asg-web',
    LaunchConfigurationName='mi-config-web',
    MinSize=2,           # Mínimo 2 instancias siempre
    MaxSize=20,          # Máximo 20 instancias
    DesiredCapacity=4,   # Normalmente 4
    AvailabilityZones=['eu-south-2a', 'eu-south-2b', 'eu-south-2c'],
    HealthCheckType='ELB',
    HealthCheckGracePeriod=300
)

# Política de escalado: Escalar si CPU > 70%
autoscaling.put_scaling_policy(
    AutoScalingGroupName='mi-asg-web',
    PolicyName='scale-up-cpu',
    AdjustmentType='ChangeInCapacity',
    ScalingAdjustment=2,  # Añadir 2 instancias
    Cooldown=300
)

# CloudWatch alarm para trigger
cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_alarm(
    AlarmName='high-cpu-alarm',
    MetricName='CPUUtilization',
    Namespace='AWS/EC2',
    Statistic='Average',
    Period=300,
    EvaluationPeriods=2,
    Threshold=70.0,
    ComparisonOperator='GreaterThanThreshold',
    AlarmActions=[
        'arn:aws:autoscaling:eu-south-2:123456789012:scalingPolicy:...'
    ]
)
```

**Auto-scaling para Big Data:**

```python
# Ejemplo: Cluster Spark con auto-scaling en AWS EMR

emr = boto3.client('emr')

cluster_id = emr.run_job_flow(
    Name='cluster-analisis-epidemiologico',
    ReleaseLabel='emr-6.9.0',
    Applications=[
        {'Name': 'Spark'},
        {'Name': 'Hadoop'},
        {'Name': 'Hive'}
    ],
    Instances={
        'InstanceGroups': [
            {
                'Name': 'Master',
                'Market': 'ON_DEMAND',
                'InstanceRole': 'MASTER',
                'InstanceType': 'r5.2xlarge',
                'InstanceCount': 1
            },
            {
                'Name': 'Core',
                'Market': 'ON_DEMAND',
                'InstanceRole': 'CORE',
                'InstanceType': 'r5.4xlarge',
                'InstanceCount': 2,  # Mínimo
                'AutoScalingPolicy': {
                    'Constraints': {
                        'MinCapacity': 2,
                        'MaxCapacity': 20
                    },
                    'Rules': [
                        {
                            'Name': 'Scale-up',
                            'Action': {
                                'SimpleScalingPolicyConfiguration': {
                                    'AdjustmentType': 'CHANGE_IN_CAPACITY',
                                    'ScalingAdjustment': 4,
                                    'CoolDown': 300
                                }
                            },
                            'Trigger': {
                                'CloudWatchAlarmDefinition': {
                                    'MetricName': 'YARNMemoryAvailablePercentage',
                                    'ComparisonOperator': 'LESS_THAN',
                                    'Threshold': 15.0,
                                    'Period': 300
                                }
                            }
                        }
                    ]
                }
            }
        ],
        'Ec2KeyName': 'my-key',
        'KeepJobFlowAliveWhenNoSteps': True
    },
    LogUri='s3://mi-bucket-logs-emr/',
    ServiceRole='EMR_DefaultRole',
    JobFlowRole='EMR_EC2_DefaultRole'
)
```

## 3.9. Modelos De Costes

**CAPEX vs OPEX:**

| CAPEX (Capital Expenditure) | OPEX (Operational Expenditure) |
|-----------------------------|--------------------------------|
| Inversión inicial alta | Sin inversión inicial |
| Compra de servidores físicos | Pago por uso mensual |
| Depreciación a lo largo de años | Gasto corriente |
| Predicción difícil de necesidades | Escala según demanda real |
| **Datacenters tradicionales** | **Cloud Computing** |

**Modelos de Pago en Cloud:**

**1. On-Demand (Bajo demanda):**

**Características:**

- Pago por hora o segundo
- Sin compromiso
- Sin pago inicial

| ✅ Ventajas | ❌ Desventajas |
|------------|----------------|
| Flexibilidad máxima | Precio más alto |
| Sin compromiso | |

**Cuándo usar:**

- Desarrollo y pruebas
- Cargas de trabajo impredecibles
- Aplicaciones nuevas

**Ejemplo de precio (AWS EC2 t3.2xlarge en eu-south-2):**
```
$0.3712 por hora = $267/mes (24x7)
```

**2. Reserved Instances (Instancias Reservadas):**

**Características:**

- Compromiso de 1 o 3 años
- Descuento del 30-75%
- Pago inicial o mensual

**Ejemplo:**
```
On-Demand: $267/mes
Reserved 1 año: $175/mes (34% descuento)
Reserved 3 años: $110/mes (59% descuento)
```

**Cuándo usar:**

- Cargas de trabajo estables y predecibles
- Bases de datos de producción
- Servidores web base

**3. Spot Instances (Instancias Spot):**

**Características:**

- Usar capacidad no utilizada de AWS
- Descuento del 50-90%
- Pueden ser interrumpidas con 2 minutos de aviso

**Ejemplo:**
```
On-Demand: $0.3712/hora
Spot: $0.08-0.15/hora (70-80% descuento)
```

**Cuándo usar:**

- Procesamiento Big Data tolerante a fallos
- Análisis batch no urgentes
- Machine Learning training

```python
# Ejemplo: Cluster Spark con Spot Instances

cluster_id = emr.run_job_flow(
    Name='cluster-spot-analisis',
    Instances={
        'InstanceGroups': [
            {
                'Name': 'Master',
                'Market': 'ON_DEMAND',  # Master siempre ON_DEMAND
                'InstanceRole': 'MASTER',
                'InstanceType': 'r5.xlarge',
                'InstanceCount': 1
            },
            {
                'Name': 'Core',
                'Market': 'ON_DEMAND',  # Core ON_DEMAND (datos)
                'InstanceRole': 'CORE',
                'InstanceType': 'r5.2xlarge',
                'InstanceCount': 2
            },
            {
                'Name': 'Task',
                'Market': 'SPOT',  # Task SPOT (solo procesamiento)
                'InstanceRole': 'TASK',
                'InstanceType': 'r5.4xlarge',
                'InstanceCount': 10,
                'BidPrice': '0.50'  # Máximo a pagar por hora
            }
        ]
    }
)

# Resultado: 80% de capacidad de cómputo a precio spot
# Ahorro: ~$500/mes → ~$150/mes
```

**Estimación de Costes:**

**Herramientas:**

- **AWS Pricing Calculator**: https://calculator.aws
- **Azure Pricing Calculator**: https://azure.microsoft.com/pricing/calculator
- **GCP Pricing Calculator**: https://cloud.google.com/products/calculator

**Ejemplo de arquitectura de Data Lake:**

```python
# Estimación mensual para Data Lake + Analytics

# 1. Almacenamiento S3
# 500 GB de datos sanitarios
s3_storage = 500 * 0.023  # $0.023/GB/mes
# = $11.5/mes

# 2. Procesamiento Spark EMR
# 2 h/día de procesamiento con cluster de 10 r5.2xlarge
emr_compute = 10 * 0.504 * 2 * 30  # $0.504/hora
# = $302.4/mes

# 3. Base de datos RDS PostgreSQL
# db.r5.xlarge Multi-AZ
rds_cost = 0.848 * 24 * 30
# = $611/mes

# 4. Transferencia de datos
# 100 GB/mes de salida
data_transfer = 100 * 0.09  # $0.09/GB
# = $9/mes

# TOTAL MENSUAL
total = s3_storage + emr_compute + rds_cost + data_transfer
# = $933.9/mes

print(f"Coste mensual estimado: ${total:.2f}")
```

**Optimización de Costes:**

**Estrategias:**

1. **Rightsizing**: Ajustar tamaño de instancias
2. **Auto-scaling**: Escalar solo cuando necesario
3. **Reserved Instances**: Para cargas base
4. **Spot Instances**: Para procesamiento batch
5. **S3 Lifecycle**: Mover datos antiguos a almacenamiento más barato
6. **Eliminar recursos no utilizados**: Snapshots, volúmenes huérfanos

```python
# Ejemplo: S3 Lifecycle para optimizar costes

s3 = boto3.client('s3')

lifecycle_policy = {
    'Rules': [
        {
            'ID': 'Mover datos antiguos a IA',
            'Status': 'Enabled',
            'Filter': {'Prefix': 'datalake/raw/'},
            'Transitions': [
                {
                    'Days': 30,
                    'StorageClass': 'INTELLIGENT_TIERING'  # Auto-optimiza
                },
                {
                    'Days': 90,
                    'StorageClass': 'GLACIER'  # Archivado barato
                }
            ]
        },
        {
            'ID': 'Eliminar logs antiguos',
            'Status': 'Enabled',
            'Filter': {'Prefix': 'logs/'},
            'Expiration': {'Days': 180}  # Borrar después de 6 meses
        }
    ]
}

s3.put_bucket_lifecycle_configuration(
    Bucket='mi-datalake-sanitario',
    LifecycleConfiguration=lifecycle_policy
)

# Ahorro: S3 Standard ($0.023/GB) → Glacier ($0.004/GB)
# Para 500 GB con >90 días: $11.5/mes → $2/mes (82% ahorro)
```

---

