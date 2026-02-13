# Unidad 2

## Bitácora de proceso de aprendizaje

### Actividad 1

¿Cuáles son los estados en el programa?

R/ En el diagrama los estados se resumen principalmente al tema de contar el tiempo. Siendo este el "wait_InON" y "wait_InOFF" que detectan si esta prendido o apagado el pixel.

¿Cuáles son los eventos en el programa?

R/ Los eventos son el ENTRY, el EXIT y el Timeout, siendo el Timeout relacionado a la clase Timer.

¿Cuáles son las acciones en el programa?

R/ Las acciones son encender y apagar los pixeles, y a su vez contar el tiempo para que cambien de estado.


Finalización de la actividad 1

### Actividad 2

Vas a realizar una modificación. Cuando el semáforo esté en verde, si se presiona el botón A, el semáforo debe cambiar inmediatamente a amarillo (sin esperar a que termine el tiempo de verde). El evento que se debe postear es “A” (post_event(“A”)).

R/ 
``` py
from microbit import *
import utime
from fsm import FSMTask, ENTRY, Timer

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
    if button_a.was_pressed(): semaforo1.post_event("A")
    semaforo1.update()
    utime.sleep_ms(20)

```

En la parte de post_event("A") al detectar el boton automaticamente manda wait_InYellow. 

Construye la máquina de estados que modela el problema usando PlantUML. Puedes encontrar el editor aquí y la documentación básica con ejemplos aquí.

R/
  Codigo de PlantUML
  
```
@startuml
title Semáforo - Máquina de Estados

[*] --> Red

' -------------------------
' Estado ROJO
' -------------------------
Red : entry /\n clear()\n set_pixel(rojo)\n start(timeInRed)

Red --> Green : Timeout /\n apagar rojo

' -------------------------
' Estado VERDE
' -------------------------
Green : entry /\n clear()\n set_pixel(verde)\n start(timeInGreen)

Green --> Yellow : Timeout /\n apagar verde
Green --> Yellow : A /\n apagar verde

' -------------------------
' Estado AMARILLO
' -------------------------
Yellow : entry /\n clear()\n set_pixel(amarillo)\n start(timeInYellow)

Yellow --> Red : Timeout /\n apagar amarillo

@enduml


```

Imagen PlantUML:

<img width="419" height="634" alt="image" src="https://github.com/user-attachments/assets/e92d7ac8-ed75-4917-a0d1-b19fb6919000" />


Finalización de la actividad 2

### Actividad 3

Vamos a tomarnos un momento para revisar de manera individual el código y la máquina de estados modelada. ¿Hay algo que aún no comprendes completamente?

R/ No, por ahora he entendido bien la forma en la que funcionan los programas y como se desarrollan los eventos. 👍

Finalización de la actividad 3

## Bitácora de aplicación 

### Actividad 4

Codigo de PlantUML

```
@startuml
title Temporizador Interactivo - UML State Machine

[*] --> Config

Config : entry /\n count=20\n display.show(FILL[count])
Config : A /\n if count<25 then count++\n display.show(FILL[count])
Config : B /\n if count>15 then count--\n display.show(FILL[count])
Config --> Counting : S /\n start timer

Counting : entry /\n start timer (1s)
Counting : Timeout /\n count--\n display.show(FILL[count])
Counting --> Alarm : Timeout [count==0]

Alarm : entry /\n display.show(Image.SKULL)\n speaker ON
Alarm --> Config : A /\n reset to 20

@enduml

```
Imagen de PlantUML:

<img width="389" height="631" alt="image" src="https://github.com/user-attachments/assets/f1050674-8bf2-4668-a716-933fe7476a7b" />


Codigo de Micro:bit
``` py
from microbit import *
import utime
import music

# -------------------------------------------------
# Crear imágenes de llenado
# -------------------------------------------------
def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for y in range(5):
            row = []
            for x in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs

FILL = make_fill_images()

# -------------------------------------------------
# Clase Timer
# -------------------------------------------------
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

# -------------------------------------------------
# Máquina de estados del Temporizador
# -------------------------------------------------
class TimerTask:
    def __init__(self):
        self.event_queue = []
        self.timers = []

        self.count = 20
        self.myTimer = self.createTimer("Timeout", 1000)

        self.estado_actual = None
        self.transicion_a(self.estado_config)

    def createTimer(self, event, duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, evento):
        self.event_queue.append(evento)

    def update(self):
        for t in self.timers:
            t.update()

        while len(self.event_queue) > 0:
            evento = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(evento)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual:
            self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    # -------------------------------
    # Estado CONFIGURACIÓN
    # -------------------------------
    def estado_config(self, evento):
        if evento == "ENTRY":
            self.count = 20
            display.show(FILL[self.count])

        elif evento == "A":
            if self.count < 25:
                self.count += 1
                display.show(FILL[self.count])

        elif evento == "B":
            if self.count > 15:
                self.count -= 1
                display.show(FILL[self.count])

        elif evento == "S":
            self.transicion_a(self.estado_counting)

    # -------------------------------
    # Estado COUNTING
    # -------------------------------
    def estado_counting(self, evento):
        if evento == "ENTRY":
            self.myTimer.start(1000)

        elif evento == "Timeout":
            self.count -= 1
            display.show(FILL[self.count])

            if self.count == 0:
                self.transicion_a(self.estado_alarm)
            else:
                self.myTimer.start(1000)

    # -------------------------------
    # Estado ALARM
    # -------------------------------
    def estado_alarm(self, evento):
        if evento == "ENTRY":
            display.show(Image.SKULL)
            music.play(music.ODE)

        elif evento == "A":
            self.transicion_a(self.estado_config)

# -------------------------------------------------
# Loop principal
# -------------------------------------------------
task = TimerTask()

while True:
    if button_a.was_pressed():
        task.post_event("A")

    if button_b.was_pressed():
        task.post_event("B")

    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    task.update()
    utime.sleep_ms(20)


```

