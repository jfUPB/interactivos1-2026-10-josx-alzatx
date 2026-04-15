# Unidad 6

## Bitácora de proceso de aprendizaje

## Actividad 1 ##

- ¿Cuál es la diferencia entre recibir un mensaje y ejecutarlo?

R/ Recibir un mensaje es unicamente que un componente detecte o guarde una señal o dato que le fue enviado. Sin que necesariamente cambie algo en el sistema. Mientras que ejecutarlo es cuando el componente actúa en respuesta e interpreta el contenido del mensaje, ya sea llamando el metodo que necesita o usandolo para alguna función del sistema.
  
- ¿Por qué un sistema audiovisual puede necesitar timestamp además de los datos del evento?

R/ Porque para los sistemas audiovisuales el cuándo es tan importante como el qué. Sin este dato, el sistema únicamente se enfocaría en qué cosas se hacen más que en cuándo las hacen, y para sistemas audiovisuales es muy importante saber cuándo deberían aparecer ciertas cosas.

- ¿Qué aspectos de la arquitectura de las unidades 4 y 5 permanecen intactos aunque ahora la fuente de datos ya no sea hardware?

R/ Lo que se conserva de las unidades anteriores es la forma en la que se procesa la información, debido a que al quitarse los adapters pero al usar la otra pagina el servidor va a seguir ejecutandose de la misma manera.

### Finalización Actividad 1 Paso 0 ###

### Paso 1 ###

- Si Strudel fuera “el dispositivo” de esta unidad, ¿Cuál sería su protocolo?

R/ Yo pienso que Strudel traduciría los sonidos en una especie de mensaje o de conjntos de digitos para enviarle esta información completa al programa (Creo que los enviaria todos organizados en algun orden)

- ¿Qué variables mínimas necesitarías extraer para poder construir una visualización útil?

R/ Necesitaria detectar: 

--  Que esta sonando
--  Cuando empieza a sonar
--  Que tanto tiempo dura este sonido
--  Cuanto dura la reproducción del sonido

### Finalización Actividad 1 Paso 1 ###

### Paso 2 ###

- ¿Qué problema resuelve la cola de eventos?

R/ La cola de eventos ayuda a que los eventos se realicen en el momento que debe ser sin necesidad de que esto. Porque si no tuviera la cola de eventos, estos dependerian mas de la red para definir cuando se ejecutan mas que el momento real en el que deben realizarse según la musica.

- ¿Por qué esta capa no pertenece al bridge sino al lado que interpreta el evento?

R/ Porque el bridge unicamente esta ahi para enviar los mensajes de un lado para el otro, por lo tanto definir en que momentos se ejecuta el mensaje necesitamos de otra parte que se encargue de los eventos en general que constituyen al programa. Gracias a que esta separado es mucho más facil manejar el programa porque no queda con una sobrecarga de trabajo el bridge.

### Finalización Actividad 1 Paso 2 ###

### Paso 3 ###

- ¿Qué papel cumple el Adapter en U4 y U5?

R/ El adapter en la unidad 4 usaba un protocolo ASCII para traducir toda la información que recibia del microbit y que el sistema pudiera ejecutarla. En la unidad 5 se buscaba el mismo objetivo pero con el protocolo binario por lo tanto tambien era necesario el framing para que pudiera indentificar cuando empezaba y cuando terminaba la información enviada.

- ¿Qué Adapter necesitas ahora para que los eventos de Strudel no entren “crudos” al sistema visual?

R/ Ahora es necesario un adapter que pueda recibir los datos que mencione anteriormente de Strudel (Duracion, tiempo, cuando inicia, que suena) y que los traduzca a un lenguaje que el sistema pueda interpretar y ejecutarlos en el sistema audiovisual.

### Finalización Actividad 1 Paso 3 ###

## Bitácora de aplicación 

### Actividad 2 ###

- ¿Cómo configuraste Strudel para emitir eventos?

R/

- ¿Qué estructura final de mensaje decidiste usar?

R/

- ¿Cómo conectaste bridgeClient.js, FSMTask, updateLogic y drawRunning?

R/

- ¿Cómo separaste recepción, cola temporal y renderizado?

R/

- ¿Qué pruebas hiciste para verificar la sincronización?

R/

- ¿Qué problemas encontraste y cómo los solucionaste?

R/ 

## Bitácora de reflexión

### Actividad 3 ###

- Realiza un diagrama detallado del flujo de datos de tu sistema. Debe incluir al menos:

-- Strudel;
-- Adapter de Strudel;
-- WebSocket de salida;
-- bridgeServer.js;
-- bridgeClient.js;
-- Cola de eventos;
-- FSM o capa de estado;
-- Render visual.

R/
  
- Compara las unidades 4, 5 y 6 en una tabla. Compara al menos:

-- Fuente de datos;
-- Formato del mensaje;
-- Problema técnico principal;
-- Mecanismo de validación o control;
-- Lugar donde ocurre la traducción del dato;
-- Papel del tiempo o sincronización.
-- Explica por qué esta unidad sigue perteneciendo a la misma arquitectura del curso, aunque la fuente de datos ya no sea hardware físico.

R/

- Explica qué decisiones tomaste para traducir eventos musicales en visualidad. Justifica por qué tu mapeo visual tiene sentido.

R/


- Si tuvieras que integrar una tercera aplicación en el futuro, ¿Qué partes de tu arquitectura actual conservarías y cuáles cambiarías?

/R
