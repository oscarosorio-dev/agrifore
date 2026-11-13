# Proceso de release

- **Versionado:** SemVer (`MAJOR.MINOR.PATCH`).
- **Changelog:** `CHANGELOG.md` en la raíz, generado a partir de Conventional Commits.

## Pasos
1. CI en verde sobre `main`.
2. Actualizar `CHANGELOG.md`.
3. Crear el tag `vX.Y.Z` y el release en GitHub.
4. El build publica las imágenes Docker con el tag de la versión.
5. Ejecutar las migraciones y verificar `/health`.

## Rollback
Desplegar la imagen del tag anterior. Las migraciones deben ser compatibles hacia atrás durante una versión.