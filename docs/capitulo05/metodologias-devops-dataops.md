# CAPÍTULO 5: Metodologías de desarrollo y despliegue en la nube

## 5.1. DevOps

**¿Qué es DevOps?:**

**DevOps** es una combinación de **filosofías culturales**, **prácticas** y **herramientas** que aumenta la capacidad de una organización para distribuir aplicaciones y servicios a gran velocidad.

**Origen del término:**

- **Dev** (Development - Desarrollo)
- **Ops** (Operations - Operaciones)

**Objetivo:** Eliminar silos entre equipos de desarrollo y operaciones, facilitando la entrega continua de software de calidad.

**Filosofía cultural de DevOps:**

**Principios clave:**

1. **Colaboración:** Equipos multidisciplinarios trabajando juntos
2. **Automatización:** Reducir tareas manuales repetitivas
3. **Iteración rápida:** Releases frecuentes en lugar de grandes lanzamientos
4. **Medición continua:** Monitoreo y feedback constante
5. **Aprendizaje:** Cultura de experimentación y mejora continua

**Cambio cultural:**

```
┌────────────────────────────────────────────────────────┐
│              ANTES DE DEVOPS                            │
│                                                        │
│  ┌──────────────┐              ┌──────────────┐      │
│  │  Desarrollo  │              │  Operaciones │      │
│  │  (Dev)       │  ≠           │  (Ops)       │      │
│  │              │              │              │      │
│  │ - Velocidad  │              │ - Estabilidad│      │
│  │ - Features   │              │ - Uptime     │      │
│  │ - Innovación │              │ - Seguridad  │      │
│  └──────────────┘              └──────────────┘      │
│       Silos        →  Fricción  →  Lentitud          │
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│              CON DEVOPS                                 │
│                                                        │
│       ┌─────────────────────────────────┐             │
│       │  Dev + Ops = DevOps              │             │
│       │                                 │             │
│       │  EQUIPO UNIFICADO               │              │
│       │  - Entrega rápida Y estable     │             │
│       │  - Innovación Y seguridad       │             │
│       │  - Automatización integral      │             │
│       └─────────────────────────────────┘             │
│              ↓                                         │
│  Releases frecuentes, alta calidad, bajo riesgo       │
└────────────────────────────────────────────────────────┘
```

**Prácticas de DevOps:**

#### 1. Integración Continua (CI - Continuous Integration)

**Definición:** Práctica de desarrollo de software donde los miembros de un equipo integran su trabajo frecuentemente (varias veces al día). Cada integración se verifica mediante una build automática y pruebas automatizadas.

**Workflow:**

```
┌──────────────────────────────────────────────────────────┐
│         INTEGRACIÓN CONTINUA (CI)                        │
│                                                          │
│  Desarrollador                                           │
│       ↓                                                  │
│  [Código fuente]                                         │
│       ↓                                                  │
│  git commit + push                                       │
│       ↓                                                  │
│  ┌────────────────────────────────────────┐             │
│  │  Repositorio Central (GitHub/GitLab)   │             │
│  └────────────────┬───────────────────────┘             │
│                   ↓                                      │
│  ┌────────────────────────────────────────┐             │
│  │  CI Server (Jenkins, GitLab CI, GitHub │             │
│  │  Actions, CircleCI)                    │             │
│  │                                        │             │
│  │  1. Build automático                   │             │
│  │  2. Unit tests                         │             │
│  │  3. Integration tests                  │             │
│  │  4. Code quality analysis (SonarQube)  │             │
│  │  5. Security scanning                  │             │
│  └────────────────┬───────────────────────┘             │
│                   ↓                                      │
│  ┌────────────────────────────────┐                     │
│  │  Resultado:                    │                     │
│  │  ✅ Build OK → Artefactor     │                     │
│  │  ❌ Build FAIL → Notificación │                     │
│  └────────────────────────────────┘                     │
└──────────────────────────────────────────────────────────┘
```

**Ejemplo: Pipeline CI con GitHub Actions**

```yaml
# .github/workflows/ci.yml

name: Continuous Integration

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    # 1. Checkout código
    - uses: actions/checkout@v3
    
    # 2. Configurar Python
    - name: Set up Python 3.11
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    # 3. Instalar dependencias
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest pytest-cov flake8
    
    # 4. Linting (calidad código)
    - name: Lint with flake8
      run: |
        # Parar build si hay errores críticos
        flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
        # Warnings (no para el build)
        flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
    
    # 5. Unit tests con cobertura
    - name: Run tests with pytest
      run: |
        pytest tests/ --cov=app --cov-report=xml --cov-report=html
    
    # 6. Subir reporte de cobertura
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage.xml
    
    # 7. Build Docker image (si tests pasan)
    - name: Build Docker image
      run: |
        docker build -t mi-app:${{ github.sha }} .
    
    # 8. Scan de seguridad de imagen
    - name: Security scan
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'mi-app:${{ github.sha }}'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    # 9. Notificar resultado
    - name: Notify Slack
      if: always()
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: 'CI Build ${{ job.status }}'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}

# Resultado: Cada push activa este pipeline
# Si falla cualquier paso: Build rojo, notificación inmediata
# Si pasa: Imagen Docker lista para desplegar
```

**Beneficios de CI:**

- ✅ **Detección temprana de errores:** Bugs encontrados minutos después de commit, no semanas
- ✅ **Reducción de riesgo:** Integraciones pequeñas y frecuentes vs grandes merges
- ✅ **Código siempre en estado "releasable"**
- ✅ **Mayor confianza:** Tests automáticos validan cada cambio
- ✅ **Documentación viva:** Pipeline CI documenta cómo buildear el proyecto

#### 2. Entrega Continua / Despliegue Continuo (CD)

**Continuous Delivery vs Continuous Deployment:**

