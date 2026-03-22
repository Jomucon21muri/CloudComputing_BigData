# CAPÍTULO 8: OLTP vs OLAP - Sistemas transaccionales y analíticos

!!! abstract "Dos Paradigmas Fundamentales"
    Los sistemas de información empresariales se dividen en dos categorías fundamentales: **OLTP** (sistemas transaccionales para operaciones) y **OLAP** (sistemas analíticos para toma de decisiones). Comprender sus diferencias es crítico para arquitecturas de datos efectivas.

---

## 8.1. OLTP (Online Transaction Processing)

!!! info "Definición"
    **OLTP** son sistemas optimizados para procesar grandes volúmenes de **transacciones** cortas y frecuentes, garantizando **ACID** (Atomicidad, Consistencia, Aislamiento, Durabilidad).

**Características de OLTP:**

| Característica | Descripción |
|----------------|-------------|
| **Objetivo** | Operaciones diarias del negocio (INSERT, UPDATE, DELETE) |
| **Usuarios** | Miles-millones de usuarios concurrentes |
| **Transacciones** | Cortas, frecuentes (milisegundos) |
| **Volumen escritura** | ⬆️ ALTO (muchas escrituras) |
| **Volumen lectura** | ➡️ Medio (consultas específicas por clave) |
| **Queries** | Simples, predecibles (SELECT * WHERE id=?) |
| **Datos** | Actuales, detallados, normalizados |
| **Modelo** | Normalizado (3NF) para evitar redundancia |
| **Storage** | Row-based (filas completas) |
| **Integridad** | ACID estricto, constraints, foreign keys |
| **Backup** | Continuo (transaction log) |
| **Ejemplos** | Sistemas bancarios, e-commerce, reservas |

**Casos de uso OLTP:**

- 🏦 **Banca:** Transferencias, retiros, depósitos
- 🛒 **E-commerce:** Carrito de compra, checkout, inventario
- ✈️ **Reservas:** Hoteles, vuelos, entradas
- 📱 **Apps móviles:** Redes sociales, mensajería
- 🏥 **Hospitales:** Historiales médicos, citas

---

**Ejemplo: base de datos OLTP (e-commerce):**

**Schema normalizado (3NF):**

```sql
-- === BASE DE DATOS OLTP NORMALIZADA ===

-- Tabla: customers
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_email (email)
);

-- Tabla: addresses (1-to-many con customers)
CREATE TABLE addresses (
    address_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    address_type ENUM('billing', 'shipping') NOT NULL,
    street VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(50),
    postal_code VARCHAR(20) NOT NULL,
    country VARCHAR(50) NOT NULL,
    is_default BOOLEAN DEFAULT FALSE,
    
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE CASCADE,
    INDEX idx_customer (customer_id)
);

-- Tabla: products
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category_id INT NOT NULL,
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    stock_quantity INT NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (category_id) REFERENCES categories(category_id),
    INDEX idx_category (category_id),
    INDEX idx_sku (sku)
);

-- Tabla: categories
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    parent_category_id INT,
    
    FOREIGN KEY (parent_category_id) REFERENCES categories(category_id)
);

-- Tabla: orders
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'confirmed', 'shipped', 'delivered', 'cancelled') NOT NULL,
    shipping_address_id INT NOT NULL,
    billing_address_id INT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (shipping_address_id) REFERENCES addresses(address_id),
    FOREIGN KEY (billing_address_id) REFERENCES addresses(address_id),
    
    INDEX idx_customer (customer_id),
    INDEX idx_order_date (order_date),
    INDEX idx_status (status)
);

-- Tabla: order_items (many-to-many entre orders y products)
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    
    INDEX idx_order (order_id),
    INDEX idx_product (product_id)
);

-- Tabla: payments
CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    payment_method ENUM('credit_card', 'debit_card', 'paypal', 'bank_transfer') NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    payment_status ENUM('pending', 'authorized', 'captured', 'failed', 'refunded') NOT NULL,
    transaction_id VARCHAR(255),
    payment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    INDEX idx_order (order_id),
    INDEX idx_transaction (transaction_id)
);
```

---

**Operaciones OLTP típicas:**

