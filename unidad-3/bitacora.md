# Unidad 3

## Bitácora de proceso de aprendizaje

### Actividad 1

1. Si el botón A se presiona el semáforo debe cambiar a modo “peatonal”. En este modo el semáforo se pone en rojo, pero antes debe pasar por amarillo para darle tiempo a los autos a detenerse.

2. Si el botón B se presiona el semáforo pasa a modo nocturno. En este modo el semáforo parpadea en amarillo. Si el botón A se presiona en este modo, el semáforo vuelve a modo normal.

Finalización actividad 1

### Actividad 2

1. Si está corriendo el temporizador, al presionar el botón A se debe pausar el conteo. Si se vuelve a presionar el botón A, el conteo se reanuda en donde se había pausado.

2. Si el temporizador está corriendo, si se presiona una secuencia de botones A-B-A, el temporizador debo volver a modo de configuración.

R/ Codigo micro:bit

main.py

```py

from microbit import *
from fsm import FSMTask, ENTRY, EXIT
from utils import FILL
import utime
import music

class Temporizador(FSMTask):
    def __init__(self):
        super().__init__()
        self.sequence = []
        self.myPassword = ["A","B","A"]
        self.counter = 20
        self.myTimer = self.add_timer("Timeout",1000)
        self.estado_actual = None
        self.transition_to(self.estado_config)


    def estado_config(self, ev):
        if ev == ENTRY:
            self.counter = 20
            display.show(FILL[self.counter])
            self.myTimer.start()
        if ev == "A":
            if self.counter > 15:
                self.counter -= 1
            display.show(FILL[self.counter])
        if ev == "B":
            if self.counter < 25:
                self.counter += 1
            display.show(FILL[self.counter])
        if ev == "S":
            self.transition_to(self.estado_armed)

    def estado_armed(self, ev):
        if ev == ENTRY:
            self.sequence.clear()
            self.myTimer.start()

        if ev == "Timeout":
            if self.counter > 0:
                self.counter -= 1
                display.show(FILL[self.counter])
                if self.counter == 0:
                    self.transition_to(self.estado_timeout)
                else:
                    self.myTimer.start()
                    
        if ev == "S":
            if self.myTimer.active == False:
                self.myTimer.start()
            else:
                self.myTimer.stop()

        if ev == "A" or ev == "B":
            self.sequence.append(ev)
            if len(self.sequence) == 3:
                if self.sequence == self.myPassword:
                    self.transition_to(self.estado_config)
                else:
                    self.sequence.clear()
            
                

    def estado_timeout(self, ev):
        if ev == ENTRY:
            display.show(Image.SKULL)
            music.play(music.FUNERAL)
        if ev == "A":
            music.stop()
            self.transition_to(self.estado_config)

temporizador = Temporizador()

while True:

    if button_a.was_pressed():
        temporizador.post_event("A")
    if button_b.was_pressed():
        temporizador.post_event("B")
    if accelerometer.was_gesture("shake"):
        temporizador.post_event("S")

    temporizador.update()
    utime.sleep_ms(20)

```
fsm.py

```py

import utime

ENTRY = "ENTRY"
EXIT  = "EXIT"

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
        if self.active and utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
            self.active = False
            self.owner.post_event(self.event)


class FSMTask:
    def __init__(self):
        self._q = []
        self._timers = []
        self._state = None

    def post_event(self, ev):
        self._q.append(ev)

    def add_timer(self, event, duration):
        t = Timer(self, event, duration)
        self._timers.append(t)
        return t

    def transition_to(self, new_state):
        if self._state:
            self._state(EXIT)
        self._state = new_state
        self._state(ENTRY)

    def update(self):
        for t in self._timers:
            t.update()
        while self._q:
            ev = self._q.pop(0)
            if self._state:
                self._state(ev)

```
utils.py

```py

from microbit import Image

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
# Para mostrar usas display.show(FILL[n]) donde n será
# un valor de 0 a 25

```

Finalización actividad 2

### Actividad 3

Registro de aprendizaje del codigo

