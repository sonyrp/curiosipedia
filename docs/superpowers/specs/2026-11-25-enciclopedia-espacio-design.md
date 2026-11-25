# Diseño: enciclopedia para el celular, versión 1 (el espacio)

Fecha: 2026-11-25 · Estado: borrador para aprobar

## 1. Para qué y para quién

- **Para quién:** un niño de 10 años que ya lee bien. Usa un celular **Android**.
- **Para qué:** que aprenda de muchas cosas por su cuenta y con gusto, en una app
  hecha para él, segura y sin distracciones.
- **Inspiración:** "Mi primera Encarta", **sin copiarla**: nombre, diseño, dibujos y
  textos propios.
- **Por qué empezar pequeño:** la versión 1 trae solo el espacio. Así el niño la
  tiene pronto y vemos si le gusta antes de construir lo demás.

## 2. Objetivos, lo que no entra y lo que sería un fracaso

**Objetivos de la versión 1**
1. Se instala en Android desde Chrome con "Añadir a pantalla de inicio" y abre
   como una app, a pantalla completa y con su propio ícono.
2. **Funciona sin internet** desde la primera vez que se abre: todo el contenido
   queda guardado en el celular.
3. Portada con las cuatro áreas: **Enciclopedia**, **Programar**, **Animar** y **Hoy**.
   Solo la Enciclopedia se abre. Las otras tres muestran "¡Pronto!".
4. Dentro de la Enciclopedia, la sección **El espacio** completa (ver punto 5).
5. Cada tema tiene un quiz corto. El niño gana estrellas y ve su avance.

**No entra en la versión 1**
- Programar con bloques, animación cuadro a cuadro y temas actuales.
- Otras secciones de la Enciclopedia: animales, historia, geografía y ciencia.
- Publicarla en Google Play.
- Cuentas, inicio de sesión y sincronizar entre celulares.

**Sería un fracaso aunque funcione**
- Que el niño pueda salir a internet desde la app: enlaces externos, anuncios,
  videos de terceros o chat con IA.
- Que tenga **datos falsos o confusos**, como pasa hoy en `planetas.html`.
- Que se parezca a Encarta en el nombre, los dibujos o los textos.
- Que recoja datos del niño o los envíe a algún lado.
- Que sea aburrida: muros de texto sin dibujos ni nada que tocar.

## 3. Restricciones

- Android con Chrome. Tiene que verse bien en una pantalla vertical de unos 360 a
  430 px de ancho y no romperse en una tableta.
- Sin internet después de la primera carga.
- Publicada en una dirección web con HTTPS para poder instalarse. Se propone
  **GitHub Pages**, que es gratis. Necesita una cuenta de GitHub.
- Liviana: toda la app debe pesar **menos de 5 MB** para que se guarde rápido.
- Todo el contenido en español, con letra y botones grandes, fáciles de tocar.

## 4. Formas de construirla (ya decidido)

| Opción | A favor | En contra |
|---|---|---|
| **A. App web instalable (elegida)** | Sirve en Android y en iPhone, funciona sin internet y se actualiza con solo cambiar los archivos. | Hay que publicarla en una dirección web. |
| B. App nativa de Android (.apk) | Se siente más "app de verdad". | Es más trabajo y cada cambio exige instalarla de nuevo. |
| C. Página en claude.ai | Es lo más rápido. | El niño necesitaría una cuenta de claude.ai, que no es apropiada para su edad. |

**Decidido por el usuario:** opción A, Android, el espacio como primer tema, sin
internet desde el principio, y solo el espacio en la versión 1.

## 5. Qué trae la versión 1

### Pantallas
```
Portada                El espacio              Tema (ej. Marte)
┌──────────────────┐   ┌──────────────────┐    ┌──────────────────┐
│ ★ Curiosipedia ★ │   │ ← El espacio     │    │ ← Marte    ☆☆☆   │
│                  │   │                  │    │   [dibujo]       │
│ [Enciclopedia]   │   │ [El Sol] [Luna]  │    │ Texto corto...   │
│ [Programar ¡Pronto!]│ │ [8 planetas...]  │    │ 💡 ¿Sabías que?  │
│ [Animar ¡Pronto!]│   │ [Estrellas]      │    │ [Hacer el quiz]  │
│ [Hoy ¡Pronto!]   │   │ [Astronautas]    │    │ [← Ant.] [Sig. →]│
│ Mis estrellas: 12│   │ [Gran quiz]      │    └──────────────────┘
└──────────────────┘   └──────────────────┘
```
Dentro de la Enciclopedia hay una tarjeta por sección. En la versión 1 solo
está **El espacio**. Las demás secciones aparecen como "¡Pronto!" para que se
vea hacia dónde va la app.

### Temas de El espacio (unos 14)
- El Sol.
- Los 8 planetas, uno por tema.
- La Luna.
- Planetas enanos, asteroides y cometas.
- Las estrellas y las constelaciones.
- Astronautas y exploración espacial.

### Cada tema tiene
- Un **dibujo propio** hecho en SVG. No se usan fotos con derechos de autor.
  Los dibujos son livianos y funcionan sin internet.
