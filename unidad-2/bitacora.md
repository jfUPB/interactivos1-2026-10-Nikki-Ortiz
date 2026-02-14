# Unidad 2

## Bitácora de proceso de aprendizaje

### ACTIVIDAD 01

**Q:¿Cuáles son los estados en el programa?**

A: Los estados del programa son:  

  - ```estado_waitInON```  corresponde al estado WaitInON

  - ```estado_waitInOFF``` corresponde al estado WaitInOFF

El estado inicial es WaitInON, porque en el constructor se ejecuta:  

```py
self.transicion_a(self.estado_waitInON)
```  

**Q:¿Cuáles son los eventos en el programa?**

A: Los eventos del programa son: 

  - "ENTRY" --> cuando se entra a un estado

  - "EXIT" --> cuando se sale de un estado

  - "Timeout" --> cuando el temporizador expira

El evento "Timeout" lo genera el Timer automáticamente cuando pasa el tiempo:

```py
self.owner.post_event(self.event)
```

**Q:Cuáles son las acciones en el programa?**

A: Las acciones del programa son: 

En WaitInON (ENTRY):

```py
self.pixelState = 9
display.set_pixel(self.x,self.y,self.pixelState)
self.myTimer.start()
```
**Acciones:**

- Encender el LED (brillo 9)
- Mostrar el LED en pantalla
- Arrancar el temporizador


En WaitInOFF (ENTRY):

```py
self.pixelState = 0
display.set_pixel(self.x,self.y,self.pixelState)
self.myTimer.start()
```
**Acciones:**

- Apagar el LED (brillo 0)
- Mostrar el LED apagado
- Arrancar el temporizador

--- 

### ACTIVIDAD 02

**Q: Vas a realizar una modificación. Cuando el semáforo esté en verde, si se presiona el botón A, el semáforo debe cambiar inmediatamente a amarillo (sin esperar a que termine el tiempo de verde). El evento que se debe postear es “A” (post_event(“A”)).**

**CÓDIGO CON LA MODIFICACIÓN**

```py
from microbit import *
import utime

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)


class Semaforo:
    def __init__(self,_x,_y,_timeInRed,_timeInGreen,_timeInYellow):
        self.event_queue = []
        self.timers = []
        self.x = _x
        self.y = _y
        self.timeInRed = _timeInRed
        self.timeInGreen = _timeInGreen
        self.timeInYellow = _timeInYellow
        self.myTimer = self.createTimer("Timeout",self.timeInRed)

        self.estado_actual = None
        self.transicion_a(self.estado_waitInRed)

    def createTimer(self,event,duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def clear(self):
        display.set_pixel(self.x,self.y,0)
        display.set_pixel(self.x,self.y+1,0)
        display.set_pixel(self.x,self.y+2,0)

    def estado_waitInRed(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y,9)
            self.myTimer.start(self.timeInRed)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y,0)
            self.transicion_a(self.estado_waitInGreen)

    def estado_waitInGreen(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y+2,9)
            self.myTimer.start(self.timeInGreen)

        if ev == "Timeout":
            display.set_pixel(self.x,self.y+2,0)
            self.transicion_a(self.estado_waitInYellow)

        if ev == "A":
            display.set_pixel(self.x,self.y+2,0)
            self.transicion_a(self.estado_waitInYellow)

    def estado_waitInYellow(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y+1,9)
            self.myTimer.start(self.timeInYellow)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y+1,0)
            self.transicion_a(self.estado_waitInRed)

semaforo1 = Semaforo(0,0,2000,1000,500)

while True:
    if button_a.was_pressed():
        semaforo1.post_event("A")

    semaforo1.update()
    utime.sleep_ms(20)
```

**QUÉ SE MODIFICÓ?**

Esta máquina de estados ya soporta el evento "A" en verde, por lo tanto, lo único que falta es postear el evento cuando se presione el botón A. Para esto vamos a modificar el "While True" de nuestro código para que haga lo siguiente:

```py
while True:
    if button_a.was_pressed():
        semaforo1.post_event("A")

    semaforo1.update()
    utime.sleep_ms(20)
```

Para que así cuando se precione el botón A el semaforo pueda saltar directamenta a amarillo sin esperar el teimpo asignado para el timer del color verde

**Q: Construye la máquina de estados que modela el problema usando PlantUML. Puedes encontrar el editor aquí y la documentación básica con ejemplos aquí.**  

<img width="600" height="600" alt="imagen" src="https://github.com/user-attachments/assets/4ff80301-b625-453e-9771-130a7e492d39" />

---

### ACTIVIDAD 03

