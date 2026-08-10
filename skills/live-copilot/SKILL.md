---
name: live-copilot
description: Co-piloto EN VIVO para una conversación. Recibe transcripción en tiempo real (por ejemplo del Elffuss Translator vía pipe, o pegada/tecleada) y va montando un DOCUMENTO vivo con señales, argumentos y el siguiente paso. Úsala para "copiloto de ventas", "asistente de reunión en vivo", "argumentos de venta en tiempo real", "notas de entrevista", "soporte guiado". Corre 100% en el navegador, sin librerías, y escucha el canal de pipe de Elffuss.
---

# live-copilot — un documento vivo que se llena mientras hablas

Panel doble: a la izquierda la **transcripción en vivo** (llega turno a turno, con hablante), a la
derecha un **documento de trabajo** que se va rellenando: *señales* detectadas, *argumentos* a usar,
*objeciones* y el *siguiente paso*. Es genérico: cambia las secciones y el mapa de disparadores según
el caso (ventas, reunión, entrevista, soporte).

**Cómo entra el texto (pipe de Elffuss):** la app escucha `BroadcastChannel('elffuss-pipe')` y acepta
mensajes `{kind:'transcript', text, speaker}` — así el **Translator** (u otra app Elffuss del MISMO
origen) le manda cada intervención sin copiar/pegar. También hay una caja para pegar/teclear manual y
`window.__pipe(text, speaker)` para pruebas. El pipe SOLO funciona entre apps del mismo origen
(mismo `elffuss.utopiaia.com`), que es donde comparten caché, motor y permisos.

**Papeles:** el demo trae un motor de reglas por palabras clave para que funcione **solo**; cuando lo
mueve el agente de Claw, es el modelo quien rellena las secciones (llamando a `window.copilot.set(...)`)
con criterio real. Deja intactos el pipe y el layout; adapta secciones y disparadores.

## App COMPLETA y probada — cópiala y adapta solo lo indicado