R/ En resumidas me parece que el resultado es mucho mas satisfactorio en parte por la forma en la que se ve y el como ya se trabaja con una pagina, pero me parece mas complejo de entender porque lo siento mas extenso y con el tema de las librerias se me complico un poco. Pero al final del dia se logro entender bien todo.

## Bitácora de aplicación 

### Actividad 4

Codigo de sketch (p5.js) con el Microbit funcionando como control para el temporizador y las acciones adicionales

``` js

// ===============================
// VARIABLES GLOBALES
// ===============================

let port;
let reader;
let connectBtn;

const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",        // Botón A
  INC: "B",        // Botón B
  START: "S",      // Shake
  TICK: "Timeout",
};

let temporizador;
let renderer = new Map();

// ===============================
// CLASE TEMPORIZADOR (FSM)
// ===============================

class Temporizador extends FSMTask {

  constructor(minValue, maxValue, defaultValue) {
    super();

    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;

    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    this.sequence = [];   // buffer para detectar A-B-A

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);

    this.transitionTo(this.estado_config);
  }

  get currentState() {
    return this.state;
  }

  // ===============================
  // ESTADO CONFIG
  // ===============================

  estado_config = (ev) => {

    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
      this.sequence = [];
    }

    else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue)
        this.configValue--;
    }

    else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue)
        this.configValue++;
    }

    else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  };

  // ===============================
  // ESTADO ARMED (corriendo)
  // ===============================

  estado_armed = (ev) => {

    if (ev === ENTRY) {
      this.sequence = [];
      this.myTimer.start();
    }

    else if (ev === EVENTS.TICK) {

      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;

        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start();
        }
      }
    }

    else if (ev === EVENTS.DEC || ev === EVENTS.INC) {

      // Detectar secuencia A-B-A
      if (this.checkSequence(ev)) {
        this.transitionTo(this.estado_config);
        return;
      }

      // Si solo fue A (sin completar secuencia) → pausa
      if (ev === EVENTS.DEC) {
        this.transitionTo(this.estado_pause);
      }
    }

    else if (ev === EXIT) {
      this.myTimer.stop();
    }
  };

  // ===============================
  // ESTADO PAUSE
  // ===============================

  estado_pause = (ev) => {

    if (ev === ENTRY) {
      this.myTimer.stop();
    }

    else if (ev === EVENTS.DEC || ev === EVENTS.INC) {

      // Detectar secuencia A-B-A
      if (this.checkSequence(ev)) {
        this.transitionTo(this.estado_config);
        return;
      }

      // Si solo fue A → reanuda
      if (ev === EVENTS.DEC) {
        this.transitionTo(this.estado_armed);
      }
    }
  };

  // ===============================
  // ESTADO TIMEOUT
  // ===============================

  estado_timeout = (ev) => {

    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
    }

    else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  };

  // ===============================
  // DETECTOR DE SECUENCIA A-B-A
  // ===============================

  checkSequence(ev) {

    this.sequence.push(ev);

    if (this.sequence.length > 3) {
      this.sequence.shift();
    }

    if (
      this.sequence.length === 3 &&
      this.sequence[0] === EVENTS.DEC &&
      this.sequence[1] === EVENTS.INC &&
      this.sequence[2] === EVENTS.DEC
    ) {
      this.sequence = []; // limpiar buffer
      return true;
    }

    return false;
  }
}

// ===============================
// SETUP
// ===============================

function setup() {

  createCanvas(windowWidth, windowHeight);

  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );

  textAlign(CENTER, CENTER);

  renderer.set(
    temporizador.estado_config,
    () => drawConfig(temporizador.configValue)
  );

  renderer.set(
    temporizador.estado_armed,
    () => drawArmed(
      temporizador.remainingSeconds,
      temporizador.totalSeconds
    )
  );

  renderer.set(
    temporizador.estado_pause,
    () => drawPause(temporizador.remainingSeconds)
  );

  renderer.set(
    temporizador.estado_timeout,
    () => drawTimeout()
  );

  connectBtn = createButton("Conectar Micro:bit");
  connectBtn.position(20, 20);
  connectBtn.mousePressed(connectMicrobit);
}

// ===============================
// SERIAL MICRO:BIT (ROBUSTO)
// ===============================

async function connectMicrobit() {

  try {
    port = await navigator.serial.requestPort();
    await port.open({ baudRate: 115200 });

    connectBtn.html("Micro:bit Conectado");

    readLoop();

  } catch (err) {
    console.error("Error al conectar:", err);
  }
}

async function readLoop() {

  reader = port.readable.getReader();
  const decoder = new TextDecoder();

  while (true) {

    const { value, done } = await reader.read();

    if (done) {
      reader.releaseLock();
      break;
    }

    if (value) {

      const text = decoder.decode(value);
      const lines = text.split("\n");

      for (let line of lines) {
        const clean = line.trim();

        if (clean.length > 0) {
          console.log("Recibido:", clean);
          handleMicrobitEvent(clean);
        }
      }
    }
  }
}

function handleMicrobitEvent(event) {

  if (event === "A")
    temporizador.postEvent(EVENTS.DEC);

  if (event === "B")
    temporizador.postEvent(EVENTS.INC);

  if (event === "S")
    temporizador.postEvent(EVENTS.START);
}

// ===============================
// DRAW LOOP
// ===============================

function draw() {

  temporizador.update();

  let drawFunction = renderer.get(temporizador.currentState);

  if (drawFunction) {
    drawFunction();
  }
}

// ===============================
// RENDER
// ===============================

function drawConfig(val) {

  background(20, 40, 80);
  fill(255);
  textSize(120);
  text(val, width / 2, height / 2);

  textSize(18);
  text("A(-) B(+) S(start)", width / 2, height / 2 + 100);
}

function drawArmed(val, total) {

  background(20);

  noFill();
  strokeWeight(20);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, 250);

  stroke(255, 150, 0);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 250, 250, -HALF_PI, angle - HALF_PI);

  fill(255);
  noStroke();
  textSize(100);
  text(val, width / 2, height / 2);
}

function drawPause(val) {

  background(40, 40, 80);
  fill(255);
  textSize(100);
  text(val, width / 2, height / 2);

  textSize(25);
  text("PAUSA", width / 2, height / 2 + 100);
}

function drawTimeout() {

  background(255, 0, 0);
  fill(255);
  textSize(100);
  text("¡TIEMPO!", width / 2, height / 2);
}

// ===============================
// TECLADO
// ===============================

function keyPressed() {

  if (key === "a" || key === "A")
    temporizador.postEvent(EVENTS.DEC);

  if (key === "b" || key === "B")
    temporizador.postEvent(EVENTS.INC);

  if (key === "s" || key === "S")
    temporizador.postEvent(EVENTS.START);
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```

