---
title: Ejecutar un trabajo paralelo
teaching: 30
exercises: 60
---




::::::::::::::::::::::::::::::::::::::: objectives

- Instale un paquete Python usando `pip`
- Prepara un script de envío de trabajos para el ejecutable paralelo.
- Lanzar trabajos con ejecución paralela.
- Registrar y resumir el tiempo y la precisión de los trabajos.
- Describa la relación entre el paralelismo de trabajos y el rendimiento.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Cómo se ejecuta una tarea en paralelo?
- ¿Qué ventajas aporta la ejecución en paralelo?
- ¿Cuáles son los límites de las ganancias de la ejecución en paralelo?

::::::::::::::::::::::::::::::::::::::::::::::::::

Ahora tenemos las herramientas que necesitamos para ejecutar un trabajo multiprocesador. Este es un aspecto muy importante de los sistemas HPC, ya que el paralelismo es una de las principales herramientas que tenemos para mejorar el rendimiento de las tareas computacionales.

Si se desconectó, vuelva a conectarse al clúster.

```bash
[you@laptop:~]$ ssh yourUsername@cluster.hpc-carpentry.org
```

## Instalar el programa Amdahl

Con el código fuente de Amdahl en el cluster, podemos instalarlo, lo que nos dará acceso al ejecutable `amdahl`. Muévete al directorio extraído, y luego utiliza el Instalador de Paquetes para Python, o `pip`, para instalarlo en tu directorio personal ("usuario"):

```bash
[yourUsername@login1 ~] cd amdahl
[yourUsername@login1 ~] python3 -m pip install --user .
```

::::::::::::::::::::::::::::::::::::::::: callout

## Amdahl es código Python

El programa Amdahl está escrito en Python, e instalarlo o utilizarlo requiere localizar el ejecutable `python3` en el nodo de inicio de sesión. Si no puede encontrarlo, intente listar los módulos disponibles usando `module avail`, cargue el apropiado, e intente el comando de nuevo.


::::::::::::::::::::::::::::::::::::::::::::::::::

### MPI para Python

El código Amdahl tiene una dependencia: **mpi4py**. Si aún no ha sido instalado en el cluster, `pip` intentará recoger mpi4py de Internet e instalarlo por ti. Si esto falla debido a un cortafuegos unidireccional, deberás recuperar mpi4py en tu máquina local y subirlo, tal y como hicimos para Amdahl.

:::::::::::::::::::::::::::::::::::::: discussion

## Recuperar y Subir `mpi4py`

Si la instalación de Amdahl falló porque mpi4py no pudo ser instalado, recupere el tarball de <https://github.com/mpi4py/mpi4py/tarball/master> luego `rsync` al cluster, extraiga, e instale:

```bash
[you@laptop:~]$ wget -O mpi4py.tar.gz https://github.com/mpi4py/mpi4py/releases/download/3.1.4/mpi4py-3.1.4.tar.gz
[you@laptop:~]$ scp mpi4py.tar.gz yourUsername@cluster.hpc-carpentry.org:
# or
[you@laptop:~]$ rsync -avP mpi4py.tar.gz yourUsername@cluster.hpc-carpentry.org:
```

```bash
[you@laptop:~]$ ssh yourUsername@cluster.hpc-carpentry.org
[yourUsername@login1 ~] tar -xvzf mpi4py.tar.gz  # extract the archive
[yourUsername@login1 ~] mv mpi4py* mpi4py        # rename the directory
[yourUsername@login1 ~] cd mpi4py
[yourUsername@login1 ~] python3 -m pip install --user .
[yourUsername@login1 ~] cd ../amdahl
[yourUsername@login1 ~] python3 -m pip install --user .
```

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::: discussion

## If `pip` Raises a Warning...

`pip` puede advertir que los binarios de su paquete de usuario no están en su PATH.

```warning
WARNING: The script amdahl is installed in "${HOME}/.local/bin" which is
not on PATH. Consider adding this directory to PATH or, if you prefer to
suppress this warning, use --no-warn-script-location.
```

Para comprobar si esta advertencia es un problema, utilice `which` para buscar el programa `amdahl`:

