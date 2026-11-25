# Curiosipedia v1 (El espacio): plan de implementación

> **Para agentes:** SUB-SKILL REQUERIDA: usar superpowers:subagent-driven-development
> (recomendado) o superpowers:executing-plans para ejecutar este plan tarea por
> tarea. Los pasos usan casillas (`- [ ]`) para llevar el avance.

**Objetivo:** construir una app web instalable en Android, que funcione sin
internet. Trae una portada de "nave" con cuatro áreas y la sección El espacio
completa: 14 temas con dibujos, quiz, estrellas y constelaciones.

**Arquitectura:**
- HTML, CSS y JavaScript con módulos ES, sin frameworks ni compilación.
- Todo lo que se publica vive en `app/`. Las pruebas y los scripts viven fuera.
- La navegación usa rutas con `#`, así el botón "atrás" de Android funciona.
- Un service worker guarda la lista completa de archivos para que funcione sin
  internet.
- El contenido son módulos de datos, uno por tema, separados de las vistas.

**Tecnologías:**
- Node 24 con `node:test` para las pruebas de lógica y contenido.
- Playwright, solo en desarrollo, para las pruebas en navegador.
- GitHub Actions para publicar en GitHub Pages.

**Diseño que implementa:** `docs/superpowers/specs/2026-11-25-enciclopedia-espacio-design.md`
(leerlo completo, sobre todo los puntos 5 y 5b).

## Restricciones globales

- Toda la app (`app/`) pesa **menos de 5 MB**.
- No tiene **dependencias en tiempo de ejecución** ni pide nada a otros dominios.
  No lleva enlaces externos, anuncios, analítica ni chat.
- Todas las rutas son **relativas**, porque la app se publica en
  `https://sonyrp.github.io/curiosipedia/`.
- Todo el texto está en **español**. Los botones miden al menos **48×48 px**. Se
  diseña para pantallas verticales de **360 a 430 px** de ancho.
- No guarda **ningún dato personal**. El avance se guarda solo en `localStorage`,
  con la clave `curiosipedia.v1`.
- Si el celular tiene activado `prefers-reduced-motion`, las animaciones se
  vuelven mínimas. Ninguna animación dura más de **1 s**.
- El texto siempre se lee sobre un fondo sólido.
- Colores base:
  - Fondo índigo `#0B0F3D`.
  - Amarillo `#FFD93D`.
  - Coral `#FF6B6B`.
  - Turquesa `#4ECDC4`.
- Letra: Nunito (licencia OFL), guardada en `app/fonts/`. Solo los caracteres
  latinos que usa el español, en **menos de 150 KB**.
- Nada se parece a Encarta: ni el nombre, ni los dibujos, ni los textos.
- Commits en español, con prefijo (`feat:`, `test:`, ...), y una rama por tarea.
- No se hace `push` a `main` sin permiso del usuario.

## Puntos a vigilar en la revisión

1. **`localStorage` dañado o no disponible.** Puede tener JSON roto o lanzar un
   error. La app abre con el avance vacío y no se rompe.
   → Lo prueba la Tarea 2.
2. **Llega una versión nueva mientras la app está abierta o sin internet.** La
   versión vieja sigue funcionando. Al volver a abrirla aparece "Hay novedades"
   y se borran los cachés viejos.
   → Lo prueba la Tarea 10.
3. **El botón "atrás" de Android y recargar la página en `#/espacio/marte` sin
   internet.** "Atrás" vuelve a la pantalla anterior, sin salir de la app. Al
   recargar, abre el mismo tema.
   → Lo prueban las Tareas 1 y 10.
4. **Tocar dos veces una respuesta o salir del quiz a la mitad.** No da
   estrellas de más. Un quiz a medias no cuenta.
   → Lo prueba la Tarea 7.
5. **Una dirección que no existe**, como `#/espacio/plutonn`. Muestra una pantalla
   amable de "No encontré ese lugar", con un botón para volver al mapa. Nunca
   queda en blanco.
   → Lo prueba la Tarea 1.

---

## Estructura de archivos

