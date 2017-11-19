##Ejercicios tema 2:

1. **Si invocamos la función  `clone() `con los argumentos que se muestran a continuación, 
indicar que tipo de hebra/proceso estamos creando. Justificarlo mostrando como quedarían
los descriptores de las tareas.
 
`clone(funcion,...,CLONE_VM|CLONE_FILES|CLONE_FS|CLONE_TRHEAD, ...)**

Estamos creando un nuevo proceso que compatirá los mismos ficheros abiertos,
la información del sistema de archivos, el espacio de direcciones 
