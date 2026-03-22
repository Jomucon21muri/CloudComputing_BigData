# CAPÍTULO 2: Fuentes oficiales y médicas de datos

## 2.1. Datos abiertos en España: Datos.gob.es

**URL**: https://datos.gob.es

**Características:**

- Portal centralizado del gobierno español
- Miles de datasets de administraciones públicas
- Formatos: CSV, JSON, XML, RDF
- Licencias abiertas
- API disponible

**Categorías principales:**

- Sector Público
- Salud
- Medio Ambiente
- Economía
- Transporte
- Educación

**Acceso programático:**

```python
import requests
import pandas as pd

# API de datos.gob.es
BASE_URL = "https://datos.gob.es/apidata"

def search_datasets(query, limit=10):
    """Buscar datasets en datos.gob.es"""
    url = f"{BASE_URL}/catalog/dataset"
    params = {
        'q': query,
        '_pageSize': limit
    }
    response = requests.get(url, params=params)
    return response.json()

# Ejemplo: Buscar datos de COVID
covid_datasets = search_datasets("COVID-19", limit=20)

# Descargar un dataset específico
def download_dataset(url):
    """Descargar CSV desde URL"""
    df = pd.read_csv(url, encoding='utf-8', sep=';')
    return df

# Ejemplo
url_covid = "https://datos.gob.es/es/catalogo/...casos_diarios.csv"
df_casos = download_dataset(url_covid)
```

**Datasets sanitarios relevantes:**

**Disponibles en datos.gob.es:**

- Casos COVID-19 por CCAA
- Farmacias y centros de salud
- Listas de espera sanitarias
- Gasto farmacéutico
- Vacunación

## 2.2. Instituto Nacional de Estadística (INE)

**URL**: https://www.ine.es

**Principales áreas:**

**Demografía y población:**

```python
# Ejemplo: Acceder a datos del INE mediante API

import requests
import json

# API del INE
INE_API = "https://servicios.ine.es/wstempus/js/ES"

def get_ine_data(operation_code):
    """
    Obtener datos del INE
    Ejemplo: operation_code = "IPC" para Índice de Precios
    """
    url = f"{INE_API}/DATOS_TABLA/{operation_code}"
    response = requests.get(url)
    return response.json()

# Ejemplo: Población por provincias
poblacion = get_ine_data("POBLACION_PROVINCIAS")
```

**Indicadores sanitarios:**

| Indicador | Código | Descripción |
|-----------|--------|-------------|
| **Esperanza de vida** | EVN | Por sexo y CCAA |
| **Tasas de mortalidad** | TMG | Causas de defunción |
| **Natalidad** | NAC | Nacimientos por año |
| **Encuesta de Salud** | ENS | Hábitos y salud |

**Estructura de datos del INE:**

```python
import pandas as pd

# Cargar datos INE (formato típico)
df_ine = pd.read_csv("ine_esperanza_vida.csv", sep=";", encoding='latin1')

print(df_ine.columns)
# Output típico:
# ['Comunidades y Ciudades Autónomas', 'Sexo', 'Periodo', 'Total']

# Limpieza típica necesaria
df_ine = df_ine.replace('.', pd.NA)  # El INE usa '.' para valores nulos
df_ine['Total'] = df_ine['Total'].str.replace(',', '.').astype(float)

# Pivotar para análisis
df_pivot = df_ine.pivot_table(
    values='Total',
    index='Comunidades y Ciudades Autónomas',
    columns='Sexo'
)
```

## 2.3. Ministerio de Sanidad

**URL**: https://www.sanidad.gob.es

**Sistemas de información sanitaria:**

**1. CMBD (Conjunto Mínimo Básico de Datos):**

**Descripción**: Registros de altas hospitalarias

**Contenido:**

- Diagnóstico principal y secundarios (CIE-10)
- Procedimientos quirúrgicos
- Datos demográficos del paciente
- Tiempos de estancia
- Circunstancias del alta

**Formato típico:**
```python
# Estructura de registro CMBD
cmbd_record = {
    'hospital': 'H001',
    'fecha_ingreso': '2026-01-15',
    'fecha_alta': '2026-01-20',
    'edad': 65,
    'sexo': 'H',
    'provincia': '28',  # Madrid
    'diagnostico_principal': 'I21.0',  # Infarto agudo de miocardio
    'diagnosticos_secundarios': ['I10', 'E11'],  # HTA, Diabetes
    'procedimientos': ['36.06'],  # Angioplastia coronaria
    'tipo_alta': '1',  # Domicilio
    'estancia': 5
}
```

**2. SIVIES (Sistema de Vigilancia Epidemiológica):**

- Enfermedades de declaración obligatoria
- Brotes epidemicos
- Alertas sanitarias

