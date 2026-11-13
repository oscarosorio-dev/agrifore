# Endpoints (v1)

- `GET  /health` — estado del servicio y frescura de los datos
- `GET  /metrics` — métricas en formato Prometheus
- `GET  /api/v1/products`
- `GET  /api/v1/markets`
- `GET  /api/v1/prices?product_id=&market_id=&from=&to=`
- `GET  /api/v1/forecasts?product_id=&market_id=&horizon=7`
- `POST /api/v1/decisions`

## POST /api/v1/decisions

Request:
```json
{
  "product_id": 1,
  "quantity_kg": 2000,
  "origin": { "lat": 5.36, "lon": -73.52 },
  "freight_cop_per_kg": { "3": 150, "5": 220 },
  "commission_pct": 5,
  "intermediary_price_cop_per_kg": 900
}
```

Response:
```json
{
  "recommendation": {
    "market_id": 3,
    "sell_date": "2026-11-10",
    "expected_price_cop_per_kg": 1400,
    "freight_cop_per_kg": 150,
    "commission_cop_per_kg": 70,
    "net_cop_per_kg": 1180,
    "net_range": [1050, 1290]
  },
  "vs_intermediary_cop_total": 560000,
  "alternatives": [ ... ],
  "data_freshness": "2026-11-07",
  "is_stale": false,
  "model_version_id": 12
}
```

**Justificación (RF-10, ADR-0003):** `recommendation` y cada elemento de `alternatives` traen el desglose del cálculo: precio esperado (p50), flete, comisión y precio neto, con su rango a partir de p10–p90. Los elementos de `alternatives` tienen la misma estructura que `recommendation`.

**Frescura (RNF-05):** `data_freshness` es la fecha del último precio usado. `is_stale` vale `true` cuando falla alguna de las reglas de frescura de [observability.md](../08-operations/observability.md#frescura-de-datos). El backend calcula `is_stale` y el frontend lo muestra sin calcular la frescura por su cuenta.

Pendiente: la regla para elegir entre opciones teniendo en cuenta la incertidumbre. Si al definirla hacen falta más campos, se agregan sin modificar los existentes.

Los valores del ejemplo son ilustrativos.