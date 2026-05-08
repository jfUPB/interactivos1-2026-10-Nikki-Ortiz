# Unidad 6

## Bitácora de proceso de aprendizaje

### ACTIVIDAD 01

**¿Qué diferencia hay entre recibir un mensaje y llevarlo a cabo?**

Cuando se recibe un mensaje, significa que los datos han llegado al sistema: el WebSocket lo capturó y permanece en la memoria. Realizarlo significa que el sistema responde a esa información: dibuja algo, modifica un estado y genera una animación.

**¿Por qué un sistema audiovisual puede necesitar `timestamp` además de los datos del evento?**

Audio y video no se rigen por el mismo reloj, y además la red puede añadir una latencia que cambia todo el tiempo. Sin un `timestamp`, el sistema no puede saber *cuándo* debería suceder la respuesta visual: solo sabe *qué* pasó.

Strudel calcula con exactitud el momento en que debe sonar cada nota y envía ese dato como `timestamp`. Si la app visual usa ese mismo `timestamp` para decidir cuándo dibujar, la animación se mantiene sincronizada con el audio incluso si el mensaje llega un poco tarde por la red.

**Qué aspectos de la arquitectura de las unidades 4 y 5 permanecen intactos?**

La logica de capas se mantiene completamente. El patron Adapter sigue siendo el mismo — en las unidades 4 y 5 se crearon MicrobitASCIIAdapter y MicrobitBinaryAdapter. En la unidad 6 se creo StrudelAdapter. El mecanismo es identico: extender BaseAdapter, implementar connect(), disconnect() y getConnectionDetail(), y llamar a this.onData?.() con datos normalizados.

El bridgeServer.js no cambió su estructura. Sigue levantando un servidor WebSocket, creando un adapter según el --device que se le pase, y retransmitiendo los datos al frontend.

El BridgeClient sigue siendo el intermediario entre el bridge y el sketch.js. La FSMTask sigue coordinando estados. Lo que cambio es solo el adapter.

**Si Strudel fuera "el dispositivo", ¿cuál sería su protocolo?**
El protocolo de Strudel es WebSocket sobre TCP. Cuando se usa .osc() en el patrón, Strudel envía mensajes JSON por WebSocket al puerto 8080.

```JSON
{
  "address": "/dirt/play",
  "args": ["s", "tr909bd", "delta", 0.5, "cps", 0.5, "cycle", 15.25],
  "timestamp": 1774966984435.2805
}
```

**¿Qué variables mínimas necesitarías extraer para construir una visualización útil?**

Las variables mínimas son tres:

- s (sound): el nombre del instrumento, que permite clasificar el tipo de animación a mostrar (bombo, caja, hihat, etc.)
- timestamp: el momento exacto en que debe ocurrir la animación
- delta: la duración del ciclo rítmico, que define cuánto tiempo dura la animación antes de desaparecer

**¿Qué problema resuelve la cola de eventos?**

La cola de eventos resuelve el problema de la sincronizacion. Sin ella, cada evento se ejecutaría en el momento en que llega al fronstend. Con la cola, los eventos se almacenan con su timestamp y se ejecutan exactamente cuando Date.now() alcanza ese valor.
¿Por qué esta capa no pertenece al bridge sino al lado que interpreta el evento?

**Porque el bridge es una capa de transporte — su única responsabilidad es mover datos de un lugar a otro. No debe saber qué significan esos datos ni cuándo deben ejecutarse.**

La cola pertenece al frontend porque es ahí donde se sabe qué hacer con el evento — cómo dibujarlo, cuánto tiempo mantenerlo visible, qué color usar. El bridge entrega el evento con su timestamp intacto y el frontend decide cuándo y cómo reaccionar.
¿Qué papel cumple el Adapter en U4 y U5?

**En las unidades 4 y 5 el Adapter cumple el papel de traductor entre el protocolo del dispositivo físico y el codigo que espera el resto del sistema.**

