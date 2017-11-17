## Planificador de tiempo-real
Los procesos de tiempo-real ejecutables que existan en el sistema se ejecutarán antes que el resto, salvo que exista uno con mayor prioridad. Los algoritmos de planificación de tiempo real dependen de:
1. Cuándo el sistema realiza análisis de planificabilidad.
2. De hacerlo, si se realiza estática o dinámicamente
3. El resultado del análisis produce un plan de planificación acorde al cual se desarrollarán las tareas en tiempo de ejecución.

La cola de ejecución es simple, como se aprecia en la figura:
<p align="center"> 
<img src="imagenes/imgdiapo73.png">
</p>

El *mapa de bits* permite seleccionar la cola con procesos en 2 (64 bits) o 4 (32 bits) instrucciones en ensamblador.

## Planificado en sistemas multipocesadores
En los sistemas multipocesador más tradicionales, los procesos no se vinculan a los procesadores. Hay una única cola para todos los procesadores o, si se utiliza algún tipo de esquema basado en prioridades, hay múltiples colas basadas en prioridad, alimentando a un único colectivo de procesadores. Puede verse como una arquitectura de colas multiservidor. A la hora de planificar, hay que considerar algunos aspectos:
1. Compartición de la carga de CPU imparcial
2. Afinidad de una tarea por un procesador
3. Migración de tareas sin sobrecarga.

En resumen, se realiza igual que en monoprocesadores, pero se tiene en cuenta el número de CPUs, la asignación y la liberación de proceso y procesador.

## Gestión de la energía
En los diseños actuales es importante considerar la gestión de la potencia, con el objetivo de reducir los costes de consumo de energía y los costes de refrigeración. Esta gestión se realiza a dos niveles:
* Nivel de CPU: P-states, C-states y T-states.
* Nivel de SO: CPUfreq (paquetes cpufrequtils y cpupower) y planificación.

Los procesadores actuales pueden funcionar a diferentes voltajes. A mayor voltaje, más alta es la frecuencia de funcionamiento y por tanto, mayor rendimiento. Cuando hay carga máxima del sistema, el procesador aumenta el voltaje para incrementar la frecuencia y por el contrario, cuando está ocioso (poca carga de trabajo), la disminuye para ahorrar energía.

### Especificación ACPI
Todos los procesadores actuales cumplen la especificación ***Advanced Configuration and Power Interface***, que se define como la especificación abierta para la gestión de potencia y gestión térmica, controladas por el SO. Se definen cuatro estados Globales (*G-Estados*):
* G0: estado de funcionamiento (estados-C y estados-P)
* G1: estado dormido (S-estados)
* G2: estado apagado soft
* G3: estado apagado mecánico

### Estados de la CPU.
* S-estados: (dormidos en G1). Van de S1 a S5. Son estados en los que se van apagando diferentes componentes del ordenador, empezando por los que más consumen.
* C-estados: son los estados de potencia en G0.C0 (activo), C1(halt), C2(stop), C3(deep sleep)... 
* P-estados: el voltaje del procesador se modifica a escalones, no de forma continua. Por, ejemplo, pasar de 5V a 3V. Éstos dependen del procesador.
* T-estados: son los conocidos como *estados throttles*. Estań relacionados con la gestión técnica, como estabilizar la temperatura, pero no se dedican a la gestión de energía. Introducen ciclos ociosos.

### Estructura CPUfreq
* El **subsitema CPUfreq** es el responsable de ajustar explícitamente la frecuencia del procesador.
*  Es una estructura modularizada que separa políticas (gobernadores) de mecanismos (*drivers* específicos de CPUs).

![gobernadores](/imagenes/diapo80.png)

#### Gobernadores
Permiten gestionar la potenciar en base a unos criterios. Es decir, según el gobernador que estemos utilizando, la gestión se hará de una determinada manera. Son los siguientes:
* **Perfomance** - mantiene la CPU a la máxima frecuencia posible dentro de un rango especificado por el usuario
* **Powersave** - intentan ahorrar energía, manteniendo la CPU a la menor frecuencia posible dentro de un rango
* **Userspace** -exporta la información disponible de la frecuencia a nivel usuario (sysfs) permitiendo su control al mismo. Permite a cualquier espacio de programa de usuario ajustar la frecuencia del procesador. Es el que más libertad dar para ajustar la velocidad del procesador. Más información en https://www.ibm.com/support/knowledgecenter/en/linuxonibm/liaai.cpufreq/TheUserspaceGovernor.htm
* **On-demand** - ajusta la frecuencia dependiendo del uso actual de la CPU.
* **Conservative** - funciona al igual que el *on-demand*, pero los ajustes son menos agresivos.

Existen una serie de herramientas para la gestión de energía y potencia del sistema, como cpufreqtils, cpupower, powerTop y TLP.

### Planificación y energía. Algoritmos
* En CPM (procesador multinúcleo) con recursos compartidos entre núcleos de un paquete físico, el rendimiento máximo se obtiene cuando el planificador distribuye la carga equitativamente entre todos los paquetes.
* En CPM sin recursos compartidos entre núcleos de un mismo paquete físico, se ahorrará energía sin afectar al rendimiento si el planificador distribuye primero la carga entre núcleos de un paquete, antes de buscar paquetes vacíos.

El administrador puede elegir entre dos algoritmos de planificación, bien optar por un rendimiento óptimo del sistema (y por tanto, mayor consumo de energía) o bien priorizar el ahorro de energía sobre el rendimiento. Se realiza modifricando las entradas *sched_mc_power_saving* y *sched_smt_power_saving*.

* Si se opta por rendimiento óptimo, el gestor asigna cada hebra de proceso a un núcleo, teniendo así una sola hebra por núcleo. De esta forma no hay colisión entre hebras, aumentando la velocidad al no haber colisión entre cachés.
* En el modelo de ahorro de energía se colocan dos hebras en cada procesador, pudiendo así apagar alguno de los procesadores para ahorrar energía.

Por ejemplo, en un procesador quad-core (4 núcleos) ponemos cada hebra en un núcleo. Al estar todos los procesadores trabajando, aumenta el rendimiento, pero tmabién el consumo. Si optamos por ahorrar energía, el gestor coloca dos hebras en cada núcleo, quedando, por tanto, dos núcleos libres, que se apagan para ahorrar energía. Como desventaja, al tener dos núcleos con dos hebras cada uno, el rendimiento disminuye.

![planificacion energia](/imagenes/diapo84.png)





