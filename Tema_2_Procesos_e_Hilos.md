
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

![](apuntes2DGIIM/imagenes/Process+Control+Block-+task_struct.jpg)



Es una estructura de datos que: 
 + Presenta subestructuras.
 
 + Define el estado del proceso.
 
 + Define los recursos que está usando.
 
	tty_struct -> Indica los dispositivos de entrada/salida asociados al proceso.
		
	files_struct -> Indica si se están usando archivos. Por defecto, siempre se utiliza esta estructura pues todo proceso tiene al menos tres archivo abiertos: Entrada estándar, Salida estándar, Salida de Error.
		
	mm_struct -> Indica el espacio de direcciones del proceso, el inicio y el final de la pila, la dirección de las librerías, etc... pero nunca el contenido.
		
	fs_struct -> Indica la dirección del directorio de trabajo actual, para que el SO busque a partir de ahí. 
		
	signal_struct -> Sistema de notificación de eventos. 
		
	thread_ info -> Información de bajo nivel. Se verá a continuación más detalladamente.
		
 

 Las subestructuras ``mm_struct`` , ``files_struct``, ``tty_struct``, y ``signal_struct`` se desgajan de la estructura principal por dos motivos:
 1. No se asignan cuando no es necesario (Por ejemplo un demonio no tiene asignada una terminal.)
 2. Permiten ser compartiadas cuando creamos hilos de un proceso. Por ejemplo, dos hilos son hermanos si comparten ``mm_struct`` (mismo espacio de usuarios).
 
 ### ESTRUCTURA thread_info
 Contiene información de bajo nivel sobre el proceso/hilo y permite acceder a la pila kernel.
 Cuando un proceso se ejecuta en modo usuario o modo supervisor, cada proceso tiene dos pilas (pila usuario y pila kernel). En el caso de linux, esta pila se almacena en ``thread_info``. De esta forma evitamos que el usuario acceda a los datos de la pila generados por el kernel.
 Una de las funciones es saber en qué CPU se está ejecutando el proceso.
 
 Campos relevantes:
 
 + Indicadores ``TIF_SIGPENDING`` y ``TIF_NEED_RESCHED``-
 + ``CPU``: número CPU en la que se ejecuta.
 + ``preempt_count``: Cuenta las veces que se le quita el control al proceso.
 
 Macros:
 
 + ``current_thread_info``: dirección de thread_info del proceso actual.
 + ``current``: dirección del descriptor del proceso actual.
 

## ESTADO DE LOS PROCESOS

El campo ``state`` del Descriptor de proceso  almacena el estado de un proceso en Linux. un proceso puede encontrarse en los siguientes estados:
+ ``TASK_RUNNING``: el proceso es ejecutable o está en ejecución.
+ ``TASK_INTERRUPTIBLE``: el proceso está bloqueado (dormido) de forma que puede ser interrumpido por una señal. Ejemplo: Espera de entrada de teclado.
+ ``TASK_UNINTERRUPTIBLE``: proceso bloqueado no despertable por otra señal. Ejemplo: Espera de lectura de disco que no se produce.
+ ``TASK_TRACED``: proceso que está siendo "traceado" por otro. Ejemplo: Cuando un proceso está siendo depurado, y su ejecución se para en un punto de ruptura.
+ ``TASK_STOPPED``: la ejecución del procesose ha detenido por alguna de las señales de control de trabajo.

El campo ``exit_state`` almacena los estados de los procesos que han finalizado:
+ ``EXIT_DEAD``: va a ser eliminado, su padre ha invocado wait().
+ ``EXIT_ZOMBIE``: el padre aún no ha realizado wait().

<p align="center"> 
<img src="imagenes/Captura estado procesos.png">
</p>

Todos los procesos menos uno, han sido creados a partir de otro. Al arrancar la máquina una de las labores es crear el primer proceso a partir del cual se crearán el resto.

Para ejecutar un proceso necesito dos llamadas -> fork+exec.

NOTA: Cuando uso exec lo que hago es: Código, Datos, Pila, SO
Cuando uso `fork` copio el stack de arriba de manera idéntica y tengo 2 procesos ejecutando igual. Cuando el segundo hace un `exec()`  el ejecutable lo que hace es destruir la memoria
nueva, se va al ejecutable y este le dice cuáles son sus características y el SO lo que hace es construir un nuevo espacio a partir del anterior (no el primero).


### Transiciones entre estados
+ ``clone()``: llamada al sistema para crear un proceso/hilo.
+ ``exit()``: llamada para finalizar un proceso.
+ ``sleep()``: bloquea/duerme a un proceso en espera de un determinado evento.
+ ``wakeup()``: desbloquea/despierta a un proceso cuando se ha producido el evento por el que espera.
+ ``schedule()``: planificador – decide que proceso tiene el control de la CPU


### Colas de estado
Existe una lista de procesos doblemente enlazada con todos los procesos del sistema y a la cabeza está el swapper (PID=0, ``task_struct_init_task``).

Los estados ``TASK_STOPPED``, ``EXIT_ZOMBIE``, ó ``EXIT_DEAD`` no están agrupados en colas.

Los procesos ``TASK_RUNNING`` están en diferentes colas de procesos ejecutables: una cola por procesador.


