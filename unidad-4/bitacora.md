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



## Bitácora de reflexión