```sql
-- === OPERACIONES OLTP ===

-- 1. Registro de nuevo cliente
INSERT INTO customers (email, first_name, last_name, password_hash)
VALUES ('john.doe@email.com', 'John', 'Doe', '$2a$10$...');

-- 2. Agregar producto al carrito (actualizar stock)
BEGIN;
    -- Verificar stock disponible
    SELECT stock_quantity FROM products WHERE product_id = 123 FOR UPDATE;
    
    -- Si stock > 0, reservar
    UPDATE products 
    SET stock_quantity = stock_quantity - 1,
        updated_at = CURRENT_TIMESTAMP
    WHERE product_id = 123 AND stock_quantity > 0;
    
    -- Si UPDATE afectó 0 filas, no había stock
    IF ROW_COUNT() = 0 THEN
        ROLLBACK;
        RAISE EXCEPTION 'Producto fuera de stock';
    END IF;
COMMIT;

-- 3. Crear orden (transacción compleja)
BEGIN;
    -- Insertar orden
    INSERT INTO orders (customer_id, status, shipping_address_id, billing_address_id, total_amount)
    VALUES (456, 'pending', 789, 790, 234.50)
    RETURNING order_id INTO @new_order_id;
    
    -- Insertar items de la orden
    INSERT INTO order_items (order_id, product_id, quantity, unit_price, subtotal)
    VALUES 
        (@new_order_id, 123, 2, 49.99, 99.98),
        (@new_order_id, 456, 1, 134.52, 134.52);
    
    -- Actualizar stock de productos
    UPDATE products SET stock_quantity = stock_quantity - 2 WHERE product_id = 123;
    UPDATE products SET stock_quantity = stock_quantity - 1 WHERE product_id = 456;
    
    -- Registrar pago
    INSERT INTO payments (order_id, payment_method, amount, payment_status)
    VALUES (@new_order_id, 'credit_card', 234.50, 'authorized');
    
COMMIT;

-- 4. Consultar orden de cliente (query simple por clave)
SELECT 
    o.order_id,
    o.order_date,
    o.status,
    o.total_amount,
    c.first_name,
    c.last_name,
    c.email
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_id = 12345;

-- 5. Actualizar estado de orden
UPDATE orders 
SET status = 'shipped', 
    updated_at = CURRENT_TIMESTAMP
WHERE order_id = 12345;

-- 6. Cancelar orden (rollback de stock)
BEGIN;
    -- Devolver stock
    UPDATE products p
    SET stock_quantity = stock_quantity + oi.quantity
    FROM order_items oi
    WHERE p.product_id = oi.product_id
      AND oi.order_id = 12345;
    
    -- Marcar orden como cancelada
    UPDATE orders SET status = 'cancelled' WHERE order_id = 12345;
    
    -- Refund pago
    UPDATE payments SET payment_status = 'refunded' WHERE order_id = 12345;
COMMIT;
```

**Características de estas queries:**

- ✅ **Simples:** Afectan pocas filas (1-100)
- ✅ **Indexadas:** Usan PRIMARY KEY o índices
- ✅ **Rápidas:** < 10ms típicamente
- ✅ **ACID:** Transacciones con BEGIN/COMMIT
- ✅ **Concurrentes:** Locks a nivel de fila

---

## 8.2. OLAP (Online Analytical Processing)

!!! info "Definición"
    **OLAP** son sistemas optimizados para **análisis** y **consultas complejas** sobre grandes volúmenes de datos históricos, con énfasis en agregaciones y multidimensionalidad.

**Características de OLAP:**

| Característica | Descripción |
|----------------|-------------|
| **Objetivo** | Análisis, reporting, inteligencia de negocio |
| **Usuarios** | Decenas-cientos analistas/ejecutivos |
| **Queries** | Largas, complejas (segundos-minutos) |
| **Volumen escritura** | ⬇️ BAJO (bulk loads periódicos) |
| **Volumen lectura** | ⬆️ ALTO (scans completos, agregaciones) |
| **Queries** | Complejas (JOIN múltiples, GROUP BY, aggregations) |
| **Datos** | Históricos, agregados, desnormalizados |
| **Modelo** | Star/Snowflake schema (dimensional) |
| **Storage** | Column-based (columnas completas) |
| **Integridad** | Relajada (eventual consistency OK) |
| **Backup** | Snapshots periódicos |
| **Ejemplos** | Data Warehouses, BI dashboards |

**Casos de uso OLAP:**

- 📊 **Reportes ejecutivos:** KPIs, dashboards
- 📈 **Análisis de tendencias:** Ventas por mes/año
- 🔍 **Data mining:** Segmentación de clientes
- 🎯 **Forecasting:** Predicción de demanda
- 💰 **Análisis financiero:** P&L, balance sheets

---

## 8.3. OLTP vs OLAP: comparación completa

```mermaid
graph TD
    subgraph OLTP[OLTP - Transaccional]
        A1[Aplicaciones<br/>Web/Móvil] --> B1[(Base OLTP<br/>Normalizada)]
        B1 --> C1[Operaciones<br/>Diarias]
    end
    
    subgraph ETL[ETL Process]
        D[Extract] --> E[Transform]
        E --> F[Load]
    end
    
    subgraph OLAP[OLAP - Analítico]
        G1[(Data Warehouse<br/>Star Schema)] --> H1[Análisis<br/>BI]
        H1 --> I1[Dashboards<br/>Reportes]
    end
    
    B1 -.ETL Nightly.-> D
    F --> G1
    
    style OLTP fill:#E3F2FD
    style ETL fill:#FFF3E0
    style OLAP fill:#E8F5E9
```

| Aspecto | OLTP | OLAP |
|---------|------|------|
| **Propósito** | Operaciones diarias | Análisis y decisiones |
| **Workload** | INSERT, UPDATE, DELETE | SELECT complejos |
| **Performance** | Milisegundos | Segundos a minutos |
| **Modelo de datos** | Normalizado (3NF) | Dimensional (Star/Snowflake) |
| **Orientación** | Aplicación | Análisis |
| **Volumen queries** | ⬆️ Miles/segundo | ⬇️ Decenas/hora |
| **Complejidad queries** | ⬇️ Simples (índices) | ⬆️ Complejas (scans, agregaciones) |
| **Datos históricos** | ❌ Solo actuales | ✅ Años de historia |
| **Tamaño DB** | GB a TB | TB a PB |
| **Escrituras** | Continuas | Batch (ETL) |
| **Lecturas** | Por clave (índice) | Rangos, scans completos |
| **Usuarios concurrentes** | ⬆️ Miles | ⬇️ Cientos |
| **Ejemplo de query** | "Dame la orden #12345" | "Ventas totales por región y mes en 2023" |
| **Storage** | Row-oriented (PostgreSQL, MySQL) | Column-oriented (Redshift, BigQuery) |
| **Diseño para** | Alta transaccionalidad | Análisis multidimensional |
| **Optimización** | Locks, índices B-tree | Particionamiento, compresión columnar |
| **Backup/Recovery** | Crítico (transacciones) | Menos crítico (reconstruible desde OLTP) |