```
app/                          ← lo único que se publica
  index.html                  esqueleto: <main id="pantalla">, carga js/main.js
  manifest.webmanifest
  sw.js                       service worker (importa precache.js)
  precache.js                 GENERADO: const VERSION, ARCHIVOS = [...]
  css/base.css                colores, letra, botones, reducción de movimiento
  css/pantallas.css           estilos de cada pantalla
  fonts/nunito-latin.woff2
  icons/icon-192.png, icon-512.png, icon-maskable-512.png, icon.svg
  art/chispa.svg              copiloto (con estados por clase CSS)
  art/espacio/<id>.svg        un dibujo por tema, estilo papel recortado
  js/main.js                  arranque: registra el SW, el router y "Hay novedades"
  js/router.js                rutas con #, lógica pura
  js/progreso.js              avance, lógica pura y almacenamiento
  js/quiz.js                  puntaje y Gran quiz, lógica pura
  js/constelaciones.js        datos de las constelaciones y puntos encendidos
  js/sonido.js                bips con WebAudio y vibración, se pueden silenciar
  js/vistas/portada.js, enciclopedia.js, mapa.js, tema.js, escala.js,
            quiz.js, gran-quiz.js, no-encontrado.js
  content/espacio/indice.js   ORDEN de ids y metadatos de la sección
  content/espacio/<id>.js     export default { ...Tema }
content-fuentes/FUENTES.md    fuentes por tema (no se publica)
scripts/servir.js             servidor estático sin dependencias, sirve app/
scripts/generar-precache.js   recorre app/ y escribe app/precache.js
scripts/vista-previa.js       arma revision/vista-previa.html para el adulto
tests/*.test.js               node:test
e2e/*.spec.js                 Playwright
.github/workflows/pages.yml   publica app/ en GitHub Pages
```

**Los 14 temas, por `id` y en orden:** `sol`, `mercurio`, `venus`, `tierra`,
`luna`, `marte`, `jupiter`, `saturno`, `urano`, `neptuno`, `planetas-enanos`,
`asteroides-cometas`, `estrellas`, `astronautas`.

**Forma de un tema (`Tema`):**
```js
{
  id: 'marte', titulo: 'Marte', color: '#E2563B', emoji: '🔴',
  tipo: 'planeta' | 'estrella' | 'luna' | 'otro',
  texto: ['párrafo', ...],            // entre 120 y 200 palabras en total
  ficha: [{ etiqueta: 'Distancia al Sol', valor: '228 millones de km' }], // de 3 a 4
  curiosidades: ['...', '...'],       // de 2 a 3
  diametroKm: 6779 | null,            // para "¿Qué tan grande?" (null en temas sin escala)
  quiz: [{ pregunta, opciones: [a, b, c], correcta: 0|1|2, explicacion }] // 3 preguntas
}
```

---

### Tarea 1: Andamiaje, router y pantalla "no encontrado"

**Archivos:**
- Crear: `package.json` (`"type": "module"`, con los scripts `test`, `e2e` y
  `servir`), `scripts/servir.js`, `app/index.html`, `app/css/base.css`,
  `app/js/main.js`, `app/js/router.js` y `app/js/vistas/no-encontrado.js`.
- Prueba: `tests/router.test.js` y `e2e/navegacion.spec.js`.

**Interfaces:**
- Produce: `resolverRuta(hash: string) → { vista: string, params: object }`.
  - `vista` es uno de estos valores: `'portada'`, `'enciclopedia'`, `'mapa'`,
    `'tema'`, `'quiz'`, `'gran-quiz'` o `'no-encontrado'`.
- Produce: `iniciarRouter(vistas: Record<string, (params, contenedor) => void>)`.
  - Escucha `hashchange` y pinta en `#pantalla`.
- Rutas:

  | Dirección | Vista |
  |---|---|
  | `''` y `#/` | portada |
  | `#/enciclopedia` | enciclopedia |
  | `#/espacio` | mapa |
  | `#/espacio/gran-quiz` | gran-quiz |
  | `#/espacio/<id>` | tema |
  | `#/espacio/<id>/quiz` | quiz |

  Un `<id>` que no está en `ORDEN` lleva a no-encontrado.