En la unidad 4, MicrobitASCIIAdapter lee líneas de texto con formato CSV del puerto serial y las convierte en objetos { x, y, btnA, btnB }. En la unidad 5, MicrobitBinaryAdapter lee paquetes y hace la misma conversión. En ambos casos, el resto del sistema no sabe nada del protocolo original.
¿Qué Adapter necesitas para que los eventos de Strudel no entren "crudos"?

Necesito un StrudelAdapter que reciba los mensajes JSON de Strudel, extraiga los campos relevantes de los args, clasifique el sonido en (bd, sd, hh, etc.) y entregue al sistema un objeto normalizado con type, timestamp y payload. Así el resto del sistema nunca ve el formato crudo de Strudel.

## Bitácora de aplicación 

Tomamos el repositorio que veniamos trabajando en la unidad 5 y creamos un adaptador para strudel 

<details>
<summary><b>StrudelAdapter.js</b></summary>

``` js
const BaseAdapter = require("./BaseAdapter");
const { WebSocketServer } = require("ws");

class StrudelAdapter extends BaseAdapter {
  constructor({ port = 8080, verbose = false } = {}) {
    super();
    this.port = port;
    this.verbose = verbose;
    this.wss = null;
  }

  async connect() {
    if (this.connected) return;

    this.wss = new WebSocketServer({ port: this.port });

    this.wss.on("connection", (ws) => {
      this.onConnected?.(`Strudel conectado en ws://127.0.0.1:${this.port}`);

      ws.on("message", (raw) => {
        const normalized = this._normalize(raw);
        if (!normalized) return;

        if (this.verbose) {
          console.log("[StrudelAdapter] normalized:", normalized);
        }

        this.onData?.(normalized);
      });

      ws.on("close", () => {
        this.onDisconnected?.("Strudel desconectado");
      });

      ws.on("error", (err) => {
        this.onError?.(`WS client error: ${err.message || err}`);
      });
    });

    this.connected = true;
    this.onConnected?.(`Esperando eventos de Strudel en ws://127.0.0.1:${this.port}`);
  }

  async disconnect() {
    if (!this.connected) return;

    if (this.wss) {
      this.wss.close();
      this.wss = null;
    }

    this.connected = false;
    this.onDisconnected?.("Strudel adapter detenido");
  }

  getConnectionDetail() {
    return `strudel ws port ${this.port}`;
  }

  _normalize(raw) {
    let msg;

    try {
      msg = JSON.parse(raw.toString("utf8"));
    } catch (e) {
      this.onError?.("Mensaje de Strudel no es JSON válido");
      return null;
    }

    if (!msg || !Array.isArray(msg.args) || typeof msg.timestamp !== "number") {
      this.onError?.("Mensaje de Strudel incompleto");
      return null;
    }

    const params = {};
    for (let i = 0; i < msg.args.length; i += 2) {
      const key = msg.args[i];
      const value = msg.args[i + 1];
      params[key] = value;
    }

    const sound = params.s || "unknown";
    const family = this._getFamily(sound);

    return {
      type: "strudel",
      timestamp: msg.timestamp,
      payload: {
        eventType: "noteEvent",
        sound: sound,
        family: family,
        delta: Number(params.delta ?? 0.25),
        cycle: Number(params.cycle ?? 0),
        cps: Number(params.cps ?? 0),
        bank: params.bank ?? null
      }
    };
  }

  _getFamily(sound) {
    if (sound.includes("bd")) return "bd";
    if (sound.includes("sd")) return "sd";
    if (sound.includes("cp")) return "cp";
    if (sound.includes("hh")) return "hh";
    if (sound.includes("oh")) return "hh";
    return "other";
  }
}

module.exports = StrudelAdapter;
```

</details>

---

Después, pasamos a modificar `bridgeServer.js` ya que tenemos que crear la constante con el nuevo adapter

``` js
const StrudelAdapter = require("./adapters/StrudelAdapter");
```

Es muy importante que en ese mismo server busquemos el `createAdapter()` y agreguemos el dispositivo del strudel para que el se conecte y se use el adaptador que acabamos de crear

``` js
  if (DEVICE === "strudel") {
    const port = parseInt(getArg("strudelPort", "8080"), 10);
    return new StrudelAdapter({ port, verbose: VERBOSE });
  }
