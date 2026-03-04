# CAPÍTULO 9: METODOLOGÍAS AVANZADAS DE CIENCIA DE DATOS

## Introducción

!!! abstract "Más allá de CRISP-DM"
    Mientras CRISP-DM es la metodología más popular y establecida, existen otras metodologías especializadas que abordan diferentes aspectos de proyectos de ciencia de datos, desde perspectivas corporativas (IBM, Microsoft) hasta enfoques ágiles y de ingeniería.

---

## 9.1. KDD (Knowledge Discovery in Databases)

!!! info "Origen Histórico"
    **KDD** es uno de los procesos **más antiguos** de descubrimiento de conocimiento en bases de datos, previo a CRISP-DM. Desarrollado en la década de 1990, establece los fundamentos conceptuales de la minería de datos moderna.

**Definición:**

**Knowledge Discovery in Databases (KDD)** es el proceso completo de descubrir conocimiento útil desde datos, incluyendo:

- Preparación de datos
- Selección de patrones
- Limpieza y transformación
- Aplicación de algoritmos de minería
- Interpretación y evaluación de resultados

**Proceso KDD: 5 etapas:**

```mermaid
graph LR
    A[1️⃣ Selección] --> B[2️⃣ Preprocesamiento]
    B --> C[3️⃣ Transformación]
    C --> D[4️⃣ Data Mining]
    D --> E[5️⃣ Interpretación]
    E -.Iteración.-> A
    
    style A fill:#e3f2fd
    style B fill:#bbdefb
    style C fill:#90caf9
    style D fill:#64b5f6
    style E fill:#42a5f5
```

#### **Etapa 1: Selección (Selection)**

**Objetivo:** Seleccionar el conjunto de datos objetivo desde la base de datos

**Actividades:**
- Identificar fuentes de datos relevantes
- Extraer subconjuntos de datos según criterios del dominio
- Definir el dataset target para análisis

**Salida:** Dataset inicial seleccionado

```python
# Ejemplo: Selección de datos KDD

import pandas as pd
from sqlalchemy import create_engine

# Conectar a base de datos fuente
engine = create_engine('postgresql://user:pass@host:5432/db')

# SELECCIÓN: Extraer datos relevantes para el problema
query = """
SELECT 
    cliente_id,
    fecha_transaccion,
    monto,
    categoria_producto,
    edad_cliente,
    ciudad
FROM 
    transacciones t
    JOIN clientes c ON t.cliente_id = c.id
WHERE 
    fecha_transaccion >= '2023-01-01'
    AND monto > 0
    AND categoria_producto IN ('Electrónica', 'Hogar', 'Ropa')
"""

# Dataset target seleccionado
df_selected = pd.read_sql(query, engine)

print(f"📊 Datos seleccionados: {len(df_selected):,} registros")
print(f"   Periodo: {df_selected['fecha_transaccion'].min()} a {df_selected['fecha_transaccion'].max()}")
print(f"   Categorías: {df_selected['categoria_producto'].unique()}")

# Output:
# 📊 Datos seleccionados: 850,000 registros
#    Periodo: 2023-01-01 a 2024-12-31
#    Categorías: ['Electrónica' 'Hogar' 'Ropa']
```

---

#### **Etapa 2: Preprocesamiento (Preprocessing)**

**Objetivo:** Limpiar y preparar los datos para análisis

**Actividades:**
- Eliminar ruido y outliers
- Manejar valores faltantes
- Resolver inconsistencias
- Eliminar duplicados

**Salida:** Dataset limpio

```python
# PREPROCESAMIENTO: Limpieza de datos

import numpy as np
from sklearn.impute import SimpleImputer

print("=== PREPROCESAMIENTO KDD ===\n")

# 1. Identificar problemas de calidad
print("1️⃣ Análisis de Calidad:")
print(f"   Valores faltantes:\n{df_selected.isnull().sum()}")
print(f"   Duplicados: {df_selected.duplicated().sum()}")

# 2. Eliminar duplicados
df_clean = df_selected.drop_duplicates()

# 3. Manejar valores faltantes
imputer = SimpleImputer(strategy='median')
df_clean['edad_cliente'] = imputer.fit_transform(
    df_clean[['edad_cliente']]
)

# 4. Eliminar outliers (IQR method)
Q1 = df_clean['monto'].quantile(0.25)
Q3 = df_clean['monto'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

df_clean = df_clean[
    (df_clean['monto'] >= lower_bound) & 
    (df_clean['monto'] <= upper_bound)
]

# 5. Resolver inconsistencias
df_clean['ciudad'] = df_clean['ciudad'].str.strip().str.title()

print(f"\n✅ Preprocesamiento completo:")
print(f"   Registros originales: {len(df_selected):,}")
print(f"   Registros limpios: {len(df_clean):,}")
print(f"   Registros eliminados: {len(df_selected) - len(df_clean):,} ({(1 - len(df_clean)/len(df_selected))*100:.1f}%)")

# Output:
# ✅ Preprocesamiento completo:
#    Registros originales: 850,000
#    Registros limpios: 812,350
#    Registros eliminados: 37,650 (4.4%)
```

---

#### **Etapa 3: Transformación (Transformation)**

**Objetivo:** Transformar datos en formato apropiado para minería

**Actividades:**
- Reducción de dimensionalidad (PCA, t-SNE)
- Normalización y estandarización
- Feature engineering
- Discretización de variables continuas

**Salida:** Dataset transformado

