# Cambios

## 1.4.1 — 2026-09-17

**Plugin**
- `/diagnostics/files` desglosa los archivos por tipo (miniaturas, WebP/AVIF generados, backups del
  optimizador de imágenes, PHP, logs…) y acepta `path` para contar solo una carpeta.

**Orquestador**
- `files --path`: tabla por tipo y aviso si hay PHP dentro de `uploads`.
- Suite de pruebas `files`.
- `publish` arma el zip desde la misma carpeta de la que lee la versión.

> Primera versión que llega sola a los sitios con la autoactualización.

## 1.4.0 — 2026-09-17

**Plugin**
- Autoactualización firmada desde `1Bit-WP-Maintenance-Releases`: verifica la firma Ed25519 del
  manifiesto y el SHA-256 del zip antes de instalar.
- El propio plugin se actualiza al final de cada corrida y, si la versión nueva no levanta su REST
  API o tira un error fatal, vuelve sola a la anterior.
- Cabecera `Update URI` para que WordPress no consulte wordpress.org por este slug.

**Orquestador**
- `keygen` y `publish` para firmar y publicar versiones.
- Pruebas dentro del repositorio (`tests/`): `bootstrap.py` y `run_all.py`, más la suite `autoupdate`.

> Es la última versión que hay que subir a mano: las anteriores no saben autoactualizarse.
