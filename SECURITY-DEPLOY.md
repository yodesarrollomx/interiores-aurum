# Acceso y seguridad — Llave Maestra (al 25-sep-2026)

## Quién ve y quién edita
- **Ver el board:** sesión del Portero YOD con Google (la dirección del Portero vive en
  `yod-portal/os/yod-acceso.js`). Sin sesión, el backend responde `{ ok:false, error:'liga' }`
  y el board pinta el respaldo `datos.json` (sin claves de proyecto desde el 24-sep).
- **Editar:** por **rol del Portero**, no por contraseña. Editan `admin`, `editor`, `proyectos`
  y `direccion`; el servidor vuelve a canjear la credencial y valida el rol en cada escritura, y
  firma el cambio en la hoja `Historial` con el nombre del canje. El botón «Diseñadora» del
  HTML es solo la puerta visual.
- **Clave por residencia** (columna `key` de la hoja `Proyectos`): es un filtro suave para los
  clientes, no protege datos. Decisión 24-sep: el board es promocional y no se rotan.

## Ya no existe
- La clave `Sayri` y el secreto `WRITE_SECRET`: se retiraron el 1-ago (edición por rol).
- El secreto `aurum-rnm-2026` (comprometido en julio): ya no se usa en ninguna parte.

## Si hay que volver a publicar el Apps Script
Editar la implementación EXISTENTE → lápiz → **Nueva versión**. Nunca «Nueva implementación»:
cambia la URL `/exec` y desconecta el board.
