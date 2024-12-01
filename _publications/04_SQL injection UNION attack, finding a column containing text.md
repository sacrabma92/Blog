---
title: "04 SQL injection UNION attack, finding a column containing text"
collection: publications
category: portswigger
permalink: /publications/04_SQL injection UNION attack, finding a column containing text
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el filtro de categoría de productos. Para resolver el laboratorio, realizamos un ataque de inyección SQL que devuelve una fila adicional que contiene el valor proporcionado. Esta técnica nos ayuda a determinar qué columnas son compatibles con datos de cadena.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQL injection UNION attack, finding a column containing text'
---

![logo]({{site.url}}/images/SQLi/sqli-4/logo.png)

Lab-Link: <https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text>  
Dificultad: PRACTITIONER  

## Descripción del laboratorio

![lab_description]({{site.url}}/images/SQLi/sqli-4/lab_description.png)

###  Objetivo

Buscar columnas que contengan datos de cadena con un ataque UNION devolviendo una cadena aleatoria proporcionada.

## Query

La consulta utilizada en el laboratorio tendrá un aspecto similar a

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>'
```

## Pasos

### Confirmar argumento inyectable

Los primeros pasos son idénticos a los del laboratorio [SQL injection UNION attack, determining the number of columns returned by the query](https://sacrabma92.github.io/Blog//publications/03_SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query) y no se repiten aquí.

Como resultado de estos pasos, descubrí que el número de columnas en el resultado es 3.

### Búsqueda de columnas de texto

En una UNION, ambas consultas deben coincidir en el número de columnas así como en los tipos de datos de cada columna.

Los campos `null` coinciden con cualquier tipo de dato. Intercambiando sucesivamente un único null con una cadena, por ejemplo 'x', puedo averiguar qué columnas contienen datos de cadena.

Añadiendo un `' UNION (SELECT null, 'x', null)--` a la consulta, la sentencia SQL tiene este aspecto y da como resultado una 'x' añadida en la tabla.

```sql
SELECT * FROM someTable WHERE category = 'Accessories' UNION (select null, 'x', null)--'
```

![Injected content]({{site.url}}/images/SQLi/sqli-4/Injected content.png)

Inyectando la cadena solicitada se resolverá el ejercicio:

![success]({{site.url}}/images/SQLi/sqli-4/success.png)
