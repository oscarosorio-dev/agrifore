# AGENTS.md

Guía obligatoria para agentes de IA y colaboradores que trabajen en este repositorio.

## Proyecto

**Pronóstico de Precios Agrícolas.** Pronostica precios mayoristas (fuente: SIPSA – DANE) y recomienda dónde y cuándo vender según el precio neto para el agricultor.

Fase actual: **Fase 1 (MVP)**. Ver [`docs/00-product/vision.md`](docs/00-product/vision.md).

## Regla 1: leer la documentación antes de cambiar código

Antes de cualquier cambio, lee como mínimo:

1. `docs/README.md` (índice)
2. Los requisitos afectados en `docs/00-product/`
3. `docs/01-architecture/overview.md` y `docs/01-architecture/data-model.md`
4. Los ADR relacionados en `docs/02-adr/`
5. `docs/04-api/conventions.md`, si tocas la API

Si el código y la documentación se contradicen, **no asumas cuál es el correcto**: repórtalo y pide aclaración.

## Regla 2: la documentación se actualiza en el mismo cambio

Un cambio no está terminado si la documentación queda desactualizada.

- Nuevo comportamiento o cambio de comportamiento → `docs/00-product/requirements-functional.md` (ID `RF-XX`).
- Cambio en rendimiento, seguridad u operación → `docs/00-product/requirements-non-functional.md` (ID `RNF-XX`).
- Endpoint nuevo o modificado → `docs/04-api/endpoints.md`.
- Tabla, columna o índice → `docs/01-architecture/data-model.md`, más una migración de Alembic.
- Nuevo componente o flujo → `docs/01-architecture/overview.md`.
- Decisión técnica (librería, patrón, motor, algoritmo) → nuevo ADR en `docs/02-adr/`, a partir de `0000-template.md`.
- Cambio grande o de una fase futura → primero un RFC en `docs/03-rfc/`. No implementar sin un RFC aceptado.
- Nueva fuente de datos o cambio de una existente → `docs/05-data/sources.md`.
- Cambio en modelos o métricas → `docs/06-ml/evaluation.md`.
- Nueva métrica, alerta o procedimiento operativo → `docs/08-operations/`.
- Nuevo paso de setup o variable de entorno → `docs/07-onboarding/setup.md` y `.env.example`.
- Todo cambio visible para el usuario → `CHANGELOG.md`.

Los ADR aceptados no se editan. Si una decisión cambia, se crea un ADR nuevo y el anterior se marca como `Reemplazado por ADR-XXXX`.

## Regla 3: enrutamiento según la tarea

Este archivo define las reglas globales. Cada área tiene su propio `AGENTS.md` con reglas específicas, que **también es obligatorio leer** antes de trabajar en ella.

- **Interfaz, páginas, componentes, estilos, textos visibles** → [`frontend/AGENTS.md`](frontend/AGENTS.md)
- **Endpoints, lógica de negocio, base de datos, ingesta, modelos de ML, tareas programadas** → [`backend/AGENTS.md`](backend/AGENTS.md)
- **Contenedores, variables de entorno, red entre servicios** → `docker-compose.yml`, `.env.example` y `docs/07-onboarding/setup.md`
- **Métricas, dashboards, alertas** → `observability/` y `docs/08-operations/observability.md`
- **Solo documentación** → `docs/` (respetar la estructura de carpetas)

### Funcionalidades que cruzan backend y frontend

Orden obligatorio:

1. Documentar el requisito en `docs/00-product/`.
2. Definir el contrato en `docs/04-api/endpoints.md`.
3. Implementar en el backend, siguiendo `backend/AGENTS.md`.
4. Regenerar los tipos del frontend desde OpenAPI.
5. Implementar en el frontend, siguiendo `frontend/AGENTS.md`.

El backend es la fuente de verdad del contrato. El frontend nunca define ni asume campos que la API no expone.

### Precedencia

El `AGENTS.md` del área puede agregar reglas, pero no contradecir las de este archivo. Si encuentras un conflicto, aplica la regla de la raíz y repórtalo.

## Estructura del repositorio

