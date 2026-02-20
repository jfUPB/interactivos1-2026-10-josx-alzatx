# Unidad 1

## Bitácora de proceso de aprendizaje

### Actividad 1

- ¿Qué es un sistema físico interactivo?
- 
R/ Es un producto con el cual las personas pueden interactuar con el y ver reflejado ciertas acciones del entorno fisico en un entorno digital, como integrar el mundo tecnologico con el mundo real.
  
- ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
- 
R/ Que tengo capacidades para hacer objetos con los que la gente pueda interactuar y que estos objetos reaccionen en tiempo real a las acciones, para mostrar diferentes tipos de contenido.

Finalización de la actividad 1

### Actividad 2

- ¿Qué es el diseño/arte generativo?
- 
R/ Es un proceso en el cual usas un programa o una AI, con el fin de generar el diseño a medida que se va ejecutando el proceso, el cual generalmente responde a acciones de los usuarios.
  
- ¿Como podrias aplicar lo que has visto en tu perfil profesional?
- 
R/ Poder crear aplicaciones las cuales reaccionen a las acciones de los usuarios en tiempo real y les den un feedback de que se recibio esa información por medio del arte.

Finalización actividad 2

### Actividad 3

Se entiende como funciona el microbit y la pagina para programar este

Finalización actividad 3 

### Actividad 4

¿Porque no funcionaba el programa con el was_pressed() y por qué funciona con is_pressed()? Explica detalladamente.

R/ Escencialmente el was_Pressed() se usa para leer acciones discretas (no muy continuas) en otras palabras solo lee 1 acción cada cierto tiempo. En cambio la is_pressed() detecta de manera continua si el botón esta siendo presionado (tambien permite detectar si el botón no se suelta a diferencia del was_pressed()

Finalización actividad 4

## Bitácora de aplicación 

### Actividad 5

El programa de p5.js.

```js

let port;
let connectBtn;
let xCircle;

function setup() {
    createCanvas(400, 400);
    port = createSerial();

    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);

    let sendBtn = createButton('Send Love');
    sendBtn.position(220, 300);
    sendBtn.mousePressed(sendBtnClick);

    xCircle = width / 2;
}

function draw() {
    background(220);

    if (port.availableBytes() > 0) {
        let dataRx = port.read(1);

        if (dataRx == 'A') {
            xCircle -= 10;   // mover a la izquierda
        }
        else if (dataRx == 'B') {
            xCircle += 10;   // mover a la derecha
        }

    }

    // Dibujar círculo
    fill('green');
    ellipse(xCircle, height / 2, 100, 100);

    // Estado del botón
    if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    } else {
        connectBtn.html('Disconnect');
    }
}

function connectBtnClick() {
    if (!port.opened()) {
        port.open('MicroPython', 115200);
    } else {
        port.close();
    }
}

function sendBtnClick() {
    port.write('h');
}

```
 

El programa de micro:bit.

```py

from microbit import *

uart.init(baudrate=115200)
display.show(Image.BUTTERFLY)

while True:
    if button_a.is_pressed():
        uart.write('A')
        sleep(500)
    if button_b.is_pressed():
        uart.write('B')
        sleep(500)
    if accelerometer.was_gesture('shake'):
        uart.write('C')
        sleep(500)
    if uart.any():
        data = uart.read(1)
        if data:
            if data[0] == ord('h'):
                display.show(Image.HEART)
                sleep(500)
                display.show(Image.HAPPY)

```

Explica detalladamente cómo funciona el sistema físico interactivo que has creado.


R/ Inicialmente creamos un programa en micro:bit Python con el cual se conecta el microbit al pc. con esta conexion nos permite usar los 2 botones que posee el microbit y que este envie las señales al computador. Acto seguido en un archivo de p5.js agregamos las librerias y el codigo con el cual se crea un circulo en la pantalla y por medio de los botones este circulo se desplaza hacia arriba/abajo y derecha/izquierda. Ya por la parte del codigo, dibujamos el circulo con el color que queramos y con el segmento de "port.AvalibleBytes" hace que el programa lea constantemente los datos entregados por el microbit. Ya con esto se le asigna una variable al circulo la cual va a variar en funcion de los botones para que este se este moviendo. Y esto seria todo lo importante para entender el funcionamiento del codigo. Muchas gracias.

# 🥇

Finalización actividad 5

## Bitácora de reflexión

### Actividad 6

Explica de que manera funciona el sistema fisico.

R/ El sistema de microbit crea las funciones y luego en p5.js se crea el cuadrado que cambia de color en función de si el botón izquierdo es presionado o no. 

Explicación de js

R/ Ahora por la parte de p5.js, inicialmente se crea un background, analiza que el microbit este configurado para comenzar a recibir la información, ya cuando el sistema detecta todo, se crea el cuadrado y con el boton izquierdo del microbit al ser pulsado el cuadrado que se encuentra en la pantalla se cambia de color verde a color rojo pero si no detecta que el boton esta siendo presionado el cuadrado conserva su color inicial (verde).

programa de js

```js

  let port;
  let connectBtn;
  let connectionInitialized = false;

  function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton("Connect to micro:bit");
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
  }

  function draw() {
    background(220);

    if (port.opened() && !connectionInitialized) {
      port.clear();
      connectionInitialized = true;
    }

    if (port.availableBytes() > 0) {
      let dataRx = port.read(1);
      if (dataRx == "A") {
        fill("red");
      } else if (dataRx == "N") {
        fill("green");
      }
    }

    rectMode(CENTER);
    rect(width / 2, height / 2, 50, 50);

    if (!port.opened()) {
      connectBtn.html("Connect to micro:bit");
    } else {
      connectBtn.html("Disconnect");
    }
  }

  function connectBtnClick() {
    if (!port.opened()) {
      port.open("MicroPython", 115200);
      connectionInitialized = false;
    } else {
      port.close();
    }
  }

```

programa de Python

Explicación del codigo Python

R/ Aca unicamente importamos el microbit y le creamos un valor que represente si los botones de este estan siendo oprimidos. Asignandose un valor "A" al ser presionado y un valor "N" al no recibir ninguna acción.

```py

from microbit import *

uart.init(baudrate=115200)

while True:

    if button_a.is_pressed():
        uart.write('A')
    else:
        uart.write('N')

    sleep(100)

```

Finalización actividad 6

