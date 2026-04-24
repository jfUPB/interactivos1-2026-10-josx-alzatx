# Unidad 7

## Bitácora de proceso de aprendizaje

## Actividad 1

### Paso 0

¿Qué diferencia hay entre un evento musical y un mensaje de control?

R/ El evento musical funciona gracias al timestamp para que se puedan ejecutar los mensajes en el orden correcto, reaccionando a los sonidos enviados por Strudel. Mientras que los mensajes de control se pueden ir modificando a medida que se ejecuta el sistema, sin necesidad de tener un registro de tiempo.

¿Qué quiere decir que un parámetro del sistema sea persistente?

R/ Que sea persistente se refiere a que a menos de que algo lo cambie, el valor de ese parametro va a mantenerse intacto.

¿Qué partes del sistema de la unidad 6 permanecen intactas en este nuevo caso?

R/ El bridgeServer.js mantiene su funcion de que unicamente ejecuta los mensajes. 
El StrudelAdapter no se modifica. 
El bridgeClient.js conserva su estructura base. 
La FSMTask y sus estados permanecen. 
El drawRunning sigue siendo la única capa que dibuja. 
La arquitectura de capas (adapter → bridge → cliente → FSM → render) se conserva completamente.

### Paso 1

Si Open Stage Control fuera “el dispositivo” de esta unidad, ¿Cuál sería su protocolo?

R/ Open Stage Control envía mensajes usando el protocolo OSC (Open Sound Control) sobre transporte UDP. Un mensaje OSC tiene tres partes: un address que es una cadena tipo ruta (/rgb_bd), un type tag string que describe los tipos de los argumentos (",iff" para enteros y flotantes), y los argumentos en binario big-endian. El adapter recibe estos paquetes crudos en el puerto UDP 9000 y los parsea manualmente sin librerías externas.

¿Qué parte de ese protocolo te interesa conservar y cuál te gustaría normalizar?

R/ Se conserva el address como identificador semántico del control, ya que es la forma natural de nombrar parámetros en OSC. Se eliminan los argumentos en binario que se mandan y se traducen a un lenguaje mas sencillo.

### Paso 2

¿Por qué no conviene procesar un mensaje OSC igual que un mensaje de Strudel?

R/ Un mensaje de Strudel contiene un timestamp que indica el momento exacto en que debe ocurrir una acción visual. Si ese timestamp se pierde o se ignora, la sincronización con la música se rompe. Un mensaje OSC no tiene timestamp porque no representa un evento que ocurre en un instante: representa un cambio de estado que debe permanecer activo indefinidamente hasta que otro mensaje lo reemplace. Procesarlo como evento temporizado significaría que el parámetro se activaría y luego desaparecería, perdiendo la idea de control persistente.

¿Qué variables del sistema deberían vivir como estado persistente y no como evento efímero?

R/ Las variables controladas por OSC: oscColor (color del bombo), oscScale (escala global de animaciones) y oscBackgroundActive (activación de la capa de fondo). Estas viven en el PainterTask y son leídas en cada frame por drawRunning, no en respuesta a un evento específico.

### Paso 3

¿Qué componentes de la arquitectura necesitas conservar obligatoriamente?

R/ La separación en capas: adapter, bridge, cliente, FSM, estado, render. El principio de que el bridge no tiene lógica de dominio. El principio de que drawRunning solo lee estado y no interpreta mensajes de red. La cola de eventos temporizados de Strudel y su procesamiento en processStrudelQueue.

¿Qué nuevas estructuras de estado necesitas introducir para soportar control paramétrico?

R/ Un objeto de estado persistente de controles OSC dentro del PainterTask, separado de la cola de eventos. Este objeto contiene las variables que los widgets de Open Stage Control modifican y que drawRunning consume en cada frame junto con las animaciones activas de Strudel.

## Bitácora de aplicación 

## Actividad 2

Cómo configuraste Open Stage Control;

R/

Qué widgets decidiste usar y por qué;

R/ ELegi:

1. Un widget rgb para cambiar el color del bombo (debido a que es el mas facil de revisar).
2. Un widget para cambiar el tamaño del bombo (slider).
3. Un widget que elige si el fondo se dibuja o no (boton).

Qué estructura final de mensaje decidiste usar para los controles;

R/

Cómo conectaste bridgeClient.js, FSMTask, updateLogic y drawRunning;

R/

Cómo integraste ambas fuentes de datos en el mismo frontend;

R/

Qué pruebas hiciste para verificar que el control paramétrico funciona sin romper la sincronización de Strudel;

R/

Qué problemas encontraste y cómo los solucionaste.

R/ 

## Bitácora de reflexión
