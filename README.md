# Innovatech EP2 - DevOps

## Descripción
Proyecto de contenedorización y despliegue automatizado en AWS para Innovatech Chile.

## Arquitectura
- **Frontend**: React + Vite, contenedor nginx, EC2 pública
- **Backend**: Spring Boot, contenedor Java, EC2 privada
- **Base de datos**: MySQL en contenedor con volumen persistente

## Cómo usar

### Levantar con Docker Compose
```bash
docker compose up -d
```

### Variables de entorno requeridas
- `DB_ENDPOINT`: IP del servidor MySQL
- `DB_PORT`: Puerto MySQL (3306)
- `DB_NAME`: Nombre de la base de datos
- `DB_USERNAME`: Usuario MySQL
- `DB_PASSWORD`: Contraseña MySQL

## Pipeline CI/CD
El pipeline se activa automáticamente al hacer push en la rama `deploy`.

Flujo: `build imagen Docker` → `push a Amazon ECR` → `deploy en EC2 via SSM`

## Volúmenes Docker
Se usa **named volume** (`db_data`) para persistencia de la base de datos, garantizando que los datos no se pierdan al reiniciar contenedores.

## Tecnologías
- Docker + Docker Compose
- GitHub Actions
- Amazon ECR
- Amazon EC2
- AWS SSM
- Spring Boot
- React + Vite