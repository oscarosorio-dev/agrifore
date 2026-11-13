# Catálogo de centrales

Registro de todas las centrales de abasto con precios diarios en SIPSA, para decidir qué centrales agregar cuando el sistema escale. Las del MVP están marcadas.

- **Fuente:** `promediosSipsaParcial` (precios diarios) y `promedioAbasSipsaMesMadr` (abastecimiento mensual). Ver [sources.md](sources.md).
- **Fecha de corte:** 2026-11-10. Las métricas usan el último año (2025-11-10 a 2026-11-10): 310 días con publicación en 53 semanas.
- **Histórico:** todas las centrales tienen precios desde los primeros días de febrero de 2020.

## Centrales con precios diarios

| `fuenId` | Central | Municipio (depto.) | Días/sem. | Días habituales | Productos | Cobertura MVP | Sin dato (MVP) | Abast. (meses) | Estado |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Bogotá, D.C., Corabastos | Bogotá, D.C. | 5,8 | lun–sáb | 35 | 100 % | — | 57 | **MVP** |
| 56 | Medellín, Central Mayorista de Antioquia | Medellín (Antioquia) | 5,5 | lun–sáb | 36 | 95 % | — | 57 | **MVP** |
| 36 | Bucaramanga, Centroabastos | Bucaramanga (Santander) | 3,8 | mar, jue, vie, sáb | 33 | 65 % | — | 57 | **MVP** |
| 26 | Cúcuta, Cenabastos | San José de Cúcuta (Norte de Santander) | 3,9 | mar, jue, vie, sáb | 32 | 53 % | papa negra | 57 | Disponible |
| 19 | Montería, Mercado del Sur | Montería (Córdoba) | 2,9 | lun, mié, vie | 27 | 49 % | — | 56 | Disponible |
| 32 | Pereira, Mercasa | Pereira (Risaralda) | 3,6 | lun, mié, jue, vie | 33 | 49 % | zanahoria | 57 | Disponible |
| 16 | Valledupar, Mercabastos | Valledupar (Cesar) | 2,8 | lun, mié, vie | 24 | 48 % | — | 53 | Disponible |
| 41 | Sincelejo, Nuevo Mercado | Sincelejo (Sucre) | 2,8 | lun, mié, vie | 24 | 47 % | — | 57 | Disponible |
| 29 | Armenia, Mercar | Armenia (Quindío) | 2,7 | lun, mar, jue | 34 | 46 % | — | 57 | Disponible |
| 1836 | Santa Marta (Magdalena) | Santa Marta (Magdalena) | 4,7 | lun–vie | 15 | 46 % | tomate, papa negra | 56 | Disponible |
| 23 | Villavicencio, CAV | Villavicencio (Meta) | 2,9 | lun, mié, vie | 31 | 39 % | tomate | 57 | Disponible |
| 43 | Ibagué, Plaza La 21 | Ibagué (Tolima) | 2,8 | mar, jue, vie | 29 | 39 % | papa negra | 56 | Disponible |
| 7 | Cartagena, Bazurto | Cartagena de Indias (Bolívar) | 2,8 | lun, mié, vie | 24 | 39 % | tomate | 57 | Disponible |
| 22 | Neiva, Surabastos | Neiva (Huila) | 3,8 | mar, jue, vie, sáb | 25 | 39 % | cebolla, papa negra | 57 | Disponible |
| 71 | Barranquilla, Barranquillita | Barranquilla (Atlántico) | 2,8 | lun, mié, vie | 24 | 38 % | zanahoria | 56 | Disponible |
| 9 | Tunja, Complejo de Servicios del Sur | Tunja (Boyacá) | 2,9 | mar, jue, vie | 28 | 30 % | tomate, papa negra | 55 | Disponible |
| 24 | Pasto, El Potrerillo | Pasto (Nariño) | 2,9 | lun, mié, jue | 22 | 30 % | papa negra, plátano | 54 | Disponible |
| 70 | Barranquilla, Granabastos | Barranquilla (Atlántico) | 2,8 | lun, mié, vie | 18 | 29 % | zanahoria, plátano | 57 | Disponible |
| 47 | Cali, Cavasa | Cali (Valle del Cauca) | 2,0 | lun, jue | 31 | 27 % | zanahoria | 57 | Disponible |
| 15 | Popayán, Plaza de mercado del barrio Bolívar | Popayán (Cauca) | 1,9 | mié, vie | 27 | 25 % | zanahoria | 56 | Disponible |
| 33 | Pereira, La 41-Impala¹ | Pereira (Risaralda) | 1,9 | mar, jue | 29 | 23 % | zanahoria | 36 | Disponible |
| 48 | Cali, Santa Elena² | Cali (Valle del Cauca) | 2,0 | mar, sáb | 29 | 20 % | tomate, zanahoria | 57 | Disponible |
| 13 | Manizales, Centro Galerías | Manizales (Caldas) | 2,0 | lun, jue | 31 | 20 % | zanahoria, plátano | 57 | Disponible |
| 17 | Valledupar, Mercado Nuevo | Valledupar (Cesar) | 2,8 | lun, mié, vie | 10 | 19 % | cebolla, zanahoria, plátano | 56 | Disponible |

¹ Antes "Pereira, La 41". ² Antes "Cali, Santa Helena". SIPSA cambió el nombre y conservó el `fuenId`, así que el catálogo debe identificar la central por `fuenId` y no por nombre.

### Cómo leer las columnas

- **Días/sem.:** promedio de días con precios por semana.
- **Días habituales:** días de la semana con datos en al menos la mitad de las semanas.
- **Productos:** cuántos productos de la canasta diaria (unos 35) reportó la central en el último año.
- **Cobertura MVP:** promedio, sobre los cinco productos del MVP, del porcentaje de días de publicación con dato.
- **Sin dato (MVP):** productos del MVP que la central no reportó en el último año.
- **Abast. (meses):** meses con abastecimiento mensual entre 2020-02 y 2026-09.

## Centrales solo con abastecimiento

Tienen abastecimiento mensual, pero no precios diarios, así que no se pueden pronosticar con el diseño actual.

| `fuenId` | Central | Abast. (meses) |
|---|---|---|
| 2 | Bogotá, D.C., Paloquemao | 57 |
| 4 | Bogotá, D.C., Plaza Las Flores | 57 |
| 4389 | Bogotá, D.C., Plaza Samper Mendoza | 56 |
| 57 | Medellín, Plaza Minorista "José María Villa" | 57 |
| 1274 | Cúcuta, La Nueva Sexta | 57 |
| 1693 | Ipiales (Nariño), Centro de acopio | 56 |
| 1990 | Florencia (Caquetá) | 39 |
| 12 | Tibasosa (Boyacá), Coomproriente | 38 |

Los métodos semanal y mensual de precios cubren más mercados (80 y 147), pero no sirven para pronosticar a 1–7 días.

## Agregar una central

1. Revisar su fila en este catálogo. Con menos días por semana, las series son más irregulares y el error se mide con menos datos.
2. Registrarla en la tabla `markets` con `sipsa_id` igual a su `fuenId`. SIPSA no publica coordenadas, así que `lat` y `lon` hay que obtenerlas de otra fuente.
3. Como el modelo es global, no hace falta un modelo nuevo. El diagnóstico por serie de `model_metrics` debe mostrar que sus series superan al baseline (ADR-0004).
4. Cambiar su estado a **MVP** (o al nombre de la fase) en este catálogo y registrar el cambio en `CHANGELOG.md`.

Cuando exista la ingesta, este catálogo debe regenerarse a partir de la base de datos.
