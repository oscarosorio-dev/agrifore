# ADR-0004: Modelo global con horizonte como variable y control agregado contra el baseline

- **Estado:** Aceptado
- **Fecha:** 2026-11-10

## Contexto
ADR-0002 define un LightGBM global con regresión por cuantiles, pero no dice cómo se cubren los horizontes de 1 a 7 días (RF-05), cuántos modelos forman una versión ni cómo se aplica RNF-06 a un modelo que cubre todas las series. La guía del backend y el modelo de datos asumían un modelo por producto y central.

## Decisión
- Una versión del modelo es global: cubre todos los productos y centrales, que entran como variables categóricas.
- El horizonte (1 a 7 días) es una variable de entrada. Cada versión tiene tres modelos, uno por cuantil: p10, p50 y p90.
- Los archivos se guardan en el volumen `/models` como `{version}_p10.txt`, `{version}_p50.txt` y `{version}_p90.txt`.
- Control de RNF-06: una versión no se activa si su MAPE agregado es peor que el del baseline naive estacional en cualquiera de los horizontes 1, 3 o 7 días. El MAPE agregado de un horizonte es el promedio del MAPE de todas las series, ponderado por su número de observaciones.
- El MAPE por producto y central se guarda en `model_metrics` como diagnóstico y no bloquea la activación.

## Alternativas consideradas
- **Un modelo por producto y central:** cada modelo aprende con pocos datos, multiplica los archivos y contradice ADR-0002.
- **Un modelo por cuantil y horizonte (21 por versión):** puede ajustar mejor cada horizonte, pero reparte los datos de entrenamiento entre siete modelos y multiplica el mantenimiento.
- **Exigir que la versión supere al baseline en cada serie:** con 15–20 series cortas, el ruido de una sola serie bloquearía con frecuencia versiones que mejoran el conjunto.

## Consecuencias
- `model_versions` deja de tener `product_id`, `market_id` y `mape`; las métricas pasan a `model_metrics`.
- Todos los pronósticos de una ejecución comparten `model_version_id`, así que `POST /decisions` devuelve un único valor.
- El rollback es de todo o nada: activar una versión anterior afecta a todas las series.
- Una versión puede pasar el control y aun así empeorar una serie concreta; esto se vigila con el diagnóstico por serie y el dashboard de modelos.
- Los tres modelos se entrenan por separado, así que los cuantiles pueden cruzarse (p10 > p50). La predicción debe ordenarlos antes de guardarlos en `forecasts`.
- Cada fecha de origen genera una fila de entrenamiento por horizonte, lo que multiplica por siete el tamaño del conjunto.
