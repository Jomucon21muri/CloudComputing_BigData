# CAPÍTULO 1: Economía y gobierno del dato

## 1.1. El dato como activo estratégico

**Definición**: Los datos son el nuevo petróleo del siglo XXI. Representan valor económico, ventaja competitiva y base para la toma de decisiones.

**Evolución del concepto:**
```
Años 80-90: Dato como subproducto
     ↓
Años 2000: Dato como recurso
     ↓
Años 2010: Dato como activo
     ↓
2020+: Dato como producto y servicio
```

**Características del dato como activo:**

| Característica | Descripción | Ejemplo |
|----------------|-------------|---------|
| **Reutilizable** | No se consume al usarse | Datos demográficos usados múltiples veces |
| **Combinable** | Valor exponencial al cruzarse | Datos clínicos + datos geográficos |
| **Escalable** | Coste marginal bajo | Distribuir dataset no cuesta más |
| **Valorizable** | Genera ingresos directos/indirectos | Monetización de datos anonimizados |
| **Perecedero** | Pierde valor con el tiempo | Datos de stock en tiempo real |

**El ciclo de vida del dato:**

```
1. GENERACIÓN
   ↓
2. CAPTURA/RECOLECCIÓN
   ↓
3. ALMACENAMIENTO
   ↓
4. PROCESAMIENTO
   ↓
5. ANÁLISIS
   ↓
6. VISUALIZACIÓN
   ↓
7. DECISIÓN
   ↓
8. ARCHIVO/ELIMINACIÓN
```

**Tipos de valor del dato:**

**1. Valor operacional:**

* Mejora de procesos
* Reducción de costes
* Eficiencia operativa

**2. Valor estratégico:**

* Ventaja competitiva
* Nuevos modelos de negocio
* Innovación

**3. Valor monetario:**

* Venta de datos (anonimizados)
* Servicios basados en datos
* Data as a Service (DaaS)

**4. Valor social:**

* Datos abiertos públicos
* Investigación científica
* Transparencia gubernamental

## 1.2. Calidad del dato

**Framework DAMA (Data Management Association):**

**Dimensiones de calidad:**

**1. Exactitud (Accuracy):**  Grado en que el dato representa correctamente la realidad.

```python
# Ejemplo: Validación de rango de edad
def validate_age(age):
    if 0 <= age <= 120:
        return True  # Exacto
    else:
        return False  # Inexacto
```

**2. Completitud (Completeness):** Proporción de valores obligatorios presentes.

```python
# Ejemplo: Verificar campos completos
import pandas as pd

df = pd.read_csv("datos_pacientes.csv")

# Calcular completitud
completeness = (1 - df.isnull().sum() / len(df)) * 100
print(f"Completitud por columna:\n{completeness}")

# Output esperado:
# nombre: 100%
# edad: 98%
# teléfono: 75%  ← Problema de calidad
```

**3. Consistencia (Consistency):** Ausencia de contradicciones entre datos relacionados.

```python
# Ejemplo: Verificar consistencia
def check_consistency(fecha_nacimiento, edad):
    from datetime import datetime
    edad_calculada = datetime.now().year - fecha_nacimiento.year
    if abs(edad_calculada - edad) <= 1:
        return True  # Consistente
    else:
        return False  # Inconsistente
```

**4. Oportunidad (Timeliness):** Los datos están disponibles cuando se necesitan.

| Tipo de Análisis | Requisito Temporal | Ejemplo |
|------------------|-------------------|---------|
| **Tiempo Real** | < 1 segundo | Detección de fraude bancario |
| **Near Real-Time** | < 5 minutos | Monitorización de pacientes UCI |
| **Batch** | Horas/Días | Informes mensuales |
| **Histórico** | Semanas/Meses | Análisis de tendencias anuales |

**5. Validez (Validity):** Los datos cumplen con reglas de negocio y formatos.

```python
import re

# Validación de DNI español
def validate_dni(dni):
    pattern = r'^\d{8}[A-Z]$'
    return bool(re.match(pattern, dni))

# Validación de email
def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

**6. Unicidad (Uniqueness):** Sin registros duplicados.

```python
# Detectar duplicados
df_pacientes = pd.read_csv("pacientes.csv")

# Duplicados completos
duplicados_completos = df_pacientes.duplicated()

# Duplicados por DNI
duplicados_dni = df_pacientes.duplicated(subset=['dni'], keep='first')

