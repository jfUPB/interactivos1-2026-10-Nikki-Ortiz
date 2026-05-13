# Unidad 5
## Bitácora de proceso de aprendizaje
### ACTIVIDAD 01

**PASO 1**

*Q: ¿Qué ventajas y desventajas ves en usar un formato binario en lugar de texto ASCII?*

A: La ventaja sería que se transmitiría la misma información con más rapidez en términos de lectura y escritura, así como utilizando menos recursos. Es un inconveniente que no sea legible para el ser humano, que sea complicado de corregir manualmente y detectar errores, y que dependa de la arquitectura.

*Q: Si xValue=500, yValue=524, aState=True, bState=False, ¿cómo se vería el paquete en hexadecimal? (Pista: convierte cada valor según su tipo y anota los bytes en orden.) Respuesta esperada: 01 F4 02 0C 01 00*

A: Si escribimos 500 en la calculadora en modo programador, en el campo decimal, su representación hexadecimal es 1F4. En datos de 2 bytes sería 01 F4. Lo mismo ocurre con yValue=524: su valor hexadecimal es 02 0C. Para true, el valor es 01 y para false, es 00.

---  

**PASO 2**

*Q: ¿Por qué el protocolo ASCII de la unidad anterior no tenía este problema de sincronización? (Pista: piensa en qué rol cumplía el carácter \n.)*

A: Debido a que se utilizaba \n como un delimitador claro para los mensajes, el programa podía determinar la línea de información con la que debía gestionar los datos.

*Q: ¿Por qué en binario no podemos usar \n como delimitador?*

A: Dado que en ASCII es un carácter especial y en binario es simplemente otro número más, cualquier byte puede tener el mismo valor.

## Bitácora de aplicación 

### ACTIVIDAD 2

**CÓDIGOS:**

<details>

<summary>ADAPTER</summary>

### MicrobitBinaryAdapter.js


```js
  // MODIFICANDO EL CÓDIGO DEL ADAPTADOR ASCII 

const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter"); // Se importa la clase base de la cual heredará este adapter

class MicrobitBinaryAdapter extends BaseAdapter { // Adapter encargado de recibir datos binarios desde la micro:bit,
  constructor({ path, baud = 115200, verbose = false } = {}) { // Inicializa configuración y variables internas
    super();
    this.path = path; // Ruta del puerto serial
    this.baud = baud; // Velocidad de comunicación
    this.port = null; // Puerto Serial
    this.buf = Buffer.alloc(0); // Buffer binario acumulador
    this.verbose = verbose;   // Modo depuración
    this.warnedCorrupt = false; // Esta linea se utiliza en el Onchunk más tarde, para verificar la suma y evitar el spameo del mensaje de error 
  }

  async connect() {  // CONEXIÓN AL PUERTO SERIAL
    if (this.connected) return;
    if (!this.path) { // Validación: se requiere un puerto
      throw new Error("serialPort is required for microbit device mode");
    }

    this.port = new SerialPort({ // Crear puerto serial
      path: this.path,
      baudRate: this.baud,
      autoOpen: false,
    });

    await new Promise((resolve, reject) => {
      this.port.open((err) => (err ? reject(err) : resolve()));
    });

    this.connected = true; // Marcar como conectado
    this.onConnected?.(`serial open ${this.path} @${this.baud}`);

    this.port.on("data", (chunk) => this._onChunk(chunk));  // Eventos del puerto, si llega datos, si hay error o si se cierra
    this.port.on("error", (err) => this._fail(err));
    this.port.on("close", () => this._closed());
  }

  async disconnect() {  // DESCONEXIÓN DEL PUERTO
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
    this.buf = Buffer.alloc(0);
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }
 // PROCESAMIENTO DE DATOS BINARIOS
  _onChunk(chunk) {  // Recibe fragmentos de datos (chunks) y reconstruye paquetes válidos

   
    this.buf = this.buf.length ? Buffer.concat([this.buf, chunk]) : chunk; // Optimización para evitar crear nuevos buffers innecesarios

    let offset = 0;
     
    while (offset <= this.buf.length - 8) { // Aquí se hace con 8 para que tenga coherencia con la documentación 

      
      const headerIndex = this.buf.indexOf(0xAA, offset); // Buscar header (0xAA) desde la posición actual

      if (headerIndex === -1) {  // Si no hay header, salir
        break; 
      }

      
      if (headerIndex + 8 > this.buf.length) {  // Verificar si hay suficientes bytes para un paquete completo (8 bytes)
        break;
      }

      const packet = this.buf.subarray(headerIndex, headerIndex + 8);  // Extraer paquete de 8 bytes

      const calculatedChk =  //  Cálculo de checksum (con & 0xFF)
        (packet[1] +
          packet[2] +
          packet[3] +
          packet[4] +
          packet[5] +
          packet[6]) & 0xFF;

      if (calculatedChk !== packet[7]) {      // Validar integridad del paquete
        if (!this.warnedCorrupt) {
          console.warn("La trama está corrupta");  // Mostrar advertencia solo una vez para evitar spam
          this.warnedCorrupt = true;
        }
        continue;  // Continuar buscando siguiente posible paquete
      }

    
      const x = packet.readInt16BE(1);   // Convertir bytes a valores interpretables
      const y = packet.readInt16BE(3);
      const btnA = packet[5] === 1;
      const btnB = packet[6] === 1;

      this.onData?.({ x, y, btnA, btnB });  // Emitir datos hacia el sistema

      offset = headerIndex + 8; // Avanzar al siguiente paquete
    }

   
    if (offset > 0) {  // Conservar solo los datos no procesados
      this.buf = this.buf.subarray(offset);
    }

    
    if (this.buf.length > 4096) { // Evitar crecimiento infinito del buffer
      this.buf = Buffer.alloc(0);
    }
  }

  _fail(err) {  // MANEJO DE ERRORES
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;  // MANEJO DE CIERRE INESPERADO
    this.connected = false;
    this.port = null;
    this.buf = Buffer.alloc(0);
    this.onDisconnected?.("serial closed (event)");
  }

  async writeLine(line) {  // ENVÍO DE DATOS A LA MICRO:BIT
    if (!this.port || !this.port.isOpen) return;
    await new Promise((resolve, reject) => {
      this.port.write(line, (err) => (err ? reject(err) : resolve()));
    });
  }

  async handleCommand(cmd) { // Permite enviar instrucciones desde el sistema hacia la micro:bit
    if (cmd?.cmd === "setLed") {
      const x = Math.max(0, Math.min(4, Math.trunc(cmd.x)));  // Validar y limitar valores
      const y = Math.max(0, Math.min(4, Math.trunc(cmd.y)));
      const v = Math.max(0, Math.min(9, Math.trunc(cmd.value)));
      await this.writeLine(`LED,${x},${y},${v}\n`);  // Enviar comando en formato ASCII
    }
  }
}
module.exports = MicrobitBinaryAdapter; // Exportar módulo
```

