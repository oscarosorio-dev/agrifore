# ADR-0001: TimescaleDB para series de tiempo

- **Estado:** Aceptado

## Contexto
Precios y abastecimiento son series diarias por producto y central, con consultas por rango de fechas.

## Decisión
Usar PostgreSQL con la extensión TimescaleDB (hypertables).

## Alternativas consideradas
- **PostgreSQL puro:** más simple, pero las consultas por rango y los agregados son más lentos a escala.
- **InfluxDB:** agrega un motor más y complica los joins con los catálogos.

## Consecuencias
Un solo motor de base de datos, SQL estándar y agregados continuos. Requiere la imagen Docker de TimescaleDB.