```
┌──────────────────────────────────────────────────────────┐
│  CONTINUOUS DELIVERY (Entrega Continua)                  │
│                                                          │
│  CI → Build → Tests → Staging → [MANUAL APPROVAL] → Production │
│                                         ↑                │
│                                 Decisión humana          │
│                                                          │
│  Código SIEMPRE listo para producción,                  │
│  pero despliegue requiere aprobación manual              │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│  CONTINUOUS DEPLOYMENT (Despliegue Continuo)             │
│                                                          │
│  CI → Build → Tests → Staging → [AUTOMATIC] → Production  │
│                                      ↑                   │
│                              Automático 100%             │
│                                                          │
│  Cada cambio que pasa tests se despliega AUTOMÁTICAMENTE │
│  a producción (sin intervención humana)                  │
└──────────────────────────────────────────────────────────┘
```

**Ejemplo: CD Pipeline completo**

```yaml
# .github/workflows/cd.yml

name: Continuous Deployment

on:
  push:
    branches: [ main ]  # Solo deploys desde main

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    # ... steps de CI (build, test) ...
    
    # Deploy a Staging
    - name: Deploy to Staging
      run: |
        # Desplegar a ECS Staging
        aws ecs update-service \
          --cluster mi-app-staging \
          --service mi-app-service \
          --force-new-deployment
    
    # Smoke tests en Staging
    - name: Smoke tests on Staging
      run: |
        sleep 60  # Esperar que servicio esté up
        curl -f https://staging.mi-app.com/health || exit 1
        pytest tests/smoke/ --env=staging
    
    # Deploy a Producción (automático si staging OK)
    - name: Deploy to Production
      if: success()
      run: |
        # Blue-Green deployment
        # 1. Desplegar nueva versión (Green)
        aws ecs update-service \
          --cluster mi-app-prod \
          --service mi-app-service-green \
          --force-new-deployment
        
        # 2. Esperar health checks
        aws ecs wait services-stable \
          --cluster mi-app-prod \
          --services mi-app-service-green
        
        # 3. Cambiar Target Group (enrutar tráfico a Green)
        aws elbv2 modify-listener \
          --listener-arn $LISTENER_ARN \
          --default-actions TargetGroupArn=$GREEN_TG_ARN
        
        # 4. Verificar métricas durante 10 min
        python scripts/monitor_deployment.py --duration=600
        
        # 5. Si todo OK, eliminar Blue (versión anterior)
        aws ecs update-service \
          --cluster mi-app-prod \
          --service mi-app-service-blue \
          --desired-count 0
    
    # Rollback automático si falla
    - name: Rollback on failure
      if: failure()
      run: |
        # Volver a Blue
        aws elbv2 modify-listener \
          --listener-arn $LISTENER_ARN \
          --default-actions TargetGroupArn=$BLUE_TG_ARN
        
        # Notificar
        echo "DEPLOYMENT FAILED - Rolled back to previous version"
```

**Beneficios de CD:**

- ✅ **Time-to-market reducido:** Features en producción en minutos/horas, no semanas
- ✅ **Menor riesgo:** Cambios pequeños y frecuentes más fáciles de debuggear
- ✅ **Feedback rápido:** Usuarios prueban features nuevas inmediatamente
- ✅ **Menos stress:** Deploys automatizados, repetibles y confiables
- ✅ **Rollback rápido:** Si falla, volver a versión anterior en segundos

#### 3. Arquitectura de Microservicios

**Definición:** Enfoque arquitectónico donde una aplicación se construye como conjunto de pequeños servicios independientes que se comunican vía APIs.

**Monolito vs Microservicios:**

```
┌─────────────────────────────────────────────────────────┐
│              APLICACIÓN MONOLÍTICA                       │
│                                                         │
│  ┌───────────────────────────────────────────────┐     │
│  │  Una Aplicación Grande                        │     │
│  │                                               │     │
│  │  ┌─────────────────┐                          │     │
│  │  │  UI             │                          │     │
│  │  ├─────────────────┤                          │     │
│  │  │  Business Logic │                          │     │
│  │  │  - Users        │                          │     │
│  │  │  - Products     │                          │     │
│  │  │  - Orders       │                          │     │
│  │  │  - Payments     │                          │     │
│  │  ├─────────────────┤                          │     │
│  │  │  Data Access    │                          │     │
│  │  └─────────────────┘                          │     │
│  │           ↓                                   │     │
│  │   ┌──────────────┐                            │     │
│  │   │   Database   │                            │     │
│  │   └──────────────┘                            │     │
│  └───────────────────────────────────────────────┘     │
│                                                         │
│  Desventajas:                                           │
│  ❌ Un cambio pequeño requiere redesplegar TODO        │
│  ❌ Escalado: Solo puedes escalar TODA la app          │
│  ❌ Un fallo puede tumbar toda la aplicación           │
│  ❌ Tecnología única (un lenguaje para todo)           │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│         ARQUITECTURA DE MICROSERVICIOS                   │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐│
│  │  User    │  │ Product  │  │  Order   │  │ Payment ││
│  │ Service  │  │ Service  │  │ Service  │  │ Service ││
│  │          │  │          │  │          │  │         ││
│  │ Node.js  │  │  Python  │  │   Java   │  │   Go    ││
│  │  Port    │  │  Port    │  │  Port    │  │  Port   ││
│  │  3000    │  │  3001    │  │  3002    │  │  3003   ││
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬────┘│
│       │             │             │             │      │
│       └─────────────┴─────────────┴─────────────┘      │
│                     ↓                                   │
│              ┌──────────────┐                           │
│              │  API Gateway │                           │
│              │  (Kong/AWS)  │                           │
│              └──────┬───────┘                           │
│                     ↓                                   │
│              ┌──────────────┐                           │
│              │   Clientes   │                           │
│              │  (Web/Móvil) │                           │
│              └──────────────┘                           │
│                                                         │
│  Ventajas:                                              │
│  ✅ Deploy independiente (User Service sin tocar resto)│
│  ✅ Escala independiente (10x Payment, 2x User)        │
│  ✅ Fallos aislados (Payment cae, resto funciona)      │
│  ✅ Tecnología heterogénea (mejor tool para cada job)  │
│  ✅ Equipos autónomos (cada equipo = 1 servicio)       │
└─────────────────────────────────────────────────────────┘
```

**Ejemplo: E-commerce con Microservicios**

