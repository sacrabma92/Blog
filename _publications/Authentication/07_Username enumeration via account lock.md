---
title: "07 Username enumeration via account lock"
collection: publications
category: portswigger
permalink: publications/Authentication/07_Username enumeration via account lock
excerpt: 'Este laboratorio es vulnerable a la enumeración de nombres de usuario. Utiliza el bloqueo de cuentas, pero esto contiene un fallo lógico.'
date: 2024-12-05
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Username enumeration via account lock'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-07/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-07/descripcion.png)

## Objetivo del laboratorio

Para resolver el problema, forzamos la contraseña de la víctima, iniciamos sesión y accedemos a la página de su cuenta.

## Recursos Necesarios

* [Nombre usuarios](https://portswigger.net/web-security/authentication/auth-lab-usernames)
* [Contraseñas](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Analisis

Probamos el funcionamiento del login, vemos que podemos realizar la misma petición y NO tiene ningun mecanismo de bloqeo aparentemente. Solamente arroja el mensaje `Invalid username or password.`

![Fail]({{site.url}}/images/Authentication/authentication-lab-07/fail.png)

## Brute Force 

Realizaremos fuerza bruta al usuario y password, para averiguar cual coinciden.

![Intruder]({{site.url}}/images/Authentication/authentication-lab-07/intruder.png)

Le colocamos `$$` en el password ya que va a ser un valor nulo.

![Nulo]({{site.url}}/images/Authentication/authentication-lab-07/nulo.png)

Realizamos  el ataque y filtramos por el tamaño `Length` y vemos un valor extraño

![Usuario]({{site.url}}/images/Authentication/authentication-lab-07/usuario.png)

Ahora probamos con el usuario encontrado, y hacemos un Brute Force al password.

![Intruder]({{site.url}}/images/Authentication/authentication-lab-07/intruder2.png)

Filtramos por `Length` y vemos un resultado que no muestra el error y probamos con esa password

![Resultado]({{site.url}}/images/Authentication/authentication-lab-07/resultado.png)

## Login

![Login]({{site.url}}/images/Authentication/authentication-lab-07/login.png)

## Aprobado

![Resuelto]({{site.url}}/images/Authentication/authentication-lab-07/resuelto.png)