---

## 8.4. Modelado dimensional: Star Schema

!!! success "Star Schema"
    El **Star Schema** es el modelo dimensional más común, con una tabla de **hechos** central rodeada de tablas de **dimensiones**.

**Componentes:**

**1. Tabla de hechos (Fact Table):**

- Contiene métricas/medidas numéricas (ventas, cantidad, costos)
- Foreign keys a dimensiones
- Grano (granularidad): Cada fila representa un evento/transacción
- Gran volumen (millones-billones de filas)

**2. Tablas de Dimensiones (Dimension Tables):**

- Atributos descriptivos (quién, qué, dónde, cuándo)
- Primary key (surrogate key)
- Desnormalizadas (pueden tener redundancia)
- Menor volumen (miles-millones de filas)

---

**Ejemplo: Star Schema de ventas:**

```sql
-- === STAR SCHEMA: DATA WAREHOUSE DE VENTAS ===

-- DIMENSIÓN: Tiempo (generada con script)
CREATE TABLE dim_time (
    time_key INT PRIMARY KEY,              -- Surrogate key: 20240220
    date DATE NOT NULL,                    -- 2024-02-20
    day_of_week VARCHAR(10),               -- Tuesday
    day_of_month INT,                      -- 20
    day_of_year INT,                       -- 51
    week_of_year INT,                      -- 8
    month_name VARCHAR(10),                -- February
    month_num INT,                         -- 2
    quarter INT,                           -- 1
    year INT,                              -- 2024
    is_weekend BOOLEAN,                    -- FALSE
    is_holiday BOOLEAN,                    -- FALSE
    fiscal_year INT,                       -- 2024
    fiscal_quarter INT                     -- Q3
);

-- DIMENSIÓN: Producto
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,           -- Surrogate key
    product_id INT NOT NULL,               -- Natural key (del OLTP)
    sku VARCHAR(50),
    product_name VARCHAR(255),
    brand VARCHAR(100),
    category VARCHAR(100),
    subcategory VARCHAR(100),
    unit_cost DECIMAL(10,2),
    unit_price DECIMAL(10,2),
    -- Atributos desnormalizados para queries rápidas
    category_manager VARCHAR(100),
    supplier_name VARCHAR(255),
    supplier_country VARCHAR(50),
    -- SCD Type 2 (Slowly Changing Dimensions)
    effective_date DATE,
    expiration_date DATE,
    is_current BOOLEAN
);

-- DIMENSIÓN: Cliente
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,          -- Surrogate key
    customer_id INT NOT NULL,              -- Natural key
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    email VARCHAR(255),
    gender VARCHAR(10),
    birth_date DATE,
    age_group VARCHAR(20),                 -- '18-25', '26-35', etc.
    -- Desnormalización de dirección
    city VARCHAR(100),
    state VARCHAR(50),
    country VARCHAR(50),
    postal_code VARCHAR(20),
    region VARCHAR(50),                    -- North, South, East, West
    -- Segmentación
    customer_segment VARCHAR(50),          -- VIP, Regular, Occasional
    lifetime_value DECIMAL(10,2),
    -- SCD Type 2
    effective_date DATE,
    expiration_date DATE,
    is_current BOOLEAN
);

-- DIMENSIÓN: Tienda/Canal
CREATE TABLE dim_store (
    store_key INT PRIMARY KEY,
    store_id INT NOT NULL,
    store_name VARCHAR(255),
    store_type VARCHAR(50),                -- Physical, Online, Mobile
    city VARCHAR(100),
    state VARCHAR(50),
    country VARCHAR(50),
    region VARCHAR(50),
    district_manager VARCHAR(100),
    opening_date DATE,
    store_size_sqft INT
);

-- DIMENSIÓN: Promoción
CREATE TABLE dim_promotion (
    promotion_key INT PRIMARY KEY,
    promotion_id INT,
    promotion_name VARCHAR(255),
    promotion_type VARCHAR(50),            -- Discount, BOGO, Freebie
    discount_percent DECIMAL(5,2),
    start_date DATE,
    end_date DATE
);

-- TABLA DE HECHOS: Ventas
CREATE TABLE fact_sales (
    -- Foreign Keys a dimensiones (desnormalizado)
    time_key INT NOT NULL,
    product_key INT NOT NULL,
    customer_key INT NOT NULL,
    store_key INT NOT NULL,
    promotion_key INT,
    
    -- Degenerate dimension (no tiene tabla dimension propia)
    order_id INT NOT NULL,
    order_line_number INT NOT NULL,
    
    -- Medidas aditivas (se pueden sumar)
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    discount_amount DECIMAL(10,2) DEFAULT 0,
    sales_amount DECIMAL(10,2) NOT NULL,   -- quantity * unit_price - discount
    cost_amount DECIMAL(10,2) NOT NULL,
    profit_amount DECIMAL(10,2) NOT NULL,  -- sales_amount - cost_amount
    tax_amount DECIMAL(10,2) NOT NULL,
    
    -- Medidas semi-aditivas (solo se suman en algunas dimensiones)
    inventory_quantity INT,                -- No aditivo en tiempo
    
    -- Medidas no aditivas (no se deben sumar, solo promediar)
    profit_margin_percent DECIMAL(5,2),    -- (profit / sales) * 100
    
    -- Foreign Keys
    FOREIGN KEY (time_key) REFERENCES dim_time(time_key),
    FOREIGN KEY (product_key) REFERENCES dim_product(product_key),
    FOREIGN KEY (customer_key) REFERENCES dim_customer(customer_key),
    FOREIGN KEY (store_key) REFERENCES dim_store(store_key),
    FOREIGN KEY (promotion_key) REFERENCES dim_promotion(promotion_key)
);

-- Índices para performance (columnar storage los hace menos necesarios)
CREATE INDEX idx_fact_sales_time ON fact_sales(time_key);
CREATE INDEX idx_fact_sales_product ON fact_sales(product_key);
CREATE INDEX idx_fact_sales_customer ON fact_sales(customer_key);
CREATE INDEX idx_fact_sales_store ON fact_sales(store_key);

-- Particionamiento por tiempo (crítico para performance)
-- En sistemas como Redshift, BigQuery, Snowflake:
-- ALTER TABLE fact_sales PARTITION BY RANGE (time_key);
```

