# Unidad 8

## Bitácora de proceso de aprendizaje

### Actividad 1

- Diagrama inicial de la arquitectura.

R/ <img width="859" height="564" alt="image" src="https://github.com/user-attachments/assets/350e06f7-3431-4376-ae18-288b7f541731" />

- Los adapters que se van a usar.

R/ 1. StrudelAdapter.js
```js
// StrudelAdapter.js
// Recibe mensajes crudos de Strudel por WebSocket (puerto 8080),
// los normaliza y los entrega al bridgeServer para reenvío.

const { WebSocketServer } = require("ws");

const STRUDEL_PORT = 8080;

const BaseAdapter = require("./BaseAdapter");  

class StrudelAdapter extends BaseAdapter {     
  constructor({ verbose = false } = {}) {
    super();                                   
    this.verbose = verbose;
    this.onData = null;       // callback: (normalizedMsg) => void
    this.onConnected = null;
    this.onDisconnected = null;
    this.onError = null;
    this.connected = false;
    this._wss = null;
  }

  // Requerido por bridgeServer para obtener detalle de conexión
  getConnectionDetail() {
    return `strudel ws://localhost:${STRUDEL_PORT}`;
  }

  connect() {
    return new Promise((resolve) => {
      this._wss = new WebSocketServer({ port: STRUDEL_PORT });

      this._wss.on("listening", () => {
        this.connected = true;
        this.onConnected?.(`Strudel WebSocket listening on port ${STRUDEL_PORT}`);
        resolve();
      });

      this._wss.on("connection", (ws) => {
        if (this.verbose) console.log("[StrudelAdapter] Strudel conectado");

        ws.on("message", (raw) => {
          try {
            const msg = JSON.parse(raw.toString("utf8"));
            const normalized = this._normalize(msg);
            if (normalized) {
              if (this.verbose) console.log("[StrudelAdapter] Evento normalizado:", normalized);
              this.onData?.(normalized);
            }
          } catch (e) {
            this.onError?.(`Error al parsear mensaje de Strudel: ${e.message}`);
          }
        });

        ws.on("close", () => {
          if (this.verbose) console.log("[StrudelAdapter] Strudel desconectado");
        });
      });

      this._wss.on("error", (e) => {
        this.onError?.(`StrudelAdapter error: ${e.message}`);
      });
    });
  }

  disconnect() {
    return new Promise((resolve) => {
      if (this._wss) {
        this._wss.close(() => {
          this.connected = false;
          this.onDisconnected?.("StrudelAdapter cerrado");
          resolve();
        });
      } else {
        resolve();
      }
    });
  }

  // Transforma el mensaje crudo de Strudel en un objeto limpio
  _normalize(msg) {
    if (!msg || !Array.isArray(msg.args)) return null;

    // Convertir la lista plana de args a un objeto clave-valor
    const params = {};
    for (let i = 0; i < msg.args.length; i += 2) {
      params[msg.args[i]] = msg.args[i + 1];
    }

    // Verificar que tenga los datos mínimos necesarios
    if (!params.s || !msg.timestamp) return null;

    return {
      type: "strudel",
      timestamp: msg.timestamp,
      payload: {
        s: params.s,
        delta: params.delta ?? 0.25,
        cps: params.cps ?? 0.5,
        cycle: params.cycle ?? 0,
      }
    };
  }

  // No aplica para Strudel pero requerido por la interfaz del bridge
  async handleCommand(msg) {}
}

module.exports = StrudelAdapter;
```

   2. OpenStageControlAdapter.js
```js
// OpenStageControlAdapter.js
// Recibe mensajes OSC de Open Stage Control por UDP,
// los normaliza y los entrega a bridgeServer para reenvío por WebSocket.
//
// Protocolo: Open Stage Control envía paquetes OSC (UDP) al puerto OSC_PORT.
// Este adapter decodifica los paquetes crudos sin dependencia de librerías pesadas,
// usando parsing manual del formato OSC 1.0 (string de address + type tag string + args).

