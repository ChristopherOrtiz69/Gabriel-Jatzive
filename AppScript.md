# Google Apps Script — Gabriel & Jatzive

Pega este código en **Extensiones → Apps Script** dentro de tu Google Sheet.
Reemplaza todo el código existente y vuelve a implementar como *Aplicación web*.

> **Versión actual del código: `v2-tipos`**
> Incluye los 3 días del evento y los 2 tipos de invitación.

---

## Código

```javascript
/* ════════════════════════════════════════════════════════════
   Gabriel & Jatzive — backend de invitaciones
   Versión: v2-tipos (3 días · 2 tipos de invitación)
   ════════════════════════════════════════════════════════════ */
var VERSION = 'v2-tipos';

/* Tipo y Días van AL FINAL a propósito: así los índices de las columnas
   anteriores no se recorren y las hojas ya existentes siguen funcionando. */
var COLS_INV  = ['Nombre', 'Lugares', 'Confirmado', 'Fecha', 'ID', 'Tipo', 'Días'];
var COLS_ASIS = ['Grupo', 'Nombre Completo', 'Estado', 'Fecha', 'Tipo', 'Días'];

function json(obj) {
  obj.version = VERSION;
  return ContentService
    .createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}

function getSS() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  if (!ss) {
    throw new Error('El script no está enlazado a ninguna hoja. Ábrelo desde ' +
                    'Extensiones → Apps Script DENTRO de la hoja de cálculo.');
  }
  return ss;
}

function getSheet(ss, nombre, encabezados) {
  var s = ss.getSheetByName(nombre);
  if (!s) s = ss.insertSheet(nombre);

  var ancho    = s.getLastColumn();
  var actuales = ancho > 0 ? s.getRange(1, 1, 1, ancho).getValues()[0] : [];

  // Hoja vacía: escribimos todos los encabezados
  if (actuales.length === 0 || String(actuales[0]).trim() === '') {
    s.getRange(1, 1, 1, encabezados.length).setValues([encabezados]).setFontWeight('bold');
    return s;
  }

  // Hoja de una versión anterior: agregamos al final las columnas que falten
  if (actuales.length < encabezados.length) {
    var faltan = encabezados.slice(actuales.length);
    s.getRange(1, actuales.length + 1, 1, faltan.length)
     .setValues([faltan])
     .setFontWeight('bold');
  }
  return s;
}

function buscarFilaPorId(s, id) {
  var rows = s.getDataRange().getValues();
  for (var i = 1; i < rows.length; i++) {
    if (String(rows[i][4]) === String(id)) return i + 1;   // fila real (1-based)
  }
  return 0;
}

function doPost(e) {
  try {
    if (!e || !e.postData || !e.postData.contents) {
      return json({ ok: false, error: 'Petición sin cuerpo.' });
    }

    var data  = JSON.parse(e.postData.contents);
    var ss    = getSS();
    var fecha = Utilities.formatDate(new Date(), 'America/Mexico_City', 'dd/MM/yyyy HH:mm');

    if (data.action === 'add') {
      var s = getSheet(ss, 'Invitados', COLS_INV);
      s.appendRow([
        data.nombre, data.cupos, 'No ⬜', fecha, data.id,
        data.tipoEtiqueta || '', data.dias || ''
      ]);

    } else if (data.action === 'confirm') {
      var s   = getSheet(ss, 'Invitados', COLS_INV);
      var fil = buscarFilaPorId(s, data.id);
      if (!fil) return json({ ok: false, error: 'No se encontró el ID ' + data.id });
      s.getRange(fil, 3).setValue(data.confirmado ? 'Sí ✅' : 'No ⬜');

    } else if (data.action === 'tipo') {
      // Cambio de tipo de invitación hecho desde el admin
      var s   = getSheet(ss, 'Invitados', COLS_INV);
      var fil = buscarFilaPorId(s, data.id);
      if (!fil) return json({ ok: false, error: 'No se encontró el ID ' + data.id });
      s.getRange(fil, 6).setValue(data.tipoEtiqueta);
      s.getRange(fil, 7).setValue(data.dias);

    } else if (data.action === 'delete') {
      var s   = getSheet(ss, 'Invitados', COLS_INV);
      var fil = buscarFilaPorId(s, data.id);
      if (fil) s.deleteRow(fil);

      if (data.nombre) {
        ['Confirmados', 'Por Confirmar'].forEach(function (hojaName) {
          var h = ss.getSheetByName(hojaName);
          if (!h) return;
          var hRows = h.getDataRange().getValues();
          for (var i = hRows.length - 1; i >= 1; i--) {
            if (String(hRows[i][0]) === String(data.nombre)) h.deleteRow(i + 1);
          }
        });
      }

    } else if (data.action === 'asistente') {
      var hoja = data.porConfirmar ? 'Por Confirmar' : 'Confirmados';
      var s    = getSheet(ss, hoja, COLS_ASIS);
      s.appendRow([
        data.grupo,
        data.nombreCompleto,
        data.porConfirmar ? 'Por confirmar ⏳' : 'Confirmado ✅',
        fecha,
        data.tipoEtiqueta || '',
        data.dias || ''
      ]);

    } else {
      // Sin este else, una acción que el script no conoce devolvía ok:true
      // y parecía que todo había funcionado.
      return json({ ok: false, error: 'Acción desconocida: ' + data.action });
    }

    return json({ ok: true });

  } catch (err) {
    return json({ ok: false, error: err.message });
  }
}

/* Diagnóstico: abre la URL /exec en el navegador y te dice si el script
   alcanza la hoja, cómo se llama y qué pestañas tiene. */
function doGet(e) {
  var out = { ok: true };
  try {
    var ss = getSS();
    out.hoja  = ss.getName();
    out.hojas = ss.getSheets().map(function (s) { return s.getName(); });
  } catch (err) {
    out.ok    = false;
    out.error = err.message;
  }
  return json(out);
}
```