**Visualización Star Schema:**

```mermaid
graph TD
    A[dim_time<br/>time_key*<br/>date<br/>month<br/>quarter<br/>year] --> E
    B[dim_product<br/>product_key*<br/>sku<br/>name<br/>category<br/>brand] --> E
    C[dim_customer<br/>customer_key*<br/>name<br/>city<br/>segment<br/>ltv] --> E
    D[dim_store<br/>store_key*<br/>name<br/>region<br/>type] --> E
    F[dim_promotion<br/>promotion_key*<br/>name<br/>discount] --> E
    
    E[fact_sales<br/>time_key FK<br/>product_key FK<br/>customer_key FK<br/>store_key FK<br/>promotion_key FK<br/>---<br/>quantity<br/>sales_amount<br/>cost_amount<br/>profit_amount]
    
    style E fill:#FFD93D
    style A fill:#E3F2FD
    style B fill:#E3F2FD
    style C fill:#E3F2FD
    style D fill:#E3F2FD
    style F fill:#E3F2FD
```

---

**Queries OLAP sobre Star Schema:**

```sql
-- === QUERIES ANALÍTICOS OLAP ===

-- 1. Ventas totales por año y mes
SELECT 
    t.year,
    t.month_name,
    SUM(f.sales_amount) as total_sales,
    SUM(f.profit_amount) as total_profit,
    COUNT(DISTINCT f.customer_key) as unique_customers,
    AVG(f.sales_amount) as avg_transaction_value
FROM fact_sales f
JOIN dim_time t ON f.time_key = t.time_key
WHERE t.year = 2024
GROUP BY t.year, t.month_num, t.month_name
ORDER BY t.month_num;

-- 2. Top 10 productos por ventas en cada región
WITH product_sales_by_region AS (
    SELECT 
        s.region,
        p.product_name,
        p.category,
        SUM(f.sales_amount) as total_sales,
        SUM(f.quantity) as units_sold,
        ROW_NUMBER() OVER (PARTITION BY s.region ORDER BY SUM(f.sales_amount) DESC) as rank
    FROM fact_sales f
    JOIN dim_product p ON f.product_key = p.product_key
    JOIN dim_store s ON f.store_key = s.store_key
    GROUP BY s.region, p.product_name, p.category
)
SELECT * FROM product_sales_by_region
WHERE rank <= 10
ORDER BY region, rank;

-- 3. Análisis de clientes: RFM (Recency, Frequency, Monetary)
SELECT 
    c.customer_key,
    c.first_name || ' ' || c.last_name as customer_name,
    c.customer_segment,
    
    -- Recency: Días desde última compra
    DATEDIFF(day, MAX(t.date), CURRENT_DATE) as days_since_last_purchase,
    
    -- Frequency: Número de transacciones
    COUNT(DISTINCT f.order_id) as num_orders,
    
    -- Monetary: Valor total gastado
    SUM(f.sales_amount) as total_spent,
    
    -- Métricas adicionales
    AVG(f.sales_amount) as avg_order_value,
    SUM(f.profit_amount) as customer_profit
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
JOIN dim_time t ON f.time_key = t.time_key
WHERE c.is_current = TRUE  -- Solo versión actual (SCD Type 2)
  AND t.year >= 2023
GROUP BY c.customer_key, c.first_name, c.last_name, c.customer_segment
HAVING COUNT(DISTINCT f.order_id) >= 3  -- Solo clientes con 3+ órdenes
ORDER BY total_spent DESC
LIMIT 100;

-- 4. Ventas comparativas año sobre año (YoY)
SELECT 
    p.category,
    SUM(CASE WHEN t.year = 2024 THEN f.sales_amount ELSE 0 END) as sales_2024,
    SUM(CASE WHEN t.year = 2023 THEN f.sales_amount ELSE 0 END) as sales_2023,
    
    -- YoY Growth
    ((SUM(CASE WHEN t.year = 2024 THEN f.sales_amount ELSE 0 END) -
      SUM(CASE WHEN t.year = 2023 THEN f.sales_amount ELSE 0 END)) /
      NULLIF(SUM(CASE WHEN t.year = 2023 THEN f.sales_amount ELSE 0 END), 0) * 100
    ) as yoy_growth_percent
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_time t ON f.time_key = t.time_key
WHERE t.year IN (2023, 2024)
  AND t.month_num = 2  -- Solo febrero
GROUP BY p.category
ORDER BY sales_2024 DESC;

-- 5. Impacto de promociones
SELECT 
    pr.promotion_name,
    pr.promotion_type,
    pr.discount_percent,
    
    -- Con promoción
    COUNT(DISTINCT f.order_id) as orders_with_promo,
    SUM(f.sales_amount) as sales_with_promo,
    SUM(f.profit_amount) as profit_with_promo,
    AVG(f.sales_amount) as avg_order_value,
    
    -- ROI de la promoción
    (SUM(f.profit_amount) / NULLIF(SUM(f.discount_amount), 0)) as promotion_roi
FROM fact_sales f
JOIN dim_promotion pr ON f.promotion_key = pr.promotion_key
WHERE pr.promotion_id IS NOT NULL  -- Solo transacciones con promoción
GROUP BY pr.promotion_name, pr.promotion_type, pr.discount_percent
HAVING SUM(f.profit_amount) > 0
ORDER BY promotion_roi DESC;

-- 6. Cohort Analysis: Retención de clientes por mes de primera compra
WITH first_purchase AS (
    SELECT 
        c.customer_key,
        MIN(t.date) as first_purchase_date,
        DATE_TRUNC('month', MIN(t.date)) as cohort_month
    FROM fact_sales f
    JOIN dim_customer c ON f.customer_key = c.customer_key
    JOIN dim_time t ON f.time_key = t.time_key
    GROUP BY c.customer_key
),
cohort_activity AS (
    SELECT 
        fp.cohort_month,
        DATE_TRUNC('month', t.date) as activity_month,
        DATEDIFF(month, fp.cohort_month, DATE_TRUNC('month', t.date)) as months_since_first,
        COUNT(DISTINCT f.customer_key) as active_customers
    FROM first_purchase fp
    JOIN fact_sales f ON fp.customer_key = f.customer_key
    JOIN dim_time t ON f.time_key = t.time_key
    GROUP BY fp.cohort_month, DATE_TRUNC('month', t.date)
)
SELECT 
    cohort_month,
    months_since_first,
    active_customers,
    FIRST_VALUE(active_customers) OVER (
        PARTITION BY cohort_month 
        ORDER BY months_since_first
    ) as cohort_size,
    
    -- Retention %
    (active_customers::FLOAT / 
     FIRST_VALUE(active_customers) OVER (
         PARTITION BY cohort_month 
         ORDER BY months_since_first
     ) * 100
    ) as retention_percent
FROM cohort_activity
ORDER BY cohort_month, months_since_first;

-- 7. Drill-down jerárquico: Año > Trimestre > Mes > Día
-- Ejemplo: Investigar caída de ventas
SELECT 
    t.year,
    t.quarter,
    t.month_name,
    t.day_of_month,
    SUM(f.sales_amount) as daily_sales,
    AVG(SUM(f.sales_amount)) OVER (
        PARTITION BY t.year, t.month_num 
        ORDER BY t.day_of_month
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7_days
FROM fact_sales f
JOIN dim_time t ON f.time_key = t.time_key
WHERE t.year = 2024 
  AND t.month_num = 2
GROUP BY t.year, t.quarter, t.month_name, t.month_num, t.day_of_month
ORDER BY t.year, t.month_num, t.day_of_month;
```