Finalización de la actividad 4

## Bitácora de reflexión

### Actividad 5

¿Como resolviste el reto?

R/ Principalmente use los comandos que ya tenia para los botones e hice con p5.js y agregando una parte en micro:bit, permite que con las teclas A,B y S se imiten los 2 botones del micro:bit y la acción de sacudir para que detectara el teclado de la misma forma.

Ubica los codigos finales

R/ Codigo de p5.js

``` js
let port;
let writer;
let connectBtn;
let statusText = "Desconectado";

function setup() {
  createCanvas(400, 200);
  background(30);

  connectBtn = createButton("Conectar micro:bit");
  connectBtn.mousePressed(connectSerial);
}

async function connectSerial() {
  try {
    port = await navigator.serial.requestPort();
    await port.open({ baudRate: 115200 });

    writer = port.writable.getWriter();
    statusText = "Conectado ✅";
  } catch (err) {
    statusText = "Error al conectar ❌";
  }
}

function draw() {
  background(30);
  fill(255);
  textSize(16);
  text("Presiona A, B o S en el teclado", 60, 80);
  text("Estado: " + statusText, 60, 120);
}

async function keyPressed() {
  if (!writer) return;

  let keyUpper = key.toUpperCase();

  if (["A", "B", "S"].includes(keyUpper)) {
    await writer.write(new TextEncoder().encode(keyUpper));
  }
}
```
En la parte de p5.js le agregue unas lineas para mostrar si estaba conectado el microbit que estaba teniendo problemas para verificarlo.

Codigo de Micro:bit 

``` py
from microbit import *
import utime
import music

# -------------------------------------------------
# Inicializar UART para comunicación con p5.js
# -------------------------------------------------
uart.init(baudrate=115200)

# -------------------------------------------------
# Crear imágenes de llenado
# -------------------------------------------------
def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for y in range(5):
            row = []
            for x in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs

FILL = make_fill_images()

# -------------------------------------------------
# Clase Timer
# -------------------------------------------------
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

# -------------------------------------------------
# Máquina de estados del Temporizador
# -------------------------------------------------
class TimerTask:
    def __init__(self):
        self.event_queue = []
        self.timers = []

        self.count = 20
        self.myTimer = self.createTimer("Timeout", 1000)

        self.estado_actual = None
        self.transicion_a(self.estado_config)

    def createTimer(self, event, duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, evento):
        self.event_queue.append(evento)

    def update(self):
        for t in self.timers:
            t.update()

        while len(self.event_queue) > 0:
            evento = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(evento)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual:
            self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    # -------------------------------
    # Estado CONFIGURACIÓN
    # -------------------------------
    def estado_config(self, evento):
        if evento == "ENTRY":
            self.count = 20
            display.show(FILL[self.count])

        elif evento == "A":
            if self.count < 25:
                self.count += 1
                display.show(FILL[self.count])

        elif evento == "B":
            if self.count > 15:
                self.count -= 1
                display.show(FILL[self.count])

        elif evento == "S":
            self.transicion_a(self.estado_counting)

    # -------------------------------
    # Estado COUNTING
    # -------------------------------
    def estado_counting(self, evento):
        if evento == "ENTRY":
            self.myTimer.start(1000)

        elif evento == "Timeout":
            self.count -= 1
            display.show(FILL[self.count])

            if self.count == 0:
                self.transicion_a(self.estado_alarm)
            else:
                self.myTimer.start(1000)

    # -------------------------------
    # Estado ALARM
    # -------------------------------
    def estado_alarm(self, evento):
        if evento == "ENTRY":
            display.show(Image.SKULL)
            music.play(music.ODE)

        elif evento == "A":
            self.transicion_a(self.estado_config)

# -------------------------------------------------
# Loop principal
# -------------------------------------------------
task = TimerTask()

while True:

    # Eventos físicos
    if button_a.was_pressed():
        task.post_event("A")

    if button_b.was_pressed():
        task.post_event("B")

    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    # -------------------------------------
    # NUEVO: Eventos desde p5.js (Serial)
    # -------------------------------------
    if uart.any():
        data = uart.read(1)
        if data:
            char = chr(data[0])
            if char in ["A", "B", "S"]:
                task.post_event(char)


    task.update()
    utime.sleep_ms(20)

```

Finalización actividad 5

# 🧮
