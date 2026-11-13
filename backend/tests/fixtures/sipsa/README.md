# Fixtures de SIPSA

Respuestas reales del servicio web de SIPSA, grabadas el 2026-11-10 a las 4:32 p.m. (hora de Colombia) durante la spike de validación. Ver [docs/05-data/sources.md](../../../../docs/05-data/sources.md).

Las pruebas usan estos archivos y nunca llaman al servicio real.

## Archivos

| Archivo | Contenido |
|---|---|
| `wsdl.xml` | WSDL completo del servicio |
| `xsd1.xml` | Esquema XSD que importa el WSDL |
| `promediosSipsaParcial.xml` | 26 de los 710.058 registros de la respuesta original (316 MB) |
| `promedioAbasSipsaMesMadr.xml` | 11 de los 167.685 registros de la respuesta original (41 MB) |

## Cómo se recortaron

Se copiaron sin modificar los bytes del sobre SOAP y de cada registro `<return>` elegido; solo se quitaron los demás registros. No hay fixtures de errores SOAP ni de respuestas vacías porque no se observaron durante la spike.

### `promediosSipsaParcial.xml`

Productos del MVP (`idArtiSemana` 3, 12, 13, 32 y 35) en Corabastos (`fuenId` 1), Cavasa (47) y CMA (56), para los días 2026-11-09 y 2026-11-10. Casos que cubre:

- **Central que no reporta todo:** Cavasa no tiene zanahoria el 2026-11-09 ni ningún dato el 2026-11-10.
- **Día en curso:** el 2026-11-10 era el día de la descarga.
- **Decimales largos:** por ejemplo, el mínimo de zanahoria en CMA y el de tomate en Corabastos.
- **Filtros:** un producto fuera del MVP (Aguacate\*, `idArtiSemana` 14, en Corabastos) y una central fuera del MVP (Armenia Mercar, `fuenId` 29).

Los precios promedio del 2026-11-09 coinciden con el anexo diario que publicó el DANE ese día.

### `promedioAbasSipsaMesMadr.xml`

Toneladas de septiembre de 2026, el último mes disponible, en Corabastos y CMA. Incluye papa criolla, tomate chonto, cebolla cabezona blanca, plátano hartón verde y zanahoria en las dos centrales, y papa superior solo en Corabastos. Usa `artiId` y nombres de variedad distintos a los de `promediosSipsaParcial.xml`.

## Actualización

Si el DANE cambia el contrato del servicio, graba respuestas nuevas, reemplaza estos archivos y actualiza este README y `docs/05-data/sources.md`.
