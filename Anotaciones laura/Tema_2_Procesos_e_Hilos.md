
# TEMA 2: PROCESOS E HILOS. #
## REPASO ##
##### PROCESO: #####
1. Programa en ejecución.
2. Instancia de un programa ejecutado en un computador.
3. La entidad que se puede asignar y ejecutar en un procesador.
4. La unidad de actividad que se caracteriza por la ejecución  de una secuencia de instrucciones, un estado actual y un conjunto de recursos del sistema asociado.
5. Entidad que consiste un número de elementos. Elementos esenciales:


 	 · Código de programa.

 	 · Conjunto de datos asociados a dicho código.

##### IMAGEN DE UN PROCESO:
1. Espacio en memoria para almacenar los distintos procesos.
2. Espacio en memoria para almacenar la **‘pila de ejecución’** (Estructura de tamaño intermedio, donde se almacenan datos temporales necesarios en un proceso.)
3. **‘Bloque de control de proceso’**, (Registro con el cual el Sistema Operativo, toma información sobre cada proceso.)

##### BLOQUE DE CONTROL DE PROCESO (PCB):

Estructura de datos asociada a un proceso que contiene la siguiente información sobre el mismo:

+ Identificador.
+ Estado.
+ Prioridad.
+ Contador de programa. 
+ Punteros a memoria.
+ Datos de contexto.
+ Información de estado de E/S. 
+ Información de auditoría. (Tiempo de procesador, o de reloj, registros contables…).

#####	ESTADOS BÁSICOS:

+ Nuevo.
+ Preparado.
+ Ejecutándose. 
+ Bloqueado.
+ Finalizado.

**Nuevo -> Preparado.** El sistema se encuentra preparado para ejecutar un nuevo proceso.

**Preparado -> Ejecutándose.** El planificador toma un proceso preparado y le cede el control de la CPU.

**Ejecutándose -> Finalizado.** El proceso ha acabado, o ha sido abortado.

**Ejecutándose -> Preparado.** Normalmente, el proceso ha agotado su tiempo de ejecución, y aun no ha acabado, por lo que debe volver a la cola de preparado.
 
**Ejecutándose -> Bloqueado.**  Un proceso se bloquea si solicita algo por lo que debe esperar.
 
**Bloqueado -> Preparado.** Un proceso bloqueado pasa a estado preparado cuando se produce el evento que estaba esperando. 
**Preparado -> Finalizado.**  Si un padre termina la ejecución de un hijo. 

**Bloqueado -> Finalizado.** Si un padre termina la ejecución de un hijo.

##### CAMBIO DE CONTEXTO:

Un cambio de contexto puede ocurrir en cualquier instante en el que el SO obtiene el control sobre el proceso actualmente en ejecución. 

1.	**Interrupciones:** Causadas por algún evento externo e independiente al proceso  actualmente en ejecución. El control se transfiere al manejador de interrupciones, que realiza determinadas tareas internas y que posteriormente salta a una rutina del SO, encargada de cada uno de los tipos de interrupciones en particular: 

	+ **Interrupción de reloj:** El SO determina si el proceso ha excedido o no la unidad máxima de tiempo de reloj (rodaja de tiempo) antes de ser interrumpido.
	+ **Interrupción de E/S:**  El SO determina qué acción de E/S ha ocurrido. Si la acción de E/S constituye un evento por el cal están esperando uno o más procesos, el SO mueve todos los procesos correspondientes al estado preparado. El SO puede decidir si reanuda la ejecución del proceso actualmente en estado Ejecutando o si lo expulsa para proceder con la ejecución de un proceso preparado de mayor prioridad.
	+ **Fallo de memoria:** El procesador se encuentra con una referencia a una dirección de memoria virtual que no se encuentra en memoria. El SO debe traer el bloque que contiene la referencia desde memoria secundaria a memoria principal. Después de que se solicita la operación de E/S para traer el bloque a memoria, el proceso que causó el fallo pasa a estado bloqueado, el SO realiza un cambio de contexto y pone a ejecutar otro proceso. Cuando el bloque solicitado sea accesible, ya en memoria principal, el proceso pasa a preparado.
	2.	**Traps(Trampas):** Asociadas a una condición de error o excepción  irreversible generada dentro del proceso que se está ejecutando. 

		Si es así el proceso en ejecución pasa a estado inalizado y se hace un cambio de contexto. 

## 1. IMPLEMENTACIÓN DE LAS ABSTRACCIONES DE PROCESO E HILO.
-----------
### BLOQUE DE CONTROL DE PROCESO
En el caso de Linux, se denomina **Descriptor de Proceso**, y viene dado por la estructura ``task_struct``.

![](apuntes2DGIIM/Process+Control+Block-+task_struct.jpg)



Es una estructura de datos que: 
 + Presenta subestructuras.
 
 + Define el estado del proceso.
 
 + Define los recursos que está usando.
 
		tty_struct -> Indica los dispositivos de entrada/salida asociados al proceso.
		
		files_struct -> Indica si se están usando archivos. Por defecto, siempre se utiliza esta estructura pues todo proceso 				tiene al menos tres archivo abiertos: Entrada estándar, Salida estándar, Salida de Error.
		
		mm_struct -> Indica el espacio de direcciones del proceso, el inicio y el final de la pila, la dirección de las 				librerías, etc... pero nunca el contenido.
		
		
		fs_struct -> Indica la dirección del directorio de trabajo actual, para que el SO busque a partir de ahí. 
		
		signal_struct -> Sistema de notificación de eventos. 
		
		thread_ info -> Información de bajo nivel. Se verá a continuación más detalladamente.
		
 

 Las subestructuras ``mm_struct`` , ``files_struct``, ``tty_struct``, y ``signal_struct`` se desgajan de la estructura principal por dos motivos:
 1. No se asignan cuando no es necesario (Por ejemplo un demonio no tiene asignada una terminal.)
 2. Permiten ser compartiadas cuando creamos hilos de un proceso. Por ejemplo, dos hilos son hermanos si comparten ``mm_struct`` (mismo espacio de usuarios).
 
 ###### ESTRUCTURA thread_info
 Contiene información de bajo nivel sobre el proceso/hilo y permite acceder a la pila kernel.
 Cuando un proceso se ejecuta en modo usuario o modo supervisor, cada proceso tiene dos pilas (pila usuario y pila kernel). En el caso de linux, esta pila se almacena en ``thread_info``. De esta forma evitamos que el usuario acceda a los datos de la pila generados por el kernel.
 Una de las funciones es saber en qué CPU se está ejecutando el proceso.
 
 Campos relevantes:
 
 + Indicadores ``TIF_SIGPENDING`` y ``TIF_NEED_RESCHED``-
 + ``CPU``: número CPU en la que se ejecuta.
 + ``preempt_count``.
 
 Macros:
 
 + ``current_thread_info``: dirección de thread_info del proceso actual.
 + ``current``: dirección del descriptor del proceso actual.
