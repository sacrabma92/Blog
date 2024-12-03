---
title: "02 2FA simple bypass"
collection: publications
category: portswigger
permalink: publications/Authentication/02 2FA simple bypass
excerpt: 'La autenticación de dos factores de este laboratorio se puede eludir. Ya has obtenido un nombre de usuario y una contraseña válidos, pero no tienes acceso al código de verificación 2FA del usuario.'
date: 2024-12-03
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: '2FA simple bypass'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-02/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-02/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, accede a la página de la cuenta de Carlos.

## Recursos Necesarios

* [Nombre usuarios](https://portswigger.net/web-security/authentication/auth-lab-usernames)
* [Contraseñas](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Procedimiento

Este deberia ser el procedimiento para realziar el 2FA correctamente.
En la descripción del laboratorio nos dan un usuario el cual podemos loguear. Procedemos a loguearnos y miramos el intercambio que se presenta cuando hacemos login.

Credenciales: `wiener:peter`
Credenciales de la victima: `carlos:montoya`

![2FA]({{site.url}}/images/Authentication/authentication-lab-02/2fa.png)

Una vez realizado el login, nos pide que ingresemos un codigo que nos fue enviado al correo. El correo lo simula la plataforma en el boton que dice `Email client` ingresamos ahi y copiamos el codigo y lo pegamos donde se solicita e interceptamos la petición de envio de codigo.

![Codigo]({{site.url}}/images/Authentication/authentication-lab-02/codigo.png)

Interceptado el codigo, vemos la petición que se llama `/login2` y envia el codigo que previamente ingresamos

![Codigo2]({{site.url}}/images/Authentication/authentication-lab-02/codigo2.png)

## Saltando el 2FA 

Vamos a evitar que nos solicite el 2FA y simular que no tenemos acceso al correo de `carlos`.
Logueamos con las credenciales de `carlos` e interceptamos la petición con BurpSuite

![Intercept]({{site.url}}/images/Authentication/authentication-lab-02/intercept.png)

Procedemos a oprimir el botón de `Forward` para seguir con la siguiente petición. Cambiamos en la url `my-account` precionamos ENTER para que nos lleve a la pagina y no nos pida 2FA. Y dejamos de interceptar la petición.

![Logueo]({{site.url}}/images/Authentication/authentication-lab-02/logueo.png)

Y vemos que nos hemos saltado la petición del 2FA.