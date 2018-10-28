# Sesión 4. Comunicación entre procesos utilizando cauces  

## Palabras clave  

Instrucción | Breve descripción | Sintaxis  
--- | --- | ---   
mknod | crea cauce de distintos tipo |  `int mknod (const char *FILENAME, mode_t MODE, dev_t DEV)`   
mkfifo | crea cauce del tipo fifo :| `int mkfifo (const char *FILENAME, mode_t MODE)`  
pipe | crea cauce sin nombre | `pipe `  
unlink | eliminir un archivo de la pila de cauces abiertos | `unlink NOMBRE`  
close | cerrar cauce lectura proceso anónimo | `close fd[0]` (lectura en este caso)  

## Concepto y tipos de cauce  

Los cauces son un mecanismo que permite la lectura y escritura de datos entre procesos. Además consiguen de manera la sincronización de estos.  
- Sigue un paradigma productor/consumidor (productor: escribe, conumidor lee).
- Tratamiento FIFO.
- Permite la sincronización, un proceso se bloqueará mientras intenta leer de un buffer hasta que lo consiga.  

Ejemplo de cauce anónimo: Órdenes en terminal `ls | wc -w`

### Cauces con nombre  

- Para usarse debe estar abierto en modo escritura y lectura, de otra manera se producirá un bloqueo. 
- Para crear usar `mknod`: `int mknod (const char *FILENAME, mode_t MODE, dev_t DEV)` .  
* FILENAME se corresponde con el nombre del cauce.  
* mode_t MODE: Valores a almacenar , su sintaxis es S_NOMBRE|código_numérico  

mode_Mode | descripción 
--- | --- 
S_IFCHR | tipo de archivo caracteres  
S_IFBLF | tipo archivo de bloques  
S_IFIFO | tipo de archivo FIFO  

* DEV, dispositovo al que se refiere, utilizaremos 0. 


- En el caso concreto de querer crear un archivo FIFO, podemos  invocar a al llamada `mkfifo` cuya sintaxis es `int mkfifo (const char *FILENAME, mode_t MODE)`  
- En el caso de querer eliminar un archivo FIFO utilizaremos `unlink`, su sintáxis es `unlink( NAME )`  
- Para escribir y leer de un archivo FIFO se hará como si se tratara de un fichero, con las órdenes `read` y `write`. 



#### Ejemplo productor  
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#define ARCHIVO_FIFO "ComunicacionFIFO"

int main(void)
{
int fd;
char buffer[80];// Almacenamiento del mensaje del cliente.
int leidos;

//Crear el cauce con nombre (FIFO) si no existe
umask(0);
mknod(ARCHIVO_FIFO,S_IFIFO|0666,0);
//también vale: mkfifo(ARCHIVO_FIFO,0666);

//Abrir el cauce para lectura-escritura
if ( (fd=open(ARCHIVO_FIFO,O_RDWR)) <0) {
perror("open");
exit(EXIT_FAILURE);
}

//Aceptar datos a consumir hasta que se envíe la cadena fin
while(1) {
leidos=read(fd,buffer,80);
if(strcmp(buffer,"fin")==0) {
close(fd);
return EXIT_SUCCESS;
}
printf("\nMensaje recibido: %s\n", buffer);
}

return EXIT_SUCCESS;
}
```

Este programilla acturará como consumidor, creará el archivo FIFO *ComunicacionFIFO* si no estaba creado y leerá los datos de tal buffer mostrando en pantalla *El mensaje recibido es: MENSAJE* y finalizará al escribir fin. 

Un método de comunicación con él,es mediante la orden **echo** redireccionando la salida hacia tal archivo, muestro ejemplo:

```c
$:~/SO/practicas/sesion_4$ echo eres mu pesao >> ComunicacionFIFO 

Mensaje recibido: eres mu pesao


$:~/SO/practicas/sesion_4$ echo fin >> ComunicacionFIFO 

Mensaje recibido: fin

```
Como podemos ver en al escribir fin el programa no acaba, y esto es que c por defecto añade a nuestra cadena un caracter salto de línea y por tanto no coincide con el fin indicado. 

Veamos ahora un programa productor: 


```c
//productorFIFO.c
//Productor que usa mecanismo de comunicacion FIFO

#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<errno.h>
#define ARCHIVO_FIFO "ComunicacionFIFO"

