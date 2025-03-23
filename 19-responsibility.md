---
title: Uso responsable de los recursos compartidos
teaching: 15
exercises: 5
---




::::::::::::::::::::::::::::::::::::::: objectives

- Describe cómo las acciones de un único usuario pueden afectar a la experiencia de otros en un sistema compartido.
- Discutir el comportamiento de un ciudadano considerado del sistema compartido.
- Explica la importancia de hacer copias de seguridad de los datos críticos.
- Describa los retos que plantea la transferencia de grandes cantidades de datos fuera de los sistemas HPC.
- Convierte muchos ficheros en un único archivo comprimido utilizando tar.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Cómo puedo ser un usuario responsable?
- ¿Cómo puedo proteger mis datos?
- ¿Cuál es la mejor forma de obtener grandes cantidades de datos de un sistema HPC?

::::::::::::::::::::::::::::::::::::::::::::::::::

Una de las principales diferencias entre utilizar recursos HPC remotos y tu propio sistema (por ejemplo, tu portátil) es que los recursos remotos se comparten. El número de usuarios que comparten el recurso en un momento dado varía de un sistema a otro, pero es poco probable que usted sea el único usuario conectado o utilizando un sistema de este tipo.

El uso generalizado de sistemas de programación en los que los usuarios envían trabajos a recursos HPC es un resultado natural de la naturaleza compartida de estos recursos. Hay otras cosas que usted, como miembro honrado de la comunidad, debe tener en cuenta.

## Sea amable con los nodos de inicio de sesión

El nodo de inicio de sesión suele estar ocupado gestionando todos los usuarios conectados, creando y editando archivos y compilando software. Si la máquina se queda sin memoria o capacidad de procesamiento, se volverá muy lenta e inutilizable para todos. Aunque la máquina está pensada para ser utilizada, asegúrate de hacerlo de forma responsable, de manera que no afecte negativamente a la experiencia de los demás usuarios.

Los nodos de inicio de sesión son siempre el lugar adecuado para lanzar trabajos. Las políticas de clúster varían, pero también pueden utilizarse para probar flujos de trabajo y, en algunos casos, pueden albergar herramientas avanzadas de depuración o desarrollo específicas del clúster. El cluster puede tener módulos que necesitan ser cargados, posiblemente en un cierto orden, y rutas o versiones de librerías que difieren de tu portátil, y hacer una prueba interactiva en el nodo principal es una forma rápida y fiable de descubrir y arreglar estos problemas.

::::::::::::::::::::::::::::::::::::::::: callout

## Los nodos de inicio de sesión son un recurso compartido

Recuerda que el nodo de inicio de sesión se comparte con todos los demás usuarios y tus acciones podrían causar problemas a otras personas. Piensa detenidamente en las posibles implicaciones de emitir comandos que puedan utilizar grandes cantidades de recursos.

¿No está seguro? Pregunte a su amable administrador de sistemas ("sysadmin") si lo que está contemplando es adecuado para el nodo de inicio de sesión, o si existe otro mecanismo para hacerlo de forma segura.


::::::::::::::::::::::::::::::::::::::::::::::::::

Siempre puede utilizar los comandos `top` y `ps ux` para listar los procesos que se están ejecutando en el nodo de inicio de sesión junto con la cantidad de CPU y memoria que están utilizando. Si esta comprobación revela que el nodo de inicio de sesión está algo inactivo, puede utilizarlo con seguridad para su tarea de procesamiento no rutinaria. Si algo va mal -- el proceso tarda demasiado, o no responde -- puede usar el comando `kill` junto con el *PID* para terminar el proceso.

::::::::::::::::::::::::::::::::::::::: challenge

## Etiqueta del nodo de inicio de sesión

¿Cuál de estos comandos sería una tarea rutinaria a ejecutar en el nodo login?

1. `python physics_sim.py`
2. `make`
3. `create_directories.sh`
4. `molecular_dynamics_2`
5. `tar -xzf R-3.3.0.tar.gz`

