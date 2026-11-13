# Requisitos funcionales

## Ingesta
- **RF-01:** Ingerir diariamente los precios mayoristas desde SIPSA.
- **RF-02:** Ingerir los datos de abastecimiento (toneladas por central).
- **RF-03:** Normalizar productos, centrales y unidades a un catálogo interno.
- **RF-04:** Permitir la carga histórica por rango de fechas.

## Pronóstico
- **RF-05:** Generar un pronóstico de 1 a 7 días por producto y central.
- **RF-06:** Incluir un intervalo de predicción (p10–p90) en cada pronóstico.
- **RF-07:** Reentrenar los modelos semanalmente y versionarlos.

## Decisión
- **RF-08:** Calcular el precio neto por central a partir de producto, cantidad, origen, flete y comisión.
- **RF-09:** Comparar contra el precio del intermediario cuando el usuario lo ingrese.
- **RF-10:** Recomendar la central y el día de venta, con la justificación de la recomendación: el desglose de precio esperado, flete, comisión y precio neto de cada opción.

## Consulta
- **RF-11:** Consultar el histórico de precios por producto y central.
- **RF-12:** Simulador web "¿Dónde vendo?".
- **RF-13:** Publicar la metodología y los resultados de evaluación en la web.