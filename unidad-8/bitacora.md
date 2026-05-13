# Unidad 8

## Bitácora de proceso de aprendizaje

### ACTIVIDAD 01

<details>

<summary> Diagrama inicial de la arquitectura</summary>

### DIAGRAMA CON PLANTUML

<img width="684" height="915" alt="image" src="https://github.com/user-attachments/assets/93981390-6c38-4fda-ab7d-5dbacb50a12a" />

</details>

**Adapters utilizados:**
 
| Adapter | Fuente | Protocolo |
|---|---|---|
| `MicrobitBinaryAdapter` | micro:bit | Serial binario con checksum, 115200 baud |
| `StrudelAdapter` | Strudel | WebSocket en puerto 8080 |
| `OpenStageControlAdapter` | Open Stage Control | UDP en puerto 9000 |
 
---
 
**Contrato de mensajes de cada fuente**
 
**micro:bit → bridgeServer**
 
Paquete binario de 8 bytes enviado a 30 Hz:
 
```
[0xAA] [x_hi] [x_lo] [y_hi] [y_lo] [btnA] [btnB] [checksum]
```
 
- `x`, `y`: Int16 big-endian (acelerómetro, rango ~-1024 a 1024)
- `btnA`, `btnB`: `0x00` o `0x01`
- `checksum`: `(x_hi + x_lo + y_hi + y_lo + btnA + btnB) & 0xFF`
- 
Mensaje normalizado que llega al frontend:
```json
{ "type": "microbit", "x": 234, "y": -512, "btnA": false, "btnB": true, "t": 1715000000000 }
```
 
**Strudel → bridgeServer**
 
```json
{ "type": "strudel", "timestamp": 1715000000100, "payload": { "sound": "bd", "family": "bd", "delta": 0.5 } }
```
 
**Open Stage Control → bridgeServer**
 
```json
{ "type": "osc", "payload": { "address": "/rgb_bd", "args": [255, 0, 80] } }
```
 
---
 
**Pruebas técnicas básicas de integración**
 
**Prueba 1 — micro:bit llega al frontend**
- Se inició `bridgeServer.js --device full --serialPort COM4`
- Se abrió el sketch en el browser y se pulsó Connect
- Se verificó en consola del browser que aparecían mensajes `BRIDGE STATUS: connected serial open COM4 @115200`
- Confirmado: los datos del micro:bit llegan al `updateLogic`
**Prueba 2 — Strudel activa eventos visuales**
- Se ejecutó el patrón `bd*4 sd hh*8 oh` en Strudel con `.osc()`
- Se verificó que las formas aparecían sincronizadas con el audio
- Confirmado: bombo, caja, hi-hat y open hi-hat disparan sus animaciones
**Prueba 3 — OSC modifica parámetros persistentes**
- Se abrió Open Stage Control con el archivo `CONTROLADORES8.json`
- Se movió el widget `rgb_bd` y se verificó que el color del bombo cambiaba en tiempo real
- Confirmado: los mensajes OSC llegan a `updateControls`
---
 
**Errores encontrados y soluciones**
 
**Error 1:** `Cannot find module 'C:\Users\nikki\bridgeServer.js'`
- **Causa:** El comando se ejecutó desde `~` en lugar de la carpeta del proyecto.
- **Solución:** Navegar con `cd` a la carpeta del proyecto antes de ejecutar `node bridgeServer.js`.
**Error 2:** `connect failed: Opening COM3: File not found`
- **Causa:** El puerto serial estaba configurado como COM3 pero el micro:bit estaba en COM4.
- **Solución:** Agregar `--serialPort COM4` al comando de inicio.
**Error 3:** El serial se desconectaba inmediatamente (`serial closed`)
- **Causa:** El `main.py` no estaba flasheado en el micro:bit. Sin programa MicroPython el puerto abre y cierra solo.
- **Solución:** Flashear el `main.py` 
**Error 4:** El modo `full` usaba `MicrobitAdapter2` en lugar de `MicrobitBinaryAdapter`
- **Causa:** El `bridgeServer.js` original instanciaba `MicrobitAdapter2` en el bloque `full`.
- **Solución:** Modificar el bloque `full` en `createAdapter()` para instanciar `MicrobitBinaryAdapter`.
---

### ACTIVIDAD 2

**Concepto de la obra**
 
La obra es una **performance audiovisual industrial** inspirada en *Closer* de Nine Inch Nails. La propuesta explora la tensión entre el control mecánico del cuerpo (el micro:bit como extensión física) y la generación algorítmica del sonido (Strudel como motor musical). El performer manipula en tiempo real tanto el espacio visual como los parámetros sonoros, creando una experiencia de live coding audiovisual de carácter oscuro e industrial.
 
---
 
**Rol de cada fuente dentro de la obra**
 
| Fuente | Rol en la obra | Justificación |
|---|---|---|
| **micro:bit** | Control físico gestual: botón A agranda las formas, botón B las achica. La inclinación en el eje Y hace aparecer un triángulo que representa la guitarra. | El gesto corporal del performer es visible en pantalla, haciendo la relación entre cuerpo e imagen perceptible para la audiencia. |
| **Strudel** | Motor musical y disparador de eventos visuales. Cada sonido del patrón activa una forma visual distinta (bombo = círculo, caja = rayos, hi-hat = cuadrado, open hi-hat = rayo). | La imagen responde directamente al ritmo musical, creando sincronía audiovisual. |
| **Open Stage Control** | Control paramétrico persistente: color de las familias de sonido.
---
 