```python
# TRANSFORMACIÓN: Preparar datos para algoritmos

from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.decomposition import PCA

print("=== TRANSFORMACIÓN KDD ===\n")

# 1. Feature Engineering
df_clean['mes'] = pd.to_datetime(df_clean['fecha_transaccion']).dt.month
df_clean['dia_semana'] = pd.to_datetime(df_clean['fecha_transaccion']).dt.dayofweek
df_clean['es_fin_semana'] = (df_clean['dia_semana'] >= 5).astype(int)

# Agregar features por cliente
cliente_stats = df_clean.groupby('cliente_id').agg({
    'monto': ['mean', 'sum', 'count'],
    'fecha_transaccion': 'max'
}).reset_index()
cliente_stats.columns = ['cliente_id', 'monto_promedio', 'monto_total', 
                         'num_transacciones', 'ultima_transaccion']

df_transformed = df_clean.merge(cliente_stats, on='cliente_id', how='left')

# 2. Encoding de categóricas
le = LabelEncoder()
df_transformed['categoria_encoded'] = le.fit_transform(df_transformed['categoria_producto'])

# 3. Normalización de variables numéricas
scaler = StandardScaler()
numeric_cols = ['monto', 'edad_cliente', 'monto_promedio', 'num_transacciones']
df_transformed[numeric_cols] = scaler.fit_transform(df_transformed[numeric_cols])

# 4. Reducción de dimensionalidad (opcional)
# Si tenemos muchas features, aplicar PCA
numeric_features = df_transformed[numeric_cols]
pca = PCA(n_components=0.95)  # Retener 95% de varianza
principal_components = pca.fit_transform(numeric_features)

print(f"✅ Transformación completa:")
print(f"   Features originales: {len(df_clean.columns)}")
print(f"   Features transformadas: {len(df_transformed.columns)}")
print(f"   PCA componentes: {pca.n_components_} (de {len(numeric_cols)} originales)")
print(f"   Varianza explicada: {pca.explained_variance_ratio_.sum():.2%}")

# Output:
# ✅ Transformación completa:
#    Features originales: 6
#    Features transformadas: 13
#    PCA componentes: 3 (de 4 originales)
#    Varianza explicada: 95.23%
```

---

#### **Etapa 4: Data Mining**

**Objetivo:** Aplicar algoritmos de minería para descubrir patrones

**Actividades:**
- Seleccionar algoritmos apropiados (clasificación, clustering, asociación)
- Configurar parámetros
- Entrenar modelos
- Extraer patrones y reglas

**Salida:** Patrones, modelos, reglas descubiertas

```python
# DATA MINING: Aplicar algoritmos

from sklearn.cluster import KMeans
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import silhouette_score, classification_report

print("=== DATA MINING KDD ===\n")

# TAREA 1: Segmentación de clientes (Clustering)
print("1️⃣ CLUSTERING: Segmentación de clientes")

X_clustering = df_transformed[['monto_promedio', 'num_transacciones', 
                                'edad_cliente']].values

# Determinar número óptimo de clusters
silhouette_scores = []
K_range = range(2, 11)

for k in K_range:
    kmeans = KMeans(n_clusters=k, random_state=42)
    labels = kmeans.fit_predict(X_clustering)
    score = silhouette_score(X_clustering, labels)
    silhouette_scores.append(score)

optimal_k = K_range[np.argmax(silhouette_scores)]
print(f"   Número óptimo de clusters: {optimal_k}")

# Aplicar clustering
kmeans_final = KMeans(n_clusters=optimal_k, random_state=42)
df_transformed['cluster'] = kmeans_final.fit_predict(X_clustering)

# Perfilar clusters
print(f"\n   Perfil de Clusters:")
for cluster_id in range(optimal_k):
    cluster_data = df_transformed[df_transformed['cluster'] == cluster_id]
    print(f"\n   Cluster {cluster_id} ({len(cluster_data):,} clientes):")
    print(f"      Monto promedio: €{cluster_data['monto'].mean():.2f}")
    print(f"      Frecuencia compra: {cluster_data['num_transacciones'].mean():.1f}")
    print(f"      Edad promedio: {cluster_data['edad_cliente'].mean():.0f} años")

# TAREA 2: Predicción de categoría preferida (Clasificación)
print(f"\n\n2️⃣ CLASIFICACIÓN: Predecir categoría preferida")

X_class = df_transformed[['edad_cliente', 'monto_promedio', 'num_transacciones',
                          'es_fin_semana']].values
y_class = df_transformed['categoria_encoded'].values

from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X_class, y_class, test_size=0.3, random_state=42
)

rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)

y_pred = rf.predict(X_test)

print(f"\n   Accuracy: {rf.score(X_test, y_test):.2%}")
print(f"\n   Reporte de clasificación:")
print(classification_report(y_test, y_pred, 
                           target_names=le.classes_))

# Output ejemplo:
# ✅ DATA MINING KDD
# 1️⃣ CLUSTERING: Segmentación de clientes
#    Número óptimo de clusters: 4
#    
#    Cluster 0 (205,000 clientes):
#       Monto promedio: €45.30
#       Frecuencia compra: 15.2
#       Edad promedio: 35 años
#    ...
# 
# 2️⃣ CLASIFICACIÓN: Predecir categoría preferida
#    Accuracy: 78.50%
```

---

#### **Etapa 5: Interpretación y Evaluación (Interpretation)**

**Objetivo:** Interpretar patrones descubiertos y evaluar utilidad

**Actividades:**
- Interpretar patrones en contexto de negocio
- Validar con expertos del dominio
- Evaluar impacto y utilidad
- Decidir si iterar o desplegar

**Salida:** Conocimiento accionable validado