const dgram = require("dgram");

const OSC_PORT = 9000; // Puerto en el que este adapter escucha mensajes OSC

const BaseAdapter = require("./BaseAdapter"); 

class OpenStageControlAdapter extends BaseAdapter { 
  constructor({ verbose = false } = {}) {
    super();                                        
    this.verbose = verbose;
    this.onData = null;        // callback: (normalizedMsg) => void
    this.onConnected = null;
    this.onDisconnected = null;
    this.onError = null;
    this.connected = false;
    this._socket = null;
  }

  getConnectionDetail() {
    return `OSC UDP udp://0.0.0.0:${OSC_PORT}`;
  }

  connect() {
    return new Promise((resolve, reject) => {
      this._socket = dgram.createSocket("udp4");

      this._socket.on("error", (e) => {
        this.onError?.(`OpenStageControlAdapter error: ${e.message}`);
        reject(e);
      });

      this._socket.on("listening", () => {
        this.connected = true;
        const addr = this._socket.address();
        this.onConnected?.(`OSC UDP listening on ${addr.address}:${addr.port}`);
        resolve();
      });

      this._socket.on("message", (buf) => {
        console.log("[OSC RAW] Bytes recibidos:", buf.length, buf.toString("hex"));
        try {
          const msg = this._parseOSC(buf);
          console.log("[OSC PARSED]", JSON.stringify(msg));
          if (!msg) return;

          const normalized = this._normalize(msg);
          if (!normalized) return;

          if (this.verbose) {
            console.log("[OSCAdapter] Mensaje normalizado:", JSON.stringify(normalized));
          }

          this.onData?.(normalized);
        } catch (e) {
          this.onError?.(`Error al parsear OSC: ${e.message}`);
        }
      });

      this._socket.bind(OSC_PORT);
    });
  }

  disconnect() {
    return new Promise((resolve) => {
      if (this._socket) {
        this._socket.close(() => {
          this.connected = false;
          this.onDisconnected?.("OpenStageControlAdapter cerrado");
          resolve();
        });
      } else {
        resolve();
      }
    });
  }

  // ─── PARSER OSC MANUAL ────────────────────────────────────────────────────
  // Formato OSC 1.0:
  //   [address string, null-padded to múltiplo de 4]
  //   [type tag string ",iff" etc., null-padded]
  //   [argumentos en binario big-endian]

  _parseOSC(buf) {
    let offset = 0;

    // Leer address string
    const { str: address, next: o1 } = this._readString(buf, offset);
    offset = o1;

    // Leer type tag string (empieza con ',')
    const { str: typeTag, next: o2 } = this._readString(buf, offset);
    offset = o2;

    if (!typeTag.startsWith(",")) return null;
    const types = typeTag.slice(1); // quitar la coma

    // Leer argumentos según tipos
    const args = [];
    for (const t of types) {
      if (t === "i") {
        args.push(buf.readInt32BE(offset));
        offset += 4;
      } else if (t === "f") {
        args.push(buf.readFloatBE(offset));
        offset += 4;
      } else if (t === "s") {
        const { str, next } = this._readString(buf, offset);
        args.push(str);
        offset = next;
      } else if (t === "T") {
        args.push(true);
      } else if (t === "F") {
        args.push(false);
      }
      // otros tipos ignorados
    }

    return { address, args };
  }

  _readString(buf, offset) {
    let end = offset;
    while (end < buf.length && buf[end] !== 0) end++;
    const str = buf.toString("utf8", offset, end);
    // avanzar al siguiente múltiplo de 4
    const next = Math.ceil((end + 1) / 4) * 4;
    return { str, next };
  }

  // ─── NORMALIZACIÓN ────────────────────────────────────────────────────────
  // Convierte el mensaje OSC crudo en el contrato estable del sistema:
  // { type: "osc", payload: { address, args } }