**Decisiones visuales**
 
- **Bombo (`bd`):** Círculo que crece desde el centro — metáfora del impacto físico del kick.
- **Caja (`sd`):** 12 rayos que explotan desde el centro — energía dispersa, industrial.
- **Hi-hat (`hh`):** Cuadrado pequeño en posición aleatoria — textura mecánica.
- **Open hi-hat (`oh`):** Zigzag de rayo que cae — el elemento más icónico de Closer.
- **Triángulo de guitarra:** Aparece según la inclinación del micro:bit en Y. Cián al inclinar hacia adelante, magenta hacia atrás. Representa la capa melódica de forma gestual.

**Decisiones musicales**
 
El patrón está inspirado en *Closer* (90 BPM = 0.375 cps):
- Bombo en cada beat (obsesivo, mecánico)
- Caja en 2 y 4
- Open hi-hat espaciado (el elemento más reconocible)
- Guitarra en arpegio Bb menor siguiendo el bombo
- Bajo muy grave en tónica pedal
- Sintetizador industrial con filtro oscilante

  
**Decisiones performáticas**
 
- El performer tiene en la mano izquierda el micro:bit y en la derecha el mouse para controlar OSC.
- El live coding de Strudel se ejecuta en pantalla secundaria.

---  

 
**Cambios entre iteración ingenieril e iteración estética**
 
| Cambio | Razón |
|---|---|
| Se reemplazó el acelerómetro como control de scale por los botones A/B | El control con botones es más preciso y expresivo en vivo |
| Se agregó el triángulo de guitarra controlado por eje Y del acelerómetro | Agregar una visual que responda al gesto corporal de forma continua |
| Se amplió el patrón Strudel con bajo, sintetizador y percusión metálica | Acercar más la propuesta sonora al referente musical |
| Se agregó la familia `oh` con su color y forma propios | El open hi-hat es el sonido más característico de Closer |
 
---
 
<details>

<summary> Evidencias de ensayo </summary>

### Evidencias de ensayo + fotos

- Se realizó un ensayo completo con todos los sistemas corriendo simultáneamente.
- Se verificó que el micro:bit en COM4 mantiene la conexión estable durante 5 minutos.
- Se verificó que los tres adapters conviven sin bloqueos ni desconexiones.
- Se ensayó el cambio de colores en OSC mientras Strudel corría sin interrupciones.

<img width="1244" height="656" alt="Captura de pantalla 2026-05-08 094226" src="https://github.com/user-attachments/assets/575461ea-f5d1-4295-937e-2a602b607bc6" />

<img width="393" height="476" alt="Captura de pantalla 2026-05-08 160202" src="https://github.com/user-attachments/assets/7778dc58-c6bd-4633-81df-6e8b5779fb36" />

<img width="547" height="232" alt="Captura de pantalla 2026-05-08 155600" src="https://github.com/user-attachments/assets/a6e955aa-452a-4635-8ff9-83169e48d400" />

<img width="1254" height="686" alt="Captura de pantalla 2026-05-13 130855" src="https://github.com/user-attachments/assets/46c70857-b05a-4ac1-bba3-e070d654cd0e" />



</details>
 
---


## Bitácora de aplicación 

### ACTIVIDAD 2.5 - CÓDIGOS

<details>

<summary> MICROBIT </summary>

### CÓDIGO PARA EL MICROBIT

```py
  from microbit import *
import utime
import struct

uart.init(baudrate=115200)

def send_packet(x, y, btnA, btnB):
x = max(-32768, min(32767, x))
y = max(-32768, min(32767, y))
x_bytes = struct.pack(">h", x)
y_bytes = struct.pack(">h", y)
a = 1 if btnA else 0
b = 1 if btnB else 0
chk = (x_bytes[0] + x_bytes[1] + y_bytes[0] + y_bytes[1] + a + b) & 0xFF
packet = bytes([0xAA, x_bytes[0], x_bytes[1], y_bytes[0], y_bytes[1], a, b, chk])
uart.write(packet)

while True:
x = accelerometer.get_x()
y = accelerometer.get_y()
a = button_a.is_pressed()
b = button_b.is_pressed()
send_packet(x, y, a, b)
utime.sleep_ms(33)
```

</details>

---

<details>

<summary> STRUDEL </summary>

### CÓDIGO PARA EL STRUDEL

```js
setcps(0.375)
const drums = s("bd bd bd bd").bank("tr909")
const snare = s("~ sd ~ sd").bank("tr909")
const hats  = s("~ ~ oh ~ ~ ~ oh ~").bank("tr909")
const tex   = s("~ ~ ~ hh ~ hh ~ ~").bank("tr909")

// Guitarra: sigue el bombo en Bb menor
const gtr = note("bf2 bf2 f2 ef2").s("sawtooth")
.lpf(800).resonance(8)
.gain(0.4)

// Bajo: muy grave y distorsionado, tónica pedal
const bass = note("bf1 ~ bf1 ~").s("sawtooth")
.lpf(300).resonance(12)
.gain(0.6)

// Sintetizador industrial: textura de fondo con ruido filtrado
const synth = note("bf2").s("square")
.lpf("200 400 200 600")
.gain(0.15)

// Percusión metálica: golpes industriales entre bombos
const metal = s("~ mt ~ mt").bank("tr909")
.gain(0.5)

$: stack(
drums.gain(0.9),
snare.gain(0.75),
hats.gain(0.6),
tex.gain(0.35),
metal,
gtr,
bass,
synth,
stack(drums, snare, hats, tex, metal).osc()
```

