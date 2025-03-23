---
title: Acceso al software a través de módulos
teaching: 30
exercises: 15
---




::::::::::::::::::::::::::::::::::::::: objectives

- Carga y utiliza un paquete de software.
- Explique cómo cambia el entorno del intérprete de órdenes cuando el mecanismo de módulos carga o descarga paquetes.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Cómo se cargan y descargan los paquetes de software?

::::::::::::::::::::::::::::::::::::::::::::::::::

En un sistema informático de alto rendimiento, rara vez ocurre que el software que queremos utilizar esté disponible cuando nos conectamos. Está instalado, pero tendremos que "cargarlo" antes de que pueda ejecutarse.

Sin embargo, antes de empezar a utilizar paquetes de software individuales, deberíamos entender el razonamiento que hay detrás de este enfoque. Los tres factores más importantes son:

- incompatibilidades de software
- versionado
- dependencias

La incompatibilidad de software es un gran dolor de cabeza para los programadores. A veces la presencia (o ausencia) de un paquete de software rompe otros que dependen de él. Dos ejemplos bien conocidos son las versiones de Python y del compilador de C. Python 3 proporciona un comando `python` que entra en conflicto con el que proporciona Python 2. El software compilado con una versión más reciente de las librerías C y ejecutado en una máquina que tiene instaladas librerías C más antiguas producirá un desagradable error `'GLIBCXX_3.4.20' not found`.

El versionado de software es otro problema común. Un equipo puede depender de una determinada versión de un paquete para su proyecto de investigación - si la versión del software cambiara (por ejemplo, si se actualizara un paquete), podría afectar a sus resultados. Tener acceso a varias versiones de software permite a un conjunto de investigadores evitar que los problemas de versiones de software afecten a sus resultados.

Las dependencias se dan cuando un paquete de software concreto (o incluso una versión concreta) depende de tener acceso a otro paquete de software (o incluso a una versión concreta de otro paquete de software). Por ejemplo, el software de ciencia de materiales VASP puede depender de tener disponible una versión concreta de la biblioteca de software FFTW (Fastest Fourier Transform in the West) para funcionar.

## Módulos de entorno

Los módulos de entorno son la solución a estos problemas. Un *módulo* es una descripción autocontenida de un paquete de software -- contiene las configuraciones necesarias para ejecutar un paquete de software y, normalmente, codifica las dependencias necesarias de otros paquetes de software.

Hay un número de implementaciones de módulos de entorno diferentes comúnmente usadas en sistemas HPC: los dos más comunes son *TCL modules* y *Lmod*. Ambos usan una sintaxis similar y los conceptos son los mismos, por lo que aprender a usar uno te permitirá usar el que esté instalado en el sistema que estés usando. En ambas implementaciones se utiliza el comando `module` para interactuar con los módulos de entorno. Normalmente se añade un subcomando adicional al comando para especificar lo que se quiere hacer. Para obtener una lista de subcomandos puede utilizar `module -h` o `module help`. Como para todos los comandos, puede acceder a la ayuda completa en las páginas *man* con `man module`.

Al iniciar la sesión puede que empiece con un conjunto de módulos cargados por defecto o puede que empiece con un entorno vacío; esto depende de la configuración del sistema que esté utilizando.

### Listado de módulos disponibles

Para ver los módulos de software disponibles, utilice `module avail`:

```bash
[yourUsername@login1 ~] module avail
```

```output
~~~ /cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/modules/all ~~~
  Bazel/3.6.0-GCCcore-x.y.z              NSS/3.51-GCCcore-x.y.z
  Bison/3.5.3-GCCcore-x.y.z              Ninja/1.10.0-GCCcore-x.y.z
  Boost/1.72.0-gompi-2020a               OSU-Micro-Benchmarks/5.6.3-gompi-2020a
  CGAL/4.14.3-gompi-2020a-Python-3.x.y   OpenBLAS/0.3.9-GCC-x.y.z
  CMake/3.16.4-GCCcore-x.y.z             OpenFOAM/v2006-foss-2020a

[removed most of the output here for clarity]

  Where:
   L:        Module is loaded
   Aliases:  Aliases exist: foo/1.2.3 (1.2) means that "module load foo/1.2"
             will load foo/1.2.3
   D:        Default Module

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching
any of the "keys".
```