```python
# INTERPRETACIÓN: Convertir patrones en conocimiento accionable

print("=== INTERPRETACIÓN Y EVALUACIÓN KDD ===\n")

# 1. Interpretar clusters
interpretaciones = {
    0: {
        'nombre': 'Compradores Frecuentes',
        'caracteristicas': 'Alta frecuencia, bajo ticket, jóvenes',
        'estrategia': 'Programas de lealtad, descuentos por volumen'
    },
    1: {
        'nombre': 'Alto Valor',
        'caracteristicas': 'Baja frecuencia, alto ticket, adultos',
        'estrategia': 'Servicios premium, atención personalizada'
    },
    2: {
        'nombre': 'Ocasionales',
        'caracteristicas': 'Muy baja frecuencia, ticket medio',
        'estrategia': 'Campañas de reactivación, ofertas especiales'
    },
    3: {
        'nombre': 'Regulares',
        'caracteristicas': 'Frecuencia media, ticket medio, variedad',
        'estrategia': 'Cross-selling, recomendaciones personalizadas'
    }
}

print("📊 SEGMENTOS DE CLIENTES IDENTIFICADOS:\n")
for cluster_id, info in interpretaciones.items():
    count = (df_transformed['cluster'] == cluster_id).sum()
    pct = count / len(df_transformed) * 100
    print(f"🎯 {info['nombre']} (Cluster {cluster_id})")
    print(f"   Tamaño: {count:,} clientes ({pct:.1f}%)")
    print(f"   Características: {info['caracteristicas']}")
    print(f"   Estrategia recomendada: {info['estrategia']}\n")

# 2. Evaluar valor de negocio
print("💼 IMPACTO DE NEGOCIO ESTIMADO:\n")

# Calcular potencial de crecimiento por segmento
for cluster_id in range(optimal_k):
    cluster_data = df_transformed[df_transformed['cluster'] == cluster_id]
    
    monto_actual = cluster_data['monto'].sum()
    frecuencia_actual = cluster_data['num_transacciones'].mean()
    
    # Escenario: Incrementar frecuencia 20%
    incremento_frecuencia = 0.20
    monto_potencial = monto_actual * (1 + incremento_frecuencia)
    ganancia = monto_potencial - monto_actual
    
    print(f"   Cluster {cluster_id} - {interpretaciones[cluster_id]['nombre']}:")
    print(f"      Ingresos actuales: €{monto_actual:,.0f}")
    print(f"      Potencial (+20% frecuencia): €{monto_potencial:,.0f}")
    print(f"      Ganancia estimada: €{ganancia:,.0f}\n")

# 3. Validación con expertos
print("✅ VALIDACIÓN:\n")
print("   [ ] Patrones revisados por equipo de marketing")
print("   [ ] Estrategias validadas con gerencia comercial")
print("   [ ] Prueba piloto en Cluster 2 planificada para Q2")
print("   [ ] ROI proyectado: 250% en 12 meses")

# Output ejemplo:
# ===INTERPRETACIÓN Y EVALUACIÓN KDD ===
#
# 📊 SEGMENTOS DE CLIENTES IDENTIFICADOS:
#
# 🎯 Compradores Frecuentes (Cluster 0)
#    Tamaño: 205,000 clientes (25.2%)
#    Características: Alta frecuencia, bajo ticket, jóvenes
#    Estrategia recomendada: Programas de lealtad, descuentos por volumen
#
# 💼 IMPACTO DE NEGOCIO ESTIMADO:
#    Cluster 0 - Compradores Frecuentes:
#       Ingresos actuales: €9,282,500
#       Potencial (+20% frecuencia): €11,139,000
#       Ganancia estimada: €1,856,500
```

---

**KDD vs CRISP-DM: comparación:**

| Aspecto | KDD | CRISP-DM |
|---------|-----|----------|
| **Origen** | Investigación académica (1990s) | Industria (1999, SPSS/NCR/Daimler) |
| **Fases** | 5 etapas secuenciales | 6 fases iterativas |
| **Énfasis** | Proceso técnico de descubrimiento | Ciclo completo de negocio |
| **Iteración** | Implícita, menos estructurada | Explícita, con retroalimentación |
| **Comprensión negocio** | Limitada (parte de selección) | Fase completa dedicada |
| **Despliegue** | No enfatizado | Fase completa dedicada |
| **Uso actual** | Fundamentos académicos | Estándar industria |

!!! tip "¿Cuándo usar KDD?"
    - Proyectos de **investigación académica**
    - Cuando el énfasis está en **algoritmos y técnicas**
    - Proyectos donde el **problema está bien definido**
    - Como **base conceptual** antes de CRISP-DM

---

## 9.2. ASUM-DM (IBM Analytics Solutions Unified Method)

!!! info "Metodología Corporativa de IBM"
    **ASUM-DM** es la evolución de CRISP-DM desarrollada por **IBM** para proyectos de analytics empresariales, integrando mejores prácticas de IBM y enfoque en soluciones end-to-end.

**Fases de ASUM-DM:**

```mermaid
graph TD
    A[1. Análisis] --> B[2. Diseño]
    B --> C[3. Implementación]
    C --> D[4. Operar y Optimizar]
    D -.Mejora Continua.-> A
    
    style A fill:#054ADA
    style B fill:#1F70C1
    style C fill:#4589FF
    style D fill:#78A9FF
```

#### **Fase 1: Análisis**

**Objetivo:** Determinar requisitos y necesidades del cliente

**Actividades:**
- Análisis de necesidades del negocio
- Definición de objetivos del proyecto
- Identificación de stakeholders
- Definición de entregables esperados
- Evaluación de viabilidad

**Equivalente CRISP-DM:** Comprensión del Negocio

```python
# Ejemplo: Fase de Análisis ASUM-DM

proyecto_asum = {
    'nombre': 'Optimización de Cadena de Suministro con Analytics',
    'cliente': 'Empresa Retail Global',
    'sponsor': 'VP Operations',
    
    'necesidades_negocio': [
        'Reducir costos de inventario 15%',
        'Mejorar precisión pronóstico demanda 20%',
        'Reducir stockouts 30%',
        'Optimizar rutas de distribución'
    ],
    
    'stakeholders': {
        'primarios': ['VP Operations', 'Director Supply Chain', 'CIO'],
        'secundarios': ['Gerentes Almacén', 'Equipo Compras', 'Analistas Demanda'],
        'usuarios_finales': ['Planificadores', 'Compradores', 'Analistas']
    },
    
    'entregables': [
        'Modelo predictivo de demanda',
        'Dashboard ejecutivo tiemporeal',
        'Sistema alertas automático',
        'Recomendaciones de reabastecimiento',
        'Optimizador de rutas'
    ],
    
    'kpis_exito': {
        'reduccion_costo_inventario': '15%',
        'accuracy_forecast': '85%',
        'reduccion_stockouts': '30%',
        'roi': '300% en 18 meses'
    },
    
    'restricciones': {
        'presupuesto': '$500,000',
        'tiempo': '9 meses',
        'tecnologia': 'IBM Cloud Pak for Data, Watson Studio',
        'cumplimiento': 'SOC2, ISO 27001'
    }
}

print("=== FASE 1: ANÁLISIS (ASUM-DM) ===\n")
print(f"📋 Proyecto: {proyecto_asum['nombre']}")
print(f"🏢 Cliente: {proyecto_asum['cliente']}\n")

print("🎯 Necesidades de Negocio:")
for i, necesidad in enumerate(proyecto_asum['necesidades_negocio'], 1):
    print(f"   {i}. {necesidad}")

print(f"\n✅ Entregables Definidos: {len(proyecto_asum['entregables'])}")
print(f"💰 Presupuesto: {proyecto_asum['restricciones']['presupuesto']}")
print(f"⏱️  Timeline: {proyecto_asum['restricciones']['tiempo']}")
```

