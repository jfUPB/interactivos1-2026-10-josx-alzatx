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

R/ Para el OSC, primero le agregue los valores de a que servidor debe llegar (por ejemplo el 8082), tambien desde cual le llega. Y cuando ya tiene esos campos con la información, se crea el archivo de OSCUI.json y se le agregan todos los componentes a utilizar, y se carga ese archivo en el OSC.

Qué widgets decidiste usar y por qué;

R/ ELegi:

1. Un widget rgb para cambiar el color del bombo (debido a que es el mas facil de revisar).
2. Un widget para cambiar el tamaño del bombo (slider).
3. Un widget que elige si el fondo se dibuja o no (boton).

Qué estructura final de mensaje decidiste usar para los controles;

R/ Se creo un campo type el cual permite al cliente diferenciar entre si es de Strudel o de OSC. y se creo el campo address que identifica cual de los parametros va a ser actualizado. Y en el campo de args se le dan al cliente los valores normalizados para que no necesite ningun procesamiento adicional.

Cómo conectaste bridgeClient.js, FSMTask, updateLogic y drawRunning;

R/ Primero el websocket envia los datos al cliente, este le envía al sketch el evento, luego el FSMTask recibe el evento y realiza los cambios en el parametro correspondiente según los datos. Y al final siguiendo el address detecta que parametro cambia.

Cómo integraste ambas fuentes de datos en el mismo frontend;

R/ Se mantiene una sola instancia de BridgeClient conectada al puerto 8081. El bridge envía ambos tipos de mensaje por el mismo canal. bridgeClient.js los separa por el type: los mensajes strudel van a onStrudel() y los mensajes OSC van a onOsc(). Cada callback publica su propio tipo de evento hacia la FSMTask. Dentro del PainterTask los dos flujos viven en estructuras separadas: Strudel alimenta la cola de eventos y los controles OSC alimentan variables de estado persistente. drawRunning consume ambas estructuras simultáneamente en cada frame.

Qué pruebas hiciste para verificar que el control paramétrico funciona sin romper la sincronización de Strudel;

R/ Se verificó que al mover los widgets de Open Stage Control mientras Strudel estaba corriendo, las animaciones rítmicas continuaban apareciendo en sus tiempos correctos sin interrupciones ni retrasos. Se probó cada control de forma independiente: primero el color del bombo, luego la escala y finalmente el toggle de fondo. Se comprobó que al activar y desactivar el fondo repetidamente la cola de eventos de Strudel no se vaciaba ni se desordenaba. Se verificó en la consola del navegador que los eventos OSC y STRUDEL llegaban por caminos separados sin interferirse. Y como voy a mencionar acontinuación, tambien se verifico mediante el Git Bash, que los mensajes se estuvieran recibiendo.

Qué problemas encontraste y cómo los solucionaste.

R/ El único problema encontrado fue que los mensajes OSC no llegaban al sistema visual aunque Open Stage Control estaba correctamente configurado. Para verificarlo se agregó un log de bytes crudos en el método connect() del OpenStageControlAdapter:

``js
  console.log("[OSC RAW] Bytes recibidos:", buf.length, buf.toString("hex"));
``
Esto permitió confirmar que los paquetes sí llegaban al puerto correcto (9000) pero el parser los estaba rechazando. Con esa información se identificó y corrigió el problema en el parser, tras lo cual los mensajes comenzaron a procesarse y reflejarse correctamente en el sistema visual.

## Bitácora de reflexión

### Actividad 3

Realiza un diagrama detallado del flujo de datos completo de tu sistema. Debe incluir al menos:

Strudel;
Open Stage Control;
Adapter de Strudel;
Adapter de Open Stage Control;
bridgeServer.js;
bridgeClient.js o cliente equivalente;
Cola de eventos;
Estado persistente de controles;
Render visual.

R/
<img width="1401" height="748" alt="image" src="https://github.com/user-attachments/assets/f4b09989-3d76-4cd2-9c5a-0c8c67f9064a" />

Compara en una tabla las fuentes de datos que has trabajado en las unidades 4, 5, 6 y 7. Compara al menos:

Fuente de datos;
Formato del mensaje;
Tipo de dato que produce;
Problema técnico principal;
Lugar donde ocurre la traducción del dato;
Papel del tiempo;
Relación con el estado del sistema.

R/

Explica por qué Open Stage Control no debe tratarse igual que Strudel dentro de la arquitectura.

R/ El Strudel se basa en una cola de eventos para definir en que momento se realiza cada evento, mientras qe en el OSC se usaban parametros persistentes los cuales cambiaban los visuales y unicamente varian si se realiza el cambio desde el Open Stage Control. Si se trataran de la misma manera, los visuales del strudel no se realizarian con el tiempo de la musica, o los visuales del OSC cambiarian frecuentemente.

Justifica los tres controles que elegiste:

Qué parámetro modifican;
Por qué ese parámetro es relevante;
Cómo se refleja en el comportamiento visual.

R/
  - rgb_bd:
      modifica el color del bombo. Es relevante porque el bombo es el elemento visual más prominente del sistema — ocupa el centro de la pantalla y tiene el mayor tamaño. Cambiarlo en tiempo real permite modificar la            paleta cromática dominante de la composición visual sin interrumpir la música. Se refleja en el color del círculo que se expande desde el centro cada vez que Strudel dispara un evento de bombo.

  - scale:
      modifica la escala global de todas las animaciones. Es relevante porque permite ajustar la densidad visual del sistema en vivo — con valores altos las formas llenan la pantalla, con valores bajos se vuelven                discretas. Afecta simultáneamente a todos los tipos de sonido. Se refleja en el tamaño de todas las formas que aparecen en pantalla — círculos, rectángulos y cuadrados se escalan proporcionalmente.

  - background_toggle:
      activa o desactiva la capa de fondo semitransparente. Es relevante porque cambia radicalmente el comportamiento visual: con fondo activo las formas se desvanecen creando un efecto de persistencia; sin fondo las            formas se acumulan indefinidamente creando saturación visual. Es el control con mayor impacto perceptual ya que modifica la estética global del sistema con un solo click.

Si tuvieras que integrar una tercera fuente de control en el futuro, ¿Qué partes de tu arquitectura actual conservarías y cuáles extenderías?

R/ Se conservaría la arquitectura de capas completa, el contrato de mensajes normalizados, la separación entre estado temporizado y estado persistente, y el principio de que drawRunning solo lee estado. Se extendería únicamente el PainterTask con nuevas variables de estado si la fuente produce parámetros persistentes, o la cola de eventos si produce eventos temporizados. Se crearía un nuevo adapter que implemente la misma interfaz que StrudelAdapter y OpenStageControlAdapter, y se registraría en bridgeServer.js con su propio callback onData.

### Fin Actividad 3
