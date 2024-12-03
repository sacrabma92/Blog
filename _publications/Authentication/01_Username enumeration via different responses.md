---
title: "01 Username enumeration via different responses"
collection: publications
category: portswigger
permalink: publications/Authentication/01_Username enumeration via different responses
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL. La aplicación utiliza una cookie de seguimiento para análisis y realiza una consulta SQL que contiene el valor de la cookie enviada. No se devuelven los resultados de la consulta SQL.'
date: 2024-12-03
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Username enumeration via different responses'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-01/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-01/descripcion.png)

## Objetivo del laboratorio

Para resolver el problema, enumera un nombre de usuario válido, fuerza su contraseña y accede a la página de su cuenta.

## Recursos Necesarios

* [Nombre usuarios](https://portswigger.net/web-security/authentication/auth-lab-usernames)
* [Contraseñas](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Login

Vamos a ver como funciona el login, si tiene restricción de intentos, miramos el mensaje que arroja. Y como podemos observar hay una falla, al mostrar que el usuario es invalido, lo podemos usar para enumerar usuarios existentes.

![logo]({{site.url}}/images/Authentication/authentication-lab-01/usuario invalido.png)

## Enumerar usuarios

Como primera medida vamos a enunmerar nombre de usuarios, hasta que el mensaje en la pagina sea diferente. Si cambia significa que el usuario existe.

![Intruder]({{site.url}}/images/Authentication/authentication-lab-01/intruder.png)

Probamos haciendo un ataque de fuerza bruta. Y filtramos, que no aparezca `Invalid username` en el codigo.

![Filtro]({{site.url}}/images/Authentication/authentication-lab-01/filtro.png)

Nos aparece un usuario que es valido.

![Filtro 2]({{site.url}}/images/Authentication/authentication-lab-01/filtro2.png)

Probamos ingresando el usuario que nos arrojo con un password, para ver que mensaje de error nos muestra.

![Passowrd]({{site.url}}/images/Authentication/authentication-lab-01/password invalido.png)

## Enumerar passwords

Ya que tenemos un usuario valido, lo ingresamos en el campo de `username`

![Intruder 2]({{site.url}}/images/Authentication/authentication-lab-01/intruder2.png)

Probamos haciendo un ataque de fuerza bruta. Y filtramos, que no aparezca `Incorrect password` en el codigo.

![Filtro 3]({{site.url}}/images/Authentication/authentication-lab-01/filtro3.png)

Nos muestra un posible password, intentamos logueando con los datos obtenidos.

![password]({{site.url}}/images/Authentication/authentication-lab-01/password.png)

# Login

Procedemos a ingresar los datos obtenidos:

![Login]({{site.url}}/images/Authentication/authentication-lab-01/aprobado.png)
