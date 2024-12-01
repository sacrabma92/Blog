---
title: "03 SQLi UNION attack determining the number of columns returned by the query"
collection: publications
category: portswigger
permalink: /publications/03_SQL injection UNION attack, determining the number of columns returned by the query
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en el campo de categoría del filtro de productos. Esta vulnerabilidad puede explotarse mediante un ataque UNION para recuperar datos de otras tablas. Para resolver el laboratorio, realizamos un ataque de inyección SQL que determina el número de columnas que devuelve la consulta. Este es el primer paso de un ataque UNION de inyección SQL. Utilizaremos esta técnica en laboratorios posteriores para construir el ataque completo.'
date: 2024-12-01
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQLi UNION attack determining the number of columns returned by the query'
---

![logo]({{site.url}}/images/SQLi/sqli-3/logo.png)

Lab-Link: <https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns>  
Dificultad: PRACTITIONER  

## Descripción del laboratorio

![lab_description]({{site.url}}/images/SQLi/sqli-3/lab_description.png)

## Query

La consulta utilizada en el laboratorio tendrá un aspecto similar a

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>'
```

## Pasos

### Confirmar argumento inyectable

El filtrado de categorías funciona basándose en el argumento URL `category`. Primero tengo que confirmar que este argumento es inyectable creando un error.

El argumento normal es, por ejemplo, `/filter?category=Accessories`. Empieza inyectando una comilla simple al final: `/filter?category=Accessories'`.

--> Esto resulta en un Error Interno del Servidor por parte de la aplicación ya que la consulta SQL es inválida ahora y se verá así, teniendo una comilla simple ilegal al final:

```sql
SELECT * FROM someTable WHERE category = 'Accessories''
```

A continuación, pruebo el caso bueno inyectando algo que resulta en una consulta válida: `/filter?category=Accessories%27%20or%201=1--`. El resultado es una consulta como ésta, que devuelve todas las filas y elimina la comilla simple errónea y todo lo que pueda venir después:

```sql
SELECT * FROM someTable WHERE category = 'Accessories' or 1=1--'
```

Devuelve el listado completo, básicamente el mismo contenido que si no se hubiera utilizado ningún filtro. Esto es una indicación de que lo que vendría después y se comenta no interfiere con el resultado.

### Contar columnas por UNION SELECT

En una UNION, los conjuntos de resultados deben contener el mismo número de columnas. Inyectar `' UNION (select null)--` producirá un error en el servidor.

```sql
SELECT * FROM someTable WHERE category = 'Accessories' UNION (select null)--'
```

Utilizo llamadas repetidas, aumentando cada vez el número de ``nulls``. El recuento de columnas correcto se puede encontrar inyectando `' UNION (select null, null, null)--`. Por lo tanto, esta consulta es válida:

```sql
SELECT * FROM someTable WHERE category = 'Accessories' UNION (select null, null, null)--'
```

Ahora sé que hay tres columnas y el laboratorio se actualiza a

![success]({{site.url}}/images/SQLi/sqli-3/success.png)

### Contar columnas por ORDER BY

Una forma alternativa de contar las columnas es con `ORDER BY`. Inyectando `' ORDER BY 1--` ordenará los resultados por la primera columna del resultado. Incrementar el valor conduce a un error interno del servidor cuando se utiliza `' ORDER BY 4--` ya que se ordena a la base de datos por una columna que no existe. Por lo tanto, el número correcto de columnas es 3.

```sql
SELECT * FROM someTable WHERE category = 'Accessories' ORDER BY 4--
```
