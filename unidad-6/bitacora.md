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





## Bitácora de reflexión
