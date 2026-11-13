# Fuentes de datos

## SIPSA – DANE

Validado en la spike del 2026-11-10 con llamadas reales al servicio web y descargas de los archivos publicados. Las cifras de esta sección son las observadas ese día.

### Servicio web

- **WSDL:** `https://appweb.dane.gov.co/sipsaWS/SrvSipsaUpraBeanService?WSDL`. SOAP 1.2, estilo document, sin autenticación. El WSDL declara la dirección en `http://…:80`, pero también responde por HTTPS.
- **Guía oficial:** [Documentación webservice SIPSA](https://www.dane.gov.co/files/investigaciones/agropecuario/sipsa/DANE-webservice-SIPSA.pdf) (marzo de 2020).
- **Sin filtros:** ningún método de precios o abastecimiento recibe parámetros. Cada llamada devuelve la tabla completa; no se puede pedir por fecha, producto ni central. Los tiempos de la tabla son de descargas simultáneas.

| Método | Contenido | Granularidad | Rango disponible | Respuesta | Registros |
|---|---|---|---|---|---|
| `promediosSipsaParcial` | Precio mínimo, promedio y máximo por kg, por central y producto | Diaria | 2020-02-01 al día en curso | 316 MB, 75 s | 710.058 |
| `promediosSipsaCiudad` | Precio promedio por ciudad y producto | Diaria | 2020-02-03 al día en curso, sin 2021 | 116 MB, 29 s | 395.336 |
| `promediosSipsaSemanaMadr` | Precio mínimo, promedio y máximo por kg, por central | Semanal | Solo las últimas 50 semanas | 66 MB, 21 s | 226.117 |
| `promediosSipsaMesMadr` | Precio mínimo, promedio y máximo por kg, por central | Mensual | 2023-02 al último mes cerrado | 60 MB, 15 s | 199.004 |
| `promedioAbasSipsaMesMadr` | Toneladas por central, producto y mes | Mensual | 2020-02 a 2026-09, sin 2021 | 41 MB, 11 s | 167.685 |
| `consultarInsumosSipsaMesMadr` | Precios de insumos agrícolas | Mensual | No consultado | — | — |

#### Comportamiento verificado

- **Publicación:** la guía indica que los datos diarios se pueden consumir desde las 2:00 p.m. Una llamada a las 4:32 p.m. ya traía los del mismo día. No hay publicación los domingos ni festivos; algunas centrales sí registran datos los sábados.
- **Valores:** `promedioKg` de `promediosSipsaParcial` coincide con el precio del anexo diario publicado. Se compararon las 14 cifras de Corabastos, Cavasa y CMA del 2026-11-09 para los productos del MVP. La guía describe `minimoKg`, `promedioKg` y `maximoKg` como "cantidad recolectada", pero son precios en COP por kg.
- **Precisión:** los precios llegan con hasta 36 decimales (por ejemplo, `2888.888…`). Deben leerse como decimales exactos y redondearse a COP enteros.
- **Bandera `enviado`:** la guía dice que `promedioAbasSipsaMesMadr` "retorna los registros que no han sido consultados". No ocurre: dos llamadas seguidas devolvieron respuestas idénticas byte a byte, y en `promediosSipsaCiudad` todos los registros tienen `enviado=0`. Releer es seguro.
- **Clave natural:** en `promediosSipsaParcial` hay un registro por (`enmaFecha`, `idArtiSemana`, `fuenId`) en todas las series del MVP revisadas. `promediosSipsaCiudad` llega a tener 2 registros por ciudad, producto y día.
- **Códigos de producto:** cambian entre métodos. `promediosSipsaParcial` usa `idArtiSemana` (canasta diaria de unos 35 productos); los métodos semanal, mensual y de abastecimiento usan `artiId`, con variedades más detalladas. Por ejemplo, "Tomate\*" en la canasta diaria frente a "Tomate chonto" (`artiId` 45). El asterisco significa "variedad predominante en el mercado".
- **Correcciones posteriores:** no se pudieron comprobar con una sola descarga. Pendiente: comparar dos descargas separadas por varios días.
- **Fixtures:** muestras recortadas de las respuestas reales en [`backend/tests/fixtures/sipsa/`](../../backend/tests/fixtures/sipsa/README.md).

#### Identificadores del MVP en `promediosSipsaParcial`

| Producto | `idArtiSemana` |
|---|---|
| Cebolla cabezona blanca | 3 |
| Tomate\* | 12 |
| Zanahoria | 13 |
| Papa negra\* | 32 |
| Plátano hartón verde | 35 |

#### Centrales y cobertura

El [catálogo de centrales](markets.md) registra las 24 centrales con precios diarios, con su `fuenId`, días de recolección, cobertura de los productos del MVP y meses de abastecimiento.

- Solo Corabastos y la CMA tienen datos casi todos los días. La mayoría de las centrales se recolectan algunos días de la semana, así que sus series son irregulares.
- El hueco más largo de Cali Cavasa va del 2021-04-28 al 2021-05-22 (24 días). Coincide con el paro nacional de 2021.

### Correspondencia de productos con el abastecimiento

Propuesta, pendiente de validar en la evaluación. Precios y abastecimiento usan catálogos distintos: el precio diario es de un producto genérico ("Papa negra\*" es la variedad predominante en cada central) y el abastecimiento viene por variedad (`artiId`). Los porcentajes son sobre las toneladas de los últimos 12 meses con datos (2025-05 a 2026-09) en las centrales del MVP.

| Producto (precio) | Variedades que suman (`artiId`) | Se excluyen | Observaciones |
|---|---|---|---|
| Papa negra\* | Todas las papas salvo la criolla: superior (162), capira (157), única (179), sabanera (173), R-12 (543), parda pastusa (167), rubí (172), suprema (175), nevada (165), Betina (5200), Morasurco (164) y "Papas negras otras" (498) | Papa criolla (541), que tiene su propio precio, y las papayas (136, 137, 537, 6002, 6100) | La variedad dominante cambia por central: la superior es el 84 % en Corabastos y el 68 % en Centroabastos; la capira es el 94 % en la CMA |
| Tomate\* | Chonto (45), Riogrande (51), larga vida (48) y "Tomates otros" (495) | Tomate de árbol (148), que es otra fruta y representa entre el 12 % y el 22 % de todo lo que se llama "tomate" | El chonto es entre el 83 % y el 98 % del tomate |
| Cebolla cabezona blanca | Cabezona blanca (18) y roja (24) desde 2022-07; "Cebolla Cabezona" (516) hasta 2022-06 | Junca (29) y puerro (34), que son otros productos; la junca es entre el 14 % y el 50 % de todo lo que se llama "cebolla" | En 2022-07 SIPSA reemplazó el código genérico 516 por blanca y roja. La roja es el 55 % de la cebolla cabezona en la CMA |
| Plátano hartón verde | Hartón verde (187) | Guineo (185), que tiene su propio precio, y "Plátanos otros" (499) | El hartón verde es entre el 88 % y el 100 % del plátano |
| Zanahoria | Zanahoria (54) | — | Correspondencia directa |

Cuidados:

- **Nunca por coincidencia de texto:** "papa" también encuentra "Papaya", y "tomate" encuentra el tomate de árbol. Pasó en esta misma spike: el primer análisis contó la papaya entre los productos del MVP. La correspondencia debe ser una tabla explícita por `artiId`, revisada por una persona.
- **Los códigos tienen vigencia:** en 2022-07 SIPSA separó la cebolla cabezona en blanca y roja, y agregó variedades como la papa sabanera. La tabla debe registrar desde qué mes aplica cada código.
- **Meses faltantes:** además de 2021, faltan otros meses. Los últimos 12 meses con datos abarcan 17 meses calendario.

### Archivos publicados

- **Anexo diario de precios (Excel):** uno por día de publicación, con páginas mensuales desde 2012. Trae solo el precio promedio en $/kg y la variación, para unos 35 productos y 14 centrales en 2026.
  - El nombre y la ruta cambian entre años: `…/sipsa/mayoristas_enero_15_2020.xlsx` en 2020, `…/SIPSA/anex-SIPSADiario-09nov2026.xlsx` desde 2024. Hay que recorrer las páginas mensuales en lugar de construir la URL.
  - Las centrales incluidas cambian con el tiempo, y la escala de la variación también: en 2020 venía en porcentaje y en 2026 como fracción.
- **Microdatos de precios mayoristas 2013–2024:** [catálogo 776](https://microdatos.dane.gov.co/index.php/catalog/776). No se descargaron. Sus condiciones prohíben "la copia o reproducción de los datos para que múltiples usuarios accedan a ellos sin autorización escrita del DANE".
- **Abastecimiento, microdatos anuales (Excel, 2013–2026):** registros diarios con fecha, central, departamento y municipio de origen, grupo, código CPC, alimento y kilogramos.
  - El archivo de 2026 pesa 94 MB y tiene unos 2 millones de filas, repartidas en una hoja por cuatrimestre.
  - Se publica con unas 3 semanas de retraso: la primera quincena de octubre de 2026 salió el 5 de noviembre.

### Implicaciones para el MVP

Salvo que se indique lo contrario, están pendientes de decisión y no modifican todavía los demás documentos.

1. **Precios diarios (decidido el 2026-11-10):** `promediosSipsaParcial` cubre la ingesta diaria y el histórico, con mínimo, promedio y máximo (encaja con `prices`). Se guarda todo desde 2020-02-01. Los anexos anteriores, que solo traen el promedio, no se cargan. La ventana de entrenamiento se elige en la evaluación ([evaluation.md](../06-ml/evaluation.md)).
2. **Volumen por ejecución:** cada ingesta descarga unos 316 MB para obtener un día. Con `zeep`, la respuesta mensual de 60 MB tardó 51,5 s y subió la memoria del proceso en 474 MiB; extrapolando, la de 316 MB rondaría 2,5 GiB (estimación, no medida). Decidido en [ADR-0005](../02-adr/0005-cliente-http-streaming-sipsa.md): cliente HTTP con lectura por streaming.
3. **Abastecimiento (decidido el 2026-11-10):** el MVP usa el dato mensual del servicio web (`promedioAbasSipsaMesMadr`), que llega con unos dos meses de retraso y no tiene 2021. Los microdatos diarios en Excel quedan para una fase posterior. El modelo solo usa meses ya publicados en la fecha del pronóstico. La correspondencia entre productos de precio y variedades de abastecimiento está [propuesta más arriba](#correspondencia-de-productos-con-el-abastecimiento), pendiente de validar.
4. **Centrales (decidido el 2026-11-10):** el MVP usa Corabastos (`fuenId` 1), la CMA (56) y Centroabastos (36). Las tres reportan los cinco productos: Corabastos y la CMA casi a diario, Centroabastos unos 4 días por semana. Las demás quedan registradas en el [catálogo de centrales](markets.md) para cuando se escale.
5. **Frescura:** como no hay publicación los domingos ni festivos, un fin de semana con festivo deja más de 48 h sin datos nuevos. Una alerta de frescura medida en horas de calendario saltaría sin que haya falla.
6. **Términos de uso:** los [términos del portal del DANE](https://www.dane.gov.co/index.php/terrminos-y-condiciones-dane) autorizan "el uso, aprovechamiento, transformación y análisis de la información", siempre con la cita textual "Fuente: Departamento Administrativo Nacional de Estadística: www.dane.gov.co". No mencionan la redistribución a múltiples usuarios ni el uso comercial por terceros, y las condiciones de los microdatos sí restringen la redistribución. Pendiente: consultar por escrito a contacto@dane.gov.co si el explorador de precios (RF-11) puede republicar el histórico. No bloquea el desarrollo; debe resolverse antes de publicarlo.

## Datos del usuario
El flete, la comisión y el precio del intermediario los ingresa el usuario. No se persisten en el MVP.
