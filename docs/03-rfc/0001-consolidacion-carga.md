# RFC-0001: Consolidación de carga entre agricultores (Fase 2)

- **Estado:** Borrador

## Motivación
El flete individual hace inviable vender en centrales lejanas. Compartir el transporte reduce el costo por kilo.

## Propuesta
- Nuevo módulo `logistics/`.
- Agrupar la oferta por cercanía y fecha de cosecha (clustering).
- Optimizar la ruta de recolección con OR-Tools (problema de ruteo con capacidad, CVRP).
- Repartir el flete de forma proporcional a la carga de cada agricultor.

## Impacto
- Nuevas tablas: `offers`, `shipments`.
- Nuevo endpoint: `POST /api/v1/shipments/plan`.

## Preguntas abiertas
- ¿Fuente de distancias reales: API de rutas o matriz precalculada?