  _normalize(msg) {
    if (!msg || !msg.address || !Array.isArray(msg.args)) return null;

    return {
      type: "osc",
      payload: {
        address: msg.address,
        args: msg.args
      }
    };
  }

  async handleCommand(_cmd) {}
}

module.exports = OpenStageControlAdapter;

```
   4. MicrobitASCIIAdapter.js
```js
const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error { }

function parseCsvLine(line) {
  const values = line.trim().split(",");
  if (values.length !== 4) throw new ParseError(`Expected 4 values, got ${values.length}`);

  const x = Number(values[0]);
  const y = Number(values[1]);
  const btnA = String(values[2]).trim().toLowerCase();
  const btnB = String(values[3]).trim().toLowerCase();

  if (!Number.isFinite(x) || !Number.isFinite(y)) throw new ParseError("Invalid numeric data");
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) throw new ParseError("Out of expected range");
  if (!["true", "false"].includes(btnA) || !["true", "false"].includes(btnB)) throw new ParseError("Invalid button data");

  return { x: x | 0, y: y | 0, btnA: btnA === "true", btnB: btnB === "true" };
}


class MicrobitAsciiAdapter extends BaseAdapter {
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
    if (!this.path) throw new Error("serialPort is required for microbit device mode");

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
        const parsed = parseCsvLine(line);
        this.onData?.(parsed);
      } catch (e) {
        if (e instanceof ParseError) {
          if (this.verbose) console.log("Bad data:", e.message, "raw:", line);
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

module.exports = MicrobitAsciiAdapter;

```

- El contrato de mensajes de cada fuente

R/ micro:bit — envía líneas CSV con 4 valores por línea: x,y,btnA,btnB

x, y: enteros del acelerómetro entre −2048 y 2047
btnA, btnB: cadenas "true" o "false"
Ejemplo: -340,112,false,true
Frecuencia: 20 lecturas por segundo (sleep_ms(50) en el Python del micro:bit)

Strudel — envía objetos JSON con lista plana de argumentos. El StrudelAdapter los normaliza a:
{ type: "strudel", timestamp, payload: { s, delta, cps, cycle } }

s: nombre del sonido ("bd", "hh", "sd", etc.)
delta: duración del evento en segundos
cps: ciclos por segundo (tempo)

Open Stage Control — envía paquetes OSC por UDP. El OpenStageControlAdapter los parsea manualmente y entrega:
{ type: "osc", payload: { address, args[] } }

/rgb_bd → [r, g, b]: color del bombo (0–255)
/scale → [float 0–1]: escala global de los elementos
/background_toggle → [0 o 1]: activa/desactiva la estela de fondo

- Pruebas tecnicas basicas de integración

R/ Para el microbit, le agregue una parte al sketch para que saliera la posición del acelerometro y tambien desde el verbose en el git bash para revisar que estuviera recibiendo los botones.

Para el strudel era que las visuales en el index fueran coordinadas con el sonido. Tambien en el git bash que recibiera los valores normalizados.

Para el OSC, que los visuales cambiaran de color o se modificaran los valores a la hora de los visuales. Tambien con el git bash revisar que estuviera recibiendo los cambios de los parametros.

- Errores encontrados y como se solucionaron

R/ 
Error 1 — Puerto del micro:bit incorrecto
Al cambiar de computador, el microbit quedó asignado al COM20 en lugar del COM15 configurado en bridgeServer.js. El sistema arrancaba sin error visible pero no llegaban datos del micro:bit. Se rastreó el path en la instanciación de MicrobitASCIIAdapter dentro de bridgeServer.js y se actualizó de "COM15" a "COM20".

Error 2 — Conflicto de puerto OSC (EADDRINUSE :9000)
Al lanzar node bridgeServer.js con Open Stage Control ya corriendo, el sistema fallaba con error fatal porque ambos competían por el puerto UDP 9000. Solución: se cambió el osc_port y lo deje en blanco. Con esto ya no se peleaban por el servidor.

## Bitácora de aplicación 


## Bitácora de reflexión
