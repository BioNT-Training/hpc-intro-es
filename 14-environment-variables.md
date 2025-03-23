---
title: Variables de entorno
teaching: 10
exercises: 5
---




::::::::::::::::::::::::::::::::::::::: objectives

- Comprender cómo se implementan las variables en el shell
- Leer el valor de una variable existente
- Crear nuevas variables y cambiar sus valores
- Cambiar el comportamiento de un programa mediante una variable de entorno
- Explica cómo el shell utiliza la variable `PATH` para buscar ejecutables

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Cómo se establecen y acceden a las variables en el intérprete de comandos Unix?
- ¿Cómo puedo utilizar variables para cambiar la forma en que se ejecuta un programa?

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::: callout

## Procedencia del episodio

Este episodio ha sido remezclado del episodio [Shell Extras sobre Variables del Shell](https://github.com/carpentries-incubator/shell-extras/blob/gh-pages/_episodes/08-environment-variables.md) y del episodio [HPC Shell sobre scripts](https://github.com/hpc-carpentry/hpc-shell/blob/gh-pages/_episodes/05-scripts.md).

::::::::::::::::::::::::::::::::::::::::::::::::::

El shell es sólo un programa, y como otros programas, tiene variables. Esas variables controlan su ejecución, de modo que cambiando sus valores puedes cambiar cómo se comporta el shell (y con un poco más de esfuerzo cómo se comportan otros programas).

Las variables son una buena forma de guardar información bajo un nombre al que puedas acceder más tarde. En lenguajes de programación como Python y R, las variables pueden almacenar prácticamente cualquier cosa que se te ocurra. En el shell, normalmente sólo almacenan texto. La mejor manera de entender cómo funcionan es verlas en acción.

Empecemos ejecutando el comando `set` y observando algunas de las variables en una sesión típica de shell:

```bash
set
```

```output
COMPUTERNAME=TURING
HOME=/home/vlad
HOSTNAME=TURING
HOSTTYPE=i686
NUMBER_OF_PROCESSORS=4
PATH=/Users/vlad/bin:/usr/local/git/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin
PWD=/home/vlad
UID=1000
USERNAME=vlad
...
```

Como puedes ver, hay bastantes - de hecho, cuatro o cinco veces más de las que se muestran aquí. Y sí, usar `set` para *mostrar* cosas puede parecer un poco extraño, incluso para Unix, pero si no le das ningún argumento, bien podría mostrarte cosas que *podrías* configurar.

Cada variable tiene un nombre. Todos los valores de las variables del shell son cadenas, incluso aquellas (como `UID`) que parecen números. Depende de los programas convertir estas cadenas a otros tipos cuando sea necesario. Por ejemplo, si un programa quisiera averiguar cuántos procesadores tiene el ordenador, convertiría el valor de la variable `NUMBER_OF_PROCESSORS` de una cadena a un número entero.

## Mostrar el valor de una variable

Mostremos el valor de la variable `HOME`:

```bash
echo HOME
```

```output
HOME
```

Esto sólo imprime "HOME", que no es lo que queríamos (aunque es lo que pedimos). Intentemos esto en su lugar:

```bash
echo $HOME
```

```output
/home/vlad
```

El signo de dólar indica al shell que queremos el *valor* de la variable en lugar de su nombre. Esto funciona igual que los comodines: el shell realiza la sustitución *antes* de ejecutar el programa que le hemos pedido. Gracias a esta expansión, lo que realmente ejecutamos es `echo /home/vlad`, que muestra lo correcto.

## Creación y modificación de variables

Crear una variable es fácil - simplemente asignamos un valor a un nombre usando "=" (¡sólo tenemos que recordar que la sintaxis requiere que *no* haya espacios alrededor de `=`!)

```bash
SECRET_IDENTITY=Dracula
echo $SECRET_IDENTITY
```

```output
Dracula
```

Para cambiar el valor, basta con asignar uno nuevo:

```bash
SECRET_IDENTITY=Camilla
echo $SECRET_IDENTITY
```

```output
Camilla
```

## Variables de entorno

Cuando ejecutamos el comando `set` vimos que había muchas variables cuyos nombres estaban en mayúsculas. Esto es porque, por convención, las variables que también están disponibles para ser usadas por *otros* programas reciben nombres en mayúsculas. Tales variables se llaman *variables de entorno* ya que son variables del shell que se definen para el shell actual y son heredadas por cualquier shell o proceso hijo.

Para crear una variable de entorno necesitas `export` una variable del shell. Por ejemplo, para hacer que nuestra `SECRET_IDENTITY` esté disponible para otros programas que llamemos desde nuestra shell podemos hacer:

```bash
SECRET_IDENTITY=Camilla
export SECRET_IDENTITY
```

También puede crear y exportar la variable en un solo paso:

```bash
export SECRET_IDENTITY=Camilla
```

::::::::::::::::::::::::::::::::::::::: challenge

## Uso de variables de entorno para cambiar el comportamiento del programa

Establece una variable de shell `TIME_STYLE` para que tenga un valor de `iso` y comprueba este valor usando el comando `echo`.

Ahora, ejecuta el comando `ls` con la opción `-l` (que da un formato largo).

`export` la variable y vuelve a ejecutar el comando `ls -l`. ¿Notas alguna diferencia?

::::::::::::::: solution

## Solución

La variable `TIME_STYLE` no es *vista* por `ls` hasta que es exportada, momento en el cual es utilizada por `ls` para decidir qué formato de fecha utilizar al presentar la marca de tiempo de los ficheros.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

Puedes ver el conjunto completo de variables de entorno en tu sesión de shell actual con el comando `env` (que devuelve un subconjunto de lo que nos dio el comando `set`). **El conjunto completo de variables de entorno se denomina *entorno de ejecución* y puede afectar al comportamiento de los programas que ejecutas**.

::::::::::::::::::::::::::::::::::::::: challenge

## Variables de entorno de trabajo

Cuando `Slurm` ejecuta un trabajo, establece una serie de variables de entorno para el trabajo. Una de ellas nos permitirá comprobar desde qué directorio se envió nuestro script de trabajo. La variable `SLURM_SUBMIT_DIR` se establece en el directorio desde el que se envió nuestro trabajo. Usando la variable `SLURM_SUBMIT_DIR`, modifica tu tarea para que imprima la ubicación desde la que se envió la tarea.

::::::::::::::: solution

## Solución

```bash
[yourUsername@login1 ~] nano example-job.sh
[yourUsername@login1 ~] cat example-job.sh
```

```output
#!/bin/bash
#SBATCH -t 00:00:30

echo -n "This script is running on "
hostname

echo "This job was launched in the following directory:"
echo ${SLURM_SUBMIT_DIR}
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

Para eliminar una variable o variable de entorno puede utilizar el comando `unset`, por ejemplo:

```bash
unset SECRET_IDENTITY
```

## Variable de entorno `PATH`

Del mismo modo, algunas variables de entorno (como `PATH`) almacenan listas de valores. En este caso, la convención es utilizar dos puntos ':' como separador. Si un programa quiere los elementos individuales de una lista de este tipo, es responsabilidad del programa dividir el valor de la cadena de la variable en trozos.

Echemos un vistazo más de cerca a la variable `PATH`. Su valor define la ruta de búsqueda de ejecutables del intérprete de comandos, es decir, la lista de directorios en los que el intérprete de comandos busca programas ejecutables cuando se escribe el nombre de un programa sin especificar en qué directorio se encuentra.

Por ejemplo, cuando escribimos un comando como `analyze`, el intérprete de órdenes tiene que decidir si ejecuta `./analyze` o `/bin/analyze`. La regla que utiliza es simple: el intérprete de comandos comprueba cada directorio de la variable `PATH` por turnos, buscando un programa con el nombre solicitado en ese directorio. En cuanto encuentra una coincidencia, deja de buscar y ejecuta el programa.

Para mostrar cómo funciona, aquí están los componentes de `PATH` listados uno por línea:

```output
/Users/vlad/bin
/usr/local/git/bin
/usr/bin
/bin
/usr/sbin
/sbin
/usr/local/bin
```

En nuestro ordenador, en realidad hay tres programas llamados `analyze` en tres directorios diferentes: `/bin/analyze`, `/usr/local/bin/analyze`, y `/users/vlad/analyze`. Como el shell busca en los directorios en el orden en que están listados en `PATH`, encuentra `/bin/analyze` primero y lo ejecuta. Observa que *nunca* encontrará el programa `/users/vlad/analyze` a menos que introduzcamos la ruta completa al programa, ya que el directorio `/users/vlad` no está en `PATH`.

Esto significa que puedo tener ejecutables en un montón de lugares diferentes, siempre y cuando recuerde que tengo que actualizar mi `PATH` para que mi shell pueda encontrarlos.

¿Qué pasa si quiero ejecutar dos versiones diferentes del mismo programa? Como comparten el mismo nombre, si añado ambos a mi `PATH` el primero que encuentre siempre ganará. En el próximo episodio aprenderemos a utilizar herramientas de ayuda para gestionar nuestro entorno de ejecución y hacer que esto sea posible sin tener que hacer un montón de contabilidad sobre cuál es o debería ser el valor de `PATH` (y otras variables de entorno importantes).

:::::::::::::::::::::::::::::::::::::::: keypoints

- Las variables del intérprete de órdenes se tratan por defecto como cadenas de caracteres
- Las variables se asignan utilizando "`=`" y se recuperan utilizando el nombre de la variable precedido por "`$`"
- Utilice "`export`" para poner una variable a disposición de otros programas
- La variable `PATH` define la ruta de búsqueda de la shell

::::::::::::::::::::::::::::::::::::::::::::::::::


