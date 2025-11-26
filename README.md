# Traefik + Go API Gateway

Pequeño ejemplo para levantar un gateway HTTP con Traefik y enrutar tráfico hacia servicios en Go.

## Qué incluye
- Traefik v3 como reverse proxy (dashboard en `http://localhost:8080/dashboard/`).
- Servicio de ejemplo en Go (`cmd/api`) expuesto en `http://localhost/api/...`.
- Middleware CORS definido en `traefik/dynamic.yml` y aplicado al router `api`.
- `docker-compose.yml` listo para levantar gateway y servicios.

## Ejecución rápida
1) Requisitos: Docker y Docker Compose v2.
2) Construir y levantar todo:
```
docker compose up --build
```
3) Probar rutas pasando por Traefik:
```
curl http://localhost/api/health
curl "http://localhost/api/v1/greet?name=Ana"
curl -X POST http://localhost/api/v1/echo \
  -H "Content-Type: application/json" \
  -d '{"mensaje":"hola"}'
```

## Cómo funciona el ruteo
- Traefik escucha en el entrypoint `web` (puerto 80) y usa la regla `Host("localhost") && PathPrefix("/api")`.
- El middleware `api-strip` elimina el prefijo `/api`, por lo que el servicio Go recibe `/health` o `/v1/...`.
- Los servicios se descubren vía etiquetas Docker (proveedor `docker`) y se enriquecen con middlewares definidos en `traefik/dynamic.yml` (proveedor `file`).

## Añadir más servicios
1) Agrega un nuevo servicio en `docker-compose.yml` con `traefik.enable=true`.
2) Define una regla de router (`traefik.http.routers.<nombre>.rule`) y opcionalmente un middleware de strip prefix para no duplicar rutas.
3) Expón el puerto interno del servicio con `traefik.http.services.<nombre>.loadbalancer.server.port=<puerto>`.
4) Si el servicio necesita CORS u otros headers, referencia un middleware del archivo `traefik/dynamic.yml` o crea uno nuevo ahí.

## Desarrollo local del servicio Go
- Ejecutar sin Docker:
```
go run ./cmd/api
```
- El servidor escucha en `:8080`. Con Traefik activo, las peticiones deben ir a `http://localhost/api/...` para que se aplique el ruteo.