</details>

---

<details>

<summary> SKETCH </summary>

### CÓDIGO PARA EL SKETCH

```js
const EVENTS = {
  CONNECT: "CONNECT",
  DISCONNECT: "DISCONNECT",
  DATA: "DATA",
};

class PainterTask extends FSMTask {
  constructor() {
    super();

    this.scheduledEvents = [];
    this.activeAnimations = [];
    this.latencyCorrection = 0;

    // _rawScale: valor crudo de botones A/B, usado para lerp suave
    this._rawScale = 1;
    // _rawY: eje Y del acelerómetro, controla el triángulo de guitarra
    this._rawY = 0;

    this.controls = {
      colors: {
        bd:    [255, 0,   80 ],
        sd:    [0,   200, 255],
        cp:    [100, 220, 255],
        hh:    [255, 255, 0  ],
        oh:    [255, 140, 0  ],
        other: [200, 200, 200]
      },
      scale: 1,
      tiltY: 0,         // -1.0 (atrás) → 1.0 (adelante), controla triángulo
      opacity: 1.0,     // 0.0 → 1.0, controlado por slider OSC /opacity
      showHatLayer: true
    };

    this.transitionTo(this.estado_esperando);
  }

  estado_esperando = (ev) => {
    if (ev.type === "ENTRY") {
      cursor();
      console.log("Waiting for bridge connection...");
    } else if (ev.type === EVENTS.CONNECT) {
      this.transitionTo(this.estado_corriendo);
    }
  };

  estado_corriendo = (ev) => {
    if (ev.type === "ENTRY") {
      noCursor();
      background(0);
      console.log("Bridge ready");
      this.scheduledEvents = [];
      this.activeAnimations = [];
    }

    else if (ev.type === EVENTS.DISCONNECT) {
      this.transitionTo(this.estado_esperando);
    }

    else if (ev.type === EVENTS.DATA) {
      this.updateLogic(ev.payload);
    }

    else if (ev.type === "EXIT") {
      cursor();
    }
  };

  updateLogic(msg) {
    // MICRO:BIT:
    // La inclinación del acelerómetro controla el tamaño global de todas
    // las visuales de Strudel. Se guarda en _rawScale y se aplica con lerp
    // en draw() para que el cambio sea suave y no brusco.
    if (msg.type === "microbit") {
      // Botón A: agranda las formas de Strudel continuamente.
      // Botón B: achica las formas de Strudel continuamente.
      const STEP = 0.04;
      if (msg.btnA) this._rawScale = min(this._rawScale + STEP, 2.5);
      if (msg.btnB) this._rawScale = max(this._rawScale - STEP, 0.3);

      // Eje Y del acelerómetro: inclinar adelante/atrás mueve el triángulo.
      // msg.y va de ~-1024 (adelante) a ~1024 (atrás).
      // Lo normalizamos a -1.0 → 1.0.
      this._rawY = constrain(msg.y / 1024, -1, 1);
      return;
    }

    // STRUDEL:
    // Eventos musicales temporizados, se guardan en cola.
    if (msg.type === "strudel") {
      this.scheduledEvents.push({
        timestamp: msg.timestamp,
        sound:     msg.payload.sound,
        family:    msg.payload.family,
        delta:     msg.payload.delta
      });
      this.scheduledEvents.sort((a, b) => a.timestamp - b.timestamp);
      return;
    }

    // OSC:
    // Controles persistentes de Open Stage Control.
    if (msg.type === "osc") {
      this.updateControls(msg.payload);
      return;
    }
  }

  updateControls(payload) {
    const address = payload.address;
    const args    = payload.args;

    // Los widgets rgb de OSC envían tres valores en rango 0-255.
    function parseColor(args) {
      return [
        Number(args[0] ?? 255),
        Number(args[1] ?? 255),
        Number(args[2] ?? 255)
      ];
    }

    if (address === "/rgb_bd")    { this.controls.colors.bd    = parseColor(args); }
    if (address === "/rgb_sd")    { this.controls.colors.sd    = parseColor(args); }
    if (address === "/rgb_hh")    { this.controls.colors.hh    = parseColor(args); }
    if (address === "/rgb_oh")    { this.controls.colors.oh    = parseColor(args); }
    if (address === "/rgb_cp")    { this.controls.colors.cp    = parseColor(args); }
    if (address === "/rgb_other") { this.controls.colors.other = parseColor(args); }

    // /scale_1 desde OSC sobreescribe el acelerómetro si se usa.
    if (address === "/scale_1") {
      this._rawScale = Number(args[0] ?? 1);
    }

    if (address === "/toggle_hats") {
      this.controls.showHatLayer = Boolean(args[0]);
    }

    if (address === "/opacity") {
      this.controls.opacity = constrain(Number(args[0] ?? 1.0), 0.0, 1.0);
    }
  }

  processScheduledEvents() {
    const now = Date.now() + this.latencyCorrection;

    while (
      this.scheduledEvents.length > 0 &&
      now >= this.scheduledEvents[0].timestamp
    ) {
      const ev = this.scheduledEvents.shift();

      this.activeAnimations.push({
        startTime: ev.timestamp,
        duration:  ev.delta * 1000,
        sound:     ev.sound,
        family:    ev.family,
        x:         random(width  * 0.2, width  * 0.8),
        y:         random(height * 0.2, height * 0.8),
        angle:     random(TWO_PI),
        color:     getColorForFamily(ev.family, this.controls)
      });
    }
  }

  cleanupAnimations() {
    const now = Date.now() + this.latencyCorrection;

    for (let i = this.activeAnimations.length - 1; i >= 0; i--) {
      const anim    = this.activeAnimations[i];
      const elapsed = now - anim.startTime;
      const progress = elapsed / anim.duration;

      if (progress > 1) {
        this.activeAnimations.splice(i, 1);
      }
    }
  }
}

let painter;
let bridge;
let connectBtn;
const renderer = new Map();

function setup() {
  createCanvas(windowWidth, windowHeight);
  rectMode(CENTER);
  noStroke();
  background(0);

  painter = new PainterTask();
  bridge  = new BridgeClient();

  bridge.onConnect(() => {
    connectBtn.html("Disconnect");
    painter.postEvent({ type: EVENTS.CONNECT });
  });

  bridge.onDisconnect(() => {
    connectBtn.html("Connect");
    painter.postEvent({ type: EVENTS.DISCONNECT });
  });

  bridge.onStatus((s) => {
    console.log("BRIDGE STATUS:", s.state, s.detail ?? "");
  });

  bridge.onData((data) => {
    painter.postEvent({
      type:    EVENTS.DATA,
      payload: data
    });
  });

  connectBtn = createButton("Connect");
  connectBtn.position(10, 10);

  connectBtn.mousePressed(() => {
    if (bridge.isOpen) bridge.close();
    else               bridge.open();
  });

  renderer.set(painter.estado_corriendo, drawRunning);
}

function draw() {
  background(0, 30);

  // Suavizado del scale proveniente del acelerómetro:
  // lerp en cada frame da una transición fluida sin saltos.
  painter.controls.scale = lerp(painter.controls.scale, painter._rawScale, 0.08);
  painter.controls.tiltY = lerp(painter.controls.tiltY, painter._rawY, 0.06);

  painter.update();

  if (painter.state === painter.estado_corriendo) {
    painter.processScheduledEvents();
    painter.cleanupAnimations();
  }

  renderer.get(painter.state)?.();
}

function drawRunning() {
  const now = Date.now() + painter.latencyCorrection;

  for (const anim of painter.activeAnimations) {
    const elapsed  = now - anim.startTime;
    const progress = elapsed / anim.duration;

    if (progress >= 0 && progress <= 1) {
      drawAnimation(anim, progress);
    }
  }

  // Triángulo de guitarra controlado por eje Y del acelerómetro
  drawGuitarTriangle();
}

function drawAnimation(anim, p) {
  push();

  if ((anim.family === "hh" || anim.family === "oh") && !painter.controls.showHatLayer) {
    pop();
    return;
  }

  switch (anim.family) {
    case "bd":  drawKick(anim, p);      break;
    case "sd":  drawRays(anim, p);      break;
    case "cp":  drawClap(anim, p);      break;
    case "hh":  drawHat(anim, p);       break;
    case "oh":  drawLightning(anim, p); break;   // ← nuevo
    default:    drawDefault(anim, p);   break;
  }

  pop();
}

// BOMBO: círculo que crece desde el centro
function drawKick(anim, p) {
  const s     = painter.controls.scale;
  const op    = painter.controls.opacity;
  const d     = lerp(100, 600, p) * s;
  const alpha = lerp(255, 0, p) * op;

  fill(anim.color[0], anim.color[1], anim.color[2], alpha);
  circle(width / 2, height / 2, d);
}

// CAJA: rayos que salen desde el centro
function drawRays(anim, p) {
  const s      = painter.controls.scale;
  const op     = painter.controls.opacity;
  const alpha  = lerp(255, 0, p) * op;
  const rayLen = lerp(0, 400, p) * s;
  const numRays = 12;

  stroke(anim.color[0], anim.color[1], anim.color[2], alpha);
  strokeWeight(lerp(4, 1, p) * s);
  noFill();

  translate(width / 2, height / 2);
  for (let i = 0; i < numRays; i++) {
    const angle = (TWO_PI / numRays) * i + anim.angle;
    const x2    = cos(angle) * rayLen;
    const y2    = sin(angle) * rayLen;
    line(0, 0, x2, y2);
  }
}

// CLAP: dos rectángulos laterales que se juntan
function drawClap(anim, p) {
  const s      = painter.controls.scale;
  const op     = painter.controls.opacity;
  const alpha  = lerp(255, 0, p) * op;
  const offset = lerp(120, 0, p) * s;

  fill(anim.color[0], anim.color[1], anim.color[2], alpha);
  rect(width / 2 - offset, height / 2, 80 * s, 160 * s);
  rect(width / 2 + offset, height / 2, 80 * s, 160 * s);
}

// HI-HAT: cuadrado pequeño en posición aleatoria
function drawHat(anim, p) {
  const s    = painter.controls.scale;
  const op   = painter.controls.opacity;
  const sz   = lerp(40, 0, p) * s;
  const alpha = lerp(255, 0, p) * op;

  fill(anim.color[0], anim.color[1], anim.color[2], alpha);
  rect(anim.x, anim.y, sz, sz);
}

// OPEN HI-HAT: zigzag de rayo que cae desde arriba y se desvanece.
// El rayo tiene SEGMENTS segmentos con desplazamiento lateral aleatorio
// por seed para que cada nota tenga su propia forma pero sea determinista.
function drawLightning(anim, p) {
  const s      = painter.controls.scale;
  const op     = painter.controls.opacity;
  const alpha  = lerp(220, 0, p) * op;
  const SEGS   = 8;
  const totalH = lerp(300, 80, p) * s;   // el rayo se contrae al final
  const spread = lerp(60, 20, p)  * s;   // los desvíos laterales también

  stroke(anim.color[0], anim.color[1], anim.color[2], alpha);
  strokeWeight(lerp(3, 1, p) * s);
  noFill();

  // Reproducir la misma forma aleatoria usando el seed del ángulo del evento
  randomSeed(anim.angle * 10000);

  const cx = anim.x;
  const ty = anim.y - totalH / 2;   // punto más alto
  const by = anim.y + totalH / 2;   // punto más bajo

  beginShape();
  vertex(cx, ty);
  for (let i = 1; i < SEGS; i++) {
    const t  = i / SEGS;
    const y  = lerp(ty, by, t);
    const dx = random(-spread, spread);
    vertex(cx + dx, y);
  }
  vertex(cx, by);
  endShape();

  // Destello en el punto de impacto (parte inferior) al inicio
  if (p < 0.25) {
    const flashAlpha = lerp(180, 0, p / 0.25);
    const flashR     = lerp(0, 60, p / 0.25) * s;
    noStroke();
    fill(anim.color[0], anim.color[1], anim.color[2], flashAlpha);
    circle(cx, by, flashR);
  }
}

// OTHER: rectángulo rotatorio
function drawDefault(anim, p) {
  const s     = painter.controls.scale;
  const op    = painter.controls.opacity;
  const size  = lerp(100, 0, p) * s;
  const angle = p * TWO_PI;
  const alpha = lerp(255, 0, p) * op;

  translate(anim.x, anim.y);
  rotate(angle);

  stroke(anim.color[0], anim.color[1], anim.color[2], alpha);
  strokeWeight(2);
  noFill();
  rect(0, 0, size, size);
}

// GUITARRA: triángulo que aparece cuando se inclina el micro:bit en Y.
// tiltY > 0  (inclinado hacia adelante) → triángulo crece hacia arriba.
// tiltY < 0  (inclinado hacia atrás)   → triángulo crece hacia abajo.
// Cuando está plano (tiltY ≈ 0) el triángulo es invisible.
function drawGuitarTriangle() {
  const t     = painter.controls.tiltY;  // -1.0 → 1.0
  const abst  = abs(t);

  // Solo dibuja si hay inclinación significativa
  if (abst < 0.05) return;

  const alpha  = map(abst, 0.05, 1.0, 0, 180);
  const size   = map(abst, 0.05, 1.0, 20, 300);
  const dir    = t > 0 ? -1 : 1;  // adelante = apunta arriba, atrás = abajo

  // Color: adelante = cian, atrás = magenta
  const r = t > 0 ? 0   : 255;
  const g = t > 0 ? 220 : 0;
  const b = t > 0 ? 255 : 200;

  push();
  noStroke();
  fill(r, g, b, alpha);
  translate(width / 2, height / 2);

  // Triángulo isósceles apuntando según dirección
  beginShape();
  vertex(0, dir * size);           // punta
  vertex(-size * 0.6, -dir * size * 0.5);  // base izquierda
  vertex( size * 0.6, -dir * size * 0.5);  // base derecha
  endShape(CLOSE);
  pop();
}

function getColorForFamily(family, controls) {
  return controls.colors[family] || controls.colors.other;
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```

