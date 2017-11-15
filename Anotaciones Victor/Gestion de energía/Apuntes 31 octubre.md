## Gestión de energía

Todos los procesadores actuales pueden funcionar a diferenets voltajes. Cuando más alto, más alta es la frecuencia de funcionamiento y por lo tanto más rápido puede trabajar el procesador. Cuando tengo una crga máximo en el sistema, aumento el voltaje para aumentar la frecuencia y así aumentar la velocidad de trabajo del procesador. Cuando el procesador está ocioso, hago lo contrario.

Todos los procesadores actuales cumplen la especificación Advanced ***Configuration and Power Interface***. Se definen cuatro estados:
*añadir lo de la diapositiva*

### Estados de la CPU
Los s-estados, estados en los que voy apagando diferentes componentes del ordenador, apagando primero los que más consumen.
En los p-estados, el voltaje lo modifico a escalones, no de forma continua. Por ejemplo de 5v a 3v, estos saltos dependen del procesador.
Los t-estados(tortel) están relacionados con la gestión técnica, como estabilizar la temperatura..., pero no se dedican a la energía

## GObernadores
Con powerSave intento ahorrar energia, por tanto, reducir la frecuencia del procesador.
El on-demand hace ajustes más agresivos que el conservative.

## Algoritmos de planificación y energía
El usuario tiene dos opciones, decirle al sistem aque trabaje al máximo rendimiento o bien que trabaje ahorrando el máximo de energía posible.
El máximo rendimiento lo tengo cuando el gestor asigna una hebra a cada núcleo, y solo en una hebra del núcleo. PAra que no haya colisión de la hebra 1, aumentando así la velocidad al no haber colisiones en las cachés.
EN el otro modelo ahorro poniendo dos hebras en un procesador, así otro procesador queda libre, se apaga y no consume energía.



## Uso de los cgroups
Los grupos de control permiten 4 interfaces para interaccionar con él:
+ Seudo-sistema
+ Herramientas de libcgroup: crear grupo, clafisicarlo...
+ Una hebra kernel funcionando, lee un arhcivo de configuración donde establezco los límites de cada grupo y establece los grupos con la configuración correspondiente cada vez que se arranca el ordenador.
+ Las otras actuan directamente sobre los grupos de control sin darnos cuenta.

¿Cómo hacer la gestión vía seudo-sistema?
