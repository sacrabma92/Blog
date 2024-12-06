---
title: "11 Password reset poisoning via middleware"
collection: publications
category: portswigger
permalink: publications/Authentication/11_Password reset poisoning via middleware
excerpt: 'Este laboratorio es vulnerable al envenenamiento por restablecimiento de contraseña. El usuario carlos hará clic por descuido en cualquier enlace de los correos electrónicos que reciba. Para resolver el laboratorio, iniciamos sesión en la cuenta de Carlos.'
date: 2024-12-06
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Password reset poisoning via middleware'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-11/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-11/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, iniciamos sesión en la cuenta de Carlos.

## Analisis

Vamos a reestabler la contraseña del usuario 

![Olvidar]({{site.url}}/images/Authentication/authentication-lab-11/olvidar.png)

Ingresamos el usuario o correo para reestablecer.

![Usuario]({{site.url}}/images/Authentication/authentication-lab-11/usuario.png)

Vamos al servidor que simula un correo electronico.

![Server]({{site.url}}/images/Authentication/authentication-lab-11/server.png)

Accedemos al servidor de correo

![Email]({{site.url}}/images/Authentication/authentication-lab-11/email.png)

El link de reestablecer el correo

![Email2]({{site.url}}/images/Authentication/authentication-lab-11/email2.png)

Resetear el password

![Reset]({{site.url}}/images/Authentication/authentication-lab-11/reset.png)

Respuesta Burp Suite

![usuario2]({{site.url}}/images/Authentication/authentication-lab-11/usuario2.png)

## Reestablecer

Vamos a reestablecer un password insertando el servidor al cual llegara la petición de reestablecer la contraseña. 

![Server2]({{site.url}}/images/Authentication/authentication-lab-11/server2.png)
![Server3]({{site.url}}/images/Authentication/authentication-lab-11/server3.png)

Añadimos lo siguiente a la cabecera y le colocamos el servidor al cual llegara la solicitud.

```El encabezado X-Forwarded-Host es una parte de las cabeceras HTTP utilizadas en entornos con proxies o balanceadores de carga para comunicar la información original del host solicitado por el cliente al servidor de backend.```

```javascript
X-Forwarded-Host:
```
![Server4]({{site.url}}/images/Authentication/authentication-lab-11/server4.png)

Una vez enviada la petición vemos el servidor y nos llego un token temporal. Lo pegamos para reestablecer el password

![Reset]({{site.url}}/images/Authentication/authentication-lab-11/reset2.png)

Probamos si reestablecio el passwor de `carlos`

![login]({{site.url}}/images/Authentication/authentication-lab-11/login.png)

## Aprobado

![Aprobado]({{site.url}}/images/Authentication/authentication-lab-11/aprobado.png)

