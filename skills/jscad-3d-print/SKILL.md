---
name: jscad-3d-print
description: Genera un objeto 3D IMPRIMIBLE (STL) desde una descripción, dentro del navegador. Úsala cuando el usuario pida "hazme una pieza/soporte/caja/adaptador para imprimir", "un modelo 3D", "un STL", o describa un objeto físico a fabricar. Crea una app con vista 3D, parámetros (sliders) y botón de descargar STL — todo local con JSCAD, sin Python ni servidor.
---

# jscad-3d-print — de una idea a un STL imprimible, en el navegador

JSCAD (`@jscad/modeling`) es CAD paramétrico que corre en el navegador y exporta STL listo
para laminar (Snapmaker, Bambu, Prusa…). No necesita Python ni backend: se importa por CDN y la
app resultante genera el modelo y lo descarga. Esta es la vía browser-native para 3D imprimible.

## Qué generar

Una app HTML autocontenida que: (1) construye el sólido con JSCAD a partir de parámetros,
(2) lo previsualiza en 3D con Three.js, (3) tiene un botón **Descargar STL**. Copia el patrón de
abajo y cambia SOLO la función `build()` para el objeto que pida el usuario.

## Patrón que FUNCIONA (cámbialo lo mínimo)

- Imports por CDN (esm.sh): `three`, `@jscad/modeling`, `@jscad/stl-serializer`.
- Geometría con JSCAD: `primitives` (cuboid, cylinder, sphere, roundedRectangle…),
  `booleans` (union, subtract, intersect), `transforms` (translate, rotate, scale),
  `extrusions` (extrudeLinear).
- Previa: convierte el geom3 de JSCAD a `THREE.BufferGeometry` recorriendo
  `geometries.geom3.toPolygons(geom)` y triangulando cada polígono en abanico.
- Export: `stlSerializer.serialize({binary:true}, solid)` → `Blob(type:'model/stl')` → descargar.

```html
<script type="module">
import * as THREE from 'https://esm.sh/three@0.160';
import { primitives, booleans, transforms, extrusions, geometries } from 'https://esm.sh/@jscad/modeling@2';
import stlSerializer from 'https://esm.sh/@jscad/stl-serializer@2';

// 1) EL MODELO (todo en mm). Cambia SOLO esto para el objeto pedido:
function build({ w, h, r, hole }) {
  let solid = extrusions.extrudeLinear({ height: h },
    primitives.roundedRectangle({ size: [w, w], roundRadius: Math.min(r, w/2 - 0.1) }));
  if (hole > 0) {
    const bore = transforms.translate([0,0,h/2], primitives.cylinder({ height: h+2, radius: hole/2, segments: 48 }));
    solid = booleans.subtract(solid, bore);
  }
  return solid;
}

// 2) JSCAD → Three (para la previa)
function toThree(geom){ const ps=[]; for(const p of geometries.geom3.toPolygons(geom)){const v=p.vertices;
  for(let i=2;i<v.length;i++) ps.push(...v[0],...v[i-1],...v[i]);}
  const g=new THREE.BufferGeometry(); g.setAttribute('position',new THREE.Float32BufferAttribute(ps,3));
  g.computeVertexNormals(); g.center(); return g; }

// 3) descargar STL
function downloadSTL(solid){ const data=stlSerializer.serialize({binary:true}, solid);
  const a=document.createElement('a'); a.href=URL.createObjectURL(new Blob(data,{type:'model/stl'}));
  a.download='modelo.stl'; a.click(); }
</script>
```

El `template.html` de esta carpeta es una app COMPLETA y probada (caja paramétrica con agujero,
sliders, previa 3D y descarga STL, 548 triángulos). Úsalo como base.

## Reglas para que salga imprimible (FDM)

- **Todo en mm**, base plana sobre Z=0, sin voladizos que necesiten soporte salvo que se pida.
- **Paredes ≥ 1.2 mm** (3 líneas de 0.4) y detalles ≥ 1 mm o desaparecen al imprimir.
- Cierra siempre el sólido (union de partes que se tocan); un STL con huecos no lamina bien.
- `segments: 32-64` en cilindros/esferas: menos se ve facetado, más pesa el STL.

## Lo que ya falló

- `@jscad/modeling` NO tiene `cone`: usa `cylinderElliptic({height, startRadius:[r,r], endRadius:[0.2,0.2]})`.
- El default de `dtype`/binario da igual para STL; para descargar usa `{binary:true}` (más ligero).
- No metas el HTML en un string JSON: Claw ya crea la app desde un bloque ```html — deja que el
  visor la renderice y que el usuario pulse Descargar STL.
