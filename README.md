# Traefik Gateway para APIs Externas

Gateway HTTP con Traefik que enruta tráfico hacia APIs externas (Mortgage e SBS).

## Qué incluye
- Traefik v3 como reverse proxy (dashboard en `http://localhost:8180`).
- Ruteo a API de Mortgage (IAM y cálculos) en puerto 8181.
- Ruteo a API de SBS (tasas TEA) en puerto 8082.
- Middleware CORS aplicado a todas las rutas.
- `docker-compose.yml` para levantar solo Traefik.

## Ejecución rápida
1) Requisitos: Docker y Docker Compose v2.
2) Levantar Traefik:
```
sudo docker-compose up -d
```
3) Probar rutas:
```
# Mortgage API
curl http://localhost/api/v1/iam/register -X POST -H "Content-Type: application/json" -d '{"email":"test@example.com","full_name":"Test","password":"123"}'
curl http://localhost/swagger/index.html  # Docs de Mortgage

# SBS API
curl http://localhost/api/v1/rates?date=2023-01-01
curl http://localhost/api/v1/date
```

## Cómo funciona el ruteo
- Traefik escucha en puerto 80.
- `/api/v1/iam/*` y `/api/v1/mortgage/*` van a Mortgage API (192.168.100.63:8181).
- `/api/v1/rates` y `/api/v1/date` van a SBS API (192.168.100.63:8082).
- `/swagger/*` va a Mortgage API.
- CORS habilitado para todos.

## Añadir más servicios
Edita `traefik/dynamic.yml` para agregar nuevos routers y servicios, apuntando a IPs/puertos externos.

## Desarrollo
Las APIs externas deben estar corriendo en sus respectivos puertos (8181 y 8082).