index

``` js

<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Temporizador Micro:bit</title>

    <link rel="stylesheet" type="text/css" href="style.css">
    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/p5.js"></script>
    <script src="https://unpkg.com/@gohai/p5.webserial@^1/libraries/p5.webserial.js"></script>
  </head>

  <body>
    <script src="fsm.js"></script>
    <script src="sketch.js"></script>
  </body>
</html>

```

fsm (p5.js)

``` js

const ENTRY = "ENTRY";
const EXIT = "EXIT";

class Timer {
  constructor(owner, eventToPost, duration) {
    this.owner = owner;
    this.event = eventToPost;
    this.duration = duration;
    this.startTime = 0;
    this.active = false;
  }

  start(newDuration = null) {
    if (newDuration !== null) this.duration = newDuration;
    this.startTime = millis();
    this.active = true;
  }

  stop() {
    this.active = false;
  }

  update() {
    if (this.active && millis() - this.startTime >= this.duration) {
      this.active = false;
      this.owner.postEvent(this.event);
    }
  }
}

class FSMTask {
  constructor() {
    this.queue = [];
    this.timers = [];
    this.state = null;
  }

  postEvent(ev) {
    this.queue.push(ev);
  }

  addTimer(event, duration) {
    let t = new Timer(this, event, duration);
    this.timers.push(t);
    return t;
  }

  transitionTo(newState) {
    if (this.state) this.state(EXIT);
    this.state = newState;
    this.state(ENTRY);
  }

  update() {
    for (let t of this.timers) {
      t.update();
    }
    while (this.queue.length > 0) {
      let ev = this.queue.shift();
      if (this.state) this.state(ev);
    }
  }
}

```

