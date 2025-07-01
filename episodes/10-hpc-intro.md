---
title: ¿Por qué utilizar un clúster?
teaching: 15
exercises: 5
---


::::::::::::::::::::::::::::::::::::::: objectives

- Describa qué es un sistema HPC
- Identifique cómo podría beneficiarle un sistema HPC.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Por qué me interesa la computación de alto rendimiento (HPC)?
- ¿Qué puedo esperar aprender de este curso?

::::::::::::::::::::::::::::::::::::::::::::::::::

Con frecuencia, los problemas de investigación que utilizan la computación pueden superar las capacidades del ordenador de sobremesa o portátil en el que comenzaron:

- Un estudiante de estadística quiere validar un modelo de forma cruzada. Para ello, debe ejecutar el modelo 1.000 veces, pero cada ejecución tarda una hora. Ejecutar el modelo en un ordenador portátil llevaría más de un mes En este problema de investigación, los resultados finales se calculan después de ejecutar los 1000 modelos, pero normalmente sólo se ejecuta un modelo a la vez (en **serie**) en el portátil. Dado que cada una de las 1000 ejecuciones es independiente de las demás, y si se dispone de suficientes ordenadores, en teoría es posible ejecutarlas todas a la vez (en **paralelo**).
- Un investigador en genómica ha estado utilizando pequeños conjuntos de datos de secuenciación, pero pronto recibirá un nuevo tipo de datos de secuenciación que es 10 veces más grande. Abrir los conjuntos de datos en un ordenador ya supone un reto; analizar estos conjuntos de datos más grandes probablemente lo colapsará. En este problema de investigación, los cálculos necesarios podrían ser imposibles de paralelizar, pero se necesitaría un ordenador con **más memoria** para analizar el futuro conjunto de datos mucho mayor.
- Un ingeniero está utilizando un paquete de dinámica de fluidos que tiene una opción para ejecutarse en paralelo. Hasta ahora, esta opción no se utilizaba en un ordenador de sobremesa. Al pasar de simulaciones 2D a 3D, el tiempo de simulación se ha más que triplicado. Podría ser útil aprovechar esa opción o característica. En este problema de investigación, los cálculos en cada región de la simulación son en gran medida independientes de los cálculos en otras regiones de la simulación. Es posible ejecutar los cálculos de cada región simultáneamente (en **paralelo**), comunicar los resultados seleccionados a las regiones adyacentes según sea necesario y repetir los cálculos para converger en un conjunto final de resultados. Al pasar de un modelo 2D a un modelo 3D, **tanto la cantidad de datos como la cantidad de cálculos aumentan enormemente**, y en teoría es posible distribuir los cálculos entre varios ordenadores que se comunican a través de una red compartida.

En todos estos casos, se necesita acceso a más ordenadores (y más grandes). Esos ordenadores deben poder utilizarse al mismo tiempo, **resolviendo en paralelo los problemas de muchos investigadores**.

## Presentación de Jargon Busting

Abra el [HPC Jargon Buster](../files/jargon#p1) en una nueva pestaña. Para presentar el contenido, pulse `C` para abrir un **c**lone en una ventana separada, después pulse `P` para cambiar el modo de **p**resentación.

:::::::::::::::::::::::::::::::::::::: challenge

## Nunca he usado un servidor, ¿verdad?

Tómese un minuto y piense cuáles de sus interacciones diarias con un ordenador pueden requerir un servidor remoto o incluso un clúster para proporcionarle resultados.

::::::::::::::: solution

## Algunas ideas

- Comprobación del correo electrónico: su ordenador (posiblemente en el bolsillo) se pone en contacto con una máquina remota, se autentica y descarga una lista de mensajes nuevos; también carga los cambios en el estado del mensaje, como si lo ha leído, marcado como basura o borrado. Como la suya no es la única cuenta, el servidor de correo es probablemente uno de tantos en un centro de datos.
- Buscar una frase en Internet implica comparar el término de búsqueda con una base de datos masiva de todos los sitios conocidos, en busca de coincidencias. Esta operación de "consulta" puede ser sencilla, pero construir esa base de datos es una [tarea monumental][mapreduce] Los servidores intervienen en cada paso.
- Buscar direcciones en un sitio web de mapas implica conectar los puntos (A) inicial y (B) final [recorriendo un grafo][dijkstra] en busca del camino "más corto" por distancia, tiempo, gasto u otra métrica. Convertir un mapa en la forma correcta es relativamente sencillo, pero calcular todas las rutas posibles entre A y B es costoso.

Consultar el correo electrónico podría ser en serie: su máquina se conecta a un servidor e intercambia datos. La búsqueda en la base de datos del término de búsqueda (o puntos finales) también podría ser en serie, ya que una máquina recibe la consulta y devuelve el resultado. Sin embargo, montar y almacenar la base de datos completa supera con creces la capacidad de una sola máquina. Por lo tanto, estas funciones se realizan en paralelo mediante una gran colección de servidores ["hiperescala"][hiperescala] que trabajan juntos.



:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::



[mapreduce]: https://en.wikipedia.org/wiki/MapReduce
[dijkstra]: https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm
[hyperscale]: https://en.wikipedia.org/wiki/Hyperscale_computing


:::::::::::::::::::::::::::::::::::::::: keypoints

- La computación de alto rendimiento (HPC) suele implicar la conexión a sistemas informáticos muy grandes de otras partes del mundo.
- Estos otros sistemas pueden utilizarse para realizar tareas que serían imposibles o mucho más lentas en sistemas más pequeños.
- Los recursos HPC son compartidos por múltiples usuarios.
- El método estándar de interacción con estos sistemas es a través de una interfaz de línea de comandos.

::::::::::::::::::::::::::::::::::::::::::::::::::