### Listado de módulos cargados actualmente

Puede utilizar el comando `module list` para ver qué módulos tiene cargados actualmente en su entorno. Si no tiene ningún módulo cargado, verá un mensaje indicándoselo

```bash
[yourUsername@login1 ~] module list
```

```output
No Modulefiles Currently Loaded.
```

## Carga y descarga de software

Para cargar un módulo de software, utilice `module load`. En este ejemplo utilizaremos Python 3.

Inicialmente, Python 3 no está cargado. Podemos comprobarlo usando el comando `which`. `which` busca programas de la misma forma que lo hace Bash, así que podemos usarlo para que nos diga dónde está almacenado un software en particular.

```bash
[yourUsername@login1 ~] which python3
```

Si el comando `python3` no estuviera disponible, veríamos una salida como

```output
/usr/bin/which: no python3 in (/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin:/opt/software/slurm/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home/yourUsername/.local/bin:/home/yourUsername/bin)
```

Observe que este muro de texto es en realidad una lista, con valores separados por el carácter `:`. La salida nos está diciendo que el comando `which` buscó `python3` en los siguientes directorios, sin éxito:

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin
/opt/software/slurm/bin
/usr/local/bin
/usr/bin
/usr/local/sbin
/usr/sbin
/opt/puppetlabs/bin
/home/yourUsername/.local/bin
/home/yourUsername/bin
```

Sin embargo, en nuestro caso tenemos una `python3` existente disponible, así que vemos

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin/python3
```

Sin embargo, necesitamos un Python diferente al proporcionado por el sistema, así que carguemos un módulo para acceder a él.

Podemos cargar el comando `python3` con `module load`:

```bash
[yourUsername@login1 ~] module load Python
[yourUsername@login1 ~] which python3
```

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/Python/3.x.y-GCCcore-x.y.z/bin/python3
```

¿Qué acaba de ocurrir?

Para entender la salida, primero necesitamos entender la naturaleza de la variable de entorno `$PATH`.`$PATH` es una variable de entorno especial que controla dónde busca software un sistema UNIX. Específicamente `$PATH` es una lista de directorios (separados por `:`) en los que el sistema operativo busca un comando antes de rendirse y decirnos que no puede encontrarlo. Como con todas las variables de entorno podemos imprimirla usando `echo`.

```bash
[yourUsername@login1 ~] echo $PATH
```

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/Python/3.x.y-GCCcore-x.y.z/bin:/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/SQLite/3.31.1-GCCcore-x.y.z/bin:/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/Tcl/8.6.10-GCCcore-x.y.z/bin:/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/GCCcore/x.y.z/bin:/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin:/opt/software/slurm/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home/user01/.local/bin:/home/user01/bin
```

Notará una similitud con la salida del comando `which`. En este caso, sólo hay una diferencia: el directorio diferente al principio. Cuando ejecutamos el comando `module load`, se añadió un directorio al principio de nuestro `$PATH`. Examinemos lo que hay allí:

```bash
[yourUsername@login1 ~] ls /cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/Python/3.x.y-GCCcore-x.y.z/bin
```

```output
2to3              nosetests-3.8  python                 rst2s5.py
2to3-3.8          pasteurize     python3                rst2xetex.py
chardetect        pbr            python3.8              rst2xml.py
cygdb             pip            python3.8-config       rstpep2html.py
cython            pip3           python3-config         runxlrd.py
cythonize         pip3.8         rst2html4.py           sphinx-apidoc
easy_install      pybabel        rst2html5.py           sphinx-autogen
easy_install-3.8  __pycache__    rst2html.py            sphinx-build
futurize          pydoc3         rst2latex.py           sphinx-quickstart
idle3             pydoc3.8       rst2man.py             tabulate
idle3.8           pygmentize     rst2odt_prepstyles.py  virtualenv
netaddr           pytest         rst2odt.py             wheel
nosetests         py.test        rst2pseudoxml.py
```