```bash
[yourUsername@login1 ~] which amdahl
```

Si el comando no devuelve ninguna salida, mostrando un nuevo prompt, significa que no se ha encontrado el fichero `amdahl`. Debe actualizar la variable de entorno llamada `PATH` para incluir la carpeta que falta. Edite el archivo de configuración de su shell de la siguiente manera, luego cierre la sesión en el cluster y vuelva a iniciarla para que surta efecto.

```bash
[yourUsername@login1 ~] nano ~/.bashrc
[yourUsername@login1 ~] tail ~/.bashrc
```

```output
export PATH=${PATH}:${HOME}/.local/bin
```

Después de volver a entrar en cluster.hpc-carpentry.org, `which` debería ser capaz de encontrar `amdahl` sin dificultades. Si ha tenido que cargar un módulo de Python, vuelva a cargarlo.


::::::::::::::::::::::::::::::::::::::::::::::::::

## ¡Ayuda!

Muchos programas de línea de comandos incluyen un mensaje de "ayuda". Pruébelo con `amdahl`:

```bash
[yourUsername@login1 ~] amdahl --help
```

```output
usage: amdahl [-h] [-p [PARALLEL_PROPORTION]] [-w [WORK_SECONDS]] [-t] [-e] [-j [JITTER_PROPORTION]]

optional arguments:
  -h, --help            show this help message and exit
  -p [PARALLEL_PROPORTION], --parallel-proportion [PARALLEL_PROPORTION]
                        Parallel proportion: a float between 0 and 1
  -w [WORK_SECONDS], --work-seconds [WORK_SECONDS]
                        Total seconds of workload: an integer greater than 0
  -t, --terse           Format output as a machine-readable object for easier analysis
  -e, --exact           Exactly match requested timing by disabling random jitter
  -j [JITTER_PROPORTION], --jitter-proportion [JITTER_PROPORTION]
                        Random jitter: a float between -1 and +1
```

Este mensaje no nos dice mucho sobre lo que el programa *hace*, pero sí nos dice las banderas importantes que podríamos querer usar al lanzarlo.

## Ejecutando el Trabajo en un Nodo de Computación

Crear un archivo de envío, solicitando una tarea en un único nodo, y luego lanzarla.

```bash
[yourUsername@login1 ~] nano serial-job.sh
[yourUsername@login1 ~] cat serial-job.sh
```

```bash
#!/bin/bash
#SBATCH -J solo-job
#SBATCH -p cpubase_bycore_b1
#SBATCH -N 1
#SBATCH -n 1

# Load the computing environment we need
module load Python

# Execute the task
amdahl
```

```bash
[yourUsername@login1 ~] sbatch serial-job.sh
```

Como antes, utilice los comandos de estado Slurm para comprobar si su trabajo se está ejecutando y cuándo finaliza:

```bash
[yourUsername@login1 ~] squeue -u yourUsername
```

Utilice `ls` para localizar el fichero de salida. La bandera `-t` ordena en orden cronológico inverso: el más reciente primero. ¿Cuál fue la salida?

::::::::::::::: spoiler

## Leer la salida del trabajo

La salida del cluster debe escribirse en un fichero en la carpeta desde la que lanzaste el trabajo. Por ejemplo,

```bash
[yourUsername@login1 ~] ls -t
```

```output
slurm-347087.out  serial-job.sh  amdahl  README.md  LICENSE.txt
```

```bash
[yourUsername@login1 ~] cat slurm-347087.out
```

```output
Doing 30.000 seconds of 'work' on 1 processor,
which should take 30.000 seconds with 0.850 parallel proportion of the workload.

  Hello, World! I am process 0 of 1 on smnode1. I will do all the serial 'work' for 4.500 seconds.
  Hello, World! I am process 0 of 1 on smnode1. I will do parallel 'work' for 25.500 seconds.

Total execution time (according to rank 0): 30.033 seconds
```

:::::::::::::::::::::::::

Como vimos antes, dos de las banderas del programa `amdahl` establecen la cantidad de trabajo y la proporción de ese trabajo que es de naturaleza paralela. Basándonos en la salida, podemos ver que el código utiliza por defecto 30 segundos de trabajo que es 85% paralelo. El programa se ejecutó durante algo más de 30 segundos en total, y si hacemos números, es cierto que el 15% del mismo se marcó como 'serie' y el 85% como 'paralelo'.

