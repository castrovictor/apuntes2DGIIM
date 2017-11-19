## Ejercicios tema 2:

1. **Si invocamos la función  `clone() `con los argumentos que se muestran a continuación, indicar que tipo de hebra/proceso estamos creando. Justificarlo mostrando como quedarían los descriptores de las tareas.**
 
`clone(funcion,...,CLONE_VM|CLONE_FILES|CLONE_FS|CLONE_TRHEAD, ...)`

Estamos creando un nuevo proceso que compatirá los mismos ficheros abiertos, la información del sistema de archivos, el espacio de direcciones y el GID y TIP con el proceso padre (que es el desde donde se ha invocado a clone() ).

Supongo que justificarlo es hacer el dibujito. *Hacer dibujo del task_struct del padre y del hijo donde el mm_struct, el files_struct, el fs_struct se compartan, es decir señalen a la misma cajita. El GID y el TIP deben tener el mismo valor numérico.
¿También comparten el tty_struct? IDK*

2. **La llamada al sistema `clone()` en Linux se utiliza tanto para crear procesos como hilos (hebras). Indicar cómo la utilizaríamos, y dibujar los descriptores de procesos, cuando:
a)Un proceso crea otro proceso independiente.
b)Un proceso crea un hilo de la misma tarea.**

a) clone(SIGCHLD,0) o clone sin ningun flag
b) clone(funcion,...,CLONE_VM|CLONE_FILES|CLONE_FS|CLONE_THREAD|CLONE_SIGHAND,...)

3.  **El   algoritmo   de   planificación  CFS de   Linux   reparte imparcialmente la CPU entre los procesos de la clase correspondiente. Justificar como es posible que un proceso que realiza muchas entradas-salidas (por tanto, tiene ráfagas de CPU muy cortas) obtenga un trato imparcial respecto de un proceso acotado por computo (pocas entradas-salidas   y   que   consume   las   ráfagas   asignadas   completamente).     Para   el razonamiento, podemos suponer que solo hay un proceso de cada tipo y que la prioridad base de ambos es 120.**

Se me ocurre asignarle mayor prioridad, por lo tanto mayor peso, al proceso que realiza muchas entradas-salidas y darle menos peso al que está acotado por computo, así la CPU asigna más tiempo al de entrada-salida, pero nunca lo usa todo ya que necesita de la E/S mientras que el otro si que usa todo su tiempo pero es menor, por lo que ajustando bien las prioridades podemos conseguir que al final vayan consumiendo los mismos tiempos.

4. **En   un   sistema   Linux   pueden   existir   procesos   que   pertenecen   a   diferentes   clases   de planificación. Indicar:
a) ¿Qué clases son estas y que algoritmo implementan?
b) Si en un momento dado, hay en el sistema al menos un proceso que pertenece a cada una de las clases, indicar en que orden se planificarán.
c) Si en la clase de tiempo compartido tenemos tres procesos: P1 y P2 con prioridad 120 y P3 con prioridad 110 ¿qué porcentaje de CPU la asignará en planificador a cada uno sabiendo que la prioridad 120 tiene un peso de 1024 y a la prioridad 110 le corresponde un peso de 110?**

a) (creo que pide esto xd)
* SCHED_FIFO: de tiempo real. Implementa el algoritmo FIFO (first in first out), es decir elige como proceso a ejecutar el que ms tiempo lleve esperando y lo ejecuta hasta el final.
* SCHED_RR: Ide tiempo real. mplementa el algoritmo round robin, en que se van ejecutando durante un periodo de tiempo (quantum), y cuando termina ese tiempo se pasa el proceso actual a listo y se ejecuta otro durante el mismo tiemppo (independientementede haber terminado el proceso o no)
* SCHED_NORMAL: Es de tiempo compartido y son manejados por CFS (completely fair scheduler). En esta clase encontramos los SCHED_BATCH y los SCHED_IDLE.

b) Primero se ejecutan los de tiempo real, luego los de CFS y por último los IDLE (que es un CFS tambien no??)
Por lo cual primero se ejecutaría el SCHED_FIFO, despues el SCHED_RR, despues el SCHED_BATCH y por ultimo el SCHED_IDLE
(no se si hay que hacer distinción entre los de tiempo real ni si el FIFO iría antes, tampoco se si debería meter lo del batch y el idle )

c) P1: 1024 / (1024 + 1024 + 110 ) 
 P2: 1024 / (1024 + 1024 + 110 ) 
 P3: 110 / (1024 + 1024 + 110 )

5. **Entre los diferentes pasos que sigue el planificador de Linux, función schedule(), está el de seleccionar la siguiente tarea a ejecutar (función pick_next_task() ). A la vista de lo estudiado, esbozar los pasos que debe seguir esta función para seleccionar el proceso teniendo en cuenta que existen tres clases de planificación (SCHED_FIFO, SCHED_RR y SCHED_NORMAL) y de las características/propiedades de los procesos dentro de cada clase**

Primero busca si hay procesos de SCHED_FIFO, si los hay, los ejecuta según su antigüedad, cuando ya no hay más de esta clase, mira si hay alguno de SCHED_RR, y los va ejecutando usando el algoritmo RR, a continuación, cuando no queden más FIFO ni RR (no han aparecido nuevos) ejecutara los de SCHED_NORMAL.

(supongo que es algo así??)

6. **¿Qué son los Gobernadores (Governors) en un sistema Linux? Indicar cuales son los dos gobernadores de usuario y su función.**
Son los que gestionan la potencia en base a unos criterios.


7. **El planificador a medio plazo de Linux implementa una política apropiativa denominada “apropiatividad mediante puntos de apropiación”. 
a) ¿En qué consiste y/o cómo se implementa? 
b) ¿Qué ventajas e inconvenientes presenta respecto a una política totalmente apropiativa**

a) Para que no ocurran condiciones de carrera, se crean puntos de apropiación, que son puntos donde es seguro hacer un cambio de contexto, por lo que podemos apropiar el proceso que se está ejecutando. Para implementarlo, el RSI activa el bit de TIF_NEED_RESCHED y en la parte del código que es seguro para ello se escribe en un if(TID_NEED_RESCHED) schedule(), se esta forma solo se planifica si estamos en un momento adecuado.

b) La ventaja principal es que no se presentan problemas de condiciones de carrera.
¿Inconveniente?

8. **En el algoritmo de planificación de la clase CFS siempre se elige como siguiente proceso para ejecución al proceso cuyo 
vruntime  es menor. Indicar que representa este parámetro y como se calcula.**
Es el tiempo real de ejecución real normalizado por el número de procesos en ejecución.
Se calcula con la siguiente formula: 
`vruntime = tiempo_ejecución_actual * 1024 / (peso rq) = (PesoNice0 / (peso cola)) * tiempo_real `

9. **Justificar por qué el kernel de Linux no es apto para la ejecución de aplicaciones de tiempo real duras, es decir que tienen plazos estrictos de ejecución. Observación, recordad la definición de latencia de apropiación y la implementación que hace de esta el kernel.**
 xd idk 


10. **¿Qué mecanismos utiliza el kernel de Linux para gestionar el consumo de energía de los procesadores?**
cpufreqtils, cpupower, powerTOP, TLP

11. **Sobre los grupos de control en Linux:
a) ¿Qué utilidades tiene esta construcción?
b) ¿Cómo dan soporte a la virtualización?
c) Indicar al menos tres subsistemas de grupos de control.**

a) 

b) 

c) blkio, CPU, memory, Freezer, cpuacct