Se realizó en clase la actividad guiada desde la opción de hacercon con herencia, dividiendo el código por achivos, en donde la implementación de ciertos metotos y clases no se tienen que realizar para cada cosa porque se pueden traer como herencia.

## Bitácora de aplicación 

### ACTIVIDAD 04

**CÓDIGO:**

``` py
from microbit import *
import utime
import audio

# Se empieza utilizando el código para el display
def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26): # Se crean imágenes desde 0 hasta 25 pixeles encendidos
        rows = []
        k = 0
        for y in range(5): # Recorre filas del display
            row = []
            for x in range(5): # Recorre columnas del display
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows))) 
    return imgs

FILL = make_fill_images()
# Para mostrar usas display.show(FILL[n]) donde n será
# un valor de 0 a 25

# Sigue la implementación del código del timer

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner # Objeto que recibirá el evento cuando termine el tiempo
        self.event = event_to_post # Nombre del evento que se enviará (ej: "Timeout")
        self.duration = duration
        self.start_time = 0
        self.active = False # Indica si el timer está corriendo

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration # Permite cambiar duración si se desea
        self.start_time = utime.ticks_ms() # Guarda el tiempo actual
        self.active = True # Activa el timer

    def stop(self):
        self.active = False # Detiene el timer

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration: # Si el tiempo transcurrido es mayor o igual a la duración
                self.active = False # Se desactiva
                self.owner.post_event(self.event) # Envía el evento a la máquina de estados

# Se implementa código para hacer la máquina de estados.

class CountdownTask:
    def __init__(self):
        self.event_queue = []  # Cola de eventos pendientes
        self.timers = []  # Lista de timers internos

        self.count = 20  # Valor inicial obligatorio (20 segundos)
        self.myTimer = self.createTimer("Timeout", 1000)  # Timer de 1 segundo

        self.estado_actual = None  # Estado actual de la máquina
        self.transicion_a(self.estado_config)  # Inicia en modo configuración

    def createTimer(self,event,duration):
        t = Timer(self, event, duration)  # Crea un Timer
        self.timers.append(t)  # Lo guarda en la lista interna
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)  # Agrega eventos a la cola

    def update(self):
        # Primero se actualizan todos los timers
        for t in self.timers:
            t.update()

        # Luego se procesan todos los eventos pendientes
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)  # Se ejecuta el estado actual con el evento

    def transicion_a(self, nuevo_estado):
        if self.estado_actual:
            self.estado_actual("EXIT")  # Evento de salida
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")  # Evento de entrada

    # ESTADO CONFIGURACIÓN

    def estado_config(self, ev):
        if ev == "ENTRY":
            self.count = 20  # Reinicia a 20 cuando entra a configuración
            display.show(FILL[self.count])  # Muestra el número de pixeles encendidos

        if ev == "A":
            if self.count < 25:  # Límite superior permitido
                self.count += 1
                display.show(FILL[self.count])  # Actualiza pantalla

        if ev == "B":
            if self.count > 15:  # Límite inferior permitido
                self.count -= 1
                display.show(FILL[self.count])  # Actualiza pantalla

        if ev == "S":   # Shake arma el temporizador
            self.transicion_a(self.estado_countdown)

    # ESTADO CUENTA REGRESIVA

    def estado_countdown(self, ev):
        if ev == "ENTRY":
            display.show(FILL[self.count])  # Muestra el tiempo inicial
            self.myTimer.start(1000)  # Inicia timer de 1 segundo

        if ev == "Timeout":
            self.count -= 1  # Reduce 1 segundo
            display.show(FILL[self.count])  # Apaga un pixel

            if self.count == 0:
                self.transicion_a(self.estado_alarm)  # Pasa a alarma
            else:
                self.myTimer.start(1000)  # Reinicia el timer para el siguiente segundo

    # ESTADO ALARMA

    def estado_alarm(self, ev):
        if ev == "ENTRY":
            display.show(Image.SKULL)  # Muestra calavera
            audio.play(Sound.HAPPY)  # Reproduce sonido

        if ev == "A":
            self.transicion_a(self.estado_config)  # Vuelve a configuración

# Finalmente, se implementa el ciclo principal

task = Task()
while True:
    # Aquí generas los eventos de los botones y el gesto
    if button_a.was_pressed():
        task.post_event("A")

    if button_b.was_pressed():
        task.post_event("B")

    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    task.update() # Ejecuta la máquina de estados
    utime.sleep_ms(20) # Pequeña pausa para estabilidad (no controla el tiempo del timer)
```





## Bitácora de reflexión