---

#### **Fase 2: Diseño**

**Objetivo:** Desarrollo de la minería y análisis de datos

**Actividades:**
- Exploración y análisis de datos
- Selección de técnicas y algoritmos
- Diseño de arquitectura de solución
- Prototipado de modelos
- Validación técnica

**Equivalente CRISP-DM:** Comprensión + Preparación de Datos + Modelado

```python
# Ejemplo: Fase de Diseño ASUM-DM

diseño_solucion = {
    'arquitectura': {
        'ingestion': 'IBM DataStage / Kafka',
        'storage': 'IBM Db2 Warehouse + IBM Cloud Object Storage',
        'processing': 'Apache Spark on IBM Cloud',
        'ml_platform': 'Watson Studio',
        'deployment': 'Watson Machine Learning',
        'visualization': 'Cognos Analytics'
    },
    
    'modelos_propuestos': [
        {
            'nombre': 'Predictor de Demanda',
            'algoritmo': 'LSTM (Deep Learning)',
            'features': ['ventas_historicas', 'estacionalidad', 'promociones',
                        'eventos', 'temperatura', 'feriados'],
            'output': 'Demanda diaria próximos 30 días',
            'accuracy_esperado': '87%'
        },
        {
            'nombre': 'Clasificador de Productos en Riesgo',
            'algoritmo': 'XGBoost',
            'features': ['velocidad_rotacion', 'dias_inventario', 
                        'variabilidad_demanda', 'lead_time_proveedor'],
            'output': 'Riesgo: Alto/Medio/Bajo',
            'precision_esperada': '82%'
        },
        {
            'nombre': 'Optimizador de Rutas',
            'algoritmo': 'Algoritmo Genético + OR',
            'input': 'Pedidos, ubicaciones, restricciones vehículos',
            'output': 'Rutas óptimas con secuencia paradas',
            'mejora_esperada': '22% reducción distancia'
        }
    ],
    
    'prototipo': {
        'dataset': 'Histórico 3 años (2021-2023)',
        'productos_piloto': 'Top 100 SKUs por volumen',
        'regiones': 'Madrid, Barcelona (2 hubs)',
        'validacion': 'Backtesting últimos 6 meses'
    }
}

print("\n=== FASE 2: DISEÑO (ASUM-DM) ===\n")

print("🏗️ Arquitectura de Solución:")
for componente, tecnologia in diseño_solucion['arquitectura'].items():
    print(f"   {componente:15s}: {tecnologia}")

print(f"\n🤖 Modelos Diseñados ({len(diseño_solucion['modelos_propuestos'])}):")
for modelo in diseño_solucion['modelos_propuestos']:
    print(f"\n   📊 {modelo['nombre']}")
    print(f"      Algoritmo: {modelo['algoritmo']}")
    print(f"      Features: {len(modelo['features'])} variables")
    if 'accuracy_esperado' in modelo:
        print(f"      Target accuracy: {modelo['accuracy_esperado']}")
```

---

#### **Fase 3: Implementación**

**Objetivo:** Implementar resultados en la empresa

**Actividades:**
- Desarrollo de solución completa
- Integración con sistemas existentes
- Pruebas de aceptación
- Capacitación de usuarios
- Documentación

**Equivalente CRISP-DM:** Evaluación + Despliegue

```python
# Ejemplo: Fase de Implementación ASUM-DM

plan_implementacion = {
    'sprint_1': {
        'duracion': '4 semanas',
        'tareas': [
            'Construir pipeline de datos producción',
            'Entrenar modelos con datos completos',
            'Desarrollar API REST para predicciones',
            'Integrar con ERP (SAP)',
            'Tests unitarios y de integración'
        ],
        'entregable': 'MVP funcional en staging'
    },
    
    'sprint_2': {
        'duracion': '3 semanas',
        'tareas': [
            'Desarrollar dashboard Cognos',
            'Sistema de alertas automático',
            'Documentación técnica',
            'Documentación usuario final',
            'UAT (User Acceptance Testing)'
        ],
        'entregable': 'Solución completa en pre-producción'
    },
    
    'sprint_3': {
        'duracion': '2 semanas',
        'tareas': [
            'Deployment a producción (canary)',
            'Capacitación usuarios (3 sesiones)',
            'Monitoreo intensivo',
            'Ajustes post-lanzamiento'
        ],
        'entregable': 'Sistema en producción operacional'
    },
    
    'capacitacion': {
        'sesion_1': 'Planificadores (uso dashboard, recomendaciones)',
        'sesion_2': 'Compradores (alertas reabastecimiento)',
        'sesion_3': 'Analistas (interpretación modelos, métricas)'
    }
}

print("\n=== FASE 3: IMPLEMENTACIÓN (ASUM-DM) ===\n")

for sprint, detalles in plan_implementacion.items():
    if 'sprint' in sprint:
        print(f"🚀 {sprint.upper()} ({detalles['duracion']}):")
        for tarea in detalles['tareas']:
            print(f"   ▪️ {tarea}")
        print(f"   ✅ Entregable: {detalles['entregable']}\n")
```