int main(int argc, char *argv[])
{
int fd;

//Comprobar el uso correcto del programa
if(argc != 2) {
printf("\nproductorFIFO: faltan argumentos (mensaje)");
printf("\nPruebe: productorFIFO <mensaje>, donde <mensaje> es una cadena de caracteres.\n");
exit(EXIT_FAILURE);
}

//Intentar abrir para escritura el cauce FIFO
if( (fd=open(ARCHIVO_FIFO,O_WRONLY)) <0) {
perror("\nError en open");
exit(EXIT_FAILURE);
}

//Escribir en el cauce FIFO el mensaje introducido como argumento
if( (write(fd,argv[1],strlen(argv[1])+1)) != strlen(argv[1])+1) {
perror("\nError al escribir en el FIFO");
exit(EXIT_FAILURE);
}

close(fd);
return EXIT_SUCCESS;
}

```

Este programa interactúa con el anterior:

```shell
$:~/SO/practicas/sesion_4$ ./productorFIFO hola

Mensaje recibido: hola
$:~/SO/practicas/sesion_4$ ./productorFIFO fin
[2]+  Hecho                   ./consumidorFIFO.out

```

Como vemos ahora escribiendo *fin* sí hemos conseguido acabar. 


### Cauces sin nombre  

- Para su creación se utiliza la instrución `pipe`, devuelve array de dos enteros, el primero de lectua y el segundo de escritura.
- Automáticamente crea dos descriptores (write y read), un proceso sólo podrá utilizar uno de ellos. 
- Se almacenan mediante i-nodos en la tabla de i-nodos.
- Cuando un proceso crea un cauce su hijo lo hereda, para su interacción es necesario cerrar el que no se vaya a utilizar. 
- La orden **dup** duplica un cauce (Ejemplo un poco más abajo) 
- **dup2** sobreescribe la salida, su sintáxis es `dup2( NUEVO, DESCRIPTOR_A_MACHACAR)`

```C
int fd[2];
pipe(fd);
close(fd[0]); //el proceso en el que se encuentre tal instrucción NO va a leer del buffer
write(fd[1],mensaje,strlen(mensaje)+1); // escritura  
read(fd[0],buffer,sizeof(buffer)); // lectura
```
Ejemplo tabla de i-nodos tras utiliar la orden pipe: 

Nombre | Identificador | salida o entrada  
--- | --- | ---  
STDIN\_FILENO | 0 | Entrada estándar, por defecto el teclado   
STDOUT\_FILEENO | 1 | Salida estándar, por defecto consola activa  
f[0] | 3 (El primer identificador libre) | Descriptor de lectura  
f[1] | 4 | Descriptor de escritura   

Si ahora se hiciera la orden **dup** con el descriptor de lectura `dup(f[0]); ` la tabla añadiría una nueva entrada: 

Nombre | Identificador | salida o entrada  
--- | --- | ---  
STDIN\_FILENO | 0 | Entrada estándar, por defecto el teclado   
STDOUT\_FILEENO | 1 | Salida estándar, por defecto consola activa  
f[0] | 3 (El primer identificador libre) | Descriptor de lectura  
f[1] | 4 | Descriptor de escritura   
f[0] | 5  | Descriptor de lectura duplicado   

La orden **close** pone a NULL el puntero y **dup2** machaca el contenido: 

- ¿Si se hace un `close(fd[0])` se machacarían los dos?
- ¿Si se hiciera un `dup2(STDIN\_FILENO , f[0])` se sobreescribirái en los dos?

```c
close(STDIN_FILENO);
```
Nombre | Identificador | salida o entrada  
--- | --- | ---  
STDIN\_FILENO | 0 | NULL (tras realizar el NULL)  
STDOUT\_FILEENO | 1 | Salida estándar, por defecto consola activa  

`dup2(fd[0],STDIN_FILENO)`

Nombre | Identificador | salida o entrada  
--- | --- | ---  
STDIN\_FILENO | 0 | Ahora tiene la entrada de f[0]  
STDOUT\_FILEENO | 1 | Salida estándar, por defecto consola activa  
f[0] | 3 (El primer identificador libre) | Descriptor de lectura  
f[1] | 4 | Descriptor de escritura   

## Paradigma maestro-esclavo  

Lo visto hasta ahora resulta realmente interesante para el desarrollo del esquema de trabajo maestro-esclavo, un proceso denominado maestro reparte y recibe el el trabajo de varios procesos, denominados ejemplos.

La resolución del ejercicio 5 que se propone y como ejemplo de tal paradigma: 

```c
/*
  @file Ejercicio5, maestro-esclavo 
  @brief Dado un rango de valores, se pide encontrar los primos dentro de éste utilizando el esquema de trabaja maestro-esclavo. 

Para ello el porceso padre recivirá como argumento los datos de entrada y creará dos procesos hijos, que calculará la mitad de los datos cada uno, devolviéndole el resultado al padre. 

Deberán ser mostardos en orden ascendente. 

@author Blanca Cano Camarero 
@date 27/10/2018  
 */