::::::::::::::: solution

## Solución

Construir software, crear directorios y desempaquetar software son tareas > comunes y aceptables para el nodo de inicio de sesión: las opciones #2 (`make`), #3 (`mkdir`) y #5 (`tar`) probablemente estén bien. Tenga en cuenta que los nombres de los scripts no siempre reflejan su contenido: antes de lanzar #3, por favor `less create_directories.sh` y asegúrese de que no es un troyano.

No está bien visto ejecutar aplicaciones que consumen muchos recursos. A menos que esté seguro de que no afectará a otros usuarios, no ejecute trabajos como #1 (`python`) o #4 (código MD personalizado). Si no está seguro, pida consejo a su administrador de sistemas.



:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

Si experimenta problemas de rendimiento con un nodo de inicio de sesión, debe informar de ello al personal del sistema (normalmente a través del servicio de asistencia) para que lo investiguen.

## Prueba antes de escalar

Recuerde que generalmente se le cobra por el uso en sistemas compartidos. Un simple error en un script de trabajo puede acabar costando una gran cantidad de presupuesto de recursos. Imagina un script de trabajo con un error que hace que se quede sin hacer nada durante 24 horas en 1000 núcleos o uno en el que has solicitado 2000 núcleos por error y ¡sólo utilizas 100 de ellos! Este problema puede agravarse cuando la gente escribe scripts que automatizan el envío de trabajos (por ejemplo, cuando se ejecuta el mismo cálculo o análisis sobre muchos parámetros o archivos diferentes). Cuando esto ocurre, te perjudica tanto a ti (ya que desperdicias muchos recursos cargados) como a otros usuarios (que ven bloqueado su acceso a los nodos de cálculo ociosos). En recursos muy ocupados puedes esperar muchos días en una cola para que tu trabajo falle a los 10 segundos de empezar debido a un error tipográfico trivial en el script del trabajo. Esto es extremadamente frustrante

La mayoría de los sistemas proporcionan recursos dedicados para pruebas que tienen tiempos de espera cortos para ayudarte a evitar este problema.

::::::::::::::::::::::::::::::::::::::::: callout

## Probar scripts de envío de trabajos que utilizan grandes cantidades de recursos

Antes de enviar una gran cantidad de trabajos, envíe primero uno como prueba para asegurarse de que todo funciona como se espera.

Antes de enviar un trabajo muy grande o muy largo envíe una pequeña prueba truncada para asegurarse de que el trabajo se inicia como se espera.


::::::::::::::::::::::::::::::::::::::::::::::::::

## Tener un plan de copias de seguridad

Aunque muchos sistemas HPC guardan copias de seguridad, no siempre cubren todos los sistemas de ficheros disponibles y puede que sólo sean para fines de recuperación de desastres (es decir, para restaurar todo el sistema de ficheros en caso de pérdida en lugar de un fichero o directorio individual que hayas borrado por error). Proteger los datos críticos de la corrupción o el borrado es principalmente tu responsabilidad: mantén tus propias copias de seguridad.

Los sistemas de control de versiones (como Git) suelen tener ofertas gratuitas basadas en la nube (por ejemplo, GitHub y GitLab) que se utilizan generalmente para almacenar el código fuente. Incluso si no estás escribiendo tus propios programas, estos pueden ser muy útiles para almacenar scripts de trabajo, scripts de análisis y pequeños archivos de entrada.

Si está creando software, es posible que tenga una gran cantidad de código fuente que compila para crear su ejecutable. Dado que estos datos generalmente se pueden recuperar volviendo a descargar el código o volviendo a ejecutar la operación de comprobación desde el repositorio de código fuente, también es menos importante proteger estos datos.

Para grandes cantidades de datos, especialmente resultados importantes de sus ejecuciones, que pueden ser irremplazables, debe asegurarse de que dispone de un sistema robusto para sacar copias de los datos del sistema HPC siempre que sea posible a un almacenamiento de copia de seguridad. Herramientas como `rsync` pueden ser muy útiles para esto.