print(f"Registros duplicados: {duplicados_dni.sum()}")
```

**Problemas comunes de calidad:**

**En datos médicos:**

❌ Falta de estándares (nomenclaturas distintas)
❌ Datos incompletos (campos opcionales no rellenados)
❌ Errores de transcripción
❌ Unidades de medida inconsistentes
❌ Codificaciones múltiples (ICD-9, ICD-10, CIE-10)

**En datos abiertos:**

❌ Formatos heterogéneos (CSV, XML, JSON, Excel)
❌ Desactualizados
❌ Metadatos insuficientes
❌ Calidad variable entre organismos
❌ Falta de APIs, solo descargas manuales


**Mejora de la calidad: Data Profiling**

```python
import pandas as pd
from ydata_profiling import ProfileReport

# Cargar datos
df = pd.read_csv("datos_sanitarios.csv")

# Generar informe de calidad
profile = ProfileReport(df, title="Análisis de Calidad de Datos")
profile.to_file("informe_calidad.html")

# Métricas clave:
# - % valores nulos
# - Tipos de datos
# - Distribuciones
# - Correlaciones
# - Duplicados
# - Cardinalidad
```

**Framework de evaluación de calidad:**

```python
class DataQualityAssessment:
    def __init__(self, dataframe):
        self.df = dataframe
        
    def assess_completeness(self):
        """Porcentaje de valores no nulos"""
        return (1 - self.df.isnull().sum() / len(self.df)) * 100
    
    def assess_uniqueness(self):
        """Porcentaje de duplicados"""
        return (self.df.duplicated().sum() / len(self.df)) * 100
    
    def assess_validity(self, column, validation_func):
        """Aplica función de validación"""
        valid_count = self.df[column].apply(validation_func).sum()
        return (valid_count / len(self.df)) * 100
    
    def generate_report(self):
        """Informe completo de calidad"""
        return {
            'completeness': self.assess_completeness(),
            'uniqueness': self.assess_uniqueness(),
            'row_count': len(self.df),
            'column_count': len(self.df.columns)
        }

# Uso
dqa = DataQualityAssessment(df)
report = dqa.generate_report()
```

## Gobierno del dato

**Data Governance:** Marco organizacional que define quién puede tomar qué acciones, con qué datos, en qué situaciones, usando qué métodos.

**Componentes del gobierno del dato:**

```
               ┌─────────────────────────────────────┐
               │   ESTRATEGIA Y LIDERAZGO            │
               │   (Data Governance Council)         │
               └────────────────┬────────────────────┘
                                ↓
               ┌─────────────────────────────────────┐
               │   POLÍTICAS Y ESTÁNDARES            │
               │   - Calidad                         │
               │   - Seguridad                       │
               │   - Privacidad                      │
               │   - Retención                       │
               └────────────────┬────────────────────┘
                                ↓
               ┌─────────────────────────────────────┐
               │   PROCESOS Y CONTROLES              │
               │   - Gestión de Metadatos            │
               │   - Linaje del Dato                 │
               │   - Catálogo de Datos               │
               └────────────────┬────────────────────┘
                                ↓
               ┌─────────────────────────────────────┐
               │   TECNOLOGÍA Y HERRAMIENTAS         │
               │   - Catálogos                       │
               │   - MDM (Master Data Management)    │
               │   - DQ Tools                        │
               └─────────────────────────────────────┘
```

**Catálogo de datos:**

**Herramientas:**

- **AWS Glue Data Catalog**
- **Azure Purview**
- **Google Data Catalog**
- **Alation**
- **Collibra**

**Ejemplo de metadatos en catálogo:**

```json
{
  "dataset_name": "casos_covid_ccaa",
  "description": "Casos de COVID-19 por Comunidad Autónoma",
  "owner": "Ministerio de Sanidad",
  "steward": "juan.perez@sanidad.gob.es",
  "source": "https://datos.gob.es/...",
  "update_frequency": "Diario",
  "last_update": "2026-02-19",
  "format": "CSV",
  "size_mb": 25.3,
  "rows": 150000,
  "schema": [
    {"column": "fecha", "type": "date", "nullable": false},
    {"column": "ccaa", "type": "string", "nullable": false},
    {"column": "casos_nuevos", "type": "integer", "nullable": false},
    {"column": "hospitalizados", "type": "integer", "nullable": true}
  ],
  "quality_score": 92,
  "classification": "Público",
  "tags": ["salud", "epidemiología", "territorial"],
  "lineage": "Sistema SIVIES → ETL → Data Lake"
}
```

**Linaje del dato (Data Lineage):**

Trazabilidad del ciclo de vida del dato desde su origen hasta su destino final.

```
Sistema Origen (Hospital A)
         ↓
   Sistema SIVIES
         ↓
   ETL de Validación
         ↓
   Data Lake (Raw Zone)
         ↓
   Transformación
         ↓
   Data Lake (Refined Zone)
         ↓
   Data Warehouse OLAP
         ↓
   Dashboard Power BI
```

**Importancia:**

✅ Auditoría y cumplimiento
✅ Análisis de impacto de cambios
✅ Resolución de problemas de calidad
✅ Transparencia

---