style 

``` js

html, body {
  margin: 0;
  padding: 0;
}

canvas {
  display: block;
}

```

Codigo del microbit

``` py

from microbit import *

uart.init(baudrate=115200)
display.show(Image.BUTTERFLY)

while True:
    if button_a.was_pressed():
        uart.write('A')
        
    if button_b.was_pressed():
        uart.write('B')
        
    if accelerometer.was_gesture('shake'):
        uart.write('S')
        

```

R/ El mayor problema que me encontre fue a la hora de que registre los comandos del microbit por el tema de las librerias y de que los detectara igual a las teclas del teclado.

Finalizacion Actividad 4

## Bitácora de reflexión

### Actividad 5

Codigo del microbit que recibe

``` py
from microbit import *
import radio

# Configurar radio
radio.config(group=77)
radio.on()

# Configuración serial
uart.init(baudrate=115200)

while True:
    # Eventos locales
    if button_a.was_pressed():
        uart.write("A")
    if button_b.was_pressed():
        uart.write("B")
    if accelerometer.was_gesture("shake"):
        uart.write("S")

    # Eventos remotos vía radio
    message = radio.receive()
    if message:
        uart.write(message + "\n")

    sleep(20)

```

Codigo del microbit que manda

``` py

from microbit import *
import radio

radio.config(group=77)
radio.on()

while True:
    if button_a.was_pressed():
        radio.send("A")
    if button_b.was_pressed():
        radio.send("B")
    if accelerometer.was_gesture("shake"):
        radio.send("S")

    sleep(20)

```

Codigo de p5.js

