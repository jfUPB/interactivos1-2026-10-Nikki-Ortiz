# Unidad 4

## Bitácora de proceso de aprendizaje
### ACTIVIDAD 01

<details>

<summary>CÓDIGO 1</summary>

### CÓDIGO DE MICRO:BIT CON COMENTARIOS

```py
from microbit import *

uart.init(115200)
display.set_pixel(0,0,9)

while True:
    xValue = accelerometer.get_x() #lee los valores del acelerometro
    yValue = accelerometer.get_y()
    aState = button_a.is_pressed() #lee los botones
    bState = button_b.is_pressed()
    data = "{},{},{},{}\n".format(xValue, yValue, aState,bState) 
    uart.write(data) #Cadena que se va a transmitir reemplazandolo los datos de los parentesis en las llaves (Solo se pueden transmitir caracteres) 
    sleep(100) # Envia datos a 10 Hz
```
</details>

**ANÁLISIS:** 

Para este codigo se estan usando un protocolo serial ASCII, para decifrar el tipo de datos se usa una tabla ASCII, se necesita saber para esto en que base está, en el caso del ejemplo sería Base 16 = Hexagecimal. Le caracter especial me dice dónde empieza y termina el codigo, además se hace uso de booleanos para saber que interacciones se estan realizando con el dispositivo, en esta caso seria el estado del botón (si esta presionado o no) 

<details>

<summary>CÓDIGO 2</summary>

### CÓDIGO DE P5.JS MODIFICADO

```js
let c;
let lineModuleSize = 0;
let angle = 0;
let angleSpeed = 1;
let lineModule = [];
let lineModuleIndex = 0;

let clickPosX = 0;
let clickPosY = 0;

function preload() {
  lineModule[1] = loadImage('data/02.svg');
  lineModule[2] = loadImage('data/03.svg');
  lineModule[3] = loadImage('data/04.svg');
  lineModule[4] = loadImage('data/05.svg');
}

function setup() {
  createCanvas(windowWidth, windowHeight);
  background(255);
  //cursor(CROSS);
  noCursor();
  strokeWeight(0.75);

  c = color(181, 157, 0);
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

function draw() {
  if (mouseIsPressed && mouseButton == LEFT) {
    let x = mouseX;
    let y = mouseY;
     
    push();
    translate(x, y);
    rotate(radians(angle));
    stroke(c);
    line(0, 0, lineModuleSize, lineModuleSize);
    angle += angleSpeed;
    pop();
  }
}

function mousePressed() {
  // create a new random color and line length
  lineModuleSize = random(50, 160);

  // remember click position
}

function keyReleased() {

  // change color
  if (key == ' ') c = color(random(255), random(255), random(255), random(80, 100));
}
```

</details>

**ANÁLISIS:** 

Hay que hacer una transición para que el micro:bit pueda interactuar con el programa en lugar del Mouse para esto se deben hacer modificaciones 

- Si al botón a esta precionado y muevo el dispositivo dibuja 
- SHIFT no se puede tener
- KEYISPRESSED no se puede tener 
- Si boton b y suelto cambio de color

**COMO FUNCIONA LA APP**

<img width="1918" height="846" alt="imagen" src="https://github.com/user-attachments/assets/3201baf0-4917-43e6-922f-32a330f9edea" />



<details>

<summary>GITBASH</summary>

### INSTRUCCIONES Y CAPTURA DE GITBASH

<img width="1657" height="1480" alt="Captura de pantalla 2026-03-04 153852" src="https://github.com/user-attachments/assets/a93a212e-eed9-4d32-80f6-5ca408966dcd" />

- mkdir: crear carpeta
- clone: Copiar
- Clear: Limpiar consola
- control + c: apagar servidor 
- cd: Change Directory + donde se quiera entrar
- ls: Mostrar Contenido de la posición actual
-  npm install: instalar modulos con node
- node bridgeServer.js: para abrir el servidor
- crear carpeta: 
- copiar el repositorio en el sistema: git clone
- Abrir visual: code .

**PARA CONECTARSE A MICRO: BIT**

-  node bridgeServer.js --device microbit

</details>


## Bitácora de aplicación 

### ACTIVIDAD 02

