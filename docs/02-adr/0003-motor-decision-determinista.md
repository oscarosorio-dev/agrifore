# ADR-0003: Motor de decisión determinista, sin LLM

- **Estado:** Aceptado

## Decisión
La recomendación se calcula con reglas explícitas (`precio_neto = p50 - flete - comisión`), considerando la incertidumbre (p10–p90).

## Razón
Una decisión económica debe ser auditable, reproducible y testeable. En la Fase 3, el LLM solo redactará la respuesta a partir de la salida de este motor.

## Consecuencias
Sin riesgo de alucinación en el cálculo. La explicación de la recomendación también se construye de forma estructurada.