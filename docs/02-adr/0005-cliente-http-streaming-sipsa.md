# ADR-0005: Cliente HTTP con lectura por streaming para SIPSA

- **Estado:** Aceptado
- **Fecha:** 2026-11-10

## Contexto
El servicio web de SIPSA (SOAP 1.2) no recibe parámetros: cada llamada devuelve la tabla completa. Al 2026-11-10, `promediosSipsaParcial` pesa 316 MB (710.058 registros) y crece unos 100.000 registros por año. El stack preveía `zeep`. En la spike, `zeep` tardó 51,5 s y subió la memoria del proceso en 474 MiB con una respuesta de 60 MB ([sources.md](../05-data/sources.md)).

## Decisión
- `ingestion/sipsa_client.py` envía el sobre SOAP con `httpx` y escribe la respuesta en un archivo temporal a medida que llega, sin cargarla en memoria.
- El XML se lee registro por registro con `defusedxml.ElementTree.iterparse`, liberando cada elemento después de procesarlo. `defusedxml` bloquea DTD y entidades, que una respuesta externa no debe traer.
- Durante la lectura solo se conservan los registros de los productos y centrales del catálogo activo; el resto se descarta.
- Cada método tiene su propia conversión de campos. Los precios se leen como `Decimal` y se redondean a COP enteros; las fechas se convierten a `date`.
- La descarga tiene timeouts explícitos de conexión y de lectura, y reintentos con espera.

## Alternativas consideradas
- **`zeep`:** genera el cliente desde el WSDL, convierte los tipos y valida contra el esquema, pero carga la respuesta completa como objetos: 474 MiB para 60 MB y unos 2,5 GiB estimados para 316 MB. Su ventaja principal, armar llamadas con parámetros, no aplica porque ningún método los tiene.
- **`lxml.etree.iterparse`:** más rápido, pero hay que configurar a mano las protecciones contra entidades externas. Se puede reconsiderar si el tiempo de lectura llega a ser un problema.
- **Leer la respuesta en memoria con `ElementTree.parse`:** es simple, pero la memoria crece con la respuesta.
- **Leer directamente del stream, sin archivo temporal:** ahorra disco, pero obliga a descargar otra vez si la lectura falla.

## Consecuencias
- Por diseño, la memoria del worker no depende del tamaño de la respuesta.
- El worker necesita espacio temporal para la respuesta: 316 MB hoy, y crece.
- El sobre SOAP y la conversión de campos se escriben a mano. Ningún esquema detecta un cambio de contrato del DANE: lo detectan las pruebas contra los fixtures de `backend/tests/fixtures/sipsa/` y la validación de campos obligatorios durante la lectura. Un registro sin fecha, central, producto o precio cuenta como rechazado en `ingestion_records_total{status}`.
- `zeep` sale del stack; entran `httpx` (ya lo usa el `TestClient` de FastAPI en las pruebas) y `defusedxml`.
- Se agregan métricas de tamaño de la respuesta y duración de la descarga ([observability.md](../08-operations/observability.md)).
