# Modelo de datos

- **products:** `id`, `sipsa_id`, `name`, `group`, `unit`
- **markets:** `id`, `sipsa_id`, `name`, `city`, `lat`, `lon`
- **prices** (hypertable): `time`, `product_id`, `market_id`, `price_min`, `price_max`, `price_avg`, `source`
- **supply** (hypertable): `time`, `market_id`, `product_id`, `tons`. Es mensual: `time` es el primer día del mes, según `promedioAbasSipsaMesMadr`.
- **model_versions:** `id`, `algorithm`, `trained_at`, `train_from`, `train_to`, `is_active`
- **model_metrics:** `model_version_id`, `product_id`, `market_id`, `horizon`, `n_obs`, `mape`, `smape`, `coverage_p10_p90`, `baseline_mape`
- **forecasts:** `generated_at`, `target_date`, `product_id`, `market_id`, `p10`, `p50`, `p90`, `model_version_id`

Clave única en `prices`: (`time`, `product_id`, `market_id`). Es lo que garantiza la idempotencia de la ingesta.

El modelo es global (ADR-0004): cada fila de `model_versions` cubre todos los productos y centrales. `model_metrics` guarda el resultado del walk-forward por serie y horizonte, con clave única (`model_version_id`, `product_id`, `market_id`, `horizon`). El MAPE agregado de un horizonte se calcula ponderando `mape` por `n_obs`.