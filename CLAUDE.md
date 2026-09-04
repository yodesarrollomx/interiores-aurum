# Llave Maestra — tablero del programa de interiores de Aurum (Residencia Navarro Muñoz)

Lee este archivo completo antes de tocar nada.

## Qué es

Tablero web de **Aurum Arquitectos** donde el cliente recorre su programa de interiores espacio por espacio: renders, acabados, mobiliario, luminarias, carpintería, herrajes, textiles, decoración y equipamiento. Elige opciones A/B, marca qué incluye, ve el total y exporta PDF. Quien puede editar (Dirección y Proyectos/Sayri) agrega, edita, oculta y borra piezas **escribiendo directo al Google Sheet**.

- **Dirección en vivo:** https://yodesarrollomx.github.io/interiores-aurum/llave-maestra.html — **HTTP 200, 199,011 bytes (curl 2026-09-04)**. El `llave-maestra.html` desplegado es byte por byte el del repo (mismo sha256 `2185a1f6…`, comprobado 2026-09-04).
- `https://yodesarrollomx.github.io/interiores-aurum/` (HTTP 200, 932 bytes) es solo `index.html`: reenvía a `llave-maestra.html`.
- La casa vieja `https://alexpueblag.github.io/interiores-aurum/llave-maestra.html` da **HTTP 404 (curl 2026-09-04)**: para ESTE repo la puerta vieja no reenvía, corta. Ligas viejas en WhatsApp/correo ya no abren.
- `https://tableros.yodesarrollo.mx/…` **no resuelve** (curl código 000, 2026-09-04): el dominio propio aún no existe en el DNS. No repartas esa dirección todavía.
- Repo público: `github.com/yodesarrollomx/interiores-aurum` (remote verificado con `git remote -v`).

## Reglas INVIOLABLES

1. **El Sheet manda; el HTML solo lee.** Nada de residencias, nombres, portadas, claves ni textos clavados en el código — si el Sheet no responde se usa un salvavidas mínimo, nunca una fuente paralela. (Regla de Alejandro 2026-06-24; memoria `llave-maestra-sheet-driven`.)
2. **Nunca crear implementación NUEVA del Apps Script.** Se edita con el lápiz → "Nueva versión" para conservar la misma URL `/exec`; una implementación nueva cambia la URL y desconecta el tablero (`instrucciones-llave-maestra.md` paso 2).
3. **Cero secretos en el repo.** Es público: lo que se escriba aquí queda expuesto. Las claves por proyecto viven SOLO en la columna `key` de la hoja `Proyectos`; el permiso de edición lo da el rol del Portero, no una contraseña en el HTML (commits `aaf62d4`, `836f664`, `3969b31`).
4. **El acceso es fail-closed.** Sin credencial del Portero el backend no entrega ni un dato: `GET /exec` sin `k` responde `{"ok":false,"error":"liga"}` (verificado por curl 2026-09-04, `Code.gs:82`). Si tocas `doGet`/`doPost`, esa puerta se queda cerrada.
5. **No truncar el texto que escribe la diseñadora.** Títulos, proveedor, atributos y opciones A/B se ven completos; si ella escribe más, debe verse (decisión 2026-06-25; en 1 línea solo quedan precios, pills, tags de acabado y breadcrumb).
6. **Las fotos van a Drive, no al repo.** Subir → compartir "Cualquiera con el enlace · Lector" → pegar el link/ID en la celda del Sheet. Así Alejandro mantiene portadas sin Claude (`img/` quedó descartado para portadas).
7. **Un solo dataset.** Hoy los 13 proyectos muestran los interiores de Navarro Muñoz. Activar un proyecto comercial deja ver interiores privados: por eso `alcinos, cabana, otorrino, pacifica, eyelab` siguen `active:no` a propósito.
8. **Validar antes de entregar** con el harness jsdom (carga el HTML real, stub de fetch). Y **las imágenes de Google se validan con curl, no con navegador headless**: playwright/jsdom las bloquea (memoria `llave-maestra-interiores`).

## Archivos

