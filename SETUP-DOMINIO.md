# Setup del dominio creamixds.com y los sistemas

Documentación de cómo quedó armada la infraestructura (dominio + los 3 sistemas en GitHub Pages).
Última actualización: 17/11/2026.

---

## Arquitectura en una línea

Un **solo dominio** (`creamixds.com`) sirve la landing de CreamixDS en la raíz y cada sistema como subcarpeta,
gratis, gracias a GitHub Pages. No hace falta un dominio por sistema.

La clave: el dominio se ancla en el repo **`creamix-ds.github.io`** (el "user site"). Al tenerlo ahí,
GitHub sirve automáticamente todos los demás repos del usuario como `creamixds.com/<nombre-del-repo>`.

---

## URLs de los sistemas (¡OJO con las mayúsculas!)

Las rutas de GitHub Pages respetan **exactamente** el nombre del repo (distinguen mayúsculas):

| URL | Repo | Qué es |
|---|---|---|
| `creamixds.com` | `creamix-ds.github.io` | Landing de CreamixDS (portada) |
| `creamixds.com/pitazo-final` | `pitazo-final` | Pitazo Final (minúscula) |
| `creamixds.com/Gebro` | `Gebro` | Gebro (**G** mayúscula) |
| `creamixds.com/Pitazoclubs` | `Pitazoclubs` | Pitazo Clubs (**P** mayúscula) |

> Al poner los botones/enlaces en la portada, usar estas mayúsculas exactas o la gente cae en 404.

---

## DNS (en DonWeb)

| Tipo | Nombre | Contenido |
|---|---|---|
| A | creamixds.com | 185.199.108.153 |
| A | creamixds.com | 185.199.109.153 |
| A | creamixds.com | 185.199.110.153 |
| A | creamixds.com | 185.199.111.153 |
| CNAME | www.creamixds.com | creamix-ds.github.io |

- Las 4 IPs son las oficiales de GitHub Pages y van **todas en el apex** (`creamixds.com`).
- **Correo @creamixds.com:** se borraron los registros A de `mail` y `mx1` (apuntaban mal a GitHub).
  Quedaron huérfanos los MX/SPF/DKIM pero no afectan a la web. Si algún día se quiere correo, usar
  Email Routing de Cloudflare o restaurar la IP real del servidor de correo.

---

## GitHub Pages

- **Repo ancla:** `creamix-ds.github.io` → Settings → Pages → Source: *Deploy from a branch*, `main` / `(root)`.
  - Custom domain: `creamixds.com` (crea un archivo `CNAME` en el repo). **Enforce HTTPS** activado.
- El dominio va **SOLO** en el repo ancla. **NO** ponerlo en pitazo-final / Gebro / Pitazoclubs:
  esos cuelgan automáticamente como subcarpetas.
- ⚠️ **No borrar el archivo `CNAME`** de `creamix-ds.github.io` al subir desde el clon local, o se cae el dominio.

---

## OAuth de Google (login de cada app)

Cada sistema tiene su propio proyecto y cliente en Google Cloud. Como ahora cargan desde `creamixds.com`,
hay que agregar ese origen en **APIs y servicios → Credenciales → (cliente) → Orígenes autorizados de JavaScript**:
agregar `https://creamixds.com` (solo el dominio, sin ruta ni barra final). No borrar el de `creamix-ds.github.io`.

| App | Proyecto | Cliente (empieza con) | Origen cargado |
|---|---|---|---|
| Gebro | `gebro-512501` | `340172069418-grc7uvkq...` | ✅ |
| Pitazoclubs | `pitazoclubs` | `513742201235-79a1b7t...` | ✅ |
| Pitazo Final | (ver config.js) | `1052940348586-oe5937r...` | ⬜ pendiente |
| Landing CreamixDS | (ver config) | `411887691042-vdh480ti...` | ⬜ pendiente |

Error típico si falta: `Error 400: origin_mismatch` (y la app puede quedar en pantalla negra si pide login al cargar).

---

## Cache-busting `?v=` (lección importante)

Los `index.html` cargan los scripts con `?v=N` (ej: `js/config.js?v=16`). Ese número es lo que obliga al
navegador a bajar la versión nueva.

**Regla de oro:** cada vez que cambiás `config.js`, `app.js` o `estilos.css`, **subí el número `?v=`** en el
`index.html`. Si no, los navegadores siguen usando la versión vieja cacheada aunque el archivo ya esté actualizado.

> Esto fue lo que rompió la portada de Gebro: el `config.js` ya tenía el dato nuevo, pero había quedado pegado
> a `?v=14`, así que los navegadores servían el `config.js` viejo. Se arregló subiendo a `?v=16`.

---

## Pendientes

- [✅] OAuth: agregar `https://creamixds.com` en Pitazo Final (`1052940348586-...`) y en la landing CreamixDS (`411887691042-...`).
- [✅] Gebro: el `og:url` / `og:image` del `index.html` todavía apuntan a `creamix-ds.github.io/Gebro/` (detalle de SEO, no afecta funcionamiento).