Como sólo le dimos una CPU al trabajo, este trabajo no fue realmente paralelo: el mismo procesador realizó el trabajo "en serie" durante 4,5 segundos, y luego la parte "paralela" durante 25,5 segundos, y no se ahorró tiempo. El clúster puede hacerlo mejor, si se lo pedimos.

## Ejecutando el Trabajo Paralelo

El programa `amdahl` utiliza la Interfaz de Paso de Mensajes (MPI) para el paralelismo - ésta es una herramienta común en los sistemas HPC.

::::::::::::::::::::::::::::::::::::::::: callout

## ¿Qué es MPI?

La interfaz de paso de mensajes es un conjunto de herramientas que permiten que varias tareas que se ejecutan simultáneamente se comuniquen entre sí. Normalmente, un único ejecutable se ejecuta varias veces, posiblemente en máquinas diferentes, y las herramientas MPI se utilizan para informar a cada instancia del ejecutable sobre sus procesos hermanos y de qué instancia se trata. MPI también proporciona herramientas que permiten la comunicación entre instancias para coordinar el trabajo, intercambiar información sobre elementos de la tarea o transferir datos. Una instancia MPI suele tener su propia copia de todas las variables locales.


::::::::::::::::::::::::::::::::::::::::::::::::::

Mientras que los ejecutables compatibles con MPI pueden ejecutarse generalmente como programas independientes, para que puedan ejecutarse en paralelo deben utilizar un *entorno de ejecución MPI*, que es una implementación específica del *estándar* MPI. Para activar el entorno MPI, el programa debe iniciarse mediante una orden como `mpiexec` (o `mpirun`, o `srun`, etc. dependiendo del entorno de ejecución MPI que necesite utilizar), que asegurará que se incluya el soporte de tiempo de ejecución apropiado para el paralelismo.

::::::::::::::::::::::::::::::::::::::::: callout

## Argumentos de ejecución MPI

Por sí solos, comandos como `mpiexec` pueden tomar muchos argumentos que especifican cuántas máquinas participarán en la ejecución, y puede que los necesite si desea ejecutar un programa MPI por su cuenta (por ejemplo, en su portátil). En el contexto de un sistema de colas, sin embargo, es frecuente que el tiempo de ejecución MPI obtenga los parámetros necesarios del sistema de colas, examinando las variables de entorno establecidas cuando se lanza el trabajo.

::::::::::::::::::::::::::::::::::::::::::::::::::

Modifiquemos el script de trabajo para solicitar más núcleos y utilizar el tiempo de ejecución MPI.

```bash
[yourUsername@login1 ~] cp serial-job.sh parallel-job.sh
[yourUsername@login1 ~] nano parallel-job.sh
[yourUsername@login1 ~] cat parallel-job.sh
```

```bash
#!/bin/bash
#SBATCH -J parallel-job
#SBATCH -p cpubase_bycore_b1
#SBATCH -N 1
#SBATCH -n 4

# Load the computing environment we need
# (mpi4py and numpy are in SciPy-bundle)
module load Python
module load SciPy-bundle

# Execute the task
mpiexec amdahl
```

A continuación, envíe su trabajo. Tenga en cuenta que el comando de envío no ha cambiado realmente con respecto a cómo enviamos el trabajo serie: todos los ajustes paralelos están en el archivo por lotes en lugar de en la línea de comandos.

```bash
[yourUsername@login1 ~] sbatch parallel-job.sh
```

Como antes, utiliza los comandos de estado para comprobar cuándo se ejecuta tu trabajo.

```bash
[yourUsername@login1 ~] ls -t
```

```output
slurm-347178.out  parallel-job.sh  slurm-347087.out  serial-job.sh  amdahl  README.md  LICENSE.txt
```

```bash
[yourUsername@login1 ~] cat slurm-347178.out
```

