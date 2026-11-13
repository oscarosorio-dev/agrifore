# Requisitos no funcionales

Los valores son objetivos del MVP y se validan en las pruebas.

## Rendimiento
- **RNF-01:** `POST /decisions` con p95 < 500 ms.
- **RNF-02:** Endpoints de consulta con p95 < 300 ms.

## Disponibilidad y datos
- **RNF-03:** Datos con no más de 48 h de retraso respecto a la publicación de SIPSA.
- **RNF-04:** Ingesta idempotente: reprocesar una fecha no duplica registros.
- **RNF-05:** Ante una falla de SIPSA, se sirve el último pronóstico válido marcado como desactualizado.

## Calidad del modelo
- **RNF-06:** Ninguna versión del modelo pasa a producción si su MAPE agregado es peor que el del baseline naive estacional en cualquiera de los horizontes 1, 3 o 7 días. El MAPE por producto y central se reporta como diagnóstico (ADR-0004).
- **RNF-07:** Todo pronóstico es trazable a su versión de modelo y a sus datos de entrenamiento.

## Seguridad
- **RNF-08:** Secretos solo en variables de entorno; nunca en el repositorio.
- **RNF-09:** Rate limiting en los endpoints públicos.
- **RNF-10:** Sin datos personales en el MVP.

## Mantenibilidad
- **RNF-11:** Cobertura de pruebas ≥ 70 % en `decision/` y `ingestion/`.
- **RNF-12:** Lint y tipado (ruff, mypy) obligatorios en CI.

## Observabilidad
- **RNF-13:** Métricas expuestas en `/metrics` (formato Prometheus).
- **RNF-14:** Logs estructurados en JSON con `request_id`.

## Portabilidad
- **RNF-15:** Todo el entorno levanta con `docker compose up`.