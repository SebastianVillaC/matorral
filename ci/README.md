# CI-CD de la aplicación (Actividad 6 — Opción A)

Dos pipelines de AWS CodePipeline (módulo `cicd` de `actividad6/aws`):

- **`matorral-pr` (CI):** al abrir o actualizar un PR hacia `main` → `buildspec-pr.yml`: construye y valida, **sin desplegar**.
- **`matorral-app` (CI-CD):** con cada **push / merge a `main`** → etapas de la tabla.

| Etapa | Servicio | Archivo | Qué hace |
|---|---|---|---|
| Source | CodeConnections (GitHub) | — | Descarga el commit de `main` |
| Build | CodeBuild | `buildspec-build.yml` | `docker build` con `docker/django/Dockerfile`, `manage.py check`, push a ECR `:<sha>` y `:latest` |
| Approval | CodePipeline | — | Aprobación manual (aviso por correo vía SNS) |
| Deploy | ECS | — | Actualiza los servicios `matorral-web` y `matorral-worker` (el contenedor web corre `migrate` al arrancar). Circuit breaker con rollback automático |
| SmokeTest | CodeBuild | `buildspec-smoke.yml` | `GET /health/` (BD + Valkey + Celery) y `/accounts/login/` |

## Rollback manual
```bash
aws ecs list-task-definitions --family-prefix matorral-web --sort DESC
aws ecs update-service --cluster matorral-cluster --service matorral-web    --task-definition matorral-web:<REV>
aws ecs update-service --cluster matorral-cluster --service matorral-worker --task-definition matorral-worker:<REV>
```

> Disparador automático: cada push/merge a `main` inicia el pipeline (trigger WebhookV2 de CodePipeline).
