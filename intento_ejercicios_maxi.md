## Ejercicios tema 2:

1. **Si invocamos la función  `clone() `con los argumentos que se muestran a continuación, indicar que tipo de hebra/proceso estamos creando. Justificarlo mostrando como quedarían los descriptores de las tareas.**
 
`clone(funcion,...,CLONE_VM|CLONE_FILES|CLONE_FS|CLONE_TRHEAD, ...)`

Estamos creando un nuevo proceso que compatirá los mismos ficheros abiertos, la información del sistema de archivos, el espacio de direcciones y el GID y TIP con el proceso padre (que es el desde donde se ha invocado a clone() ).

Supongo que justificarlo es hacer el dibujito. *Hacer dibujo del task_struct del padre y del hijo donde el mm_struct, el files_struct, el fs_struct se compartan, es decir señalen a la misma cajita. El GID y el TIP deben tener el mismo valor numérico.
El tty siempre se comparte.

2. **La llamada al sistema `clone()` en Linux se utiliza tanto para crear procesos como hilos (hebras). Indicar cómo la utilizaríamos, y dibujar los descriptores de procesos, cuando:
a)Un proceso crea otro proceso independiente.
b)Un proceso crea un hilo de la misma tarea.**

a) clone(SIGCHLD,0)
b) clone(funcion,...,CLONE_VM|CLONE_FILES|CLONE_FS|CLONE_THREAD|CLONE_SIGHAND,...)

3.  **El   algoritmo   de   planificación  CFS de   Linux   reparte imparcialmente la CPU entre los procesos de la clase correspondiente. Justificar como es posible que un proceso que realiza muchas entradas-salidas (por tanto, tiene ráfagas de CPU muy cortas) obtenga un trato imparcial respecto de un proceso acotado por computo (pocas entradas-salidas   y   que   consume   las   ráfagas   asignadas   completamente).     Para   el razonamiento, podemos suponer que solo hay un proceso de cada tipo y que la prioridad base de ambos es 120.**

El algoritmo de planificación de CFS siempre elige como el siguiente a ejecutar al proceso que tenga un vruntime más pequeño. El vruntime es el tiempo real de ejecucción normzalizado por el número de procesos en ejecución. Este tiempo es gestionado por el update_curr(). Como el proceso de de muchas E/S tiene ráfadas de CPU muy cortas, también lo será su vruntime, por lo tanto siempre que este listo se escogerá para ser ejecutado frente al proceso acotado por cómputo que tendrá un vruntime mayor, por tanto se equilibran los tiempos virtuales de ejecución.

4. **En   un   sistema   Linux   pueden   existir   procesos   que   pertenecen   a   diferentes   clases   de planificación. Indicar:
a) ¿Qué clases son estas y que algoritmo implementan?
b) Si en un momento dado, hay en el sistema al menos un proceso que pertenece a cada una de las clases, indicar en que orden se planificarán.
c) Si en la clase de tiempo compartido tenemos tres procesos: P1 y P2 con prioridad 120 y P3 con prioridad 110 ¿qué porcentaje de CPU la asignará en planificador a cada uno sabiendo que la prioridad 120 tiene un peso de 1024 y a la prioridad 110 le corresponde un peso de 110?**

a) * SCHED_FIFO: de tiempo real. Implementa el algoritmo FIFO (first in first out), es decir elige como proceso a ejecutar el que ms tiempo lleve esperando y lo ejecuta hasta el final.
* SCHED_RR: Ide tiempo real. Implementa el algoritmo round robin, en que se van ejecutando durante un periodo de tiempo (quantum), y cuando termina ese tiempo se pasa el proceso actual a listo y se ejecuta otro durante el mismo tiemppo (independientementede haber terminado el proceso o no)
* SCHED_NORMAL: Es de tiempo compartido y son manejados por CFS (completely fair scheduler), este algoritmo siempre elige al proceso que tendra un vruntime más pequeño. El vruntime es el tiempo real de ejecución de un proceso normalizado por el número de procesos en ejecución. En esta clase encontramos los SCHED_BATCH (procesos batch acotados por cómputo) y los SCHED_IDLE(procesos con poca importancia).