---

#### **Fase 4: Operar y Optimizar**

**Objetivo:** Control e introducción de ajustes continuos

**Actividades:**
- Monitoreo de performance
- Detección de data drift
- Reentrenamiento de modelos
- Optimización continua
- Gestión de cambios

**Única a ASUM-DM:** Énfasis en mejora continua operacional

```python
# Ejemplo: Fase Operar y Optimizar ASUM-DM

plan_operacion = {
    'monitoreo_diario': {
        'metricas_modelo': [
            'Accuracy predicciones vs real',
            'Latencia API (<200ms)',
            'Volumen predicciones/día',
            'Errores y excepciones'
        ],
        'metricas_negocio': [
            'Stockouts diarios',
            'Nivel inventario vs objetivo',
            'Costos almacenamiento',
            'Fill rate pedidos'
        ],
        'alertas': [
            'Accuracy cae >5% → reentrenar',
            'Latencia >500ms → escalar',
            'Stockouts aumentan 20% → revisar parámetros'
        ]
    },
    
    'reentrenamiento': {
        'frecuencia': 'Mensual (automático)',
        'trigger_manual': 'Data drift detectado',
        'datasets': 'Rolling window últimos 24 meses',
        'validacion': 'A/B test vs modelo actual'
    },
    
    'optimizaciones_trimestrales': [
        'Revisión features (agregar/quitar)',
        'Hyperparameter tuning',
        'Explorar nuevos algoritmos',
        'Feedback usuarios → mejoras'
    ],
    
    'mejora_continua': {
        'q1': 'Expandir a 50 SKUs adicionales',
        'q2': 'Integrar datos externos (clima API)',
        'q3': 'Modelo multi-región',
        'q4': 'Auto-ML para selección algoritmos'
    }
}

print("\n=== FASE 4: OPERAR Y OPTIMIZAR (ASUM-DM) ===\n")

print("📊 Monitoreo Diario:")
print("   Métricas Modelo:")
for metrica in plan_operacion['monitoreo_diario']['metricas_modelo']:
    print(f"      ▪️ {metrica}")

print("\n🔄 Reentrenamiento:")
print(f"   Frecuencia: {plan_operacion['reentrenamiento']['frecuencia']}")
print(f"   Trigger: {plan_operacion['reentrenamiento']['trigger_manual']}")

print("\n🎯 Mejora Continua (Roadmap):")
for trimestre, mejora in plan_operacion['mejora_continua'].items():
    print(f"   {trimestre.upper()}: {mejora}")
```

---

**ASUM-DM vs CRISP-DM: comparación:**

```mermaid
graph TD
    subgraph CRISP-DM
        C1[Comprensión Negocio] --> C2[Comprensión Datos]
        C2 --> C3[Preparación Datos]
        C3 --> C4[Modelado]
        C4 --> C5[Evaluación]
        C5 --> C6[Despliegue]
    end
    
    subgraph ASUM-DM
        A1[Análisis] --> A2[Diseño]
        A2 --> A3[Implementación]
        A3 --> A4[Operar y Optimizar]
    end
    
    C1 -.equivale.-> A1
    C2 -.equivale.-> A2
    C3 -.equivale.-> A2
    C4 -.equivale.-> A2
    C5 -.equivale.-> A3
    C6 -.equivale.-> A3
    A4 -.único.-> A4
```

| Aspecto | CRISP-DM | ASUM-DM |
|---------|----------|---------|
| **Origen** | Alianza SPSS/NCR/Daimler | IBM |
| **Fases** | 6 fases | 4 fases (más macro) |
| **Énfasis** | Proceso de minería | Solución analytics end-to-end |
| **Fase única** | - | **Operar y Optimizar** (mejora continua) |
| **Tecnología** | Agnóstico | Optimizado para stack IBM |
| **Gobernanza** | Implícita | Explícita (IBM framework) |
| **MLOps** | No incorporado | Integrado en Fase 4 |

!!! tip "¿Cuándo usar ASUM-DM?"
    - Proyectos en **ecosistema IBM** (Watson, Cloud Pak)
    - Empresas con **cultura DevOps/MLOps madura**
    - Cuando se requiere **mejora continua estructurada**
    - Proyectos analytics **complejos y enterprise**

---

## 9.3. TDSP (Team Data Science Process - Microsoft)

!!! info "Metodología de Microsoft"
    **TDSP** es la metodología ágil e iterativa de **Microsoft** para proyectos de data science en equipo, enfocada en colaboración, estándares y mejores prácticas corporativas.

**Componentes de TDSP:**

```mermaid
graph TB
    A[1️⃣ Ciclo de Vida] --> E[TDSP]
    B[2️⃣ Estructura Proyecto] --> E
    C[3️⃣ Infraestructura] --> E
    D[4️⃣ Herramientas] --> E
    
    E --> F[Éxito Proyecto DS]
    
    style E fill:#00BCF2
    style F fill:#FFB900
```

---

**1. Ciclo de vida de TDSP:**

**5 Etapas iterativas:**

```mermaid
graph LR
    A[Business Understanding] --> B[Data Acquisition]
    B --> C[Modeling]
    C --> D[Deployment]
    D --> E[Customer Acceptance]
    E -.Feedback.-> A
    
    style A fill:#00BCF2
    style B fill:#0078D4
    style C fill:#50E6FF
    style D fill:#FFB900
    style E fill:#00CC6A
```

#### **Etapa 1: Business Understanding (Comprensión del Negocio)**

- Definir objetivos
- Identificar fuentes de datos
- Crear plan del proyecto

#### **Etapa 2: Data Acquisition & Understanding (Adquisición y Comprensión de Datos)**

- Ingerir datos en ambiente analítico
- Explorar calidad y adecuación
- Crear diccionario de datos

#### **Etapa 3: Modeling (Modelado)**

- Feature engineering
- Entrenar modelos
- Evaluar performance

#### **Etapa 4: Deployment (Despliegue)**

