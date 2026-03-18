# Laboratorio 08 — TLS en servicios

## Ejecutar un servidor HTTPS con NGINX

En los laboratorios anteriores generamos certificados utilizando nuestra propia autoridad certificadora.
Ahora utilizaremos esos certificados para proteger un servicio real.

En este ejercicio ejecutaremos un **servidor web NGINX dentro de Docker** utilizando el certificado generado anteriormente.

Esto nos permitirá probar conexiones TLS entre sistemas dentro del entorno Codespace.

Asegúrate de haber hecho antes el Lab05 (los tres ejercicios), para tener `server.crt`, `server.key` y `fullchain.crt` en `~/pki-labs/web-server`. Todo lo que hagas en este ejercicio va en la terminal del Codespace, en el directorio `~/tls-web` que creas en el paso 1; así, cuando arranques el contenedor, los volúmenes montan bien.

---

### Paso 1 — Preparar un directorio para el servidor web

Crea un directorio para el servidor de prueba.

```bash
mkdir ~/tls-web
cd ~/tls-web
```

Comprueba que te encuentras en el directorio correcto.

```bash
pwd
```

Este directorio contendrá los archivos que el servidor web utilizará.

---

### Paso 2 — Crear una página web simple

Crea un archivo HTML sencillo que servirá el servidor.

```bash
echo "Servidor HTTPS de prueba" > index.html
```

Comprueba el contenido del archivo.

```bash
cat index.html
```

Este archivo será la página que servirá el servidor HTTPS.

---

### Paso 3 — Copiar los certificados necesarios

Copia el certificado y la clave privada generados anteriormente.

```bash
cp ~/pki-labs/web-server/server.crt .
cp ~/pki-labs/web-server/server.key .
cp ~/pki-labs/web-server/fullchain.crt .
```

Comprueba que los archivos están disponibles.

```bash
ls
```

Deberías ver algo similar a:

```
index.html
server.crt
server.key
fullchain.crt
```

Estos archivos se utilizarán para configurar TLS en NGINX.

---

### Paso 4 — Crear la configuración de NGINX

Crea un archivo de configuración para el servidor.

```bash
nano nginx.conf
```

Añade el siguiente contenido:

```
events {}

http {
    server {
        listen 443 ssl;

        ssl_certificate /etc/nginx/certs/fullchain.crt;
        ssl_certificate_key /etc/nginx/certs/server.key;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
}
```

Guarda el archivo.

Esta configuración indica a NGINX que escuche en el puerto 443 y utilice los certificados proporcionados.

---

### Paso 5 — Crear una red Docker para el laboratorio

Para que **otros contenedores puedan resolver el servidor por nombre** (sin Docker Compose), vamos a usar una red Docker *user-defined*, que sí incluye DNS interno.

En el host (Codespace), crea la red si no existe:

```bash
docker network inspect lab08-net >/dev/null 2>&1 || docker network create lab08-net
```

---

### Paso 6 — Iniciar el servidor NGINX con Docker

Desde `~/tls-web` (para que los volúmenes monten bien), ejecuta el servidor con Docker:

```bash
docker run -d \
--name lab08-nginx \
--network lab08-net \
--network-alias web.test.local \
-p 8443:443 \
-v $(pwd)/index.html:/usr/share/nginx/html/index.html \
-v $(pwd)/nginx.conf:/etc/nginx/nginx.conf \
-v $(pwd)/fullchain.crt:/etc/nginx/certs/fullchain.crt \
-v $(pwd)/server.key:/etc/nginx/certs/server.key \
nginx
```

Comprueba que el contenedor se está ejecutando.

```bash
docker ps
```

Deberías ver un contenedor NGINX activo.

---

### Paso 7 — Probar la conexión HTTPS

Desde la terminal del Codespace (da igual el directorio), prueba a conectarte al servidor:

```bash
curl https://localhost:8443
```

Si el certificado aún no es reconocido por el sistema puede aparecer un error de verificación.

En ese caso prueba con:

```bash
curl -k https://localhost:8443
```

La salida debería mostrar:

```
Servidor HTTPS de prueba
```

Esto confirma que el servidor está utilizando TLS correctamente y que el certificado generado en los laboratorios se está utilizando para proteger la conexión.
