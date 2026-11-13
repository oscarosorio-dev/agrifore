# Observabilidad

## Métricas clave
- `ingestion_last_success_timestamp`: última ejecución exitosa de la ingesta, aunque no haya traído datos nuevos.
- `ingestion_records_total{status}`: registros leídos, aceptados y rechazados.
- `ingestion_response_bytes` y `ingestion_download_seconds`: tamaño y duración de la descarga de SIPSA (ADR-0005).
- `sipsa_last_published_timestamp{market}`: última fecha con precios de cada central en la respuesta de SIPSA.
- `prices_last_stored_timestamp{market}`: última fecha guardada en `prices` para cada central.
- `sipsa_business_days_without_publication`: días hábiles desde la última publicación de SIPSA, en cualquier central.
- `forecast_mape{product,market,horizon}`
- `http_request_duration_seconds{route}`

## Dashboards (Grafana)
1. **Datos:** frescura por central y volumen ingerido.
2. **Modelos:** error en producción y cobertura del intervalo.
3. **API:** latencia, errores, throughput.

## Alertas

### Frescura de datos
No se mide en horas de calendario. SIPSA no publica los domingos ni los festivos, y varias centrales solo se recolectan algunos días ([catálogo de centrales](../05-data/markets.md)), así que pasar más de 48 h sin datos nuevos es normal. Por ejemplo, Centroabastos no tiene datos entre el sábado y el martes.

La ingesta corre todos los días, publique o no SIPSA. Con eso, la frescura se vigila con tres reglas:

| Alerta | Condición | Qué significa |
|---|---|---|
| Ingesta caída | `ingestion_last_success_timestamp` con más de 26 h | La ingesta no está terminando bien: falla nuestra o del servicio |
| Ingesta atrasada frente a SIPSA | Después de una ejecución exitosa, `sipsa_last_published_timestamp{market}` es posterior a `prices_last_stored_timestamp{market}` | SIPSA publicó datos que no guardamos: falla nuestra (catálogo, normalización o upsert) |
| SIPSA sin publicar | `sipsa_business_days_without_publication` > 2 | Problema externo. La API sirve el último pronóstico marcado como desactualizado (RNF-05) |

Los días hábiles se cuentan con el calendario de festivos de Colombia, el mismo que usa el modelo. Pendiente: definir la fuente de ese calendario.

### Otras
- Tasa de errores 5xx > 2 % durante 5 minutos.
- MAPE en producción por encima del baseline durante 7 días.
