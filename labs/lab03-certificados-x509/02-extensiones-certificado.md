# Laboratorio 03 — Certificados X.509

## Explorar extensiones de certificados

Los certificados X.509 no solo contienen información básica como identidad, clave pública y periodo de validez.
También incluyen **extensiones**, que definen cómo puede utilizarse el certificado.

En este ejercicio exploraremos algunas de estas extensiones y veremos cómo afectan al uso de los certificados en TLS.

---

### Paso 1 — Buscar las extensiones del certificado

Utiliza OpenSSL para mostrar nuevamente el contenido del certificado que generaste anteriormente.

```bash
openssl x509 -in server.crt -text -noout
```

Desplázate por la salida hasta encontrar una sección similar a:

```
X509v3 extensions:
```

Debajo aparecerán diferentes extensiones.

Estas extensiones indican restricciones o usos permitidos para el certificado.

---

### Paso 2 — Identificar extensiones comunes

Dentro de la sección de extensiones busca entradas similares a:

```
X509v3 Subject Key Identifier
X509v3 Authority Key Identifier
X509v3 Basic Constraints
```

Estas extensiones permiten a los clientes entender cómo se relaciona el certificado con otros certificados de la cadena.

Por ejemplo:

* **Subject Key Identifier** identifica la clave del certificado
* **Authority Key Identifier** identifica la clave del emisor
* **Basic Constraints** indica si el certificado puede actuar como CA.

---

### Paso 3 — Extraer únicamente las extensiones del certificado

Podemos filtrar la salida para centrarnos solo en las extensiones.

Ejecuta:

```bash
openssl x509 -in server.crt -text -noout | grep -A5 "X509v3"
```

Esto mostrará las extensiones y algunas líneas adicionales de contexto.

Observa cómo cada extensión añade información adicional que puede influir en cómo se utiliza el certificado.

---

### Paso 4 — Generar un certificado con Subject Alternative Name

Muchos sistemas modernos ignoran el campo **Common Name** y utilizan la extensión **Subject Alternative Name (SAN)** para validar el nombre del servidor.

Las versiones recientes de OpenSSL (3.x) permiten añadir esta extensión directamente desde la línea de comandos sin necesidad de un archivo de configuración adicional.

Genera un nuevo certificado autofirmado con SAN ejecutando:

```bash
openssl req -new -x509 \
-key rsa-private.key \
-out server-san.crt \
-days 365 \
-subj "/CN=test.local" \
-addext "subjectAltName = DNS:test.local"
```

Cuando el comando finalice se generará un nuevo certificado llamado `server-san.crt` que incluye la extensión SAN adecuada.

---

### Paso 5 — Comprobar la extensión SAN en el certificado

Analiza el nuevo certificado generado.

```bash
openssl x509 -in server-san.crt -text -noout
```

Busca una sección similar a:

```
X509v3 Subject Alternative Name
```

Debajo aparecerá algo como:

```
DNS:test.local
```

Esto indica que el certificado es válido para ese nombre de host.

En conexiones TLS modernas esta extensión es esencial para que los clientes acepten el certificado.

---

En este ejercicio hemos explorado algunas de las extensiones que pueden incluir los certificados X.509 y cómo influyen en su uso dentro de TLS.
