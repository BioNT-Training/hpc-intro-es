---
title: Uso eficaz de los recursos
teaching: 10
exercises: 20
---




::::::::::::::::::::::::::::::::::::::: objectives

- Consulta las estadísticas de los trabajos.
- Realiza peticiones de recursos más precisas en los scripts de trabajo basándose en datos que describen el rendimiento pasado.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Cómo puedo revisar trabajos anteriores?
- ¿Cómo puedo utilizar estos conocimientos para crear un script de envío más preciso?

::::::::::::::::::::::::::::::::::::::::::::::::::

Hemos tocado todas las habilidades necesarias para interactuar con un clúster HPC: iniciar sesión a través de SSH, cargar módulos de software, enviar trabajos paralelos y encontrar la salida. Vamos a aprender a estimar el uso de recursos y por qué puede ser importante.

## Estimación de Recursos Requeridos Usando el Planificador

Aunque antes hemos cubierto la solicitud de recursos al planificador con el código π, ¿cómo sabemos qué tipo de recursos necesitará el software en primer lugar, y su demanda de cada uno? En general, a menos que la documentación del software o los testimonios de los usuarios nos den alguna idea, no sabremos cuánta memoria o tiempo de computación necesitará un programa.

::::::::::::::::::::::::::::::::::::::::: callout

## Leer la documentación

La mayoría de las instalaciones HPC mantienen documentación en forma de wiki, sitio web o documento que se envía al registrarse para obtener una cuenta. Echa un vistazo a estos recursos y busca el software que piensas utilizar: puede que alguien haya escrito una guía para sacarle el máximo partido.

::::::::::::::::::::::::::::::::::::::::::::::::::

Una forma conveniente de averiguar los recursos necesarios para que un trabajo se ejecute con éxito es enviar un trabajo de prueba, y luego preguntar al planificador sobre su impacto utilizando `sacct -u yourUsername`. Puede utilizar este conocimiento para configurar el siguiente trabajo con una estimación más cercana de su carga en el sistema. Una buena regla general es pedir al programador entre un 20% y un 30% más de tiempo y memoria de lo que espera que necesite el trabajo. Esto garantiza que las pequeñas fluctuaciones en el tiempo de ejecución o en el uso de memoria no provoquen la cancelación del trabajo por parte del programador. Ten en cuenta que si pides demasiado, puede que tu trabajo no se ejecute aunque haya suficientes recursos disponibles, porque el programador estará esperando a que los trabajos de otras personas terminen y liberen los recursos necesarios para igualar lo que has pedido.

## Estadísticas

Como ya hemos enviado `amdahl` para que se ejecute en el cluster, podemos consultar el planificador para ver cuánto tiempo ha tardado nuestro trabajo y qué recursos se han utilizado. Usaremos `sacct -u yourUsername` para obtener estadísticas sobre `parallel-job.sh`.

```bash
[yourUsername@login1 ~] sacct -u yourUsername
```

```output
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
7               file.sh cpubase_b+ def-spons+          1  COMPLETED      0:0
7.batch           batch            def-spons+          1  COMPLETED      0:0
7.extern         extern            def-spons+          1  COMPLETED      0:0
8               file.sh cpubase_b+ def-spons+          1  COMPLETED      0:0
8.batch           batch            def-spons+          1  COMPLETED      0:0
8.extern         extern            def-spons+          1  COMPLETED      0:0
9            example-j+ cpubase_b+ def-spons+          1  COMPLETED      0:0
9.batch           batch            def-spons+          1  COMPLETED      0:0
9.extern         extern            def-spons+          1  COMPLETED      0:0
```

Muestra todos los trabajos ejecutados hoy (tenga en cuenta que hay varias entradas por trabajo). Para obtener información sobre un trabajo específico (por ejemplo, 347087), cambiamos ligeramente el comando.

```bash
[yourUsername@login1 ~] sacct -u yourUsername -l -j 347087
```

Mostrará mucha información; de hecho, cada pieza de información recopilada sobre su trabajo por el planificador aparecerá aquí. Puede ser útil redirigir esta información a `less` para facilitar su visualización (utilice las teclas de flecha izquierda y derecha para desplazarse por los campos).

```bash
[yourUsername@login1 ~] sacct -u yourUsername -l -j 347087 | less -S
```

:::::::::::::::::::::::::::::::::::::: discussion

## Discusión

Esta vista puede ayudar a comparar la cantidad de tiempo solicitado y realmente utilizado, la duración de la residencia en la cola antes del lanzamiento, y la huella de memoria en el nodo(s) de computación.

¿Hasta qué punto fueron precisas nuestras estimaciones?


::::::::::::::::::::::::::::::::::::::::::::::::::

## Mejora de las solicitudes de recursos

A partir del historial de trabajos, vemos que los trabajos `amdahl` terminaron de ejecutarse en unos pocos minutos como máximo, una vez despachados. La estimación de tiempo que proporcionamos en el script del trabajo era demasiado larga Esto hace que sea más difícil para el sistema de colas estimar con precisión cuándo los recursos quedarán libres para otros trabajos. En la práctica, esto significa que el sistema de colas espera para despachar nuestro trabajo `amdahl` hasta que se abre todo el espacio de tiempo solicitado, en lugar de "colarlo" en una ventana mucho más corta en la que el trabajo podría realmente terminar. Especificar el tiempo de ejecución esperado en el script de envío con mayor precisión ayudará a aliviar la congestión del cluster y puede conseguir que su trabajo se despache antes.

::::::::::::::::::::::::::::::::::::::: challenge

## Reducir la estimación de tiempo

Edita `parallel_job.sh` para establecer una mejor estimación del tiempo. ¿Qué tan cerca puedes estar?

Sugerencia: utilice `-t`.

::::::::::::::: solution

## Solución

La siguiente línea indica a Slurm que nuestro trabajo debe terminar en 2 minutos:

```bash
#SBATCH -t 00:02:00
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: keypoints

- La precisión de los guiones de trabajo ayuda al sistema de colas a asignar eficazmente los recursos compartidos.

::::::::::::::::::::::::::::::::::::::::::::::::::


