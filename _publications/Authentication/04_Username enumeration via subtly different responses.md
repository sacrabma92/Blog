---
title: "04 Username enumeration via subtly different responses"
collection: publications
category: portswigger
permalink: publications/Authentication/04_Username enumeration via subtly different responses
excerpt: 'Este laboratorio es sutilmente vulnerable a ataques de enumeración de nombres de usuario y de fuerza bruta de contraseñas. Tiene una cuenta con un nombre de usuario y una contraseña predecibles.'
date: 2034-12-04
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Username enumeration via subtly different responses'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-04/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-04/descripcion.png)

## Objetivo del laboratorio

Para resolver el problema, enumera un nombre de usuario válido, fuerza su contraseña y accede a la página de su cuenta.

## Recursos Necesarios

* [Nombre usuarios](https://portswigger.net/web-security/authentication/auth-lab-usernames)
* [Contraseñas](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Respuesta

Analizamos la respuesta al ingresar un usuario y password equivocados, el mensaje de error que sale es `Invalid username or password.`.
Realice varios intento a ver si depronto me bloqueaba pero no sucedio nada. Eso es bueno porque puedo realizar un ataque con diferentes usuarios y enumerar usuarios validos.

![Mensaje]({{site.url}}/images/Authentication/authentication-lab-04/mensaje.png)

## Ataque usuario

Realizamos un ataque al campo de usuario.

![Intruder]({{site.url}}/images/Authentication/authentication-lab-04/intruder.png)

Filtramos por `Negative search` de `Invalid username or password.`.

![Filtro]({{site.url}}/images/Authentication/authentication-lab-04/filtro.png)

Aparece un posible usuario

![Usuario]({{site.url}}/images/Authentication/authentication-lab-04/usuario.png)

## Ataque password

Teniendo un usuario 'valido' lo colocamos e intentamos con diferentes password.

![Intruder2]({{site.url}}/images/Authentication/authentication-lab-04/intruder2.png)

Filtramos por `Status code` y vemos que hay un `302` de redirección.

![Resultado]({{site.url}}/images/Authentication/authentication-lab-04/resultado.png)

## Login

![login]({{site.url}}/images/Authentication/authentication-lab-04/login.png)

## Aprobado

![Respuesta]({{site.url}}/images/Authentication/authentication-lab-04/aprobado.png)