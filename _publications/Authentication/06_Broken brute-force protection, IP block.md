---
title: "06 Broken brute-force protection, IP block"
collection: publications
category: portswigger
permalink: publications/Authentication/06_Broken brute-force protection, IP block
excerpt: 'Este laboratorio es vulnerable debido a un fallo lógico en su protección contra la fuerza bruta de contraseñas. Para resolver el problema, forzamos la contraseña de la víctima, iniciamos sesión y accedemos a la página de su cuenta. 
Your credentials: wiener:peter
Victim/s username: carlos'
date: 2024-12-05
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Broken brute-force protection, IP block'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-06/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-06/descripcion.png)

## Objetivo del laboratorio

Para resolver el problema, forzamos la contraseña de la víctima, iniciamos sesión y accedemos a la página de su cuenta.

## Recursos Necesarios

* [Nombre usuarios](https://portswigger.net/web-security/authentication/auth-lab-usernames)
* [Contraseñas](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Analisis

Probamos el funcionamiento del login, obtenemos el primer fallo que muestra que el usuario es invalido, asi que podemos realizar un ataque para enumerar usuarios.

![Usuarios]({{site.url}}/images/Authentication/authentication-lab-06/usuarios.png)

Al realizar varios intentos fallidos, obtenemos un bloqueo temporal de 1 minuto.

![Bloqueo]({{site.url}}/images/Authentication/authentication-lab-06/bloqueo.png)

Se encontro un patrón, del cual nos podemos aprovechar. Cuando hacemos 2 intentos fallidos de login, luego hacemos un login con credenciales validas, reseteamos el contados de los intentos fallidos. 

## Script

Script que nos permitira crear la lista de usuarios y passwords, de forma tal que, tenga 2 intentos de pruebas y 1 credencial valida para realizar login.

```python
print("###########The following are the usernames:###############")
for i in range(150):
    if i % 3:
        print("carlos")
    else:
        print("wiener")


print("##############The following are the passwords:############")
with open('passwords.txt', 'r') as f:
    lines = f.readlines()

i = 0
for pwd in lines:
    if i % 3:
        print(pwd.strip('\n'))
    else:
        print("peter")
        print(pwd.strip('\n'))
        i = i+1
    i = i +1 
```
Convinación de nombres de usuarios

![convi]({{site.url}}/images/Authentication/authentication-lab-06/convi.png)

Convinación de passwords 

![convi2]({{site.url}}/images/Authentication/authentication-lab-06/convi2.png)

## Enumeración de usuarios

Realizamos un Intruder con las credenciales creadas. Y tener presente que debemos configurar el `Resource pool` para 1 hilo el ataque.

![Ataque]({{site.url}}/images/Authentication/authentication-lab-06/ataque.png)

Obtuvimos una respuesta para el usuarios `carlos`

## Login

![Login]({{site.url}}/images/Authentication/authentication-lab-06/login.png)

## Resuelto

![Resuelto]({{site.url}}/images/Authentication/authentication-lab-06/resuelto.png)