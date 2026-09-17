# Cambios

## 1.4.3 — 2026-09-17

**Plugin**
- `/diagnostics/malware`: busca código típico de webshell en plugins y temas, PHP dentro de `uploads`
  y archivos del núcleo de WordPress modificados o faltantes (sumas de verificación de wordpress.org).
- `/status` informa qué manifiesto de actualización está viendo el sitio (`update_manifest`), para
  entender por qué un sitio no ve una versión nueva del plugin.

**Orquestador**
- `malware --site X [--path]`: lista los hallazgos separando lo grave de lo que solo hay que mirar.
- Suite de pruebas `malware`.

## 1.4.2 — 2026-09-17

**Plugin**
- `/inventory` informa el tema padre de cada tema (`parent`).

**Orquestador**
- Los temas hijo y los plugins listados en `licensed` del sitio ya no se avisan como truchos: muchos
  premium con licencia solo avisan a WordPress cuando hay versión nueva.
- `inventory` imprime las carpetas de los posibles truchos para copiarlas a `licensed`.

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