```output
Doing 30.000 seconds of 'work' on 4 processors,
which should take 10.875 seconds with 0.850 parallel proportion of the workload.

  Hello, World! I am process 0 of 4 on smnode1. I will do all the serial 'work' for 4.500 seconds.
  Hello, World! I am process 2 of 4 on smnode1. I will do parallel 'work' for 6.375 seconds.
  Hello, World! I am process 1 of 4 on smnode1. I will do parallel 'work' for 6.375 seconds.
  Hello, World! I am process 3 of 4 on smnode1. I will do parallel 'work' for 6.375 seconds.
  Hello, World! I am process 0 of 4 on smnode1. I will do parallel 'work' for 6.375 seconds.

Total execution time (according to rank 0): 10.888 seconds
```

::::::::::::::::::::::::::::::::::::::: challenge

## ¿Es 4× más rápido?

El trabajo paralelo recibió 4× más procesadores que el trabajo en serie: ¿significa eso que terminó en ¼ del tiempo?

::::::::::::::: solution

## Solución

El trabajo en paralelo tardó *menos* tiempo: ¡11 segundos es mejor que 30! Pero sólo es una mejora de 2,7×, no de 4×.

Mira la salida del trabajo:

- Mientras el "proceso 0" hacía el trabajo en serie, los procesos del 1 al 3 hacían su trabajo en paralelo.
- Mientras el proceso 0 se ponía al día en su trabajo paralelo, el resto no hacía nada.

El proceso 0 siempre tiene que terminar su tarea en serie antes de poder empezar con el trabajo en paralelo. Esto establece un límite inferior en la cantidad de tiempo que este trabajo va a tomar, no importa cuántos núcleos que lanzar en él.

Este es el principio básico de la [Ley de Amdahl][amdahl], que es una forma de predecir mejoras en el tiempo de ejecución para una carga de trabajo **fija** que puede subdividirse y ejecutarse en paralelo hasta cierto punto.



:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## ¿Cuánto mejora el rendimiento la ejecución en paralelo?

En teoría, dividir un cálculo perfectamente paralelo entre *n* procesos MPI debería producir una disminución del tiempo total de ejecución en un factor de *n*. Como acabamos de ver, los programas reales necesitan cierto tiempo para que los procesos MPI se comuniquen y coordinen, y algunos tipos de cálculos no pueden subdividirse: sólo se ejecutan eficazmente en una única CPU.

Además, si los procesos MPI operan en diferentes CPUs físicas en el ordenador, o a través de múltiples nodos de computación, se requiere incluso más tiempo para la comunicación del que se necesita cuando todos los procesos operan en una única CPU.

En la práctica, es habitual evaluar el paralelismo de un programa MPI mediante

- ejecutar el programa en un rango de CPUs,
- registrando el tiempo de ejecución en cada ejecución,
- comparando cada tiempo de ejecución con el tiempo cuando se utiliza una única CPU.

Dado que "más es mejor" (la mejora es más fácil de interpretar a partir de aumentos en alguna cantidad que de disminuciones), las comparaciones se realizan utilizando el factor de aceleración *S*, que se calcula como el tiempo de ejecución en una sola CPU dividido por el tiempo de ejecución en varias CPU. Para un programa perfectamente paralelo, un gráfico del aumento de velocidad *S* frente al número de CPUs *n* daría una línea recta, *S* = *n*.

Vamos a ejecutar un trabajo más, para que podamos ver lo cerca de una línea recta que llega nuestro código `amdahl`.

```bash
[yourUsername@login1 ~] nano parallel-job.sh
[yourUsername@login1 ~] cat parallel-job.sh
```

```bash
#!/bin/bash
#SBATCH -J parallel-job
#SBATCH -p cpubase_bycore_b1
#SBATCH -N 1
#SBATCH -n 8

# Load the computing environment we need
# (mpi4py and numpy are in SciPy-bundle)
module load Python
module load SciPy-bundle

# Execute the task
mpiexec amdahl
```

A continuación, envíe su trabajo. Tenga en cuenta que el comando de envío no ha cambiado realmente con respecto a cómo enviamos el trabajo serie: todos los ajustes paralelos están en el archivo por lotes en lugar de en la línea de comandos.

```bash
[yourUsername@login1 ~] sbatch parallel-job.sh
```

