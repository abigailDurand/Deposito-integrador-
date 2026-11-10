# Banco Integrador

Repositorio integrador de los microservicios del banco.

## Estructura

| Carpeta | Propósito |
|---|---|
| `services-config/` | Configuración de cada microservicio (usuarios, depósitos PEN/USD, billeteras PEN/USD, transferencias) |
| `kubernetes/base/` | Manifiestos base de Kubernetes |
| `kubernetes/overlays/` | Ajustes por entorno (`desarrollo`, `produccion`) |
| `contracts/` | Contratos entre servicios (OpenAPI, eventos, etc.) |
| `tests/integration/` | Pruebas de integración entre servicios |
| `.github/workflows/` | Pipelines de CI/CD |

## Uso local

```bash
cp .env.example .env
docker compose up -d
```
