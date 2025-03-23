---
title: Fundamentos del programador
teaching: 45
exercises: 30
---




::::::::::::::::::::::::::::::::::::::: objectives

- Envía un script simple al cluster.
- Supervisar la ejecución de los trabajos mediante herramientas de línea de comandos.
- Inspeccione los archivos de salida y error de sus trabajos.
- Encontrar el lugar adecuado para colocar grandes conjuntos de datos en el clúster.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Qué es un planificador y por qué un clúster necesita uno?
- ¿Cómo lanzo un programa para que se ejecute en un nodo de cálculo del clúster?
- ¿Cómo puedo capturar la salida de un programa que se ejecuta en un nodo del clúster?

::::::::::::::::::::::::::::::::::::::::::::::::::

## Programador de trabajos

Un sistema HPC puede tener miles de nodos y miles de usuarios. ¿Cómo decidimos quién recibe qué y cuándo? ¿Cómo nos aseguramos de que una tarea se ejecuta con los recursos que necesita? De esta tarea se encarga un software especial llamado *programador*. En un sistema HPC, el programador gestiona qué tareas se ejecutan, dónde y cuándo.

La siguiente ilustración compara las tareas de un programador de tareas con las de un camarero en un restaurante. Si puede relacionarlo con un caso en el que tuvo que esperar un rato en una cola para entrar en un restaurante popular, entonces ahora puede entender por qué a veces su trabajo no se inicia instantáneamente como en su ordenador portátil.

![](fig/restaurant_queue_manager.svg){alt="Compara un programador de tareas con un camarero en un restaurante" max-width="75%"}

El planificador utilizado en esta lección es Slurm. Aunque Slurm no se utiliza en todas partes, la ejecución de trabajos es bastante similar independientemente del software que se utilice. La sintaxis exacta puede cambiar, pero los conceptos siguen siendo los mismos.

## Ejecución de un trabajo por lotes

El uso más básico del planificador es ejecutar un comando de forma no interactiva. Cualquier comando (o serie de comandos) que desee ejecutar en el cluster se denomina *job*, y el proceso de utilizar un planificador para ejecutar el trabajo se denomina *sometimiento de trabajo por lotes*.

En este caso, el trabajo que queremos ejecutar es un script de shell -- esencialmente un archivo de texto que contiene una lista de comandos UNIX para ser ejecutados de manera secuencial. Nuestro script de shell tendrá tres partes:

- En la primera línea, añada `#!/bin/bash`. El `#!` (pronunciado "hash-bang" o "shebang") indica al ordenador qué programa debe procesar el contenido de este fichero. En este caso, le estamos diciendo que los comandos que siguen están escritos para la shell de línea de comandos (en la que hemos estado haciendo todo hasta ahora).
- En cualquier lugar debajo de la primera línea, añadiremos un comando `echo` con un saludo amistoso. Cuando se ejecute, el script de shell imprimirá lo que venga después de `echo` en el terminal.
  - `echo -n` imprimirá todo lo que sigue, *sin* terminar la línea imprimiendo el carácter de nueva línea.
- En la última línea, invocaremos el comando `hostname`, que imprimirá el nombre de la máquina en la que se ejecuta el script.

```bash
[yourUsername@login1 ~] nano example-job.sh
```

```bash
#!/bin/bash

echo -n "This script is running on "
hostname
```

::::::::::::::::::::::::::::::::::::::: challenge

## Creación de nuestro trabajo de prueba

Ejecuta el script. ¿Se ejecuta en el clúster o sólo en nuestro nodo de inicio de sesión?

::::::::::::::: solution

## Solución

```bash
[yourUsername@login1 ~] bash example-job.sh
```

```output
This script is running on login1
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

Este script se ejecutó en el nodo de inicio de sesión, pero queremos aprovechar los nodos de computación: necesitamos que el planificador ponga en cola `example-job.sh` para ejecutarse en un nodo de computación.

Para enviar esta tarea al planificador, usamos el comando `sbatch`. Esto crea un *job* que ejecutará el *script* cuando sea *despachado* a un nodo de computación que el sistema de colas haya identificado como disponible para realizar el trabajo.

```bash
[yourUsername@login1 ~] sbatch  example-job.sh
```


```output
Submitted batch job 7
```

Y eso es todo lo que tenemos que hacer para enviar un trabajo. Nuestro trabajo está hecho -- ahora el programador toma el relevo e intenta ejecutar el trabajo por nosotros. Mientras el trabajo espera a ejecutarse, entra en una lista de trabajos llamada *cola*. Para comprobar el estado de nuestro trabajo, comprobamos la cola utilizando el comando `squeue -u yourUsername`.

```bash
[yourUsername@login1 ~] squeue -u yourUsername
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
    9 cpubase_b example-   user01  R       0:05      1 node1