Como antes, utiliza los comandos de estado para comprobar cuándo se ejecuta tu trabajo.

```bash
[yourUsername@login1 ~] ls -t
```

```output
slurm-347271.out  parallel-job.sh  slurm-347178.out  slurm-347087.out  serial-job.sh  amdahl  README.md  LICENSE.txt
```

```bash
[yourUsername@login1 ~] cat slurm-347178.out
```

```output
which should take 7.688 seconds with 0.850 parallel proportion of the workload.

  Hello, World! I am process 4 of 8 on smnode1. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 0 of 8 on smnode1. I will do all the serial 'work' for 4.500 seconds.
  Hello, World! I am process 2 of 8 on smnode1. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 1 of 8 on smnode1. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 3 of 8 on smnode1. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 5 of 8 on smnode1. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 6 of 8 on smnode1. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 7 of 8 on smnode1. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 0 of 8 on smnode1. I will do parallel 'work' for 3.188 seconds.

Total execution time (according to rank 0): 7.697 seconds
```

:::::::::::::::::::::::::::::::::::::: discussion

## Salida no lineal

Cuando ejecutamos el trabajo con 4 trabajadores paralelos, el trabajo en serie escribió su salida primero, luego los procesos paralelos escribieron su salida, con el proceso 0 llegando primero y último.

Con 8 trabajadores, este no es el caso: como los trabajadores paralelos tardan menos que el trabajo en serie, es difícil decir qué proceso escribirá su salida primero, ¡excepto que *no* será el proceso 0!

::::::::::::::::::::::::::::::::::::::::::::::::::

Ahora, vamos a resumir la cantidad de tiempo que tardó cada trabajo en ejecutarse:

| Number of CPUs | Runtime (sec) |
| -------------- | ------------- |
| 1              | 30\.033       |
| 4              | 10\.888       |
| 8              | 7\.697        |

Entonces, utilice la primera fila para calcular los speedups $S$, utilizando Python como calculadora de línea de comandos y la fórmula

$$ S(t_{n}) = \frac{t_{1}}{t_{n}} $$

```bash
[yourUsername@login1 ~] for n in 30.033 10.888 7.697; do python3 -c "print(30.033 / $n)"; done
```

| Number of CPUs | Speedup | Ideal |
| -------------- | ------- | ----- |
| 1              | 1\.0    | 1     |
| 4              | 2\.75   | 4     |
| 8              | 3\.90   | 8     |

Los archivos de salida del trabajo nos han estado diciendo que este programa está realizando el 85% de su trabajo en paralelo, dejando el 15% para ejecutarse en serie. Esto parece razonablemente alto, pero nuestro rápido estudio del aumento de velocidad muestra que para obtener un aumento de velocidad de 4×, tenemos que utilizar 8 o 9 procesadores en paralelo. En los programas reales, el factor de aumento de velocidad está influido por

- Diseño de la CPU
- red de comunicación entre nodos de cálculo
- Implementaciones de bibliotecas MPI
- detalles del propio programa MPI

Usando la Ley de Amdahl, puedes probar que con este programa es *imposible* alcanzar 8× de velocidad, sin importar cuántos procesadores tengas a mano. Los detalles de ese análisis, con resultados que lo respalden, se dejan para la siguiente clase del taller HPC Carpentry, *HPC Workflows*.

En un entorno HPC, intentamos reducir el tiempo de ejecución de todo tipo de trabajos, y MPI es una forma extremadamente común de combinar docenas, cientos o miles de CPUs en la resolución de un único problema. Para aprender más sobre paralelización, vea la lección [parallel novice lesson][parallel-novice].


[amdahl]: https://en.wikipedia.org/wiki/Amdahl\'s_law
[parallel-novice]: https://www.hpc-carpentry.org/hpc-parallel-novice/


:::::::::::::::::::::::::::::::::::::::: keypoints

- La programación paralela permite a las aplicaciones aprovechar las ventajas del hardware paralelo.
- El sistema de colas facilita la ejecución de tareas paralelas.
- Las mejoras de rendimiento de la ejecución paralela no escalan linealmente.

::::::::::::::::::::::::::::::::::::::::::::::::::


