# Innovatech Chile — DevOps EP3

Sistema de Despachos contenedorizado y orquestado en AWS ECS Fargate con pipeline CI/CD automatizado mediante GitHub Actions.

## Descripción

Proyecto desarrollado en la asignatura ISY1101 — Introducción a Herramientas DevOps (DuocUC).
Evoluciona desde la contenedorización en EC2 (EP2) hacia orquestación productiva en AWS ECS Fargate (EP3),
implementando escalabilidad automática, observabilidad con CloudWatch y despliegue continuo.

**Empresa:** Innovatech Chile  
**Integrantes:** Jean Flores · Christian Fuentes · Emmanuel Valenzuela

---

## Arquitectura
GitHub (rama deploy)

│

▼

GitHub Actions (CI/CD)

│

├── Build imagen Docker

├── Push a Amazon ECR

└── Deploy automático en Amazon ECS Fargate

│

┌─────────┴─────────┐

│                   │

Frontend Service     Backend Service

React + Nginx        Spring Boot

Puerto 80            Puerto 8081

IP: 32.198.108.172   (interno)

│                   │

└────── VPC default (172.31.0.0/16) ──────┘

Security Group: puertos 80 y 8081

Región: us-east-1

### Componentes

| Componente | Tecnología | Detalle |
|---|---|---|
| Frontend | React + Vite + Nginx | Contenedor Fargate, puerto 80 |
| Backend | Spring Boot (Java 17) | Contenedor Fargate, puerto 8081 |
| Registro de imágenes | Amazon ECR | Repositorios privados |
| Orquestación | Amazon ECS Fargate | Clúster `innovatech-cluster` |
| CI/CD | GitHub Actions | Build → Push → Deploy |
| Logs | Amazon CloudWatch | `/ecs/innovatech-backend` y `/ecs/innovatech-frontend` |
| Escalado | ECS Target Tracking | 50% CPU, mín 1 — máx 3 tareas |
| Seguridad | IAM LabRole + GitHub Secrets | Sin credenciales en código |

---

## Requisitos previos

- Docker Desktop instalado
- AWS CLI configurado (`aws configure`)
- Cuenta AWS con acceso a ECS, ECR y CloudWatch
- Node.js 18+ (para desarrollo local del frontend)
- Java 17 + Maven (para desarrollo local del backend)

---

## Ejecución local con Docker Compose

Clona el repositorio y levanta todos los servicios:

```bash
git clone https://github.com/j3vnloko/innovatech-ep2.git
cd innovatech-ep2
docker compose up -d
```

Accede en `http://localhost` (frontend) y `http://localhost:8081` (backend).

### Variables de entorno requeridas

| Variable | Descripción | Valor por defecto |
|---|---|---|
| `DB_ENDPOINT` | Host del servidor MySQL | `localhost` |
| `DB_PORT` | Puerto MySQL | `3306` |
| `DB_NAME` | Nombre de la base de datos | `despachos_db` |
| `DB_USERNAME` | Usuario MySQL | `appuser` |
| `DB_PASSWORD` | Contraseña MySQL | `apppass123` |

---

## Pipeline CI/CD — GitHub Actions

El pipeline se activa automáticamente con cada push a la rama `deploy`.

### Flujo completo
Push rama deploy

│

▼

Checkout código
Configurar credenciales AWS
Login a Amazon ECR
Build imagen Docker
Push imagen a ECR
Render Task Definition ECS
Deploy automático en ECS Fargate


### Archivos del pipeline

| Archivo | Descripción |
|---|---|
| `.github/workflows/deploy-backend.yml` | Pipeline del backend Spring Boot |
| `.github/workflows/deploy-frontend.yml` | Pipeline del frontend React |
| `ecs/task-definition-backend.json` | Task Definition ECS para el backend |
| `ecs/task-definition-frontend.json` | Task Definition ECS para el frontend |

### Secrets requeridos en GitHub

Configurar en `Settings → Secrets and variables → Actions`:

| Secret | Descripción |
|---|---|
| `AWS_ACCESS_KEY_ID` | Clave de acceso AWS |
| `AWS_SECRET_ACCESS_KEY` | Clave secreta AWS |
| `AWS_SESSION_TOKEN` | Token de sesión AWS (AWS Academy) |
| `AWS_REGION` | Región AWS (`us-east-1`) |
| `ECR_REGISTRY` | URL del registro ECR |
| `ECS_CLUSTER` | Nombre del clúster (`innovatech-cluster`) |
| `ECS_SERVICE_BACKEND` | Nombre del servicio backend (`innovatech-backend-service`) |
| `ECS_SERVICE_FRONTEND` | Nombre del servicio frontend (`innovatech-frontend-service`) |

> **Nota AWS Academy:** Los tokens expiran con cada sesión. Actualizar `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` y `AWS_SESSION_TOKEN` desde AWS Details antes de cada deploy.

---

## Configuración del clúster ECS

### Crear clúster (primera vez)

```bash
aws ecs create-cluster \
  --cluster-name innovatech-cluster \
  --capacity-providers FARGATE \
  --default-capacity-provider-strategy capacityProvider=FARGATE,weight=1
```

### Crear Log Groups en CloudWatch

```bash
aws logs create-log-group --log-group-name /ecs/innovatech-backend
aws logs create-log-group --log-group-name /ecs/innovatech-frontend
```

### Autoscaling

Configurado con Target Tracking Scaling al 50% de CPU para ambos servicios:

| Parámetro | Valor |
|---|---|
| Tipo | Target Tracking Scaling |
| Métrica | ECSServiceAverageCPUUtilization |
| Umbral | 50% CPU |
| Mínimo de tareas | 1 |
| Máximo de tareas | 3 |
| Scale-Out cooldown | 60 segundos |
| Scale-In cooldown | 60 segundos |

---

## Estructura del repositorio
innovatech-ep2/

├── .github/

│   └── workflows/

│       ├── deploy-backend.yml       # Pipeline CI/CD backend

│       └── deploy-frontend.yml      # Pipeline CI/CD frontend

├── back-Despachos_SpringBoot/

│   └── Springboot-API-REST-DESPACHO/

│       ├── Dockerfile               # Multi-stage, Java 17

│       ├── pom.xml

│       └── src/

├── front_despacho/

│   ├── Dockerfile                   # Multi-stage, Node 18 + Nginx

│   ├── vite.config.js

│   └── src/

├── ecs/

│   ├── task-definition-backend.json  # Task Definition ECS backend

│   └── task-definition-frontend.json # Task Definition ECS frontend

├── docker-compose.yml               # Para desarrollo local

└── README.md

---

## Evolución del proyecto

| Etapa | Deploy | Pipeline |
|---|---|---|
| EP1 | EC2 manual | Sin automatización |
| EP2 | EC2 via AWS SSM | GitHub Actions → ECR → docker run en EC2 |
| EP3 | ECS Fargate (orquestado) | GitHub Actions → ECR → ECS Task Definition |

---

## Tecnologías

- **Contenedores:** Docker + Docker Compose
- **Orquestación:** Amazon ECS Fargate
- **Registro:** Amazon ECR
- **CI/CD:** GitHub Actions
- **Observabilidad:** Amazon CloudWatch Logs
- **Escalado:** Application Auto Scaling (Target Tracking)
- **Seguridad:** IAM LabRole · GitHub Secrets · Security Groups
- **Backend:** Spring Boot 3.4.4 · Java 17 · Maven · JPA · MySQL
- **Frontend:** React + Vite · Tailwind CSS · Nginx