Se crea un archivo para el Adapter 2 y se toma el código muestra del Adapter ASCII para modifiarlo según los parámetros que se nos piden

<details>

<summary>MirobitAdapter2.js</summary>

### CÓDIGO COMENTADO Y CON LAS MODIFICACIONES 

```js
// MODIFICANDO EL CÓDIGO DEL ADAPTADOR ASCII PARA QUE SIRVA 


const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error {}

function parseFrame(line) { // Una función de limpieza y validación. Se asegura de que los datos recibidos no estén corruptos
  const trimmed = line.trim();

  if (!trimmed.startsWith("$")) {
    throw new ParseError("Frame does not start with $");
  }

  const body = trimmed.slice(1);
  const parts = body.split("|"); // Divide la cadena de texto usando la barrita vertical como separador de campos, EN VEZ DE ;.

  if (parts.length !== 6) {
    throw new ParseError(`Expected 6 fields, got ${parts.length}`);
  }

  const data = {};

  for (const part of parts) {
    const [key, value] = part.split(":");
    if (key === undefined || value === undefined) {
      throw new ParseError(`Malformed field: ${part}`);
    }
    data[key] = value;
  }

  if ( // Esto para luego ejecutar el funcionamiento que se indica en los comentarios de cada dato
    data.T === undefined ||  // T: Timestamp en milisegundos desde el arranque del dispositivo (entero).
    data.X === undefined ||
    data.Y === undefined || // X, Y: Valores del acelerómetro (enteros entre -2048 y 2047).
    data.A === undefined ||
    data.B === undefined || // A, B: Estado de los botones, 1 presionado, 0 liberado.
    data.CHK === undefined  // CHK: Checksum calculable. Es un número entero de 3 dígitos que representa la suma de los valores absolutos de X, Y, A y B.
  ) { // nota adicional: al checksum ser el último no necesita de ||
    throw new ParseError("Missing required fields");
  }

  const t = Number(data.T); // convierte el data que se mande a num
  const x = Number(data.X);
  const y = Number(data.Y);
  const a = Number(data.A);
  const b = Number(data.B);
  const chk = Number(data.CHK);

  if (![t, x, y, a, b, chk].every(Number.isFinite)) { // verifica que los datos sean numeros finitos
    throw new ParseError("Invalid numeric data");
  }

  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) { // verifica que los rangos de X y Y se conserven según los parámetros.
    throw new ParseError("Out of expected range");
  }

  if (![0, 1].includes(a) || ![0, 1].includes(b)) { // Solo permite que los botones A y B sean 0 (suelto) o 1 (presionado).
    throw new ParseError("Invalid button data");
  }

  const calculatedChk = Math.abs(x) + Math.abs(y) + a + b; // El código suma los valores absolutos de los sensores y lo compara con el valor CHK que envió la placa. Si no coinciden, significa que el mensaje se "rompió" o se perdió información durante el viaje por el cable.

  if (calculatedChk !== chk) { // Muestra error si no es así 
    throw new ParseError(
      `Corrupt frame: CHK=${chk}, expected=${calculatedChk}`
    );
  }

  return { 
    x: x | 0, // enteros 
    y: y | 0,
    btnA: a === 1,
    btnB: b === 1,
  };
}

class MicrobitAdapter2 extends BaseAdapter { // HERENCIA DEL BASE ADAPTER
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = "";
    this.verbose = verbose;
  }

  async connect() {
    if (this.connected) return;
    if (!this.path) {
      throw new Error("serialPort is required for microbit device mode");
    }

    this.port = new SerialPort({
      path: this.path,
      baudRate: this.baud,
      autoOpen: false,
    });

    await new Promise((resolve, reject) => {
      this.port.open((err) => (err ? reject(err) : resolve()));
    });

    this.connected = true;
    this.onConnected?.(`serial open ${this.path} @${this.baud}`);

    this.port.on("data", (chunk) => this._onChunk(chunk));
    this.port.on("error", (err) => this._fail(err));
    this.port.on("close", () => this._closed());
  }

  async disconnect() {
    if (!this.connected) return;
    this.connected = false;

    if (this.port && this.port.isOpen) {
      await new Promise((resolve, reject) => {
        this.port.close((err) => {
          if (err) reject(err);
          else resolve();
        });
      });
    }

    this.port = null;
    this.buf = "";
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
    this.buf += chunk.toString("utf8");

    let idx;
    while ((idx = this.buf.indexOf("\n")) >= 0) {
      const line = this.buf.slice(0, idx).trim();
      this.buf = this.buf.slice(idx + 1);

      if (!line) continue;

      try {
        const parsed = parseFrame(line);
        this.onData?.(parsed);
      } catch (e) {
        if (e instanceof ParseError) {
          if (String(e.message).startsWith("Corrupt frame")) {
            console.warn("[MicrobitAdapter2] Warning:", e.message, "raw:", line);
          } else if (this.verbose) {
            console.log("[MicrobitAdapter2] Bad data:", e.message, "raw:", line);
          }
        } else {
          this._fail(e);
        }
      }
    }

    if (this.buf.length > 4096) this.buf = "";
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = "";
    this.onDisconnected?.("serial closed (event)");
  }

  async writeLine(line) {
    if (!this.port || !this.port.isOpen) return;
    await new Promise((resolve, reject) => {
      this.port.write(line, (err) => (err ? reject(err) : resolve()));
    });
  }

  async handleCommand(cmd) {
    if (cmd?.cmd === "setLed") {
      const x = Math.max(0, Math.min(4, Math.trunc(cmd.x)));
      const y = Math.max(0, Math.min(4, Math.trunc(cmd.y)));
      const v = Math.max(0, Math.min(9, Math.trunc(cmd.value)));
      await this.writeLine(`LED,${x},${y},${v}\n`);
    }
  }
}

module.exports = MicrobitAdapter2;
```
</details>