```

Podemos ver todos los detalles de nuestro trabajo, lo más importante es que está en el estado `R` o `RUNNING`. A veces nuestros trabajos pueden necesitar esperar en una cola (`PENDING`) o tener un error (`E`).

:::::::::::::::::::::::::::::::::::::: discussion

## ¿Dónde está la salida?

En el nodo de inicio de sesión, este script imprimió la salida en el terminal -- pero ahora, cuando `squeue` muestra que el trabajo ha finalizado, no se imprimió nada en el terminal.

La salida del trabajo de cluster se redirige normalmente a un archivo en el directorio desde el que se lanzó. Utilice `ls` para buscar y `cat` para leer el archivo.

::::::::::::::::::::::::::::::::::::::::::::::::::

## Personalización de un trabajo

El trabajo que acabamos de ejecutar utilizaba todas las opciones por defecto del planificador. En un escenario del mundo real, eso no es probablemente lo que queremos. Las opciones por defecto representan un mínimo razonable. Lo más probable es que necesitemos más núcleos, más memoria, más tiempo, entre otras consideraciones especiales. Para tener acceso a estos recursos debemos personalizar nuestro script de trabajo.

Los comentarios en los scripts de shell UNIX (denotados por `#`) son normalmente ignorados, pero hay excepciones. Por ejemplo, el comentario especial `#!` al principio de los scripts especifica qué programa debe usarse para ejecutarlo (normalmente verá `#!/usr/bin/env bash`). Los programadores como Slurm también tienen un comentario especial que se utiliza para indicar opciones específicas del programador. Aunque estos comentarios difieren de un programador a otro, el comentario especial de Slurm es `#SBATCH`. Todo lo que sigue al comentario `#SBATCH` se interpreta como una instrucción para el programador.

Vamos a ilustrarlo con un ejemplo. Por defecto, el nombre de un trabajo es el nombre del script, pero se puede utilizar la opción `-J` para cambiar el nombre de un trabajo. Añade una opción al script:

```bash
[yourUsername@login1 ~] cat example-job.sh
```

```bash
#!/bin/bash
#SBATCH -J hello-world

echo -n "This script is running on "
hostname
```

Envía el trabajo y supervisa su estado:

```bash
[yourUsername@login1 ~] sbatch  example-job.sh
[yourUsername@login1 ~] squeue -u yourUsername
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   10 cpubase_b hello-wo   user01  R       0:02      1 node1
```

¡Fantástico, hemos cambiado con éxito el nombre de nuestro trabajo!

### Solicitudes de recursos

¿Qué pasa con los cambios más importantes, como el número de núcleos y memoria para nuestros trabajos? Una cosa que es absolutamente crítica cuando se trabaja en un sistema HPC es especificar los recursos necesarios para ejecutar un trabajo. Esto permite al programador encontrar el momento y el lugar adecuados para programar nuestro trabajo. Si no especifica los requisitos (como la cantidad de tiempo que necesita), es probable que se quede con los recursos predeterminados de su sitio, que probablemente no es lo que desea.

A continuación se muestran varias solicitudes de recursos clave:

- `--ntasks=<ntasks>` o `-n <ntasks>`: ¿Cuántos núcleos de CPU necesita su trabajo, en total?

- `--time <days-hours:minutes:seconds>` o `-t <days-hours:minutes:seconds>`: ¿Cuánto tiempo real (walltime) tardará en ejecutarse tu tarea? La parte `<days>` puede omitirse.

- `--mem=<megabytes>`: ¿Cuánta memoria en un nodo necesita su trabajo en megabytes? También puede especificar gigabytes añadiendo una pequeña "g" después (ejemplo: `--mem=5g`)

- `--nodes=<nnodes>` o `-N <nnodes>`: ¿En cuántas máquinas distintas debe ejecutarse su trabajo? Tenga en cuenta que si establece `ntasks` en un número superior al que puede ofrecer una máquina, Slurm establecerá este valor automáticamente.