Llevando esto a su conclusión, `module load` añadirá software a su `$PATH`. Carga" software. Una nota especial sobre esto - dependiendo de la versión del programa `module` que esté instalado en su sitio, `module load` también cargará las dependencias de software necesarias.

Para demostrarlo, utilicemos `module list`.`module list` muestra todos los módulos de software cargados.

```bash
[yourUsername@login1 ~] module list
```

```output
Currently Loaded Modules:
  1) GCCcore/x.y.z                 4) GMP/6.2.0-GCCcore-x.y.z
  2) Tcl/8.6.10-GCCcore-x.y.z      5) libffi/3.3-GCCcore-x.y.z
  3) SQLite/3.31.1-GCCcore-x.y.z   6) Python/3.x.y-GCCcore-x.y.z
```

```bash
[yourUsername@login1 ~] module load GROMACS
[yourUsername@login1 ~] module list
```

```output
Currently Loaded Modules:
  1) GCCcore/x.y.z                    14) libfabric/1.11.0-GCCcore-x.y.z
  2) Tcl/8.6.10-GCCcore-x.y.z         15) PMIx/3.1.5-GCCcore-x.y.z
  3) SQLite/3.31.1-GCCcore-x.y.z      16) OpenMPI/4.0.3-GCC-x.y.z
  4) GMP/6.2.0-GCCcore-x.y.z          17) OpenBLAS/0.3.9-GCC-x.y.z
  5) libffi/3.3-GCCcore-x.y.z         18) gompi/2020a
  6) Python/3.x.y-GCCcore-x.y.z       19) FFTW/3.3.8-gompi-2020a
  7) GCC/x.y.z                        20) ScaLAPACK/2.1.0-gompi-2020a
  8) numactl/2.0.13-GCCcore-x.y.z     21) foss/2020a
  9) libxml2/2.9.10-GCCcore-x.y.z     22) pybind11/2.4.3-GCCcore-x.y.z-Pytho...
 10) libpciaccess/0.16-GCCcore-x.y.z  23) SciPy-bundle/2020.03-foss-2020a-Py...
 11) hwloc/2.2.0-GCCcore-x.y.z        24) networkx/2.4-foss-2020a-Python-3.8...
 12) libevent/2.1.11-GCCcore-x.y.z    25) GROMACS/2020.1-foss-2020a-Python-3...
 13) UCX/1.8.0-GCCcore-x.y.z
```

En este caso, al cargar el módulo `GROMACS` (un paquete de software bioinformático), también se han cargado `GMP/6.2.0-GCCcore-x.y.z` y `SciPy-bundle/2020.03-foss-2020a-Python-3.x.y`. Intentemos descargar el paquete `GROMACS`.

```bash
[yourUsername@login1 ~] module unload GROMACS
[yourUsername@login1 ~] module list
```

```output
Currently Loaded Modules:
  1) GCCcore/x.y.z                    13) UCX/1.8.0-GCCcore-x.y.z
  2) Tcl/8.6.10-GCCcore-x.y.z         14) libfabric/1.11.0-GCCcore-x.y.z
  3) SQLite/3.31.1-GCCcore-x.y.z      15) PMIx/3.1.5-GCCcore-x.y.z
  4) GMP/6.2.0-GCCcore-x.y.z          16) OpenMPI/4.0.3-GCC-x.y.z
  5) libffi/3.3-GCCcore-x.y.z         17) OpenBLAS/0.3.9-GCC-x.y.z
  6) Python/3.x.y-GCCcore-x.y.z       18) gompi/2020a
  7) GCC/x.y.z                        19) FFTW/3.3.8-gompi-2020a
  8) numactl/2.0.13-GCCcore-x.y.z     20) ScaLAPACK/2.1.0-gompi-2020a
  9) libxml2/2.9.10-GCCcore-x.y.z     21) foss/2020a
 10) libpciaccess/0.16-GCCcore-x.y.z  22) pybind11/2.4.3-GCCcore-x.y.z-Pytho...
 11) hwloc/2.2.0-GCCcore-x.y.z        23) SciPy-bundle/2020.03-foss-2020a-Py...
 12) libevent/2.1.11-GCCcore-x.y.z    24) networkx/2.4-foss-2020a-Python-3.x.y
```