- [ ] **Paso 1:** escribir `tests/router.test.js`. Debe comprobar:
  - `resolverRuta('')` y `resolverRuta('#/')` dan `{vista:'portada', params:{}}`.
  - `resolverRuta('#/espacio/marte')` da
    `{vista:'tema', params:{seccion:'espacio', id:'marte'}}`.
  - `resolverRuta('#/espacio/marte/quiz')` da la vista `'quiz'`.
  - `resolverRuta('#/espacio/plutonn')` da la vista `'no-encontrado'`.
  - `resolverRuta('#/xyz')` da la vista `'no-encontrado'`.
- [ ] **Paso 2:** correr `npm test`. Resultado esperado: FALLA porque el módulo no
  existe.
- [ ] **Paso 3:** escribir `router.js`. Importa `ORDEN` desde
  `content/espacio/indice.js`. Por ahora ese archivo trae la lista de 14 ids con
  `TEMAS = {}`; la Tarea 3 lo completa.
- [ ] **Paso 4:** correr `npm test`. Resultado esperado: PASA.
- [ ] **Paso 5:** escribir `scripts/servir.js` con `node:http`. Sirve `app/`, toma
  el puerto de `PORT` (por defecto 5173) y entrega el tipo de contenido correcto
  para `.js`, `.css`, `.svg`, `.woff2` y `.webmanifest`.
- [ ] **Paso 6:** escribir `e2e/navegacion.spec.js`. Con Playwright, en un viewport
  de Pixel 7:
  - Ir a `#/espacio/plutonn` muestra el texto "No encontré ese lugar" y el botón
    "Volver al mapa".
  - Ir a `#/`, luego a `#/enciclopedia` y después `page.goBack()` deja de nuevo la
    portada.

  Instalar Playwright con `npm i -D @playwright/test` y
  `npx playwright install chromium`.
- [ ] **Paso 7:** correr `npm run e2e`. Resultado esperado: PASA.
- [ ] **Paso 8:** hacer commit con el mensaje `feat: andamiaje, router con # y pantalla no encontrado`.

### Tarea 2: Progreso del niño

**Archivos:**
- Crear: `app/js/progreso.js`.
- Prueba: `tests/progreso.test.js`.

**Interfaces:**
- Produce estas funciones puras:
  - `progresoVacio() → { leidos: [], estrellas: {}, granQuiz: 0, sonido: true }`
  - `marcarLeido(p, id) → p'`
  - `registrarQuiz(p, id, aciertos: 0..3) → p'`. Guarda el máximo entre el valor
    anterior y el nuevo.
  - `registrarGranQuiz(p, estrellas: 0..3) → p'`. También guarda el máximo.
  - `totalEstrellas(p) → number`. Es la suma de `estrellas` más `granQuiz`. El
    máximo es 45.
  - `ponerSonido(p, bool) → p'`
- Produce las funciones de almacenamiento, que reciben `almacen` (un objeto con
  `getItem` y `setItem`, por defecto `localStorage`):
  - `cargar(almacen) → p`
  - `guardar(almacen, p)`
  - `borrar(almacen)`
- La clave es `curiosipedia.v1`.

- [ ] **Paso 1:** escribir las pruebas:
  - Hacer quiz con 2 aciertos y luego con 1 deja 2.
  - Hacer quiz con 1 acierto y luego con 3 deja 3.
  - `marcarLeido` dos veces seguidas no duplica el tema.
  - `totalEstrellas` suma también el Gran quiz.
  - `cargar` con `'{roto'` da `progresoVacio()`.
  - `cargar` con un `getItem` que lanza error da `progresoVacio()`.
  - `guardar` con un `setItem` que lanza error, por ejemplo por cuota llena, no
    lanza error.
  - `cargar` con un JSON viejo al que le faltan campos completa los que faltan
    con los valores por defecto.
- [ ] **Paso 2:** correr `npm test`. Resultado esperado: FALLA.
- [ ] **Paso 3:** implementar. Las funciones puras no modifican el objeto de
  entrada.
- [ ] **Paso 4:** correr `npm test`. Resultado esperado: PASA.
- [ ] **Paso 5:** hacer commit con el mensaje `feat: progreso con estrellas, mejor resultado y almacenamiento a prueba de fallos`.

