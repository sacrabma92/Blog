---
title: "05 Username enumeration via response timing"
collection: publications
category: portswigger
permalink: publications/Authentication/04_Username enumeration via subtly different responses
excerpt: 'Este laboratorio es sutilmente vulnerable a ataques de enumeración de nombres de usuario y de fuerza bruta de contraseñas. Tiene una cuenta con un nombre de usuario y una contraseña predecibles.'
date: 2034-12-04
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Username enumeration via response timing'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-05/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-05/descripcion.png)

## Objetivo del laboratorio

Para resolverlo, enumeramos un nombre de usuario válido, forzamos su contraseña y accedemos a la página de su cuenta.

## Recursos Necesarios

* [Nombre usuarios](https://portswigger.net/web-security/authentication/auth-lab-usernames)
* [Contraseñas](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Analisis

Tratamos de loguearnos con algún usuario valido pero con password erronea. Con la longitud de password que estamos ingresando el computo dura 408 mili segundos.

![Password]({{site.url}}/images/Authentication/authentication-lab-05/password.png)

Y nos fijamos que si ingresamos un password muy largo, el backend se demora mas tiempo en realizar el computo y devolver una respuesta. Si el password es mas grande el tiempo aumenta el tiempo que tarda en responder.

![Password 2]({{site.url}}/images/Authentication/authentication-lab-05/password2.png)

Si seguimos tratando de hacer login, la web al intento 3 nos bloquea y nos toca esperar un tiempo preventivo.

![Tiempo]({{site.url}}/images/Authentication/authentication-lab-05/tiempo.png)

Para saltar esta validación colocamos en la cabecera de la petición `X-Forwarded_For:` y vemos que se pueden realizar mas peticiones, pero tener presente que maximo son 3 intentos, al tercero se bloquea y toca cambiar IP.

`X-Forwarded-For` es un encabezado HTTP estándar que se utiliza principalmente para identificar la dirección IP original del cliente que realiza una solicitud a un servidor web a través de un proxy, balanceador de carga o cualquier otro intermediario.

![x]({{site.url}}/images/Authentication/authentication-lab-05/x.png)

## Enumeración de usuarios

Como debemos cambiar el `X-Forwarded-For` en el Intruder y el usuario y filtrar por tiempo.

![Intruder]({{site.url}}/images/Authentication/authentication-lab-05/intruder.png)

Filtramos por tiempo, con la longitud de password que estamos ingresando, el computo dura 900 mili segundos. Asi que ordenamos por ese tiempo ya que estamos ingresando el mismo password. De esta forma enumeramos por tiempo de respuesta.

![Mili]({{site.url}}/images/Authentication/authentication-lab-05/mili.png)

## Password

Ya que tenemos usuarios ahora procedemos a realizar un ataque con los usuarios a ver con cual podemos obtener el password correcto. Para validar obtendremos una redirección `302`.

![Pass]({{site.url}}/images/Authentication/authentication-lab-05/pass.png)

## Login

Realizamos el login con las credenciales obtenidas.

![Login]({{site.url}}/images/Authentication/authentication-lab-05/login.png)

Obtuvimos una redirección

![Redirección]({{site.url}}/images/Authentication/authentication-lab-05/302.png)

## Aprobado

![Aprobado]({{site.url}}/images/Authentication/authentication-lab-05/aprobado.png)