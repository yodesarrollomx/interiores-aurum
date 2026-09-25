# Llave Maestra — cómo se opera (al 25-sep-2026)

## Entrar y editar
1. Abre https://yodesarrollomx.github.io/interiores-aurum/llave-maestra.html y entra con Google.
2. Si tu cuenta tiene rol de edición (Dirección, Proyectos/Sayri), aparece **Diseñadora**:
   lápiz en cada tarjeta y **Agregar** por sección. Guardar escribe directo al Sheet y queda
   firmado en `Historial`. Los clientes solo ven.

## Portadas y residencias (hoja `Proyectos`)
Columnas: `id | name | sub | active | key | cover | url`.
- `active`: `sí` la abre; `no` la deja como *Próximamente*.
- `key`: la clave que se le da al cliente (filtro suave; ver SECURITY-DEPLOY.md).
- `cover`: link de Drive de la portada (compartido «Cualquiera con el enlace · Lector»).
- `url`: si la residencia tiene su propio board, su liga.

## Imágenes
Van a Drive, no al repo: subir → compartir «Cualquiera con el enlace · Lector» → pegar el link
en la celda. Si una imagen no se ve, casi siempre es el permiso.

## Publicar cambios del Apps Script
Extensiones ▸ Apps Script → pegar → guardar → **Implementar ▸ Administrar implementaciones ▸
lápiz ▸ Nueva versión ▸ Implementar**. Nunca «Nueva implementación» (cambia la URL `/exec`).
