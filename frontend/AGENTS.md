# AGENTS.md — Frontend

Lee primero el [`AGENTS.md` raíz](../AGENTS.md). Este archivo solo agrega reglas del frontend.

## Stack

- Astro (TypeScript en modo estricto)
- pnpm como único gestor de paquetes
- Islas de React (`@astrojs/react`) para interacción compleja
- `openapi-typescript` para los tipos de la API
- Vitest y Testing Library para pruebas

## Gestor de paquetes

- Solo **pnpm**. Prohibido usar `npm` o `yarn` y commitear `package-lock.json` o `yarn.lock`.
- `package.json` declara `"packageManager": "pnpm@<versión>"` y se activa con `corepack enable`.
- `pnpm-lock.yaml` siempre se commitea.
- Para agregar dependencias usa `pnpm add` o `pnpm add -D`. Las integraciones de Astro se agregan con `pnpm astro add <integración>`.

## Usuario objetivo

Agricultores con celulares de gama media o baja y conexión inestable:

- **Mobile-first.**
- **HTML estático por defecto; JavaScript solo donde aporta.**
- **Lenguaje simple** y unidades familiares (kilo, bulto, arroba) con su conversión.
- **Accesibilidad:** contraste AA, objetivos táctiles de al menos 44 px, navegación por teclado. Las gráficas incluyen un resumen en texto.

## Cuándo usar cada enfoque

1. **Componente `.astro` sin JavaScript:** contenido, tarjetas, tablas simples, metodología. Es la opción por defecto.
2. **`.astro` con `<script>`:** interacción simple, como mostrar u ocultar un bloque, copiar al portapapeles o un selector sin estado compartido.
3. **Isla de React:** estado complejo o visualización, como gráficas de precios, el simulador "¿Dónde vendo?" o filtros que actualizan varias vistas a la vez.

Si dudas, empieza por el nivel más bajo y sube solo si es necesario.

## Estructura

```
frontend/
├── astro.config.mjs
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json                # extends "astro/tsconfigs/strict"
├── src/
│   ├── pages/                   # rutas: index, simulador, precios, metodologia
│   ├── layouts/                 # BaseLayout.astro
│   ├── components/              # componentes .astro (niveles 1 y 2)
│   ├── islands/                 # componentes React (nivel 3)
│   │   ├── charts/              # gráficas de precios y pronósticos
│   │   └── simulator/           # simulador ¿Dónde vendo?
│   ├── hooks/                   # hooks de React compartidos por las islas
│   ├── lib/
│   │   ├── api/
│   │   │   ├── client.ts        # único punto de acceso a la API
│   │   │   └── types.gen.ts     # generado; no se edita a mano
│   │   └── format.ts            # formato de COP, fechas, unidades
│   ├── content/                 # textos largos con content collections
│   └── styles/
│       └── tokens.css           # colores, espaciado, tipografía
├── public/
└── tests/
```

## ¿Dónde va cada cosa?

- **Nueva página:** `src/pages/`, usando `BaseLayout`.
- **Componente visual sin interacción o con interacción simple:** `src/components/`.
- **Gráfica:** `src/islands/charts/`.
- **Formulario o flujo con estado:** `src/islands/<feature>/`.
- **Lógica reutilizable entre islas:** `src/hooks/`.
- **Llamada a la API:** `src/lib/api/client.ts`. Nunca uses `fetch` directo en componentes, islas ni hooks.
- **Formato de moneda, fecha o unidad:** `src/lib/format.ts`.
- **Colores, espaciado o tipografía:** `src/styles/tokens.css`. Las gráficas también leen los colores de aquí.

## Reglas de islas

- **Hidratación:** `client:visible` por defecto, ideal para gráficas debajo del pliegue. `client:load` solo para el simulador si está arriba en la página. `client:only="react"` solo cuando la librería no soporta SSR, y con justificación.
- **Props serializables:** una isla recibe solo datos planos (JSON). No le pases funciones ni instancias de clases.
- **Una isla, una responsabilidad.** Si dos islas necesitan compartir estado, se unen en una sola o se usa `nanostores`, previo ADR.
- **Librería de gráficas:** una sola en todo el proyecto, elegida mediante ADR y cargada solo dentro de `islands/charts/`.
- **Las islas no importan desde `components/`**; los componentes `.astro` sí pueden usar islas.

## Reglas generales

- **Sin lógica de negocio:** el frontend no calcula precio neto, recomendaciones ni pronósticos. Solo muestra lo que devuelve la API.
- **Tipos:** solo desde `types.gen.ts`. Si falta un campo, se cambia el contrato en el backend primero (Regla 3 de la raíz).
- **Renderizado:** salida estática por defecto. Cambiar a SSR requiere un ADR.
- **Estados obligatorios** en toda vista o isla que consume la API: cargando, error, vacío y datos desactualizados (`data_freshness`).
- **Incertidumbre:** todo pronóstico se muestra con su rango (p10–p90). En las gráficas, como una banda.
- **Configuración:** la URL de la API va en `PUBLIC_API_URL`; ninguna otra variable de entorno es pública.
- **Presupuesto de JavaScript:** páginas sin islas, 0 KB. Páginas con islas, menos de 150 KB comprimidos en la carga inicial.

## Pruebas

- `lib/` y `hooks/`: pruebas unitarias con Vitest.
- Islas: pruebas con Testing Library de sus estados de carga, error, vacío y datos.
- Antes de un PR: `pnpm check`, `pnpm test` y `pnpm build` sin errores.

## Comandos

```bash
corepack enable
pnpm install
pnpm dev
pnpm build
pnpm preview
pnpm check                       # astro check
pnpm test                        # vitest
pnpm gen:api                     # openapi-typescript ../openapi.json -o src/lib/api/types.gen.ts
```

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)
