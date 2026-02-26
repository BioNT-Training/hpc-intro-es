---
site: sandpaper::sandpaper_site
---

::::::::::::::::::::::::::::::::::::::::::  callout

## Aviso legal

Este material de formación tiene fines exclusivamente educativos e
informativos. Explica cómo utilizar un entorno HPC basado en
[SLURM](https://slurm.schedmd.com/overview.html),
[módulos](https://lmod.readthedocs.io)
y tecnologías relacionadas, pero no proporciona acceso
a la infraestructura computacional necesaria para completar los ejercicios.

Los participantes son responsables de gestionar su propio acceso a recursos
computacionales adecuados.

Como alternativa, los usuarios pueden considerar la configuración de un
entorno de prueba local utilizando el proyecto de código abierto
[slurm-docker-cluster](https://github.com/giovtorres/slurm-docker-cluster).
Un ejemplo detallado de cómo utilizar este proyecto se describe en una
[entrada de blog de terceros de Thomas Sandmann](https://tomsing1.github.io/blog/posts/slurm_docker_cluster/).

El uso de herramientas o documentación de terceros se realiza bajo su propia
discreción y riesgo.

::::::::::::::::::::::::::::::::::::::::::::::::::


Este taller es una introducción al uso eficaz de los sistemas informáticos de alto rendimiento. No podemos cubrir todos los casos ni dar un curso exhaustivo sobre programación paralela en sólo dos días de docencia. En su lugar, este taller pretende ofrecer a los estudiantes una buena introducción y una visión general de las herramientas disponibles y de cómo utilizarlas eficazmente.

:::::::::::::::::::::::::::::::::::::::::: prereq

## Requisitos previos

Para esta lección es necesario tener experiencia con la línea de comandos. Recomendamos a los participantes que pasen por [shell-novice](https://swcarpentry.github.io/shell-novice/), si son nuevos en la línea de comandos (también conocida como terminal o shell).

::::::::::::::::::::::::::::::::::::::::::::::::::

Al finalizar este taller, los alumnos sabrán cómo:

- Identificar los problemas que una agrupación puede ayudar a resolver
- Utiliza el shell UNIX (también conocido como terminal o línea de comandos) para conectarte a un cluster.
- Transferencia de archivos a un clúster.
- Enviar y gestionar trabajos en un cluster utilizando un planificador.
- Observa las ventajas y limitaciones de la ejecución en paralelo.

::::::::::::::::::::::::::::::::::::::::: callout

## Primeros pasos

Para empezar, por favor sigue las [Instrucciones de configuración](learners/setup.md) para asegurarte de que tienes un terminal y una aplicación SSH.

::::::::::::::::::::::::::::::::::::::::::::::::::

Tenga en cuenta que este es el borrador de HPC Carpentry. Comentarios y opiniones son bienvenidos.

::::::::::::::::::::::::::::::::::::::::: callout

## Para instructores

Si impartes esta lección en un taller, consulta las [Notas del instructor](instructors/instructor-notes.md).

::::::::::::::::::::::::::::::::::::::::::::::::::