</details>

---

<details>

<summary> BRIDGESERVER </summary>

### CÓDIGO PARA EL BRIDGESERVER

```js
//   Uso:
//     node bridgeServer.js --device sim --wsPort 8081 --hz 30
//     node bridgeServer.js --device microbit --wsPort 8081 --serialPort COM5 --baud 115200
//     node bridgeServer.js --device full --wsPort 8081 --strudelPort 8080 --oscPort 9000 --verbose

//   WS contract:
//    * bridge To client:
//        {type:"status", state:"ready|connected|disconnected|error", detail:"..."}
//        {type:"microbit", x:int, y:int, btnA:bool, btnB:bool, t:ms}
//    * client To bridge:
//        {cmd:"connect"} | {cmd:"disconnect"}
//        {cmd:"setSimHz", hz:30}
//        {cmd:"setLed", x:2, y:3, value:9}

const { WebSocketServer } = require("ws");
const { SerialPort } = require("serialport");
const SimAdapter = require("./adapters/SimAdapter");
const MicrobitAsciiAdapter = require("./adapters/MicrobitASCIIAdapter");
const MicrobitBinaryAdapter = require("./adapters/MicrobitBinaryAdapter");
const MicrobitAdapter2 = require("./adapters/MicrobitAdapter2");
const log = {
  info: (...args) => console.log(`[${new Date().toISOString()}] [INFO]`, ...args),
  warn: (...args) => console.warn(`[${new Date().toISOString()}] [WARN]`, ...args),
  error: (...args) => console.error(`[${new Date().toISOString()}] [ERROR]`, ...args)
};
const StrudelAdapter = require("./adapters/StrudelAdapter");
const OpenStageControlAdapter = require("./adapters/OpenStageControlAdapter");

function getArg(name, def = null) {
  const i = process.argv.indexOf(`--${name}`);
  if (i >= 0 && i + 1 < process.argv.length) return process.argv[i + 1];
  return def;
}

function hasFlag(name) {
  return process.argv.includes(`--${name}`);
}

function nowMs() { return Date.now(); }

function safeJsonParse(s) {
  try {
    return JSON.parse(s);
  } catch (e) {
    log.warn("Failed to parse JSON: ", s, e);
    return null;
  }
}

function broadcast(wss, obj) {
  const text = JSON.stringify(obj);
  for (const client of wss.clients) {
    if (client.readyState === 1) client.send(text);
  }
}

function status(wss, state, detail = "") {
  broadcast(wss, { type: "status", state, detail, t: nowMs() });
}

const DEVICE = (getArg("device", "sim") || "sim").toLowerCase();
const WS_PORT = parseInt(getArg("wsPort", "8081"), 10);
const SERIAL_PATH = getArg("serialPort", null);
const BAUD = parseInt(getArg("baud", "115200"), 10);
const SIM_HZ = parseInt(getArg("hz", "30"), 10);
const VERBOSE = hasFlag("verbose");

async function findMicrobitPort() {
  const ports = await SerialPort.list();
  log.info(`[DEBUG] Ports found: ${JSON.stringify(ports.map(p => ({ path: p.path, vendorId: p.vendorId })))}`);
  const microbit = ports.find(p =>
    p.vendorId && parseInt(p.vendorId, 16) === 0x0D28
  );
  return microbit?.path ?? null;
}

async function createAdapter() {
  if (DEVICE === "microbit") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);
    return new MicrobitAsciiAdapter({ path, baud: BAUD, verbose: VERBOSE });
  }

  if (DEVICE === "microbit-two") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);
    return new MicrobitAdapter2({ path, baud: BAUD, verbose: VERBOSE });
  }

  if (DEVICE === "microbit-bin") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);
    return new MicrobitBinaryAdapter({ path, baud: BAUD });
  }

  if (DEVICE === "strudel") {
    const port = parseInt(getArg("strudelPort", "8080"), 10);
    return new StrudelAdapter({ port, verbose: VERBOSE });
  }

  if (DEVICE === "strudel-osc") {
    const strudelPort = parseInt(getArg("strudelPort", "8080"), 10);
    const oscPort = parseInt(getArg("oscPort", "9000"), 10);
    return [
      new StrudelAdapter({ port: strudelPort, verbose: VERBOSE }),
      new OpenStageControlAdapter({ port: oscPort, verbose: VERBOSE })
    ];
  }

  if (DEVICE === "full") {
    log.info(`[DEBUG] Entering full mode, SERIAL_PATH=${SERIAL_PATH}`);
    const strudelPort = parseInt(getArg("strudelPort", "8080"), 10);
    const oscPort = parseInt(getArg("oscPort", "9000"), 10);
    const path = SERIAL_PATH ?? await findMicrobitPort();
    log.info(`[DEBUG] path found=${path}`);
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);
    return [
      new MicrobitBinaryAdapter({ path, baud: BAUD, verbose: VERBOSE }),
      new StrudelAdapter({ port: strudelPort, verbose: VERBOSE }),
      new OpenStageControlAdapter({ port: oscPort, verbose: VERBOSE })
    ];
  }

  return new SimAdapter({ hz: SIM_HZ });
}

async function main() {
  const wss = new WebSocketServer({ port: WS_PORT });
  log.info(`WS listening on ws://127.0.0.1:${WS_PORT} device=${DEVICE}`);

  const adapters = [].concat(await createAdapter());

  for (const adapter of adapters) {
    adapter.onConnected = (detail) => {
      log.info(`[ADAPTER] Device Connected: ${detail}`);
      status(wss, "connected", detail);
    };

    adapter.onDisconnected = (detail) => {
      log.warn(`[ADAPTER] Device Disconnected: ${detail}`);
      status(wss, "disconnected", detail);
    };

    adapter.onError = (detail) => {
      log.error(`[ADAPTER] Device Error: ${detail}`);
      status(wss, "error", detail);
    };

    adapter.onData = (d) => {
      if (d.type === "strudel" || d.type === "osc") {
        broadcast(wss, d);
        return;
      }

      // Log solo cuando hay botón presionado para no saturar la terminal
      if (d.btnA || d.btnB) {
        log.info(`[MICROBIT] btnA=${d.btnA} btnB=${d.btnB} x=${d.x} y=${d.y}`);
      }

      broadcast(wss, {
        type: "microbit",
        x: d.x,
        y: d.y,
        btnA: !!d.btnA,
        btnB: !!d.btnB,
        t: nowMs(),
      });
    };
  }

  status(wss, "ready", `bridge up (${DEVICE})`);

  wss.on("connection", (ws, req) => {
    log.info(`[NETWORK] Remote Client connected from ${req.socket.remoteAddress}. Total clients: ${wss.clients.size}`);

    const anyConnected = adapters.some(a => a.connected);
    const state = anyConnected ? "connected" : "ready";
    const detail = anyConnected
      ? adapters.map(a => a.getConnectionDetail()).join(" + ")
      : `bridge (${DEVICE})`;

    ws.send(JSON.stringify({ type: "status", state, detail, t: nowMs() }));

    ws.on("message", async (raw) => {
      const msg = safeJsonParse(raw.toString("utf8"));
      if (!msg) return;

      if (msg.cmd === "connect") {
        log.info(`[NETWORK] Client requested adapters connect`);
        try {
          for (const adapter of adapters) {
            if (!adapter.connected) {
              await adapter.connect();
            }
          }
        } catch (e) {
          const detail = `connect failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }

      if (msg.cmd === "disconnect") {
        log.info(`[NETWORK] Client requested adapters disconnect`);
        try {
          for (const adapter of adapters) {
            if (adapter.connected) {
              await adapter.disconnect();
            }
          }
          status(wss, "disconnected", "all adapters disconnected");
        } catch (e) {
          const detail = `disconnect failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }

      if (msg.cmd === "setSimHz") {
        const simAdapter = adapters.find(a => a instanceof SimAdapter);
        if (simAdapter) {
          log.info(`Setting Sim Hz to ${msg.hz}`);
          await simAdapter.handleCommand(msg);
          status(wss, "connected", `sim hz=${simAdapter.hz}`);
        }
        return;
      }

      if (msg.cmd === "setLed") {
        try {
          for (const adapter of adapters) {
            await adapter.handleCommand?.(msg);
          }
        } catch (e) {
          const detail = `command failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }
    });

    ws.on("close", () => {
      log.info(`[NETWORK] Remote Client disconnected. Total clients left: ${wss.clients.size}`);
      if (wss.clients.size === 0) {
        log.info("[HW-POLICY] No more remote clients. Auto-disconnecting adapter device to free resources...");
        for (const adapter of adapters) {
          adapter.disconnect();
        }
      }
    });
  });

  if (DEVICE === "sim") {
    for (const adapter of adapters) {
      await adapter.connect();
    }
  }
}

main().catch((e) => {
  log.error("Fatal:", e);
  process.exit(1);
});
```

</details>

---

<details>

<summary> BRIDGECLIENT </summary>

### CÓDIGO PARA EL BRIDGECLIENT

```js
class BridgeClient {
  constructor(url = "ws://127.0.0.1:8081") {
    this._url = url;
    this._ws = null;
    this._isOpen = false;

    this._onData = null;
    this._onConnect = null;
    this._onDisconnect = null;
    this._onStatus = null;
  }

  get isOpen() {
    return this._isOpen;
  }

  onData(callback) { this._onData = callback; }
  onConnect(callback) { this._onConnect = callback; }
  onDisconnect(callback) { this._onDisconnect = callback; }
  onStatus(callback) { this._onStatus = callback; }

  open() {
    if (this._ws && this._ws.readyState === WebSocket.OPEN) {
      if (!this._isOpen) this.send({ cmd: "connect" });
      return;
    }

    if (this._ws) {
      this.close();
    }

    this._ws = new WebSocket(this._url);

    this._ws.onopen = () => {
      this.send({ cmd: "connect" });
    };

    this._ws.onmessage = (event) => {
      // Esperamos JSON normalizado desde el bridge
      let msg;
      try {
        msg = JSON.parse(event.data);
      } catch (e) {
        console.warn("WS message is not JSON:", event.data);
        return;
      }

      // Convención mínima:
      // - {type:"status", state:"...", detail:"..."}
      // - {type:"microbit", x:..., y:..., btnA:..., btnB:...}
      if (msg.type === "status") {
        this._onStatus?.(msg);

        if (msg.state === "connected") {
          this._isOpen = true;
          this._onConnect?.();
        }

        if (msg.state === "disconnected" || msg.state === "error" || msg.state === "ready") {
          this._isOpen = false; 
          this._onDisconnect?.();
          if (msg.state === "error") {
            this._ws?.close();
            this._ws = null;
          }          
        }
        return;
      }

      if (msg.type === "microbit"  || msg.type === "strudel" || msg.type === "osc") {
        // payload ya normalizado
        this._onData?.(msg);
        return;
      }
    };

    this._ws.onerror = (err) => {
      console.warn("WS error:", err);
    };

    this._ws.onclose = () => {
      this._handleDisconnect();
    };
  }

  close() {
    if (!this._ws || this._ws.readyState !== WebSocket.OPEN) return;

    try {
      this.send({ cmd: "disconnect" });
      this._isOpen = false;
    } catch (e) {
      console.warn("Failed to send disconnect command:", e);
    }
  }

  send(obj) {
    if (!this._ws || this._ws.readyState !== WebSocket.OPEN) return;
    this._ws.send(JSON.stringify(obj));
  }

  _handleDisconnect() {
    this._isOpen = false;
    this._ws = null;
    this._onDisconnect?.();
  }
}
```

</details>

---

<details>

<summary>GITBASH</summary>

### CODIGO PARA ABRIR EL SERVIDOR

```
node bridgeServer.js --device full --wsPort 8081 --strudelPort 8080 --oscPort 9000 --serialPort COM4
```

</details>


## Bitácora de reflexión

### ACTIVIDAD 03

---

<details>

<summary> DIAGRAMA 1 </summary>

### Diagrama completo del flujo de datos

<img width="2032" height="914" alt="image" src="https://github.com/user-attachments/assets/e86fd7fa-b873-4006-80c9-bfa658cdb719" />

</details>

--- 

<details>

<summary> DIAGRAMA 2 </summary>

### Recorrido completo de la arquitectura

<img width="921" height="1064" alt="image" src="https://github.com/user-attachments/assets/7daee5dd-8651-4ece-a6c8-237fead08531" />

**Explicación del recorrido:**
 
1. **Adapter:** Cada fuente tiene su propio adapter que recibe los datos crudos, los normaliza a un contrato claro y los entrega a `bridgeServer.js`. El `MicrobitBinaryAdapter` parsea el protocolo binario con checksum. El `StrudelAdapter` recibe eventos musicales por WebSocket. El `OpenStageControlAdapter` recibe mensajes OSC por UDP.
2. **bridgeServer.js:** Actúa como concentrador y distribuidor. Recibe los datos de todos los adapters y los reenvía por WebSocket a todos los clientes conectados. No contiene lógica de dominio del proyecto.
3. **bridgeClient.js:** Mantiene la conexión WebSocket con el servidor. Cuando llega un mensaje, inspecciona `msg.type` y llama al callback `onData`, que en el sketch dispara un evento hacia la FSM.
4. **FSMTask (PainterTask):** Máquina de estados finita. Tiene dos estados: `estado_esperando` (antes de conectar) y `estado_corriendo` (con el bridge activo). Cuando está corriendo, delega los eventos de tipo `DATA` a `updateLogic`.
5. **updateLogic():** Actualiza el estado del sistema según el tipo de mensaje. Para `microbit` actualiza `_rawScale` y `_rawY`. Para `strudel` agrega eventos a `scheduledEvents`. Para `osc` actualiza colores y opacidad en `controls`.
6. **drawRunning():** Lee el estado ya calculado (`controls`, `activeAnimations`) y dibuja. No toma decisiones de lógica, solo visualiza.


</details>

---


**Justificación de la propuesta estética y performática**
 
La obra toma *Closer* de Nine Inch Nails como referente no para reproducirla sino para habitar su estética industrial, oscura y mecánica. La elección de este referente justifica todas las decisiones del sistema:
 
- El **bombo en cada beat** crea la obsesión mecánica característica de la canción. Visualmente se traduce en círculos que explotan desde el centro del canvas en cada pulso.
- El **rayo del open hi-hat** es el elemento más reconocible de Closer. Tener una forma visual propia (el zigzag de `drawLightning`) para este sonido crea una correspondencia directa entre el icono sonoro y el icono visual de la obra.
- El **triángulo de guitarra** aparece solo cuando el performer inclina el micro:bit, haciendo que la capa melódica sea literalmente un gesto corporal. Si el performer no se mueve, la guitarra desaparece.
---

**PREGUNTAS**


¿Cómo entra cada fuente al sistema?

- **micro:bit:** Envía paquetes binarios de 8 bytes por puerto serial a 115200 baud a 30 Hz. El `MicrobitBinaryAdapter` los lee, verifica el checksum y los normaliza a `{x, y, btnA, btnB}`.
- **Strudel:** Envía eventos musicales temporizados por WebSocket al puerto 8080. El `StrudelAdapter` los recibe y los normaliza a `{type:"strudel", timestamp, payload:{sound, family, delta}}`.
- **Open Stage Control:** Envía mensajes OSC por UDP al puerto 9000. El `OpenStageControlAdapter` los recibe y los normaliza a `{type:"osc", payload:{address, args}}`.

---

¿Qué hace cada Adapter?

Cada adapter tiene tres responsabilidades: recibir los datos crudos de su fuente, normalizarlos a un contrato claro, y entregarlos a `bridgeServer.js`. El `MicrobitBinaryAdapter` además verifica la integridad del paquete con el checksum antes de emitir los datos. Además de su respectivo rol performativo ya mencionado

---

¿Qué papel cumple `bridgeServer.js`?

Actúa como concentrador y distribuidor. Recibe los datos normalizados de los tres adapters y los reenvía por WebSocket a todos los clientes conectados. No contiene lógica de dominio del proyecto — no decide qué hacer con los datos, solo los transporta.

---

¿Cómo se conectan `bridgeClient.js`, `FSMTask`, `updateLogic` y `drawRunning`?

1. `bridgeClient.js` mantiene la conexión WebSocket con el servidor, inspecciona `msg.type` y dispara un evento hacia la FSM con `postEvent({type: DATA, payload})`.
2. `FSMTask` recibe ese evento y, si está en `estado_corriendo`, lo delega a `updateLogic`.
3. `updateLogic` actualiza el estado del sistema: para `microbit` actualiza `_rawScale` y `_rawY`, para `strudel` agrega a `scheduledEvents`, para `osc` actualiza `controls.colors`
4. `drawRunning` lee el estado ya calculado (`controls`, `activeAnimations`) y dibuja. No toma ninguna decisión de lógica.

---

¿Qué rol cumple cada fuente dentro de la obra?

- **micro:bit:** Control físico gestual. Botón A agranda todas las formas, botón B las achica. La inclinación en eje Y hace aparecer el triángulo de guitarra — la capa melódica es literalmente un gesto corporal.
- **Strudel:** Motor musical y disparador de eventos visuales. Cada familia de sonido activa una forma distinta sincronizada con el audio: bombo = círculo, caja = rayos, hi-hat = cuadrado, open hi-hat = rayo.
- **Open Stage Control:** Control paramétrico persistente. El performer modifica el color de cada familia de sonido.

---

¿Por qué el sistema conserva la arquitectura del curso?

Porque respeta estrictamente el flujo `Adapter → bridgeServer → bridgeClient → FSMTask → updateLogic → drawRunning`:

- Cada fuente tiene su propio adapter independiente.
- `bridgeServer.js` solo transporta, no resuelve lógica.
- `bridgeClient.js` solo recibe y dispara eventos, no dibuja.
- `FSMTask` coordina el sistema sin parsear mensajes de red.
- `updateLogic` traduce mensajes a estado, no dibuja.
- `drawRunning` solo dibuja usando estado ya calculado, no interpreta mensajes.

Ningún paso se salta ni se mezcla con otro.

