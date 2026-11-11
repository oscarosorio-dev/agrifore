# Pronóstico de Precios Agrícolas

Sistema que pronostica precios mayoristas de productos agrícolas en las centrales de abasto de Colombia y calcula **dónde y cuándo vender** según el precio neto para el agricultor.

## Resumen

El DANE publica los precios mayoristas a través de SIPSA, pero un precio no es una decisión. Este proyecto convierte esos datos en una recomendación concreta: precio esperado por central, menos flete y comisión, comparado con la oferta del intermediario. La recomendación incluye su nivel de incertidumbre.

## Problema

El pequeño agricultor suele vender al intermediario sin referencia del precio real del mercado ni de lo que le quedaría en limpio al vender en otra central. La asimetría de información reduce su ingreso.

## Alcance del MVP (Fase 1)

- Ingesta diaria de precios y abastecimiento desde el servicio web de SIPSA (DANE).
- Pronóstico de precios a 1–7 días por producto y central, con intervalos de predicción.
- Motor de decisión determinista: `precio_neto = precio_esperado - flete - comisión`.
- Interfaz web con un simulador "¿Dónde vendo?" y un explorador de precios.
- Observabilidad de datos, modelos y API.

**Cobertura inicial:** papa negra, tomate, cebolla cabezona blanca, plátano hartón verde y zanahoria, en 3 centrales de abasto: Corabastos (Bogotá), Central Mayorista de Antioquia (Medellín) y Centroabastos (Bucaramanga). Las demás centrales disponibles están en el [catálogo de centrales](docs/05-data/markets.md).

## Arquitectura

```
SIPSA (SOAP) → Ingesta (worker) → TimescaleDB
                                      ↓
              Entrenamiento / Pronóstico (LightGBM)
                                      ↓
                  Motor de decisión → FastAPI → Astro
```

El pronóstico usa machine learning clásico. La decisión es determinista y explicable. En esta fase no se usa LLM.

## Stack

- **Backend:** Python 3.12, FastAPI, SQLAlchemy, Alembic, httpx y defusedxml (cliente SIPSA)
- **ML:** pandas, LightGBM, statsforecast (baseline)
- **Datos:** PostgreSQL + TimescaleDB, Redis
- **Frontend:** Astro
- **Infraestructura:** Docker Compose
- **Observabilidad:** Prometheus, Grafana, Langfuse (preparado para la Fase 3)

## Estructura

```
├── backend/
│   ├── app/
│   │   ├── api/            # endpoints REST
│   │   ├── schemas/        # esquemas de entrada y salida
│   │   ├── services/       # casos de uso
│   │   ├── ingestion/      # cliente SIPSA y normalización
│   │   ├── forecasting/    # features, entrenamiento, predicción
│   │   ├── decision/       # cálculo de precio neto y recomendación
│   │   ├── core/           # configuración, logging, métricas
│   │   └── db/             # modelos y migraciones
│   ├── worker/             # tareas programadas
│   └── evals/              # métricas y backtest
├── frontend/               # Astro
├── observability/          # dashboards Grafana, Prometheus
├── docs/                   # documentación (índice en docs/README.md)
└── docker-compose.yml
```

## Puesta en marcha

```bash
cp .env.example .env
docker compose up -d --build
docker compose exec api alembic upgrade head
docker compose exec api python -m app.ingestion.run --desde 2020-02-01
docker compose exec api python -m app.forecasting.train
```

- Frontend: http://localhost:4321
- API docs: http://localhost:8000/docs
- Grafana: http://localhost:3000

## Evaluación

- **Pronóstico:** MAPE por producto y horizonte, comparado con un baseline naive estacional.
- **Backtest de decisión:** ganancia de seguir la recomendación frente a vender siempre el mismo día en el mercado local.

Los resultados se publican en [`docs/06-ml/evaluation.md`](docs/06-ml/evaluation.md).

## Limitaciones

- SIPSA reporta precios mayoristas de referencia; el precio real varía por calidad y negociación.
- El flete y la comisión los ingresa el usuario.
- Eventos atípicos como paros o cierres de vías degradan el pronóstico.
- El sistema recomienda, no garantiza un precio.

## Roadmap

- [ ] **Fase 1:** pronóstico + motor de decisión + web
- [ ] **Fase 2:** consolidación de carga entre agricultores y optimización de rutas
- [ ] **Fase 3:** asistente por WhatsApp con voz (STT, LLM, TTS)

## Fuente de datos

DANE – Sistema de Información de Precios y Abastecimiento del Sector Agropecuario (SIPSA).

## Licencia

MIT