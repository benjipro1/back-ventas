# Backend - API REST Ventas 🛒

API REST desarrollada con **Spring Boot 3.4.4** y **Java 17** para la gestión de ventas de Innovatech Chile.

---

## 🏗️ Arquitectura

```
EC2 Data (subred privada)
│
├── Contenedor: backend_ventas (puerto 8080)
│   └── Spring Boot API REST Ventas
│
└── Contenedor: mysql_ventas (puerto 3306)
    └── MySQL 8.0
        └── Named Volume: mysql_ventas_data (persistencia)
```

---

## ⚙️ Variables de entorno

Crear `.env` basado en `.env.example`:

```env
DOCKERHUB_USERNAME=tu_usuario
DB_NAME=ventas_db
DB_USERNAME=ventas_user
DB_PASSWORD=password_seguro
MYSQL_ROOT_PASSWORD=root_password_seguro
```

---

## 🚀 Levantar con Docker Compose

```bash
cp .env.example .env
# Editar .env con valores reales
docker compose up -d
docker compose logs -f backend-ventas
```

---

## 🐳 Dockerfile (multi-stage)

| Etapa | Imagen base | Propósito |
|-------|-------------|-----------|
| `builder` | `maven:3.9.6-eclipse-temurin-17` | Compila y genera el JAR |
| `runtime` | `eclipse-temurin:17-jre-alpine` | Corre el JAR con mínimo peso |

**Buenas prácticas:** multi-stage, usuario no-root, cache de dependencias Maven.

---

## 💾 Persistencia

Named volume `mysql_ventas_data` para MySQL. Se elige sobre bind mount por portabilidad y mejor rendimiento en Linux.

---

## 🔄 Pipeline CI/CD

El pipeline se activa con push a rama `deploy`:
1. **Build** → compila JAR con Maven
2. **Push** → sube imagen a Docker Hub
3. **Deploy** → SSH al Frontend → SSH interno al EC2 Data → actualiza contenedor

### Secrets requeridos en GitHub

| Secret | Descripción |
|--------|-------------|
| `DOCKERHUB_USERNAME` | Usuario Docker Hub |
| `DOCKERHUB_TOKEN` | Token Docker Hub |
| `EC2_FRONTEND_HOST` | IP pública EC2 Frontend (jump host) |
| `EC2_DATA_HOST` | IP privada EC2 Data |
| `EC2_USERNAME` | Usuario SSH (`ec2-user`) |
| `EC2_SSH_KEY` | Clave privada SSH (.pem) |
| `DB_VENTAS_NAME` | Nombre BD |
| `DB_VENTAS_USERNAME` | Usuario BD |
| `DB_VENTAS_PASSWORD` | Contraseña BD |
| `MYSQL_VENTAS_ROOT_PASSWORD` | Root password MySQL |

---

## 📡 Endpoints

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/ventas` | Listar todas las ventas |
| GET | `/api/v1/ventas/{id}` | Obtener venta por ID |
| POST | `/api/v1/ventas` | Crear venta |
| PUT | `/api/v1/ventas/{id}` | Actualizar venta |
| DELETE | `/api/v1/ventas/{id}` | Eliminar venta |
| GET | `/swagger-ui.html` | Documentación Swagger |