#include<sys/types.h>
#include<fcntl.h>
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<errno.h>
#include<math.h>
#include<string.h>

#define NUM_HIJOS  3

// Devuelve 0 si es primo, 1 en caso contrario
int esPrimo(int n)
{
  //int raiz = ceill( sqrt(n) ); //ERROR DE ENLAZADO EN CEILL Y SQRT
  for(int i=2; i<n; i++) //sustituir n por raiz
	{
	  if( n%i == 0) //si no es múltiplo
	    {
	      return (1); 
	    }
	}
  
  return(0);
}

int main(int argc, char *argv[]) 
{

  //VARIABLES 
  int fd[2];//int fd[NUM_HIJOS][2];
  pipe(fd); //LLAMADA AL SISTEMA PARA CREAR EL CAUCE
  pid_t PID = 1;

  int rango[2]; //rangos del intervalo en el que se trabajará
  int mi_rango[2];
  char buffer[4]; //almacena contenido de escritura o lectura del buffer
  //~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  
  // COMPROBACIONES

  //argumentso correctos intervalo min, max
  if (argc != 3) 
    {
      perror("Número de argumentos no valido, introduzca un intervalo, es decir dos elementos\n");
      exit(EXIT_FAILURE);
    
    }
  for(int i=0; i< 2  ; i++) //Escribo el intervalo
    {
      rango[i] = atoi(argv[i+1]);
      if(rango[i] <= 0) //Valor que devuelve atoi en el caso de que no se haya introducido un valor válido
	{
	  perror("Las cotas del intervalo deben  ser naturares mayores que 0\n");
	  exit(EXIT_FAILURE);
	}
    }

  //~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  //CREACIÓN DE LOS HIJOS Y REPARCITICÍN DEL TRABAJO
  
  int intervalo = (rango[1]-rango[0])/NUM_HIJOS; //en el caso de que no sea exacto el último hijo se llevará más trabajos hasta completar todos los intervalos
  
  mi_rango[0] = rango[0]-intervalo-1;
  mi_rango[1] = rango[0]-1;

  //creo hijo asignándole a cada uno su rango 
  for( int i=0; i<NUM_HIJOS-1; i++)
    {
      mi_rango[0]=mi_rango[1]+1;
      mi_rango[1]+=intervalo;
            
      if( (PID = fork())<0)
	{
	  perror("Error en el fork");
	  exit(EXIT_FAILURE);
	}
      
      if(!(PID !=0 )) //para que se salga con el hijo
	{
	  break;
	}
    }

  // RAPARTICIÓN DE TRABAJO
  if(PID != 0) //padre
    {
      //El último hijo tine un rango distinto, se asigna como caso especial aquí 
      if((PID = fork())<0)
	{
	  perror("Error en el fork");
	  exit(EXIT_FAILURE);
	}

      mi_rango[0]=mi_rango[1]+1;
      mi_rango[1]=rango[1];
      //BUCLE DE TRABAJO DEL PADRE 
      if(PID != 0)
	{
	  
	  close(fd[1]); //cerramos modo escritura
	  int cnt = 0;
	  while(cnt < NUM_HIJOS)
	    {
	      int leidos = read(fd[0], buffer, sizeof(buffer));
	      if( strcmp(buffer,"fin") == 0)
		{
		  cnt++;
		}
	      else
		{
		  printf("%s ", buffer);
		}
	    }
	  printf("\nYa se han mostrado todos los primos del rango [%d,%d]\n", rango[0] , rango[1]);
	  exit(0);
	}
    }

   close(fd[0]); //cerramos modo lectura, EL PROCESO HIJO ESCRIBE

  //ESCRIBIR CÓDIGO PARA BUSCAR PRIMOS e ir inviando
  for( int i= mi_rango[0]; i<= mi_rango[1]; i++)
    {
      if( esPrimo(i) == 0) //devuelve 0 si es primo
	{
	  sprintf(buffer,"%d " , i);
	  write( fd[1], buffer,4);// strlen(buffer)+1 );
	}
    }
  sprintf(buffer,"fin");
  write( fd[1], buffer, strlen(buffer)+1 );
  
  exit(0);
}

```