**Características de estas queries:**

- ❌ **Complejas:** Múltiples JOINs, GROUP BY, ventanas
- ❌ **Lentas:** Segundos-minutos (vs milisegundos en OLTP)
- ✅ **Agregaciones:** SUM, COUNT, AVG sobre millones de filas
- ✅ **Análisis temporal:** Comparaciones año/año, tendencias
- ✅ **Multidimensional:** Slice & dice por producto/cliente/tiempo

---

## 8.5. Slowly Changing Dimensions (SCD)

!!! tip "SCD: Manejo de Cambios en Dimensiones"
    Las dimensiones cambian con el tiempo (ej: cliente se muda, producto cambia de precio). SCD define cómo manejar estos cambios.

**Tipos de SCD:**

| Tipo | Estrategia | Descripción | Uso |
|------|-----------|-------------|-----|
| **Type 0** | No cambiar | Mantener valor original siempre | Atributos inmutables (fecha nacimiento) |
| **Type 1** | Sobrescribir | Actualizar y perder historia | Correcciones (typos), datos no críticos |
| **Type 2** | Nueva fila** | Crear nueva versión con fechas efectivas | **Más común**, mantiene historia completa|
| **Type 3** | Nueva columna | Mantener valor actual y anterior | Solo un cambio histórico (ej: previous_city) |
| **Type 4** | Mini-dimension | Tabla separada para atributos que cambian | Alternativa a Type 2 |
| **Type 6** | Híbrido (1+2+3) | Combina Type 1, 2 y 3 | Máxima flexibilidad |

---

**SCD Type 2: ejemplo completo:**

