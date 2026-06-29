# TACNA · Centro de Comando — Reforma de Jornada 2027–2030

Tablero interactivo de un solo archivo (`index.html`) para gestionar la transición de la reducción
de jornada (LFT, 48→46→44→42→40 h, 2027–2030) en todo el portafolio de clientes shelter de TACNA:

- **Portafolio (inicio):** avance global, avance por cluster, tabla filtrable con brecha por año,
  jornada '27 registrada, costo full-burden estimado y estatus por cliente.
- **Detalle de cliente:** captura de salario/factor, turnos con clasificación auto (Art. 60),
  **Diseñador de jornada** (actual vs. propuestas viables, selección registrada por año) y
  resumen de impacto financiero (tiempo extra vs. plantilla vs. mezcla).
- **Rezagados:** orden por riesgo para cerrar al 100% antes del 1-ene-2027.
- **Supuestos:** topes estatutarios y fórmula de carga social como referencia.

---

## Archivos y dónde va cada uno (no todos van a GitHub)

| Archivo | Destino | Para qué |
|---|---|---|
| `index.html` | **GitHub** (raíz del repo) | Es el tablero. Lo sirve GitHub Pages. **Obligatorio.** |
| `README.md` | **GitHub** (raíz del repo) | Esta documentación. Recomendado. |
| `Code.gs` | **Google Apps Script** (NO a GitHub) | Se pega en Extensiones → Apps Script de tu hoja "Store". Es el backend de persistencia. |

> Resumen: a **GitHub** subes solo `index.html` y `README.md`. El `Code.gs` se **pega dentro de Apps Script**, no se sube al repo.

## Crear el repo y publicar (GitHub Pages)

1. Ve a **https://github.com/new** e inicia sesión con tu cuenta.
2. Nombre sugerido: `tacna-jornada`. Marca **Private**. Crea el repo.
3. **Add file → Upload files** → sube `index.html` y `README.md` a la raíz. Commit.
4. **Settings → Pages → Source: Deploy from a branch** → rama `main`, carpeta `/ (root)`. Save.
5. En ~1 min tu liga viva será: `https://<tu-usuario-de-github>.github.io/tacna-jornada/`
   (sustituye `<tu-usuario-de-github>` por tu usuario real; ese es el enlace que compartes al equipo).

---

## Despliegue rápido en GitHub Pages (~5 min)

1. Crea un repositorio nuevo en GitHub, p. ej. `tacna-jornada` (puede ser **privado**).
2. Sube **`index.html`** a la raíz del repo (botón *Add file → Upload files*).
3. Ve a **Settings → Pages**.
4. En *Build and deployment → Source* elige **Deploy from a branch**; rama `main`, carpeta `/ (root)`. Guarda.
5. Espera ~1 min. Tu liga viva queda en:
   `https://<tu-usuario>.github.io/tacna-jornada/`

> Para acceso restringido al equipo, mantén el repo privado y usa GitHub Pages privado (planes Team/Enterprise),
> o publica detrás de un acceso (Cloudflare Access, Netlify con contraseña, etc.).

---

## Persistencia compartida con Google Sheets (recomendado)

El tablero detecta su entorno automáticamente y elige dónde guardar, en este orden:

| Prioridad | Entorno | Almacenamiento | Alcance |
|---|---|---|---|
| 1 | `GS_API` configurado | Google Sheets (Apps Script) | **Compartido** entre todo el equipo (una sola fuente) |
| 2 | Artefacto en Claude.ai | `window.storage` | Compartido dentro del artefacto |
| 3 | GitHub Pages sin `GS_API` | `localStorage` | Solo ese navegador |

### Cómo conectar Sheets (una vez, ~10 min)

1. Crea una hoja de cálculo de Google (o usa una nueva dedicada al tablero).
2. **Extensiones → Apps Script**. Borra lo que haya y pega el contenido de **`Code.gs`**.
3. En `Code.gs`, cambia `TOKEN` por un secreto tuyo (p. ej. `tacna-2027-xY9`).
4. **Implementar → Nueva implementación → Aplicación web**:
   - *Ejecutar como:* **Yo**
   - *Quién tiene acceso:* **Cualquier usuario**
   - Autoriza los permisos cuando lo pida. Copia la **URL `…/exec`**.
5. Abre **`index.html`** y edita las dos líneas de arriba del bloque storage:
   ```js
   const GS_API   = 'https://script.google.com/macros/s/AKfy.../exec'; // tu URL
   const GS_TOKEN = 'tacna-2027-xY9';                                  // mismo TOKEN que Code.gs
   ```
6. Sube `index.html` a GitHub. Listo: cada captura se escribe en la pestaña `dashboard_store`
   y todo el equipo lee/escribe la misma fuente. Dirección puede ver la data cruda en esa pestaña.

> El tablero crea la pestaña `dashboard_store` (key | value | updated) sola en el primer guardado.
> Una fila por cliente (`c:<id>`). Escrituras protegidas con `LockService` (last-write-wins por celda).

### Seguridad — léelo

- El `TOKEN` evita accesos casuales, **no es autenticación fuerte**: cualquiera con la URL y el token
  puede leer/escribir. Para datos de salarios, mantén el **repo/Pages privado** o detrás de un acceso
  (Cloudflare Access, Netlify con contraseña). Cambia el `TOKEN` si se filtra.
- Para control de acceso real por usuario (login Google, permisos por cluster), el paso siguiente es
  un backend con autenticación (Supabase/Firebase con reglas, o un proxy). Pídelo cuando lo necesites.

### Cambiar de backend más adelante

La capa de datos está aislada en el objeto `STORE` (`list / get / set / delete`). Migrar a Supabase o
Firebase toca **solo esa sección** de `index.html`; el resto del tablero no cambia.

---

## Datos semilla

`index.html` viene precargado con el portafolio (78 plantas, ~9,373 HC, 8 clusters), población
sindicalizada, código BD/Tress y las jornadas documentadas con turno auto-clasificado. La captura
del equipo (salario, HC por turno, jornada seleccionada, checklist) se guarda encima de la semilla.

## Aviso legal

La clasificación de turno (Diurna/Mixta/Nocturna), el reparto Art. 59 y el modelo de costo son
herramientas de planeación. **Valida cada convenio y jornada con tu abogado laboral.** Los topes
provienen del cuadro Index Mexicali (reforma LFT, may-2026) y pueden cambiar por enmiendas o criterios.