``` js

// ===============================
// VARIABLES GLOBALES
// ===============================

let port;
let reader;
let connectBtn;

const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",        // Botón A
  INC: "B",        // Botón B
  START: "S",      // Shake
  TICK: "Timeout",
};

let temporizador;
let renderer = new Map();

// ===============================
// CLASE TEMPORIZADOR (FSM)
// ===============================

class Temporizador extends FSMTask {

  constructor(minValue, maxValue, defaultValue) {
    super();

    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;

    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    this.sequence = [];   // buffer para detectar A-B-A

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);

    this.transitionTo(this.estado_config);
  }

  get currentState() {
    return this.state;
  }

  // ===============================
  // ESTADO CONFIG
  // ===============================

  estado_config = (ev) => {

    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
      this.sequence = [];
    }

    else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue)
        this.configValue--;
    }

    else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue)
        this.configValue++;
    }

    else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  };

  // ===============================
  // ESTADO ARMED (corriendo)
  // ===============================

  estado_armed = (ev) => {

    if (ev === ENTRY) {
      this.sequence = [];
      this.myTimer.start();
    }

    else if (ev === EVENTS.TICK) {

      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;

        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start();
        }
      }
    }

    else if (ev === EVENTS.DEC || ev === EVENTS.INC) {

      // Detectar secuencia A-B-A
      if (this.checkSequence(ev)) {
        this.transitionTo(this.estado_config);
        return;
      }

      // Si solo fue A (sin completar secuencia) → pausa
      if (ev === EVENTS.DEC) {
        this.transitionTo(this.estado_pause);
      }
    }

    else if (ev === EXIT) {
      this.myTimer.stop();
    }
  };

  // ===============================
  // ESTADO PAUSE
  // ===============================

  estado_pause = (ev) => {

    if (ev === ENTRY) {
      this.myTimer.stop();
    }

    else if (ev === EVENTS.DEC || ev === EVENTS.INC) {

      // Detectar secuencia A-B-A
      if (this.checkSequence(ev)) {
        this.transitionTo(this.estado_config);
        return;
      }

      // Si solo fue A → reanuda
      if (ev === EVENTS.DEC) {
        this.transitionTo(this.estado_armed);
      }
    }
  };

  // ===============================
  // ESTADO TIMEOUT
  // ===============================

  estado_timeout = (ev) => {

    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
    }

    else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  };

  // ===============================
  // DETECTOR DE SECUENCIA A-B-A
  // ===============================

  checkSequence(ev) {

    this.sequence.push(ev);

    if (this.sequence.length > 3) {
      this.sequence.shift();
    }

    if (
      this.sequence.length === 3 &&
      this.sequence[0] === EVENTS.DEC &&
      this.sequence[1] === EVENTS.INC &&
      this.sequence[2] === EVENTS.DEC
    ) {
      this.sequence = []; // limpiar buffer
      return true;
    }

    return false;
  }
}

// ===============================
// SETUP
// ===============================

function setup() {

  createCanvas(windowWidth, windowHeight);

  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );

  textAlign(CENTER, CENTER);

  renderer.set(
    temporizador.estado_config,
    () => drawConfig(temporizador.configValue)
  );

  renderer.set(
    temporizador.estado_armed,
    () => drawArmed(
      temporizador.remainingSeconds,
      temporizador.totalSeconds
    )
  );

  renderer.set(
    temporizador.estado_pause,
    () => drawPause(temporizador.remainingSeconds)
  );

  renderer.set(
    temporizador.estado_timeout,
    () => drawTimeout()
  );

  connectBtn = createButton("Conectar Micro:bit");
  connectBtn.position(20, 20);
  connectBtn.mousePressed(connectMicrobit);
}

// ===============================
// SERIAL MICRO:BIT (ROBUSTO)
// ===============================

async function connectMicrobit() {

  try {
    port = await navigator.serial.requestPort();
    await port.open({ baudRate: 115200 });

    connectBtn.html("Micro:bit Conectado");

    readLoop();

  } catch (err) {
    console.error("Error al conectar:", err);
  }
}

async function readLoop() {

  reader = port.readable.getReader();
  const decoder = new TextDecoder();

  while (true) {

    const { value, done } = await reader.read();

    if (done) {
      reader.releaseLock();
      break;
    }

    if (value) {

      const text = decoder.decode(value);
      const lines = text.split("\n");

      for (let line of lines) {
        const clean = line.trim();

        if (clean.length > 0) {
          console.log("Recibido:", clean);
          handleMicrobitEvent(clean);
        }
      }
    }
  }
}

function handleMicrobitEvent(event) {

  if (event === "A")
    temporizador.postEvent(EVENTS.DEC);

  if (event === "B")
    temporizador.postEvent(EVENTS.INC);

  if (event === "S")
    temporizador.postEvent(EVENTS.START);
}

// ===============================
// DRAW LOOP
// ===============================

function draw() {

  temporizador.update();

  let drawFunction = renderer.get(temporizador.currentState);

  if (drawFunction) {
    drawFunction();
  }
}

// ===============================
// RENDER
// ===============================

function drawConfig(val) {

  background(20, 40, 80);
  fill(255);
  textSize(120);
  text(val, width / 2, height / 2);

  textSize(18);
  text("A(-) B(+) S(start)", width / 2, height / 2 + 100);
}

function drawArmed(val, total) {

  background(20);

  noFill();
  strokeWeight(20);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, 250);

  stroke(255, 150, 0);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 250, 250, -HALF_PI, angle - HALF_PI);

  fill(255);
  noStroke();
  textSize(100);
  text(val, width / 2, height / 2);
}

function drawPause(val) {

  background(40, 40, 80);
  fill(255);
  textSize(100);
  text(val, width / 2, height / 2);

  textSize(25);
  text("PAUSA", width / 2, height / 2 + 100);
}

function drawTimeout() {

  background(255, 0, 0);
  fill(255);
  textSize(100);
  text("¡TIEMPO!", width / 2, height / 2);
}

// ===============================
// TECLADO
// ===============================

function keyPressed() {

  if (key === "a" || key === "A")
    temporizador.postEvent(EVENTS.DEC);

  if (key === "b" || key === "B")
    temporizador.postEvent(EVENTS.INC);

  if (key === "s" || key === "S")
    temporizador.postEvent(EVENTS.START);
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

```