Su acceso al sistema HPC compartido estará generalmente limitado en el tiempo, por lo que debe asegurarse de tener un plan para transferir sus datos fuera del sistema antes de que termine su acceso. No subestime el tiempo necesario para transferir grandes cantidades de datos y asegúrese de planificarlo con suficiente antelación (idealmente, antes de empezar a utilizar el sistema para su investigación).

En todos estos casos, el servicio de asistencia del sistema que esté utilizando debería poder proporcionarle orientación útil sobre las opciones de transferencia de datos para los volúmenes de datos que vaya a utilizar.

::::::::::::::::::::::::::::::::::::::::: callout

## Tus datos son tu responsabilidad

Asegúrate de que entiendes cuál es la política de copias de seguridad de los sistemas de archivos del sistema que estás utilizando y qué implicaciones tiene para tu trabajo si pierdes tus datos en el sistema. Planifica tus copias de seguridad de los datos críticos y cómo transferirás los datos fuera del sistema a lo largo del proyecto.


::::::::::::::::::::::::::::::::::::::::::::::::::

## Transfiriendo datos

Como se mencionó anteriormente, muchos usuarios se encuentran con el reto de transferir grandes cantidades de datos fuera de los sistemas HPC en algún momento (esto es más frecuente en la transferencia de datos fuera que dentro de los sistemas, pero los consejos a continuación se aplican en ambos casos). La velocidad de transferencia de datos puede estar limitada por muchos factores diferentes, por lo que el mejor mecanismo de transferencia de datos que se puede utilizar depende del tipo de datos que se transfieren y a dónde van los datos.

Los componentes entre el origen y el destino de tus datos tienen diferentes niveles de rendimiento y, en particular, pueden tener diferentes capacidades con respecto al **ancho de banda** y la **latencia**.

**El ancho de banda** es generalmente la cantidad bruta de datos por unidad de tiempo que un dispositivo es capaz de transmitir o recibir. Es una métrica común y generalmente bien entendida.

**La latencia** es algo más sutil. Para las transferencias de datos, se puede considerar como la cantidad de tiempo que se tarda en sacar los datos del almacenamiento y ponerlos en una forma transmisible. Los problemas de latencia son la razón por la que es aconsejable ejecutar las transferencias de datos moviendo un pequeño número de archivos grandes, en lugar de lo contrario.

Algunos de los componentes clave y sus problemas asociados son:

- **Velocidad del disco**: Los sistemas de archivos de los sistemas HPC suelen ser altamente paralelos y constan de un gran número de unidades de disco de alto rendimiento. Esto les permite soportar un ancho de banda de datos muy elevado. A menos que el sistema remoto tenga un sistema de ficheros paralelo similar, la velocidad de transferencia puede verse limitada por el rendimiento del disco en ese extremo.
- **Rendimiento de los metadatos**: *Las operaciones de metadatos*, como abrir y cerrar archivos o listar el propietario o el tamaño de un archivo, son mucho menos paralelas que las operaciones de lectura/escritura. Si tus datos consisten en un gran número de archivos pequeños, puede que la velocidad de transferencia se vea limitada por las operaciones de metadatos. Las operaciones de metadatos realizadas por otros usuarios del sistema también pueden interactuar fuertemente con las que tú realizas, por lo que reducir el número de operaciones de este tipo que utilizas (combinando varios archivos en uno solo) puede reducir la variabilidad en tus tasas de transferencia y aumentar la velocidad de transferencia.
- **Velocidad de red**: El rendimiento de la transferencia de datos puede verse limitado por la velocidad de la red. Más importante aún, está limitado por la sección más lenta de la red entre el origen y el destino. Si está transfiriendo a su ordenador portátil/estación de trabajo, es probable que ésta sea su conexión (ya sea a través de LAN o WiFi).
- **Velocidad del cortafuegos**: La mayoría de las redes modernas están protegidas por algún tipo de cortafuegos que filtra el tráfico malicioso. Este filtrado tiene cierta sobrecarga y puede reducir el rendimiento de la transferencia de datos. Las necesidades de una red de uso general que aloja servidores web o de correo electrónico y equipos de sobremesa son muy distintas de las de una red de investigación que necesita transferir grandes volúmenes de datos. Si intentas transferir datos a o desde un host en una red de propósito general, puede que el cortafuegos de esa red limite la velocidad de transferencia que puedes alcanzar.

