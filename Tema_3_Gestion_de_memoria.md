# TEMA 3: GESTIÓN DE MEMORIA
## REPASO
### Elementos hardware
Los computadores mantienen una jerarquía de memoria
para mejorar el acceso de la CPU a memoria.

**▷** **MEMORIAS CACHÉ:**
memorias intermedias que mantienen instrucciones/datos
previamente accedidos y que son más rápidas que la RAM.

**▷** Para cachés, definimos:

 · **Acierto de cache:** la instrucción/dato buscado está en ella. 
 
 · **Fallo de caché:** la instrucción/dato no esta en la misma.
 
 
 
 Las cachés “funcionan” porque explotan las localidad de las
referencias del código, datos, y pila de los programas.

**▷TIPOS DE LOCALIDAD:**

 · **Espacial:** si un item es referenciado, las direcciones
próximas a él tienden también a ser referenciadas.

 · **Temporal:** si un item referenciado, tiende de nuevo a ser
referenciado en breve.

**▷TIEMPO DE ACCESO EFECTIVO(TAE)**: Coste de acceder a memoria.
**TAE = p·ta + (1-p)·tf**   donde: **p**=probabilidad de acierto; **ta**=tiempo de acceso si
hay acierto;  **1-p**= probabilidad de fallo, y **tf**=tiempo de
acceso si hay fallo.


**▷ESPACIOS LÓGICOS Y FÍSICOS**

· La necesidad de poder reubicar un programa en
memoria hace necesario separar el espacio de
direcciones generadas por el compilador, espacio
lógico o virtual, del espacio físico en el que se
carga, el espacio de direcciones físicas (en RAM).

· **Dirección lógica** - la generada por la CPU;
también conocida como virtual.

· **Dirección física** - dirección que se pasa al
controlador de memoria.

**▷ TRADUCCIÓN DE DIRECCIONES**

La separación de espacios me obliga a realizar una
traducción de direcciones:

FALTA IMAGEN

**+ Unidad de Gestión de Memoria MMU (Memory Management Unit):**
 Dispositivo hardware que se encarga de: 
 1. Traducir direcciones virtuales en direcciones físicas.
 2. Implementar la protección.
 
La forma de la MMU dependerá del esquema de gestión
de memoria implementado en hardware. En el esquema
más simple, contendrá un registro de reubicación que
almacena el valor a sumar a cada dirección generada por
el proceso de usuario al mismo tiempo que es enviado a
memoria.

       - **Funcionamiento: **
           Cuando el procesador genera una dirección virtual, (el compilador es el que genera el espacio
           del proceso, por lo tanto, es el que asigna direcciones). Lo que lee la CPU del código binario son
           direcciones de memoria virtuales y es la MMU la que traduce esas direcciones virtuales en físicas y si no
           puede acceder a ella, genera una excepción.
           
       - Fragmentación:  fracción de memoria que no es asignable debido al propio mecanismo de gestión de memoria.

          Los sistemas de gestión de memoria han evolucionado con el objetivo principal de reducir la fragmentación de memoria.
          Al desacoplar los espacios lógicos de los físicos, podemos hacer que el espacio de direcciones de un proceso no sea continuo,
          podemos trocearlo, reduciendo así la demanda de memoria contigua.
          Los SOs actuales suelen utilizar paginación como esquema básico de gestión de memoria, si bien, dependiendo del procesador,             deben también utilizar segmentación. Por ejemplo, los procesadores Intel implementan segmentación como esquema básico de                 gestión de memoria (protección:modos de funcionamiento del procesador) y opcionalmente se puede activar o no la paginación.
   
