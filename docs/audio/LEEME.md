# Música de fondo

Archivo en uso: **`cancion.m4a`**

> Carlos Rivera — *Que Lo Nuestro Se Quede Nuestro* (versión acústica)
> AAC 129 kbps · 44.1 kHz · 3:45 · 3.48 MB

`docs/index.html` lo carga desde `audio/cancion.m4a`. Si el archivo no existe
o el navegador no puede decodificarlo, el botón flotante simplemente no aparece
y la invitación funciona igual.

## Por qué `.m4a` y no `.mp3`

El archivo original llegó con extensión `.mp3`, pero **no era un MP3**. Su
cabecera es `ftyp` con marcas `dash/iso6/mp41`: un contenedor **MP4 fragmentado**
con pista de audio **AAC** (`mp4a` + `esds`).

Servirlo como `.mp3` haría que GitHub Pages enviara `Content-Type: audio/mpeg`
para un contenido que en realidad es MP4. La mayoría de los navegadores lo
tolera porque analizan el contenedor, pero es incorrecto y frágil. Con `.m4a`
el servidor manda `audio/mp4`, que sí corresponde.

No hay pérdida de calidad: no se recodificó nada, solo se renombró. AAC a
129 kbps suena mejor que un MP3 al mismo bitrate, así que **no conviene
convertirlo a MP3** — solo perdería calidad y ganaría peso.

## Compatibilidad

AAC en contenedor MP4 se reproduce en Chrome, Edge, Safari, Firefox, iOS y
Android. El `moov` está al inicio del archivo, así que empieza a sonar sin
esperar la descarga completa.

## Cómo se comporta

La canción **nunca se reproduce en silencio**: si suena, suena con audio,
y siempre desde el principio.

1. Al cargar la página se intenta reproducir de inmediato **con sonido**.
2. Si el navegador lo bloquea (ver abajo), la canción arranca en el primer
   toque, tecla o scroll del invitado — desde el segundo cero, con un fundido
   de entrada hasta 35% de volumen.
3. El botón flotante permite silenciar en cualquier momento. Si el invitado
   silencia a propósito, la música ya no vuelve a arrancar sola.
4. Al cambiar de pestaña o minimizar se pausa, y retoma al volver.

### El límite del autoplay

Ningún navegador moderno permite reproducir audio automáticamente si el
visitante no ha interactuado antes con la página. Es una política de Chrome,
Safari, Firefox, Edge, iOS y Android, y **no existe forma de evitarla desde
el código**.

En la práctica esto significa que un invitado que abre el enlace por primera
vez escuchará la música en cuanto haga su primer toque o scroll, no antes.

En tu propia computadora quizá sí arranque sola: Chrome lleva un índice de
interacción por sitio (*Media Engagement Index*) y permite el autoplay en
sitios que visitas seguido. No te confíes de esa prueba — para tus invitados
será su primera visita.

Si quieres garantizar que la música suene desde el arranque, la solución es
una **pantalla de entrada** con un botón tipo «Abrir invitación». Ese primer
toque es la interacción que el navegador exige, así que la canción entra con
sonido siempre. Es lo que hacen la mayoría de las invitaciones digitales.

## Cambiar la canción

Reemplaza el archivo y ajusta esta línea en `docs/index.html`, cuidando que
`type` corresponda al formato real:

```html
<source src="audio/cancion.m4a" type="audio/mp4">
```

| Formato real | Extensión | `type` |
|---|---|---|
| MPEG audio (MP3 de verdad) | `.mp3` | `audio/mpeg` |
| AAC en MP4 | `.m4a` | `audio/mp4` |
| Ogg Vorbis | `.ogg` | `audio/ogg` |

Para música de fondo, 128–160 kbps y menos de 4 MB es el punto justo: muchos
invitados abrirán la invitación con datos móviles. El volumen final se ajusta
con `VOL_MAX` en `docs/index.html`.
