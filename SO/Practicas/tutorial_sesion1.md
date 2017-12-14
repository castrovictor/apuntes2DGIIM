# Sesión 1: Manejo de archivos mediante llamadas al sistema.

>Antes de continuar con este tutorial, es recomendable leer la sesión 1 (módulo II) correspondiente a la guía de prácticas de Sistemas Operativos. El desarrollo del tutorial se enfocará en la explicación de conceptos y ejemplos prácticos, para ver la sintaxis de las órdenes con sus opciones puede consultarse el *man*.
>El objetivo es aprender a abrir, cerrar y crear ficheros mediante llamadas al sistema, así como leer y escribir en los mismos.

Veamos un primer ejemplo, en el que creamos un fichero y escribiremos una frase dentro de él.
~~~c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<errno.h>
#include<string.h> //Para poder usar strlen

int main(int argc, char *argv[]) {
	char cadena1[150]="¡Hola!Soy Víctor, he creado estos tutoriales para ayudar a comprender las praćticas de 	Sistemas Operativos. Espero que te sirva :).\n";
	char cadena2[150]="No dudes en compartir y modificar lo que creas conveniente, a través de github.com/libreim/apuntesDGIIM\n";
	int f1;

	if((f1=open("archivo_salida",O_CREAT|O_TRUNC|O_WRONLY,S_IRUSR|S_IWUSR)) < 0) {
		printf("\nError %d en open",errno);
		perror("\nError en open");
		exit(EXIT_FAILURE);
	}

	int longitud = strlen(cadena1);

	int escritos = write(f1,cadena1,longitud);

	if(escritos != longitud) {
		perror("\nError en write.");
		exit(EXIT_FAILURE);
	}
	
/* Esto se introduce en el código en la modificación posteriormente explicada
	if(lseek(f1,0,SEEK_SET) < 0) {
		perror("\nError en lseek");
		exit(EXIT_FAILURE);
	}
*/

	longitud = strlen(cadena2);

	if(write(f1,cadena2,longitud) != longitud) {
		perror("\nError en segundo write.");
		exit(EXIT_FAILURE);
	}

	return EXIT_SUCCESS;
	}
~~~

El objetivo del programa es escribir los dos mensajes en un fichero, *archivo_salida*, que vienen dados por dos cadenas de caracteres. 
Primero tenemos que abrir el fichero de salida. Si no existe, lo creamos. El fichero de salida lo referenciamos mediante la variable *f1*.
~~~c
f1=open("archivo_salida",O_CREAT|O_TRUNC|O_WRONLY,S_IRUSR|S_IWUSR)
~~~

En primer lugar, la orden ``open`` devuelve el descriptor del fichero de salida, que le damos el nombre de *fichero_salida*. Se almacenará en el entero *f1* que declaramos antes. Si es negativo, es que se ha producido un error al abrirlo, de ahí la comprobación del *if* y el aborto del programa de cumplirse la comprobación.
1.El primer argumento de la orden ``read`` es la ruta del fichero de salida. Como no le estamos dando ninguna ruta, se asume el directorio de trabajo actual, desde donde ejecutamos el programa.
2.El segundo argumento son las opciones con las que que queremos abrirlo. En nuestro caso, *O_CREAT* es porque no existe el fichero, queremos crearlo. Con *O_TRUNC* decidimos que si el fichero estaba creaod y contenía información, la sobreescribimos. Si queríamos conservarla, podríamos haber usado la opción *O_APPEND*, que situará el offset al final del fichero antes de cada invocación de ``writte``. Como solo vamos a usarlo para escribir, lo abrimos con permisos de escritura únicamente con *O_WRONLY*.
3. En el tercer argumento decidimos los permisos que tendrá el fichero, en nuestro caso, le damos permisos de lectura y escritura al usuario con *S_IRUSR* y *S_IWUSR* respectivamente.

Ahora pasamos a escribir en el fichero. Con la función ``strlen`` conocemos la longitud exacta de la primera cadena. La función ``write```devuelve el número bytes escritos, por eso si no es igual a la longitud de la cadena, es que se ha producido un error en la escritura y abortamos el proceso.
La función ``write```escribe en el fichero dado como primer argumento, mediante su descriptor de fichero, *f1*, los n primeros bytes indicados mediante el tercer argumento, los cuales los extrae del segundo argumento. Como queremos escribir la cadena1 entera, pues le decimos que coja tantos como indica longitud, que era la longitud de la cadena obtenida con strlen.

El número de escritos no es necesario almacenarlo en una variable, podemos escribir y hacer la comprobación directamente:
~~~c
longitud = strlen(cadena2);

if(write(f1,cadena2,longitud) != longitud) {
	perror("\nError en segundo write.");
	exit(EXIT_FAILURE);
	}
~~~
Con este segundo *write*, hemos escrito la segunda cadena al final de la primera, ya que cuando hicimos el primer *write* el offset se quedó apuntando al final del fichero, por lo que la siguiente escritura empezó en esa posición.













