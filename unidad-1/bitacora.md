# Unidad 1

## Bitácora de proceso de aprendizaje

### ACTIVIDAD 01

**¿Qué es un sistema físico interactivo?**  

Es un intercambio de información o una relación entre un sistema (que tiene imputs, outputs y procesos) y el mundo o medio físico, el cual permite materializar ideas y diseños definidos para que el usuario interactúe con ellos. Es decir, un Sistema Físico Interactivo hace uso de las tecnologías disponibles (sobre todo herramientas TIC, programación y tecnologías digitales y de sensórica) para conceptualizar, diseñar e implementar una experiencia que conecte y genere emociones con el usuario por medio de la interacción.  


**¿Cómo podrías aplicar lo que has visto en tu perfil profesional?**  

Como persona que desea irse por la rama de *Experiencias Interactivas* creo que esta clase es ideal para aprender a transformar algo sencillo del mundo físico en una experiencia que haga que el usuario se conecte con las ideas y conceptos que se buscan transmitir o del problema que se busque solucionar dentro del mundo del entretenimiento, inclusive, busca también realzar y acompañar proyectos que ya existen, dándoles un valor agregado, estando presente en procesos como: la captura de movimiento para juegos y animaciones, el branding y el diseño de producto, el cine y la música, el marketing, el uso de la realidad virtual y aumentada como complememto de espacios físicos, el mapping, assets y haptics, entre otras cosas que puedan ser usadas para causar mayor inmersión (ej. escuchar una canción de Aurora vs. Escuchar la misma canción, pero con una interpretación generativa que crea arte a partir de las vibraciones de la canción). Por lo cual, creo firmemente que los Sistemas Físicos Interactivos poco a poco empiezan a integrarse en nuestro día a día, toman relevancia y son una muestra de a dónde se encamina el futuro.

---

### ACTIVIDAD 02  

**¿Qué es el diseño/arte generativo?**  

El arte generativo es una práctica que se vale de un sistema con cierto grado de autonomía, en el cuales definen unas reglas, en algunos casos por medio de códigos que contienen instrucciones que facilitaran la realización de una obra (uno de los ejemplos de arte generativo muestra por ejemplo como por medio del código se asignan formas y colores para hacer arte abstracto). Siendo así como se basa la generación de productos y comunicaciones basadas en datos, influencias sistemáticas y prográmicas multifacéticas  

**¿Cómo podrías aplicar lo que has visto en tu perfil profesional?**  

El diseño/arte generativo en muy útil en el perfil profesional porque proporciona al usuario experiencias únicas, innovativas y automatizadas, aumentando las posibilidades en el factor de personalización y mejora el factor de la carga de trabajo sin recurrir a otras herramientas controversiales que hacen que la carga de trabajo disminuya de manera poco ética; no afecta la creatividad y mejora el ingenio a la hora de inventar nuevas experiencias multifaceticas, proviendo narrativas dinamicas, automatización de enternos visuales, parametrizaciones, entre otras cosas

--- 

> NOTA: Actividad 3 hecha en clase en p5js y en micr.bit, no hay preguntas para responder en la bitacora

### ACTIVIDAD 04

**INSTRUCCIÓN:** ¿Por qué no funcionaba el programa con was_pressed() y por qué funciona con is_pressed()? Explica detalladamente

La clave esta en la diapositiva 2 donde se nos explica que button_a.was_pressed() se usa para detectar si el botón *HA SIDO* presionado, mientras que button_a.is_pressed() si quieres saber si el botón *ESTÁ* presionado en ese momento, además de que nos da la especificación de que was_pressed() es mas para *EVENTOS ÚNICOS*, como lo son un click, pero si usas is_pressed(), el programa podría enviar *MÚLTIPLES* mensajes si el botón se *MANTIENE* presionado.

Si bien el programa consiste en que el circulo cambie de color mientras se presiona el botón y cambie cuando no esta presionado, la clave está en que son 2 eventos y no uno, lo que significa que debe funcionar de un color si se mantiene precionado y cuando no lo está de otro, por esto es que funciona con is.pressed y no con was.pressed, este último funcionaria si solo necesitaramos la instruccion de que cambie de color si se hace click y no regresara a otro color cuando no está presionado, esto se refleja en la corrección cuando is.pressed cuenta con un if y un else que corresponde al código de p5.js que tiene 2 eventos también.

## Bitácora de aplicación 

### ACTIVIDAD 05  

**INSTRUCCIÓN:** Crea un programa en p5.js que muestre un círculo en la pantalla. Utiliza los botones A y B del micro:bit para controlar la posición en x del círculo en el canvas de p5.js.

**CÓDIGO DE MICRO:BIT**
``` py
# Imports go at the top 
from microbit import *
# Siempre recordar poner el uart para iniciar la comunicacion serial
uart.init(baudrate=115200)

while True: # Se utiliza este loop para que el programa siga corriedo 
    if button_a.was_pressed(): #Si el botón A es precionado entonces manda A
        uart.write('A')
    if button_b.was_pressed(): #Si el botón A es precionado entonces manda A
        uart.write('B')
```

**CÓDIGO DE P5.JS**
``` js
let port;
let connectBtn;
let dirX;

function setup() {
  createCanvas(400, 400);
  background(220);
  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(80, 300);
  connectBtn.mousePressed(connectBtnClick);
  fill("white");
  dirX = width / 2;
  ellipse(dirX, height / 2, 100, 100);
}

function draw() {
  if (port.availableBytes() > 0) {
    let dataRx = port.read(1);

    if (dataRx == "A") {
      dirX = dirX - 30;
    }
    if (dataRx == "B") {
      dirX = dirX + 30;
    }
    background(220);

    ellipse(dirX, height / 2, 100, 100);
  }

  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
  } else {
    connectBtn.html("Disconnect");
  }
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open("MicroPython", 115200);
  } else {
    port.close();
  }
}
```
**FUNCIONAMIENTO:** 

Se utilizo el código de la Actividad 03 como base para empezar este código, es así como empezamos declarando la variable dirX para después asignarle
## Bitácora de reflexión



