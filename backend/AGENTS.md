# AGENTS.md — Backend

Lee primero el [`AGENTS.md` raíz](../AGENTS.md). Este archivo solo agrega reglas del backend.

## Stack

Python 3.12, FastAPI, SQLAlchemy 2.x (async), Alembic, Pydantic v2, pydantic-settings, httpx y defusedxml (cliente SIPSA, ADR-0005), pandas, LightGBM, arq (cola sobre Redis), prometheus-client, pytest.

## Estructura

```
backend/
├── app/
│   ├── main.py                 # creación de la app, middlewares, routers
│   ├── api/
│   │   ├── deps.py             # dependencias (sesión de DB, paginación)
│   │   ├── errors.py           # handlers RFC 7807
│   │   └── v1/                 # un router por recurso: products, markets, prices, forecasts, decisions
│   ├── schemas/                # Pydantic de entrada y salida de la API
│   ├── services/               # casos de uso: orquestan repositorios y dominio
│   ├── decision/               # dominio puro del motor de decisión
│   ├── forecasting/
│   │   ├── features.py
│   │   ├── train.py
│   │   ├── predict.py
│   │   └── registry.py         # activar y desactivar versiones de modelo
│   ├── ingestion/
│   │   ├── sipsa_client.py     # único punto de contacto con SIPSA
│   │   ├── normalizers.py
│   │   └── run.py              # CLI de ingesta
│   ├── db/
│   │   ├── models/             # modelos ORM
│   │   ├── repositories/       # todo acceso a datos
│   │   ├── session.py
│   │   └── migrations/         # Alembic
│   └── core/
│       ├── config.py
│       ├── logging.py
│       └── metrics.py
├── worker/
│   ├── main.py                 # configuración de arq y cron
│   └── tasks/                  # ingest.py, train.py, forecast.py
├── evals/
└── tests/
    ├── unit/                   # decision/, normalizers, features
    ├── integration/            # repositorios y API contra una DB de prueba
    └── fixtures/sipsa/         # respuestas SOAP grabadas
```

## ¿Dónde va cada cosa?

- **Nuevo endpoint:** router en `api/v1/`, esquemas en `schemas/`, lógica en `services/`.
- **Nueva consulta a la base de datos:** `db/repositories/`. Nunca escribas consultas SQL o del ORM en routers ni en servicios.
- **Nueva regla de cálculo de precio o recomendación:** `decision/`.
- **Nueva variable para el modelo:** `forecasting/features.py`, más una nota en `docs/06-ml/evaluation.md`.
- **Cambio en cómo se consume SIPSA:** `ingestion/sipsa_client.py`, más una actualización de `docs/05-data/sources.md`.
- **Nueva tarea programada:** `worker/tasks/`, registrada en `worker/main.py`.
- **Nueva variable de entorno:** `core/config.py` y `.env.example`.
- **Nueva métrica:** `core/metrics.py`, más una actualización de `docs/08-operations/observability.md`.

## Dependencias permitidas entre capas

```
api → services → (decision | forecasting | db.repositories)
worker → services
```

- `decision/` no importa nada del proyecto, excepto sus propios tipos.
- `forecasting/` no importa nada de `api/`.
- Ninguna capa importa nada de `api/`.

## Reglas

- **Contrato:** cada cambio de endpoint actualiza `docs/04-api/endpoints.md`. La especificación OpenAPI generada debe coincidir con ese documento.
- **Errores:** los servicios lanzan excepciones de dominio; `api/errors.py` las traduce a respuestas RFC 7807. No uses `HTTPException` dentro de `services/`.
- **Sesiones de base de datos:** se inyectan con `Depends`; un servicio nunca crea su propia sesión.
- **Dinero:** COP como `int` (sin `float`). Precios por kilogramo.
- **Fechas:** `date` para días de mercado, `datetime` con zona horaria UTC para timestamps.
- **Modelos de ML:** un modelo global por versión, con un archivo por cuantil en el volumen `/models`: `{version}_p10.txt`, `{version}_p50.txt` y `{version}_p90.txt`, registrados en `model_versions` (ADR-0004). Las métricas por serie y horizonte van en `model_metrics`. Activar una versión que no supere el control de RNF-06 está prohibido.
- **Ingesta:** usa upsert y reprocesa los últimos 7 días en cada ejecución.
- **Rendimiento:** la API no carga modelos ni hace inferencia durante una petición.
- **Logs:** con `logger` de `core/logging.py`, incluyendo `request_id` en contexto.

## Pruebas

- `decision/`: pruebas unitarias exhaustivas, incluidos los casos borde (flete mayor que el precio, falta de pronóstico, cantidad cero).
- Repositorios y API: pruebas de integración contra una base de datos de prueba.
- SIPSA: siempre con fixtures grabados; nunca contra el servicio real en CI.

## Comandos

```bash
docker compose exec api pytest tests/unit
docker compose exec api pytest tests/integration
docker compose exec api ruff check . && mypy app
docker compose exec api python -c "import json; from app.main import app; print(json.dumps(app.openapi()))" > openapi.json
```