```python
# user-service/app.py (Puerto 3000)
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/users/<user_id>', methods=['GET'])
def get_user(user_id):
    # Lógica de usuario
    user = database.get_user(user_id)
    return jsonify(user)

@app.route('/users', methods=['POST'])
def create_user():
    data = request.json
    user_id = database.create_user(data)
    return jsonify({'user_id': user_id}), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3000)

# ─────────────────────────────────────────────────────────

# product-service/app.py (Puerto 3001)
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/products', methods=['GET'])
def list_products():
    products = database.get_all_products()
    return jsonify(products)

@app.route('/products/<product_id>', methods=['GET'])
def get_product(product_id):
    product = database.get_product(product_id)
    return jsonify(product)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3001)

# ─────────────────────────────────────────────────────────

# order-service/app.py (Puerto 3002)
from flask import Flask, jsonify, request
import requests  # Para comunicarse con otros servicios

app = Flask(__name__)

USER_SERVICE_URL = 'http://user-service:3000'
PRODUCT_SERVICE_URL = 'http://product-service:3001'
PAYMENT_SERVICE_URL = 'http://payment-service:3003'

@app.route('/orders', methods=['POST'])
def create_order():
    data = request.json
    user_id = data['user_id']
    product_ids = data['product_ids']
    
    # 1. Verificar usuario existe
    user_response = requests.get(f'{USER_SERVICE_URL}/users/{user_id}')
    if user_response.status_code != 200:
        return jsonify({'error': 'User not found'}), 404
    
    # 2. Verificar productos y calcular total
    total = 0
    for prod_id in product_ids:
        prod_response = requests.get(f'{PRODUCT_SERVICE_URL}/products/{prod_id}')
        if prod_response.status_code == 200:
            product = prod_response.json()
            total += product['price']
    
    # 3. Crear orden
    order_id = database.create_order(user_id, product_ids, total)
    
    # 4. Procesar pago (llamar a Payment Service)
    payment_response = requests.post(
        f'{PAYMENT_SERVICE_URL}/payments',
        json={'order_id': order_id, 'amount': total}
    )
    
    if payment_response.status_code != 201:
        # Rollback orden
        database.delete_order(order_id)
        return jsonify({'error': 'Payment failed'}), 400
    
    return jsonify({'order_id': order_id, 'status': 'confirmed'}), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3002)

# ─────────────────────────────────────────────────────────

# payment-service/app.go (Puerto 3003)
package main

import (
    "encoding/json"
    "net/http"
)

type PaymentRequest struct {
    OrderID string  `json:"order_id"`
    Amount  float64 `json:"amount"`
}

func processPayment(w http.ResponseWriter, r *http.Request) {
    var req PaymentRequest
    json.NewDecoder(r.Body).Decode(&req)
    
    // Lógica de pago (Stripe, PayPal, etc.)
    success := chargeCard(req.Amount)
    
    if success {
        w.WriteHeader(http.StatusCreated)
        json.NewEncoder(w).Encode(map[string]string{
            "status": "paid",
            "order_id": req.OrderID,
        })
    } else {
        w.WriteHeader(http.StatusBadRequest)
        json.NewEncoder(w).Encode(map[string]string{
            "error": "Payment declined",
        })
    }
}

func main() {
    http.HandleFunc("/payments", processPayment)
    http.ListenAndServe(":3003", nil)
}
```

**Docker Compose para desarrollo local:**

```yaml
# docker-compose.yml

version: '3.8'

services:
  user-service:
    build: ./user-service
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=user-db
  
  product-service:
    build: ./product-service
    ports:
      - "3001:3001"
    environment:
      - DB_HOST=product-db
  
  order-service:
    build: ./order-service
    ports:
      - "3002:3002"
    depends_on:
      - user-service
      - product-service
      - payment-service
  
  payment-service:
    build: ./payment-service
    ports:
      - "3003:3003"
  
  api-gateway:
    image: kong:latest
    ports:
      - "8000:8000"  # Punto de entrada único
    environment:
      - KONG_DATABASE=off
      - KONG_PROXY_ACCESS_LOG=/dev/stdout
      - KONG_ADMIN_ACCESS_LOG=/dev/stdout
    volumes:
      - ./kong.yml:/usr/local/kong/declarative/kong.yml

# Usar: docker-compose up
# Acceder: http://localhost:8000/users, /products, /orders
```

**Despliegue en Kubernetes:**

```yaml
# kubernetes/deployments.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3  # 3 instancias para alta disponibilidad
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: mi-registry.azurecr.io/user-service:v1.2.3
        ports:
        - containerPort: 3000
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP  # Interno, solo accesible dentro del cluster

# Similar para product-service, order-service, payment-service...
```

**Comunicación entre Microservicios:**

1. **Síncrona (REST API):**
   - HTTP/HTTPS requests
   - Pros: Simple, fácil debugging
   - Cons: Acoplamiento temporal (si servicio B cae, A falla)

2. **Asíncrona (Message Queue):**
   - RabbitMQ, Apache Kafka, AWS SQS
   - Pros: Desacoplamiento, tolerancia a fallos
   - Cons: Mayor complejidad

```python
# Ejemplo async con RabbitMQ

# order-service publica evento
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('rabbitmq'))
channel = connection.channel()
channel.queue_declare(queue='orders')

# Publicar evento
channel.basic_publish(
    exchange='',
    routing_key='orders',
    body=json.dumps({'order_id': order_id, 'user_id': user_id})
)

# ─────────────────────────────────────────────────────────

# email-service escucha eventos
def callback(ch, method, properties, body):
    order = json.loads(body)
    send_confirmation_email(order['user_id'], order['order_id'])
    print(f"Email sent for order {order['order_id']}")

channel.basic_consume(queue='orders', on_message_callback=callback, auto_ack=True)
channel.start_consuming()

# Ventaja: Si email-service cae, eventos se acumulan en cola
# Cuando vuelve, procesa todos los pendientes
```

#### 4. Infraestructura como Código (IaC)

**Definición:** Gestionar y aprovisionar infraestructura mediante archivos de configuración legibles por máquina, en lugar de configuración manual o herramientas interactivas.

**Herramientas principales:**

