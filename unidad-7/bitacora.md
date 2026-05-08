# Unidad 7

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

Así como hicimos en la unidad ¿anterior, lo primero es crear el Adapter para OSC

<details>
<summary><b>OpenStageControlAdapter.js</b></summary>

```jsx
const BaseAdapter = require("./BaseAdapter");
const osc = require("osc");

class OpenStageControlAdapter extends BaseAdapter {
  constructor({ port = 9000, verbose = false } = {}) {
    super();
    this.port = port;
    this.verbose = verbose;
    this.udpPort = null;
  }

  async connect() {
    if (this.connected) return;

    this.udpPort = new osc.UDPPort({
      localAddress: "0.0.0.0",
      localPort: this.port,
      metadata: false
    });

    this.udpPort.on("message", (oscMsg) => {
      const normalized = this._normalize(oscMsg);
      if (!normalized) return;

      if (this.verbose) {
        console.log("[OpenStageControlAdapter]", normalized);
      }

      this.onData?.(normalized);
    });

    this.udpPort.on("error", (err) => {
      this.onError?.(`OSC error: ${err.message || err}`);
    });

    this.udpPort.open();

    this.connected = true;
    this.onConnected?.(`Open Stage Control escuchando OSC UDP en puerto ${this.port}`);
  }

  async disconnect() {
    if (!this.connected) return;

    if (this.udpPort) {
      this.udpPort.close();
      this.udpPort = null;
    }

    this.connected = false;
    this.onDisconnected?.("Open Stage Control adapter detenido");
  }

  getConnectionDetail() {
    return `Open Stage Control OSC UDP port ${this.port}`;
  }

  _normalize(oscMsg) {
    if (!oscMsg || !oscMsg.address) return null;

    return {
      type: "osc",
      payload: {
        address: oscMsg.address,
        args: oscMsg.args ?? []
      }
    };
  }
}

module.exports = OpenStageControlAdapter;
```

</details>

---

Después editamos un par de cosas en el BridgeServer.js

<details>
<summary><b>BridgeServer.js</b></summary><br>

Creamos el Adapter

```jsx
const OpenStageControlAdapter = require("./adapters/OpenStageControlAdapter");
```

en `createAdapter()` añadir:

```jsx
if (DEVICE === "strudel-osc") {
    const strudelPort = parseInt(getArg("strudelPort", "8080"), 10);
    const oscPort = parseInt(getArg("oscPort", "9000"), 10);

    return [
      new StrudelAdapter({ port: strudelPort, verbose: VERBOSE }),
      new OpenStageControlAdapter({ port: oscPort, verbose: VERBOSE })
    ];
  }
```

Al principios de `main()` cambiar:

```jsx
const adapter = await createAdapter();
```

por

```jsx
 const adapters = [].concat(await createAdapter());
```
para que sea iterable y devuelve un arreglo

y el siguiente bloque de codigo que contiene el onConnected, onDisconnected, onError y onData, reemplazar el código por:

```jsx
for (const adapter of adapters) {
  adapter.onConnected = (detail) => {
    log.info(`[ADAPTER] Connected: ${detail}`);
    status(wss, "connected", detail);
  };

  adapter.onDisconnected = (detail) => {
    log.warn(`[ADAPTER] Disconnected: ${detail}`);
    status(wss, "disconnected", detail);
  };

  adapter.onError = (detail) => {
    log.error(`[ADAPTER] Error: ${detail}`);
    status(wss, "error", detail);
  };

  adapter.onData = (d) => {
    if (d.type === "strudel" || d.type === "osc") {
      broadcast(wss, d);
      return;
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
```

Dentro de `wss.on("connection")` hay que cambiar la conexión del adapter para que permita conectarse al strudel y al OSC al mismo tiempo, por lo que, se cambia el `cons state = adapter...` por:

```jsx
const anyConnected = adapters.some(a => a.connected);
const state = anyConnected ? "connected" : "ready";

const detail = anyConnected
  ? adapters.map(a => a.getConnectionDetail()).join(" + ")
  : `bridge (${DEVICE})`;
```

Cambiar el comando `connect` (en la linea 190) y reemplazar el contenido de `if (msg.cmd === "connect") {` por

```jsx
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
```

hacer lo mismo con `disconnect`

```jsx
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
```

en `if (msg.cmd === "setSimHz" && adapters instanceof SimAdapter)`, quedaria

```jsx
if (msg.cmd === "setSimHz") {
  const simAdapter = adapters.find(a => a instanceof SimAdapter);

  if (simAdapter) {
    log.info(`Setting Sim Hz to ${msg.hz}`);
    await simAdapter.handleCommand(msg);
    status(wss, "connected", `sim hz=${simAdapter.hz}`);
  }
  return;
}
```

y en `if (msg.cmd === "setLed")` seria

```jsx
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
```