| Archivo | Qué hace |
|---|---|
| `llave-maestra.html` | El tablero completo: 2,095 líneas, HTML+CSS+JS vanilla en un solo archivo, sin build. CONFIG en la línea 745. |
| `index.html` | Redirección a `llave-maestra.html` (meta refresh + `location.replace`). |
| `datos.json` | **Respaldo empacado** del Sheet (13 proyectos, 12 espacios, 129 productos). Se carga si el backend responde `liga` o falla la red (`llave-maestra.html:1481-1496`, commit `b47a5b5`). |
| `Code.gs` | Backend Apps Script. **NO está rastreado por git** (`.gitignore`) desde el commit `4213307`: es copia de trabajo local. |
| `SECURITY-DEPLOY.md` | Cómo se reabrió el board tras la contención 2026-07-12. **Parcialmente obsoleto** (ver Decisiones). |
| `instrucciones-llave-maestra.md` | Guía de despliegue para Alejandro. **Obsoleta y con claves adentro** (ver Hallazgos). |

## Arquitectura de datos

El Google Sheet es la raíz. Nunca invertir la dirección.

```
Google Sheet "Aurum_LlaveMaestra_Base_NavarroMunoz"
  1gRFwq27ec8nM6g_3LG7xW6ORpLhOyTWkPPUQWgAnsok   (17 hojas)
  Meta · Concepto · Pilares · Paleta · Materiales · Mood · Espacios · Productos ·
  Luminarias · Herrajes · Equipamiento · Acabados · Carpinteria · Renders ·
  Planos · Proyectos · Historial
        │
        │  Apps Script container-bound (doGet arma el JSON; doPost escribe y firma en Historial)
        │  /exec: https://script.google.com/macros/s/AKfycbyEr0MA48NgxPMJ1-i2FFblxQfLV_vocdsiBVk1I_k70FumxebwJSR-yGpGuP6n4yR_/exec
        │
        ├─ GET ?k=<credencial Portero> ──→ llave-maestra.html (fetchData, refresco cada 10 min)
        │      sin k  →  {"ok":false,"error":"liga"}  →  se pinta datos.json (respaldo)
        │
        └─ POST {k, action, tab, row} ──→ upsert/append/setRow/delete/delRow/kv/list
               el servidor canjea k contra el Portero (quienEs_) y exige rol editor

Portero YOD (compartido)  https://yodesarrollomx.github.io/potenciales-yod/portero.js
   · pone la credencial en localStorage `pyod_clave_v1`
   · canje: .../AKfycbwlDDCWWzOWYZsUpBU9uqsQ7aenQ469PF6s6FkNlBFS1_cJSU5njG9oQmuyELy5zlqzFg/exec?recurso=canje
   · roles que EDITAN: admin, editor, proyectos, direccion (Code.gs:53 y llave-maestra.html:1719)

Portadas/renders: Google Drive → celda `cover`/`img` del Sheet → imgUrl() → lh3.googleusercontent.com/d/<ID>=w<N>
   (w800 portada · w400 tarjetas · w1600 lightbox; si lh3 falla, reintenta con thumbnail)
```

**El repo es ESPEJO del backend: lo que corre es lo que está pegado en el editor de Apps Script**, no `Code.gs` del repo (que además ni siquiera se sube). Antes de cambiar el backend, abre el editor y compara; después pega y publica "Nueva versión" con el lápiz.

Almacenamiento local del navegador: `aurum_sel_v2` (selección), `aurum_cache_v1` + `aurum_cache_ts_v1` (copia y su edad), `aurum_unlock_v1` (proyectos desbloqueados), `aurum_route_v1`, `aurum_postq_v1` (cola de escrituras con reintento); `sessionStorage: aurum_designer_v1`.

## Decisiones (fechadas, no revertir sin razón)