- **Terraform** (HashiCorp) - Multi-cloud, agnóstico
- **AWS CloudFormation** - Específico AWS
- **Azure Resource Manager (ARM) Templates** - Específico Azure
- **Pulumi** - IaC con lenguajes de programación reales (Python, TypeScript)

**Ejemplo: Terraform para AWS**

```hcl
# main.tf - Infraestructura completa como código

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "eu-south-2"  # España
}

# 1. VPC (Red privada)
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  
  tags = {
    Name        = "production-vpc"
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}

# 2. Subnets (públicas y privadas)
resource "aws_subnet" "public" {
  count = 3  # 3 subnets públicas (1 por AZ)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 1}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${count.index + 1}"
    Type = "public"
  }
}

resource "aws_subnet" "private" {
  count = 3  # 3 subnets privadas
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 101}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "private-subnet-${count.index + 1}"
    Type = "private"
  }
}

# 3. Security Group (Firewall)
resource "aws_security_group" "web" {
  name        = "allow-web-traffic"
  description = "Allow HTTP/HTTPS inbound"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# 4. EC2 Instance con Auto Scaling
resource "aws_launch_template" "web" {
  name_prefix   = "web-server-"
  image_id      = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2
  instance_type = "t3.medium"
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = base64encode(<<-EOF
    #!/bin/bash
    yum update -y
    yum install -y nginx
    systemctl start nginx
    systemctl enable nginx
  EOF
  )
  
  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "web-server"
    }
  }
}

resource "aws_autoscaling_group" "web" {
  name                = "web-asg"
  vpc_zone_identifier = aws_subnet.private[*].id
  min_size            = 2
  max_size            = 10
  desired_capacity    = 3
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
  
  target_group_arns = [aws_lb_target_group.web.arn]
  
  tag {
    key                 = "Environment"
    value               = "production"
    propagate_at_launch = true
  }
}

# 5. Load Balancer (distribuir tráfico)
resource "aws_lb" "web" {
  name               = "web-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web.id]
  subnets            = aws_subnet.public[*].id
}

resource "aws_lb_target_group" "web" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  
  health_check {
    path                = "/health"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }
}

resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.web.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-2016-08"
  certificate_arn   = aws_acm_certificate.main.arn
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

# 6. RDS (Base de datos)
resource "aws_db_instance" "main" {
  identifier = "production-db"
  
  engine         = "postgres"
  engine_version = "15.3"
  instance_class = "db.r5.xlarge"
  
  allocated_storage     = 100
  storage_type          = "gp3"
  storage_encrypted     = true
  
  db_name  = "myapp"
  username = "admin"
  password = var.db_password  # Desde variable segura
  
  multi_az               = true  # Alta disponibilidad
  backup_retention_period = 30
  
  vpc_security_group_ids = [aws_security_group.database.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  skip_final_snapshot = false
  final_snapshot_identifier = "production-db-final-snapshot"
}

# 7. S3 Bucket para assets
resource "aws_s3_bucket" "assets" {
  bucket = "mi-app-assets-production"
  
  tags = {
    Environment = "production"
  }
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# 8. CloudFront (CDN)
resource "aws_cloudfront_distribution" "assets" {
  origin {
    domain_name = aws_s3_bucket.assets.bucket_regional_domain_name
    origin_id   = "S3-assets"
  }
  
  enabled             = true
  default_root_object = "index.html"
  
  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-assets"
    
    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
    
    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }
  
  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
  
  viewer_certificate {
    cloudfront_default_certificate = true
  }
}

# Outputs (info después de crear)
output "load_balancer_dns" {
  value = aws_lb.web.dns_name
  description = "DNS del Load Balancer para acceder a la aplicación"
}

output "database_endpoint" {
  value = aws_db_instance.main.endpoint
  description = "Endpoint de la base de datos"
  sensitive = true
}
```

**Usar Terraform:**

```bash
# 1. Inicializar
terraform init

# 2. Ver qué se va a crear (dry-run)
terraform plan

# Output ejemplo:
# Terraform will perform the following actions:
#
#   # aws_vpc.main will be created
#   + resource "aws_vpc" "main" {
#       + cidr_block = "10.0.0.0/16"
#       ...
#   }
#
# Plan: 25 to add, 0 to change, 0 to destroy.

# 3. Aplicar cambios (crear infraestructura REAL)
terraform apply

# Confirmación: yes
# ...
# Apply complete! Resources: 25 added, 0 changed, 0 destroyed.
#
# Outputs:
# load_balancer_dns = "web-lb-123456789.eu-south-2.elb.amazonaws.com"

# 4. Modificar: Editar .tf files y volver a aplicar
# Terraform detecta diferencias y aplica solo cambios necesarios

# 5. Destruir todo (cuidado!)
terraform destroy
```

**Beneficios de IaC:**

- ✅ **Versionado:** Infraestructura en Git, historial completo
- ✅ **Reproducible:** Crear entornos idénticos (dev, staging, prod)
- ✅ **Documentación:** Código ES la documentación
- ✅ **Disaster Recovery:** Recrear infraestructura en minutos
- ✅ **Testing:** Crear infra temporal para tests, destruir después
- ✅ **Colaboración:** Code reviews para cambios de infraestructura

#### 5. Monitoreo y Logging

**Observabilidad** = Logging + Monitoring + Tracing

**Stack típico:**

- **Logs:** ELK (Elasticsearch, Logstash, Kibana) o EFK (Fluentd en lugar de Logstash)
- **Métricas:** Prometheus + Grafana
- **Tracing:** Jaeger, Zipkin
- **Alertas:** Alertmanager, PagerDuty

**Ejemplo: Monitoreo completo**

```yaml
# docker-compose-monitoring.yml

version: '3.8'

services:
  # Aplicación (genera logs y métricas)
  app:
    image: mi-app:latest
    ports:
      - "8080:8080"
    environment:
      - PROMETHEUS_ENABLED=true
      - JAEGER_AGENT_HOST=jaeger
    logging:
      driver: "fluentd"
      options:
        fluentd-address: "localhost:24224"
        tag: "mi-app"
  
  # Prometheus: Recolectar métricas
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  
  # Grafana: Visualizar métricas
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
  
  # Elasticsearch: Almacenar logs
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
  
  # Fluentd: Recolectar y routear logs
  fluentd:
    image: fluent/fluentd:latest
    volumes:
      - ./fluentd.conf:/fluentd/etc/fluent.conf
    ports:
      - "24224:24224"
    depends_on:
      - elasticsearch
  
  # Kibana: Visualizar logs
  kibana:
    image: docker.elastic.co/kibana/kibana:8.10.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
  
  # Jaeger: Distributed tracing
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"  # UI
      - "6831:6831/udp"  # Agent

volumes:
  grafana-data:
```