- Operacionalizar modelos
- Integrar en pipelines
- Setup monitoreo

#### **Etapa 5: Customer Acceptance (Aceptación del Cliente)**

- Validar con stakeholders
- Entregar documentación
- Transferir conocimiento

---

**2. Estructura de proyecto estandarizada:**

**TDSP proporciona una estructura de carpetas estándar:**

```
project/
│
├── docs/                      # Documentación
│   ├── project-charter.md
│   ├── data-dictionary.md
│   └── model-report.md
│
├── data/
│   ├── raw/                   # Datos crudos (inmutables)
│   ├── processed/             # Datos procesados
│   └── interim/               # Datos intermedios
│
├── notebooks/                 # Jupyter notebooks exploratorios
│   ├── 01-data-exploration.ipynb
│   ├── 02-feature-engineering.ipynb
│   └── 03-model-training.ipynb
│
├── src/                       # Código fuente
│   ├── data/
│   │   ├── make_dataset.py
│   │   └── clean_data.py
│   ├── features/
│   │   └── build_features.py
│   ├── models/
│   │   ├── train_model.py
│   │   └── predict_model.py
│   └── visualization/
│       └── visualize.py
│
├── models/                    # Modelos entrenados
│   ├── model_v1.pkl
│   └── model_v2.pkl
│
├── reports/                   # Reportes y figuras
│   └── figures/
│
├── requirements.txt           # Dependencias Python
├── environment.yml            # Conda environment
└── README.md                  # Overview del proyecto
```

**Ejemplo de código siguiendo TDSP:**

```python
# src/data/make_dataset.py - TDSP Structure

import pandas as pd
from azure.storage.blob import BlobServiceClient
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class DataLoader:
    """
    Carga datos desde Azure Blob Storage siguiendo estructura TDSP
    """
    
    def __init__(self, connection_string, container_name):
        self.blob_service = BlobServiceClient.from_connection_string(connection_string)
        self.container = container_name
    
    def load_raw_data(self, blob_name, output_path='data/raw/'):
        """
        Descarga datos crudos desde blob storage
        
        Args:
            blob_name: Nombre del blob en Azure
            output_path: Ruta local para guardar (dentro de data/raw/)
        
        Returns:
            DataFrame con datos cargados
        """
        logger.info(f"Descargando {blob_name} desde Azure Blob Storage...")
        
        blob_client = self.blob_service.get_blob_client(
            container=self.container, 
            blob=blob_name
        )
        
        # Descargar a data/raw/ (inmutable)
        local_path = output_path + blob_name
        with open(local_path, "wb") as download_file:
            download_file.write(blob_client.download_blob().readall())
        
        logger.info(f"✅ Datos descargados a {local_path}")
        
        # Cargar a DataFrame
        df = pd.read_csv(local_path)
        logger.info(f"📊 Shape: {df.shape}")
        
        return df
    
    def save_processed_data(self, df, filename):
        """
        Guarda datos procesados en data/processed/
        """
        output_path = f'data/processed/{filename}'
        df.to_parquet(output_path, index=False)
        logger.info(f"💾 Datos procesados guardados en {output_path}")

# Uso
if __name__ == "__main__":
    # Configuración desde variables de entorno
    import os
    
    connection_string = os.getenv('AZURE_STORAGE_CONNECTION_STRING')
    loader = DataLoader(connection_string, 'datos-proyecto')
    
    # Cargar datos crudos
    df_raw = loader.load_raw_data('clientes_2024.csv')
    
    # Procesar (siguiente script lo haría)
    # Ver src/data/clean_data.py
```

---

**3. Infraestructura y recursos:**

TDSP recomienda infraestructura compartida del equipo:

| Componente | Recomendación Microsoft | Alternativa Open Source |
|------------|------------------------|------------------------|
| **File Storage** | Azure Data Lake Storage | HDFS, S3 |
| **Databases** | Azure SQL Database | PostgreSQL, MySQL |
| **Big Data Clusters** | Azure HDInsight Spark | Apache Spark on-prem |
| **ML Service** | Azure Machine Learning | MLflow + Kubernetes |
| **BI/Viz** | Power BI | Tableau, Metabase |
| **Version Control** | Azure DevOps | GitHub, GitLab |
| **Container Registry** | Azure Container Registry | Docker Hub, Harbor |
| **Orchestration** | Azure Data Factory | Apache Airflow |

**Ejemplo de configuración de infraestructura:**

```yaml
# infrastructure/azure-ml-workspace.yml - TDSP Infrastructure as Code

resources:
  - name: ml-workspace
    type: Microsoft.MachineLearningServices/workspaces@2021-07-01
    properties:
      friendlyName: "TDSP Team Workspace"
      description: "Workspace compartido del equipo Data Science"
      location: "westeurope"
      
      # Storage Account asociado
      storageAccount:
        id: "/subscriptions/.../storageAccounts/tdspstorageacct"
      
      # Key Vault para secrets
      keyVault:
        id: "/subscriptions/.../vaults/tdsp-keyvault"
      
      # Application Insights para logging
      applicationInsights:
        id: "/subscriptions/.../components/tdsp-appinsights"
      
      # Container Registry
      containerRegistry:
        id: "/subscriptions/.../registries/tdspregistry"

  - name: compute-cluster
    type: Microsoft.MachineLearningServices/workspaces/computes@2021-07-01
    properties:
      computeType: AmlCompute
      properties:
        vmSize: "Standard_DS3_v2"
        scaleSettings:
          minNodeCount: 0
          maxNodeCount: 10
          nodeIdleTimeBeforeScaleDown: "PT120S"  # 2 minutos
```

---

**4. Herramientas y utilidades:**

**Stack de herramientas TDSP:**

