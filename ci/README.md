# CI-CD de la aplicación (Actividad 6 — Opción A)

Pipeline AWS CodePipeline `matorral-app` (definido en `actividad6/aws/cicd.tf`) que se dispara con cada
**push / merge de PR a `main`** de este fork.

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