**Configuración Prometheus:**

```yaml
# prometheus.yml

global:
  scrape_interval: 15s  # Recolectar métricas cada 15 seg

scrape_configs:
  - job_name: 'mi-app'
    static_configs:
      - targets: ['app:8080']
  
  - job_name: 'node-exporter'  # Métricas de sistema (CPU, RAM, disk)
    static_configs:
      - targets: ['node-exporter:9100']
```

** Aplicación expone métricas:**

```python
# app.py - Flask con Prometheus metrics

from flask import Flask
from prometheus_client import Counter, Histogram, generate_latest, REGISTRY
import time

app = Flask(__name__)

# Definir métricas
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint']
)

@app.route('/api/users')
def get_users():
    start = time.time()
    
    # Lógica de negocio
    users = database.get_users()
    
    # Registrar métricas
    REQUEST_COUNT.labels(method='GET', endpoint='/api/users', status=200).inc()
    REQUEST_LATENCY.labels(method='GET', endpoint='/api/users').observe(time.time() - start)
    
    return jsonify(users)

@app.route('/metrics')
def metrics():
    # Endpoint que Prometheus scrapea
    return generate_latest(REGISTRY)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

**Dashboard Grafana:**

Queries ejemplo:
```promql
# Requests por segundo
rate(http_requests_total[5m])

# Latencia p95
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# CPU usage
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**Alertas (Alertmanager):**

```yaml
# alertmanager.yml

route:
  receiver: 'slack'
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h

receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#alerts'
        text: 'Alert: {{ .GroupLabels.alertname }}'

# alerts.yml (reglas)
groups:
  - name: example
    rules:
      # Alerta si API latency > 1 segundo
      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        annotations:
          summary: "High API latency detected"
          description: "p95 latency is {{ $value }}s"
      
      # Alerta si error rate > 5%
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m
        annotations:
          summary: "High error rate"
          description: "Error rate is {{ $value | humanizePercentage }}"
```

#### 6. Comunicación y Colaboración

**Prácticas clave:**

- **ChatOps:** Slack/Teams con bots para deploys, alertas, status
- **Blameless Postmortems:** Cuando falla algo, documentar sin culpar personas
- **Pair Programming:** Dos desarrolladores, un teclado
- **Code Reviews:** Pull Requests revisadas por peers
- **Shared On-Call:** Dev y Ops comparten guardias

**Beneficios de DevOps:**

| Beneficio | Descripción | Impacto |
|-----------|-------------|---------|
| **Velocidad** | Releases más frecuentes | Weeks → Hours/Days |
| **Entrega Rápida** | Features a producción rápidamente | Time-to-market -50% |
| **Confiabilidad** | CI/CD automático reduce errores humanos | Defectos -60% |
| **Escala** | Automatización permite gestionar sistemas complejos | Sin límite de tamaño |
| **Colaboración Mejorada** | Equipos unificados, menos silos | Satisfacción +40% |
| **Seguridad** | Security integrada desde el inicio (DevSecOps) | Vulnerabilidades -70% |

**Datos reales (State of DevOps Report 2023):**

- Organizaciones elite DevOps:
  - **208x** más deploys que low performers
  - **106x** más rápido desde commit a deploy
  - **7x** menor tasa de fallos en cambios
  - **2,604x** más rápido recovery de incidentes

---

## 5.2. DataOps

**¿Qué es DataOps?:**

**DataOps** es una metodología automatizada y basada en procesos utilizada por equipos de análisis y datos para mejorar la calidad y reducir el tiempo de ciclo del análisis de datos.

**Inspirado en:** DevOps + Agile + Lean Manufacturing

**Objetivo:** Mejorar la velocidad, calidad y colaboración en el desarrollo de soluciones de datos y analytics.

**¿Por qué DataOps?:**

**Problemas tradicionales en proyectos de datos:**

1. **Silos de datos:** Datos fragmentados en múltiples sistemas
2. **Calidad de datos inconsistente:** ETLs manuales propensos a errores
3. **Largo tiempo de entrega:** Meses para crear un nuevo pipeline
4. **Falta de colaboración:** Data Engineers, Data Scientists, Analysts trabajando aislados
5. **Difícil reproducibilidad:** "Funciona en mi máquina"

**DataOps resuelve estos problemas aplicando principios DevOps a datos.**

**Ciclo de vida DataOps:**

```
┌──────────────────────────────────────────────────────────┐
│            PIPELINE DATAOPS (3 Ambientes)                │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │  1. SANDBOX (Exploración y Desarrollo)             │ │
│  │                                                    │ │
│  │  - Data Scientists exploran datos                 │ │
│  │  - Prototipado rápido de modelos                  │ │
│  │  - Notebooks (Jupyter, Databricks)                │ │
│  │  - Datos de muestra/sintéticos                    │ │
│  └────────────────┬───────────────────────────────────┘ │
│                   ↓ [Git commit]                        │
│  ┌────────────────────────────────────────────────────┐ │
│  │  2. STAGING (Testing y Validación)                 │ │
│  │                                                    │ │
│  │  - Tests automatizados (unit, integration)        │ │
│  │  - Data quality checks                            │ │
│  │  - Performance testing                            │ │
│  │  - Datos reales (subset)                          │ │
│  │  - CI/CD pipeline ejecuta                         │ │
│  └────────────────┬───────────────────────────────────┘ │
│                   ↓ [Approval]                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │  3. PRODUCTION (Operación)                         │ │
│  │                                                    │ │
│  │  - Pipelines orquestados (Airflow, Dagster)       │ │
│  │  - Monitoreo continuo (drift detection)           │ │
│  │  - Alertas automáticas                            │ │
│  │  - Datos completos                                │ │
│  │  - SLAs garantizados                              │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
│  FEEDBACK: Métricas de producción → Sandbox (iteración) │
└──────────────────────────────────────────────────────────┘
```

