# Unidad 5
## Bitácora de proceso de aprendizaje

### Actividad 1 ###

### Paso 1 ###

¿Qué ventajas y desventajas ves en usar un formato binario en lugar de texto ASCII?
- R/  El sistema binario es mucho mejor que el sistema ASCII a la hora de recibir una amplia variedad de datos. Debido a que este ahorra demasiados bits en comparación de la manera que codifica el ASCII. Pero en el tema de las desventajas, el sistema binario es mas complicado de entender si no posees una herramienta para leerlo  

Si xValue=500, yValue=524, aState=True, bState=False, ¿cómo se vería el paquete en hexadecimal? (Pista: convierte cada valor según su tipo y anota los bytes en orden.)
- R/ ( 1F4 - 500) ( 20C - 524) ( 01 - True) ( 00 - False) ---> Respuesta Fnal: 01 F4 02 0C 01 00

### Paso 2 ###

¿Por qué el protocolo ASCII de la unidad anterior no tenía este problema de sincronización? (Pista: piensa en qué rol cumplía el carácter \n.)
- R/ Porque en el protocolo ASCII el \n. servía para separar cada linea de datos y que solo hubiera 1 forma correcta de leerlos. Mientras que con el binario si no especificas y comienza a leer los datos desde donde no deberia el resultado de los datos recibidos seria completamente diferente.

¿Por qué en binario no podemos usar \n como delimitador?
- R/ Porque en binario es mucho mas complicado que el protocolo lo detecte como un enter, entonces de pronto lo puede detectar como un caracter más.


## Bitácora de aplicación 

### Actividad 2 ###

¿como fue mi proceso para realizar la actividad?

R/ A la hora del proceso, la principal complicacion que tuve fue a la hora de el constructor. Debido a que cometia errores a la hora de nombrar las clases y cuando iban a ser llamadas no se llamaban correctamente. En otras ocasiones se me olvidaba cambiar la información del micro:bit Python y me asustaba porque crei que se me habia dañado el trabajo 🥹. Pero en general se me resulto facil de comprender debido a que su funcionamiento era relativamente similar al trabajo anterior solo poniendo atención en clase y revisando las maneras en las que el profe documentaba los datos binarios, logre finalizar.

Errores del checksum y pruebas de como recibe los datos:

<img width="437" height="111" alt="image" src="https://github.com/user-attachments/assets/5a29bd85-8e13-45dd-9f70-f9a77bc011f7" />

Codigo del microbitBinary

```js
const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class MicrobitBinaryAdapter extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = Buffer.alloc(0);
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
    this.buf = Buffer.alloc(0);
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
    console.log("RAW HEX:", chunk.toString("hex")); // ← solo agrega esta línea
    this.buf = Buffer.concat([this.buf, chunk]);

    while (this.buf.length >= 8) {

      // Buscar header 0xAA
      if (this.buf[0] !== 0xAA) {
        this.buf = this.buf.slice(1);
        continue;
      }

      const packet = this.buf.slice(0, 8);

      // Calcular checksum (bytes 1 a 6)
      let checksum = 0;
      for (let i = 1; i <= 6; i++) {
        checksum += packet[i];
      }
      checksum = checksum % 256;

      const receivedChecksum = packet[7];

      // ✅ CORRECCIÓN: warning siempre, sin depender de verbose
      if (checksum !== receivedChecksum) {
        console.warn(`[MicrobitBinary] Checksum inválido: esperado ${checksum}, recibido ${receivedChecksum}`);
        this.buf = this.buf.slice(1);
        continue;
      }

      try {
        const x = packet.readInt16BE(1);
        const y = packet.readInt16BE(3);
        const btnA = packet.readUInt8(5) === 1;
        const btnB = packet.readUInt8(6) === 1;

        if (x < -2048 || x > 2047 || y < -2048 || y > 2047) {
          throw new Error("Out of expected range");
        }

        this.onData?.({ x, y, btnA, btnB });

      } catch (e) {
        if (this.verbose) {
          console.warn("[MicrobitBinary] Error parseando paquete:", e.message);
        }
      }

      this.buf = this.buf.slice(8);
    }

    if (this.buf.length > 4096) {
      this.buf = Buffer.alloc(0);
    }
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = Buffer.alloc(0);
    this.onDisconnected?.("serial closed (event)");
  }
}

module.exports = MicrobitBinaryAdapter;
```


## Bitácora de reflexión

### Actividad 3 ###

1. Realiza una tabla comparativa entre el adapter ASCII que creaste en la Unidad 4 (MicrobitV2Adapter.js) y el adapter binario de esta unidad (MicrobitBinaryAdapter.js). Compara al menos:

Tamaño de cada paquete en bytes.
Mecanismo de delimitación/framing.
Mecanismo de verificación de integridad (checksum).
Complejidad de implementación del parser.
Facilidad de depuración (¿cuál es más fácil de leer con un terminal serial?).

R/ Tabla comparativa (ASCII - Binary)

| **Pregunta** | **ASCII** | **Binary** |
|---------|-------|--------|
| **Tamaño por paquete** | Varia según la información | 8 bytes |
| **Mecanismo de delimitación** |  .\n  | Por cada header (0xAA) y longitud |
| **Mecanismo de verificación (checksum)** | abs(x) + abs(y) + a + b | Suma de bytes 1–6 mod 256 |
| **Complejidad de implementación** | Baja | Media-Alta (buffers) |
| **Facilidad de depuración** | Facil debido a que es entendible | Dificil por el tema de los hexadecimales |


2. ¿Por qué la arquitectura desacoplada (patrón Adapter + Bridge + FSM) te permite añadir soporte para un protocolo completamente diferente sin modificar el frontend (sketch.js) ni el transporte (bridgeClient.js)?

R/ Porque en la forma que se trabajo en la arquitectura, cada parte tiene una responsabilidad separada, lo que permite que unicamente se enfoquen en esta. Porque el adapter se encarga de recibir la información (no importa de que dispositivo venga), luego esta es enviada al bridge (el cual no le interesa de donde viene la información, solo le importa que le llegue) y ya por parte del frontend, este unicamente recibe ya la información para activar el sistema.

3. ¿En qué situaciones del mundo real preferirías un protocolo binario sobre uno ASCII y viceversa? Justifica con ejemplos concretos.

R/ Inicialmente yo usaria el protocolo binario principalmente para sistemas que actuan en tiempo real, debido a que por el tamaño de los bits y la forma en la que trabaja facilita mucho el performance que debe realizar el servidor. En cambio para el protocolo ASCII es mucho mas util a la hora del debugging o de realizar prototipos gracias a la facilidad que tienen los desarrolladores de entender la información que este envia y no necesitan de ningún otro sistema para comprenderlo.

Ejemplos que usaría el protocolo binario:

- Controladores de videojuegos
- Sistemas interactivos en tiempo real
- Dispositivos de captura de imagen o de audio

Ejemplos que usaría el protocolo ASCII:

- Para verificar que el microbit este enviando la información y que el pc la reciba.
- Sistemas interactivos de sensores que se activen cada cierto tiempo
- Para revisar que los prototipos esten funcionando correctamente


4. Actualiza el diagrama de flujo de datos de la Unidad 4 para reflejar el protocolo binario. ¿Qué componentes cambiaron? ¿Qué componentes permanecieron intactos?

R/ Permanecio intacto en su mayoria debido a que ya tenia agregado el adapter del binario. Pero le agregue en rojo para que se distinga mas la forma en la que manejan los datos. 

<img width="1196" height="682" alt="image" src="https://github.com/user-attachments/assets/d916f679-2201-4ecc-91df-43f2535e2a25" />

