---
name: live-copilot
description: Co-piloto EN VIVO para una conversación. Recibe transcripción en tiempo real (por ejemplo del Elffuss Translator vía pipe, o pegada/tecleada) y va montando un DOCUMENTO vivo con señales, argumentos y el siguiente paso. Úsala para "copiloto de ventas", "asistente de reunión en vivo", "argumentos de venta en tiempo real", "notas de entrevista", "soporte guiado". Corre 100% en el navegador, sin librerías, y escucha el canal de pipe de Elffuss.
---

# live-copilot — un documento vivo que se llena mientras hablas

Panel doble: a la izquierda la **transcripción en vivo** (llega turno a turno, con hablante), a la
derecha un **documento de trabajo** que se rellena solo: *señales* detectadas, *argumentos* a usar,
*objeciones* y el *siguiente paso*. Es genérico: cambia las secciones y el mapa de disparadores según
el caso (ventas, reunión, entrevista, soporte).

**Cómo entra el texto (pipe de Elffuss):** la app escucha `BroadcastChannel('elffuss-pipe')` y acepta
mensajes `{kind:'transcript', text, speaker}` — así el **Translator** (u otra app Elffuss del MISMO
origen) le manda cada intervención sin copiar/pegar. También hay una caja para pegar/teclear y
`window.__pipe(text, speaker)` para pruebas. El pipe SOLO funciona entre apps del mismo origen (mismo
`elffuss.com`), que es donde comparten caché, motor y permisos.

**Diseño:** sobrio y editorial — sin emojis, sin frameworks, sin gradientes de colores. Tipografía de
sistema con titulares serif, filetes finos, etiquetas en versalitas. Mantén ese registro si lo adaptas.

**Papeles:** el demo trae un motor de reglas por palabras clave para que funcione **solo**; cuando lo
mueve el agente de Claw, es el modelo quien rellena las secciones llamando a `window.copilot.set(...)`
con criterio real. Deja intactos el pipe y el layout; adapta secciones y disparadores.

## App COMPLETA y probada — cópiala y adapta solo lo indicado