**Elementos esenciales de DataOps:**

DataOps requiere **5 elementos clave:**

#### 1. Enabling Technologies (Tecnologías Habilitadoras)

- **Version Control** para datos y código (Git, DVC)
- **Data Pipelines** orquestados (Apache Airflow, Dagster, Prefect)
- **Testing frameworks** para datos (Great Expectations, dbt tests)
- **CI/CD** para pipelines de datos
- **Observability** (data lineage, quality monitoring)

#### 2. Adaptive Architecture (Arquitectura Adaptable)

- **Modular:** Pipelines como bloques reutilizables
- **Scalable:** De GBs a PBs sin rediseño
- **Cloud-native:** Aprovechar elasticidad del cloud
- **Technology-agnostic:** No lock-in a una herramienta

#### 3. Data Enrichment (Enriquecimiento de Datos)

- **Data quality:** Validaciones automatizadas
- **Metadata management:** Catalogar y documentar datasets
- **Data lineage:** Trazar origen y transformaciones
- **Semantic layer:** Business metrics consistentes

#### 4. Methodology (Metodología)

- **Agile:** Sprints cortos, entregas incrementales
- **Lean:** Eliminar desperdicios en procesos
- **Continuous improvement:** Retrospectivas y mejoras
- **Automation-first:** Automatizar todo lo repetible

#### 5. Culture & People (Cultura y Personas)

- **Collaboration:** DataOps team = Data Engineers + Scientists + Analysts + DevOps
- **Shared responsibility:** Todos responsables de calidad de datos
- **Blameless culture:** Aprender de fallos sin culpar
- **Transparency:** Dashboards de estado de pipelines visibles para todos

**DataOps vs DevOps: diferencias clave:**

| Dimensión | DevOps | DataOps |
|-----------|--------|---------|
| **Objetivo Principal** | Entregar software rápido y confiable | Entregar insights de datos rápidos y confiables |
| **Artefacto** | Código (aplicaciones) | Datos + Código (pipelines) |
| **Testing** | Unit, integration, E2E tests | Data quality, schema validation, statistical tests |
| **Despliegue** | Blue-green, canary deployments | Data pipeline deployments, backfills |
| **Rollback** | Revert código a versión anterior | Revert transformaciones, reprocess data |
| **Monitoreo** | Server health, API latency, errors | Data freshness, completeness, accuracy |
| **Equipo** | Devs + Ops | Data Engineers + Scientists + Analysts + DevOps |
| **Complejidad** | Código es determinístico | Datos son probabilísticos y cambiantes |
| **Scope** | Ciclo de vida del software | Ciclo de vida de datos end-to-end |

**Clave:** DataOps es MÁS AMPLIO que DevOps. No solo tecnología, sino cómo los datos fluyen por toda la organización.

**Ejemplo: pipeline DataOps completo:**

**Caso de uso:** Dashboard de métricas COVID-19 actualizado diariamente.