Así que usar `module unload` "descarga" un módulo, y dependiendo de cómo esté configurado un sitio puede que también descargue todas las dependencias (en nuestro caso no lo hace). Si quisiéramos descargar todo a la vez, podríamos ejecutar `module purge` (descarga todo).

```bash
[yourUsername@login1 ~] module purge
[yourUsername@login1 ~] module list
```

```output
No modules loaded
```

Tenga en cuenta que `module purge` es informativa. También nos permitirá saber si un conjunto predeterminado de paquetes "pegajosos" no puede ser descargado (y cómo descargarlos si realmente lo deseamos).

Tenga en cuenta que este proceso de carga de módulos se produce principalmente a través de la manipulación de variables de entorno como `$PATH`. Normalmente hay poca o ninguna transferencia de datos implicada.

El proceso de carga de módulos manipula también otras variables de entorno especiales, incluyendo variables que influyen en dónde busca el sistema las bibliotecas de software, y a veces variables que indican a los paquetes de software comercial dónde encontrar los servidores de licencias.

El comando module también restaura estas variables de entorno del shell a su estado anterior cuando se descarga un módulo.

## Versiones de software

Hasta ahora, hemos aprendido cómo cargar y descargar paquetes de software. Esto es muy útil. Sin embargo, aún no hemos abordado la cuestión del versionado de software. En un momento u otro, te encontrarás con problemas en los que sólo una versión concreta de algún software será adecuada. Puede que un error clave sólo se haya corregido en una versión determinada, o que la versión X haya roto la compatibilidad con un formato de archivo que utilizas. En cualquiera de estos casos, ayuda ser muy específico sobre el software que se carga.

Examinemos la salida de `module avail` más de cerca.

```bash
[yourUsername@login1 ~] module avail
```

```output
~~~ /cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/modules/all ~~~
  Bazel/3.6.0-GCCcore-x.y.z              NSS/3.51-GCCcore-x.y.z
  Bison/3.5.3-GCCcore-x.y.z              Ninja/1.10.0-GCCcore-x.y.z
  Boost/1.72.0-gompi-2020a               OSU-Micro-Benchmarks/5.6.3-gompi-2020a
  CGAL/4.14.3-gompi-2020a-Python-3.x.y   OpenBLAS/0.3.9-GCC-x.y.z
  CMake/3.16.4-GCCcore-x.y.z             OpenFOAM/v2006-foss-2020a

[removed most of the output here for clarity]

  Where:
   L:        Module is loaded
   Aliases:  Aliases exist: foo/1.2.3 (1.2) means that "module load foo/1.2"
             will load foo/1.2.3
   D:        Default Module

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching
any of the "keys".
```

::::::::::::::::::::::::::::::::::::::: challenge

## Uso de módulos de software en scripts

Crea un trabajo capaz de ejecutar `python3 --version`. Recuerde, ¡no se carga ningún software por defecto! Ejecutar un trabajo es como iniciar sesión en el sistema (no debe asumir que un módulo cargado en el nodo de inicio de sesión está cargado en un nodo de computación).

::::::::::::::: solution

## Solución

```bash
[yourUsername@login1 ~] nano python-module.sh
[yourUsername@login1 ~] cat python-module.sh
```

```output
#!/bin/bash
#SBATCH 
r config$sched$comment` -t 00:00:30

module load Python

python3 --version
```

```bash
[yourUsername@login1 ~] sbatch  python-module.sh
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: keypoints

- Cargar software con `module load softwareName`.
- Descargar software con `module unload`
- El sistema de módulos gestiona automáticamente las versiones de software y los conflictos entre paquetes.

::::::::::::::::::::::::::::::::::::::::::::::::::