```sql
-- === SCD TYPE 2: CAMBIOS EN dim_customer ===

-- Estructura con SCD Type 2
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,          -- Surrogate key (auto-increment)
    customer_id INT NOT NULL,              -- Natural key (del OLTP)
    
    -- Atributos que pueden cambiar
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    email VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(50),
    customer_segment VARCHAR(50),          -- VIP, Regular, Occasional
    
    -- Campos SCD Type 2
    effective_date DATE NOT NULL,          -- Cuándo empezó esta versión
    expiration_date DATE,                  -- Cuándo terminó (NULL = actual)
    is_current BOOLEAN DEFAULT TRUE,       -- TRUE solo para versión actual
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ESCENARIO: Cliente se muda y cambia de segmento
-- Estado inicial:
INSERT INTO dim_customer VALUES (
    1,                          -- customer_key
    1001,                       -- customer_id (natural key)
    'John', 'Doe',
    'john@email.com',
    'New York', 'NY',
    'Regular',
    '2020-01-15',              -- effective_date (cuando se dio de alta)
    NULL,                       -- expiration_date (NULL = actual)
    TRUE,                       -- is_current
    CURRENT_TIMESTAMP,
    CURRENT_TIMESTAMP
);

-- El 2024-02-20, cliente se muda a California y ahora es VIP

-- PROCESO SCD Type 2:

-- 1. Cerrar versión anterior (dejar de ser current)
UPDATE dim_customer
SET 
    expiration_date = '2024-02-19',       -- Día antes del cambio
    is_current = FALSE,                    -- Ya no es la versión actual
    updated_at = CURRENT_TIMESTAMP
WHERE customer_id = 1001 
  AND is_current = TRUE;

-- 2. Insertar nueva versión (nueva fila)
INSERT INTO dim_customer VALUES (
    2,                          -- NUEVO customer_key (surrogate key diferente)
    1001,                       -- MISMO customer_id (natural key)
    'John', 'Doe',
    'john@email.com',
    'San Francisco', 'CA',      -- Nueva ciudad/estado
    'VIP',                      -- Nuevo segmento
    '2024-02-20',              -- effective_date (fecha del cambio)
    NULL,                       -- expiration_date (NULL = actual)
    TRUE,                       -- is_current
    CURRENT_TIMESTAMP,
    CURRENT_TIMESTAMP
);

-- RESULTADO: Ahora tenemos 2 filas para customer_id = 1001
SELECT * FROM dim_customer WHERE customer_id = 1001 ORDER BY effective_date;

/*
customer_key | customer_id | city          | state | segment  | effective_date | expiration_date | is_current
-------------|-------------|---------------|-------|----------|----------------|-----------------|------------
1            | 1001        | New York      | NY    | Regular  | 2020-01-15     | 2024-02-19      | FALSE
2            | 1001        | San Francisco | CA    | VIP      | 2024-02-20     | NULL            | TRUE
*/

-- QUERIES CON SCD TYPE 2:

-- Query 1: Ver versión ACTUAL de cliente
SELECT * FROM dim_customer 
WHERE customer_id = 1001 AND is_current = TRUE;

-- Query 2: Ver TODAS las versiones (historia completa)
SELECT 
    customer_key,
    city || ', ' || state as location,
    customer_segment,
    effective_date,
    expiration_date,
    DATEDIFF(day, effective_date, COALESCE(expiration_date, CURRENT_DATE)) as days_in_this_version
FROM dim_customer
WHERE customer_id = 1001
ORDER BY effective_date;

-- Query 3: Ver cuál era el segmento del cliente en una fecha específica (time-travel)
SELECT customer_segment
FROM dim_customer
WHERE customer_id = 1001
  AND effective_date <= '2022-06-15'
  AND (expiration_date IS NULL OR expiration_date >= '2022-06-15');
-- Resultado: "Regular" (porque el cambio a VIP fue en 2024-02-20)

-- Query 4: Análisis histórico de fact_sales (respeta la versión correcta)
-- Las fact_sales usan customer_key (surrogate), no customer_id (natural)
-- Entonces automáticamente apuntan a la versión correcta del cliente según la fecha

SELECT 
    c.customer_segment,
    COUNT(*) as num_transactions,
    SUM(f.sales_amount) as total_sales
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key  -- Join por surrogate key
GROUP BY c.customer_segment;

/*
Esta query respeta la historia:
- Ventas hechas cuando era "Regular" se cuentan en "Regular"
- Ventas hechas después de 2024-02-20 se cuentan en "VIP"
*/
```

---

## 8.6. ETL: de OLTP a OLAP

!!! info "ETL Process"
    **ETL** (Extract, Transform, Load) es el proceso de mover datos desde sistemas OLTP a Data Warehouse OLAP.

**Proceso ETL completo:**

