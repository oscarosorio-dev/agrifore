# Convenciones de la API

- **Base:** `/api/v1`. Versionado en la URL; un cambio incompatible implica una nueva versión.
- **Formato:** JSON, campos en `snake_case`, fechas en ISO 8601, moneda en COP sin decimales.
- **Paginación:** `?limit=&offset=` (por defecto `limit=100`, máximo `1000`).
- **Errores:** formato RFC 7807:

```json
{ "type": "about:blank", "title": "Not Found", "status": 404, "detail": "Producto no existe", "request_id": "..." }
```

- **Trazabilidad:** header `X-Request-ID` en todas las respuestas.
- **Contrato:** la especificación OpenAPI se genera automáticamente en `/docs`.