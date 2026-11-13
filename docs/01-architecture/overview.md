# Arquitectura

## Componentes
- **api** (FastAPI): consultas, pronósticos y decisiones.
- **worker:** ingesta diaria, reentrenamiento semanal, generación de pronósticos.
- **db** (PostgreSQL + TimescaleDB): series de tiempo y catálogos.
- **redis:** cola de tareas y caché.
- **frontend** (Astro): simulador y explorador de precios.
- **observabilidad:** Prometheus + Grafana.

## Flujo
1. El worker consume SIPSA, normaliza los datos y los persiste.
2. El worker reentrena y genera los pronósticos, que quedan guardados en la tabla `forecasts`.
3. La API lee los pronósticos ya calculados; nunca predice en línea durante la petición.
4. El motor de decisión combina el pronóstico con los costos del usuario.

## Principios
- Pronósticos precalculados, para que la latencia de la API sea predecible.
- Lógica de decisión pura, sin I/O, y testeable.
- Módulos desacoplados por dominio: `ingestion`, `forecasting`, `decision`.