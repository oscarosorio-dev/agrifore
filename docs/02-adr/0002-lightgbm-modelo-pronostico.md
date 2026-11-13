# ADR-0002: LightGBM como modelo principal de pronóstico

- **Estado:** Aceptado

## Contexto
Hay muchas series cortas, con variables exógenas (abastecimiento, festivos).

## Decisión
- Modelo principal: LightGBM global con variables de rezagos, abastecimiento y calendario.
- Baseline obligatorio: naive estacional.
- Intervalos de predicción mediante regresión por cuantiles.

## Alternativas consideradas
- **ARIMA por serie:** costoso de mantener con muchas series y sin manejo natural de variables exógenas.
- **Deep learning:** datos insuficientes para justificarlo en el MVP.

## Consecuencias
Entrenamiento rápido y modelo interpretable mediante importancia de variables. Requiere ingeniería de variables explícita.