---

## Cómo verificar que quedó bien

Abre la URL `/exec` en el navegador. Debe responder algo así:

```json
{"ok":true,"hoja":"Invitados Boda","hojas":["Invitados","Confirmados","Por Confirmar"],"version":"v2-tipos"}
```

| Respuesta | Qué significa |
|---|---|
| `"ok":true` + lista de hojas | Todo correcto. |
| `"ok":false` con *You do not have permission…* | La implementación está como **«Ejecutar como: Usuario que accede»**. Cámbiala a **«Yo»**. |
| `"ok":false` con *no está enlazado a ninguna hoja* | El script es independiente. Créalo desde **Extensiones → Apps Script** dentro de la hoja. |
| `"version"` distinta de `v2-tipos` | Estás viendo una implementación vieja. Publica una **nueva versión**. |

---

## Implementación (¡importante!)

**Implementar → Administrar implementaciones → ✏️ → Nueva versión → Implementar**

Los dos ajustes que tienen que quedar así:

| Campo | Valor obligatorio |
|---|---|
| Ejecutar como | **Yo** (`tu-correo@gmail.com`) |
| Quién tiene acceso | **Cualquier persona** |

Si «Ejecutar como» queda en *Usuario que accede a la aplicación web*, el script
corre con la identidad del invitado —que no tiene acceso a tu hoja— y **todas
las escrituras fallan en silencio**. La página del invitado igual muestra
«¡Asistencia confirmada!» porque el navegador no puede leer la respuesta.

Al publicar una versión nueva la URL **no cambia**; no hay que tocar nada en el sitio.

---

## Hojas que se crean automáticamente

| Hoja | Columnas | Quién escribe |
|------|----------|---------------|
| **Invitados** | Nombre · Lugares · Confirmado · Fecha · ID · **Tipo** · **Días** | Admin (admin.html) |
| **Confirmados** | Grupo · Nombre Completo · Estado · Fecha · **Tipo** · **Días** | Invitado (desde su invitación) |
| **Por Confirmar** | Grupo · Nombre Completo · Estado · Fecha · **Tipo** · **Días** | Invitado (marcó "por confirmar") |

En **Invitados**, la columna *Días* son los días a los que la invitación da
acceso. En **Confirmados** y **Por Confirmar**, son los días que esa persona
efectivamente marcó — ese es el número que sirve para cotizar cada día.

### Actualización a las columnas Tipo y Días

Si ya tenías las hojas con las columnas originales, no hay que hacer nada a
mano: `getSheet` detecta que faltan y **las agrega al final** la próxima vez
que se escriba algo. Las filas anteriores quedan con esas celdas vacías.

Para llenar una fila vieja de *Invitados*, usa el botón **Tipo** del invitado
en `admin.html`: dispara la acción `tipo` y escribe ambas columnas.

---

## Tipos de invitación

| Tipo | Nombre | 🥐 Brunch (19 mar) | 💍 Boda (20 mar) | 🌤️ Postcelebración (21 mar) |
|------|--------|:---:|:---:|:---:|
| **1** | Completa | ✅ | ✅ | ✅ |
| **2** | Boda y post | — | ✅ | ✅ |

El tipo viaja en el link como `&tipo=1` o `&tipo=2`, junto a `?nombre=` y `&cupos=`.
