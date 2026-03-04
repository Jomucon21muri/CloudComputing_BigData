# CAPÍTULO 12: GOBIERNO DE DATOS - CALIDAD Y TRAZABILIDAD

!!! abstract "Resumen del Capítulo"
    La **calidad de datos** es fundamental para tomar decisiones empresariales confiables. Este capítulo explora las dimensiones de calidad, métricas, herramientas y estrategias para implementar un programa completo de Data Quality y trazabilidad.

## Contenido

1. [Dimensiones de Calidad de Datos](#121-dimensiones-de-calidad-de-datos)
2. [Data Quality Assessment](#122-data-quality-assessment)
3. [Trazabilidad y Linaje de Datos](#123-trazabilidad-y-linaje-de-datos)
4. [Herramientas de Data Quality](#124-herramientas-de-data-quality)
5. [Implementación Práctica con Great Expectations](#125-implementacion-practica-con-great-expectations)
6. [Monitoreo y Alertas](#126-monitoreo-y-alertas)

---

## Dimensiones de calidad de datos

**Las 6 dimensiones principales:**

| Dimensión | Definición | Ejemplo de Problema | Impacto |
|-----------|------------|---------------------|---------|
| **Exactitud** (Accuracy) | Datos representan correctamente la realidad | Email: "usuario@gmial.com" | ❌ Comunicación fallida |
| **Completitud** (Completeness) | Todos los campos requeridos tienen valor | Dirección sin código postal | ⚠️ Envíos fallidos |
| **Consistencia** (Consistency) | Mismos datos en múltiples sistemas | Cliente "Juan Pérez" vs "J. Perez" | 🔄 Duplicados |
| **Actualidad** (Timeliness) | Datos disponibles cuando se necesitan | Precios desactualizados | 💰 Pérdidas |
| **Validez** (Validity) | Datos cumplen reglas de negocio | Edad = -5 años | 🚫 Análisis erróneo |
| **Unicidad** (Uniqueness) | Sin duplicados innecesarios | Mismo cliente 3 veces | 📊 Métricas incorrectas |

**Diagrama de calidad de datos:**

```mermaid
graph TD
    A[Datos de Origen] --> B{Validación de Calidad}
    B -->|Exactitud| C[Verificación de Formato]
    B -->|Completitud| D[Check de Campos Nulos]
    B -->|Consistencia| E[Validación Cruzada]
    B -->|Actualidad| F[Verificación de Timestamp]
    B -->|Validez| G[Reglas de Negocio]
    B -->|Unicidad| H[Detección de Duplicados]
    
    C --> I{¿Pasa todas<br/>las validaciones?}
    D --> I
    E --> I
    F --> I
    G --> I
    H --> I
    
    I -->|Sí| J[Datos de Alta Calidad]
    I -->|No| K[Cuarentena/Rechazo]
    K --> L[Alerta al Equipo]
    K --> M[Log de Errores]
    
    style J fill:#c8e6c9
    style K fill:#ffccbc
    style L fill:#ffe0b2
```

---

## Data Quality Assessment

**Métricas clave:**

**1. Completitud score:**

```python
import pandas as pd

def calculate_completeness(df):
    """
    Calcula el porcentaje de completitud por columna
    """
    completeness = {}
    
    for column in df.columns:
        total_rows = len(df)
        non_null_rows = df[column].notna().sum()
        completeness_pct = (non_null_rows / total_rows) * 100
        
        completeness[column] = {
            'non_null_count': int(non_null_rows),
            'null_count': int(total_rows - non_null_rows),
            'completeness_pct': round(completeness_pct, 2)
        }
    
    return pd.DataFrame(completeness).T

# Ejemplo de uso
df = pd.read_csv('customers.csv')
completeness_report = calculate_completeness(df)
print(completeness_report)
```

**Salida:**
```
                   non_null_count  null_count  completeness_pct
customer_id              10000           0              100.00
email                     9856         144               98.56
phone                     9234         766               92.34
address                   8901        1099               89.01
```

**2. Validez score:**

```python
import re

def validate_data_quality(df):
    """
    Valida reglas de negocio y devuelve score de calidad
    """
    validations = {}
    
    # Email válido
    if 'email' in df.columns:
        email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        valid_emails = df['email'].apply(
            lambda x: bool(re.match(email_pattern, str(x))) if pd.notna(x) else False
        )
        validations['email_validity'] = (valid_emails.sum() / len(df)) * 100
    
    # Edad válida (0-120)
    if 'age' in df.columns:
        valid_ages = df['age'].between(0, 120, inclusive='both')
        validations['age_validity'] = (valid_ages.sum() / len(df)) * 100
    
    # Monto positivo
    if 'amount' in df.columns:
        valid_amounts = df['amount'] > 0
        validations['amount_validity'] = (valid_amounts.sum() / len(df)) * 100
    
    return pd.Series(validations)

# Uso
validity_scores = validate_data_quality(df)
print(f"\n📊 Validity Scores:\n{validity_scores}")
```

**3. Unicidad score:**

```python
def calculate_uniqueness(df, columns):
    """
    Calcula el porcentaje de registros únicos
    """
    total_rows = len(df)
    unique_rows = df[columns].drop_duplicates().shape[0]
    duplicates = total_rows - unique_rows
    
    uniqueness_pct = (unique_rows / total_rows) * 100
    
    return {
        'total_records': total_rows,
        'unique_records': unique_rows,
        'duplicate_records': duplicates,
        'uniqueness_pct': round(uniqueness_pct, 2)
    }

# Ejemplo
uniqueness = calculate_uniqueness(df, ['email'])
print(f"\n🔍 Uniqueness Report: {uniqueness}")
```

**Dashboard de calidad:**

```python
import matplotlib.pyplot as plt
import seaborn as sns

def create_quality_dashboard(df):
    """
    Crea un dashboard visual de calidad de datos
    """
    fig, axes = plt.subplots(2, 2, figsize=(15, 10))
    fig.suptitle('📊 Data Quality Dashboard', fontsize=16, fontweight='bold')
    
    # 1. Completitud por columna
    completeness = df.notna().sum() / len(df) * 100
    completeness.plot(kind='barh', ax=axes[0, 0], color='skyblue')
    axes[0, 0].set_title('Completitud por Columna (%)')
    axes[0, 0].set_xlabel('Completitud %')
    axes[0, 0].axvline(x=95, color='green', linestyle='--', label='Target 95%')
    axes[0, 0].legend()
    
    # 2. Distribución de nulos
    null_counts = df.isnull().sum()
    null_counts[null_counts > 0].plot(kind='bar', ax=axes[0, 1], color='coral')
    axes[0, 1].set_title('Registros con Valores Nulos')
    axes[0, 1].set_ylabel('Cantidad')
    axes[0, 1].set_xlabel('Columnas')
    
    # 3. Duplicados
    duplicates_info = pd.DataFrame({
        'Tipo': ['Únicos', 'Duplicados'],
        'Cantidad': [
            df.drop_duplicates().shape[0],
            df.shape[0] - df.drop_duplicates().shape[0]
        ]
    })
    duplicates_info.plot(kind='pie', y='Cantidad', labels=duplicates_info['Tipo'],
                         ax=axes[1, 0], autopct='%1.1f%%', startangle=90,
                         colors=['lightgreen', 'lightcoral'])
    axes[1, 0].set_title('Distribución Únicos vs Duplicados')
    axes[1, 0].set_ylabel('')
    
    # 4. Score general de calidad
    overall_score = completeness.mean()
    axes[1, 1].text(0.5, 0.5, f'{overall_score:.1f}%', 
                    ha='center', va='center', fontsize=60, fontweight='bold',
                    color='green' if overall_score >= 95 else 'orange' if overall_score >= 80 else 'red')
    axes[1, 1].text(0.5, 0.2, 'Overall Quality Score',
                    ha='center', va='center', fontsize=14)
    axes[1, 1].axis('off')
    
    plt.tight_layout()
    return fig

# Generar dashboard
dashboard = create_quality_dashboard(df)
plt.savefig('quality_dashboard.png', dpi=300, bbox_inches='tight')
```

---

## Trazabilidad y linaje de datos

**Data Lineage:**

**Data Lineage** documenta el flujo de datos desde el origen hasta el destino, incluyendo todas las transformaciones.

```mermaid
graph LR
    A[CRM Database<br/>customers] --> B[ETL Job 1<br/>Extract & Clean]
    C[ERP Database<br/>orders] --> B
    B --> D[Data Lake<br/>S3 Trusted]
    D --> E[Spark Job<br/>Transform & Enrich]
    E --> F[Data Warehouse<br/>Redshift]
    F --> G[BI Dashboard<br/>Tableau]
    F --> H[ML Model<br/>Churn Prediction]
    
    style A fill:#e3f2fd
    style C fill:#e3f2fd
    style D fill:#fff9c4
    style F fill:#c8e6c9
    style G fill:#f8bbd0
    style H fill:#d1c4e9
```

**Metadata Management:**

```python
from dataclasses import dataclass
from datetime import datetime
from typing import List, Optional

@dataclass
class DatasetMetadata:
    """
    Metadatos completos de un dataset
    """
    dataset_name: str
    schema_name: str
    table_name: str
    source_system: str
    owner: str
    created_at: datetime
    last_modified: datetime
    row_count: int
    size_mb: float
    columns: List[dict]
    upstream_datasets: List[str]
    downstream_datasets: List[str]
    data_classification: str  # Public, Internal, Confidential, Restricted
    retention_days: int
    quality_score: Optional[float] = None
    description: Optional[str] = None
    
    def to_dict(self):
        """Convierte a diccionario para serialización"""
        return {
            'dataset_name': self.dataset_name,
            'schema': self.schema_name,
            'table': self.table_name,
            'source': self.source_system,
            'owner': self.owner,
            'created_at': self.created_at.isoformat(),
            'last_modified': self.last_modified.isoformat(),
            'row_count': self.row_count,
            'size_mb': round(self.size_mb, 2),
            'columns': self.columns,
            'lineage': {
                'upstream': self.upstream_datasets,
                'downstream': self.downstream_datasets
            },
            'governance': {
                'classification': self.data_classification,
                'retention_days': self.retention_days
            },
            'quality_score': self.quality_score,
            'description': self.description
        }

# Ejemplo de uso
metadata = DatasetMetadata(
    dataset_name='customer_360',
    schema_name='analytics',
    table_name='dim_customer',
    source_system='Salesforce CRM',
    owner='data-team@company.com',
    created_at=datetime(2024, 1, 15),
    last_modified=datetime.now(),
    row_count=1250000,
    size_mb=450.5,
    columns=[
        {'name': 'customer_id', 'type': 'bigint', 'nullable': False, 'is_pii': False},
        {'name': 'email', 'type': 'varchar(255)', 'nullable': False, 'is_pii': True},
        {'name': 'phone', 'type': 'varchar(20)', 'nullable': True, 'is_pii': True},
        {'name': 'lifetime_value', 'type': 'decimal(12,2)', 'nullable': True, 'is_pii': False}
    ],
    upstream_datasets=['crm.customers', 'erp.orders', 'web.events'],
    downstream_datasets=['ml_models.churn_prediction', 'dashboards.executive_kpis'],
    data_classification='Confidential',
    retention_days=2555,  # 7 años
    quality_score=94.5,
    description='Vista 360 del cliente con datos consolidados de CRM, ERP y Web'
)

import json
print(json.dumps(metadata.to_dict(), indent=2))
```

**Apache Atlas: catálogo de datos:**

```python
from atlasclient.client import Atlas

# Conexión a Atlas
atlas_client = Atlas(
    host='atlas.company.com',
    port=21000,
    username='admin',
    password='password'
)

# Registrar un dataset en Atlas
def register_dataset_in_atlas(metadata):
    """
    Registra un dataset y su linaje en Apache Atlas
    """
    # Crear entidad de tipo 'Table'
    table_entity = {
        'typeName': 'hive_table',
        'attributes': {
            'qualifiedName': f'{metadata.schema_name}.{metadata.table_name}@cluster',
            'name': metadata.table_name,
            'owner': metadata.owner,
            'createTime': int(metadata.created_at.timestamp() * 1000),
            'lastAccessTime': int(metadata.last_modified.timestamp() * 1000),
            'retention': metadata.retention_days,
            'comment': metadata.description
        },
        'classifications': [
            {'typeName': metadata.data_classification}
        ]
    }
    
    # Crear entidades de columnas
    column_entities = []
    for col in metadata.columns:
        column_entity = {
            'typeName': 'hive_column',
            'attributes': {
                'qualifiedName': f'{metadata.schema_name}.{metadata.table_name}.{col["name"]}@cluster',
                'name': col['name'],
                'type': col['type'],
                'isNullable': col['nullable'],
                'comment': f'PII: {col["is_pii"]}'
            },
            'classifications': [
                {'typeName': 'PII'}
            ] if col['is_pii'] else []
        }
        column_entities.append(column_entity)
    
    # Crear linaje (procesos)
    for upstream in metadata.upstream_datasets:
        lineage_process = {
            'typeName': 'Process',
            'attributes': {
                'qualifiedName': f'etl_{upstream}_to_{metadata.table_name}@cluster',
                'name': f'ETL: {upstream} → {metadata.table_name}',
                'inputs': [{'typeName': 'hive_table', 'uniqueAttributes': {'qualifiedName': f'{upstream}@cluster'}}],
                'outputs': [{'typeName': 'hive_table', 'uniqueAttributes': {'qualifiedName': f'{metadata.schema_name}.{metadata.table_name}@cluster'}}]
            }
        }
        # atlas_client.entity_post.create(data=lineage_process)
    
    print(f"✅ Dataset {metadata.table_name} registrado en Atlas")
    print(f"📊 Linaje: {len(metadata.upstream_datasets)} upstream, {len(metadata.downstream_datasets)} downstream")

# Registrar
register_dataset_in_atlas(metadata)
```

---

## Herramientas de Data Quality

**Comparativa de herramientas:**

| Herramienta | Vendor | Tipo | Características | Precio |
|-------------|--------|------|-----------------|--------|
| **Great Expectations** | Open Source | Python Library | Validaciones declarativas, Profiling | Gratis |
| **Deequ** | AWS/Open | Scala (Spark) | Spark-native, métricas, sugerencias | Gratis |
| **Monte Carlo** | Commercial | SaaS | ML-powered, anomaly detection | $$$$ |
| **Datadog** | Commercial | SaaS | Monitoring end-to-end, alertas | $$$ |
| **Talend** | Commercial | Platform | ETL + Data Quality integrado | $$$$ |
| **Informatica DQ** | Commercial | Platform | Enterprise-grade, MDM integration | $$$$$ |

**Great Expectations:**

```bash
# Instalación
pip install great-expectations
```

```python
import great_expectations as gx

# Inicializar contexto
context = gx.get_context()

# Conectar a datasource
datasource = context.sources.add_pandas("my_datasource")
data_asset = datasource.add_dataframe_asset(name="customers_df")
batch_request = data_asset.build_batch_request(dataframe=df)

# Crear Expectation Suite
suite = context.add_expectation_suite("customer_data_quality")

# Agregar expectations
validator = context.get_validator(
    batch_request=batch_request,
    expectation_suite_name="customer_data_quality"
)

# Expectations específicas
validator.expect_table_row_count_to_be_between(min_value=1000, max_value=2000000)
validator.expect_column_values_to_not_be_null("customer_id")
validator.expect_column_values_to_be_unique("customer_id")
validator.expect_column_values_to_match_regex("email", r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
validator.expect_column_values_to_be_between("age", min_value=0, max_value=120)
validator.expect_column_mean_to_be_between("order_amount", min_value=10, max_value=500)

# Guardar suite
validator.save_expectation_suite(discard_failed_expectations=False)

# Ejecutar validación
checkpoint = context.add_checkpoint(
    name="customer_checkpoint",
    validator=validator
)

results = checkpoint.run()

if results["success"]:
    print("✅ Todas las validaciones pasaron")
else:
    print("❌ Algunas validaciones fallaron:")
    for result in results["run_results"].values():
        for check in result["validation_result"]["results"]:
            if not check["success"]:
                print(f"  - {check['expectation_config']['expectation_type']}: {check['exception_info']['raised_exception']}")
```

---

## Implementación práctica con Great Expectations

**Pipeline completo de validación:**

```python
import pandas as pd
import great_expectations as gx
from great_expectations.checkpoint import SimpleCheckpoint
from datetime import datetime
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class DataQualityPipeline:
    """
    Pipeline completo de Data Quality
    """
    
    def __init__(self, context_root_dir='./gx'):
        self.context = gx.get_context(context_root_dir=context_root_dir)
        self.validation_results = []
    
    def create_expectation_suite(self, suite_name, df):
        """
        Crea una suite de expectations basada en profiling automático
        """
        logger.info(f"Creando expectation suite: {suite_name}")
        
        # Crear o actualizar suite
        try:
            suite = self.context.get_expectation_suite(suite_name)
            logger.info(f"Suite {suite_name} ya existe, actualizando...")
        except:
            suite = self.context.add_expectation_suite(suite_name)
            logger.info(f"Suite {suite_name} creada")
        
        # Agregar datasource temporal
        datasource = self.context.sources.add_pandas("temp_datasource")
        data_asset = datasource.add_dataframe_asset(name="temp_asset")
        batch_request = data_asset.build_batch_request(dataframe=df)
        
        # Crear validator
        validator = self.context.get_validator(
            batch_request=batch_request,
            expectation_suite_name=suite_name
        )
        
        # Expectations automáticas basadas en el dataframe
        for column in df.columns:
            # No nulos para columnas críticas
            if df[column].notna().all():
                validator.expect_column_values_to_not_be_null(column)
            
            # Unicidad para IDs
            if 'id' in column.lower():
                validator.expect_column_values_to_be_unique(column)
            
            # Rangos para numéricos
            if pd.api.types.is_numeric_dtype(df[column]):
                min_val = df[column].min()
                max_val = df[column].max()
                validator.expect_column_values_to_be_between(
                    column,
                    min_value=min_val * 0.9,  # 10% tolerancia
                    max_value=max_val * 1.1
                )
            
            # Regex para emails
            if 'email' in column.lower():
                validator.expect_column_values_to_match_regex(
                    column,
                    regex=r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
                )
        
        # Guardar suite
        validator.save_expectation_suite(discard_failed_expectations=False)
        logger.info(f"✅ Suite {suite_name} guardada con {len(validator.get_expectation_suite().expectations)} expectations")
        
        return validator
    
    def validate_dataframe(self, df, suite_name):
        """
        Valida un dataframe contra una suite de expectations
        """
        logger.info(f"Validando dataframe contra suite: {suite_name}")
        
        # Preparar batch
        datasource = self.context.sources.add_pandas(f"datasource_{datetime.now().timestamp()}")
        data_asset = datasource.add_dataframe_asset(name="validation_asset")
        batch_request = data_asset.build_batch_request(dataframe=df)
        
        # Crear checkpoint
        checkpoint = SimpleCheckpoint(
            name=f"checkpoint_{suite_name}",
            data_context=self.context,
            validations=[
                {
                    "batch_request": batch_request,
                    "expectation_suite_name": suite_name
                }
            ]
        )
        
        # Ejecutar validación
        results = checkpoint.run()
        
        # Procesar resultados
        validation_result = {
            'suite_name': suite_name,
            'success': results["success"],
            'timestamp': datetime.now(),
            'statistics': results["run_results"][list(results["run_results"].keys())[0]]["validation_result"]["statistics"],
            'failed_expectations': []
        }
        
        if not results["success"]:
            for result in results["run_results"].values():
                for check in result["validation_result"]["results"]:
                    if not check["success"]:
                        validation_result['failed_expectations'].append({
                            'expectation': check['expectation_config']['expectation_type'],
                            'column': check['expectation_config'].get('kwargs', {}).get('column'),
                            'details': check.get('exception_info', {}).get('raised_exception', 'Unknown error')
                        })
        
        self.validation_results.append(validation_result)
        return validation_result
    
    def generate_quality_report(self):
        """
        Genera un reporte consolidado de calidad
        """
        report = {
            'total_validations': len(self.validation_results),
            'successful_validations': sum(1 for r in self.validation_results if r['success']),
            'failed_validations': sum(1 for r in self.validation_results if not r['success']),
            'overall_success_rate': 0,
            'validations': self.validation_results
        }
        
        if report['total_validations'] > 0:
            report['overall_success_rate'] = (report['successful_validations'] / report['total_validations']) * 100
        
        return report

# === EJEMPLO DE USO ===

# 1. Inicializar pipeline
pipeline = DataQualityPipeline()

# 2. Cargar datos
df_customers = pd.read_csv('customers.csv')

# 3. Crear suite de expectations
validator = pipeline.create_expectation_suite('customer_quality_suite', df_customers)

# 4. Agregar expectations personalizadas
validator.expect_column_values_to_be_in_set('country', ['ES', 'FR', 'DE', 'IT', 'UK'])
validator.expect_column_pair_values_a_to_be_greater_than_b('registration_date', 'birth_date')
validator.save_expectation_suite()

# 5. Validar datos
validation_result = pipeline.validate_dataframe(df_customers, 'customer_quality_suite')

# 6. Mostrar resultados
if validation_result['success']:
    print("✅ Validación exitosa")
    print(f"📊 Estadísticas: {validation_result['statistics']}")
else:
    print("❌ Validación falló")
    print(f"🚨 Errores encontrados: {len(validation_result['failed_expectations'])}")
    for error in validation_result['failed_expectations']:
        print(f"  - {error['expectation']} en columna '{error['column']}': {error['details']}")

# 7. Generar reporte
quality_report = pipeline.generate_quality_report()
print(f"\n📈 Success Rate: {quality_report['overall_success_rate']:.2f}%")
```

---

## Monitoreo y alertas

**Sistema de alertas automáticas:**

```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from slack_sdk import WebClient
from slack_sdk.errors import SlackApiError

class DataQualityAlerts:
    """
    Sistema de alertas para problemas de calidad de datos
    """
    
    def __init__(self, slack_token=None, email_config=None):
        self.slack_client = WebClient(token=slack_token) if slack_token else None
        self.email_config = email_config
    
    def send_slack_alert(self, channel, message, severity='warning'):
        """
        Envía alerta a Slack
        """
        if not self.slack_client:
            logger.warning("Slack client not configured")
            return
        
        emoji = {
            'critical': ':rotating_light:',
            'warning': ':warning:',
            'info': ':information_source:'
        }.get(severity, ':warning:')
        
        try:
            response = self.slack_client.chat_postMessage(
                channel=channel,
                text=f"{emoji} *Data Quality Alert*\n{message}"
            )
            logger.info(f"Slack alert sent to {channel}")
        except SlackApiError as e:
            logger.error(f"Error sending Slack message: {e}")
    
    def send_email_alert(self, recipients, subject, body):
        """
        Envía alerta por email
        """
        if not self.email_config:
            logger.warning("Email not configured")
            return
        
        msg = MIMEMultipart()
        msg['From'] = self.email_config['from']
        msg['To'] = ', '.join(recipients)
        msg['Subject'] = subject
        
        msg.attach(MIMEText(body, 'html'))
        
        try:
            with smtplib.SMTP(self.email_config['smtp_server'], self.email_config['smtp_port']) as server:
                server.starttls()
                server.login(self.email_config['username'], self.email_config['password'])
                server.send_message(msg)
            logger.info(f"Email alert sent to {recipients}")
        except Exception as e:
            logger.error(f"Error sending email: {e}")
    
    def process_validation_result(self, validation_result):
        """
        Procesa resultado de validación y envía alertas si es necesario
        """
        if not validation_result['success']:
            # Determinar severidad
            failed_count = len(validation_result['failed_expectations'])
            total_expectations = validation_result['statistics']['evaluated_expectations']
            failure_rate = (failed_count / total_expectations) * 100
            
            if failure_rate > 20:
                severity = 'critical'
            elif failure_rate > 5:
                severity = 'warning'
            else:
                severity = 'info'
            
            # Construir mensaje
            message = f"""
*Dataset:* {validation_result['suite_name']}
*Timestamp:* {validation_result['timestamp']}
*Status:* ❌ FAILED
*Failed Expectations:* {failed_count}/{total_expectations} ({failure_rate:.1f}%)

*Failed Checks:*
"""
            for error in validation_result['failed_expectations'][:5]:  # Top 5
                message += f"\n• {error['expectation']} ({error['column']})"
            
            if failed_count > 5:
                message += f"\n\n_...and {failed_count - 5} more issues_"
            
            # Enviar alertas
            self.send_slack_alert('#data-quality', message, severity)
            
            # Email para critical
            if severity == 'critical':
                email_body = f"""
                <html>
                <body>
                    <h2 style="color: #d32f2f;">🚨 Critical Data Quality Issue</h2>
                    <p><strong>Dataset:</strong> {validation_result['suite_name']}</p>
                    <p><strong>Failure Rate:</strong> {failure_rate:.1f}%</p>
                    <p><strong>Failed Checks:</strong> {failed_count}</p>
                    <h3>Details:</h3>
                    <ul>
                    {"".join([f"<li>{e['expectation']} - {e['column']}</li>" for e in validation_result['failed_expectations']])}
                    </ul>
                    <p>Please investigate immediately.</p>
                </body>
                </html>
                """
                self.send_email_alert(
                    ['data-team@company.com'],
                    f"[CRITICAL] Data Quality Alert: {validation_result['suite_name']}",
                    email_body
                )

# Uso en el pipeline
alerts = DataQualityAlerts(
    slack_token='xoxb-your-token',
    email_config={
        'from': 'alerts@company.com',
        'smtp_server': 'smtp.gmail.com',
        'smtp_port': 587,
        'username': 'alerts@company.com',
        'password': 'your-password'
    }
)

# Procesar resultados y enviar alertas
alerts.process_validation_result(validation_result)
```

---

## Resumen y Best Practices

!!! success "Mejores Prácticas de Data Quality"
    1. **Shift Left**: Validar datos lo antes posible en el pipeline
    2. **Automatización**: Integrar validaciones en CI/CD
    3. **Monitoreo Continuo**: Dashboards en tiempo real
    4. **Ownership**: Asignar responsables por cada dataset
    5. **Documentación**: Mantener catálogo de datos actualizado
    6. **Alertas Inteligentes**: No sobrecargar con falsos positivos
    7. **Trazabilidad**: Rastrear origen de cada problema
    8. **Remediación**: Planes de acción claros para fix

**Checklist de implementación:**

- [ ] Definir dimensiones de calidad críticas
- [ ] Establecer métricas y KPIs
- [ ] Implementar herramientas (Great Expectations, Atlas)
- [ ] Crear dashboards de monitoreo
- [ ] Configurar alertas automatizadas
- [ ] Documentar ownership y responsabilidades
- [ ] Establecer procesos de remediación
- [ ] Auditorías periódicas de calidad

---

## Referencias y Recursos

- 📚 [Great Expectations Documentation](https://docs.greatexpectations.io/)
- 📚 [Apache Atlas Documentation](https://atlas.apache.org/)
- 📚 [DAMA DMBOK - Data Quality Chapter](https://www.dama.org/cpages/body-of-knowledge)
- 📚 [ISO 8000 - Data Quality Standard](https://www.iso.org/standard/50798.html)
- 🎥 [Great Expectations Tutorial Series](https://www.youtube.com/greatexpectations)
- 📄 [AWS Deequ GitHub](https://github.com/awslabs/deequ)
