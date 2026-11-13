# Runbook

## La ingesta falló
1. Revisar los logs: `docker compose logs worker | grep ingestion`.
2. Verificar la disponibilidad del WSDL de SIPSA.
3. Reprocesar manualmente: `python -m app.ingestion.run --desde <fecha>`.

## Alertas de frescura
Las reglas están en [observability.md](observability.md#frescura-de-datos). En los tres casos, la API sigue respondiendo con el último pronóstico válido, marcado como desactualizado (RNF-05).

- **Ingesta caída:** seguir los pasos de "La ingesta falló".
- **Ingesta atrasada frente a SIPSA:**
  1. Revisar los registros rechazados en `ingestion_records_total{status="rejected"}` y en los logs.
  2. Verificar si SIPSA cambió un código o un nombre de producto o central; comparar con los fixtures y con el [catálogo de centrales](../05-data/markets.md).
  3. Corregir y reprocesar con `python -m app.ingestion.run --desde <fecha>`.
- **SIPSA sin publicar:** confirmar en la página de boletines diarios del DANE que no hubo publicación. No hay acción en el sistema; si se prolonga, avisar a los usuarios.

## Degradación del modelo
1. Revisar el panel de error en producción en Grafana.
2. Si el MAPE supera el umbral, hacer rollback: activar la versión anterior en `model_versions`. El modelo es global, así que el rollback afecta a todas las series.
3. Abrir un issue con el análisis de causa.