```python
# dags/covid_pipeline.py - Apache Airflow DAG

from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.amazon.aws.operators.glue import GlueJobOperator
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor
from datetime import datetime, timedelta
import pandas as pd
import great_expectations as ge

default_args = {
    'owner': 'data-team',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': True,
    'email': ['data-alerts@hospital.com'],
    'retries': 3,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'covid_daily_pipeline',
    default_args=default_args,
    description='Pipeline diario de datos COV ID',
    schedule_interval='0 6 * * *',  # Cada día a las 6 AM
    catchup=False,
    tags=['covid', 'daily', 'production']
)

# TAREA 1: Esperar que datos estén disponibles
wait_for_data = S3KeySensor(
    task_id='wait_for_raw_data',
    bucket_name='covid-data-raw',
    bucket_key='incoming/covid_casos_{{ ds }}.csv',  # {{ ds }} = fecha ejecución
    aws_conn_id='aws_default',
    timeout=3600,  # Max 1 hora de espera
    poke_interval=60,  # Verificar cada 60 segundos
    dag=dag
)

# TAREA 2: Validar calidad de datos RAW
def validate_raw_data(**context):
    \"\"\"Data quality checks con Great Expectations\"\"\"
    import boto3
    import io
    
    s3 = boto3.client('s3')
    date = context['ds']
    
    # Descargar CSV
    obj = s3.get_object(Bucket='covid-data-raw', Key=f'incoming/covid_casos_{date}.csv')
    df = pd.read_csv(io.BytesIO(obj['Body'].read()))
    
    # Convertir a Great Expectations dataset
    ge_df = ge.from_pandas(df)
    
    # Expectativas (reglas de calidad)
    results = ge_df.expect_table_row_count_to_be_between(min_value=1000, max_value=100000)
    assert results['success'], f\"Row count out of range: {results}\"
    
    results = ge_df.expect_column_values_to_not_be_null('fecha')
    assert results['success'], \"Column 'fecha' has null values\"
    
    results = ge_df.expect_column_values_to_be_between('casos_confirmados', min_value=0, max_value=1000000)
    assert results['success'], f\"casos_confirmados out of range: {results}\"
    
    results = ge_df.expect_column_values_to_be_of_type('ccaa', 'object')
    assert results['success'], \"ccaa column type mismatch\"
    
    # Si llega aquí, datos válidos
    print(f\"✅ Data quality checks passed for {date}\")
    
    # Mover a zona validated
    s3.copy_object(
        Bucket='covid-data-raw',
        CopySource={'Bucket': 'covid-data-raw', 'Key': f'incoming/covid_casos_{date}.csv'},
        Key=f'validated/covid_casos_{date}.csv'
    )

validate_data = PythonOperator(
    task_id='validate_raw_data',
    python_callable=validate_raw_data,
    provide_context=True,
    dag=dag
)

# TAREA 3: Transformación ETL con AWS Glue
transform_data = GlueJobOperator(
    task_id='transform_data',
    job_name='covid_etl_job',
    script_location='s3://covid-data-scripts/etl/transform_covid.py',
    s3_bucket='covid-data-glue-temp',
    iam_role_name='GlueServiceRole',
    create_job_kwargs={
        'GlueVersion': '4.0',
        'NumberOfWorkers': 10,
        'WorkerType': 'G.1X',
        'Arguments': {
            '--INPUT_PATH': 's3://covid-data-raw/validated/covid_casos_{{ ds }}.csv',
            '--OUTPUT_PATH': 's3://covid-data-trusted/covid_cases/year={{ execution_date.year }}/month={{ execution_date.month }}/',
            '--DATE': '{{ ds }}'
        }
    },
    dag=dag
)

# TAREA 4: Agregar datos (refinado)
def aggregate_data(**context):
    \"\"\"Agregar por CCAA para dashboard\"\"\"
   import boto3
    from pyspark.sql import SparkSession
    
    spark = SparkSession.builder.appName("COVID-Aggregation").getOrCreate()
    date = context['ds']
    year = context['execution_date'].year
    month = context['execution_date'].month
    
    # Leer datos transformados (Parquet)
    df =spark.read.parquet(f's3://covid-data-trusted/covid_cases/year={year}/month={month}/')
    
    # Agregaciones
    agg_ccaa = df.groupBy('ccaa').agg(
        sum('casos_confirmados').alias('total_casos'),
        sum('hospitalizados').alias('total_hospitalizados'),
        sum('uci').alias('total_uci'),
        sum('fallecidos').alias('total_fallecidos'),
        (sum('fallecidos') * 100 / sum('casos_confirmados')).alias('letalidad')
    )
    
    # Guardar en zona refinada
    agg_ccaa.write.mode('overwrite').parquet(
        f's3://covid-data-refined/aggregations/by_ccaa/date={date}/'
    )
    
    # También cargar a Redshift para BI tool
    agg_ccaa.write \
        .format('jdbc') \
        .option('url', 'jdbc:redshift://redshift-cluster:5439/covid_dw') \
        .option('dbtable', 'metrics.covid_by_ccaa') \
        .option('user', 'etl_user') \
        .option('password', context['var']['value'].get('redshift_password')) \
        .option('driver', 'com.amazon.redshift.jdbc.Driver') \
        .mode('append') \
        .save()
    
    print(f\"✅ Aggregation complete for {date}\")

aggregate = PythonOperator(
    task_id='aggregate_data',
    python_callable=aggregate_data,
    provide_context=True,
    dag=dag
)

# TAREA 5: Validar datos REFINED
def validate_refined_data(**context):
    \"\"\"Validar resultados finales\"\"\"
    import boto3
    from pyspark.sql import SparkSession
    
    spark = SparkSession.builder.appName("Validation").getOrCreate()
    date = context['ds']
    
    df = spark.read.parquet(f's3://covid-data-refined/aggregations/by_ccaa/date={date}/')
    
    # Validaciones
    assert df.count() == 19, f\"Expected 19 CCAA, got {df.count()}\"  # 17 CCAA + 2 ciudades autónomas
    
    # No puede haber letalidad > 10% (alerta de anomalía)
    high_lethality = df.filter(df.letalidad > 10.0)
    if high_lethality.count() > 0:
        # No fallar, pero enviar alerta
        print(f\"⚠️ WARNING: High lethality detected in {high_lethality.collect()}\")
        # send_slack_alert(f\"High lethality in {date}\")
    
    print(f\"✅ Refined data validation passed for {date}\")

validate_refined = PythonOperator(
    task_id='validate_refined_data',
    python_callable=validate_refined_data,
    provide_context=True,
    dag=dag
)

# TAREA 6: Refrescar dashboard
def refresh_dashboard(**context):
    \"\"\"Notificar a QuickSight/Tableau que datos están listos\"\"\"
    import boto3
    
    quicksight = boto3.client('quicksight')
    
    # Refrescar dataset de QuickSight
    quicksight.create_ingestion(
        DataSetId='covid-dashboard-dataset',
        IngestionId=f'ingestion-{context["ds"]}',
        AwsAccountId='123456789012'
    )
    
    print(f\"✅ Dashboard refreshed for {context['ds']}\")

refresh = PythonOperator(
    task_id='refresh_dashboard',
    python_callable=refresh_dashboard,
    provide_context=True,
    dag=dag
)

# TAREA 7: Data Quality Report
def generate_dq_report(**context):
    \"\"\"Generar reporte de calidad de datos\"\"\"
    # Métricas de calidad
    report = {
        'date': context['ds'],
        'pipeline': 'covid_daily',
        'status': 'SUCCESS',
        'rows_processed': 15234,
        'data_quality_score': 0.998,
        'issues': []
    }
    
    # Guardar en S3
    import boto3
    import json
    
    s3 = boto3.client('s3')
    s3.put_object(
        Bucket='covid-data-reports',
        Key=f'dq_reports/covid_daily_{context["ds"]}.json',
        Body=json.dumps(report)
    )

dq_report = PythonOperator(
    task_id='generate_dq_report',
    python_callable=generate_dq_report,
    provide_context=True,
    dag=dag
)

# DEFINIR DEPENDENCIAS (orden de ejecución)
wait_for_data >> validate_data >> transform_data >> aggregate >> validate_refined >> [refresh, dq_report]

# Visualización del DAG:
#
#  wait_for_data
#       ↓
#  validate_data
#       ↓
#  transform_data
#       ↓
#  aggregate
#       ↓
#  validate_refined
#       ↓
#  ┌────┴─────┐
#  ↓          ↓
# refresh   dq_report
```

**Monitoreo del Pipeline:**

