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

## Servicios

Cada servicio tiene su carpeta en `services-config/`. Cuando un servicio publica una imagen nueva, avisa al integrador y el workflow `Recibir actualizacion` fija la versión en `services-config/<servicio>/version.env`.

| Carpeta | Valor de `servicio` en el aviso | Responsable | Descripción |
|---|---|---|---|
| `deposito-service/` | `deposito-service` | Equipo deposito-service | Depósitos PEN y USD en un solo servicio |
| `depositos-pen/` | `depositos-pen` | Por asignar | Depósitos en soles |
| `depositos-usd/` | `depositos-usd` | Por asignar | Depósitos en dólares |
| `usuarios/` | `usuarios` | Por asignar | Usuarios |
| `billetera-pen/` | `billetera-pen` | Por asignar | Billetera en soles |
| `billetera-usd/` | `billetera-usd` | Por asignar | Billetera en dólares |
| `transferencias/` | `transferencias` | Por asignar | Transferencias |

> Cada equipo debe enviar **su propio** valor de `servicio`. Si dos equipos usan el mismo, se sobrescriben la versión fijada.

### Cómo avisar al integrador desde un servicio

1. Crear un token *fine-grained* con acceso solo a este repo y permiso **Contents: Read and write**.
2. Guardarlo en el repo del servicio como secreto de Actions llamado `INTEGRADOR_TOKEN`.
3. Agregar este paso al final del workflow, después de publicar la imagen:

```yaml
      - name: Avisar al integrador
        env:
          GH_TOKEN: ${{ secrets.INTEGRADOR_TOKEN }}
        run: |
          gh api repos/abigailDurand/Deposito-integrador-/dispatches \
            -f event_type=servicio-actualizado \
            -f "client_payload[servicio]=<valor-de-la-tabla>" \
            -f "client_payload[imagen]=ghcr.io/<owner>/<imagen>" \
            -f "client_payload[tag]=${{ github.sha }}" \
            -f "client_payload[commit]=${{ github.sha }}"
```

`servicio` solo acepta minúsculas, números y guiones. Los avisos se ven en la pestaña **Actions → Recibir actualizacion** de este repo.

## Uso local

```bash
cp .env.example .env
docker compose up -d
```

## Kubernetes

Los manifiestos usan Kustomize: `kubernetes/base/<servicio>/` tiene el Deployment, Service y ConfigMap de cada
servicio, y `kubernetes/overlays/` ajusta por entorno (`desarrollo` → namespace `banco-desarrollo`, 1 réplica;
`produccion` → namespace `banco`, 2 réplicas).

Convención para que los servicios se encuentren entre sí: cada uno publica un Service con el **nombre de su carpeta**
y **puerto 80** (p. ej. `http://deposito-service`, `http://billetera-pen`).

```bash
# Ver lo que se aplicaría
kubectl kustomize kubernetes/overlays/desarrollo

# Secretos (una vez por namespace; plantilla en kubernetes/base/deposito-service/secret.example.yaml)
kubectl create namespace banco-desarrollo
kubectl -n banco-desarrollo create secret generic deposito-service-secrets \
  --from-literal=DATABASE_URL='postgres://...' \
  --from-literal=JWT_PUBLIC_KEY="$(cat usuario-service.pub.pem)" \
  --from-literal=BILLETERA_TOKEN='...'

# Desplegar
kubectl apply -k kubernetes/overlays/desarrollo
```
