# Sesión 2: llamadas al sistema para el Sistema de Archivos (parte II)

## Gestión de permisos
La llamada al sistema **umask** fija la máscara de permisos para el proceso y devuelve el valor previamente establecido. La llamada **chmod** trabaja sobre un archivo especificado por su pathname, mientras que la función **fchmod** opera sobre un archivo que  ha sido previamente abierto con *open*.
Para entender un poco mejor las máscaras puede consultarse el siguiente enlace [https://wiki.archlinux.org/index.php/Umask_(Espa%C3%B1ol)](https://wiki.archlinux.org/index.php/Umask_(Espa%C3%B1ol))

Como primer ejercicio se proporciona un programa para decir qué hace. El programa basicamente cambia los permisos de dos archivos dados. Se han añadido comentarios al código para facilitar su entendimiento.
~~~c
#include<sys/types.h>
#include<unistd.h>		
#include<sys/stat.h>
#include<fcntl.h>	
#include<stdio.h>
#include<errno.h>
#include<stdlib.h>

int main(int argc, char *argv[]) {
	int fd1,fd2;
	//Estructura para manejar los atributos de un archivo
	struct stat atributos;

	//CREACION DE ARCHIVOS
	if( (fd1=open("archivo1",O_CREAT|O_TRUNC|O_WRONLY,S_IRGRP|S_IWGRP|S_IXGRP))<0) {
		printf("\nError %d en open(archivo1,...)",errno);
		perror("\nError en open");
		exit(EXIT_FAILURE);
	}

	umask(0);
	if( (fd2=open("archivo2",O_CREAT|O_TRUNC|O_WRONLY,S_IRGRP|S_IWGRP|S_IXGRP))<0) {
		printf("\nError %d en open(archivo2,...)",errno);
		perror("\nError en open");
		exit(EXIT_FAILURE);
	}

	//Guardamos los atributos dedl archivo 1 en atributos
	if(stat("archivo1",&atributos) < 0) {
		printf("\nError al intentar acceder a los atributos de archivo1");
		perror("\nError en lstat");
		exit(EXIT_FAILURE);
	}
	//Cambiamos los permisos del fichero uno para que pase a tener S_ISGID. También tendrá los permisos que tuviera antes(almacenados en atributos.st_mode) tras aplicarle una máscara. De máscara usamos S_IXGRP(es decir, 00010)
	if(chmod("archivo1", (atributos.st_mode & ~S_IXGRP) | S_ISGID) < 0) {
		perror("\nError en chmod para archivo1");
		exit(EXIT_FAILURE);
	}
	//El archivo2 tendrá los permisos S_IRWXU,S_IRGRP,S_IWGRP y S_IROTH
	if(chmod("archivo2",S_IRWXU | S_IRGRP | S_IWGRP | S_IROTH) < 0) {
		perror("\nError en chmod para archivo2");
		exit(EXIT_FAILURE);
	}

	return EXIT_SUCCESS;
}
~~~

## Trabajo con funciones estándar de manejo de directorios
Una vez leídas las funciones de manejo de direcotorios(opendir,readdir...), vamos a resolver el ejercicio número 3.Se pide construir un programa que reciba como argumentos un directorio y un número en octal de 4 dígitos. El programa deberá cambiar los permisos de todos los archivos que contiene el directorio a los permisos especificados en el segun argumento. 
Además, el programa debe imprimir los permisos que tenía el archivo antes del cambio y los nuevos tras haberlos modificado. Si no se han podido cambiar los permisos, mostrar un mensaje de error con los permisos antiguos.
El código es el siguiente, explicado con comentarios:
~~~c
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <errno.h>
#include <dirent.h>
#include <string.h>

int main(int argc, char *argv[]) {
	//Puntero que apuntará al directorio
	DIR *directorio;
	//Estructura que lleva un puntero a una entrada del directorio
	struct dirent *direntp;
	struct stat atributos;
	//Los nuevos permisos que contendrá el archivo
	int permisos;

	//Los usaremos para poder almacenar lo que vayamos a imprimir y guardar información
	char cadena[100];
	char cadena2[200];
  	extern int errno;

	if(argc != 3 || strlen(argv[2]) != 4) {
		printf("Argumentos inválidos, indicar pathname del directorio y permisos en octal de 4 dígitos");
		return -1;
  	}
	
	//Guardamos los permisos nuevos. Con 8 indicamos que lo guarde en base 8 el número (octal)
	permisos = strtol(argv[2],NULL,8);


	//Primero tenemos que abrir el directorio
	char *pathname = argv[1];
	directorio = opendir(pathname);

	if (directorio == NULL) {
	 printf("Error: No se puede abrir el directorio\n");
	 exit(-1);
 	}

	//Ahora tenemos que recorrer el directorio por dentro. Nos moveremos con un puntero, manejado con la estructura struct dirent. El puntero apunta a una entrada del directorio
 	//Conforme leemos el directorio, el puntero avanza hacia la siguiente entrada. Por tanto, tenemos que leer del directorio hasta llegar al final, quedando el puntero apuntando a NULL.

	direntp = readdir(directorio);
	while(direntp != NULL) {
		//Metemos en cadena la ruta del archivo que vamos a tratar, apuntado por la estructura direntp
		//d_name contiene el nombre del archivo, por lo que nos queda un mensaje tipo /ruta-del-directorio/nombre-archivo
		sprintf(cadena,"%s/%s",pathname,direntp->d_name);
		//Esta misma ruta del archivo la usamos para almacenar sus metadatos en atributos
		if(stat(cadena, &atributos) < 0) {
		 printf("\nError al intentar acceder a los atributos del fichero");
		 perror("\nError en lstat");
		 exit(-1);
	   	}

		//Ahora miramos si es un archivo regular(si es por ejemplo, una carpeta, no le hacemos nada)
		if(S_ISREG(atributos.st_mode)) {
		//A la hora de imprimir, imprimos el nombe del archivo, no su ruta
		sprintf(cadena2,"%s",direntp->d_name);
		//Imprimimos el nombre del archivo y sus permisos, que aún no los hemos cambiado
		printf("%s: %o ",cadena2,atributos.st_mode);

		//Cambiamos los permisos al fichero, especificado por su ruta, a los permisos que le dimos como argumento
		if(chmod(cadena,permisos) < 0) {
		   printf("Error: %s\n",strerror(errno));;
		} else {
			//Actualizo en el stat atributos los nuevos atributos del archivo(sus permisos han cambiado) e imprimo los nuevos permisos.
			stat(cadena,&atributos);
			printf("%o \n",atributos.st_mode);
			}
		}


	//Antes de salir del bucle, volvemos a leer del directorio, para poder avanzar
	direntp = readdir(directorio);
	}

	closedir(directorio);
	return 0;
}

~~~
Un ejemplo de ejecución para cambiar los permisos a 777 (en octal, 1411), sería ``/ejercicio2 /home/victor/Descargas 1411``.
*Importante: para rutas de directorio muy largas, puede dar un core. Para resolverlo, aumentar el tamaño de cadena*

	
Veamos otro ejemplo de uso de señales al sistema para manejar directorios. En el siguiente ejercicio, se pide que dado un directorio, se devuelve la cantidad de archivos regulares que contiene, tanto él mismo como sus subdirectorios.
Para resolver el ejercicio necesitamos una función recursiva, que se llame a sí misma en caso de encontrar un directorio mientras estamos leyendo el directorio. Es importante tener en cuenta que un directorio siempre tiene dos entradas, que indican el directorio propio y el directorio padre, representandos con ``.`` y ``..`` respectivamente.
Puede comprobarse con un ``ls -la```en la terminal. Por tanto, debemos verificar que un subdirectorio no es ninguno de esos dos para evitar caer en una recursividad infinita.

Los archivos que se cuentan tienen que cumplir que tengan permiso de ejecución para *grupos* y para *otros*, de lo contrairo, no los tendremos en cuenta. También tenemos que imprimir el nombre del archivo, número de inodo y tamaño total que ocupan todos los archivos que cumplen la condición citada. 
Para comprobar los permisos, tenemos que definir un macro.
~~~c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <dirent.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

//Macro para comprobar los permisos. Tener permisos de ejecución para grupos y otros equivale a tener los permisos 011
#define criterio(mode) (((mode) & 011) == 011)

//Para evitar tener variables globales, pasamos por referencia n y tam, para almacenar el contador y el tamaño
void recorrerDir(char *path, int *n, int *tam) {

	struct stat atributos;
	DIR *direct;
	struct dirent *dir;
	char nombre[256];

	direct = opendir(path);

	if(direct == NULL) {
		printf("\nError al abrir el directorio");
		exit(-1);
	}

	dir = readdir(direct);

	while(dir != NULL) {
		//Comprobamos que la entrada que vamos a leer del directorio, no es ni el propio directorio ni el directorio padre
		if(strcmp(dir->d_name, ".") != 0 && strcmp(dir->d_name, "..") != 0) {
			//Guardamos la ruta de la entrada
			sprintf(nombre,"%s/%s",path,dir->d_name);
			//Guardamos los atributos de la entrada
			if(stat(nombre, &atributos) < 0) {
				printf("\nError al acceder a los atributos de %s\n", nombre);
				exit(-1);
			}
			
			//Si es un archivo y tiene los permisos, incrementamos el contador y sumamos su tamaño a la variable que lleva el tamaño
			if(S_ISREG(atributos.st_mode) && criterio(atributos.st_mode)) {
				//Imprimos el nombre del archivo y su número de inodo
				printf("%s %ld \n\n", nombre, atributos.st_ino);
				(*n)++;
				//El tamaño del archivo se encuentra almacenadi en su estructura stat atributos, en el campo st_size
				(*tam) += (int) atributos.st_size;
			}
			//Si no es regular, miramos si es directorio para llamar de nuevo la función. Sino, no hacemos nada más
			else if(S_ISDIR(atributos.st_mode)) {
				//LLamamos de nuevo a la función recursivamente
				recorrerDir(nombre,n,tam);
			}
		}

		//Leemos la siguiente entrada para la próxima iteracción
		dir = readdir(direct);
		}

	closedir(direct);
}


//Realizamos un main para probar la función
int main(int argc, char *argv[]) {
	printf("Los inodos son: \n\n");

	int n = 0;
	int tam = 0;
	//Variables que llevan el contador y el tamaño ocupado por los ficheros, respectivamente

	if(argc == 2)
		recorrerDir(argv[1],&n,&tam);
	//Si no se le pasan argumentos, asumimos el directorio actual
	else
		recorrerDir(".",&n,&tam);

  printf("Existen %d archivos regulares con permiso de ejecución para grupos y otros\n\n", n);
  printf("El tamaño total ocupado por dichos archivos es %d bytes\n\n", tam);
}
~~~

## La llamada ntfw()
Esta llamada permite recorrer recursivamente un sub-árbol y realizar algunas operaciones sobre ellos, si necesidad de hacerlo a mano.
Realizamos el ejercicio anterior implementando la función ``ntfw()``. Se presenta el código y posteriormente se explica.
~~~c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <dirent.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ftw.h>
#include <errno.h>

#define criterio(mode) (((mode) & 011) == 011)

//Variables para llevar el conteo
static int n = 0;
static int tam = 0;

int count(const char *path, const struct stat* stat, int flags, struct FTW *ftw) {
	//Esto es igual que en el ejercicio anterior
	if(S_ISREG(stat->st_mode) && criterio(stat->st_mode)){
	    printf("%s %ld \n\n", path, stat->st_ino);
	    n++;
	    tam += (int) stat->st_size;
	  }

  return 0;
}

int main(int argc, char *argv[]) {
  printf("Los inodos son: \n\n");

	if(nftw(argc >= 2 ? argv[1] : ".",count,20,0) != 0)
		perror("\nError en ntfw");

  printf("Existen %d archivos regulares con permiso de ejecución para grupos y otros\n\n", n);
  printf("El tamaño total ocupado por dichos archivos es %d bytes\n\n", tam);

  exit(0);
}
~~~

El ejercicio sigue la misma estructura que el ejemplo propuesto previamente en el guión de prácticas. Definimos una función *count* que recibe la ruta de la entrada del directorio. Comprueba si es un archivo regular y tiene los permisos correspondientes, en cuyo caso, incrementa las variables correspondientes e imprime el inodo del archivo.

En el main, llamamos a la función *ntfw*:
~~~c
nftw(argc >= 2 ? argv[1] : ".", count, 20, 0) != 0
~~~
La función, recibe como primer argumento un path que contendrá el directorio a examinar. Si le pasamos al programa un directorio como argumento, a la función le pasamos dicho path, sino, asumimos el directorio actual. Obsérvese que esta disyuntiva se ha realizado implícitamente, podríamos haber hecho las comprobaciones fuera y pasarle a *ntfw* una cadena con el directorio a examinar.
El segundo argumento es la función que va a realizar *ntfw* sobre el directorio al recorrerlo recursivamente, la cual recibe automáticamente el path dado a *ntfw*.
El tercer argumento, *20*, es el número máximo de directorios abiertos sobre los que puede estar trabajando la función. Cuanto mayor sea este número, más rápida será la ejecución, pero a costa de consumir más memoria. Finalemente, el cuarto argumento, indica que si la función *count* ha ido bien devuelve un cero. En caso contrario, si se ha producido algún tipo de error, se aborta la ejecución.



















			