Tenga en cuenta que el simple hecho de *solicitar* estos recursos no hace que su trabajo se ejecute más rápido, ni significa necesariamente que vaya a consumir todos estos recursos. Sólo significa que se ponen a su disposición. Tu trabajo puede terminar usando menos memoria, o menos tiempo, o menos nodos de los que has solicitado, y aún así se ejecutará.

Lo mejor es que tus solicitudes reflejen fielmente los requisitos de tu trabajo. Hablaremos más acerca de cómo asegurarse de que está utilizando los recursos de manera efectiva en un episodio posterior de esta lección.

::::::::::::::::::::::::::::::::::::::: challenge

## Envío de solicitudes de recursos

Modifique nuestro script `hostname` para que se ejecute durante un minuto y, a continuación, envíe un trabajo para él en el clúster.

::::::::::::::: solution

## Solución

```bash
[yourUsername@login1 ~] cat example-job.sh
```

```bash
#!/bin/bash
#SBATCH -t 00:01 # timeout in HH:MM

echo -n "This script is running on "
sleep 20 # time in seconds
hostname
```

```bash
[yourUsername@login1 ~] sbatch  example-job.sh
```

¿Por qué el tiempo de ejecución Slurm y el tiempo `sleep` no son idénticos?



:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

Las solicitudes de recursos suelen ser vinculantes. Si las excedes, tu trabajo será eliminado. Usemos el tiempo de muro como ejemplo. Solicitaremos 1 minuto de tiempo de muro, e intentaremos ejecutar un trabajo durante dos minutos.

```bash
[yourUsername@login1 ~] cat example-job.sh
```

```bash
#!/bin/bash
#SBATCH -J long_job
#SBATCH -t 00:01 # timeout in HH:MM

echo "This script is running on ... "
sleep 240 # time in seconds
hostname
```

Envía el trabajo y espera a que termine. Una vez que haya terminado, compruebe el archivo de registro.

```bash
[yourUsername@login1 ~] sbatch  example-job.sh
[yourUsername@login1 ~] squeue -u yourUsername
```

```bash
[yourUsername@login1 ~] cat slurm-12.out
```

```output
This script is running on ...
slurmstepd: error: *** JOB 12 ON node1 CANCELLED AT 2021-02-19T13:55:57
DUE TO TIME LIMIT ***
```

Nuestro trabajo ha sido cancelado por exceder la cantidad de recursos solicitados. Aunque esto parece duro, en realidad es una característica. El cumplimiento estricto de las solicitudes de recursos permite al planificador encontrar el mejor lugar posible para sus trabajos. Aún más importante, asegura que otro usuario no pueda usar más recursos de los que se le han dado. Si otro usuario mete la pata y accidentalmente intenta utilizar todos los núcleos o la memoria de un nodo, Slurm restringirá su trabajo a los recursos solicitados o matará el trabajo directamente. Otros trabajos en el nodo no se verán afectados. Esto significa que un usuario no puede estropear la experiencia de los demás, los únicos trabajos afectados por un error en la programación serán los suyos propios.

## Cancelación de un trabajo

A veces cometeremos un error y necesitaremos cancelar un trabajo. Esto se puede hacer con el comando `scancel`. Vamos a enviar un trabajo y luego cancelarlo usando su número de trabajo (¡recuerda cambiar el tiempo de ejecución para que se ejecute el tiempo suficiente para que puedas cancelarlo antes de que se mate!)

```bash
[yourUsername@login1 ~] sbatch  example-job.sh
[yourUsername@login1 ~] squeue -u yourUsername
```

```output
Submitted batch job 13

JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   13 cpubase_b long_job   user01  R       0:02      1 node1
```

Ahora cancele el trabajo con su número de trabajo (impreso en su terminal). Un retorno limpio de su símbolo del sistema indica que la solicitud de cancelación del trabajo se ha realizado correctamente.

```bash
[yourUsername@login1 ~] scancel 38759
# It might take a minute for the job to disappear from the queue...
[yourUsername@login1 ~] squeue -u yourUsername
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```

::::::::::::::::::::::::::::::::::::::: challenge

## Cancelación de múltiples trabajos