### Tarea 3: Esquema del contenido y validador

**Archivos:**
- Crear: `app/content/espacio/indice.js` (`ORDEN`, `TEMAS`, `SECCION`), el tema
  de ejemplo completo `app/content/espacio/marte.js` y `tests/contenido.test.js`.

**Interfaces:**
- Produce `ORDEN: string[]`, con los 14 ids en el orden de arriba.
- Produce `TEMAS: Record<id, Tema>`, que importa cada `<id>.js`. Mientras un tema
  no exista se usa un tema provisional que marca `borrador: true`.
- Produce `SECCION = { id:'espacio', titulo:'El espacio' }`.
- Produce `validarTema(t) → string[]`, en `tests/validar-tema.js`. Devuelve la
  lista de errores; una lista vacía significa que el tema es válido.

- [ ] **Paso 1:** escribir las pruebas. Para cada id de `ORDEN` que no sea
  `borrador`, `validarTema` debe devolver `[]`. El validador comprueba:
  - El texto tiene entre 120 y 200 palabras.
  - La ficha tiene 3 o 4 datos y hay 2 o 3 curiosidades.
  - El quiz tiene 3 preguntas, cada una con 3 opciones distintas.
  - `correcta` está en {0, 1, 2} y cada pregunta tiene `explicacion`.
  - No hay `http`, `www.` ni `<a` en ningún texto.
  - Existe `app/art/espacio/<id>.svg`. Mientras sea `borrador`, esta comprobación
    solo avisa.

  Agregar dos pruebas más del validador con temas de ejemplo mal hechos:
  - Uno con 4 opciones debe dar un error.
  - Uno con el texto `"ver https://x.com"` debe dar un error.
- [ ] **Paso 2:** correr `npm test`. Resultado esperado: FALLA.
- [ ] **Paso 3:** implementar el validador y escribir `marte.js` con texto real.
  Este tema sirve de modelo de tono para la Tarea 9.
- [ ] **Paso 4:** correr `npm test`. Resultado esperado: PASA.
- [ ] **Paso 5:** hacer commit con el mensaje `feat: esquema de contenido, validador y tema modelo Marte`.

### Tarea 4: Portada "tablero de mando", constelaciones y Chispa

**Archivos:**
- Crear: `app/js/constelaciones.js`, `app/js/vistas/portada.js`,
  `app/js/vistas/enciclopedia.js` y `app/art/chispa.svg`.
- Agregar: estilos en `css/pantallas.css`.
- Prueba: `tests/constelaciones.test.js` y `e2e/portada.spec.js`.

**Interfaces:**
- Consume: `cargar` y `totalEstrellas` (Tarea 2).
- Produce: `CONSTELACIONES`, una lista de 5 constelaciones
  `{ nombre, puntos: [{x,y}], lineas: [[i,j]] }`.
  - Son 9 puntos por constelación, 45 en total.
  - Nombres: "El Cohete", "La Ballena de Chispa", "El Telescopio",
    "La Bicicleta Lunar" y "El Gato Cometa".
- Produce: `puntosEncendidos(total) → { completas: string[], parcial: { nombre, encendidos } | null }`.
  - Va llenando las constelaciones en orden.
- `chispa.svg` tiene tres estados, que se cambian con una clase en el contenedor:
  `.chispa--normal`, `.chispa--feliz` y `.chispa--animo`.

- [ ] **Paso 1:** escribir las pruebas:
  - `puntosEncendidos(0)` no tiene constelaciones completas ni parcial.
  - `puntosEncendidos(9)` tiene completa "El Cohete" y la parcial es `null`.
  - `puntosEncendidos(12)` tiene una completa y una parcial con 3 encendidos.
  - `puntosEncendidos(45)` tiene las 5 completas.
  - Ninguna constelación tiene un número de puntos distinto de 9.