- **2026-06-09** — Un archivo HTML vanilla sin build, Sheets como base de datos y Apps Script como API: la diseñadora edita sin tocar código y se despliega subiendo el archivo. (CLAUDE.md previo.)
- **2026-06-09** — Ocultar sin borrar (columna `oculto`): `liveRows()` lo filtra y queda fuera de BUYABLE/selección/totales/PDF, para que todo quede consistente sin parchar cada render.
- **2026-06-09** — Observaciones por categoría (`obs_*` en Espacios), editables inline y visibles también en el PDF.
- **2026-06-24 (Alejandro)** — La portada se administra desde la hoja `Proyectos` (`id,name,sub,active,key,cover,url`). Motivo: él cambia proyectos y fachadas sin pedirlo. Hoy: 13 proyectos = 8 residencias + 5 comerciales.
- **2026-06-25** — Imágenes por `lh3.googleusercontent.com/d/ID=wN` en vez de `drive.google.com/thumbnail`: thumbnail hace un 302 a lh3 en CADA foto (curl: lh3 ~0.5 s vs ~2-4 s). Commit `94bc8c3`.
- **2026-06-25** — Se quitó el truncado del texto editable (ver regla 5). Commit `9e652ab`.
- **2026-07-12** — Contención: se apagaron lecturas/escrituras y se retiró el secreto publicado `aurum-rnm-2026` (comprometido: estaba en el HTML público). `SECURITY-DEPLOY.md`.
- **2026-07-15** — `Code.gs` deja de rastrearse en el repo público (commit `4213307`) y el board dice "reconectando" en vez de colgarse en esqueletos (`aaa5ef9`).
- **2026-07-21** — Reconexión: backend limpio que valida Portero, `READS_ENABLED=true`, scope `script.external_request` autorizado por Alejandro, y `CONFIG.SHEET_URL` apuntando al `/exec` actual (commit `53a04f6`; memoria `llave-maestra-interiores`).
- **2026-07-29** — Auditoría: se borran del repo las 13 claves de proyecto y la `ENTRY_KEY:"Mona"` quemada; además se corrigió que un proyecto **sin** clave abría el tablero con el campo vacío (ahora exige clave no vacía). Commits `aaf62d4`, `836f664`.
- **2026-07-30** — Botón a la carpeta de Drive del proyecto, administrado desde la columna `drive`/`planos`/`carpeta` del Sheet; la URL se valida contra drive/docs.google.com por https para que el Sheet no pueda inyectar `javascript:` (commit `51a73b3`).
- **2026-08-01 (Alejandro)** — ~~Modo diseñadora con clave `Sayri` validada contra `WRITE_SECRET`.~~ **OBSOLETO desde 2026-08-01.** Editar se autoriza por **cuenta y rol del Portero** (admin/editor/proyectos/direccion); se creó el rol `editor` porque los clientes del board también tienen acceso `IN` y no se podía autorizar por tablero. Cada escritura se firma en `Historial` con el nombre que devuelve el canje, no con lo que mande el navegador. Commit `3969b31`; memoria `interiores-editar-por-rol`.
- **2026-08-03** — El PDF separa "Generado el <hoy>" de "Precios y avances al <fecha del dato>", y el sello dice la edad de la copia guardada: dejó de hacer pasar precios viejos por actuales (commit `867473b`).
- **2026-08-13** — Respaldo empacado `datos.json`: con el dominio suspendido el backend contestaba `liga` y el board entraba en bucle/blanco. Ahora `liga` ya no cierra sesión ni recarga; se pinta la foto del Sheet (commit `b47a5b5`).
- **2026-09-01/04** — Mudanza: el tablero vive en `yodesarrollomx.github.io` (commits `ffc8d17`, `0ea0a0f`). Las URLs a `tableros.yodesarrollo.mx` de los commits `dac0815`/`b2049ea` **todavía no responden**: el DNS no existe (verificado 2026-09-04).

## Pendientes

| Tema | Dueño | Qué evidencia lo cierra |
|---|---|---|
| **`datos.json` publica las 13 claves de proyecto y el catálogo completo** (ver Hallazgos) | Alejandro decide; ejecuta Claude | `curl .../datos.json` sin `key` en `_projects` (o el archivo fuera de GitHub Pages) |
| Rotar las 13 claves de la hoja `Proyectos` (quedaron expuestas desde el 29-jul) | Alejandro (edita el Sheet) | Claves nuevas en el Sheet + entrar con una vieja y que rebote |
| Que **Sayri** entre y edite algo de verdad con su cuenta (rol `editor`) | Sayri / Dirección | Un renglón nuevo en la hoja `Historial` firmado con su nombre |
| Confirmar que el `Code.gs` desplegado es el mismo del disco (Versión 6, con `quienEs_`) | Claude, con sesión de Google de Alejandro | Diff editor vs `/Users/a./interiores-aurum/Code.gs` |
| `instrucciones-llave-maestra.md` y `SECURITY-DEPLOY.md` describen el modelo viejo (clave `Sayri`, `WRITE_SECRET`) | Alejandro autoriza; ejecuta Claude | Ambos archivos sin claves y describiendo el modelo por rol |
| Filas del Sheet por corregir: Luminarias sin `id` (l11–l14), Acabados `f09` con columnas corridas, Carpinteria `c04-c06` vacías | Sayri | Las filas con `id` y el lápiz visible en el board |
| Aislamiento real por proyecto (hoy 1 dataset para 13 tarjetas) | Alejandro | Cada proyecto con su propio board vía la columna `url`, o su propio Sheet |
| Opcional: columna `orden` para acomodar tarjetas | Alejandro decide | Su OK; toca estructura |

