# CV — portfolio estático de Fernando Moya Llasat

Sitio web de currículum / línea de tiempo. **Sin backend, sin bundler, sin framework.** Tres archivos: `index.html`, `styles.css`, `script.js`. Los datos viven en un array JS en el cliente. Se abre con `index.html` o en [CodePen](https://codepen.io/nandordena/full/VwGMwVY).

Idioma de la UI: español (`lang="es"`). Autor: Fernando Moya Llasat. Contacto: `nandordena@gmail.com`. Enlaces: LinkedIn, CodePen, GitHub.

---

## Propósito para agentes / IAs

1. Entender el CV como **fuente de datos en `script.js` (`cv`)**, no como HTML estático.
2. Saber qué se **renderiza** (historial vs habilidades) y qué es **CSS de estado**.
3. Distinguir **intención** (filtrar, agrupar skills por tipo, copiar URL) de **código incompleto o muerto**.
4. Al editar: tocar `cv` o el objeto `skills` hardcodeado; no hay API ni persistencia.

---

## Arquitectura

index.html → markup, menú, contenedores vacíos #history y #skills styles.css → timeline, visibilidad por data-section-status, responsive script.js → datos cv[], render, filtros, helpers de fechas, clipboard


Dependencias externas:

- Font Awesome 4.7 (CDN Cloudflare)
- Imágenes remotas (ibb.co, logos de empresas, YouTube, etc.)

No hay `package.json`, tests ni CI.

Arranque (`window` `load`):

1. `historyEle = #history`
2. `printHistory()` (sin filtro)
3. `printSkills()`
4. `change` en `#filter select` → `changeFilter`
5. `applyFilterFromGet()` si hay query `filter` válido

---

## UI y estado (HTML + CSS)

`body` nace con `data-section-status="h"`.

| Valor | Vista | Qué se muestra |
|-------|--------|----------------|
| `h` | Historial | `#history` (clase `.display-history`) |
| `s` | Habilidades | `#skills` y botones `.display-skills` |

CSS (regla “Dinamics-Display”):

- Si `data-section-status` **no** es `h` → oculta `.display-history`
- Si **no** es `s` → oculta `.display-skills`

Cambio de sección: `chageSection(this)` lee `data-section` del botón y lo pone en `body`.

Botones:

- Historial → `data-section="h"`
- Habilidades → `data-section="s"`
- “Por Tipo” → `chageSubSection(this)` `data-section="s"` — **la función no existe en `script.js`**
- “Por Experiencia (Pronto)” → `chageSubSection` `data-section="s-x"` — **igual, no implementado**; `s-x` no tiene reglas CSS de visibilidad

Filtro de categorías (`#filter select`): solo afecta al **historial**. Valores: `all`, `it`, `dev`, `manager`, `pm`, `av`, `art`. El filtro va en una fila debajo de Historial/Habilidades (`#menu` en columna, `.menu-row`). El `<select>` está **siempre visible**.

GET `?filter=`: en `load`, `applyFilterFromGet()` lee `URLSearchParams` `filter`. Si coincide **exactamente** con el `value` de un `<option>` del select, asigna el select y llama `changeFilter`. Si falta o no coincide, no filtra. No hay lista duplicada en JS: la fuente de valores válidos es el propio `<select>`. Ejemplo: `index.html?filter=dev`.

Cabecera: foto, nombre, iconos. El botón share (`fa-share-alt`) llama `clickToCopy(this)` con `text-copy="https://codepen.io/nandordena/full/VwGMwVY"`. Clase `.copyed` ~1s → tooltip “Copiado en el portapapeles”.

`#contact` está vacío.

HTML: hay un `</body>` de más y un segundo `<body>` antes del script; el navegador suele tolerarlo.

---

## Modelo de datos: array `cv`

Cada entrada es un objeto. Campos usados en render:

| Campo | Uso |
|-------|-----|
| `position` | Título del puesto / curso / evento |
| `date` | Inicio `YYYY-MM-DD`. Ordenación (más reciente primero) |
| `dateEnd` | Fin, omitido = “Actualidad”, `'no'` = evento puntual (sin duración ni flecha de fin) |
| `description` | Texto; saltos de línea → `<br>` vía `String.prototype.newRowRender` |
| `skills` | CSV de skills usadas; chips grises |
| `newSkills` | CSV de skills **nuevas** en ese tramo; chips cian + icono `fa-plus` |
| `type` | Icono del nodo. Default CSS: `employment` si falta |
| `image` | Logo; fallback `imageDefault` |
| `imageStyle` | CSS inline (scale, filter) |
| `url` | Enlace de la imagen (`target=_blank`); default `#` |
| `category` | CSV para el filtro: `it`, `dev`, `manager`, `pm`, `av`, `art` |

`type` conocidos y iconos Font Awesome (`::before` en `.h-box`):

| `type` | Icono | Significado |
|--------|--------|-------------|
| `employment` | maletín `\f0b1` | Trabajo |
| `promotion` | flecha `\f062` | Ascenso (misma empresa, otro cargo) |
| `course` | birrete `\f19d` | Formación reglada |
| `event` | estrella `\f005` | Evento puntual |
| `voluntary` | manos `\f256` | Voluntariado |
| `study` | libro `\f02d` | Definido en CSS, **casi no usado** en datos (estudios van como `course`) |

Bloques comentados en el array: no se renderizan.

Tipos de entrada en la práctica: empleos, promociones internas (Oxinity, Beraca), ciclos formativos, voluntariado (música, radio, audiovisuales), eventos (conciertos, exposición, masterclass).

---

## Render del historial: `printHistory(filter)`

1. Vacía `#history`.
2. Filtra `cv`:
   - Sin `filter` → todo.
   - `filter.type` → `entry.type.includes(...)` (el `type` actual es un string, no lista).
   - `filter.category` → exige `category` y `category.includes(valor)` (substring en CSV).
   - `filter.skill` → **bucle `forEach` que no altera el filtro externo**; no funciona como AND de skills.
3. Orden: `date` descendente.
4. Duración: `dateEnd` − `date` (o `now` si no hay fin). Años vía `toLocaleDateString` year − 1970. `0` años → vacío; `1` → “N Año”; resto → “N Años”. Si `dateEnd=='no'` → sin años.
5. HTML: `.h-box` con `data-type`, fechas capitalizadas (`toDate(dateFormat)` = año + mes largo), logo, cargo, descripción, chips.

El filtro de la UI solo pasa `{ category: valor }` o nada si `all`.

---

## Habilidades: `calculateSkills` + `printSkills`

Hay **dos** `function calculateSkills`. La segunda **sobrescribe** la primera.

- Primera (muerta): recorre `cv`, parte `newSkills`, acumula `getWorkTime(work)` en `skills[nombre].time`. `getWorkTime`: `dateEnd=='no'` → 0; si no, ms entre inicio y fin (o ahora).
- Segunda (la que corre): objeto **literal** skill → `{ time, type }`. `time` en **milisegundos**. `type` = categoría de UI: Desarrollo, Gestion de equipos, Informática, Idiomas, Software, Otros, Música.

`printSkills`:

1. Llama `calculateSkills()`.
2. `Object.entries(skills)`.
3. Categorías únicas (orden de primera aparición; **no** ordena por `time` — el `.sort` está comentado).
4. `maxTime` = máximo `time`.
5. Por categoría: `<h2>` + barras. Ancho `%` = `round(time/maxTime*100)+1`. `data-time` = `"Xy Ym"` (`getYearsOfTime` / `getMontsRestOfTime`). El CSS de `content: attr(data-time)` está **comentado**; el tooltip de duración no se pinta.

Conclusión: las barras **no** se derivan del array `cv` en runtime. Añadir un trabajo **no** actualiza skills salvo que se edite el objeto hardcodeado. Los nombres de skills en `cv` y en el objeto no están normalizados (typos: Phyton, Scroom, Autocat, JScript vs JavaScript, etc.).

---

## Helpers globales (script no modular)

- `dateFormat = { year: 'numeric', month: 'long' }`
- `imageDefault` — placeholder ibb
- `String.prototype.newRowRender` / `toDate`
- `copyToClipboard` / `clickToCopy` (`document.execCommand('copy')`)
- `getYearsOfTime(t)` / `getMontsRestOfTime(t)` — `t` es duration en ms interpretada como epoch Date (truco year−1970)

Variables globales sin `let`/`const`: `cv`, `skills`, `historyEle`, `ele`, `text`.

---

## CSS relevante (timeline)

- Tokens: `--bgColor`, `--boxColor`, `--color1/2/3` (acento `#43c8d7`).
- `#history::before`: eje vertical cian al 50%.
- `.h-box` impar a la izquierda, par a la derecha; overlap `-5em`.
- `::after`: chevron hacia el eje; `::before`: círculo + icono por `data-type`.
- `@media (max-width: 800px)`: una columna, eje a la derecha.
- Botón activo: `[data-section-status="h"] [data-section="h"]` (y análogo `s`) fondo cian. La regla de `border` está **cortada** (`border:in`).

Typos CSS: `heigth`, `rigth` en `[data-time]:after`.

---

## Cómo extender (contrato implícito)

**Nuevo hito en el timeline:** objeto en `cv` con al menos `position`, `date`. Recomendado: `type`, `category` (CSV de valores del `<select>`), `skills`/`newSkills` CSV, `image`+`url`.

**Nueva categoría de filtro:** `<option>` en HTML **y** el mismo token en `category` de las entradas.

**Nueva skill en la vista Habilidades:** entrada en el objeto de la **segunda** `calculateSkills`, con `time` (ms) y `type` (string de grupo). O reactivar la primera función y borrar la segunda (hoy no está cableado).

**Nueva vista:** nuevo `data-section-status`, clases `.display-*` y CSS de show/hide; implementar `chageSubSection` si se quiere “Por Tipo / Por Experiencia”.

---

## Limitaciones y trampas para IAs

- No es un CV en Markdown: el contenido canónico es JS.
- Skills UI ≠ skills del historial.
- Filtro `skill` y `chageSubSection` incompletos.
- `category.includes('it')` es substring: un valor futuro que contenga `it` colisionaría.
- Duraciones de eventos `dateEnd:'no'` no suman tiempo.
- HTML inválido menor (`<body>` duplicado); chips usan tag no estándar `<spam>`.
- `index.html` title: “Mi Página”, no el nombre.

---

## Cómo ejecutarlo

Abrir `index.html` en el navegador. Sin `file://` puede haber avisos de clipboard; el resto es estático.