- [ ] **Paso 2:** correr `npm test`. Resultado esperado: FALLA.
- [ ] **Paso 3:** implementar.
  - La portada tiene un cielo en SVG con los puntos encendidos y las líneas de
    las constelaciones completas, cada una con su nombre.
  - Debajo hay 4 palancas o botones grandes:
    - Enciclopedia lleva a `#/enciclopedia`.
    - Programar, Animar y Hoy muestran "En construcción en el taller", con Chispa
      y una llave inglesa.
  - Se ve el contador "Mis estrellas: N / 45".
  - La vista Enciclopedia tiene la tarjeta El espacio, que lleva a `#/espacio`.
    Animales, Historia, Geografía y Ciencia aparecen como "¡Pronto!".
- [ ] **Paso 4:** escribir `e2e/portada.spec.js`:
  - Con `curiosipedia.v1` sembrado en 12 estrellas, se ven el texto "El Cohete" y
    "Mis estrellas: 12".
  - Tocar "Programar" muestra "En construcción".
  - El botón pequeño "Empezar de cero", al pie de la portada, abre una
    confirmación dentro de la app. No usa `confirm()`.
    - "Sí, borrar" deja las estrellas en 0.
    - "No" conserva el avance.
- [ ] **Paso 5:** correr `npm test && npm run e2e`. Resultado esperado: PASA.
- [ ] **Paso 6:** hacer commit con el mensaje `feat: portada tablero de mando con constelaciones y Chispa`.

### Tarea 5: Mapa del Sistema Solar

**Archivos:**
- Crear: `app/js/vistas/mapa.js` y los estilos del mapa.
- Prueba: `e2e/mapa.spec.js`.

**Interfaces:**
- Consume: `ORDEN`, `TEMAS` (Tarea 3) y `cargar` (Tarea 2).
- Produce: botones `[data-tema="<id>"]` y un botón `[data-accion="gran-quiz"]`.

- [ ] **Paso 1:** escribir la prueba e2e:
  - El mapa tiene 14 botones `[data-tema]`.
  - Los temas leídos llevan la clase `visitado`, que muestra la banderita.
  - Tocar Marte lleva a `#/espacio/marte` en menos de 1,5 s.
  - Existe el botón del Gran quiz.
- [ ] **Paso 2:** correr `npm run e2e`. Resultado esperado: FALLA.
- [ ] **Paso 3:** implementar.
  - Es una franja horizontal con `scroll-snap`.
  - El Sol queda a la izquierda y las órbitas son arcos en SVG.
  - Los planetas van en orden.
  - Al final hay un "cinturón" con planetas enanos y asteroides, y un "cielo
    profundo" con estrellas y astronautas.
  - La animación de viaje mueve la nave hasta el planeta en 600 ms y luego cambia
    el `hash`. Con movimiento reducido, cambia el `hash` de inmediato.
- [ ] **Paso 4:** correr `npm run e2e`. Resultado esperado: PASA.
- [ ] **Paso 5:** hacer commit con el mensaje `feat: mapa del Sistema Solar desplazable con viaje animado`.

### Tarea 6: Vista de tema y "¿Qué tan grande?"

**Archivos:**
- Crear: `app/js/vistas/tema.js` y `app/js/vistas/escala.js`.
- Prueba: `tests/escala.test.js` y `e2e/tema.spec.js`.

**Interfaces:**
- Consume: `TEMAS`, `marcarLeido`, `guardar` y Chispa.
- Produce: `escalaRelativa(diametroKm, anchoPx) → { tierraPx, otroPx, recortado: boolean }`.
  - El diámetro de la Tierra es 12 742 km.
  - El cuerpo más grande ocupa como máximo el 90 % del ancho.
  - Si el radio del otro cuerpo supera 4 veces el ancho, `recortado` es `true` y
    solo se dibuja un pedazo de su borde. Eso pasa con el Sol.

- [ ] **Paso 1:** escribir las pruebas:
  - Para Marte con un ancho de 400, `otroPx` y `tierraPx` guardan la proporción
    6779/12742, con una tolerancia de ±1 px.
  - Para el Sol (1 392 700 km) con un ancho de 400, `recortado` es `true`.
  - Para la Luna (3474 km), `otroPx` es menor que `tierraPx`.
