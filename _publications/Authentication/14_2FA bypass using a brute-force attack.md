---
title: "14 2FA bypass using a brute-force attack"
collection: publications
category: portswigger
permalink: publications/Authentication/14 2FA bypass using a brute-force attack
excerpt: 'La autenticación de dos factores de este laboratorio es vulnerable a la fuerza bruta. Ya has obtenido un nombre de usuario y una contraseña válidos, pero no tienes acceso al código de verificación 2FA del usuario. '
date: 2024-12-06
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: '2FA bypass using a brute-force attack'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-14/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-14/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, forzaremos el código 2FA y accederemos a la página de la cuenta de Carlos.

## Analisis

Logueamos con un usuario correcto.

![Login]({{site.url}}/images/Authentication/authentication-lab-14/login.png)

Vemos que nos pide un codigo 2FA para autenticarnos, ingresamos algunos numeros para obtener el endpoint.

![Auth]({{site.url}}/images/Authentication/authentication-lab-14/auth.png)

Obtenemos los endpoints de `Login` y `Login2` que es el de 2FA

![EndPoint]({{site.url}}/images/Authentication/authentication-lab-14/endpoints.png)

## Macro

```
Una macro en Burp Suite es una funcionalidad utilizada para automatizar una secuencia de solicitudes y respuestas HTTP. Es especialmente útil cuando se trabaja con aplicaciones que requieren autenticación o procesos específicos antes de ejecutar pruebas. Una macro permite simular el comportamiento del usuario o completar pasos previos necesarios para realizar pruebas, como iniciar sesión automáticamente.
```

Necesitamos configurar una macro para que BurpSuite simule que una persona esta haciendo login. Y poder automatizar el ataque.

Vamos a la pestaña de `Settings`

![Conf1]({{site.url}}/images/Authentication/authentication-lab-14/conf1.png)

Presionamos en la pestaña `Sessions` luego en `Add` para añadir una regla

![Conf2]({{site.url}}/images/Authentication/authentication-lab-14/conf2.png)

Vamos a la pestaña `Scope` y marcamos incluir todas las url `Include URLs`

![Conf3]({{site.url}}/images/Authentication/authentication-lab-14/conf3.png)

Volvemos a la pestaña de `Details` y presionamos el boton `Add` seleccionamos la opción de `Run a macro`

![Conf4]({{site.url}}/images/Authentication/authentication-lab-14/conf4.png)

Debemos seleccionar los endpoints en la logica la cual el programa va a simular hacer un login como un humano

![Conf5]({{site.url}}/images/Authentication/authentication-lab-14/conf5.png)

Corremos un test para verificar que la macro esta bien configurada

![Conf6]({{site.url}}/images/Authentication/authentication-lab-14/conf6.png)

## Intruder

Vamos a configurar un payload para la petición `Login2` para hacer un ataque al codigo de 2FA.

![Conf7]({{site.url}}/images/Authentication/authentication-lab-14/conf7.png)

Cargamos el payload de `Brute Force` y colocamos el `Resource Pool` en 1. Y como va a ser de puros numero.

![Conf8]({{site.url}}/images/Authentication/authentication-lab-14/conf8.png)

Buscamos por `Status` y vemos que tenemos una redirección `302`

![Conf9]({{site.url}}/images/Authentication/authentication-lab-14/conf9.png)

## Aprobado

![Conf10]({{site.url}}/images/Authentication/authentication-lab-14/conf10.png)