También podemos cancelar todos nuestros trabajos a la vez utilizando la opción `-u`. Esto borrará todos los trabajos de un usuario específico (en este caso, usted mismo). Tenga en cuenta que sólo puede eliminar sus propios trabajos.

Pruebe a enviar varios trabajos y luego cancélelos todos.

::::::::::::::: solution

## Solución

En primer lugar, envíe un trío de trabajos:

```bash
[yourUsername@login1 ~] sbatch  example-job.sh
[yourUsername@login1 ~] sbatch  example-job.sh
[yourUsername@login1 ~] sbatch  example-job.sh
```

A continuación, cancélelos todos:

```bash
[yourUsername@login1 ~] scancel -u yourUsername
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Otros tipos de trabajos

Hasta ahora, nos hemos centrado en la ejecución de tareas por lotes. `Slurm` también ofrece la posibilidad de iniciar una sesión interactiva.

Con mucha frecuencia hay tareas que deben realizarse de forma interactiva. Crear un script de trabajo completo puede ser excesivo, pero la cantidad de recursos requeridos es demasiado para que un nodo de inicio de sesión pueda manejarlo. Un buen ejemplo de esto podría ser la construcción de un índice del genoma para la alineación con una herramienta como [HISAT2][hisat]. Afortunadamente, podemos ejecutar este tipo de tareas de una sola vez con `srun`.

`srun` ejecuta un único comando en el cluster y luego se cierra. Demostremos esto ejecutando el comando `hostname` con `srun`. (Podemos cancelar un trabajo `srun` con `Ctrl-c`)

```bash
[yourUsername@login1 ~] srun hostname
```

```output
smnode1
```

`srun` acepta las mismas opciones que `sbatch`. Sin embargo, en lugar de especificarlas en un script, estas opciones se especifican en la línea de comandos al iniciar un trabajo. Para enviar un trabajo que utilice 2 CPUs, por ejemplo, podríamos utilizar el siguiente comando:

```bash
[yourUsername@login1 ~] srun -n 2 echo "This job will use 2 CPUs."
```

```output
This job will use 2 CPUs.
This job will use 2 CPUs.
```

Normalmente, el entorno de shell resultante será el mismo que el de `sbatch`.

### Trabajos interactivos

A veces, necesitaremos muchos recursos para un uso interactivo. Quizás es la primera vez que ejecutamos un análisis o estamos intentando depurar algo que salió mal en un trabajo anterior. Afortunadamente, Slurm facilita el inicio de un trabajo interactivo con `srun`:

```bash
[yourUsername@login1 ~] srun --pty bash
```

Aparecerá un prompt bash. Tenga en cuenta que el prompt probablemente cambiará para reflejar su nueva ubicación, en este caso el nodo de computación en el que estamos conectados. También puedes verificarlo con `hostname`.

::::::::::::::::::::::::::::::::::::::::: callout

## Creación de gráficos remotos

Para ver la salida gráfica dentro de tus trabajos, necesitas usar X11 forwarding. Para conectarse con esta característica activada, utilice la opción `-Y` cuando se conecte con el comando `ssh`, por ejemplo, `ssh -Y yourUsername@cluster.hpc-carpentry.org`.

Para demostrar lo que ocurre cuando creas una ventana gráfica en el nodo remoto, utiliza el comando `xeyes`. Debería aparecer un par de ojos relativamente adorables (pulse `Ctrl-C` para parar). Si utiliza un Mac, debe haber instalado XQuartz (y reiniciado su ordenador) para que esto funcione.

Si su cluster tiene instalado el plugin [slurm-spank-x11](https://github.com/hautreux/slurm-spank-x11), puede asegurar el reenvío X11 dentro de los trabajos interactivos utilizando la opción `--x11` para `srun` con el comando `srun --x11 --pty bash`.

::::::::::::::::::::::::::::::::::::::::::::::::::

Cuando haya terminado con el trabajo interactivo, escriba `exit` para salir de la sesión.

[hisat]: https://daehwankimlab.github.io/hisat2/

:::::::::::::::::::::::::::::::::::::::: keypoints

- El planificador gestiona cómo se comparten los recursos informáticos entre los usuarios.
- Un trabajo no es más que un script de shell.
- Solicita *ligeramente* más recursos de los que necesitará.

::::::::::::::::::::::::::::::::::::::::::::::::::