- [ ] **Paso 2:** correr `npm test`. Resultado esperado: FALLA.
- [ ] **Paso 3:** implementar la vista de tema:
  - Arriba, el dibujo `art/espacio/<id>.svg`, con un giro o flotación suave.
  - El texto, sobre una tarjeta de fondo sólido.
  - La ficha de datos.
  - Al tocar el dibujo, Chispa dice la siguiente curiosidad en un globo y la
    lista da la vuelta.
  - Los botones "¿Qué tan grande?" (solo si `diametroKm` existe), "Hacer el quiz",
    "← Anterior" y "Siguiente →", según `ORDEN`.
  - Las estrellas ganadas en el tema (☆☆☆).
  - Al abrir el tema se llama a `marcarLeido`.
- [ ] **Paso 4:** escribir la prueba e2e:
  - Abrir Marte lo marca como leído en `localStorage`.
  - Tocar el dibujo muestra el globo con la primera curiosidad.
  - "Siguiente →" en Neptuno lleva a `planetas-enanos`.
  - En `astronautas` no aparece "Siguiente".
- [ ] **Paso 5:** correr `npm test && npm run e2e`. Resultado esperado: PASA.
- [ ] **Paso 6:** hacer commit con el mensaje `feat: vista de tema con curiosidades de Chispa y comparación de tamaños`.

### Tarea 7: Quiz, Gran quiz, sonido y vibración

**Archivos:**
- Crear: `app/js/quiz.js`, `app/js/sonido.js`, `app/js/vistas/quiz.js` y
  `app/js/vistas/gran-quiz.js`.
- Prueba: `tests/quiz.test.js` y `e2e/quiz.spec.js`.

**Interfaces:**
- Consume: `registrarQuiz`, `registrarGranQuiz`, `ponerSonido` y `TEMAS`.
- Produce: `estrellasGranQuiz(aciertos) → 0..3`.
  - 9 aciertos o más dan 3, 7 o más dan 2, 5 o más dan 1, y menos dan 0.
- Produce: `armarGranQuiz(temas: Tema[], azar = Math.random) → Pregunta[10]`.
  - No repite preguntas y toma de al menos 7 temas distintos.
- Produce: `bip(tipo: 'bien'|'mal'|'estrella')` y `vibrar(ms)`.
  - No hacen nada si el sonido está apagado o si no hay `AudioContext` o
    `navigator.vibrate`.

- [ ] **Paso 1:** escribir las pruebas:
  - `estrellasGranQuiz` devuelve 0 con 4 aciertos, 1 con 5, 1 con 6, 2 con 7,
    3 con 9 y 3 con 10.
  - `armarGranQuiz` con un azar fijo devuelve 10 preguntas sin repetidas, de al
    menos 7 temas.
  - `armarGranQuiz` con 2 temas, que dan solo 6 preguntas, devuelve 6 sin lanzar
    error.
- [ ] **Paso 2:** correr `npm test`. Resultado esperado: FALLA.
- [ ] **Paso 3:** implementar las vistas:
  - Cada pregunta llega como "📡 Mensaje de la base".
  - Al elegir una opción, todas las opciones se desactivan. Esto evita contar dos
    veces un doble toque.
  - Se marca la opción correcta y se muestra la `explicacion`. Al acertar suena
    el bip, el celular vibra 30 ms y Chispa pone cara `feliz`. Al fallar, Chispa
    pone cara `animo`.
  - Las estrellas **solo se guardan en la pantalla final**. Salir a la mitad no
    cuenta.
  - Hay un botón para silenciar el sonido en la cabecera.
- [ ] **Paso 4:** escribir la prueba e2e:
  - Dar doble clic rápido en la respuesta correcta suma 1 acierto, no 2.
  - Responder 2 preguntas y salir deja `estrellas.marte` sin definir.
  - Terminar con 3 aciertos deja `estrellas.marte === 3`.
  - Repetir con 1 acierto mantiene 3.
- [ ] **Paso 5:** correr `npm test && npm run e2e`. Resultado esperado: PASA.
- [ ] **Paso 6:** hacer commit con el mensaje `feat: quiz de radio, Gran quiz, sonido y vibración`.

### Tarea 8: Dibujos en papel recortado (en paralelo con la Tarea 9)

