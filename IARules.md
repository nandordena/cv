# IARules — reglas de modificación

Leer `IA.md` antes de tocar código. Este archivo manda sobre estilo y alcance de los cambios. `IA.md` describe cómo funciona el proyecto hoy.

## Alcance

- Stack fijo: HTML + CSS + JS vanilla. No añadir bundler, framework, backend, npm ni tests salvo petición explícita.
- Archivos de producto: `index.html`, `styles.css`, `script.js`. Documentación de IAs: `IA.md`, `IARules.md`. No reescribir `readme.md` humano salvo que se pida.
- Cambios mínimos. No refactorizar “de paso”, no modernizar ES modules, TypeScript ni `let`/`const` globales salvo que el usuario lo pida.
- No inventar empleos, fechas, empresas, URLs ni skills. Si falta un dato, preguntar o dejar el campo vacío / omitirlo según el contrato de `cv`.
- Respetar simepre la metodologia del contexto de la modificacion

## Dónde editar cada cosa

| Intención | Archivo | Qué tocar |
|-----------|---------|-----------|
| Nuevo hito / editar CV | `script.js` | Array `cv` |
| Barras de habilidades | `script.js` | **Segunda** `calculateSkills` (objeto literal). Editar `cv` **no** actualiza esa vista |
| Filtro de categoría | `index.html` + `cv` | `<option value="...">` y el mismo token en `category` (CSV) |
| Layout / timeline / colores | `styles.css` | Preferir variables `:root` |
| Cabecera, menú, secciones | `index.html` | IDs `#history` y `#skills` no se renombran sin actualizar JS y CSS |
| Comportamiento de vistas | `script.js` + CSS | `data-section-status` en `body` + clases `.display-history` / `.display-skills` |

Tras cambiar `cv` o skills, no hace falta build: recargar `index.html`.

## Contrato del array `cv`

Campos mínimos: `position`, `date` (`YYYY-MM-DD`).

Recomendados: `type`, `category`, `description`, `skills`, `newSkills`, `image`, `imageStyle`, `url`, `dateEnd`.

- `dateEnd` omitido → “Actualidad” y duración hasta ahora.
- `dateEnd: 'no'` → evento puntual: sin años ni flecha de fin; `getWorkTime` = 0.
- `skills` / `newSkills` / `category`: CSV **sin espacios raros**; tokens de categoría solo los del `<select>`: `it`, `dev`, `manager`, `pm`, `av`, `art`.
- `type` solo valores con icono: `employment`, `promotion`, `course`, `event`, `voluntary` (evitar `study` salvo que se use de verdad).
- Mismo empleador en el tiempo: entradas distintas; ascenso → `type: 'promotion'`.
- No borrar entradas: comentar el objeto como el resto del archivo.
- Mantener el estilo del array (comas iniciales, comentarios `}//nombre`).
- Orden en el array da igual: `printHistory` ordena por `date` descendente.

Al añadir un hito con skills nuevas, **actualizar también** el objeto hardcodeado de habilidades si deben verse en la pestaña Habilidades (`time` en ms, `type` de grupo existente: Desarrollo, Gestion de equipos, Informática, Idiomas, Software, Otros, Música). No inventar milisegundos: preguntar o calcular solo si el usuario da fechas claras.

## Qué no “arreglar” sin que lo pidan

Tratar como deuda conocida, no como bugs a cazar:

- Dos `calculateSkills`; la primera está muerta.
- `chageSubSection` no existe; botones “Por Tipo” / “Por Experiencia (Pronto)”.
- Filtro `filter.skill` roto.
- Tags `<spam>`, `body` duplicado, `border:in`, typos CSS (`heigth`, `rigth`).
- Nombres de skills inconsistentes (Phyton, Scroom, Autocat, JScript).
- Extensiones en `String.prototype` y variables globales sin `let`.
- Share copia la URL de CodePen, no la del repo.

Sí se pueden tocar si el usuario pide esa feature o el cambio es imprescindible para lo que pidió.

No reactivar la primera `calculateSkills` ni borrar la segunda sin instrucción explícita: las barras actuales dependen del objeto literal.

## Estilo de código

- Seguir el estilo existente: funciones globales, `chageSection` (typo del nombre: no renombrar), `onclick` en HTML, Font Awesome 4.
- Chips: seguir generando `<spam>`, no cambiar a `<span>` salvo unificación pedida (rompe CSS `.h-skill>spam`).
- No introducir módulos, `defer` extraño, ni mover datos a JSON externo salvo petición.
- CSS: no romper el eje de `#history` ni los márgenes impar/par de `.h-box`. Probar mentalmente `max-width: 800px`.
- Nuevas vistas: nuevo valor de `data-section-status`, clase `.display-*`, regla de ocultar en el bloque Dinamics-Display, y botón con `data-section` coherente.

## Documentación

Si cambia el contrato (campos de `cv`, secciones, skills derivadas vs hardcodeadas, filtros), actualizar `IA.md` en el mismo cambio. `IARules.md` solo si cambian las reglas de edición.

## Verificación

Abrir `index.html`, Historial y Habilidades, un valor del filtro ≠ `all`, y el icono share. Comprobar que entradas sin `category` desaparecen al filtrar (comportamiento actual).