## Hallazgos al documentar (2026-09-04, no se arreglaron)

1. **`datos.json` reabre el hueco que cerró la auditoría del 29-jul.** Está rastreado y servido: `curl https://yodesarrollomx.github.io/interiores-aurum/datos.json` → **HTTP 200, 183,823 bytes, sin credencial**, y su bloque `_projects` trae **la columna `key` llena en los 13 proyectos**, más el catálogo entero (129 productos, ~$1.22 M en precios, nombre del cliente y del lote). El backend está bien cerrado (`liga`), pero el respaldo del commit `b47a5b5` publica por la puerta de atrás lo mismo que el Portero protege.
2. **La puerta vieja no reenvía este tablero.** El commit `ffc8d17` dice "la puerta vieja reenvia", pero `alexpueblag.github.io/interiores-aurum/llave-maestra.html` da **404** (curl 2026-09-04).
3. **`instrucciones-llave-maestra.md` sigue publicando claves** en el repo público: `Sayri` (líneas 31, 73, 80) y `Mona` (líneas 40, 44, 77), y enseña un modo diseñadora que ya no existe desde el 1-ago.
4. **`SECURITY-DEPLOY.md` manda configurar `WRITE_SECRET`**, que el `Code.gs` actual ya no usa para autorizar (usa `quienEs_` + rol). También dice "Implementación NUEVA", que choca con la regla 2 salvo por esa única vez de 2026-07-12.
5. **Conteo de hojas:** el CLAUDE.md anterior decía "16 hojas"; `doGet` lee **17** (faltaba contar `Historial` o `Proyectos` en el total).
6. **Ningún espacio trae `budget`**: los 12 renglones de `Espacios` tienen `budget` vacío en la copia, así que la barra de "presupuesto global" (clase `ok`/`over`) hoy no tiene contra qué comparar.
7. **El comentario del código quedó viejo:** `llave-maestra.html:856` todavía dice `/* modo diseñadora (clave Sayri) */` aunque ya no hay clave.

## Por confirmar (preguntar, no asumir)

- ¿`datos.json` con las claves adentro fue decisión consciente (que el board no se vea en blanco) o se coló? ¿Se puede publicar el respaldo **sin** la columna `key`?
- ¿Ya se rotaron las 13 claves de la hoja `Proyectos`? En el Sheet están; aquí no se documentan a propósito.
- ¿El Sheet `1gRFwq27…` sigue en **Restringido** (se cerró el 29-jul-2026)?
- ¿Cuándo entra `tableros.yodesarrollo.mx` al DNS, para cambiar las ligas de un jalón?
- ¿El 404 de `alexpueblag.github.io/interiores-aurum/` se deja así o le ponemos reenvío como a los demás tableros?
- ¿El harness jsdom de las 46 pruebas sigue existiendo y dónde vive? No está en este repo.

## Cómo se trabaja aquí

- Español de México, conciso, sin emojis. Alejandro no lee texto largo: estado primero, decisiones en opción múltiple.
- Se entregan archivos completos listos para subir (no parches), siempre con los pasos de despliegue.
- **Front:** editar `llave-maestra.html` → subir a la RAÍZ del repo → recargar con Ctrl+Shift+R (~1 min de GitHub Pages).
- **Back:** pegar `Code.gs` en el editor de Apps Script → Implementar ▸ Administrar implementaciones ▸ lápiz ▸ **Nueva versión** (conserva `/exec`).