**Archivos:**
- Crear: `app/art/espacio/<id>.svg` (14 archivos), `app/icons/icon.svg` y los PNG
  del ícono: 192, 512 y maskable de 512. Los PNG se generan con Playwright a
  partir del SVG, usando `scripts/iconos.js`.
- Prueba: agregar a `tests/contenido.test.js`.

- [ ] **Paso 1:** agregar estas pruebas:
  - Cada SVG pesa menos de 60 KB.
  - Cada SVG tiene `viewBox`.
  - Ningún SVG tiene `<image` ni `href="http`.
  - Existen los 3 PNG del ícono, con las medidas correctas leídas de la
    cabecera PNG.
- [ ] **Paso 2:** correr `npm test`. Resultado esperado: FALLA.
- [ ] **Paso 3:** dibujar. El estilo es papel recortado:
  - Figuras planas en 3 a 5 capas.
  - Sombra suave con `feDropShadow`.
  - Textura de papel con `feTurbulence`, de opacidad baja, definida una sola vez
    por archivo.
  - Cada dibujo usa el `color` de su tema.
  - El ícono es un cohete de papel sobre un planeta, y el de versión adaptable
    deja un margen del 20 %.
- [ ] **Paso 4:** correr `npm test`. Resultado esperado: PASA. Revisar a ojo en
  `npm run servir`.
- [ ] **Paso 5:** hacer commit con el mensaje `feat: dibujos de papel recortado e ícono`.

### Tarea 9: Textos de los 14 temas y sus fuentes (en paralelo con la Tarea 8)

**Archivos:**
- Crear: `app/content/espacio/<id>.js`, para los 13 temas que faltan, y
  `content-fuentes/FUENTES.md`.

- [ ] **Paso 1:** escribir cada tema con el tono de `marte.js`:
  - Frases cortas, segunda persona y algo de humor.
  - Las cifras deben estar comprobadas en NASA, ESA o IAU.
  - Las cifras que cambian se escriben con fecha ("hasta 2026, ...").
  - En `FUENTES.md` va una lista por tema: dato → URL.
  - No se copian frases de las fuentes ni de Encarta.
  - No se reutilizan los textos de `planetas.html`.
- [ ] **Paso 2:** correr `npm test`. Resultado esperado: PASA. Los 14 temas tienen
  que validar y ninguno puede quedar como `borrador`.
- [ ] **Paso 3:** hacer una revisión cruzada. Un agente distinto al que escribió
  los textos comprueba cada cifra contra `FUENTES.md` y corrige lo que esté mal.
- [ ] **Paso 4:** hacer commit con el mensaje `feat: textos de la sección El espacio con fuentes`.

### Tarea 10: Sin internet, instalación y aviso de novedades

**Archivos:**
- Crear: `app/manifest.webmanifest`, `app/sw.js`, `scripts/generar-precache.js`,
  `app/precache.js` (generado) y `app/fonts/nunito-latin.woff2`.
- Modificar: `app/js/main.js`.
- Prueba: `tests/precache.test.js` y `e2e/sin-internet.spec.js`.

**Interfaces:**
- `precache.js` define `self.VERSION`, que es un hash corto del contenido, y
  `self.ARCHIVOS`, con todas las rutas relativas de `app/` excepto `sw.js`, más
  `./`.
- `sw.js` funciona así:
  - En `install`, guarda todo en el caché `curiosipedia-<VERSION>`.
  - En `activate`, borra los cachés `curiosipedia-*` de otras versiones.
  - En `fetch`, sirve primero desde el caché. Si la página no está guardada,
    devuelve `index.html`.
  - **No** llama a `skipWaiting` por su cuenta. Lo hace cuando recibe el mensaje
    `{tipo:'activar'}`.
- `main.js` muestra la banda "Hay novedades ✨ Tocar para actualizar" cuando
  existe `registration.waiting`. Al tocarla envía `activar` y recarga la página
  cuando cambia el `controllerchange`.
- `manifest.webmanifest`:
  - `name` y `short_name` son `"Curiosipedia"`.
  - `start_url` y `scope` son `"./"`.
  - `display` es `"standalone"`.
  - `orientation` es `"portrait"`.
  - `background_color` y `theme_color` son `"#0B0F3D"`.
  - Lleva los 3 íconos.