```python
# === ETL: OLTP → OLAP ===

import pandas as pd
from sqlalchemy import create_engine
from datetime import datetime, date

# Conexiones
oltp_engine = create_engine('postgresql://user:pass@oltp-db:5432/ecommerce')
olap_engine = create_engine('postgresql://user:pass@dwh-db:5432/datawarehouse')

def etl_sales_daily(execution_date):
    """
    ETL diario para cargar ventas del día anterior al DWH
    
    Args:
        execution_date: Fecha de ejecución (format: YYYY-MM-DD)
    """
    print(f"🔄 Iniciando ETL para {execution_date}")
    
    # ==================
    # 1. EXTRACT
    # ==================
    print("\n📥 EXTRACT: Extrayendo datos del OLTP...")
    
    # Extraer órdenes del día
    query_orders = f"""
    SELECT 
        o.order_id,
        o.customer_id,
        o.order_date,
        o.total_amount,
        oi.order_item_id,
        oi.product_id,
        oi.quantity,
        oi.unit_price,
        oi.subtotal,
        p.payment_method
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    LEFT JOIN payments p ON o.order_id = p.order_id
    WHERE DATE(o.order_date) = '{execution_date}'
      AND o.status IN ('confirmed', 'delivered')
      AND p.payment_status = 'captured'
    """
    
    df_orders = pd.read_sql(query_orders, oltp_engine)
    print(f"   ✅ Extraídas {len(df_orders):,} transacciones")
    
    # Extraer datos de clientes
    customer_ids = df_orders['customer_id'].unique()
    query_customers = f"""
    SELECT 
        c.customer_id,
        c.first_name,
        c.last_name,
        c.email,
        a.city,
        a.state,
        a.country
    FROM customers c
    LEFT JOIN addresses a ON c.customer_id = a.customer_id AND a.is_default = TRUE
    WHERE c.customer_id IN ({','.join(map(str, customer_ids))})
    """
    
    df_customers = pd.read_sql(query_customers, oltp_engine)
    print(f"   ✅ Extraídos {len(df_customers):,} clientes")
    
    # Extraer datos de products
    product_ids = df_orders['product_id'].unique()
    query_products = f"""
    SELECT 
        p.product_id,
        p.sku,
        p.name as product_name,
        p.price,
        c.name as category
    FROM products p
    JOIN categories c ON p.category_id = c.category_id
    WHERE p.product_id IN ({','.join(map(str, product_ids))})
    """
    
    df_products = pd.read_sql(query_products, oltp_engine)
    print(f"   ✅ Extraídos {len(df_products):,} productos")
    
    # ==================
    # 2. TRANSFORM
    # ==================
    print("\n🔄 TRANSFORM: Transformando datos...")
    
    # 2.1. Lookup de dimension keys (en lugar de natural keys)
    
    # dim_time
    df_orders['time_key'] = pd.to_datetime(df_orders['order_date']).dt.strftime('%Y%m%d').astype(int)
    
    # dim_customer (con SCD Type 2)
    query_customer_keys = """
    SELECT customer_id, customer_key 
    FROM dim_customer 
    WHERE is_current = TRUE
    """
    df_customer_keys = pd.read_sql(query_customer_keys, olap_engine)
    df_orders = df_orders.merge(df_customer_keys, on='customer_id', how='left')
    
    # Clientes nuevos que no existen en DWH
    new_customers = df_orders[df_orders['customer_key'].isna()]['customer_id'].unique()
    if len(new_customers) > 0:
        print(f"   ⚠️  {len(new_customers)} clientes nuevos detectados, insertando en dim_customer...")
        for customer_id in new_customers:
            customer_data = df_customers[df_customers['customer_id'] == customer_id].iloc[0]
            
            # Determinar segmento (lógica de negocio)
            lifetime_value = df_orders[df_orders['customer_id'] == customer_id]['subtotal'].sum()
            if lifetime_value > 1000:
                segment = 'VIP'
            elif lifetime_value > 500:
                segment = 'Regular'
            else:
                segment = 'Occasional'
            
            # Insertar en dim_customer
            insert_query = f"""
            INSERT INTO dim_customer (customer_id, first_name, last_name, email, city, state, 
                                     customer_segment, effective_date, expiration_date, is_current)
            VALUES ({customer_id}, '{customer_data['first_name']}', '{customer_data['last_name']}',
                    '{customer_data['email']}', '{customer_data['city']}', '{customer_data['state']}',
                    '{segment}', '{execution_date}', NULL, TRUE)
            RETURNING customer_key
            """
            result = olap_engine.execute(insert_query)
            customer_key = result.fetchone()[0]
            df_orders.loc[df_orders['customer_id'] == customer_id, 'customer_key'] = customer_key
    
    # Similar para dim_product (omitido por brevedad)
    # ...
    
    # 2.2. Calcular métricas derivadas
    df_orders['cost_amount'] = df_orders['quantity'] * df_products['price'] * 0.6  # Costo = 60% precio
    df_orders['profit_amount'] = df_orders['subtotal'] - df_orders['cost_amount']
    df_orders['profit_margin_percent'] = (df_orders['profit_amount'] / df_orders['subtotal']) * 100
    
    # 2.3. Data Quality: Validaciones
    assert df_orders['quantity'].min() > 0, "❌ Quantities negativas"
    assert df_orders['subtotal'].min() >= 0, "❌ Subtotals negativos"
    assert df_orders['customer_key'].notna().all(), "❌ customer_key nulos"
    
    print(f"   ✅ Transformación completa: {len(df_orders):,} filas listas")
    
    # ==================
    # 3. LOAD
    # ==================
    print("\n💾 LOAD: Cargando a Data Warehouse...")
    
    # Preparar DataFrame para fact_sales
    df_fact_sales = df_orders[[
        'time_key', 'product_key', 'customer_key', 'store_key', 'promotion_key',
        'order_id', 'order_item_id',
        'quantity', 'unit_price', 'subtotal', 'cost_amount', 'profit_amount', 'profit_margin_percent'
    ]].rename(columns={
        'subtotal': 'sales_amount',
        'order_item_id': 'order_line_number'
    })
    
    # Insertar en fact_sales (append)
    df_fact_sales.to_sql(
        'fact_sales',
        olap_engine,
        if_exists='append',
        index=False,
        method='multi',
        chunksize=1000
    )
    
    print(f"   ✅ Insertadas {len(df_fact_sales):,} filas en fact_sales")
    
    # ==================
    # 4. POST-LOAD
    # ==================
    print("\n🔧 POST-LOAD: Actualizando agregados...")
    
    # Actualizar tabla agregada (materialized view)
    olap_engine.execute(f"""
    INSERT INTO agg_daily_sales (date, total_sales, total_profit, num_transactions)
    SELECT 
        '{execution_date}'::DATE,
        SUM(sales_amount),
        SUM(profit_amount),
        COUNT(DISTINCT order_id)
    FROM fact_sales
    WHERE time_key = {int(execution_date.replace('-', ''))}
    ON CONFLICT (date) DO UPDATE SET
        total_sales = EXCLUDED.total_sales,
        total_profit = EXCLUDED.total_profit,
        num_transactions = EXCLUDED.num_transactions
    """)
    
    print(f"   ✅ Agregados actualizados")
    
    # ==================
    # 5. DATA QUALITY REPORT
    # ==================
    print("\n📊 QUALITY REPORT:")
    print(f"   Total ventas: ${df_fact_sales['sales_amount'].sum():,.2f}")
    print(f"   Total profit: ${df_fact_sales['profit_amount'].sum():,.2f}")
    print(f"   Margen promedio: {df_fact_sales['profit_margin_percent'].mean():.2f}%")
    print(f"   Clientes únicos: {df_fact_sales['customer_key'].nunique():,}")
    print(f"   Productos únicos: {df_fact_sales['product_key'].nunique():,}")
    
    print(f"\n✅ ETL completado exitosamente para {execution_date}")

# Ejecutar ETL
if __name__ == '__main__':
    execution_date = (datetime.now().date() - pd.Timedelta(days=1)).strftime('%Y-%m-%d')
    etl_sales_daily(execution_date)

# Output:
# 🔄 Iniciando ETL para 2024-02-19
# 
# 📥 EXTRACT: Extrayendo datos del OLTP...
#    ✅ Extraídas 12,456 transacciones
#    ✅ Extraídos 3,421 clientes
#    ✅ Extraídos 892 productos
# 
# 🔄 TRANSFORM: Transformando datos...
#    ⚠️  127 clientes nuevos detectados, insertando en dim_customer...
#    ✅ Transformación completa: 12,456 filas listas
# 
# 💾 LOAD: Cargando a Data Warehouse...
#    ✅ Insertadas 12,456 filas en fact_sales
# 
# 🔧 POST-LOAD: Actualizando agregados...
#    ✅ Agregados actualizados
# 
# 📊 QUALITY REPORT:
#    Total ventas: $456,789.50
#    Total profit: $123,456.78
#    Margen promedio: 27.03%
#    Clientes únicos: 3,421
#    Productos únicos: 892
# 
# ✅ ETL completado exitosamente para 2024-02-19
```

