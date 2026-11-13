# Evaluación

## Modelo
- LightGBM global con un modelo por cuantil (p10, p50 y p90). El producto, la central y el horizonte (1 a 7 días) son variables de entrada (ADR-0002, ADR-0004).
- Abastecimiento: dato mensual de `promedioAbasSipsaMesMadr`. Solo se usan meses publicados antes de la fecha del pronóstico, para no filtrar información futura; el 2026-11-10 el último mes disponible era 2026-09. El walk-forward compara las opciones de [correspondencia de productos](../05-data/sources.md#correspondencia-de-productos-con-el-abastecimiento): todas las variedades sumadas frente a solo la dominante de cada central, cebolla cabezona total frente a solo blanca, y el modelo sin abastecimiento. Se usa la de menor error. Resultado: _Pendiente de la primera ejecución._

## Pronóstico
- Validación temporal tipo walk-forward (sin mezclar datos futuros en el entrenamiento).
- Ventana de entrenamiento: se guardan todos los datos desde 2020-02-01, pero la fecha de inicio del entrenamiento es un parámetro. El walk-forward compara ventanas que empiezan en 2020-02-01 (todo el histórico), 2022-01-01 (sin pandemia, sin paro de 2021 y con abastecimiento completo) y 2023-01-01 (unos 3 años), y se usa la de menor MAPE agregado frente al baseline. Resultado: _Pendiente de la primera ejecución._
- Métricas: MAPE y sMAPE por producto, central y horizonte (1, 3 y 7 días).
- Cobertura del intervalo: porcentaje de valores reales que caen dentro de p10–p90 (objetivo ≈ 80 %).
- Toda versión se compara contra el baseline naive estacional en las mismas ventanas del walk-forward.
- Control para pasar a producción (RNF-06): MAPE agregado por horizonte (1, 3 y 7 días), ponderado por número de observaciones. El MAPE por producto y central es diagnóstico y no bloquea la activación.

## Decisión (backtest)
- Estrategia A: vender el mismo día en el mercado local.
- Estrategia B: seguir la recomendación del sistema.
- Métrica: diferencia de ingreso neto acumulado.

## Resultados
_Pendiente de la primera ejecución._

## Análisis de fallos
_Documentar los periodos atípicos (paros, cierres de vías, clima)._