```html
<!doctype html>
<html lang="es"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>Co-piloto en vivo</title>
<style>
  :root{--bg:#0b0d10;--panel:#101317;--line:#20252c;--fg:#e8eaed;--mut:#8b929c;--accent:#b79268;--sig:#7f9cc0;--ok:#7faa7f;--warn:#c9a15a}
  *{box-sizing:border-box;margin:0}
  body{font-family:-apple-system,system-ui,"Segoe UI",sans-serif;background:var(--bg);color:var(--fg);height:100vh;display:flex;flex-direction:column;font-size:15px}
  header{padding:14px 22px;border-bottom:1px solid var(--line);display:flex;align-items:baseline;gap:14px}
  header h1{font-family:Georgia,"Times New Roman",serif;font-weight:600;font-size:1.15rem;letter-spacing:.2px}
  header .sub{font-size:.72rem;color:var(--mut);text-transform:uppercase;letter-spacing:.14em}
  header .state{margin-left:auto;font-size:.68rem;color:var(--mut);text-transform:uppercase;letter-spacing:.12em;border:1px solid var(--line);border-radius:2px;padding:4px 10px}
  header .state.on{color:var(--ok);border-color:#2c3a2c}
  .wrap{flex:1;display:grid;grid-template-columns:1fr 1fr;min-height:0}
  @media(max-width:820px){.wrap{grid-template-columns:1fr}}
  .col{display:flex;flex-direction:column;min-height:0;border-right:1px solid var(--line)}
  .col:last-child{border-right:none}
  .col h2{font-size:.66rem;text-transform:uppercase;letter-spacing:.16em;color:var(--mut);padding:14px 20px 10px}
  .feed{flex:1;overflow:auto;padding:0 20px 16px;display:flex;flex-direction:column;gap:2px}
  .turn{padding:11px 0;border-bottom:1px solid var(--line)}
  .turn .spk{font-size:.64rem;text-transform:uppercase;letter-spacing:.12em;color:var(--sig);margin-bottom:4px}
  .turn.them .spk{color:var(--accent)}
  .turn .tx{line-height:1.5}
  .inbar{display:flex;gap:0;border-top:1px solid var(--line)}
  .inbar input{flex:1;background:transparent;border:none;color:var(--fg);padding:14px 20px;font:inherit;outline:none}
  .inbar button{background:transparent;border:none;border-left:1px solid var(--line);color:var(--accent);padding:0 20px;font-weight:600;cursor:pointer;letter-spacing:.02em}
  .doc{flex:1;overflow:auto;padding:6px 20px 20px}
  .sec{padding:16px 0;border-bottom:1px solid var(--line)}
  .sec:last-child{border-bottom:none}
  .sec h3{font-size:.66rem;text-transform:uppercase;letter-spacing:.16em;color:var(--mut);margin-bottom:10px}
  .sec ul{list-style:none;display:flex;flex-direction:column;gap:8px}
  .sec li{line-height:1.45;padding-left:14px;position:relative}
  .sec li::before{content:"";position:absolute;left:0;top:.62em;width:5px;height:5px;border-radius:50%;background:var(--accent)}
  .sec.sig li::before{background:var(--sig)} .sec.arg li::before{background:var(--ok)} .sec.obj li::before{background:var(--warn)}
  .sec.next .big{font-family:Georgia,serif;font-size:1.05rem;line-height:1.4;color:var(--fg)}
  .empty{color:var(--mut);font-style:italic;padding-left:0}
  .empty::before{display:none}
</style></head>
<body>
<header><h1>Co-piloto en vivo</h1><span class="sub">Documento de venta</span><span class="state" id="state">Esperando</span></header>
<div class="wrap">
  <div class="col">
    <h2>Conversación</h2>
    <div class="feed" id="feed"></div>
    <div class="inbar"><input id="inp" placeholder="Escribe una intervención, o llega por pipe"><button id="send">Añadir</button></div>
  </div>
  <div class="col">
    <h2>Documento en vivo</h2>
    <div class="doc" id="doc">
      <div class="sec sig"><h3>Señales</h3><ul id="sig"><li class="empty">Aún nada</li></ul></div>
      <div class="sec arg"><h3>Argumentos a usar</h3><ul id="arg"><li class="empty">Aún nada</li></ul></div>
      <div class="sec obj"><h3>Objeciones</h3><ul id="obj"><li class="empty">Aún nada</li></ul></div>
      <div class="sec next"><h3>Siguiente paso</h3><div class="big" id="next">Escucha y detecta la necesidad.</div></div>
    </div>
  </div>
</div>
<script>
const $=id=>document.getElementById(id); const feed=$('feed');
function addTurn(text, speaker='S1'){ const them=/S2|B|cliente|prospect/i.test(speaker);
  const d=document.createElement('div'); d.className='turn'+(them?' them':''); d.innerHTML='<div class="spk"></div><div class="tx"></div>';
  d.querySelector('.spk').textContent=speaker; d.querySelector('.tx').textContent=text; feed.appendChild(d); feed.scrollTop=feed.scrollHeight; rules(text, them); }
// API para el agente/pipe: rellena secciones con criterio real
const copilot={ set(sec, items){ const ul=$(sec); if(!ul){ if(sec==='next') $('next').textContent=items; return; }
    ul.innerHTML=''; (items||[]).forEach(t=>{ const li=document.createElement('li'); li.textContent=t; ul.appendChild(li); });
    if(!ul.children.length){ const li=document.createElement('li'); li.className='empty'; li.textContent='—'; ul.appendChild(li);} },
  add(sec, item){ const ul=$(sec); if(!ul) return; const e=ul.querySelector('.empty'); if(e)e.remove(); const li=document.createElement('li'); li.textContent=item; ul.appendChild(li); },
  next(t){ $('next').textContent=t; } };
window.copilot=copilot;
// motor de reglas DEMO (para que funcione solo; el agente lo sustituye con criterio)
const TRIG=[
  {re:/(caro|precio|coste|presupuesto)/i, obj:'Sensibilidad al precio', arg:'Enfoca en el ROI y el coste marginal cero: corre en su GPU, sin cuota.'},
  {re:/(seguridad|privacidad|datos|nube|gdpr)/i, sig:'Le preocupa la privacidad', arg:'Nada sale de su máquina: es local por diseño.'},
  {re:/(lento|rápido|rendimiento|espera)/i, sig:'Le importa el rendimiento', arg:'WebGPU: el modelo en su tarjeta, sin ida y vuelta al servidor.'},
  {re:/(comparar|competencia|alternativa|otro)/i, sig:'Está comparando opciones', arg:'Diferencial: open-source Apache-2.0, sin instalar, sin cuenta.'},
  {re:/(interesa|me gusta|adelante|contratar|comprar|perfecto)/i, sig:'Señal de compra', next:'Propón el siguiente paso concreto: demo, prueba o cierre.'},
];
function rules(text){ for(const t of TRIG){ if(t.re.test(text)){ if(t.sig)copilot.add('sig',t.sig); if(t.arg)copilot.add('arg',t.arg); if(t.obj)copilot.add('obj',t.obj); if(t.next)copilot.next(t.next); } } }
$('send').onclick=()=>{ const v=$('inp').value.trim(); if(v){ addTurn(v,'Tú'); $('inp').value=''; } };
$('inp').addEventListener('keydown',e=>{ if(e.key==='Enter') $('send').click(); });
// PIPE de Elffuss (mismo origen)
try{ const bc=new BroadcastChannel('elffuss-pipe'); bc.onmessage=e=>{ const m=e.data||{}; if(m.kind==='transcript'&&m.text){ $('state').textContent='En vivo'; $('state').className='state on'; addTurn(m.text, m.speaker||'Cliente'); } }; }catch{}
window.__pipe=(text,speaker)=>addTurn(text, speaker||'Cliente');
</script>
</body></html>
```
