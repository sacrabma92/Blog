---
title: "02 Blind OS command injection with time delays"
collection: publications
category: portswigger
permalink: publications/Command Injection/02_Blind OS command injection with time delays
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección de comandos de SO ciego en la función de retroalimentación. La aplicación ejecuta un comando shell que contiene los detalles proporcionados por el usuario. La salida del comando no se devuelve en la respuesta.'
date: 2024-12-07
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind OS command injection with time delays'
---

![logo]({{site.url}}/images/Command Injection/command-lab-2/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Command Injection/command-lab-2/descripcion.png)

## Objetivo del laboratorio

Para resolver el problema, explotamos la vulnerabilidad de inyección ciega de comandos del sistema operativo para provocar un retraso de 10 segundos.

## Analisis

Como la descripcion del problema nos indica, es vulnerableal comprobador de existencias del producto. Vamos a analizar esta entrada para realizar la vulneración.