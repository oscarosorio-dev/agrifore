---
description: "Estándares obligatorios de codificación, tipado estricto y estilo PEP 8 para Python"
trigger: auto
paths:
  - "**/*.py"
---

# 📜 Estándar Estricto de Codificación - Python

Esta regla rige de forma absoluta la implementación de código Python. Toda solución que viole estas directrices será rechazada.

## 1. Tipado Estricto (Type Hinting)
- **Tipado Obligatorio:** Todas las funciones y métodos deben incluir obligatoriamente anotaciones de tipo (*type hints*) tanto para sus argumentos como para su valor de retorno (ej. `def get_user(user_id: int) -> User | None:`).
- **Prohibición de 'Any':** Queda estrictamente prohibido el uso de `typing.Any`. Si el tipo es verdaderamente dinámico o desconocido, debes utilizar `typing.object` y verificar sus atributos mediante inspección en tiempo de ejecución.
- **Colecciones Genéricas:** No uses los tipos genéricos nativos crudos en firmas si estás en versiones anteriores a 3.10. Especifica siempre el contenido de las colecciones utilizando la sintaxis moderna (ej. `list[str]` o `dict[str, int]`).

## 2. Manejo de Errores y Robustez
- **Excepciones Específicas:** Queda estrictamente prohibido usar bloques `except:` vacíos o capturar de forma genérica `except Exception:`, a menos que sea el nivel más alto de un manejador global de logs. Captura siempre la excepción específica esperada (ej. `except ValueError:` o `except KeyError:`).
- **Estructura Con Contexto:** Para el manejo de recursos externos (I/O, sockets, conexiones a bases de datos), es obligatorio el uso de administradores de contexto mediante la sentencia `with`.

## 3. Estilo y Sintaxis (PEP 8 Avanzado)
- **Nomenclatura:** 
  - Clases: `PascalCase` (ej. `DatabaseConnection`).
  - Funciones, Métodos, Variables y Atributos: `snake_case` (ej. `process_data`).
  - Constantes: `UPPER_SNAKE_CASE` (ej. `MAX_RETRY_ATTEMPTS`).
- **Docstrings:** Todas las clases, funciones públicas y módulos deben incluir un Docstring descriptivo bajo el formato Google Style Docs, detallando argumentos, retornos y excepciones lanzadas.
- **Asincronía:** Si el proyecto interactúa con I/O intensivo, prefiere el uso de funciones nativas asíncronas (`async def` y `await`) bajo arquitecturas no bloqueantes.