Como se ha mencionado anteriormente, si tienes datos relacionados que consisten en un gran número de archivos pequeños, se recomienda encarecidamente empaquetar los archivos en un archivo *archive* más grande para su almacenamiento y transferencia a largo plazo. Un único archivo grande hace un uso más eficiente del sistema de archivos y es más fácil de mover, copiar y transferir porque se requieren muchas menos operaciones de metadatos. Los ficheros de archivo pueden crearse con herramientas como `tar` y `zip`. Ya conocimos `tar` cuando hablamos anteriormente de la transferencia de datos.

¡[Diagrama esquemático de ancho de banda y latencia para E/S de disco y red. Cada uno de los componentes de la figura está conectado por una línea azul de anchura proporcional al ancho de banda de la interfaz. Los pequeños laberintos en los puntos de enlace ilustran la latencia del enlace, con laberintos más tortuosos que indican una latencia mayor.](/fig/responsibility-bandwidth.svg){max-width="90%" alt="Esquema del ancho de banda de la red"}

::::::::::::::::::::::::::::::::::::::: challenge

## Considere la mejor forma de transferir datos

Si vas a transferir grandes cantidades de datos, tendrás que pensar en qué puede afectar al rendimiento de la transferencia. Siempre es útil realizar algunas pruebas que puedas utilizar para extrapolar el tiempo que tardará la transferencia de tus datos.

Supongamos que tienes una carpeta "data" que contiene unos 10.000 archivos, una buena mezcla de pequeños y grandes datos ASCII y binarios. ¿Cuál de las siguientes opciones sería la mejor para transferirlos a HPC Carpentry's Cloud Cluster?

1. `scp -r data yourUsername@cluster.hpc-carpentry.org:~/`
2. `rsync -ra data yourUsername@cluster.hpc-carpentry.org:~/`
3. `rsync -raz data yourUsername@cluster.hpc-carpentry.org:~/`
4. `tar -cvf data.tar data;`

   `rsync -raz data.tar yourUsername@cluster.hpc-carpentry.org:~/`
5. `tar -cvzf data.tar.gz data;`

   `rsync -ra data.tar.gz yourUsername@cluster.hpc-carpentry.org:~/`

::::::::::::::: solution

## Solución

1. `scp` copiará recursivamente el directorio. Esto funciona, pero sin compresión.
2. `rsync -ra` funciona como `scp -r`, pero conserva la información del archivo, como las horas de creación. Esto es ligeramente mejor.
3. `rsync -raz` añade compresión, lo que ahorrará algo de ancho de banda. Si tienes una CPU potente en ambos extremos de la línea, y estás en una red lenta, ésta es una buena opción.
4. Este comando utiliza primero `tar` para unir todo en un único archivo, y luego `rsync -z` para transferirlo con compresión. Con este *número* tan grande de ficheros, la sobrecarga de metadatos puede dificultar la transferencia, por lo que es una buena idea.
5. Este comando utiliza `tar -z` para comprimir el archivo y luego `rsync` para transferirlo. Puede tener un rendimiento similar a #4, pero en la mayoría de los casos (para grandes conjuntos de datos), es la mejor combinación de alto rendimiento y baja latencia (aprovechando al máximo el tiempo y la conexión de red).

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Ten cuidado con el uso del nodo login.
- Tus datos en el sistema son tu responsabilidad.
- Planifica y prueba grandes transferencias de datos.
- A menudo es mejor convertir muchos ficheros en un único archivo antes de transferirlos.

::::::::::::::::::::::::::::::::::::::::::::::::::



