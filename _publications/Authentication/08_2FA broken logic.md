---
title: "08 2FA broken logic"
collection: publications
category: portswigger
permalink: publications/Authentication/08_2FA broken logic
excerpt: 'La autenticación de dos factores de este laboratorio es vulnerable debido a su lógica defectuosa.'
date: 2024-12-05
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: '2FA broken logic'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-08/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-08/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, accedemos a la página de la cuenta de Carlos.

## Recursos Necesarios

* [Nombre usuarios](https://portswigger.net/web-security/authentication/auth-lab-usernames)
* [Contraseñas](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Analisis

Probamos la autenticación con un usuario valido con el cual tengamos las credenciales y vemos el proceso de login desde el BurpSuite.

![login]({{site.url}}/images/Authentication/authentication-lab-08/login.png)

El primer Endponit es `/login` el cual transporta el usuario y la contraseña ingresados

![login 1]({{site.url}}/images/Authentication/authentication-lab-08/login1.png)

El segundo Endpoint es `/login2` el cual es el que recibe el token 2FA que se envio al correo para validar el usuario

![login 2]({{site.url}}/images/Authentication/authentication-lab-08/login2.png)

## Modificar Cookie

Modificamos para que envie un codigo 2FA al usuario `carlos`, tambien modificamos el metodo a `GET`.

![Eliminar]({{site.url}}/images/Authentication/authentication-lab-08/eliminar.png)

Con el metodo anterior enviamos un codigo 2FA al correo del usuario `carlos` ahora podemos hacer un intruder al codigo para averiguar cual es valido.

![Intruder]({{site.url}}/images/Authentication/authentication-lab-08/intruder.png)

Configuración del payload

![brute]({{site.url}}/images/Authentication/authentication-lab-08/Brute.png)

Filtramos por `Status code`

![Estatus]({{site.url}}/images/Authentication/authentication-lab-08/estatus.png)

Copiamos el link que nos proporciona y lo pegamos en el navegador

# Aprobado

![Aprobado]({{site.url}}/images/Authentication/authentication-lab-08/aprobado.png)