Después se hacen las modificaciones en el ```bridgeSerber.js``` 

Modificación en el área de las const al principio:

```js
const MicrobitAdapter2 = require("./adapters/MicrobitAdapter2");
```

Modificación en el área "async function createAdapter()":

```js
if (DEVICE === "microbit-two") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    return new MicrobitAdapter2({ path, baud: BAUD, verbose: VERBOSE }); //USANDO COMO REFERENCIA EL CÓDIGO DE ARRIBA Y EL COMENTADO SE PONE "MICROBIT-TWO" COOMO EL NOMBRE QUE VA A IDENTIFICAR EL ADAPTADOR NUEVO.
  }
```
> NOTA: Se hizo así para no reemplazar el código del ASCII y basandose en el código comentado del bin, se tiene la duda si se puede hacer así, si se puede hacer mediante un return, o si se debe reemplazar en el ASCII para no generar posibles cruzes de datos. Se tuvo en cuenta cambiarle el nombre a "Microbit-two" (basado en el bin) para evitar esto, respetando la arquitectura de muestra.

Se continua entonces haciendo las modificaciones a el Sketch.js para que cumpla con la base que se nos dió para el arte generativo, respetando el codigo muestra 

<details>

<summary>Sketch.js</summary>

### CÓDIGO COMENTADO Y CON LAS MODIFICACIONES 
``` js
// BASADO EN EL CÓDIGO QUE SE DIÓ SE ADAPTO PARA HACER ESTE SKETCH JS 

const EVENTS = {
    CONNECT: "CONNECT",
    DISCONNECT: "DISCONNECT",
    DATA: "DATA",
    KEY_PRESSED: "KEY_PRESSED",
    KEY_RELEASED: "KEY_RELEASED",
};

class PainterTask extends FSMTask {
    constructor() {
        super();

        this.rxData = { // Un objeto que guarda el estado actual de los sensores (X, Y, botones).
            x: 0,
            y: 0,
            btnA: false,
            btnB: false,
            ready: false
        };

        this.circleResolution = 2;
        this.radius = 10;
        this.shouldFill = false;
        this.shouldDraw = false;

        this.transitionTo(this.estado_esperando);
    }

    estado_esperando = (ev) => { // Es el punto de partida definido en el constructor. Su objetivo es mantener la aplicación en "reposo" hasta que el hardware esté listo.
        if (ev.type === "ENTRY") {
            cursor();
            console.log("Waiting for connection...");
        } else if (ev.type === EVENTS.CONNECT) {
            this.transitionTo(this.estado_corriendo);
        }
    };

    estado_corriendo = (ev) => { // Es el estado activo donde ocurre el dibujo generativo y la interacción con los sensores.
        if (ev.type === "ENTRY") {
            noCursor();
            background(255);
            strokeWeight(2);
            stroke(0, 25);
            noFill();

            console.log("Microbit ready to draw");

            this.rxData = {
                x: 0,
                y: 0,
                btnA: false,
                btnB: false,
                ready: false
            };

            this.circleResolution = 2;
            this.radius = 10;
            this.shouldFill = false;
            this.shouldDraw = false;
        }

        else if (ev.type === EVENTS.DISCONNECT) {
            this.transitionTo(this.estado_esperando);
        }

        else if (ev.type === EVENTS.DATA) {
            this.updateLogic(ev.payload);
        }

        else if (ev.type === EVENTS.KEY_PRESSED) {
            this.handleKeys?.(ev.keyCode, ev.key);
        }

        else if (ev.type === EVENTS.KEY_RELEASED) {
            this.handleKeyRelease?.(ev.keyCode, ev.key);
        }

        else if (ev.type === "EXIT") {
            cursor();
        }
    };

    updateLogic(data) { // traduce los valores del acelerómetro de la Micro:bit (que suelen ir de -2048 a 2047) a valores que p5.js pueda usar para dibujar
        this.rxData.ready = true;
        this.rxData.x = data.x;
        this.rxData.y = data.y;
        this.rxData.btnA = data.btnA;
        this.rxData.btnB = data.btnB;

        // Y del acelerómetro -> resolución del polígono
        this.circleResolution = int(map(data.y, -2048, 2047, 2, 10));
        this.circleResolution = constrain(this.circleResolution, 2, 10);

        // X del acelerómetro -> radio
        this.radius = map(data.x, -2048, 2047, -360, 360);

        // Botón B -> fill
        this.shouldFill = data.btnB;

        // Botón A -> dibujar
        this.shouldDraw = data.btnA;
    }
}

let painter;
let bridge;
let connectBtn;
const renderer = new Map();

function setup() { // Crea el canvas y los demás parametros que nos dieron en la aplicación inicial
    createCanvas(720, 720);
    noFill();
    background(255);
    strokeWeight(2);
    stroke(0, 25);

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
            payload: {
                x: data.x,
                y: data.y,
                btnA: data.btnA,
                btnB: data.btnB
            }
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

function draw() { // Se ejecuta constantemente y le pregunta a la FSM en qué estado está y llama a la función de dibujo correspondiente.
    painter.update();
    renderer.get(painter.state)?.();
}

function drawRunning() { //  A diferencia del código original que usaba if (mouseIsPressed), aquí dependemos de lo que guardamos en this.shouldDraw (que viene del botón A de la Micro:bit).
    let task = painter;

    if (!task.rxData.ready) return;
    if (!task.shouldDraw) return;

    push();
    translate(width / 2, height / 2);

    let angle = TAU / task.circleResolution;

    if (task.shouldFill) {
        fill(34, 45, 122, 50);
    } else {
        noFill();
    }

    beginShape();
    for (let i = 0; i <= task.circleResolution; i++) {
        let x = cos(angle * i) * task.radius;
        let y = sin(angle * i) * task.radius;
        vertex(x, y);
    }
    endShape();

    pop();
}

function windowResized() {
    resizeCanvas(windowWidth, windowHeight);
}
```
</details>

Finalmente se arregla el código para el Micro:bit

```py
from microbit import *

uart.init(115200)
display.set_pixel(0,0,9)

startTime = running_time()

while True:
    t = running_time()
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()
    aState = 1 if button_a.is_pressed() else 0
    bState = 1 if button_b.is_pressed() else 0
    checksum = abs(xValue) + abs(yValue) + aState + bState
    
    data = "$T:{}|X:{}|Y:{}|A:{}|B:{}|CHK:{}\n".format(t, xValue, yValue, aState, bState, checksum)
    
    uart.write(data)
    
    sleep(100) # Envia datos a 10 Hz
```
## Bitácora de reflexión