=== "Data Science"

    ```python
    # requirements.txt - TDSP Tools
    
    # Core Data Science
    pandas==2.0.3
    numpy==1.24.3
    scikit-learn==1.3.0
    xgboost==2.0.0
    
    # Azure Integration
    azureml-sdk==1.52.0
    azureml-core==1.52.0
    azureml-train-automl==1.52.0
    azure-storage-blob==12.17.0
    azure-identity==1.13.0
    
    # Visualization
    matplotlib==3.7.2
    seaborn==0.12.2
    plotly==5.15.0
    
    # Utilities
    python-dotenv==1.0.0
    pyyaml==6.0.1
    click==8.1.6  # CLI apps
    
    # Testing
    pytest==7.4.0
    pytest-cov==4.1.0
    ```

=== "Infrastructure"

    ```python
    # Deployment con Azure ML
    
    from azureml.core import Workspace, Environment, ScriptRunConfig
    from azureml.core.compute import ComputeTarget, AmlCompute
    
    # 1. Conectar a workspace
    ws = Workspace.from_config()  # Lee config.json
    
    # 2. Crear/obtener compute cluster
    compute_name = "cpu-cluster"
    
    if compute_name not in ws.compute_targets:
        compute_config = AmlCompute.provisioning_configuration(
            vm_size='STANDARD_D2_V2',
            max_nodes=4
        )
        compute_target = ComputeTarget.create(ws, compute_name, compute_config)
        compute_target.wait_for_completion(show_output=True)
    else:
        compute_target = ws.compute_targets[compute_name]
    
    # 3. Crear environment con dependencias
    env = Environment.from_conda_specification(
        name='tdsp-env',
        file_path='environment.yml'
    )
    
    # 4. Configurar training run
    src = ScriptRunConfig(
        source_directory='src/models',
        script='train_model.py',
        arguments=[
            '--data-path', 'data/processed/train.parquet',
            '--model-output', 'models/'
        ],
        compute_target=compute_target,
        environment=env
    )
    
    # 5. Submit experiment
    from azureml.core import Experiment
    
    experiment = Experiment(workspace=ws, name='churn-prediction')
    run = experiment.submit(config=src)
    
    print(f"Run ID: {run.id}")
    print(f"Status: {run.get_status()}")
    
    # Wait for completion
    run.wait_for_completion(show_output=True)
    ```

=== "MLOps"

    ```python
    # Automated Retraining Pipeline con Azure ML
    
    from azureml.pipeline.core import Pipeline, PipelineData
    from azureml.pipeline.steps import PythonScriptStep
    from azureml.core import Datastore
    
    # Datastore para datos intermedios
    default_store = ws.get_default_datastore()
    
    # Pipeline data
    processed_data = PipelineData("processed", datastore=default_store)
    trained_model = PipelineData("model", datastore=default_store)
    
    # Step 1: Data Preparation
    prep_step = PythonScriptStep(
        name="Data Preparation",
        script_name="prepare_data.py",
        arguments=[
            '--input-data', 'data/raw/',
            '--output-data', processed_data
        ],
        outputs=[processed_data],
        compute_target=compute_target,
        source_directory='src/data'
    )
    
    # Step 2: Model Training
    train_step = PythonScriptStep(
        name="Model Training",
        script_name="train_model.py",
        arguments=[
            '--input-data', processed_data,
            '--model-output', trained_model
        ],
        inputs=[processed_data],
        outputs=[trained_model],
        compute_target=compute_target,
        source_directory='src/models'
    )
    
    # Step 3: Model Evaluation & Registration
    eval_step = PythonScriptStep(
        name="Evaluate & Register",
        script_name="evaluate_model.py",
        arguments=['--model', trained_model],
        inputs=[trained_model],
        compute_target=compute_target,
        source_directory='src/models'
    )
    
    # Create pipeline
    pipeline = Pipeline(workspace=ws, steps=[prep_step, train_step, eval_step])
    
    # Publish pipeline
    published_pipeline = pipeline.publish(
        name="Churn Prediction Retraining",
        description="Monthly automated retraining"
    )
    
    # Schedule monthly execution
    from azureml.pipeline.core.schedule import ScheduleRecurrence, Schedule
    
    recurrence = ScheduleRecurrence(frequency="Month", interval=1, start_time="2024-03-01T00:00:00")
    schedule = Schedule.create(
        workspace=ws,
        name="Monthly-Retraining",
        pipeline_id=published_pipeline.id,
        experiment_name="Scheduled-Retraining",
        recurrence=recurrence
    )
    ```

---

**Ventajas de TDSP:**

!!! success "Beneficios Clave"
    - ✅ **Estandarización** del equipo completo
    - ✅ **Colaboración** facilitada (estructura común)
    - ✅ **Reproducibilidad** (control versiones código + datos)
    - ✅ **Escalabilidad** cloud-native
    - ✅ **MLOps integrado** desde el inicio
    - ✅ **Documentación estructurada**

---

## 9.4. Otras Metodologías

**Domino Data Science Lifecycle:**

**6 Fases basadas en CRISP-DM + Agile:**

```mermaid
graph LR
    A[1. Ideación] --> B[2. Adquisición<br/>Exploración]
    B --> C[3. Investigación<br/>Desarrollo]
    C --> D[4. Validación]
    D --> E[5. Entrega]
    E --> F[6. Supervisión]
    F -.Feedback.-> A
    
    style A fill:#FF6B6B
    style B fill:#FFD93D
    style C fill:#6BCB77
    style D fill:#4D96FF
    style E fill:#9D84B7
    style F fill:#FF8C42
```

**Características únicas:**
- Integra **backlog ágil** con ciencia de datos
- Énfasis en **mejora continua**
- **Observabilidad** y feedback de usuarios
- Iteraciones cortas (sprints 2-3 semanas)

---

**Data Science Workflow:**

**Fases de alto nivel:**

| Fase | Subfases | Descripción |
|------|----------|-------------|
| **Preparación** | • Definir problema<br/>• Recolectar datos<br/>• Explorar | Fase inicial iterativa |
| **Experimentación** | • Feature engineering<br/>• Modelado<br/>• Evaluación | Ciclo de prueba-error |
| **Distribución** | • Deployment<br/>• Monitoreo<br/>• Mantenimiento | Puesta en producción |