```html
<!doctype html>
<html lang="es"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>Co-piloto en vivo</title>
<style>
  :root{--bg:#0d1117;--panel:#161b22;--line:#232a34;--fg:#e6edf3;--mut:#8b949e;--a:#7c5cff;--b:#ff5cd6;--ok:#3fb950;--warn:#f0b429}
  *{box-sizing:border-box;margin:0} body{font-family:system-ui,sans-serif;background:#0d1117;color:var(--fg);height:100vh;display:flex;flex-direction:column}
  header{padding:12px 16px;border-bottom:1px solid var(--line);display:flex;gap:10px;align-items:center;font-weight:700}
  header .live{margin-left:auto;font-size:.72rem;color:var(--ok);border:1px solid #234;border-radius:999px;padding:3px 10px}
  header .live.off{color:var(--mut)}
  .wrap{flex:1;display:grid;grid-template-columns:1fr 1fr;gap:0;min-height:0}
  @media(max-width:760px){.wrap{grid-template-columns:1fr}}
  .col{display:flex;flex-direction:column;min-height:0;border-right:1px solid var(--line)}
  .col h2{font-size:.72rem;text-transform:uppercase;letter-spacing:.4px;color:var(--mut);padding:10px 14px;border-bottom:1px solid var(--line)}
  .feed{flex:1;overflow:auto;padding:12px 14px;display:flex;flex-direction:column;gap:8px}
  .turn{padding:8px 11px;border-radius:11px;background:#1b2540;border:1px solid #2b3866;font-size:.92rem;line-height:1.4}
  .turn .spk{font-size:.66rem;font-weight:700;color:#9db0ff;margin-bottom:2px}
  .turn.them{background:#3a1b34;border-color:#66284f} .turn.them .spk{color:#ff9de0}
  .inbar{display:flex;gap:8px;padding:10px 14px;border-top:1px solid var(--line)}
  .inbar input{flex:1;background:#0b0f14;border:1px solid var(--line);color:var(--fg);border-radius:9px;padding:8px;font:inherit}
  .inbar button{background:linear-gradient(135deg,var(--a),var(--b));border:none;color:#fff;border-radius:9px;padding:8px 12px;font-weight:600;cursor:pointer}
  .doc{flex:1;overflow:auto;padding:14px}
  .sec{background:var(--panel);border:1px solid var(--line);border-radius:12px;padding:12px 14px;margin-bottom:12px}
  .sec h3{font-size:.78rem;color:#c9d3de;margin-bottom:8px;display:flex;align-items:center;gap:6px}
  .sec ul{list-style:none;display:flex;flex-direction:column;gap:6px}
  .sec li{font-size:.9rem;padding:7px 10px;border-radius:9px;background:#0f1520;border:1px solid var(--line);line-height:1.35}
  .sec.arg li{border-left:3px solid var(--ok)} .sec.obj li{border-left:3px solid var(--warn)} .sec.sig li{border-left:3px solid var(--a)}
  .next{background:linear-gradient(135deg,rgba(124,92,255,.16),rgba(255,92,214,.16));border:1px solid #33406e}
  .next .big{font-size:1rem;font-weight:700}
  .empty{color:var(--mut);font-size:.82rem;font-style:italic}
</style></head>
<body>
<header>🎧 Co-piloto en vivo <span class="live off" id="live">● esperando pipe</span></header>
<div class="wrap">
  <div class="col">
    <h2>Conversación en vivo</h2>
    <div class="feed" id="feed"></div>
    <div class="inbar"><input id="inp" placeholder="Escribe una intervención (o llega por pipe)…"><button id="send">Añadir</button></div>
  </div>
  <div class="col" style="border-right:none">
    <h2>Documento de venta (vivo)</h2>
    <div class="doc" id="doc">
      <div class="sec sig"><h3>🔎 Señales</h3><ul id="sig"><li class="empty">Aún nada…</li></ul></div>
      <div class="sec arg"><h3>💬 Argumentos a usar</h3><ul id="arg"><li class="empty">Aún nada…</li></ul></div>
      <div class="sec obj"><h3>⚠️ Objeciones</h3><ul id="obj"><li class="empty">Aún nada…</li></ul></div>
      <div class="sec next"><h3>➡️ Siguiente paso</h3><div class="big" id="next">Escucha y detecta la necesidad.</div></div>
    </div>
  </div>
</div>
<script>
const $=id=>document.getElementById(id);
const feed=$('feed'); let sinceAgent=Date.now();
function addTurn(text, speaker='S1'){ const them=/S2|B|cliente|prospect/i.test(speaker);
  const d=document.createElement('div'); d.className='turn'+(them?' them':''); d.innerHTML=`<div class="spk"></div><div class="tx"></div>`;
  d.querySelector('.spk').textContent=speaker; d.querySelector('.tx').textContent=text; feed.appendChild(d); feed.scrollTop=feed.scrollHeight;
  rules(text, them); }
// API para el agente/pipe: rellena secciones con criterio real
const copilot={ set(sec, items){ const ul=$(sec); if(!ul) { if(sec==='next'){ $('next').textContent=items; } return; }
    ul.innerHTML=''; (items||[]).forEach(t=>{ const li=document.createElement('li'); li.textContent=t; ul.appendChild(li); }); if(!ul.children.length){ const li=document.createElement('li'); li.className='empty'; li.textContent='—'; ul.appendChild(li);} },
  add(sec, item){ const ul=$(sec); if(!ul) return; const e=ul.querySelector('.empty'); if(e)e.remove(); const li=document.createElement('li'); li.textContent=item; ul.appendChild(li); },
  next(t){ $('next').textContent=t; } };
window.copilot=copilot;
// motor de reglas DEMO (para que funcione solo; el agente lo sustituye con criterio)
const TRIG=[
  {re:/(caro|precio|coste|presupuesto)/i, obj:'Sensibilidad al precio', arg:'Enfoca en ROI y coste marginal ~0 (corre en su GPU, sin cuota).'},
  {re:/(seguridad|privacidad|datos|nube|gdpr)/i, sig:'Le preocupa la privacidad', arg:'Recuerda: nada sale de su máquina, es local por diseño.'},
  {re:/(lento|rápido|rendimiento|espera)/i, sig:'Le importa el rendimiento', arg:'WebGPU: modelo en su tarjeta, sin ida y vuelta al servidor.'},
  {re:/(comparar|competencia|alternativa|otro)/i, sig:'Está comparando opciones', arg:'Diferénciate: open-source Apache-2.0, sin instalar, sin cuenta.'},
  {re:/(interesa|me gusta|genial|adelante|contratar|comprar)/i, sig:'👍 Señal de compra', next:'Propón el siguiente paso concreto (demo/prueba/cierre).'},
];
function rules(text, them){ for(const t of TRIG){ if(t.re.test(text)){ if(t.sig)copilot.add('sig',t.sig); if(t.arg)copilot.add('arg',t.arg); if(t.obj)copilot.add('obj',t.obj); if(t.next)copilot.next(t.next); } } }
// entrada manual
$('send').onclick=()=>{ const v=$('inp').value.trim(); if(v){ addTurn(v, 'S1 (tú)'); $('inp').value=''; } };
$('inp').addEventListener('keydown',e=>{ if(e.key==='Enter') $('send').click(); });
// PIPE de Elffuss (mismo origen): el Translator manda {kind:'transcript', text, speaker}
try{ const bc=new BroadcastChannel('elffuss-pipe'); bc.onmessage=e=>{ const m=e.data||{}; if(m.kind==='transcript'&&m.text){ $('live').textContent='● pipe conectado'; $('live').className='live'; addTurn(m.text, m.speaker||'S2 (cliente)'); } }; }catch{}
window.__pipe=(text,speaker)=>addTurn(text, speaker||'S2 (cliente)'); // test
</script>
</body></html>
```