**3. Registros de Vacunación:**

- Cobertura vacunal por CCAA
- Campañas de vacunación
- Efectividad vacunal

**Acceso a datos del Ministerio:**

```python
# Ejemplo: Datos COVID-19 del Ministerio de Sanidad

url_mscbs = "https://cnecovid.isciii.es/covid19/resources/casos_hosp_uci_def_sexo_edad_provres.csv"

df_covid = pd.read_csv(url_mscbs, encoding='utf-8')

print(df_covid.head())
# Columnas típicas:
# provincia_iso, sexo, grupo_edad, fecha,
# casos_confirmados, hospitalizados, ingresos_uci, fallecidos
```

## Organización Mundial de la Salud (OMS/WHO)

**URL**: https://www.who.int/data

**WHO Global Health Observatory (GHO):**

**API de la OMS:**
```python
import requests

# API OMS
WHO_API = "https://ghoapi.azureedge.net/api"

def get_who_indicator(indicator_code, country=None):
    """
    Obtener indicador de salud global
    
    Ejemplos de códigos:
    - WHOSIS_000001: Esperanza de vida
    - MDG_0000000001: Mortalidad infantil
    - NCD_BMI_30A: Prevalencia de obesidad
    """
    url = f"{WHO_API}/{indicator_code}"
    if country:
        url += f"?$filter=SpatialDim eq '{country}'"
    
    response = requests.get(url)
    return response.json()

# Ejemplo: Esperanza de vida en España
esp_life_exp = get_who_indicator("WHOSIS_000001", country="ESP")

# Procesar resultados
import pandas as pd

values = esp_life_exp['value']
df_who = pd.DataFrame(values)

# Columnas típicas:
# - SpatialDim (país)
# - TimeDim (año)
# - NumericValue (valor del indicador)
# - Dim1 (desagregación, ej: sexo)
```

**Datasets clave de la OMS:**

| Dataset | Descripción | Actualización |
|---------|-------------|---------------|
| **GHS (Global Health Estimates)** | Causas de muerte y DALYs | Anual |
| **GHED (Health Expenditure)** | Gasto sanitario por país | Anual |
| **Vaccine Coverage** | Cobertura vacunal mundial | Mensual |
| **Disease Outbreaks** | Brotes epidémicos | Tiempo real |
| **Air Pollution** | Calidad del aire | Continuo |

## Problemas comunes con datos oficiales

**1. Formatos heterogéneos:**

```python
# Problema: Diferentes formatos para el mismo tipo de dato

# INE usa separador ; y decimal ,
df_ine = pd.read_csv("ine.csv", sep=";", decimal=",")

# Datos.gob.es usa separador , y decimal .
df_gob = pd.read_csv("gob.csv", sep=",", decimal=".")

# Ministerio usa codificaciones mixtas
df_mscbs = pd.read_csv("mscbs.csv", encoding='latin1')

# Solución: Funciones de normalización
def normalize_df(df, source='ine'):
    """Normalizar dataframes de diferentes fuentes"""
    if source == 'ine':
        # Convertir decimal , a .
        for col in df.select_dtypes(include=['object']):
            df[col] = df[col].str.replace(',', '.')
            df[col] = pd.to_numeric(df[col], errors='ignore')
    
    # Normalizar nombres de columnas
    df.columns = df.columns.str.lower().str.replace(' ', '_')
    
    return df
```

**2. Codificaciones territoriales:**

```python
# Problema: Múltiples sistemas de códigos

# INE: Código numérico de provincia (01-52)
codigo_ine = "28"  # Madrid

# ISO 3166-2: ES-M
codigo_iso = "ES-M"  # Madrid

# Nombre completo: "Comunidad de Madrid"

# Solución: Tabla de equivalencias
territorios = {
    'codigo_ine': ['28', '08', '41'],
    'codigo_iso': ['ES-M', 'ES-CT', 'ES-AN'],
    'nombre': ['Madrid', 'Cataluña', 'Andalucía'],
    'nombre_completo': [
        'Comunidad de Madrid',
        'Comunidad Autónoma de Cataluña',
        'Comunidad Autónoma de Andalucía'
    ]
}

df_territorios = pd.DataFrame(territorios)

# Usar para joins
df_merged = df_datos.merge(
    df_territorios,
    left_on='provincia',
    right_on='codigo_ine'
)
```

**3. Granularidad temporal inconsistente:**

