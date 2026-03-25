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


## Bitácora de reflexión