b) Primero se ejecutan los de tiempo real, (dentro de los de tiempo real se planifica primero el que tenga más prioridad o lleve más tiempo esperando), luego si no quedan de tiempo real, irían los de tiempo compartido, ejecutándose primero los SCHED_BATCH que los SCHED_IDLE

c) P1: 1024 / (1024 + 1024 + 110 ) 
 P2: 1024 / (1024 + 1024 + 110 ) 
 P3: 110 / (1024 + 1024 + 110 )

5. **Entre los diferentes pasos que sigue el planificador de Linux, función schedule(), está el de seleccionar la siguiente tarea a ejecutar (función pick_next_task() ). A la vista de lo estudiado, esbozar los pasos que debe seguir esta función para seleccionar el proceso teniendo en cuenta que existen tres clases de planificación (SCHED_FIFO, SCHED_RR y SCHED_NORMAL) y de las características/propiedades de los procesos dentro de cada clase**

Primero vemos los de tiempo real, y se ejecuta el que está antes en la cola (mayor prioridad o dentro de la misma prioridad el que lleve más tiempo), si es FIFO se ejecutará hasta el final o hasta que se bloquee mientras que si es RR seguirá el algoritmo RR. Una vez acabados los procesos de tiempo real se ejecutan los de la clase SCHED_NORMAL, según el CFS (primero los de vruntime menor), que dejan a la cola a los de la subclase SCHED_IDLE.

6. **¿Qué son los Gobernadores (Governors) en un sistema Linux? Indicar cuales son los dos gobernadores de usuario y su función.**
Los gobernadores permiten gestionar la potencia en base a unos criterios. Es decir, según el gobernador que estemos utilizando, la gestión se hará de una determinada manera Es Los dos gobernadores usuario son cpuspeed, que aumenta la velocidad de la CPU es decir, aumenta el rendimiento y el powersaved que ahorra energía a cambio de una disminución del rendimiento.

7. **El planificador a medio plazo de Linux implementa una política apropiativa denominada “apropiatividad mediante puntos de apropiación”. 
a) ¿En qué consiste y/o cómo se implementa? 
b) ¿Qué ventajas e inconvenientes presenta respecto a una política totalmente apropiativa**

a) Son puntos del flujo de ejecución kernel donde es posible apropiar el proceso sin incurrir a una condición de carrera. Para implementarlo, el RSI activa el bit de TIF_NEED_RESCHED y en la parte del código que es seguro para ello se escribe en un if(TID_NEED_RESCHED) schedule(), se esta forma solo se planifica si estamos en un momento adecuado.

b) La ventaja principal es que no se presentan problemas de condiciones de carrera.
Inconveniente: La fase de conflicto es mayor, por tanto la latencia de despacho es mayor. Esto hace que empeore la responsividad.


8. **En el algoritmo de planificación de la clase CFS siempre se elige como siguiente proceso para ejecución al proceso cuyo 
vruntime  es menor. Indicar que representa este parámetro y como se calcula.**
Es el tiempo real de ejecución real normalizado por el número de procesos en ejecución.
Se calcula con la siguiente formula: 
`vruntime = tiempo_ejecución_actual * 1024 / (peso rq) = (PesoNice0 / (peso cola)) * tiempo_real `

9. **Justificar por qué el kernel de Linux no es apto para la ejecución de aplicaciones de tiempo real duras, es decir que tienen plazos estrictos de ejecución. Observación, recordad la definición de latencia de apropiación y la implementación que hace de esta el kernel.**
 Debido a que el kernel de linux utiliza puntos de apropiación, la fase de conflicto del tiempo de latencia es mayor ya que deben esperar a que llegue a un punto de apropiación para apropiarse del proceso. Esto hace que no podamos garantizar que se vayan a cumplir todos los deadlineas lo que puede llevar a graves consecuencias.

10. **¿Qué mecanismos utiliza el kernel de Linux para gestionar el consumo de energía de los procesadores?**
Gobernadores

11. **Sobre los grupos de control en Linux:
a) ¿Qué utilidades tiene esta construcción?
b) ¿Cómo dan soporte a la virtualización?
c) Indicar al menos tres subsistemas de grupos de control.**

a) Asignarle a un grupo de procesos un porcentaje de CPU o de memoria.

b) Limitan los recursos de la maquina virtual.

c) blkio, CPU, memory, Freezer, cpuacct