- La letra se obtiene así: se descarga Nunito, que tiene licencia OFL, y se
  reduce a los caracteres latinos con `pyftsubset` (`pip install fonttools`). Su
  licencia se guarda en `app/fonts/OFL.txt`.

- [ ] **Paso 1:** escribir `tests/precache.test.js`:
  - Todo archivo bajo `app/`, excepto `sw.js`, está en `ARCHIVOS`.
  - Todo lo que está en `ARCHIVOS` existe en `app/`.
  - `app/` pesa menos de 5 MB.
  - `nunito-latin.woff2` pesa menos de 150 KB.
  - No hay ningún `http` en `ARCHIVOS`.
- [ ] **Paso 2:** escribir `e2e/sin-internet.spec.js`:
  - Abre la app, espera a que el service worker esté `activated` y luego activa
    `context.setOffline(true)`.
  - Al recargar `#/espacio/marte` se ve el título "Marte".
  - Al ir a `#/espacio/saturno/quiz` se ve "Mensaje de la base".
  - Después se cambia un archivo, se regenera el precache y se vuelve a cargar con
    internet. Debe aparecer "Hay novedades".
- [ ] **Paso 3:** correr `npm test && npm run e2e`. Resultado esperado: FALLA.
- [ ] **Paso 4:** implementar.
  - Agregar `generar-precache` como paso previo de `npm test`, para que la lista
    nunca quede desactualizada.
- [ ] **Paso 5:** correr `npm test && npm run e2e`. Resultado esperado: PASA.
- [ ] **Paso 6:** hacer commit con el mensaje `feat: funciona sin internet, se instala y avisa novedades`.

### Tarea 11: Vista previa para que el adulto apruebe

**Archivos:**
- Crear: `scripts/vista-previa.js`, que genera `revision/vista-previa.html`.
- Agregar `revision/` al `.gitignore`.

- [ ] **Paso 1:** el script arma una sola página HTML. Para cada tema muestra:
  - El dibujo, el texto, la ficha y las curiosidades.
  - El quiz, con la respuesta correcta marcada.
  - Las fuentes de ese tema, tomadas de `FUENTES.md`.
- [ ] **Paso 2:** correr `node scripts/vista-previa.js`. Resultado esperado: la
  página existe y tiene 14 secciones `<section data-tema>`.
- [ ] **Paso 3:** hacer commit con el mensaje `build: vista previa de contenidos para revisión`.
- [ ] **Paso 4:** entregar la vista previa al usuario. **Esperar su aprobación** y
  aplicar los cambios que pida, en commits `fix:`.

### Tarea 12: Publicar en GitHub Pages (requiere permiso del usuario)

**Archivos:**
- Crear: `.github/workflows/pages.yml`.

- [ ] **Paso 1:** crear un flujo que corre al hacer `push` a `main`:
  - `npm ci`, luego `npm test`, luego `actions/upload-pages-artifact` con
    `path: app` y luego `actions/deploy-pages`.
  - Permisos: `pages: write` e `id-token: write`.
- [ ] **Paso 2:** hacer commit con el mensaje `build: publicación automática en GitHub Pages`.
- [ ] **Paso 3:** pedir permiso al usuario. Con su "sí", hacer merge a `main` y
  `push`. El usuario activa Pages en *Settings → Pages → Source: GitHub Actions*.
- [ ] **Paso 4:** comprobar que `https://sonyrp.github.io/curiosipedia/` responde
  200 y que `manifest.webmanifest` carga.
- [ ] **Paso 5:** guiar la instalación en el Android: abrir el enlace en Chrome,
  menú ⋮, "Añadir a pantalla de inicio". Después probar en modo avión.

---

## Orden y paralelismo

- Las Tareas **1 → 2 → 3** van en serie, porque son la base.
- Después pueden ir en paralelo, cada una en su propio worktree:
  - **4**.
  - **5 + 6**, que comparten la navegación del mapa al tema.
  - **7**.
  - **8**.
  - **9**.
- **10** va cuando ya existen todos los archivos.
- Luego **11**.
- Al final **12**, con permiso del usuario.
