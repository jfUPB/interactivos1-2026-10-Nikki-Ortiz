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

El arte generativo es una práctica que se vale de un sistema con cierto grado de autonomía, en el cual se definen unas reglas, en algunos casos por medio de códigos que contienen instrucciones que facilitarán la realización de una obra (uno de los ejemplos de arte generativo muestra, por ejemplo, cómo por medio del código se asignan formas y colores para hacer arte abstracto). Siendo así como se basa la generación de productos y comunicaciones basadas en datos, influencias sistemáticas y programáticas multifacéticas. 

**¿Cómo podrías aplicar lo que has visto en tu perfil profesional?**  

El diseño/arte generativo en muy útil en el perfil profesional porque proporciona al usuario experiencias únicas, innovativas y automatizadas, aumentando las posibilidades en el factor de personalización y mejora el factor de la carga de trabajo sin recurrir a otras herramientas controversiales que hacen que la carga de trabajo disminuya de manera poco ética; no afecta la creatividad y mejora el ingenio a la hora de inventar nuevas experiencias multifaceticas, proviendo narrativas dinamicas, automatización de enternos visuales, parametrizaciones, entre otras cosas

--- 

> NOTA: Actividad 3 hecha en clase en p5js y en micr.bit, no hay preguntas para responder en la bitacora

### ACTIVIDAD 04

**INSTRUCCIÓN:** ¿Por qué no funcionaba el programa con was_pressed() y por qué funciona con is_pressed()? Explica detalladamente

La clave está en la diapositiva 2, donde se nos explica que button_a.was_pressed() se usa para detectar si el botón *HA SIDO* presionado, mientras que button_a.is_pressed() si quieres saber si el botón *ESTÁ* presionado en ese momento, además de que nos da la especificación de que was_pressed() es más para *EVENTOS ÚNICOS*, como lo es un click, pero si usas is_pressed(), el programa podría enviar *MÚLTIPLES* mensajes si el botón se *MANTIENE* presionado.

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

En setup() se crea el lienzo, el botón de conexión y se dibuja el círculo en el centro. En draw() el programa revisa si llegan datos desde la micro:bit, Ssi recibe la letra "A", el círculo se mueve a la izquierda, y si recibe "B", se mueve a la derecha (mediante el uso de sumas o restas de la posición del circulo). Cada vez que se mueve, la pantalla se limpia y se vuelve a dibujar el círculo en la nueva posición. El botón permite conectarse o desconectarse de la micro:bit y cambia su texto según el estado de la conexión.

## Bitácora de reflexión

### ACTIVIDAD 06

**INSTRUCCI0NES:** Vas a repasar lo aprendido en esta unidad. Regresa a la actividad 4 y trata de explicar en tus propias palabras de la manera más detallada que puedas cómo funciona el sistema físico interactivo. Analiza cada parte del código y su función dentro del sistema. Si aún tienes dudas sobre alguna parte, aprovecha para aclararlas.  


A: En la actividad 4 tenemos un sistema interactivo que permite utilizar un input como apretar un botón (en este caso A), un proceso como esperar a que los botones sean presionados y suceda un evento que cumpla con la programación del sistema, para obtener un output en donde en pantalla se mostrará el resultado de que el usuario presione un botón, que será que un cuadrado dibujado en un canvas establecido en p5.js cambie de color de acuerdo con si el botón está presionado o no.

Es por esto que empezamos programando en **micro.bit** un código que nos permita decirle al sistema que si se presiona "A", la UART mande "A" al micro:bit, y que si no presiona ningún botón, UART mande "N" al micro:bit; posteriormente, programamos las funcionalidades en p5.js, en donde se inicializa el port y las conexiones para poder añadir la variable gobal **setup** en donde se crea el botón para la conexión (con el click para conectar), el canvas y el fondo. Después, se crea la variable global **draw**, en donde se crea el cuadrado, se le asigna un tamaño, una posición y se le dice al programa que, si el micro:bit está conectado al computador y la conexión fue inicializada mediante el botón que creamos, entonces que el cuadrado se pinte de rojo si el botón "A" del micro:bit es presionado y, si no se presiona nada, que esté en verde. Finalmente, se especifica que en la función connect, en caso de que se le dé al botón de "Connect Micro:bit" el programa debe inicializar conexión a una velocidad establecida; si no se le da al botón, entonces el port está cerrado.