y en `if (DEVICE === "sim")` usar

```jsx
if (DEVICE === "sim") {
  for (const adapter of adapters) {
    await adapter.connect();
  }
}
```

</details>

</details>

---
Ahora pasamos al bridgeClient

<details>
<summary><b>bridgeClient.js</b></summary><br>

En bridgeClient.js, en la linea 70 que cambiamos en el otro ejercicio, cambiar de nuevo a:

```jsx
if (msg.type === "microbit" || msg.type === "strudel") {
  this._onData?.(msg);
  return;
}
```

por

```jsx
if (msg.type === "microbit" || msg.type === "strudel" || msg.type === "osc") {
  this._onData?.(msg);
  return;
}
```

</details>

---

Por ultimo, haremos el sketch

<details>
<summary><b>sketch.js</b></summary>

```jsx
const EVENTS = {
  CONNECT: "CONNECT",
  DISCONNECT: "DISCONNECT",
  DATA: "DATA",
};

class PainterTask extends FSMTask {
  constructor() {
    super();

    // Cola de eventos musicales de Strudel.
    // Aquí se guardan hasta que llegue su timestamp.
    this.scheduledEvents = [];

    // Animaciones que ya fueron activadas y están siendo dibujadas.
    this.activeAnimations = [];

    // Corrección opcional de latencia.
    this.latencyCorrection = 0;

    // Controles persistentes enviados desde Open Stage Control.
    // Estos valores no disparan animaciones por sí solos;
    // modifican cómo se dibujan las animaciones de Strudel.
    this.controls = {
      colors: {
        bd: [255, 0, 80],       // bombo
        sd: [0, 200, 255],      // caja
        cp: [100, 220, 255],    // clap
        hh: [255, 255, 0],      // hi-hat
        other: [200, 200, 200]  // sonidos no clasificados
      },

      scale: 1,              // tamaño general de las visuales
      showHatLayer: true     // activa/desactiva visuales de hi-hat
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

      // Se limpia la cola al entrar al estado activo.
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
    // STRUDEL:
    // Eventos musicales temporizados.
    // No se dibujan apenas llegan: se guardan en cola.
    if (msg.type === "strudel") {
      this.scheduledEvents.push({
        timestamp: msg.timestamp,
        sound: msg.payload.sound,
        family: msg.payload.family,
        delta: msg.payload.delta
      });

      // Mantiene la cola ordenada por tiempo.
      this.scheduledEvents.sort((a, b) => a.timestamp - b.timestamp);
      return;
    }

    // OSC:
    // Controles persistentes de Open Stage Control.
    // No entran a la cola temporal.
    // Actualizan variables del estado visual.
    if (msg.type === "osc") {
      this.updateControls(msg.payload);
      return;
    }
  }

  updateControls(payload) {
    const address = payload.address;
    const args = payload.args;

    // Cada RGB controla una familia sonora distinta.
    // Open Stage Control debe enviar tres valores: [R, G, B].

    if (address === "/rgb_bd") {
      this.controls.colors.bd = [
        Number(args[0] ?? 255),
        Number(args[1] ?? 0),
        Number(args[2] ?? 80)
      ];
    }

    if (address === "/rgb_sd") {
      this.controls.colors.sd = [
        Number(args[0] ?? 0),
        Number(args[1] ?? 200),
        Number(args[2] ?? 255)
      ];
    }

    if (address === "/rgb_cp") {
      this.controls.colors.cp = [
        Number(args[0] ?? 100),
        Number(args[1] ?? 220),
        Number(args[2] ?? 255)
      ];
    }

    if (address === "/rgb_hh") {
      this.controls.colors.hh = [
        Number(args[0] ?? 255),
        Number(args[1] ?? 255),
        Number(args[2] ?? 0)
      ];
    }

    if (address === "/rgb_other") {
      this.controls.colors.other = [
        Number(args[0] ?? 200),
        Number(args[1] ?? 200),
        Number(args[2] ?? 200)
      ];
    }

    // Slider para controlar escala general.
    // Recomendado en Open Stage Control: rango 0.5 a 2.0
    if (address === "/scale_1") {
      this.controls.scale = Number(args[0] ?? 1);
    }

    // Toggle para mostrar u ocultar hi-hats.
    // 1 = visible, 0 = oculto
    if (address === "/toggle_hats") {
      this.controls.showHatLayer = Boolean(args[0]);
    }
  }

  processScheduledEvents() {
    const now = Date.now() + this.latencyCorrection;

    // Activa eventos solo cuando el reloj local alcanza su timestamp.
    while (
      this.scheduledEvents.length > 0 &&
      now >= this.scheduledEvents[0].timestamp
    ) {
      const ev = this.scheduledEvents.shift();

      this.activeAnimations.push({
        startTime: ev.timestamp,
        duration: ev.delta * 1000,
        sound: ev.sound,
        family: ev.family,
        x: random(width * 0.2, width * 0.8),
        y: random(height * 0.2, height * 0.8),

        // El color se toma del estado persistente de OSC.
        color: getColorForFamily(ev.family, this.controls)
      });
    }
  }

  cleanupAnimations() {
    const now = Date.now() + this.latencyCorrection;

    // Elimina animaciones que ya terminaron.
    for (let i = this.activeAnimations.length - 1; i >= 0; i--) {
      const anim = this.activeAnimations[i];
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
  bridge = new BridgeClient();

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
      type: EVENTS.DATA,
      payload: data
    });
  });

  connectBtn = createButton("Connect");
  connectBtn.position(10, 10);

  connectBtn.mousePressed(() => {
    if (bridge.isOpen) bridge.close();
    else bridge.open();
  });

  renderer.set(painter.estado_corriendo, drawRunning);
}

function draw() {
  // Fondo con transparencia para dejar estela visual.
  background(0, 30);

  painter.update();

  if (painter.state === painter.estado_corriendo) {
    painter.processScheduledEvents();
    painter.cleanupAnimations();
  }

  renderer.get(painter.state)?.();
}

function drawRunning() {
  const now = Date.now() + painter.latencyCorrection;

  // drawRunning no interpreta mensajes de red.
  // Solo dibuja animaciones ya activadas.
  for (const anim of painter.activeAnimations) {
    const elapsed = now - anim.startTime;
    const progress = elapsed / anim.duration;

    if (progress >= 0 && progress <= 1) {
      drawAnimation(anim, progress);
    }
  }
}

function drawAnimation(anim, p) {
  push();

  // Toggle persistente controlado desde Open Stage Control.
  if (anim.family === "hh" && !painter.controls.showHatLayer) {
    pop();
    return;
  }

  // Selección visual según familia sonora ya normalizada.
  switch (anim.family) {
    case "bd":
      drawKick(anim, p);
      break;

    case "sd":
      drawSnare(anim, p);
      break;

    case "cp":
      drawClap(anim, p);
      break;

    case "hh":
      drawHat(anim, p);
      break;

    default:
      drawDefault(anim, p);
      break;
  }

  pop();
}

function drawKick(anim, p) {
  const s = painter.controls.scale;
  const d = lerp(100, 600, p) * s;
  const alpha = lerp(255, 0, p);

  fill(anim.color[0], anim.color[1], anim.color[2], alpha);
  circle(width / 2, height / 2, d);
}

function drawSnare(anim, p) {
  const s = painter.controls.scale;
  const w = lerp(width, 0, p) * s;
  const alpha = lerp(255, 0, p);

  fill(anim.color[0], anim.color[1], anim.color[2], alpha);
  rect(width / 2, height / 2, w, 50 * s);
}

function drawClap(anim, p) {
  const s = painter.controls.scale;
  const alpha = lerp(255, 0, p);
  const offset = lerp(120, 0, p) * s;

  fill(anim.color[0], anim.color[1], anim.color[2], alpha);

  // Dos rectángulos laterales para diferenciar clap de snare.
  rect(width / 2 - offset, height / 2, 80 * s, 160 * s);
  rect(width / 2 + offset, height / 2, 80 * s, 160 * s);
}

function drawHat(anim, p) {
  const s = painter.controls.scale;
  const sz = lerp(40, 0, p) * s;
  const alpha = lerp(255, 0, p);

  fill(anim.color[0], anim.color[1], anim.color[2], alpha);
  rect(anim.x, anim.y, sz, sz);
}

function drawDefault(anim, p) {
  const s = painter.controls.scale;
  const size = lerp(100, 0, p) * s;
  const angle = p * TWO_PI;
  const alpha = lerp(255, 0, p);

  translate(anim.x, anim.y);
  rotate(angle);

  stroke(anim.color[0], anim.color[1], anim.color[2], alpha);
  strokeWeight(2);
  noFill();
  rect(0, 0, size, size);
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

Para abrir el servidor

```bash
node bridgeServer.js --device strudel-osc --wsPort 8081 --strudelPort 8080 --oscPort 9000 --verbose
```

- `-device strudel-osc`
le dice al bridge que va a escuchar dos adapters al mismo tiempo:
- Strudel
- Open Stage Control
`-wsPort 8081`
puerto WebSocket hacia el frontend (bridgeClient.js)
`-strudelPort 8080`
puerto donde Strudel manda .osc()
`-oscPort 9000`
puerto UDP donde Open Stage Control manda OSC
`-verbose`
para que te imprima logs y puedas ver si llegan mensajes

ASEGURARSE DE TENER

```bash
npm install osc
```

Y mi mesa de Open Stage Control fabricada fue:

unidad-7/CONTROLADORES67.json




## Bitácora de reflexión
