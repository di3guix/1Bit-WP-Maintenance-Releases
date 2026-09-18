# 1Bit WP Maintenance · versiones publicadas

Versiones firmadas del plugin **`1bit-wp-maintenance`**, que instalan y actualizan los sitios
WordPress mantenidos por [1Bit](https://1bit.com.ar).

> Este repositorio **no contiene código fuente**. El código y las herramientas de
> mantenimiento están en un repositorio privado.

**Última versión: 1.4.9** (2026-09-18)

---

## Qué hace el plugin

Permite mantener un sitio WordPress a distancia **sin SSH ni FTP**, desde la REST API:

- actualiza plugins y temas de a uno, con **backup** previo y **snapshot cifrado de la base**;
- verifica que el sitio siga funcionando (error fatal, chequeo externo, recorrido de la tienda);
- si algo se rompe, **vuelve solo a la versión anterior**, incluso con el sitio caído;
- informa licencias vencidas, plugins sin canal de actualización y uso de archivos.

Cada pedido exige una contraseña de aplicación de WordPress y un token propio.

---

## Cómo se actualizan los sitios

1. Se publica una versión nueva en este repositorio: el zip en `releases/` y `update.json`.
2. Cada sitio consulta `update.json` cada 6 horas y WordPress la muestra como actualización pendiente.
3. El mantenimiento diario la instala **al final**, con el mismo ciclo de backup, verificación y
   rollback que cualquier otro plugin.
4. Si la versión nueva tira un error fatal o no levanta su REST API, el plugin vuelve solo a la anterior.

## Cómo se verifica cada versión

`update.json` trae la versión, la URL del zip, su **SHA-256** y una **firma Ed25519** de esos datos.
La clave privada no está en este repositorio; el plugin trae la pública. Antes de instalar,
cada sitio comprueba:

- que la **firma** del manifiesto sea válida: si no, ni siquiera ofrece la actualización;
- que el **SHA-256** del zip descargado sea el firmado: si no coincide, no lo instala.

Aunque alguien pudiera modificar este repositorio, un zip alterado no se instala en ningún sitio.

<details>
<summary>Verificar una versión a mano</summary>

Mensaje firmado: `1bit-wp-maintenance`, versión, SHA-256 y URL del paquete, separados por un salto de línea.

```python
import base64, hashlib, json, urllib.request
import nacl.signing  # pip install pynacl

PUBLIC_KEY = "UqFk2NIIt7Xq75t1Wx9teYObIXOZyM9d+uJNEJyT6Po="
m = json.load(urllib.request.urlopen("https://raw.githubusercontent.com/di3guix/1Bit-WP-Maintenance-Releases/main/update.json"))
zip_bytes = urllib.request.urlopen(m["package"]).read()
assert hashlib.sha256(zip_bytes).hexdigest() == m["sha256"], "el zip no coincide"
message = "\n".join(["1bit-wp-maintenance", m["version"], m["sha256"], m["package"]]).encode()
nacl.signing.VerifyKey(base64.b64decode(PUBLIC_KEY)).verify(message, base64.b64decode(m["signature"]))
print("versión", m["version"], "verificada")
```
</details>

---

## Instalación manual

Para un sitio nuevo, o uno con una versión anterior a la 1.4.0 (que no sabe autoactualizarse):

1. Descargar el zip de la última versión (tabla de abajo).
2. En WordPress: **Plugins → Añadir nuevo → Subir plugin** → elegir el zip.
3. Si ya estaba instalado: **Reemplazar con la versión subida**. No se pierden el token, los
   backups ni la configuración.

---

## Versiones

| Versión | Fecha | Descarga | SHA-256 |
|---|---|---|---|
| **1.4.9** | 2026-09-18 | [zip](releases/1bit-wp-maintenance-1.4.9.zip) | `0bd7db1ed6e1fd3b…` |
| **1.4.8** | 2026-09-17 | [zip](releases/1bit-wp-maintenance-1.4.8.zip) | `5d072c7afd171659…` |
| **1.4.7** | 2026-09-17 | [zip](releases/1bit-wp-maintenance-1.4.7.zip) | `cbc6ae50bbb818db…` |
| **1.4.6** | 2026-09-17 | [zip](releases/1bit-wp-maintenance-1.4.6.zip) | `52ca2341e93272cd…` |
| **1.4.5** | 2026-09-17 | [zip](releases/1bit-wp-maintenance-1.4.5.zip) | `d0add65f1869bca2…` |
| **1.4.4** | 2026-09-17 | [zip](releases/1bit-wp-maintenance-1.4.4.zip) | `a956b331700ada3a…` |
| **1.4.3** | 2026-09-17 | [zip](releases/1bit-wp-maintenance-1.4.3.zip) | `de9ab114a26ec206…` |
| **1.4.2** | 2026-09-17 | [zip](releases/1bit-wp-maintenance-1.4.2.zip) | `c29d5bd8cd973aa8…` |
| **1.4.1** | 2026-09-17 | [zip](releases/1bit-wp-maintenance-1.4.1.zip) | `5c670e3ae2510074…` |
| **1.4.0** | 2026-09-17 | [zip](releases/1bit-wp-maintenance-1.4.0.zip) | `f46f82b5260878fa…` |

## Cambios

### 1.4.9 — 2026-09-18

**Plugin**
- `/licenses/inspect`: licencia guardada de un plugin o tema (sin devolver la clave, solo si existe
  y qué estado dice) y marcas de copias piratas en su código.
- `/licenses/requests`: a qué fabricante le consulta cada plugin por actualizaciones y qué responde.

### 1.4.8 — 2026-09-17

**Plugin**
- El control del núcleo compara contra el paquete del idioma del sitio: en un WordPress en español,
  `version.php` y otros archivos son distintos a los del paquete en inglés y salían como modificados.

### 1.4.7 — 2026-09-17

**Plugin**
- `/diagnostics/file`: muestra un archivo del sitio para revisar a mano lo que marcó el escaneo
  (solo lectura, acotado y con usuario administrador).

**Orquestador**
- `show --site X --path archivo`.
- `malware_ignore` por sitio en `sites.json`: rutas legítimas conocidas, como las plantillas que
  el framework Redux escribe en `uploads`.

### 1.4.6 — 2026-09-17

**Plugin**
- El manifiesto se pide con cabeceras contra la caché: varios servidores seguían recibiendo una
  copia vieja de `update.json` y no veían las versiones nuevas.

### 1.4.5 — 2026-09-17

**Plugin**
- Se corrige la detección de `preg_replace` con modificador `/e`: marcaba como grave cualquier
  patrón que tuviera barras y comillas adentro (por ejemplo el de UpdraftPlus).

### 1.4.4 — 2026-09-17

**Plugin**
- La ofuscación (texto en hexadecimal, `chr()`, base64 largo) solo se informa si en el mismo archivo
  hay algo que ejecute código: WooCommerce, phpseclib y otras librerías serias la usan y llenaban el
  informe de falsos positivos.

### 1.4.3 — 2026-09-17

**Plugin**
- `/diagnostics/malware`: busca código típico de webshell en plugins y temas, PHP dentro de `uploads`
  y archivos del núcleo de WordPress modificados o faltantes (sumas de verificación de wordpress.org).
- `/status` informa qué manifiesto de actualización está viendo el sitio (`update_manifest`), para
  entender por qué un sitio no ve una versión nueva del plugin.

**Orquestador**
- `malware --site X [--path]`: lista los hallazgos separando lo grave de lo que solo hay que mirar.
- Suite de pruebas `malware`.

### 1.4.2 — 2026-09-17

**Plugin**
- `/inventory` informa el tema padre de cada tema (`parent`).

**Orquestador**
- Los temas hijo y los plugins listados en `licensed` del sitio ya no se avisan como truchos: muchos
  premium con licencia solo avisan a WordPress cuando hay versión nueva.
- `inventory` imprime las carpetas de los posibles truchos para copiarlas a `licensed`.

### 1.4.1 — 2026-09-17

**Plugin**
- `/diagnostics/files` desglosa los archivos por tipo (miniaturas, WebP/AVIF generados, backups del
  optimizador de imágenes, PHP, logs…) y acepta `path` para contar solo una carpeta.

**Orquestador**
- `files --path`: tabla por tipo y aviso si hay PHP dentro de `uploads`.
- Suite de pruebas `files`.
- `publish` arma el zip desde la misma carpeta de la que lee la versión.

> Primera versión que llega sola a los sitios con la autoactualización.

### 1.4.0 — 2026-09-17

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

---

## Archivos

| Archivo | Qué es |
|---|---|
| `update.json` | Última versión publicada, con URL, SHA-256 y firma. Es lo que consultan los sitios |
| `releases/` | Zips de cada versión |
| `history.json` | Todas las versiones publicadas, con fecha, hash y notas |
| `CHANGELOG.md` | Qué cambió en cada versión |