</details>

<details>

<summary>MICRO:BIT</summary>

### CÓDIGO PARA EL MICROBIT


```py
from microbit import *

uart.init(115200)
display.set_pixel(0, 0, 9)

HEADER = 0xAA

def int16_to_bytes(value):
    # limitar rango del acelerómetro
    if value < -2048:
        value = -2048
    if value > 2047:
        value = 2047

    # convertir a unsigned 16 bits si es negativo
    if value < 0:
        value = 65536 + value

    high = (value >> 8) & 0xFF
    low = value & 0xFF

    return high, low

while True:

    # Leer acelerómetro
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()

    #  Botones
    aState = 1 if button_a.is_pressed() else 0
    bState = 1 if button_b.is_pressed() else 0

    # Convertir a bytes (Big Endian)
    xHigh, xLow = int16_to_bytes(xValue)
    yHigh, yLow = int16_to_bytes(yValue)

    # Checksum (igual al adapter)
    checksum = (xHigh + xLow + yHigh + yLow + aState + bState) % 256

    # Paquete binario de 8 bytes
    packet = bytes([
        HEADER,   # 0
        xHigh,    # 1
        xLow,     # 2
        yHigh,    # 3
        yLow,     # 4
        aState,   # 5
        bState,   # 6
        checksum  # 7
    ])

    #  Enviar datos
    uart.write(packet)

    sleep(100)  # 10 Hz
```

</details>

**EVIDENCIAS**

**CHECKSUM**

<img width="1111" height="251" alt="image" src="https://github.com/user-attachments/assets/7b76c2d4-814f-493a-b3c7-b8314c469e75" />

*Con Spam de mensaje*

<img width="1720" height="281" alt="Captura de pantalla 2026-03-26 152832" src="https://github.com/user-attachments/assets/2f3f4e32-9452-4a1a-8a8f-48690245e856" />

*Sin spam de mensaje*
<img width="1825" height="277" alt="Captura de pantalla 2026-03-26 153525" src="https://github.com/user-attachments/assets/20fd399d-47fb-4d57-b1b8-04f06fc6a15e" />

## Bitácora de reflexión