**Características:**
- Muy **flexible** y adaptable
- Ideal para **startups y equipos pequeños**
- Menos prescriptivo que CRISP-DM

---

**Toward Data Mining Engineering:**

**Extensión de CRISP-DM con enfoque de ingeniería:**

!!! tip "Incorpora estándares de ingeniería de software"
    - **IEEE Std 1074**: Procesos de desarrollo de software
    - **ISO 12207**: Ciclo de vida de software
    - **Gestión de proyectos**: WBS, Gantt, RACI
    - **Procesos organizativos**: Roles, gobernanza

**Fases adicionales:**
1. **Gestión de Proyecto**: Planning, tracking, control
2. **Gestión de Calidad**: Testing, validación, auditoría
3. **Gestión de Riesgos**: Identificación, mitigación
4. **Gestión de Configuración**: Versiones, releases

---

**Agile Delivery Framework for BI & Analytics:**

**Integra Agile con BI y Analytics:**

```mermaid
graph TD
    A[Product Backlog] --> B[Sprint Planning]
    B --> C[Sprint 2-3 semanas]
    C --> D{Demo & Review}
    D --> E[Retrospectiva]
    E --> F[Increment]
    F --> A
    
    C --> C1[Daily Standup]
    C1 --> C
    
    style A fill:#FFE5B4
    style C fill:#B4D7FF
    style F fill:#B4FFB4
```

**Roles:**
- **Product Owner**: Define prioridades analytics
- **Scrum Master**: Facilita proceso ágil
- **Data Science Team**: Analistas + Data Scientists + Engineers

**Artefactos:**
- **Product Backlog**: User stories de analytics
- **Sprint Backlog**: Tareas específicas del sprint
- **Increment**: Dashboard, modelo, insight accionable

---

## 9.5. Comparación de Todas las Metodologías

| Metodología | Origen | Fases | Énfasis Principal | Mejor Para |
|-------------|--------|-------|-------------------|------------|
| **KDD** | Academia (1990s) | 5 | Proceso técnico discovery | Investigación, algoritmos |
| **CRISP-DM** | Industria (1999) | 6 | Ciclo completo negocio | General, cualquier industria |
| **ASUM-DM** | IBM | 4 | Soluciones enterprise | Ecosistema IBM, proyectos grandes |
| **TDSP** | Microsoft | 5 + estructura | Colaboración en equipo | Azure, equipos corporativos |
| **Domino** | Domino Data Lab | 6 | Agilidad + observabilidad | Startups, iteración rápida |
| **DS Workflow** | Varios | 3 (flexible) | Simplicidad | Equipos pequeños, MVPs |
| **Engineering** | Academia | CRISP-DM + gestión | Ingeniería rigurosa | Proyectos críticos, regulados |
| **Agile BI** | Comunidad Agile | Sprints | Entregas incrementales | BI + Analytics ágil |

---

## 9.6. ¿Cuál Metodología Elegir?

!!! question "Guía de Selección"

    **Usa CRISP-DM si:**
    - Es tu primer proyecto de data science
    - No tienes restricciones de plataforma tecnológica
    - Necesitas un estándar probado y documentado
    - El equipo es multidisciplinario (negocio + técnico)
    
    **Usa ASUM-DM si:**
    - Trabajas con tecnologías IBM (Watson, Cloud Pak)
    - El proyecto es enterprise-grade
    - Requieres gobernanza estricta
    - Necesitas mejora continua estructurada (Operar y Optimizar)
    
    **Usa TDSP si:**
    - Tu stack es Azure/Microsoft
    - Trabajas en equipo (>3 personas)
    - Necesitas estandarización y reproducibilidad
    - MLOps es prioridad desde el inicio
    
    **Usa KDD si:**
    - Proyecto de investigación académica
    - Énfasis en desarrollo de algoritmos
    - No requieres deployment inmediato
    
    **Usa Domino/Agile si:**
    - Startup o equipo pequeño
    - Necesitas velocidad de iteración
    - Cultura ágil establecida
    - Feedback rápido de usuarios

---

## 9.7. Resumen y Mejores Prácticas

!!! success "Mejores Prácticas Universales (Todas las Metodologías)"
    
    **1. Comprensión del Negocio es CRÍTICA**
    - Todas las metodologías coinciden: empezar por el problema
    - Definir KPIs medibles
    - Alinear con stakeholders
    
    **2. Iteración es Esencial**
    - Ningún proyecto es lineal
    - Planificar para ciclos de retroalimentación
    - Aprender de cada iteración
    
    **3. Documentación Estructurada**
    - Código + Datos + Decisiones
    - Facilita reproducibilidad
    - Transfer de conocimiento
    
    **4. Colaboración Multidisciplinaria**
    - Negocio + Datos + IT + Dominio
    - Comunicación constante
    - Objetivos compartidos
    
    **5. Deploy != Final**
    - Monitoreo continuo
    - Reentrenamiento planificado
    - Mejora incremental
    
    **6. Gestión de Expectativas**
    - No todos los proyectos tienen ROI inmediato
    - Comunicar incertidumbre
    - Celebrar pequeños wins

---

## Conclusión

!!! abstract "Key Takeaways"
    - **CRISP-DM** sigue siendo el estándar de facto (agnóstico, probado)
    - **ASUM-DM** y **TDSP** son evoluciones corporativas con MLOps integrado
    - **KDD** proporciona los fundamentos teóricos
    - **Metodologías ágiles** ganan terreno en equipos modernos
    - **No hay "mejor" metodología** universal, depende del contexto
    - **Combinar** elementos de múltiples metodologías es válido y común

!!! tip "Recomendación Final"
    **Empieza con CRISP-DM** como base sólida. Luego, incorpora elementos de:
    - **TDSP** para estructura de proyecto y colaboración
    - **ASUM-DM** para fase operacional y mejora continua
    - **Agile** para sprints y entregas incrementales
    - **Tu propio playbook** perfeccionado con experiencia

---

*Fin del Capítulo 9: Metodologías Avanzadas de Ciencia de Datos*