```

Y también, hay que añadir lo siguiente en `adapter.onData = (d) => {`, que está dentro de la funcion `main`, al principio

``` js
 if (d.type === "strudel") {
      broadcast(wss, d);
      return;
    }
```

Esto permite que, si el adapter es de Strudel, reenvía el mensaje ya normalizado; y si es de micro:bit siga con normalidad, lo revelante aque es poner el if arriba del microbit.

---

El paso que siguie es: Modificar `bridgeClient.js` solo para permitir que pase `type: "strudel"` entonces el el msg.type que antes era solo para el micro:bit ahora ponemos esto:


``` js
      if (msg.type === "microbit" || msg.type === "strudel") {
        this._onData?.(msg);
        return;
      }
```
---

Por ultimo: Modificar `sketch.js` 

<details>
  <summary><b>sketch.js</b></summary>

``` js
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

    this.transitionTo(this.estado_esperando);
  }

  estado_esperando = (ev) => {
    if (ev.type === "ENTRY") {
      cursor();
      console.log("Waiting for Strudel connection...");
    } else if (ev.type === EVENTS.CONNECT) {
      this.transitionTo(this.estado_corriendo);
    }
  };

  estado_corriendo = (ev) => {
    if (ev.type === "ENTRY") {
      noCursor();
      background(0);
      console.log("Strudel ready");
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
    if (msg.type !== "strudel") return;

    this.scheduledEvents.push({
      timestamp: msg.timestamp,
      sound: msg.payload.sound,
      family: msg.payload.family,
      delta: msg.payload.delta
    });

    this.scheduledEvents.sort((a, b) => a.timestamp - b.timestamp);
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
        duration: ev.delta * 1000,
        sound: ev.sound,
        family: ev.family,
        x: random(width * 0.2, width * 0.8),
        y: random(height * 0.2, height * 0.8),
        color: getColorForFamily(ev.family)
      });
    }
  }

  cleanupAnimations() {
    const now = Date.now() + this.latencyCorrection;

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

  switch (anim.family) {
    case "bd":
      drawKick(anim, p);
      break;

    case "sd":
    case "cp":
      drawSnare(anim, p);
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
  const d = lerp(100, 600, p);
  const alpha = lerp(255, 0, p);
  fill(anim.color[0], anim.color[1], anim.color[2], alpha);
  circle(width / 2, height / 2, d);
}

function drawSnare(anim, p) {
  const w = lerp(width, 0, p);
  const alpha = lerp(255, 0, p);
  fill(anim.color[0], anim.color[1], anim.color[2], alpha);
  rect(width / 2, height / 2, w, 50);
}

function drawHat(anim, p) {
  const sz = lerp(40, 0, p);
  const alpha = lerp(255, 0, p);
  fill(anim.color[0], anim.color[1], anim.color[2], alpha);
  rect(anim.x, anim.y, sz, sz);
}

function drawDefault(anim, p) {
  const size = lerp(100, 0, p);
  const angle = p * TWO_PI;
  const alpha = lerp(255, 0, p);

  translate(anim.x, anim.y);
  rotate(angle);

  stroke(anim.color[0], anim.color[1], anim.color[2], alpha);
  strokeWeight(2);
  noFill();
  rect(0, 0, size, size);
}

function getColorForFamily(family) {
  const colors = {
    bd: [255, 0, 80],
    sd: [0, 200, 255],
    cp: [100, 220, 255],
    hh: [255, 255, 0],
    other: [200, 200, 200]
  };

  return colors[family] || colors.other;
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```
</details>

---
para conectarse usar en el Git Bash:

``` gitbash
node bridgeServer.js --device strudel --wsPort 8081 --strudelPort 8080
```
`device strudel` → usa el nuevo adapter

`wsPort 8081` → frontend escucha ahí

`strudelPort 8080` → Strudel manda eventos ahí

**PRUEBA DE FUNCIONALIDAD**

<img width="1244" height="656" alt="Captura de pantalla 2026-05-08 094226" src="https://github.com/user-attachments/assets/02ea78db-8d7a-4ac9-9842-ab7519a62255" />

## Bitácora de reflexión
