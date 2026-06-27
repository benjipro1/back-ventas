# Backend — API REST Ventas 🛒

API REST desarrollada con **Spring Boot 3.4.4** y **Java 17** para la gestión de ventas de Innovatech Chile.

---

## 🏗️ Arquitectura de despliegue (EP3/EFT — ECS Fargate)

```
Internet
   │
   ▼
Application Load Balancer (innovatech-alb)
   │  ruta: /api/v1/ventas*
   ▼
ECS Fargate — Service: innovatech-svc-ventas
   │  Task Definition: innovatech-ventas (CPU 512 / 1024 MB)
   │  Container: ventas (puerto 8080)
   ▼
Amazon RDS MySQL (innovatech-db) — esquema ventas_db
```

- **Orquestación:** Amazon ECS con capacity provider Fargate (sin servidores que administrar).
- **Autoscaling:** Target Tracking por CPU (objetivo 50%), min 1 / max 3 tasks.
- **Imagen:** publicada en Amazon ECR (`innovatech-ventas`), con escaneo de vulnerabilidades automático (`scanOnPush`).
- **Secrets:** credenciales de base de datos gestionadas por AWS Secrets Manager, inyectadas como `secrets` en la Task Definition (nunca en texto plano).
- **Observabilidad:** logs en CloudWatch (`/ecs/innovatech-ventas`).
- **Health check:** Spring Boot Actuator expuesto en `/actuator/health`.

> Este servicio comparte la instancia RDS con `back-despachos` (un esquema independiente cada uno: `ventas_db` / `despachos_db`).

---

## ⚙️ Variables de entorno

| Variable | Origen | Descripción |
|---|---|---|
| `DB_ENDPOINT` | Task Definition (`environment`) | Endpoint de la instancia RDS |
| `DB_PORT` | Task Definition (`environment`) | Puerto de MySQL (3306) |
| `DB_NAME` | Task Definition (`environment`) | `ventas_db` |
| `DB_USERNAME` | Task Definition (`secrets`) | Desde AWS Secrets Manager |
| `DB_PASSWORD` | Task Definition (`secrets`) | Desde AWS Secrets Manager |

---

## 🐳 Desarrollo local

```bash
docker compose up --build
```

---

## 🔄 CI/CD

El pipeline (`.github/workflows/deploy.yml`) se dispara en cada push a la rama `deploy`:

1. Tests unitarios con Maven (incluye `VentaServiceTest`; excluye el test de carga de contexto que requiere RDS real).
2. Build de la imagen Docker.
3. Push a Amazon ECR (tags `latest` y hash del commit).
4. Registro de nueva revisión de la Task Definition.
5. Deploy automático al servicio ECS, esperando estabilidad.

**Secrets requeridos en GitHub:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` (credenciales temporales de AWS Academy Learner Lab).

---

## 📚 Endpoints principales

Base path: `/api/v1/ventas`

Documentación interactiva (Swagger/OpenAPI) disponible en `/swagger-ui.html`.