### Jerarquías de procesos
Para gestionar procesos de forma conjunta, todos los procesos forman parte de una jerarquía, con el proceso: systemd / init (PID=1) a la cabeza.
Todo proceso tiene exactamente un padre.
Procesos hermanos (``siblings``) = procesos con el mismo padre.
La relación entre procesos se almacena en el PCB:
+ ``parent``: puntero al padre
+ ``children``: lista de hijos.
…



## PLANIFICADOR CFS

### Pesos de Carga:

La importancia de un proceso viene dada por su prioridad, o por su peso de carga (load weight). Cuanto mayor sea el peso de carga de un proceso mayor será su prioridad (menor será la prioridad asignada con la orden `nice`). 
Cada nivel de prioridad con la orden nice equivale a un 10% de apropiación de la CPU, luego al disminuir un nivel obtienes 10% más de CPU, y viceversa.
El kernel convierte prioridades a peso de carga con la función `prio_to_weight()`. Los cálculos se realizan con la función `set_load_weight()` . 

El % de CPU que obtiene un proceso se calcula:  
`% de CPU del proceso i = Peso del proceso i  / \[\sum_{j=1}^nPeso del proceso j\]`

### Ejemplo de uso:

Dados dos procesos, P1, P2, ambos con la misma prioridad (`nice` = 0), y por tanto, el mismo peso, 1024, obtenemos:

```
% de CPU de P1 : 1024 / 1024 + 1024 = 50% CPU
% de CPU de P2 : 1024 / 1024 + 1024 = 50% CPU
```

Si P2 decrementa su prioridad con nice = 1, subiendo así de nivel, decrementa su uso de CPU un 10%: 

`Peso actual de P2 : 1024 / 1.25 = 1024 * 0.8 = 819.2 ⋍ 820` 

El 0.8 se obtiene: 

`% Peso a quitar = % a restar * nº procesos existentes = 10% * 2 = 20%`

Luego me quedo con el 80% del peso inicial del proceso.
Así, el % de CPU asignado a cada uno es: 

```
% de CPU de P1 : 1024 / 1024 + 820 ⋍ 55.53% CPU
% de CPU de P2 : 820 / 1024 + 820 ⋍ 44.47% CPU
```

Como podemos ver, ahora el proceso P2 tiene un 10% menos de CPU del que disponía antes (el 10% de 50 es 5, luego 50-5=45%)

### Clase CFS:

CFS (*Completely Fair Scheduler*) intenta modelar un procesador multitarea perfecto. Este algoritmo tiene como objetivo el maximizar el uso de la CPU pero permitiendo el uso interactivo de la máquina, es decir, tratará de que en ningún momento un usuario vea una bajada de rendimiento. 

El CFS no asigna rodajas de tiempo, sino que asigna una proporción del procesador dependiente de la carga del sistema.
El tiempo del que dispone un proceso para usar la CPU es:  
`
Tiempo de CPU del proceso i = [ Peso del proceso i  / \[\sum_{j=1}^nPeso del proceso j\] ] * P  
`  
Siendo P la latencia de planificación, que es el tiempo mínimo que se le va a asignar a un proceso, si el nº de procesos es mayor a `nr_latency`, en otro caso, P es `min_granularidad*n`, siendo n el nº de procesos y `min_granularidad` el mínimo tiempo asignado a cada proceso, ya que si n tiende a infinito, el tiempo asignado a cada proceso tiende a cero, por lo que es necesario definir dicha variable.

En la implementación actual, `sched_latency`=8 (latencia de planificación), `nr_latency`=8, y `min_granularity`= 1 us.

La clase CFS está definida en `kernell/sched_fair.c`.

### Tiempo asignado a un proceso:

Como la carga es dinámica, para el cálculo del tiempo asignado se usa un periodo:  
5 o menos procesos: 20 ms  
Sistema cargado: 5 ms más por proceso  
`Tiempo asignado = ( Longitud del periodo * peso ) / peso rq `

### Ejemplo:

Dados tres procesos, P1, P2, P3, tal que:  
P1: Nice = 5, Peso = 335  
P2: Nice = 0, Peso = 1024  
P3: Nice = -5, Peso = 3121  
```
Tiempo asignado a P1 = ( 20 * 335 ) / 4550 ⋍ 1.47 
Tiempo asignado a P2 = ( 20 * 1024 ) / 4550 ⋍ 4.5 
Tiempo asignado a P3 = ( 20 * 3121 ) / 4550 ⋍ 13.72
```
### Selección de proceso:

El tiempo virtual de ejecución (`vruntime`) es el tiempo de ejecución real normalizado por el número de procesos en ejecución. Este tiempo es gestionado por `update_curr()` definida en `kernel/sched_fair.c`. 
CFS intenta equilibrar los tiempos virtuales de ejecución de los procesos de tal manera que se elige siempre el proceso con `vruntime` más pequeño.

La cola de ejecución de CFS es un árbol rojo-negro, un árbol de búsqueda binaria autoequilibrado donde la clave de búsqueda es `vruntime`. En el árbol rojo-negro, el proceso con el menor vruntime es el que está más a la izquierda.

### Parámetros de planificación:

Lista de las variables relacionadas con planificación:   
	`% sysctl -A|grep "sched"|grep -v "domain"`  
Valor actual de las variables ajustables:  
	`/proc/sched_debug`  
Estadísticas cola actual:  
	`/proc/schedstat`  
Información planificación proceso PID:  
	`/proc/<PID>/sched`