```python
# monitoring/data_observability.py

from airflow.providers.amazon.aws.hooks.cloudwatch import CloudWatchHook
import time

class DataObservability:
    def __init__(self):
        self.cw = CloudWatchHook().get_conn()
    
    def track_data_freshness(self, dataset, timestamp):
        \"\"\"Cuán recientes son los datos\"\"\"
        age_seconds = time.time() - timestamp
        
        self.cw.put_metric_data(
            Namespace='DataOps/COVID',
            MetricData=[{
                'MetricName': 'DataFreshness',
                'Value': age_seconds,
                'Unit': 'Seconds',
                'Dimensions': [{'Name': 'Dataset', 'Value': dataset}]
            }]
        )
        
        # Alerta si datos > 2 horas de antigüedad
        if age_seconds > 7200:
            self.send_alert(f\"Datos obsoletos: {dataset}\")
    
    def track_data_completeness(self, expected_rows, actual_rows):
        \"\"\"% de datos completos\"\"\"
        completeness = (actual_rows / expected_rows) * 100
        
        self.cw.put_metric_data(
            Namespace='DataOps/COVID',
            MetricData=[{
                'MetricName': 'DataCompleteness',
                'Value': completeness,
                'Unit': 'Percent'
            }]
        )
        
        if completeness < 95:
            self.send_alert(f\"Completeness bajo: {completeness:.2f}%\")
    
    def track_data_accuracy(self, df):
        \"\"\"Validaciones estadísticas\"\"\"
        # Ejemplo: Z-score para detectar outliers
        from scipy import stats
        import numpy as np
        
        z_scores = np.abs(stats.zscore(df['casos_confirmados'].dropna()))
        outliers = (z_scores > 3).sum()
        
        self.cw.put_metric_data(
            Namespace='DataOps/COVID',
            MetricData=[{
                'MetricName': 'DataOutliers',
                'Value': outliers,
                'Unit': 'Count'
            }]
        )

# Usar en pipeline:
obs = DataObservability()
obs.track_data_freshness('covid_casos', df['timestamp'].max())
obs.track_data_completeness(expected_rows=15000, actual_rows=len(df))
obs.track_data_accuracy(df)
```

**Beneficios de DataOps:**

| Beneficio | Descripción | Impacto |
|-----------|-------------|---------|
| **Velocidad** | Pipelines nuevos en días, no meses | Time-to-insight -70% |
| **Calidad** | Data quality automatizada | Errores de datos -80% |
| **Reproducibilidad** | Pipelines versionados y testeados | \"Funciona en mi máquina\" eliminado |
| **Colaboración** | Equipos de datos unificados | Productividad +50% |
| **Confianza** | Validaciones continuas, lineage claro | Trust en datos +90% |
| **Escala** | De GBs a PBs sin rediseño | Sin límite |

---

## 5.3. MLOps y Otras Metodologías

**MLOps (Machine Learning Operations):**

**Definición:** Conjunto de prácticas para desplegar y mantener modelos de Machine Learning en producción de forma confiable y eficiente.

**MLOps = DevOps + DataOps + ML**

**Desafíos específicos de ML:**

1. **Modelos se degradan con el tiempo (drift):** Datos cambian, modelo pierde precisión
2. **Experimentación:** Cientos de experimentos antes de encontrar mejor modelo
3. **Reproducibilidad:** Mismo código con diferentes datos → diferentes resultados
4. **Monitoreo complejo:** No solo latencia del API, sino accuracy, bias, drift

**Ciclo de Vida MLOps:**

```
┌──────────────────────────────────────────────────────────────┐
│                  CICLO DE VIDA MLOPS                         │
│                                                              │
│  1. DESARROLLO (Data Scientist)                             │
│       ├─ Exploración de datos (Jupyter)                     │
│       ├─ Feature engineering                                │
│       ├─ Entrenamiento de modelos (experimentos)            │
│       └─ Evaluación (accuracy, precision, recall)           │
│                      ↓                                       │
│  2. PREPARACIÓN (ML Engineer)                               │
│       ├─ Versionado de modelo (MLflow, Weights & Biases)    │
│       ├─ Containerización (Docker)                          │
│       ├─ Testing (unit, integration, A/B)                   │
│       └─ CI/CD pipeline                                     │
│                      ↓                                       │
│  3. DESPLIEGUE (MLOps Engineer)                             │
│       ├─ Model serving (SageMaker, Vertex AI, Seldon)       │
│       ├─ API (REST/gRPC)                                    │
│       ├─ Scaling (auto-scaling según demanda)               │
│       └─ Canary/Blue-Green deployment                       │
│                      ↓                                       │
│  4. MONITOREO (Todos)                                        │
│       ├─ Performance (latencia, throughput)                 │
│       ├─ Accuracy en producción (ground truth)              │
│       ├─ Data drift detection                               │
│       ├─ Model drift detection                              │
│       └─ Fairness & Bias monitoring                         │
│                      ↓                                       │
│  5. REENTRENAMIENTO (Automated)                             │
│       ├─ Trigger cuando drift detectado                      │
│       ├─ Pipeline automático: datos nuevos → entrenar       │
│       ├─ Validación de nuevo modelo                         │
│       └─ Deployment si mejora métricas                      │
│                      ↓                                       │
│  [LOOP BACK TO STEP 3]                                       │
└──────────────────────────────────────────────────────────────┘
```

**No cubre al completo por limitaciones de espacio, pero los conceptos clave están establecidos.**

**ModelOps:**

**Definición:** Extensión de MLOps para gestionar TODOS los tipos de modelos analíticos, no solo ML (incluye modelos estadísticos, reglas de negocio, etc.).

**AIOps (AI for IT Operations):**

**Definición:** Uso de AI/ML para automatizar y mejorar operaciones de IT (detect anomalías, predict failures, auto-remediation).

---

## 5.4. Conclusión del Capítulo

**Resumen de Metodologías:**

| Metodología | Alcance | Objetivo Principal |
|-------------|---------|-------------------|
| **DevOps** | Desarrollo y operaciones de software | Entrega continua de software |
| **DataOps** | Pipelines y análisis de datos | Entrega continua de insights de datos |
| **MLOps** | Modelos de Machine Learning | Despliegue y monitoreo de modelos ML en producción |
| **ModelOps** | Todos los modelos analíticos | Gobernar lifecycle completo de modelos |
| **AIOps** | Operaciones de IT | Automatizar IT ops con AI |

Todas comparten principios:
- ✅ Automatización
- ✅ Colaboración
- ✅ Monitoreo continuo
- ✅ Iteración rápida
- ✅ Cultura de mejora continua

---

