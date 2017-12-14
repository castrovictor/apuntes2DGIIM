# Sesión 3: LLamadas al sistema para el control de procesos

## Creación de procesos
Una vez leída la introducción del guión de prácticas, veamos el primer ejemplo propuesto, que tiene el siguiente código:
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

void main() {
	pid_t id_proceso;
	pid_t id_padre;

	id_proceso = getpid();
	id_padre = getppid();

	printf("Identificador de proceso: %d\n", id_proceso);
	printf("Identificador del proceso padre: %d\n", id_padre);
	sleep(60);
}
~~~
En primer lugar, se declaran dos variables de tipo *pid_t* para almacenar dos PIDs de dos procesos. Los PIDs son valores enteros, pero en vez de declararlos como *int*, lo declaramos como *pid_t*, que es un *typedef*. Está definido así ya que algunos sistemas, los PIDs se almacenan com enteros, mientras que en otros se almacenan como *unsigned short*.

La orden *getpid()* devuelve el PID del proceso actual, lo almacenamos en *id_proceso*. 
La orden *getppid()*, devuelve el PID del proceso padre. ¿Quién es el padre de nuestro proceso? Pues justo el proceso que nos permite ejecutar programas, el proceso *bash*. Para comprobarlo, se ha añadido la función ``sleep(60)`` poder comprobar que el proceso padre es *bash*. En los 60 segundos que *duerme* el programa antes de finalizar, podemos ejecutar en una terminal la orden ``top`` para verificar que el PID del padre corresponde al proceso *bash*.


## Llamada al sistema fork. Procesos padre e hijo
La llamada al sistema ``fork()`` se utiliza para crear un proceso hijo. Cuando en un programa invocamos a ``fork()``, se crea un hijo del proceso, que contendrá el mismo código y contenido que el padre. La orden ``fork()`` devuelve un PID, que será 0 si el proceso corresponde al hijo y un entero positivo si corresponde al padre. La mejor forma de verlo es con un ejemplo. Vamos a ejecutar un programa que cree un proceso hijo, imprimiendo el prceso que es con su PID correspondiente.
~~~c
#include <stdio.h>
 
int main() {
	pid_t pid;
 
	printf("PADRE: Soy el proceso padre y mi pid es: %d\n", getpid());
 
	pid = fork();
 
	// En cuanto llamamos a fork se crea un nuevo proceso. En el proceso
	// padre 'pid' contendrá el pid del proceso hijo. En el proceso hijo
	// 'pid' valdrá 0. Eso es lo que usamos para distinguir si el código
	// que se está ejecutando pertenece al padre o al hijo.
 
	if (pid) // Este es el proceso padre
	{
		printf("PADRE: Soy el proceso padre y mi pid sigue siendo: %d\n", getpid());
		printf("PADRE: Mi hijo tiene el pid: %d\n", pid);
	}
	else // Proceso hijo
	{
		printf("HIJO: Soy el proceso hijo y mi pid es: %d\n", getpid());
		printf("HIJO: mi padre tiene el pid: %d\n", getppid());
	}
}
~~~

Este programa crea un proceso hijo. Cuando invocamos a *fork()*, creamos el proceso hijo, con una copia del código del actual.
En el proceso padre, fork() almacena en la variable *pid* el PID actual del proceso. En el proceso hijo, devuelve un 0. 
Gracias a esto, podemos diferenciar cuándo estamos en el proceso padre y cuando en el hijo.
En el proceso hijo tenemos el mismo código que en el padre, se ejecuta todo lo que hay a partir del *fork()* que ha creado al hijo, pudiendo usar variables que se hayan declarado antes, ya que se han copiado en el hijo.
Por tanto, si estamos en el proceso padre, PID no vale 0, por lo que entra en el primer *if* para imprimir que estamos en el proceso padre, informando de su PID.
Si estamos en el hijo, la variable *pid* vale 0, por lo que entra en la parte del *else*, para informar de que es el hijo, el *pid* que tiene asignado en el sistema el proceso, y el PID de su padre, obtenido con *getppid()*.

Habiendo entendido esto, podemos resolver el ejercicio 1 de la sesión. Se pide implementar un programa que dado un número como argumento, cree un proceso hijo. El hijo debe informar si es un número par o impar. El padre comprobará si el número es divisible por 4. 
El mecanismo es exactamente el mismo que en el ejemplo anterior. Basta con sustituir los mensajes que imprimos para informar, por las operaciones correspondientes que pide el ejercicio. El código resultante sería algo así:
~~~c
#include<sys/types.h>						
#include<unistd.h>		 
#include<stdio.h>
#include<errno.h>
#include <stdlib.h>
#include <stdbool.h>