```python
# Problema: Diferentes frecuencias de actualización

# Diaria: Casos COVID
df_daily = pd.read_csv("casos_diarios.csv")
df_daily['fecha'] = pd.to_datetime(df_daily['fecha'])

# Mensual: Gasto farmacéutico
df_monthly = pd.read_csv("gasto_mensual.csv")
df_monthly['mes'] = pd.to_datetime(df_monthly['mes'], format='%Y-%m')

# Anual: Población
df_yearly = pd.read_csv("poblacion_anual.csv")
df_yearly['año'] = pd.to_datetime(df_yearly['año'], format='%Y')

# Solución: Agregación temporal
def aggregate_to_monthly(df_daily, date_col, value_col):
    """Agregar datos diarios a mensuales"""
    df = df_daily.copy()
    df['mes'] = df[date_col].dt.to_period('M')
    df_monthly = df.groupby('mes')[value_col].sum().reset_index()
    return df_monthly
```

**4. Valores nulos y códigos especiales:**

```python
# Diferentes representaciones de nulo:
# - INE: "."
# - Otros: "..", "N/D", ":", "-"

def clean_nulls(df):
    """Limpiar diferentes representaciones de nulos"""
    null_values = ['.', '..', ':', '-', 'N/D', 'n.d.']
    
    for col in df.select_dtypes(include=['object']):
        df[col] = df[col].replace(null_values, pd.NA)
    
    return df
```

## Protección de datos sanitarios

**Marco legal:**

**RGPD (Reglamento General de Protección de Datos)**

- Datos de salud: Categoría especial (Art. 9)
- Requiere consentimiento explícito
- Derechos de los interesados
- Obligaciones del responsable

**LOPD-GDD (Ley Orgánica de Protección de Datos)**

- Adaptación española del RGPD
- Especificidades para datos de salud

**Técnicas de anonimización:**

**1. Supresión de identificadores:**

```python
def remove_identifiers(df):
    """Eliminar identificadores directos"""
    identifiers = ['dni', 'nombre', 'apellidos', 'num_ss', 'telefono']
    return df.drop(columns=identifiers, errors='ignore')
```

**2. Generalización:**

```python
def generalize_age(age):
    """Agrupar edad en rangos"""
    if age < 18:
        return '0-17'
    elif age < 30:
        return '18-29'
    elif age < 45:
        return '30-44'
    elif age < 65:
        return '45-64'
    else:
        return '65+'

df['grupo_edad'] = df['edad'].apply(generalize_age)
df = df.drop('edad', axis=1)
```

**3. Agregación:**

```python
# En lugar de registros individuales, publicar agregados
df_agregado = df.groupby(['provincia', 'grupo_edad', 'diagnostico']).agg({
    'casos': 'count'
}).reset_index()

# Aplicar k-anonimato: Mínimo k individuos por grupo
K_THRESHOLD = 5

df_filtered = df_agregado[df_agregado['casos'] >= K_THRESHOLD]
```

**4. Perturbación de datos (Differential Privacy):**

```python
import numpy as np

def add_noise(value, epsilon=0.1):
    """Añadir ruido laplaciano (Differential Privacy)"""
    noise = np.random.laplace(0, 1/epsilon)
    return value + noise

# Aplicar a columnas numéricas sensibles
df['estancia_hospital'] = df['estancia_hospital'].apply(
    lambda x: add_noise(x, epsilon=0.1)
)
```

**Niveles de identificación:**

| Nivel | Definición | Ejemplo | Requisitos |
|-------|------------|---------|------------|
| **Identificado** | Datos personales directos | Nombre + DNI + Dirección | Consentimiento explícito |
| **Pseudonimizado** | Identificador sustituido | ID_PACIENTE_12345 | Clave guardada separadamente |
| **Anonimizado** | No re-identificable | Agregados, generalizados | Uso libre (investigación) |


**Ejercicios propuestos:**

**Ejercicio 1: Análisis de Paradigmas**

Para cada uno de los siguientes escenarios, indica qué paradigma sería más apropiado y justifica:

1. Una startup quiere lanzar una app móvil con demanda impredecible
2. Un consorcio de 10 universidades europeas quiere analizar datos del CERN
3. Un investigador independiente quiere buscar patrones en señales astronómicas
4. Un hospital necesita procesar imágenes médicas con datos sensibles
5. Una empresa Fortune 500 necesita capacidad de cómputo para cierre de fin de año

**Ejercicio 2: Cálculo de Coste-Beneficio**

Compara el coste de:

- Un cluster on-premise de 100 nodos (calcular CAPEX + OPEX 3 años)
- Equivalente en AWS EC2 (calcular coste mensual y 3 años)
- ¿Cuándo es más rentable cada opción?

**Ejercicio 3: Arquitectura Híbrida**

Diseña una arquitectura cloud + computación voluntaria para:

- Procesamiento de datos genómicos
- Parte crítica en cloud (bajo sensibilidad)
- Análisis estadístico en BOINC


