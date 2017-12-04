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
