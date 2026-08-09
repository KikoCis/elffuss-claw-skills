---
name: doc-to-excel
description: Coge datos de un documento (texto, CSV, tabla pegada o un .xlsx/.csv de una carpeta autorizada) y genera un Excel DESCARGABLE con fórmulas y gráficas, dentro del navegador. Úsala cuando el usuario pida "pasa esto a Excel", "hazme una hoja con estos datos", "un Excel con fórmulas y gráficas", "convierte este documento en una tabla", o quiera analizar/visualizar datos y llevárselos a Excel.
---

# doc-to-excel — de un documento a un Excel con fórmulas y gráficas

Todo local, en el navegador: se leen los datos, se construye un `.xlsx` con **SheetJS** (incluye
**fórmulas** de verdad, ej. `SUM`, `AVERAGE`), se dibujan **gráficas** con **Chart.js**, y se ofrece
descargar el Excel. Nada sale de la máquina.

## Flujo

1. **Conseguir los datos.** Si el usuario los pega, parséalos. Si están en una carpeta autorizada,
   léelos con las tools de Claw (`fs.read` / lectura de `.xlsx` con SheetJS). Normaliza a una matriz
   `[[cabeceras...], [fila...], ...]`.
2. **Construir el Excel** con SheetJS: datos + una fila/columna de **fórmulas** (totales, medias).
3. **Gráficas** con Chart.js en la propia app (barras/líneas/tarta según los datos).
4. **Descargar** el `.xlsx`.

## Patrón que FUNCIONA (probado)

```html
<script type="module">
import * as XLSX from 'https://esm.sh/xlsx@0.18.5';
import Chart from 'https://esm.sh/chart.js@4/auto';

const rows = [['Mes','Ventas'],['Ene',100],['Feb',150],['Mar',210]];

// 1) hoja con datos + FÓRMULA real (SUM). Ojo: la fórmula va sin '=' y con {t:'n', f:'...'}
const data = [...rows, ['Total', { t:'n', f:`SUM(B2:B${rows.length})` }]];
const ws = XLSX.utils.aoa_to_sheet(data);
const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, 'Datos');

// 2) descargar el .xlsx
function downloadXLSX(){ const buf = XLSX.write(wb,{bookType:'xlsx',type:'array'});
  const a=document.createElement('a'); a.href=URL.createObjectURL(new Blob([buf],{type:'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'}));
  a.download='datos.xlsx'; a.click(); }

// 3) gráfica en la app
new Chart(canvas, { type:'bar',
  data:{ labels: rows.slice(1).map(r=>r[0]), datasets:[{ label: rows[0][1], data: rows.slice(1).map(r=>r[1]) }] } });
</script>
```

La app debe tener: la **tabla** visible, la **gráfica**, y un botón **Descargar Excel**.

## Lo honesto sobre las gráficas

SheetJS (community) **escribe datos y fórmulas**, pero **no incrusta gráficas nativas** dentro del
`.xlsx` de forma fiable en el navegador. Así que:
- El **.xlsx** que se descarga lleva **datos + fórmulas** (recalculan solos al abrirlo en Excel).
- Las **gráficas** se muestran en la app (Chart.js) y, si el usuario las quiere en el fichero, se
  exportan como **imagen** (`chart.toBase64Image()`) para pegar, o se le dice que en Excel es
  «Insertar gráfico» sobre los datos ya puestos. No prometas gráficos nativos incrustados.

## Lo que ya falló

- La fórmula en SheetJS va **sin `=`** y como celda `{ t:'n', f:'SUM(B2:B4)' }`; con `=` delante no cala.
- Números como strings ('100') rompen las fórmulas: convierte a `Number` antes de meterlos.
- Para leer un `.xlsx` de entrada usa `XLSX.read(arrayBuffer)` + `XLSX.utils.sheet_to_json`.
