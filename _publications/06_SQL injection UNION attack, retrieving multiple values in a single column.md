---
title: "06 SQL injection UNION attack, retrieving multiple values in a single column"
collection: publications
category: portswigger
permalink: /publications/06_SQL injection UNION attack, retrieving multiple values in a single column
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el campo categoría de producto. Para resolver el laboratorio, realizamos un ataque de inyección SQL basado en UNION que recupera los nombres de usuario y las contraseñas de los usuarios de la aplicación en una única columna.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQL injection UNION attack, retrieving multiple values in a single column'
---

![logo]({{site.url}}/images/SQLi/sqli-6/logo.png)

Lab-Link: <https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column>  
Dificultad: PRACTITIONER  

## Descripción del laboratorio

![lab_description]({{site.url}}/images/SQLi/sqli-6/lab_description.png)

## Query

La consulta utilizada en el laboratorio tendrá un aspecto similar a

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>'
```

## Pasos

### Confirmar argumento inyectable

Los primeros pasos son idénticos a los de los laboratorios [SQL injection UNION attack, determining the number of columns returned by the query](https://sacrabma92.github.io/Blog//publications/03_SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query) y [SQL injection UNION attack, finding a column containing text](https://sacrabma92.github.io/Blog//publications/04_SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text) y no se repiten aquí.

Como resultado de estos pasos, descubro que el número de columnas es 2, siendo sólo la segunda una columna de texto.

### Buscar base de datos utilizada

Quiero extraer tanto los nombres de usuario como las contraseñas, pero con una sola columna de cadena. Para ello, puedo concatenar varios valores en campos únicos. La sintaxis depende del motor de la base de datos, así que averigua qué base de datos se utiliza para manejar la página.

Portswigger tiene una buena [hoja de trucos](https://portswigger.net/web-security/sql-injection/cheat-sheet) que contiene consultas sobre la versión de la base de datos.

Afortunadamente, el segundo intento tiene éxito y da como resultado los detalles de la base de datos.

![database version query]({{site.url}}/images/SQLi/sqli-6/db_query.png)

![database used]({{site.url}}/images/SQLi/sqli-6/db_used.png)

### Extracción de nombres de usuario y contraseñas

Sé qué tabla (`users`) contiene las credenciales (columnas `username` y `password`). Como sólo tengo una columna de cadena, puedo hacer dos consultas y combinar manualmente los datos (para que esto funcione tengo que asegurarme de que el orden es idéntico), o concatenar estos valores en una sola cadena para poder extraerlos de una sola vez.

Esta última versión es mucho más cómoda. Afortunadamente, la [hoja de trucos](https://portswigger.net/web-security/sql-injection/cheat-sheet) mencionada anteriormente también contiene la información necesaria:

![cheat sheet content regarding concat]({{site.url}}/images/SQLi/sqli-6/postgres_concat_cheatsheet.png)

Para poder distinguir el nombre de usuario de la contraseña, también necesito concatenar alguna cadena única entre ambos. Utilizo una categoría inválida para que no se encuentre ningún artículo y sólo aparezca mi salida.

Inyecto `X' UNION (SELECT null,username || '~~~' || password FROM users)--` para crear una consulta SQL como esta:

```sql
SELECT * FROM someTable WHERE category = 'X' UNION (SELECT null,username || '~~~' || password FROM users)--
```

El resultado son tres credenciales de usuario:

![credentials]({{site.url}}/images/SQLi/sqli-6/credentials.png)

El único paso que queda es iniciar sesión como administrador y el laboratorio se actualiza a

![success]({{site.url}}/images/SQLi/sqli-6/success.png)
