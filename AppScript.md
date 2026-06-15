# Google Apps Script — Gabriel & Jatzive

Pega este código en **Extensiones → Apps Script** dentro de tu Google Sheet.  
Reemplaza todo el código existente y vuelve a implementar como *Aplicación web*.

---

## Código

```javascript
function getSheet(ss, nombre, encabezados) {
  var s = ss.getSheetByName(nombre);
  if (!s) {
    s = ss.insertSheet(nombre);
    s.appendRow(encabezados);
    s.getRange(1, 1, 1, encabezados.length).setFontWeight('bold');
  }
  return s;
}

function doPost(e) {
  try {
    var data  = JSON.parse(e.postData.contents);
    var ss    = SpreadsheetApp.getActiveSpreadsheet();
    var fecha = Utilities.formatDate(new Date(), 'America/Mexico_City', 'dd/MM/yyyy HH:mm');

    if (data.action === 'add') {
      var s = getSheet(ss, 'Invitados', ['Nombre', 'Lugares', 'Confirmado', 'Fecha', 'ID']);
      s.appendRow([data.nombre, data.cupos, 'No ⬜', fecha, data.id]);

    } else if (data.action === 'confirm') {
      var s    = getSheet(ss, 'Invitados', ['Nombre', 'Lugares', 'Confirmado', 'Fecha', 'ID']);
      var rows = s.getDataRange().getValues();
      for (var i = 1; i < rows.length; i++) {
        if (String(rows[i][4]) === String(data.id)) {
          s.getRange(i + 1, 3).setValue(data.confirmado ? 'Sí ✅' : 'No ⬜');
          break;
        }
      }

    } else if (data.action === 'delete') {
      var s    = getSheet(ss, 'Invitados', ['Nombre', 'Lugares', 'Confirmado', 'Fecha', 'ID']);
      var rows = s.getDataRange().getValues();
      for (var i = rows.length - 1; i >= 1; i--) {
        if (String(rows[i][4]) === String(data.id)) {
          s.deleteRow(i + 1);
          break;
        }
      }

    } else if (data.action === 'asistente') {
      var hoja = data.porConfirmar ? 'Por Confirmar' : 'Confirmados';
      var s    = getSheet(ss, hoja, ['Grupo', 'Nombre Completo', 'Estado', 'Fecha']);
      s.appendRow([
        data.grupo,
        data.nombreCompleto,
        data.porConfirmar ? 'Por confirmar ⏳' : 'Confirmado ✅',
        fecha
      ]);
    }

    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: err.message }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  return ContentService
    .createTextOutput(JSON.stringify({ ok: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## Hojas que se crean automáticamente

| Hoja | Columnas | Quién escribe |
|------|----------|---------------|
| **Invitados** | Nombre · Lugares · Confirmado · Fecha · ID | Admin (admin.html) |
| **Confirmados** | Grupo · Nombre Completo · Estado · Fecha | Invitado (desde su invitación) |
| **Por Confirmar** | Grupo · Nombre Completo · Estado · Fecha | Invitado (marcó "por confirmar") |

---

## Cómo re-implementar

1. Abre tu Google Sheet → **Extensiones → Apps Script**
2. Borra todo el código anterior y pega el código de arriba
3. Haz clic en **Implementar → Administrar implementaciones**
4. En tu implementación existente haz clic en el lápiz ✏️ → **Nueva versión** → **Implementar**
5. La URL no cambia, no necesitas actualizar nada en el sitio