- Un **texto corto**, de 120 a 200 palabras, escrito para un niño de 10 años.
- Una **ficha de datos** con 3 o 4 cifras clave. Por ejemplo, para Marte:
  distancia al Sol, cuánto dura un día y número de lunas.
- 2 o 3 **"¿Sabías que?"**.
- Un **quiz** de 3 preguntas con tres opciones cada una. Da hasta 3 estrellas.
- **Botones** para ir al tema anterior y al siguiente.
- Al final de la sección hay un **Gran quiz** de 10 preguntas mezcladas.

### Dibujos (decisión de Claude)
- Dibujos fijos en SVG, con animaciones suaves hechas en CSS. Por ejemplo, el
  planeta gira despacio y las estrellas titilan.
- Si el niño toca el dibujo, aparece una curiosidad. Así cada tema tiene algo
  para tocar, además del quiz.
- No hay escenas complejas ni partes del dibujo que se puedan tocar por
  separado. Eso puede venir en versiones siguientes.

### Reglas del quiz (decisión de Claude)
- Cada respuesta correcta da 1 estrella, hasta 3 por tema.
- Se puede repetir el quiz. Cuenta el mejor resultado.
- Al responder, la app dice si la respuesta está bien o mal y explica por qué
  en una frase.
- El Gran quiz da hasta 3 estrellas extra:
  - 1 estrella con 5 aciertos o más.
  - 2 estrellas con 7 o más.
  - 3 estrellas con 9 o más.
- Un tema cuenta como "leído" cuando el niño lo abre.

### Avance del niño
- Se guarda solo en el celular: qué temas leyó y cuántas estrellas ganó.
- No se pide el nombre del niño ni ningún otro dato personal.
- La portada muestra el total de estrellas.
- Hay un botón para borrar el avance y empezar de cero, con confirmación
  dentro de la app.

## 6. Contenido: cómo se asegura que sea correcto

- **Claude** escribe los textos y comprueba cada dato con fuentes confiables:
  NASA, la Agencia Espacial Europea (ESA) y la Unión Astronómica Internacional
  (IAU). Las fuentes quedan anotadas en un archivo aparte, no en la app.
- Las cifras que cambian, como el número de lunas, se escriben con su fecha
  ("hasta 2026, Saturno tiene más de 270 lunas conocidas") o de forma general.
- **El adulto** lee y aprueba los textos antes de publicarlos. Decisión de Claude
  sobre cómo hacerlo:
  - Claude genera una página de vista previa con todos los temas, tal como se
    verán, y una lista de las fuentes.
  - El adulto aprueba todo junto o señala qué cambiar.
  - La app se puede construir mientras tanto, pero no se publica sin esa
    aprobación.
- No se reutilizan los textos de `planetas.html`: tienen errores. Sí se reutiliza
  su estilo: fondo de estrellas, colores vivos y botones redondos.

## 7. Cómo se construye (decisiones de Claude, se pueden cambiar)

- **Tecnología:** HTML, CSS y JavaScript sin frameworks ni dependencias externas,
  y sin paso de compilación. Es lo más simple de mantener y lo más liviano.
- **Sin internet:**
  - Un *service worker* guarda todos los archivos la primera vez que se abre la app.
  - Un archivo *manifest* le da nombre, ícono y pantalla completa.
  - Cada versión nueva cambia el número de versión del service worker.
  - La app avisa "Hay novedades" y se actualiza la próxima vez que se abra.
- **Contenido separado del código:** cada tema es un archivo de datos con su
  título, texto, ficha, curiosidades y quiz. Agregar una sección nueva es sobre
  todo agregar datos.
- **Letra:** una tipografía redondeada y de licencia libre (OFL), por ejemplo
  Nunito. Se guarda dentro de la app, solo con los caracteres del español
  (tildes, ñ, ¿ y ¡), en menos de 150 KB. La Comic Sans de `planetas.html` no
  existe en Android.
- **Ícono:** Claude lo dibuja: un cohete o un planeta. Se hace en los tamaños
  que pide Android, 192 y 512 px, y en versión adaptable ("maskable").
- **Rutas relativas:** todas las rutas de archivos son relativas, para que la
  app funcione igual en la raíz de un dominio o en una subcarpeta como
  `usuario.github.io/curiosipedia/`.
- **Nombre:** nombre provisional **"Curiosipedia"**. El adulto puede cambiarlo.
- **Pruebas:**
  - Pruebas automáticas que revisan que todos los temas tengan sus partes
    completas, que cada quiz tenga una sola respuesta correcta y que no haya
    enlaces externos.
  - Una prueba en el navegador que abre la app, la pasa a modo sin internet y
    confirma que sigue funcionando.
- **Proyecto:** en la carpeta `app-enciclopedia/`, con git y una rama por tarea.
  Nada se sube a GitHub sin permiso del adulto.

## 8. Lo que queda a criterio de quien construye

- Los colores exactos, las animaciones y el diseño de cada dibujo, siempre que
  sigan el estilo espacial alegre de `planetas.html`.
- Las preguntas concretas de cada quiz.
- El orden de los temas dentro de la sección.