```
├── AGENTS.md
├── README.md
├── CHANGELOG.md
├── .env.example
├── docker-compose.yml
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/            # routers FastAPI (solo HTTP: validación y serialización)
│   │   ├── schemas/        # esquemas Pydantic de entrada y salida de la API
│   │   ├── services/       # casos de uso: orquestan el dominio y los repositorios
│   │   ├── ingestion/      # cliente SIPSA, normalización, carga
│   │   ├── forecasting/    # features, entrenamiento, predicción, registro de modelos
│   │   ├── decision/       # motor de decisión (lógica pura, sin I/O)
│   │   ├── core/           # config, logging, métricas
│   │   └── db/             # modelos SQLAlchemy, sesiones, migraciones Alembic
│   ├── worker/             # tareas programadas
│   ├── evals/              # métricas de modelo y backtest
│   ├── AGENTS.md
│   └── tests/              # unit/, integration/ y fixtures/ (ver backend/AGENTS.md)
├── frontend/               # Astro
│   ├── AGENTS.md
├── observability/          # Prometheus, dashboards Grafana
└── docs/
    ├── README.md
    ├── 00-product/         # visión, requisitos funcionales y no funcionales
    ├── 01-architecture/    # visión general, modelo de datos
    ├── 02-adr/             # decisiones tomadas
    ├── 03-rfc/             # propuestas
    ├── 04-api/             # convenciones y endpoints
    ├── 05-data/            # fuentes de datos, catálogo de centrales
    ├── 06-ml/              # evaluación de modelos
    ├── 07-onboarding/      # setup, contribución
    ├── 08-operations/      # runbook, observabilidad
    └── 09-release/         # proceso de release
```

No crees carpetas fuera de esta estructura sin un ADR que lo justifique.

## Reglas de arquitectura

- **`api/`** no contiene lógica de negocio: delega en `services/`, que orquesta `decision/`, `forecasting/` y los repositorios de `db/`.
- **`decision/`** es lógica pura: sin acceso a la base de datos, a la red ni al disco. Recibe datos y devuelve resultados.
- **La API no predice en línea:** solo lee los pronósticos precalculados de la tabla `forecasts`.
- **Ingesta idempotente:** usa upsert sobre la clave única (`time`, `product_id`, `market_id`).
- **Sin LLM en la Fase 1** (ADR-0003). El cálculo de la decisión nunca depende de un LLM.
- **Configuración** solo en `core/config.py`, leída de variables de entorno (pydantic-settings).
- **Dependencias:** no agregues librerías sin justificarlas. Si una dependencia es relevante, registra un ADR.

## Convenciones de código

- Python 3.12, tipado completo, `ruff` y `mypy` sin errores.
- Esquemas de entrada y salida con Pydantic; nunca devuelvas modelos ORM directamente en la API.
- `async` en la capa HTTP; el código de ML puede ser síncrono y ejecutarse en el worker.
- Logs estructurados en JSON; sin `print`.
- Elimina imports, código muerto y archivos sin uso en el mismo cambio.
- Commits en formato Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`).

## Comandos

```bash
docker compose up -d --build                                   # levantar entorno
docker compose exec api alembic upgrade head                   # migraciones
docker compose exec api alembic revision --autogenerate -m ""  # nueva migración
docker compose exec api pytest                                 # tests
docker compose exec api ruff check . && mypy app               # lint y tipos
docker compose exec api python -m app.ingestion.run --desde YYYY-MM-DD
docker compose exec api python -m app.forecasting.train
```

## Pruebas

- Todo cambio en `decision/` o `ingestion/` lleva tests (cobertura ≥ 70 %, RNF-11).
- Los tests no llaman a SIPSA real: usa fixtures con respuestas grabadas.
- Un modelo nuevo debe superar al baseline naive estacional (RNF-06), con evidencia en `docs/06-ml/evaluation.md`.

## Definición de terminado

- [ ] Documentación relevante leída antes del cambio
- [ ] Código con tipado, lint y tests en verde
- [ ] Documentación de `docs/` actualizada según la Regla 2
- [ ] ADR o RFC creado si aplica
- [ ] Migración incluida si cambió el modelo de datos
- [ ] `.env.example` actualizado si hay variables nuevas
- [ ] `CHANGELOG.md` actualizado

## Prohibido

- Commitear secretos, `.env` o datos crudos pesados.
- Implementar funcionalidades de la Fase 2 o 3 sin un RFC aceptado.
- Modificar migraciones ya aplicadas; crea una nueva.
- Inventar datos, métricas o resultados en la documentación. Si algo no está medido, se marca como _Pendiente_.
- Cambiar contratos de la API v1 de forma incompatible.