---

## 8.7. Tecnologías OLTP vs OLAP

**Bases de datos OLTP:**

| DB | Tipo | Características | Casos de Uso |
|----|------|-----------------|--------------|
| **PostgreSQL** | Relacional | ACID fuerte, extensible, JSON support | Apps web, SaaS |
| **MySQL** | Relacional | Rápido reads, popular, fácil | WordPress, e-commerce |
| **Oracle** | Relacional | Enterprise, alta disponibilidad | Bancos, Fortune 500 |
| **SQL Server** | Relacional | Integración Microsoft stack | Empresas Windows |
| **PostgreSQL** | NoSQL Document | Flexible schema, escala horizontal | Apps real-time |

**Bases de datos OLAP (Data Warehouses):**

| DWH | Storage | Características | Pricing |
|-----|---------|-----------------|---------|
| **Amazon Redshift** | Columnar (ParAccel) | Escala a petabytes, integración AWS | Por hora cluster |
| **Google BigQuery** | Columnar | Serverless, SQL estándar, ML integrado | Por TB escaneado |
| **Snowflake** | Columnar | Multi-cloud, separación compute/storage | Por segundo compute |
| **Azure Synapse** | Columnar (MPP) | Integración Azure, Spark incluido | Por DWU o serverless |
| **Apache Hive** | Parquet/ORC en HDFS | Open-source, sobre Hadoop | Infra propia |
| **ClickHouse** | Columnar | Ultrarrápido, time-series | Open-source |

---

## Conclusión

!!! success "Resumen OLTP vs OLAP"
    
    **OLTP:**
    - 🎯 Operaciones transaccionales día a día
    - ⚡ Sub-segundo response time
    - 🔒 ACID estricto
    - 📐 Normalizado (3NF)
    - 📝 Row-oriented storage
    
    **OLAP:**
    - 📊 Análisis y reporting
    - 🕐 Segundos-minutos response time
    - 🔓 Eventually consistent OK
    - ⭐ Star/Snowflake schema
    - 📊 Column-oriented storage
    
    **Arquitectura Moderna:**
    ```
    OLTP (PostgreSQL) 
         ↓ ETL (Airflow)
    OLAP (Redshift/BigQuery)
         ↓
    BI Tools (Tableau/PowerBI)
    ```

!!! tip "Mejores Prácticas"
    1. ✅ **No mezclar** OLTP y OLAP en la misma DB
    2. ✅ Usar **ETL incremental** (no full refresh diario)
    3. ✅ Implementar **SCD Type 2** para dimensiones críticas
    4. ✅ **Particionar** fact tables por fecha
    5. ✅ Crear **tablas agregadas** para queries frecuentes
    6. ✅ **Monitorear** queries lentos y optimizar
    7. ✅ **Documentar** lógica de negocio en transformaciones

---

*Fin del Capítulo 8: OLTP vs OLAP*