int main(int argc, char *argv[]) {
  if(argc != 2) {
    printf("\nNúmero de argumentos inválido.");
    exit(-1);
  }
  int n = strtol(argv[1], NULL, 10);
  int pid;
  pid = fork();

  // En cuanto llamamos a fork se crea un nuevo proceso. En el proceso
	// padre 'pid' contendrá el pid del proceso hijo. En el proceso hijo
	// 'pid' valdrá 0. Eso es lo que usamos para distinguir si el código
	// que se está ejecutando pertenece al padre o al hijo.
  if(pid) { //Proceso padre
    bool es_divisible = (n%4 == 0);
    printf("PADRE: Soy el proceso padre y mi pid sigue siendo: %d\n", getpid());
		printf("PADRE: Mi hijo tiene el pid: %d\n", pid);

    if(es_divisible) {
        printf("\nEl número es divisible por 4.");
    } else {
      printf("\nEl número no es divisible por 4.");
    }
  } else { //Proceso hijo
    bool es_par = (n%2 == 0);
    printf("\nHIJO: Soy el proceso hijo y mi pid es: %d\n", getpid());
		printf("\nHIJO: mi padre tiene el pid: %d\n", getppid());

    if(es_par) {
        printf("\nEl número es par.\n");
    } else {
      printf("\nEl número no es par.\n");
    }
  }
  return 0;
}
~~~

