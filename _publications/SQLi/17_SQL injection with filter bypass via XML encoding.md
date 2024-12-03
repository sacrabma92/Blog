---
title: "17 SQL injection with filter bypass via XML encoding"
collection: publications
category: portswigger
permalink: publications/SQLi/17_SQL injection with filter bypass via XML encoding
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL en su función de comprobación de existencias. Los resultados de la consulta se devuelven en la respuesta de la aplicación, por lo que puede utilizar un ataque UNION para recuperar datos de otras tablas.'
date: 2024-12-02
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'SQL injection with filter bypass via XML encoding'
---

![logo]({{site.url}}/images/SQLi/sqli-17/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-17/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, realice un ataque de inyección SQL para recuperar las credenciales del usuario administrador y, a continuación, inicie sesión en su cuenta.

## Buscamos el vector de ataque

Realizamos busqueda de URL's que realicen consulta a la BD para ver si es una posible realizar un ataque de SQLi.

![Vector]({{site.url}}/images/SQLi/sqli-17/vector.png)

## Firewall

Al tratar de insertar codigo, nos detecta un Firewall y nos bloquea la consulta

![Detectado]({{site.url}}/images/SQLi/sqli-17/detectado.png)

## Ofuscar el codigo

Para prevenir que nos detecten podemos `Ofuscar` el codigo. Para Ofuscar BurpSuite tiene una extension la cual se puede instalar para realizar este trabajo, se llama `Hackvertor`.

![Hackvertor]({{site.url}}/images/SQLi/sqli-17/hackvertor.png)

La extensión nos ofusco la Query y nos arrojo la columna vulnerable.

![Ofuscado]({{site.url}}/images/SQLi/sqli-17/ofuscado.png)

## Obtenemos los datos 

Tener presente que toca ofuscar el codigo para saltar el firewall

```sql
UNION SELECT username || '==' || password FROM users
```

![Resultado]({{site.url}}/images/SQLi/sqli-17/resultado.png)

## Login

![Login]({{site.url}}/images/SQLi/sqli-17/login.png)

## Resuelto

![Aprobado]({{site.url}}/images/SQLi/sqli-17/aprobado.png)

