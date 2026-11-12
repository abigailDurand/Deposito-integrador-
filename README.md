# Banco Integrador

Repositorio integrador de los microservicios del banco.

## Estructura

| Carpeta | Propósito |
|---|---|
| `services-config/` | Configuración de cada microservicio (ver tabla de servicios) |
| `kubernetes/base/` | Manifiestos base de Kubernetes |
| `kubernetes/overlays/` | Ajustes por entorno (`desarrollo`, `produccion`) |
| `contracts/` | Contratos entre servicios (OpenAPI, eventos, etc.) |
| `tests/integration/` | Pruebas de integración entre servicios |
| `.github/workflows/` | Pipelines de CI/CD |

## Servicios

Cada carpeta de `services-config/` corresponde al campo `servicio` del aviso `servicio-actualizado`
y guarda en `version.env` la imagen y el tag fijados.

| Carpeta | Servicio | Repositorio | Notas |
|---|---|---|---|
| `deposito-service/` | Depósitos PEN y USD en un solo servicio | `abigailDurand/Deposito-Arnes` | Imagen `ghcr.io/abigaildurand/deposito-service`; usa billetera PEN y billetera USD |
| `depositos-pen/` | Depósitos PEN | _otro equipo_ | |
| `depositos-usd/` | Depósitos USD | _otro equipo_ | |
| `billetera-pen/` | Billetera PEN | | |
| `billetera-usd/` | Billetera USD | | |
| `transferencias/` | Transferencias | | |
| `usuarios/` | Usuarios | | |

## Uso local

```bash
cp .env.example .env
docker compose up -d
```