Para el ejercicio 2, basta leer detenidamente el código y las notas proporcionadas. Veamos el ejercicio 3. La explicación viene implícita en el código.
~~~c
/*
Jerarquía de procesos tipo 1
*/
for(i = 1; i < nprocs; i++) {
	//Creamos un hijo y guardamos en childpid lo devuelvo por fork(). Si es -1 es que se ha producido un error
	if((childpid = fork() == -1) {
		fprintf(stderr, "Could not create child %d: %s\n", strerror(errno));
		exit(-1);
		}

	//Si estamos en el proceso hijo, childpid=0, por lo que no entra en el if. Si estamos en el padre, entra
	//en la condición y se sale del bucle con break
	if(childpid)
		break;
}
~~~

Por tanto, vamos creando hijos con un bucle *for*. Recordemos que en los hijos se va copiando el código del padre. Como se sale el bucle cuando estamos en el padre, lo que hace este fragmento de código es crear un hijo y en el padre no hace nada. En el hijo, se crea otro hijo de nuevo. Ahora, el proceso hijo tiene un hijo que acaba de crear, y él mismo, una vez creado el hijo, no hace nada. 
Por tanto, lo que se hace es crear un proceso detrás de otro recursivamente. De manera que debajo del padre tenemos un hijo, debajo otro hijo, y así sucesivamente. Sería algo así.
padre->hijo->hijo->hijo.....

Es un poco lioso al principio, pero leyéndolo detenidamente se acaba comprendiendo.

~~~c
/*
Jerarquía de procesos tipo 2
*/
for(i=1; i < nprocs; i++) {
	if((childpid = fork()) == -1) {
		if((childpid = fork()) == -1) {
			fprintf(stderr,"Could not create child %d: %s\n",i,strerro(errno));
		exi(-1);
		}

	if(!childpid)
		break;
}
~~~

Este otro fragmento de código hace justo lo contrario. Cuando está en el hijo, no hace nada, mientras que en el padre itera para crear otor hijo. Por tanto, tenemos un proceso padre, del que cuelgan, al mismo nivel, todos sus hijos

## Trabajo con llamadas al sistema wait, waitpid y exit
Con estas llamadas podemos sincronizar los procesos hijos con los procesos padre. Por ejemplo, podemos decirle al padre que se espere a que acabe su hijo antes de continuar con su ejecución, con la orden ``waitpid``.
En el ejercicio 4, se pide implementar un programa que cree 5 procesos hijo, identificándose con un mensaje en la salida estándar(la terminal). El padre debe esperar a la finalización de los hijos. Cada vez que detecte la finalización de un hijo, informando de los hijos que les quedan vivos.
~~~c
#include<sys/types.h>
#include<unistd.h>
#include<stdio.h>
#include<errno.h>
#include<stdlib.h>
int main(){
	int i, estado;
	pid_t PID;
	//CREAMOS HIJOS
	for(i=0; i<5; i++){
		if((PID = fork())<0){
			perror("Error en fork\n");
			exit(-1);
		}

		if(PID==0) {//Hijo imprime y muere
			printf("Soy el hijo PID = %i\n", getpid());
			exit(0);
		}
	}
	//ESPERAMOS HIJOS. Sabemos que estamos en el padre ya que si en el hijo se hace exit(0)
	for(i=4; i>=0; i--){
		PID = wait(&estado);
		printf("Ha finalizado mi hijo con PID = %i\n", PID);
		printf("Solo me quedan %i hijos vivos\n", i);
		}
}
~~~
Primero se crean los hijos. Una vez creados, el padre los espera. Podríamos haber metido el segundo bucle *for* en un condicional con ``if(PID > 0)```pero dado que dentro del hijo, hacemos ``exit(0)``, esto no es necesario.

La orden ``wait`` espera a que finalice un hijo. Cuando un hijo termina, se almacena en la variable *estado* que ha terminado, devoldiendo ``wait`` el PID del proceso que ha terminado, para almacenarlo en PID. La orden *wait* espera a que termine un hijo, mientras no termina, actualizo su estado en la variable *estado*.
Si queremos esperar a un hijo concreto, debemos usar la orden ``waitpid``, que nos permite seleccionar el hijo al que queremos esperar. El uso de *wait* es equivalente a declarar *waitpid* de la siguiente manera:
~~~c
waitpid(-1, &estado, 0)
~~~

Por defecto, *waitpid* espera a que termine el proceso, especificado mediante su PID, dado en el primer argumento, aunque esto puede modificarse mediante opciones especificadas en el tercer argumento. Para más información, puede consultarse el man.

Por tanto, como hemos creado 5 hijos, iteramos 5 veces sobre *wait*, cada vez que se detecta que ha acabado uno, informa del PID del hijo que ha acabado e informa de los hijos que quedan vivos.


El ejercicio se podría haber interpretado de otra forma. El código presentado a continuación crea un hijo. El padre, espera a que muera el hijo, y una vez que el hijo ha acabado, procede a crear el siguiente hijo. De eesta forma, el padre nunca tiene más de un hijo a la vez:
~~~c
#include<sys/types.h>	
#include<unistd.h>		
#include<stdio.h>
#include<errno.h>
#include<stdlib.h>

int main() {
  int status;
  pid_t PID;
  for(int i = 5; i > 0; --i) {
    if((PID = fork()) <0) {
      perror("Error en fork\n");
      exit(-1);
    }
    if(PID==0) {
      printf("\nSoy el hijo, mi PID es: %d\n", getpid());
      printf("\nAcabo como proceso\n");
      exit(0);
    } else {
      // Esperamos al primer hijo i
      //Esta versión es para que el padre se espere a que acabe el hijo antes
      //de crear el siguiente proceso hijo
      waitpid(PID, &status, 0);
      printf("\nHa finalizado mi hijo con PID = %i\n", PID);
      int n;
      n = i - 1;
      printf("\nSolo me quedan %i hijos vivos\n", n);

    }
  }

  return 0;
}
~~~

En el siguiente ejercicio, se propone esperar primero a los hijos creados en orden impar(1º,3º,5º) y después a los de orden par (2º,4º). Por tanto, tenemos que utilizar la orden *waitpid* en vez de *wait*, ya que vamos a esperar a un hijo concreto. En concreto, vamos a esperar a los hijos así: 1-3-5-2-4, donde el 1 es el primer hijo creado, el 3 es el tercero creado...

La idea es tener un vector de PIDs y esperar a los procesos iterando primero sobre los índices pares y después sobre los impares. Recordemos que los vectores en C empiezan en 0, por lo que la posición 1 corresponderá al hijo 2, no al creado en primer lugar. El código viene comentado.
~~~c
#include<sys/types.h>	
#include<unistd.h>		
#include<stdio.h>
#include<errno.h>
#include<stdlib.h>

int main() {
  int status;
	//Vector para almacenar los PIDs de los hijos
  int PIDs[5];
  pid_t PID;
  int i;
  int hijos = 5;

  for(i = 0; i < hijos; i++) {
    if((PIDs[i] = fork()) <0) {
      perror("Error en fork\n");
      exit(-1);
    }
    if(PIDs[i]==0) {
      printf("\nSoy el hijo, mi PID es: %d\n", getpid());
      printf("\nAcabo como proceso\n");
      exit(0);
    }
  }

	//Esperamos a los hijos impares(1,3,5), iterando sobre los índices correspondientes(0,2,4 respectivamente)
  for(i=0; i < 5; i = i+2) {
    waitpid(PIDs[i],&status);
    printf("Acaba de finalizar mi hijo con PID = %d y estado %d\n", PIDs[i], status);
    printf("Solo me quedan %d hijos vivos, este es el %do hijo.\n", --  hijos, i+1);
  }

	//Esperamos a los hijos pares(2,4), iterando sobre los índices correspondientes(1,3 respectivamente)
  for(i=1; i < 4; i = i+2) {
    waitpid(PIDs[i],&status);
    printf("Acaba de finalizar mi hijo con PID = %d y estado %d\n", PIDs[i], status);
    printf("Solo me quedan %d hijos vivos, este es el %do hijo.\n", --  hijos, i+1);
  }

  return 0;
}
~~~

## Familia de llamadas al sistema **exec**
En C, existen un conjunto de llamadas al sistema que permiten ejecutar un programa distinto al que se está ejecutando en el programa padre, la familia de llamadas exec. Cada una de estas llamadas tienen unos argumentos y se comportan de una manera concreta. 
Por ejemplo, a ``execl`` le pasamos los argumentos del programa, uno a uno como argumentos, mientras que a execv le pasamos un vector, estando todos los argumentos del programa están en dicho vector. Em ambos casos, el último argumento en el primer caso, o la última componente en el segundo caso, deben ser un *NULL*.

Es importante destacar que cuando llamamos a una orden ``exec``, se destruye el espacio de direcciones de nuestro programa para crear un nuevo, por lo que invocar a ``exec`` es lo último que debemos hacer en nuestro programa, ya que lo que esté a continuación no se invocará. 

En el siguiente programa, se crea un hijo que ejecutará una orden. El padre, espera a que acabe el hijo antes de finalizar.
~~~c
//tarea5.c
//Trabajo con llamadas al sistema del Subsistema de Procesos conforme a POSIX 2.10

#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>
#include<errno.h>
#include <stdlib.h>


int main(int argc, char *argv[])
{
pid_t pid;
int estado;

if( (pid=fork())<0) {
	perror("\nError en el fork");
	exit(EXIT_FAILURE);
}
else if(pid==0) {  //proceso hijo ejecutando el programa
	if( (execl("/usr/bin/ldd","ldd","./tarea5",NULL)<0)) {
		perror("\nError en el execl");
		exit(EXIT_FAILURE);
	}
}
//wait(&estado);
/*
<estado> mantiene información codificada a nivel de bit sobre el motivo de finalización del proceso hijo
que puede ser el número de señal o 0 si alcanzó su finalización normalmente.
Mediante la variable estado de wait(), el proceso padre recupera el valor especificado por el proceso hijo como argumento de la llamada exit(), pero desplazado 1 byte porque el sistema incluye en el byte menos significativo
el código de la señal que puede estar asociada a la terminación del hijo. Por eso se utiliza estado>>8
de forma que obtenemos el valor del argumento del exit() del hijo.
*/

printf("\nMi hijo %d ha finalizado con el estado %d\n",pid,estado>>8);

exit(EXIT_SUCCESS);

}
~~~

Para el ejercicio 7, se propone escribir un programa que acepte como argumentos el nombre de un programa, sus argumentos si los tiene, y opcionalmente la cadena *bg*. Nuestro programa deberá ejecutar el programa pasado como primer argumento en *foreground* si no se especifica la cadena *bg* y en *background* en caso contrario. Si el programa tiene argumentos hay que ejecutarlos con éstos.
~~~c
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>
#include<errno.h>
#include<stdbool.h>
#include<stdlib.h>
#include<string.h>


int main(int argc, char* argv[]) {
  if(argc <= 1) {
    printf("\nNúmero de argumentos inválido.\n");
    exit(-1);
  }

  int status;
  int pid;

  int id=fork();
  if(id==0) {
    //child does work her
  int nArgumentos = argc;
  //Si el último argumento es bg, tenemos en cuenta todos los argumentos menos el último, bg
  if(strcmp(argv[argc-1],"bg")==0)
    nArgumentos = argc - 1;
  char* argumentos[nArgumentos];
  int i;

  //Metemos los argumentos en un vector, que se los pasaremos algunos a exec.
  for (i = 0; i < nArgumentos - 1; ++i)
    argumentos[i] = argv[i+1];
  argumentos[nArgumentos - 1] = NULL;

  if((execv(argv[1], argumentos) < 0)) {
      perror("\nError al hacer exec()");
      return(-1);
    }
  } else {
    //Parent does work here
    printf("PADRE: Soy el proceso padre y mi pid sigue siendo: %d\n", getpid());
    //Si el último argumento no es *bg*, se ejecuta en primer plano, por lo que el padre se espera al hijo.
    if(!strcmp(argv[argc-1],"bg")==0)
      waitpid(id,&status,0);
  }

  return 0;
}
~~~
*El último if que se encarga de mantener o no el programa en background, ha sido sacado de stackoverflow, pero no he conseguido recuperar el enlace de donde fue sacado*









	
	








































